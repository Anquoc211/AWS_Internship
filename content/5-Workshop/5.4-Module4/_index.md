---
title: "Module 4: Content Delivery with CloudFront"
date: "2025-01-15T12:00:00+07:00"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Overview

In this module, you'll implement secure content delivery using Amazon CloudFront with signed URLs. You'll configure Origin Access Control (OAC) to protect S3 content, generate time-limited signed URLs for authorized access, and build a PDF/ePub reader interface.

**Duration:** ~90 minutes

**Services Used:**
- Amazon CloudFront (CDN with signed URLs)
- Amazon S3 (protected origin)
- AWS Lambda (signed URL generation)
- CloudFront Key Pairs (for signing)

---

## What You'll Learn

- Configure CloudFront distribution with OAC
- Generate CloudFront signed URLs from Lambda
- Implement URL expiration and security policies
- Build PDF and ePub reader components
- Handle content streaming and caching

---

## Architecture for This Module

```
User → Request Book → Lambda (getReadUrl)
                        ↓
                  Check DynamoDB (APPROVED?)
                        ↓
                  Generate CloudFront Signed URL (15 min TTL)
                        ↓
User ← Signed URL ← Response
     ↓
Access Content via CloudFront (with OAC to S3)
```

---

## Step 1: Create CloudFront Distribution with OAC

### 1.1 Add CloudFront to CDK Stack

```typescript
// filepath: lib/online-library-cdk-stack.ts
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront';
import * as origins from 'aws-cdk-lib/aws-cloudfront-origins';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ...existing code...

    // Create Origin Access Control (OAC)
    const oac = new cloudfront.CfnOriginAccessControl(this, 'BooksOAC', {
      originAccessControlConfig: {
        name: 'online-library-oac',
        originAccessControlOriginType: 's3',
        signingBehavior: 'always',
        signingProtocol: 'sigv4',
        description: 'OAC for Online Library books bucket',
      },
    });

    // Create CloudFront Distribution
    const distribution = new cloudfront.Distribution(this, 'BooksDistribution', {
      defaultBehavior: {
        origin: new origins.S3Origin(booksBucket),
        viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
        cachePolicy: cloudfront.CachePolicy.CACHING_OPTIMIZED,
        originRequestPolicy: cloudfront.OriginRequestPolicy.CORS_S3_ORIGIN,
        trustedKeyGroups: [/* Will add signing key group */],
      },
      priceClass: cloudfront.PriceClass.PRICE_CLASS_200,
      enableLogging: true,
      logBucket: new s3.Bucket(this, 'CloudFrontLogsBucket', {
        bucketName: `online-library-cf-logs-${this.account}`,
        removalPolicy: cdk.RemovalPolicy.DESTROY,
        autoDeleteObjects: true,
      }),
    });

    // Update S3 bucket policy to allow only CloudFront OAC
    booksBucket.addToResourcePolicy(
      new iam.PolicyStatement({
        actions: ['s3:GetObject'],
        resources: [`${booksBucket.bucketArn}/public/books/*`],
        principals: [new iam.ServicePrincipal('cloudfront.amazonaws.com')],
        conditions: {
          'StringEquals': {
            'AWS:SourceArn': `arn:aws:cloudfront::${this.account}:distribution/${distribution.distributionId}`,
          },
        },
      })
    );

    // Output distribution domain
    new cdk.CfnOutput(this, 'DistributionDomain', {
      value: distribution.distributionDomainName,
      exportName: 'OnlineLibraryDistributionDomain',
    });

    new cdk.CfnOutput(this, 'DistributionId', {
      value: distribution.distributionId,
      exportName: 'OnlineLibraryDistributionId',
    });
  }
}
```

### 1.2 Create CloudFront Key Pair

Via AWS Console (one-time setup):

```bash
# 1. Navigate to CloudFront Console
# 2. Go to Key Management → Public keys
# 3. Create a new key pair
# 4. Download private key (keep secure!)
# 5. Note the Key ID

# Store private key in Secrets Manager
aws secretsmanager create-secret \
  --name online-library/cloudfront-private-key \
  --secret-string file://private_key.pem \
  --region ap-southeast-1
```

---

## Step 2: Create Lambda for Signed URL Generation

### 2.1 Create getReadUrl Lambda

Create `lambda/getReadUrl/index.py`:

```python
# filepath: lambda/getReadUrl/index.py
import json
import boto3
from datetime import datetime, timedelta
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.backends import default_backend
import base64
import os

dynamodb = boto3.resource('dynamodb')
secrets_manager = boto3.client('secretsmanager')

BOOKS_TABLE = os.environ['BOOKS_TABLE']
DISTRIBUTION_DOMAIN = os.environ['DISTRIBUTION_DOMAIN']
KEY_PAIR_ID = os.environ['KEY_PAIR_ID']
PRIVATE_KEY_SECRET = os.environ['PRIVATE_KEY_SECRET']

table = dynamodb.Table(BOOKS_TABLE)

# Cache private key
_private_key = None

def get_private_key():
    """Retrieve and cache CloudFront private key"""
    global _private_key
    
    if _private_key is None:
        response = secrets_manager.get_secret_value(SecretId=PRIVATE_KEY_SECRET)
        key_pem = response['SecretString']
        _private_key = serialization.load_pem_private_key(
            key_pem.encode(),
            password=None,
            backend=default_backend()
        )
    
    return _private_key

def create_signed_url(url, expires_in_minutes=15):
    """Generate CloudFront signed URL"""
    
    # Calculate expiration time
    expire_time = datetime.utcnow() + timedelta(minutes=expires_in_minutes)
    expire_timestamp = int(expire_time.timestamp())
    
    # Create policy
    policy = {
        "Statement": [
            {
                "Resource": url,
                "Condition": {
                    "DateLessThan": {
                        "AWS:EpochTime": expire_timestamp
                    }
                }
            }
        ]
    }
    
    policy_json = json.dumps(policy, separators=(',', ':'))
    
    # Sign policy
    private_key = get_private_key()
    signature = private_key.sign(
        policy_json.encode(),
        padding.PKCS1v15(),
        hashes.SHA1()
    )
    
    # Encode for URL
    policy_b64 = base64.b64encode(policy_json.encode()).decode()
    signature_b64 = base64.b64encode(signature).decode()
    
    # URL-safe encoding
    policy_b64 = policy_b64.replace('+', '-').replace('=', '_').replace('/', '~')
    signature_b64 = signature_b64.replace('+', '-').replace('=', '_').replace('/', '~')
    
    # Build signed URL
    signed_url = f"{url}?Policy={policy_b64}&Signature={signature_b64}&Key-Pair-Id={KEY_PAIR_ID}"
    
    return signed_url, expire_timestamp

def lambda_handler(event, context):
    """
    Generate signed URL for approved book
    """
    try:
        # Get book ID from path
        book_id = event['pathParameters']['bookId']
        
        # Get user info from JWT
        claims = event['requestContext']['authorizer']['jwt']['claims']
        user_id = claims['sub']
        
        # Get book metadata
        response = table.get_item(
            Key={
                'bookId': book_id,
                'version': 1
            }
        )
        
        if 'Item' not in response:
            return {
                'statusCode': 404,
                'body': json.dumps({'error': 'Book not found'})
            }
        
        book = response['Item']
        
        # Check if book is approved
        if book['status'] != 'APPROVED':
            return {
                'statusCode': 403,
                'body': json.dumps({
                    'error': f'Book not available. Status: {book["status"]}'
                })
            }
        
        # Build CloudFront URL
        s3_key = book['publicS3Key']  # e.g., public/books/book-id.pdf
        cloudfront_url = f"https://{DISTRIBUTION_DOMAIN}/{s3_key}"
        
        # Generate signed URL
        signed_url, expires_at = create_signed_url(cloudfront_url)
        
        # Optional: Log access
        print(f"User {user_id} accessed book {book_id}")
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
            },
            'body': json.dumps({
                'url': signed_url,
                'expiresAt': expires_at,
                'title': book['title'],
                'author': book['author'],
                'fileType': book['fileType'],
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'Internal server error'})
        }
```

### 2.2 Add Lambda to CDK

```typescript
// filepath: lib/online-library-cdk-stack.ts
// ...existing code...

// Create Lambda layer for cryptography
const cryptographyLayer = new lambda.LayerVersion(this, 'CryptographyLayer', {
  code: lambda.Code.fromAsset('lambda/layers/cryptography'),
  compatibleRuntimes: [lambda.Runtime.PYTHON_3_9],
  description: 'Cryptography library for CloudFront signing',
});

// Create getReadUrl Lambda
const getReadUrlFunction = new lambda.Function(this, 'GetReadUrl', {
  runtime: lambda.Runtime.PYTHON_3_9,
  handler: 'index.lambda_handler',
  code: lambda.Code.fromAsset('lambda/getReadUrl'),
  layers: [cryptographyLayer],
  environment: {
    BOOKS_TABLE: booksTable.tableName,
    DISTRIBUTION_DOMAIN: distribution.distributionDomainName,
    KEY_PAIR_ID: process.env.CLOUDFRONT_KEY_PAIR_ID!,
    PRIVATE_KEY_SECRET: 'online-library/cloudfront-private-key',
  },
  timeout: cdk.Duration.seconds(30),
});

// Grant permissions
booksTable.grantReadData(getReadUrlFunction);
getReadUrlFunction.addToRolePolicy(
  new iam.PolicyStatement({
    actions: ['secretsmanager:GetSecretValue'],
    resources: [
      `arn:aws:secretsmanager:${this.region}:${this.account}:secret:online-library/cloudfront-private-key-*`,
    ],
  })
);

// Add API route
httpApi.addRoutes({
  path: '/books/{bookId}/read',
  methods: [apigateway.HttpMethod.GET],
  integration: new integrations.HttpLambdaIntegration(
    'GetReadUrlIntegration',
    getReadUrlFunction
  ),
  authorizer: jwtAuthorizer,
});
```

---

## Step 3: Build Reader Interface

### 3.1 Create Reader API Client

Create `src/lib/api/reader.ts`:

```typescript
// filepath: src/lib/api/reader.ts
import { fetchAuthSession } from 'aws-amplify/auth';

interface ReadUrlResponse {
  url: string;
  expiresAt: number;
  title: string;
  author: string;
  fileType: string;
}

export async function getReadUrl(bookId: string): Promise<ReadUrlResponse> {
  const session = await fetchAuthSession();
  const idToken = session.tokens?.idToken?.toString();

  const response = await fetch(
    `${process.env.NEXT_PUBLIC_API_URL}/books/${bookId}/read`,
    {
      headers: {
        'Authorization': `Bearer ${idToken}`,
      },
    }
  );

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'Failed to get read URL');
  }

  return response.json();
}
```

### 3.2 Create PDF Reader Component

Create `src/components/reader/PdfReader.tsx`:

```typescript
// filepath: src/components/reader/PdfReader.tsx
'use client';

import { useState, useEffect } from 'react';
import { Document, Page, pdfjs } from 'react-pdf';
import 'react-pdf/dist/Page/AnnotationLayer.css';
import 'react-pdf/dist/Page/TextLayer.css';

// Configure PDF.js worker
pdfjs.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/${pdfjs.version}/pdf.worker.min.js`;

interface PdfReaderProps {
  url: string;
  title: string;
}

export function PdfReader({ url, title }: PdfReaderProps) {
  const [numPages, setNumPages] = useState<number>(0);
  const [pageNumber, setPageNumber] = useState<number>(1);
  const [scale, setScale] = useState<number>(1.0);
  const [loading, setLoading] = useState<boolean>(true);

  function onDocumentLoadSuccess({ numPages }: { numPages: number }) {
    setNumPages(numPages);
    setLoading(false);
  }

  const goToPrevPage = () => {
    setPageNumber((prev) => Math.max(prev - 1, 1));
  };

  const goToNextPage = () => {
    setPageNumber((prev) => Math.min(prev + 1, numPages));
  };

  const zoomIn = () => {
    setScale((prev) => Math.min(prev + 0.2, 2.0));
  };

  const zoomOut = () => {
    setScale((prev) => Math.max(prev - 0.2, 0.5));
  };

  return (
    <div className="flex flex-col h-screen bg-gray-900">
      {/* Toolbar */}
      <div className="bg-gray-800 text-white p-4 flex justify-between items-center">
        <div>
          <h1 className="text-xl font-bold">{title}</h1>
          <p className="text-sm text-gray-400">
            Page {pageNumber} of {numPages}
          </p>
        </div>

        <div className="flex gap-2">
          {/* Zoom controls */}
          <button
            onClick={zoomOut}
            className="px-3 py-2 bg-gray-700 rounded hover:bg-gray-600"
          >
            -
          </button>
          <span className="px-3 py-2">{Math.round(scale * 100)}%</span>
          <button
            onClick={zoomIn}
            className="px-3 py-2 bg-gray-700 rounded hover:bg-gray-600"
          >
            +
          </button>

          {/* Navigation */}
          <button
            onClick={goToPrevPage}
            disabled={pageNumber <= 1}
            className="px-4 py-2 bg-blue-600 rounded hover:bg-blue-700 disabled:bg-gray-600"
          >
            Previous
          </button>
          <button
            onClick={goToNextPage}
            disabled={pageNumber >= numPages}
            className="px-4 py-2 bg-blue-600 rounded hover:bg-blue-700 disabled:bg-gray-600"
          >
            Next
          </button>
        </div>
      </div>

      {/* PDF Viewer */}
      <div className="flex-1 overflow-auto bg-gray-900 flex justify-center p-4">
        {loading && (
          <div className="text-white">Loading PDF...</div>
        )}
        
        <Document
          file={url}
          onLoadSuccess={onDocumentLoadSuccess}
          loading={<div className="text-white">Loading document...</div>}
          error={<div className="text-red-500">Failed to load PDF</div>}
        >
          <Page
            pageNumber={pageNumber}
            scale={scale}
            renderTextLayer={true}
            renderAnnotationLayer={true}
          />
        </Document>
      </div>
    </div>
  );
}
```

### 3.3 Create Reader Page

Create `src/app/read/[bookId]/page.tsx`:

```typescript
// filepath: src/app/read/[bookId]/page.tsx
'use client';

import { useEffect, useState } from 'react';
import { useParams } from 'next/navigation';
import { ProtectedRoute } from '@/components/auth/ProtectedRoute';
import { PdfReader } from '@/components/reader/PdfReader';
import { getReadUrl } from '@/lib/api/reader';

export default function ReadPage() {
  const params = useParams();
  const bookId = params.bookId as string;
  
  const [readUrl, setReadUrl] = useState<string | null>(null);
  const [title, setTitle] = useState<string>('');
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchReadUrl() {
      try {
        const data = await getReadUrl(bookId);
        setReadUrl(data.url);
        setTitle(data.title);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Failed to load book');
      } finally {
        setLoading(false);
      }
    }

    fetchReadUrl();
  }, [bookId]);

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-gray-900"></div>
      </div>
    );
  }

  if (error) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="text-center">
          <h1 className="text-2xl font-bold text-red-600 mb-4">Error</h1>
          <p>{error}</p>
        </div>
      </div>
    );
  }

  return (
    <ProtectedRoute>
      {readUrl && <PdfReader url={readUrl} title={title} />}
    </ProtectedRoute>
  );
}
```

---

## Verification & Testing

### Checklist

- ✅ CloudFront distribution created with OAC
- ✅ S3 bucket policy allows only CloudFront access
- ✅ Lambda generates valid signed URLs
- ✅ Signed URLs expire after TTL
- ✅ PDF renders correctly in browser
- ✅ Navigation controls work (prev/next page)
- ✅ Zoom functionality works
- ✅ Unauthorized users cannot access content

### Common Issues & Solutions

**Issue:** PDF fails to load with CORS error  
**Solution:** Verify CloudFront CORS settings and S3 bucket CORS configuration

**Issue:** Signed URL expires too quickly  
**Solution:** Adjust TTL in Lambda or implement URL refresh mechanism

**Issue:** PDF.js worker fails to load  
**Solution:** Ensure PDF.js worker URL is accessible and version matches

---

## Clean Up

```bash
cdk destroy
```

---

## Next Steps

Congratulations! You've completed Module 4. You now have:
- Secure content delivery via CloudFront
- Signed URL generation with expiration
- PDF reader with navigation and zoom
- Protected S3 content with OAC

**Continue to:** [Module 5: Search & Discovery](../5.5-module5/)

---

## Additional Resources

- [CloudFront Signed URLs Documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-urls.html)
- [CloudFront OAC Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- [React PDF Documentation](https://github.com/wojtekmaj/react-pdf)