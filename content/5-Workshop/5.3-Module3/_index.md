---
title: "Module 3: Hệ Thống Duyệt Admin"
date: "2025-01-15T11:00:00+07:00"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Tổng quan

Trong module này, bạn sẽ xây dựng quy trình duyệt admin hoàn chỉnh cho nền tảng Thư Viện Online. Bạn sẽ triển khai kiểm soát truy cập dựa trên vai trò sử dụng Cognito User Groups, tạo các hàm Lambda cho logic phê duyệt/từ chối, và xây dựng giao diện bảng điều khiển admin.

**Thời gian thực hiện:** ~90 phút

**Các dịch vụ sử dụng:**
- Amazon Cognito (User Groups)
- AWS Lambda (logic phê duyệt/từ chối)
- Amazon S3 (các thao tác file)
- Amazon DynamoDB (cập nhật trạng thái)
- Amazon API Gateway (các endpoint admin)

---

## Những gì bạn sẽ học được

- Cấu hình Cognito User Groups cho kiểm soát dựa trên vai trò
- Triển khai các hàm Lambda chỉ dành cho admin
- Xây dựng các thao tác sao chép file giữa các thư mục S3
- Cập nhật trạng thái DynamoDB với quy trình phê duyệt
- Tạo giao diện bảng điều khiển admin với các điều khiển phê duyệt
- Triển khai ghi nhật ký audit cho tính minh bạch

---

## Kiến trúc cho module này

```
Người dùng Admin → Bảng điều khiển → API Gateway → Lambda (approveBook/rejectBook)
                                        ↓
                                  Kiểm tra vai trò Admin (JWT)
                                        ↓
                                  Sao chép S3: uploads/ → public/books/
                                        ↓
                                  Cập nhật DynamoDB (APPROVED/REJECTED)
                                        ↓
                                  Ghi vào CloudWatch
```

---

## Bước 1: Cấu hình Cognito User Groups

### 1.1 Tạo nhóm Admin qua CDK

Cập nhật `lib/online-library-cdk-stack.ts`:

```typescript
// đường dẫn: lib/online-library-cdk-stack.ts
import * as cognito from 'aws-cdk-lib/aws-cognito';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ...mã Cognito User Pool hiện có...

    // Tạo nhóm Admin
    const adminGroup = new cognito.CfnUserPoolGroup(this, 'AdminGroup', {
      userPoolId: userPool.userPoolId,
      groupName: 'Admins',
      description: 'Người dùng quản trị viên có quyền phê duyệt',
      precedence: 1,
    });

    // Tạo nhóm Người dùng (tùy chọn, để rõ ràng hơn)
    const usersGroup = new cognito.CfnUserPoolGroup(this, 'UsersGroup', {
      userPoolId: userPool.userPoolId,
      groupName: 'Users',
      description: 'Người dùng thông thường có quyền tải lên',
      precedence: 10,
    });

    // Xuất tên nhóm
    new cdk.CfnOutput(this, 'AdminGroupName', {
      value: adminGroup.groupName!,
      exportName: 'OnlineLibraryAdminGroup',
    });
  }
}
```

### 1.2 Thêm người dùng vào nhóm Admin

Qua AWS CLI:

```bash
# Thêm người dùng vào nhóm Admins
aws cognito-idp admin-add-user-to-group \
  --user-pool-id YOUR_USER_POOL_ID \
  --username admin@example.com \
  --group-name Admins

# Xác minh tư cách thành viên nhóm
aws cognito-idp admin-list-groups-for-user \
  --user-pool-id YOUR_USER_POOL_ID \
  --username admin@example.com
```

---

## Bước 2: Tạo các hàm Lambda phê duyệt

### 2.1 Tạo hàm Lambda approveBook

Tạo `lambda/approveBook/index.py`:

```python
# đường dẫn: lambda/approveBook/index.py
import json
import boto3
from datetime import datetime
import os

s3_client = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')

UPLOADS_BUCKET = os.environ['UPLOADS_BUCKET']
BOOKS_BUCKET = os.environ['BOOKS_BUCKET']
BOOKS_TABLE = os.environ['BOOKS_TABLE']

table = dynamodb.Table(BOOKS_TABLE)

def lambda_handler(event, context):
    """
    Phê duyệt tải sách lên và di chuyển đến thư mục công khai
    """
    try:
        # Xác thực vai trò admin
        claims = event['requestContext']['authorizer']['jwt']['claims']
        groups = claims.get('cognito:groups', '').split(',')
        
        if 'Admins' not in groups:
            return {
                'statusCode': 403,
                'body': json.dumps({
                    'error': 'Cấm truy cập: Cần quyền admin'
                })
            }
        
        # Phân tích request
        body = json.loads(event['body'])
        book_id = body.get('bookId')
        approval_notes = body.get('notes', '')
        
        if not book_id:
            return {
                'statusCode': 400,
                'body': json.dumps({
                    'error': 'Thiếu bookId'
                })
            }
        
        # Lấy metadata sách từ DynamoDB
        response = table.get_item(
            Key={
                'bookId': book_id,
                'version': 1
            }
        )
        
        if 'Item' not in response:
            return {
                'statusCode': 404,
                'body': json.dumps({
                    'error': 'Sách không tồn tại'
                })
            }
        
        book = response['Item']
        
        # Kiểm tra trạng thái sách
        if book['status'] != 'PENDING':
            return {
                'statusCode': 400,
                'body': json.dumps({
                    'error': f'Trạng thái sách là {book["status"]}, không phải PENDING'
                })
            }
        
        # Sao chép file từ uploads/ sang public/books/
        source_key = book['s3Key']  # ví dụ: pending/book-id.pdf
        file_extension = source_key.split('.')[-1]
        destination_key = f"public/books/{book_id}.{file_extension}"
        
        # Sao chép đối tượng
        s3_client.copy_object(
            CopySource={
                'Bucket': UPLOADS_BUCKET,
                'Key': source_key
            },
            Bucket': BOOKS_BUCKET,
            Key': destination_key
        )
        
        # Xóa bản gốc trong uploads/
        s3_client.delete_object(
            Bucket=UPLOADS_BUCKET,
            Key=source_key
        )
        
        # Cập nhật DynamoDB
        admin_id = claims['sub']
        admin_email = claims.get('email', 'unknown')
        approval_timestamp = datetime.utcnow().isoformat()
        
        table.update_item(
            Key={
                'bookId': book_id,
                'version': 1
            },
            UpdateExpression='''
                SET #status = :status,
                    approvedBy = :admin_id,
                    approverEmail = :admin_email,
                    approvalTimestamp = :timestamp,
                    approvalNotes = :notes,
                    publicS3Key = :public_key,
                    publicBucket = :public_bucket
            ''',
            ExpressionAttributeNames={
                '#status': 'status'
            },
            ExpressionAttributeValues={
                ':status': 'APPROVED',
                ':admin_id': admin_id,
                ':admin_email': admin_email,
                ':timestamp': approval_timestamp,
                ':notes': approval_notes,
                ':public_key': destination_key,
                ':public_bucket': BOOKS_BUCKET
            }
        )
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
            },
            'body': json.dumps({
                'message': 'Phê duyệt sách thành công',
                'bookId': book_id,
                'approvedBy': admin_email,
                'timestamp': approval_timestamp
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': 'Lỗi máy chủ nội bộ'
            })
        }
```

### 2.2 Tạo hàm Lambda rejectBook

Tạo `lambda/rejectBook/index.py`:

```python
# đường dẫn: lambda/rejectBook/index.py
import json
import boto3
from datetime import datetime
import os

s3_client = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')

UPLOADS_BUCKET = os.environ['UPLOADS_BUCKET']
BOOKS_TABLE = os.environ['BOOKS_TABLE']

table = dynamodb.Table(BOOKS_TABLE)

def lambda_handler(event, context):
    """
    Từ chối tải sách lên với lý do
    """
    try:
        # Xác thực vai trò admin
        claims = event['requestContext']['authorizer']['jwt']['claims']
        groups = claims.get('cognito:groups', '').split(',')
        
        if 'Admins' not in groups:
            return {
                'statusCode': 403,
                'body': json.dumps({
                    'error': 'Cấm truy cập: Cần quyền admin'
                })
            }
        
        # Phân tích request
        body = json.loads(event['body'])
        book_id = body.get('bookId')
        rejection_reason = body.get('reason', 'Không có lý do được cung cấp')
        rejection_category = body.get('category', 'other')
        
        if not book_id:
            return {
                'statusCode': 400,
                'body': json.dumps({
                    'error': 'Thiếu bookId'
                })
            }
        
        # Xác thực danh mục từ chối
        valid_categories = ['copyright', 'inappropriate', 'quality', 'format', 'other']
        if rejection_category not in valid_categories:
            rejection_category = 'other'
        
        # Lấy metadata sách
        response = table.get_item(
            Key={
                'bookId': book_id,
                'version': 1
            }
        )
        
        if 'Item' not in response:
            return {
                'statusCode': 404,
                'body': json.dumps({
                    'error': 'Sách không tồn tại'
                })
            }
        
        book = response['Item']
        
        # Kiểm tra trạng thái sách
        if book['status'] != 'PENDING':
            return {
                'statusCode': 400,
                'body': json.dumps({
                    'error': f'Trạng thái sách là {book["status"]}, không phải PENDING'
                })
            }
        
        # Tùy chọn: Di chuyển sang khu vực cách ly thay vì xóa
        source_key = book['s3Key']
        quarantine_key = source_key.replace('pending/', 'quarantine/')
        
        s3_client.copy_object(
            CopySource={
                'Bucket': UPLOADS_BUCKET,
                'Key': source_key
            },
            Bucket=UPLOADS_BUCKET,
            Key=quarantine_key
        )
        
        # Xóa bản gốc
        s3_client.delete_object(
            Bucket=UPLOADS_BUCKET,
            Key=source_key
        )
        
        # Cập nhật DynamoDB
        admin_id = claims['sub']
        admin_email = claims.get('email', 'unknown')
        rejection_timestamp = datetime.utcnow().isoformat()
        
        table.update_item(
            Key={
                'bookId': book_id,
                'version': 1
            },
            UpdateExpression='''
                SET #status = :status,
                    rejectedBy = :admin_id,
                    rejectorEmail = :admin_email,
                    rejectionTimestamp = :timestamp,
                    rejectionReason = :reason,
                    rejectionCategory = :category,
                    quarantineS3Key = :quarantine_key
            ''',
            ExpressionAttributeNames={
                '#status': 'status'
            },
            ExpressionAttributeValues={
                ':status': 'REJECTED',
                ':admin_id': admin_id,
                ':admin_email': admin_email,
                ':timestamp': rejection_timestamp,
                ':reason': rejection_reason,
                ':category': rejection_category,
                ':quarantine_key': quarantine_key
            }
        )
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
            },
            'body': json.dumps({
                'message': 'Sách đã bị từ chối thành công',
                'bookId': book_id,
                'rejectedBy': admin_email,
                'reason': rejection_reason,
                'timestamp': rejection_timestamp
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': 'Lỗi máy chủ nội bộ'
            })
        }
```

### 2.3 Thêm các hàm Lambda vào CDK Stack

```typescript
// đường dẫn: lib/online-library-cdk-stack.ts
// ...mã hiện có...

// Tạo hàm Lambda phê duyệt
const approveBookFunction = new lambda.Function(this, 'ApproveBook', {
  runtime: lambda.Runtime.PYTHON_3_9,
  handler: 'index.lambda_handler',
  code: lambda.Code.fromAsset('lambda/approveBook'),
  environment: {
    UPLOADS_BUCKET: uploadsBucket.bucketName,
    BOOKS_BUCKET: booksBucket.bucketName,
    BOOKS_TABLE: booksTable.tableName,
  },
  timeout: cdk.Duration.seconds(30),
});

// Cấp quyền
uploadsBucket.grantRead(approveBookFunction);
uploadsBucket.grantDelete(approveBookFunction);
booksBucket.grantPut(approveBookFunction);
booksTable.grantReadWriteData(approveBookFunction);

// Tạo hàm Lambda từ chối
const rejectBookFunction = new lambda.Function(this, 'RejectBook', {
  runtime: lambda.Runtime.PYTHON_3_9,
  handler: 'index.lambda_handler',
  code: lambda.Code.fromAsset('lambda/rejectBook'),
  environment: {
    UPLOADS_BUCKET: uploadsBucket.bucketName,
    BOOKS_TABLE: booksTable.tableName,
  },
  timeout: cdk.Duration.seconds(30),
});

// Cấp quyền
uploadsBucket.grantReadWrite(rejectBookFunction);
booksTable.grantReadWriteData(rejectBookFunction);

// Thêm các route
httpApi.addRoutes({
  path: '/admin/approve',
  methods: [apigateway.HttpMethod.POST],
  integration: new integrations.HttpLambdaIntegration(
    'ApproveBookIntegration',
    approveBookFunction
  ),
  authorizer: jwtAuthorizer,
});

httpApi.addRoutes({
  path: '/admin/reject',
  methods: [apigateway.HttpMethod.POST],
  integration: new integrations.HttpLambdaIntegration(
    'RejectBookIntegration',
    rejectBookFunction
  ),
  authorizer: jwtAuthorizer,
});
```

---

## Bước 3: Xây dựng giao diện bảng điều khiển Admin

### 3.1 Tạo Client API Admin

Tạo `src/lib/api/admin.ts`:

```typescript
// đường dẫn: src/lib/api/admin.ts
import { fetchAuthSession } from 'aws-amplify/auth';

export interface PendingBook {
  bookId: string;
  title: string;
  author: string;
  description?: string;
  fileSize: number;
  fileType: string;
  uploadTimestamp: string;
  uploaderEmail: string;
  status: string;
}

export async function getPendingBooks(): Promise<PendingBook[]> {
  const session = await fetchAuthSession();
  const idToken = session.tokens?.idToken?.toString();

  const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/admin/pending`, {
    headers: {
      'Authorization': `Bearer ${idToken}`,
    },
  });

  if (!response.ok) {
    throw new Error('Failed to fetch pending books');
  }

  return response.json();
}

export async function approveBook(bookId: string, notes?: string): Promise<void> {
  const session = await fetchAuthSession();
  const idToken = session.tokens?.idToken?.toString();

  const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/admin/approve`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${idToken}`,
    },
    body: JSON.stringify({ bookId, notes }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'Failed to approve book');
  }
}

export async function rejectBook(
  bookId: string,
  reason: string,
  category: string
): Promise<void> {
  const session = await fetchAuthSession();
  const idToken = session.tokens?.idToken?.toString();

  const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/admin/reject`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${idToken}`,
    },
    body: JSON.dumps({ bookId, reason, category }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'Failed to reject book');
  }
}
```

### 3.2 Tạo Component Bảng điều khiển Admin

Tạo `src/components/admin/PendingBooksList.tsx`:

```typescript
// đường dẫn: src/components/admin/PendingBooksList.tsx
'use client';

import { useState, useEffect } from 'react';
import { getPendingBooks, approveBook, rejectBook, PendingBook } from '@/lib/api/admin';

export function PendingBooksList() {
  const [books, setBooks] = useState<PendingBook[]>([]);
  const [loading, setLoading] = useState(true);
  const [selectedBook, setSelectedBook] = useState<PendingBook | null>(null);
  const [action, setAction] = useState<'approve' | 'reject' | null>(null);
  const [rejectionReason, setRejectionReason] = useState('');
  const [rejectionCategory, setRejectionCategory] = useState('other');
  const [approvalNotes, setApprovalNotes] = useState('');
  const [processing, setProcessing] = useState(false);

  useEffect(() => {
    loadPendingBooks();
  }, []);

  const loadPendingBooks = async () => {
    try {
      const data = await getPendingBooks();
      setBooks(data);
    } catch (error) {
      console.error('Error loading pending books:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleApprove = async () => {
    if (!selectedBook) return;
    
    setProcessing(true);
    try {
      await approveBook(selectedBook.bookId, approvalNotes);
      setBooks(books.filter(b => b.bookId !== selectedBook.bookId));
      setSelectedBook(null);
      setAction(null);
      setApprovalNotes('');
    } catch (error) {
      alert('Failed to approve book: ' + error);
    } finally {
      setProcessing(false);
    }
  };

  const handleReject = async () => {
    if (!selectedBook || !rejectionReason) return;
    
    setProcessing(true);
    try {
      await rejectBook(selectedBook.bookId, rejectionReason, rejectionCategory);
      setBooks(books.filter(b => b.bookId !== selectedBook.bookId));
      setSelectedBook(null);
      setAction(null);
      setRejectionReason('');
    } catch (error) {
      alert('Failed to reject book: ' + error);
    } finally {
      setProcessing(false);
    }
  };

  if (loading) {
    return <div className="text-center py-8">Đang tải danh sách sách chờ duyệt...</div>;
  }

  return (
    <div className="space-y-6">
      <h2 className="text-2xl font-bold">Sách Chờ Duyệt ({books.length})</h2>

      {books.length === 0 ? (
        <p className="text-gray-600">Không có sách nào chờ duyệt</p>
      ) : (
        <div className="grid gap-4">
          {books.map((book) => (
            <div key={book.bookId} className="border rounded-lg p-4 bg-white shadow">
              <div className="flex justify-between items-start">
                <div className="flex-1">
                  <h3 className="font-semibold text-lg">{book.title}</h3>
                  <p className="text-gray-600">tác giả {book.author}</p>
                  {book.description && (
                    <p className="text-sm text-gray-500 mt-2">{book.description}</p>
                  )}
                  <div className="mt-2 text-sm text-gray-500">
                    <p>Người tải lên: {book.uploaderEmail}</p>
                    <p>Kích thước: {(book.fileSize / 1024 / 1024).toFixed(2)} MB</p>
                    <p>Ngày tải lên: {new Date(book.uploadTimestamp).toLocaleString()}</p>
                  </div>
                </div>
                
                <div className="flex gap-2 ml-4">
                  <button
                    onClick={() => {
                      setSelectedBook(book);
                      setAction('approve');
                    }}
                    className="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700"
                  >
                    Duyệt
                  </button>
                  <button
                    onClick={() => {
                      setSelectedBook(book);
                      setAction('reject');
                    }}
                    className="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
                  >
                    Từ chối
                  </button>
                </div>
              </div>
            </div>
          ))}
        </div>
      )}

      {/* Approval Modal */}
      {action === 'approve' && selectedBook && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
          <div className="bg-white rounded-lg p-6 max-w-md w-full">
            <h3 className="text-xl font-bold mb-4">Phê duyệt Sách</h3>
            <p className="mb-4">
              Bạn có chắc chắn muốn phê duyệt "{selectedBook.title}" không?
            </p>
            
            <div className="mb-4">
              <label className="block text-sm font-medium mb-2">
                Ghi chú phê duyệt (tùy chọn)
              </label>
              <textarea
                value={approvalNotes}
                onChange={(e) => setApprovalNotes(e.target.value)}
                className="w-full px-3 py-2 border rounded"
                rows={3}
                placeholder="Thêm bất kỳ ghi chú nào về phê duyệt này..."
              />
            </div>
            
            <div className="flex gap-2">
              <button
                onClick={handleApprove}
                disabled={processing}
                className="flex-1 px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700 disabled:bg-gray-300"
              >
                {processing ? 'Đang xử lý...' : 'Xác nhận phê duyệt'}
              </button>
              <button
                onClick={() => {
                  setAction(null);
                  setSelectedBook(null);
                  setApprovalNotes('');
                }}
                disabled={processing}
                className="px-4 py-2 border rounded hover:bg-gray-100"
              >
                Hủy
              </button>
            </div>
          </div>
        </div>
      )}

      {/* Rejection Modal */}
      {action === 'reject' && selectedBook && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
          <div className="bg-white rounded-lg p-6 max-w-md w-full">
            <h3 className="text-xl font-bold mb-4">Từ chối Sách</h3>
            <p className="mb-4">
              Vui lòng cung cấp lý do từ chối cho "{selectedBook.title}":
            </p>
            
            <div className="mb-4">
              <label className="block text-sm font-medium mb-2">Danh mục</label>
              <select
                value={rejectionCategory}
                onChange={(e) => setRejectionCategory(e.target.value)}
                className="w-full px-3 py-2 border rounded"
              >
                <option value="copyright">Vấn đề bản quyền</option>
                <option value="inappropriate">Nội dung không phù hợp</option>
                <option value="quality">Chất lượng kém</option>
                <option value="format">Định dạng không hợp lệ</option>
                <option value="other">Khác</option>
              </select>
            </div>
            
            <div className="mb-4">
              <label className="block text-sm font-medium mb-2">Lý do *</label>
              <textarea
                value={rejectionReason}
                onChange={(e) => setRejectionReason(e.target.value)}
                className="w-full px-3 py-2 border rounded"
                rows={4}
                placeholder="Giải thích lý do từ chối sách này..."
                required
              />
            </div>
            
            <div className="flex gap-2">
              <button
                onClick={handleReject}
                disabled={processing || !rejectionReason}
                className="flex-1 px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 disabled:bg-gray-300"
              >
                {processing ? 'Đang xử lý...' : 'Xác nhận từ chối'}
              </button>
              <button
                onClick={() => {
                  setAction(null);
                  setSelectedBook(null);
                  setRejectionReason('');
                }}
                disabled={processing}
                className="px-4 py-2 border rounded hover:bg-gray-100"
              >
                Hủy
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## Xác minh & Kiểm thử

### Danh sách kiểm tra

- ✅ Nhóm Admin trong Cognito được tạo và người dùng được gán
- ✅ Lambda phê duyệt sao chép file chính xác
- ✅ Lambda từ chối di chuyển file vào khu vực cách ly
- ✅ DynamoDB cập nhật đúng trạng thái và metadata
- ✅ Bảng điều khiển admin hiển thị sách chờ duyệt
- ✅ Modal phê duyệt hoạt động và cập nhật UI
- ✅ Modal từ chối yêu cầu lý do
- ✅ Người dùng không phải admin không truy cập được các endpoint admin

### Các vấn đề thường gặp & Giải pháp

**Vấn đề:** JWT không chứa claim cognito:groups  
**Giải pháp:** Đảm bảo người dùng được gán vào nhóm và đăng nhập lại để nhận token mới

**Vấn đề:** Sao chép S3 thất bại với lỗi quyền truy cập  
**Giải pháp:** Xác minh Lambda có quyền IAM chính xác cho cả hai bucket

**Vấn đề:** Bảng điều khiển admin hiển thị danh sách trống  
**Giải pháp:** Kiểm tra Lambda getPendingBooks truy vấn đúng GSI StatusIndex

---

## Dọn dẹp (Tùy chọn)

```bash
cdk destroy
```

---

## Bước tiếp theo

Chúc mừng! Bạn đã hoàn thành Module 3. Bạn đã có:
- Kiểm soát truy cập dựa trên vai trò với Cognito User Groups
- Các hàm Lambda cho quy trình phê duyệt và từ chối
- Các thao tác file S3 với quyền hạn chế
- Bảng điều khiển admin với các điều khiển phê duyệt

**Tiếp tục với:** [Module 4: Phân phối nội dung](../5.4-module4/)

---

## Tài nguyên bổ sung

- [Tài liệu về Cognito User Groups](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-user-groups.html)
- [Các thao tác sao chép đối tượng S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/copy-object.html)
- [Các biểu thức cập nhật DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.UpdateExpressions.html)