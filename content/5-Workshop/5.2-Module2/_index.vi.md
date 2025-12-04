---
title: "Module 2: Hạ tầng Upload"
date: "2025-01-15T10:00:00+07:00"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Tổng quan

Trong module này, bạn sẽ xây dựng hạ tầng upload hoàn chỉnh cho nền tảng Thư viện Online. Bạn sẽ triển khai tạo S3 presigned URL, tạo quy trình upload với theo dõi tiến trình, và lưu metadata trong DynamoDB với quản lý trạng thái phù hợp.

**Thời lượng:** ~90 phút

**Dịch vụ sử dụng:**
- Amazon S3 (buckets và presigned URLs)
- AWS Lambda (tạo upload URL)
- Amazon DynamoDB (lưu trữ metadata)
- Amazon API Gateway (upload endpoints)

---

## Những gì bạn sẽ học

- Tạo và cấu hình S3 buckets với policies phù hợp
- Triển khai tạo presigned URL cho uploads an toàn
- Xây dựng form upload với theo dõi tiến trình thời gian thực
- Lưu trữ và quản lý upload metadata trong DynamoDB
- Triển khai validation file và giới hạn kích thước
- Xử lý lỗi upload và retry logic

---

## Kiến trúc cho Module này

```
User → Form Upload → API Gateway → Lambda (createUploadUrl)
                                    ↓
                              Tạo Presigned PUT URL
                                    ↓
                              Ghi DynamoDB (PENDING)
                                    ↓
User ← Presigned URL ← Response
     ↓
Upload trực tiếp lên S3 (thư mục uploads/)
```

---

## Bước 1: Tạo S3 Buckets với CDK

### 1.1 Cập nhật CDK Stack

Chỉnh sửa `lib/online-library-cdk-stack.ts`:

```typescript
// filepath: lib/online-library-cdk-stack.ts
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as iam from 'aws-cdk-lib/aws-iam';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ...code Cognito hiện có...

    // Tạo S3 Buckets
    const uploadsBucket = new s3.Bucket(this, 'UploadsBucket', {
      bucketName: `online-library-uploads-${this.account}`,
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      lifecycleRules: [
        {
          id: 'DeleteOldUploads',
          enabled: true,
          expiration: cdk.Duration.days(3), // Tự động xóa sau 3 ngày
          prefix: 'pending/',
        },
      ],
      cors: [
        {
          allowedMethods: [
            s3.HttpMethods.GET,
            s3.HttpMethods.PUT,
            s3.HttpMethods.POST,
          ],
          allowedOrigins: ['*'], // Hạn chế trong production
          allowedHeaders: ['*'],
          exposedHeaders: ['ETag'],
        },
      ],
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Chỉ cho dev
      autoDeleteObjects: true, // Chỉ cho dev
    });

    const booksBucket = new s3.Bucket(this, 'BooksBucket', {
      bucketName: `online-library-books-${this.account}`,
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      versioned: true, // Bật versioning để an toàn
      removalPolicy: cdk.RemovalPolicy.RETAIN, // Giữ trong production
    });

    // Output tên buckets
    new cdk.CfnOutput(this, 'UploadsBucketName', {
      value: uploadsBucket.bucketName,
      exportName: 'OnlineLibraryUploadsBucket',
    });

    new cdk.CfnOutput(this, 'BooksBucketName', {
      value: booksBucket.bucketName,
      exportName: 'OnlineLibraryBooksBucket',
    });
  }
}
```

### 1.2 Triển khai Stack đã cập nhật

```bash
cdk synth
cdk deploy
```

---

## Bước 2: Tạo DynamoDB Table cho Metadata

### 2.1 Thêm DynamoDB Table vào CDK

```typescript
// filepath: lib/online-library-cdk-stack.ts
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ...code hiện có...

    // Tạo DynamoDB Table
    const booksTable = new dynamodb.Table(this, 'BooksTable', {
      tableName: 'online-library-books',
      partitionKey: {
        name: 'bookId',
        type: dynamodb.AttributeType.STRING,
      },
      sortKey: {
        name: 'version',
        type: dynamodb.AttributeType.NUMBER,
      },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true,
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Chỉ cho dev
    });

    // Thêm GSI cho queries theo status
    booksTable.addGlobalSecondaryIndex({
      indexName: 'StatusIndex',
      partitionKey: {
        name: 'status',
        type: dynamodb.AttributeType.STRING,
      },
      sortKey: {
        name: 'uploadTimestamp',
        type: dynamodb.AttributeType.STRING,
      },
      projectionType: dynamodb.ProjectionType.ALL,
    });

    // Thêm GSI cho queries theo uploader
    booksTable.addGlobalSecondaryIndex({
      indexName: 'UploaderIndex',
      partitionKey: {
        name: 'uploaderId',
        type: dynamodb.AttributeType.STRING,
      },
      sortKey: {
        name: 'uploadTimestamp',
        type: dynamodb.AttributeType.STRING,
      },
      projectionType: dynamodb.ProjectionType.ALL,
    });

    // Output tên table
    new cdk.CfnOutput(this, 'BooksTableName', {
      value: booksTable.tableName,
      exportName: 'OnlineLibraryBooksTable',
    });
  }
}
```

---

## Xác minh & Kiểm thử

### Checklist

- S3 buckets được tạo với policies đúng
- DynamoDB table được tạo với GSIs
- Lambda tạo presigned URLs thành công
- API Gateway route được bảo vệ bằng JWT
- Form upload validate loại file và kích thước
- Theo dõi tiến trình hoạt động khi upload
- Metadata được lưu trong DynamoDB với status PENDING
- Files được upload đến đúng vị trí S3

### Các vấn đề thường gặp & Giải pháp

**Vấn đề:** Presigned URL hết hạn trong quá trình upload  
**Giải pháp:** Tăng thời gian hết hạn hoặc triển khai logic refresh URL

**Vấn đề:** Lỗi CORS khi upload lên S3  
**Giải pháp:** Xác minh cấu hình CORS của S3 bucket khớp với frontend domain của bạn

**Vấn đề:** Upload file thất bại im lặng  
**Giải pháp:** Kiểm tra browser console và CloudWatch logs của Lambda để tìm lỗi

---

## Dọn dẹp (Tùy chọn)

Để xóa các tài nguyên được tạo trong module này:

```bash
cdk destroy
```

---

## Bước tiếp theo

Chúc mừng! Bạn đã hoàn thành Module 2. Bây giờ bạn có:
- S3 buckets cho uploads và sách đã duyệt
- Lambda function tạo presigned URLs an toàn
- DynamoDB table lưu trữ upload metadata
- Form upload với theo dõi tiến trình

**Tiếp tục đến:** [Module 3: Hệ thống duyệt Admin](../5.3-module3/)

---

## Tài nguyên bổ sung

- [Tài liệu S3 Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [Lambda với API Gateway](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html)
