---
title: "Module 2: Upload Infrastructure"
date: "2025-01-15T10:00:00+07:00"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Overview

In this module, you'll build the complete upload infrastructure for the Online Library platform. You'll implement S3 presigned URL generation, create the upload workflow with progress tracking, and store metadata in DynamoDB with proper status management.

**Duration:** ~90 minutes

**Services Used:**
- Amazon S3 (buckets and presigned URLs)
- AWS Lambda (upload URL generation)
- Amazon DynamoDB (metadata storage)
- Amazon API Gateway (upload endpoints)

---

## What You'll Learn

- Create and configure S3 buckets with proper policies
- Implement presigned URL generation for secure uploads
- Build upload forms with real-time progress tracking
- Store and manage upload metadata in DynamoDB
- Implement file validation and size restrictions
- Handle upload errors and retry logic

---

## Architecture for This Module

```
User → Upload Form → API Gateway → Lambda (createUploadUrl)
                                    ↓
                              Generate Presigned PUT URL
                                    ↓
                              Write DynamoDB (PENDING)
                                    ↓
User ← Presigned URL ← Response
     ↓
Direct Upload to S3 (uploads/ folder)
```

---

## Step 1: Create S3 Buckets with CDK

### 1.1 Update CDK Stack

Edit `lib/online-library-cdk-stack.ts`:

```typescript
// filepath: lib/online-library-cdk-stack.ts
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as iam from 'aws-cdk-lib/aws-iam';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ...existing Cognito code...

    // Create S3 Buckets
    const uploadsBucket = new s3.Bucket(this, 'UploadsBucket', {
      bucketName: `online-library-uploads-${this.account}`,
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      lifecycleRules: [
        {
          id: 'DeleteOldUploads',
          enabled: true,
          expiration: cdk.Duration.days(3), // Auto-delete after 3 days
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
          allowedOrigins: ['*'], // Restrict in production
          allowedHeaders: ['*'],
          exposedHeaders: ['ETag'],
        },
      ],
      removalPolicy: cdk.RemovalPolicy.DESTROY, // For dev only
      autoDeleteObjects: true, // For dev only
    });

    const booksBucket = new s3.Bucket(this, 'BooksBucket', {
      bucketName: `online-library-books-${this.account}`,
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      versioned: true, // Enable versioning for safety
      removalPolicy: cdk.RemovalPolicy.RETAIN, // Keep in production
    });

    // Output bucket names
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

### 1.2 Deploy Updated Stack

```bash
cdk synth
cdk deploy
```

---

## Step 2: Create DynamoDB Table for Metadata

### 2.1 Add DynamoDB Table to CDK

```typescript
// filepath: lib/online-library-cdk-stack.ts
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ...existing code...

    // Create DynamoDB Table
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
      removalPolicy: cdk.RemovalPolicy.DESTROY, // For dev only
    });

    // Add GSI for status queries
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

    // Add GSI for uploader queries
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

    // Output table name
    new cdk.CfnOutput(this, 'BooksTableName', {
      value: booksTable.tableName,
      exportName: 'OnlineLibraryBooksTable',
    });
  }
}
```

---

## Step 3: Create Lambda for Presigned URL Generation

### 3.1 Create Lambda Function Code

Create `lambda/createUploadUrl/index.py`:

```python
# filepath: lambda/createUploadUrl/index.py
import json
import boto3
import uuid
from datetime import datetime
import os

s3_client = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')

UPLOADS_BUCKET = os.environ['UPLOADS_BUCKET']
BOOKS_TABLE = os.environ['BOOKS_TABLE']

table = dynamodb.Table(BOOKS_TABLE)

def lambda_handler(event, context):
    """
    Generate presigned PUT URL for file upload
    """
    try:
        # Parse request body
        body = json.loads(event['body'])
        
        # Extract metadata
        title = body.get('title')
        author = body.get('author')
        description = body.get('description', '')
        file_type = body.get('fileType')
        file_size = body.get('fileSize')
        
        # Validate required fields
        if not all([title, author, file_type, file_size]):
            return {
                'statusCode': 400,
                'body': json.dumps({
                    'error': 'Missing required fields'
                })
            }
        
        # Validate file type
        allowed_types = ['application/pdf', 'application/epub+zip']
        if file_type not in allowed_types:
            return {
                'statusCode': 400,
                'body': json.dumps({
                    'error': 'Invalid file type. Only PDF and ePub allowed'
                })
            }
        
        # Validate file size (max 50MB)
        max_size = 50 * 1024 * 1024  # 50MB
        if file_size > max_size:
            return {
                'statusCode': 400,
                'body': json.dumps({
                    'error': f'File too large. Maximum size is 50MB'
                })
            }
        
        # Get user info from JWT
        claims = event['requestContext']['authorizer']['jwt']['claims']
        uploader_id = claims['sub']
        uploader_email = claims['email']
        
        # Generate unique book ID
        book_id = str(uuid.uuid4())
        
        # Generate S3 key
        file_extension = 'pdf' if file_type == 'application/pdf' else 'epub'
        s3_key = f"pending/{book_id}.{file_extension}"
        
        # Generate presigned URL (15 minutes expiration)
        presigned_url = s3_client.generate_presigned_url(
            'put_object',
            Params={
                'Bucket': UPLOADS_BUCKET,
                'Key': s3_key,
                'ContentType': file_type,
                'ContentLength': file_size,
            },
            ExpiresIn=900,  # 15 minutes
            HttpMethod='PUT'
        )
        
        # Store metadata in DynamoDB
        timestamp = datetime.utcnow().isoformat()
        
        table.put_item(
            Item={
                'bookId': book_id,
                'version': 1,
                'title': title,
                'author': author,
                'description': description,
                'fileType': file_type,
                'fileSize': file_size,
                'status': 'PENDING',
                'uploaderId': uploader_id,
                'uploaderEmail': uploader_email,
                'uploadTimestamp': timestamp,
                's3Key': s3_key,
                'bucket': UPLOADS_BUCKET,
            }
        )
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
            },
            'body': json.dumps({
                'uploadUrl': presigned_url,
                'bookId': book_id,
                'expiresIn': 900,
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': 'Internal server error'
            })
        }
```

### 3.2 Add Lambda to CDK Stack

```typescript
// filepath: lib/online-library-cdk-stack.ts
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigatewayv2';
import * as integrations from 'aws-cdk-lib/aws-apigatewayv2-integrations';
import * as authorizers from 'aws-cdk-lib/aws-apigatewayv2-authorizers';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ...existing code...

    // Create Lambda function
    const createUploadUrlFunction = new lambda.Function(this, 'CreateUploadUrl', {
      runtime: lambda.Runtime.PYTHON_3_9,
      handler: 'index.lambda_handler',
      code: lambda.Code.fromAsset('lambda/createUploadUrl'),
      environment: {
        UPLOADS_BUCKET: uploadsBucket.bucketName,
        BOOKS_TABLE: booksTable.tableName,
      },
      timeout: cdk.Duration.seconds(30),
    });

    // Grant permissions
    uploadsBucket.grantPut(createUploadUrlFunction);
    booksTable.grantWriteData(createUploadUrlFunction);

    // Create HTTP API
    const httpApi = new apigateway.HttpApi(this, 'OnlineLibraryApi', {
      apiName: 'online-library-api',
      corsPreflight: {
        allowOrigins: ['*'], // Restrict in production
        allowMethods: [
          apigateway.CorsHttpMethod.GET,
          apigateway.CorsHttpMethod.POST,
          apigateway.CorsHttpMethod.PUT,
          apigateway.CorsHttpMethod.DELETE,
        ],
        allowHeaders: ['Content-Type', 'Authorization'],
      },
    });

    // Create JWT Authorizer
    const jwtAuthorizer = new authorizers.HttpJwtAuthorizer(
      'JwtAuthorizer',
      `https://cognito-idp.${this.region}.amazonaws.com/${userPool.userPoolId}`,
      {
        jwtAudience: [userPoolClient.userPoolClientId],
      }
    );

    // Add route
    httpApi.addRoutes({
      path: '/upload',
      methods: [apigateway.HttpMethod.POST],
      integration: new integrations.HttpLambdaIntegration(
        'CreateUploadUrlIntegration',
        createUploadUrlFunction
      ),
      authorizer: jwtAuthorizer,
    });

    // Output API URL
    new cdk.CfnOutput(this, 'ApiUrl', {
      value: httpApi.apiEndpoint,
      exportName: 'OnlineLibraryApiUrl',
    });
  }
}
```

### 3.3 Deploy Lambda

```bash
cdk deploy
```

---

## Step 4: Build Upload Form Frontend

### 4.1 Create Upload API Client

Create `src/lib/api/upload.ts`:

```typescript
// filepath: src/lib/api/upload.ts
import { fetchAuthSession } from 'aws-amplify/auth';

interface UploadUrlRequest {
  title: string;
  author: string;
  description?: string;
  fileType: string;
  fileSize: number;
}

interface UploadUrlResponse {
  uploadUrl: string;
  bookId: string;
  expiresIn: number;
}

export async function createUploadUrl(
  metadata: UploadUrlRequest
): Promise<UploadUrlResponse> {
  const session = await fetchAuthSession();
  const idToken = session.tokens?.idToken?.toString();

  const response = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/upload`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${idToken}`,
    },
    body: JSON.stringify(metadata),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'Failed to create upload URL');
  }

  return response.json();
}

export async function uploadFileToS3(
  file: File,
  presignedUrl: string,
  onProgress?: (progress: number) => void
): Promise<void> {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();

    xhr.upload.addEventListener('progress', (event) => {
      if (event.lengthComputable && onProgress) {
        const progress = (event.loaded / event.total) * 100;
        onProgress(progress);
      }
    });

    xhr.addEventListener('load', () => {
      if (xhr.status === 200) {
        resolve();
      } else {
        reject(new Error(`Upload failed with status ${xhr.status}`));
      }
    });

    xhr.addEventListener('error', () => {
      reject(new Error('Upload failed'));
    });

    xhr.open('PUT', presignedUrl);
    xhr.setRequestHeader('Content-Type', file.type);
    xhr.send(file);
  });
}
```

### 4.2 Create Upload Form Component

Create `src/components/upload/UploadForm.tsx`:

```typescript
// filepath: src/components/upload/UploadForm.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { createUploadUrl, uploadFileToS3 } from '@/lib/api/upload';

export function UploadForm() {
  const router = useRouter();
  const [file, setFile] = useState<File | null>(null);
  const [title, setTitle] = useState('');
  const [author, setAuthor] = useState('');
  const [description, setDescription] = useState('');
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(0);
  const [error, setError] = useState<string | null>(null);

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const selectedFile = e.target.files?.[0];
    if (!selectedFile) return;

    // Validate file type
    const allowedTypes = ['application/pdf', 'application/epub+zip'];
    if (!allowedTypes.includes(selectedFile.type)) {
      setError('Only PDF and ePub files are allowed');
      return;
    }

    // Validate file size (50MB)
    const maxSize = 50 * 1024 * 1024;
    if (selectedFile.size > maxSize) {
      setError('File size must be less than 50MB');
      return;
    }

    setFile(selectedFile);
    setError(null);
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!file || !title || !author) {
      setError('Please fill in all required fields');
      return;
    }

    setUploading(true);
    setError(null);
    setProgress(0);

    try {
      // Step 1: Get presigned URL
      const { uploadUrl, bookId } = await createUploadUrl({
        title,
        author,
        description,
        fileType: file.type,
        fileSize: file.size,
      });

      // Step 2: Upload file to S3
      await uploadFileToS3(file, uploadUrl, (progress) => {
        setProgress(progress);
      });

      // Success - redirect to my uploads
      router.push(`/my-uploads?success=${bookId}`);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Upload failed');
    } finally {
      setUploading(false);
    }
  };

  return (
    <div className="max-w-2xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">Upload Book</h1>

      <form onSubmit={handleSubmit} className="space-y-6">
        {error && (
          <div className="bg-red-50 border border-red-200 text-red-800 px-4 py-3 rounded">
            {error}
          </div>
        )}

        <div>
          <label className="block text-sm font-medium mb-2">
            File *
          </label>
          <input
            type="file"
            accept=".pdf,.epub"
            onChange={handleFileChange}
            disabled={uploading}
            className="block w-full text-sm text-gray-500
              file:mr-4 file:py-2 file:px-4
              file:rounded file:border-0
              file:text-sm file:font-semibold
              file:bg-blue-50 file:text-blue-700
              hover:file:bg-blue-100"
          />
          {file && (
            <p className="mt-2 text-sm text-gray-600">
              Selected: {file.name} ({(file.size / 1024 / 1024).toFixed(2)} MB)
            </p>
          )}
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Title *
          </label>
          <input
            type="text"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            disabled={uploading}
            className="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500"
            placeholder="Enter book title"
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Author *
          </label>
          <input
            type="text"
            value={author}
            onChange={(e) => setAuthor(e.target.value)}
            disabled={uploading}
            className="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500"
            placeholder="Enter author name"
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Description
          </label>
          <textarea
            value={description}
            onChange={(e) => setDescription(e.target.value)}
            disabled={uploading}
            rows={4}
            className="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500"
            placeholder="Enter book description"
          />
        </div>

        {uploading && (
          <div>
            <div className="mb-2 flex justify-between text-sm">
              <span>Uploading...</span>
              <span>{Math.round(progress)}%</span>
            </div>
            <div className="w-full bg-gray-200 rounded-full h-2">
              <div
                className="bg-blue-600 h-2 rounded-full transition-all duration-300"
                style={{ width: `${progress}%` }}
              />
            </div>
          </div>
        )}

        <button
          type="submit"
          disabled={uploading || !file || !title || !author}
          className="w-full bg-blue-600 text-white py-3 rounded-md font-medium
            hover:bg-blue-700 disabled:bg-gray-300 disabled:cursor-not-allowed"
        >
          {uploading ? 'Uploading...' : 'Upload Book'}
        </button>
      </form>
    </div>
  );
}
```

### 4.3 Create Upload Page

Create `src/app/upload/page.tsx`:

```typescript
// filepath: src/app/upload/page.tsx
import { ProtectedRoute } from '@/components/auth/ProtectedRoute';
import { UploadForm } from '@/components/upload/UploadForm';

export default function UploadPage() {
  return (
    <ProtectedRoute>
      <UploadForm />
    </ProtectedRoute>
  );
}
```

---

## Verification & Testing

### Checklist

- ✅ S3 buckets created with correct policies
- ✅ DynamoDB table created with GSIs
- ✅ Lambda generates presigned URLs successfully
- ✅ API Gateway route protected with JWT
- ✅ Upload form validates file type and size
- ✅ Progress tracking works during upload
- ✅ Metadata stored in DynamoDB with PENDING status
- ✅ Files uploaded to correct S3 location

### Testing Steps

1. **Test Lambda directly:**
```bash
aws lambda invoke \
  --function-name CreateUploadUrl \
  --payload '{"body":"{\"title\":\"Test\",\"author\":\"Author\",\"fileType\":\"application/pdf\",\"fileSize\":1024}"}' \
  response.json
```

2. **Test via API:**
```bash
curl -X POST https://your-api-url/upload \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Test Book","author":"Test Author","fileType":"application/pdf","fileSize":1024000}'
```

3. **Test Frontend:**
   - Navigate to `/upload`
   - Select a PDF file
   - Fill in title and author
   - Click upload and verify progress bar

### Common Issues & Solutions

**Issue:** Presigned URL expires during upload  
**Solution:** Increase expiration time or implement URL refresh logic

**Issue:** CORS errors when uploading to S3  
**Solution:** Verify S3 bucket CORS configuration matches your frontend domain

**Issue:** File upload fails silently  
**Solution:** Check browser console and Lambda CloudWatch logs for errors

---

## Clean Up (Optional)

To remove the resources created in this module:

```bash
cdk destroy
```

---

## Next Steps

Congratulations! You've completed Module 2. You now have:
- S3 buckets for uploads and approved books
- Lambda function generating secure presigned URLs
- DynamoDB table storing upload metadata
- Upload form with progress tracking

**Continue to:** [Module 3: Admin Approval System](../5.3-module3/)

---

## Additional Resources

- [S3 Presigned URLs Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [Lambda with API Gateway](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html)