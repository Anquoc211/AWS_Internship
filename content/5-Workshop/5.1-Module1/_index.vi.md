---
title: "Module 1: Nền tảng & Xác thực"
date: "2025-01-15T09:00:00+07:00"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Tổng quan

Trong module này, bạn sẽ thiết lập hạ tầng nền tảng cho nền tảng Thư viện Online và triển khai xác thực người dùng sử dụng Amazon Cognito. Bạn sẽ tạo một dự án CDK để quản lý hạ tầng dưới dạng mã và xây dựng luồng xác thực.

**Thời lượng:** ~60 phút

**Dịch vụ sử dụng:**
- AWS CDK
- Amazon Cognito
- AWS Amplify (cho hosting)

---

## Những gì bạn sẽ học

- Khởi tạo dự án AWS CDK với TypeScript
- Triển khai Amazon Cognito User Pool với cấu hình phù hợp
- Triển khai luồng đăng ký và đăng nhập người dùng
- Tích hợp Cognito với frontend Next.js
- Cấu hình xác thực email và chính sách mật khẩu

---

## Kiến trúc cho Module này

```
User → Amplify Hosting → Cognito User Pool
                       ↓
                   JWT Tokens
```

---

## Bước 1: Thiết lập dự án AWS CDK

### 1.1 Cài đặt yêu cầu

```bash
# Cài đặt AWS CDK toàn cục
npm install -g aws-cdk

# Xác minh cài đặt
cdk --version

# Cấu hình AWS CLI (nếu chưa làm)
aws configure
```

### 1.2 Tạo dự án CDK

```bash
# Tạo thư mục dự án
mkdir online-library-cdk
cd online-library-cdk

# Khởi tạo dự án CDK
cdk init app --language typescript

# Cài đặt dependencies
npm install
```

### 1.3 Cấu trúc dự án

```
online-library-cdk/
├── bin/
│   └── online-library-cdk.ts    # Điểm entry của app
├── lib/
│   └── online-library-cdk-stack.ts  # Stack chính
├── test/
├── cdk.json
├── package.json
└── tsconfig.json
```

---

## Bước 2: Tạo Cognito User Pool

### 2.1 Cài đặt Cognito Construct Library

```bash
npm install @aws-cdk/aws-cognito
```

### 2.2 Định nghĩa Cognito Stack

Chỉnh sửa `lib/online-library-cdk-stack.ts`:

```typescript
// filepath: lib/online-library-cdk-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import { Construct } from 'constructs';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Tạo Cognito User Pool
    const userPool = new cognito.UserPool(this, 'OnlineLibraryUserPool', {
      userPoolName: 'online-library-users',
      selfSignUpEnabled: true,
      signInAliases: {
        email: true,
      },
      autoVerify: {
        email: true,
      },
      standardAttributes: {
        email: {
          required: true,
          mutable: true,
        },
        fullname: {
          required: true,
          mutable: true,
        },
      },
      passwordPolicy: {
        minLength: 8,
        requireLowercase: true,
        requireUppercase: true,
        requireDigits: true,
        requireSymbols: false,
      },
      accountRecovery: cognito.AccountRecovery.EMAIL_ONLY,
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Chỉ cho dev
    });

    // Tạo User Pool Client
    const userPoolClient = userPool.addClient('OnlineLibraryWebClient', {
      authFlows: {
        userPassword: true,
        userSrp: true,
      },
      oAuth: {
        flows: {
          authorizationCodeGrant: true,
        },
        scopes: [
          cognito.OAuthScope.EMAIL,
          cognito.OAuthScope.OPENID,
          cognito.OAuthScope.PROFILE,
        ],
      },
    });

    // Tạo Admin Group
    new cognito.CfnUserPoolGroup(this, 'AdminsGroup', {
      userPoolId: userPool.userPoolId,
      groupName: 'Admins',
      description: 'Người dùng quản trị với quyền truy cập đầy đủ',
      precedence: 1,
    });

    // Output values cho frontend
    new cdk.CfnOutput(this, 'UserPoolId', {
      value: userPool.userPoolId,
      description: 'Cognito User Pool ID',
      exportName: 'OnlineLibraryUserPoolId',
    });

    new cdk.CfnOutput(this, 'UserPoolClientId', {
      value: userPoolClient.userPoolClientId,
      description: 'Cognito User Pool Client ID',
      exportName: 'OnlineLibraryUserPoolClientId',
    });

    new cdk.CfnOutput(this, 'UserPoolArn', {
      value: userPool.userPoolArn,
      description: 'Cognito User Pool ARN',
      exportName: 'OnlineLibraryUserPoolArn',
    });
  }
}
```

### 2.3 Triển khai Stack

```bash
# Bootstrap CDK (chỉ lần đầu)
cdk bootstrap

# Tổng hợp CloudFormation template
cdk synth

# Triển khai stack
cdk deploy

# Ghi chú output values
```

---

## Bước 3: Thiết lập Next.js Frontend

### 3.1 Tạo dự án Next.js

```bash
# Trong thư mục riêng biệt
npx create-next-app@latest online-library-frontend --typescript --tailwind --app

cd online-library-frontend
```

### 3.2 Cài đặt Dependencies

```bash
npm install aws-amplify @aws-amplify/ui-react
```

### 3.3 Cấu hình Amplify

Tạo `src/lib/amplify-config.ts`:

```typescript
// filepath: src/lib/amplify-config.ts
import { Amplify } from 'aws-amplify';

export const configureAmplify = () => {
  Amplify.configure({
    Auth: {
      Cognito: {
        region: process.env.NEXT_PUBLIC_AWS_REGION!,
        userPoolId: process.env.NEXT_PUBLIC_USER_POOL_ID!,
        userPoolClientId: process.env.NEXT_PUBLIC_USER_POOL_CLIENT_ID!,
        loginWith: {
          email: true,
        },
        signUpVerificationMethod: 'code',
        userAttributes: {
          email: {
            required: true,
          },
          name: {
            required: true,
          },
        },
      },
    },
  });
};
```

### 3.4 Tạo File Environment

Tạo `.env.local`:

```bash
NEXT_PUBLIC_AWS_REGION=ap-southeast-1
NEXT_PUBLIC_USER_POOL_ID=ap-southeast-1_xxxxxxxxx
NEXT_PUBLIC_USER_POOL_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## Bước 4: Xây dựng trang xác thực

### 4.1 Tạo Layout với Auth Provider

Chỉnh sửa `src/app/layout.tsx`:

```typescript
// filepath: src/app/layout.tsx
'use client';

import { useEffect } from 'react';
import { configureAmplify } from '@/lib/amplify-config';
import { Authenticator } from '@aws-amplify/ui-react';
import '@aws-amplify/ui-react/styles.css';
import './globals.css';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  useEffect(() => {
    configureAmplify();
  }, []);

  return (
    <html lang="vi">
      <body>
        <Authenticator.Provider>
          {children}
        </Authenticator.Provider>
      </body>
    </html>
  );
}
```

### 4.2 Tạo trang đăng nhập

Tạo `src/app/auth/login/page.tsx`:

```typescript
// filepath: src/app/auth/login/page.tsx
'use client';

import { Authenticator, useAuthenticator } from '@aws-amplify/ui-react';
import { useRouter } from 'next/navigation';
import { useEffect } from 'react';

export default function LoginPage() {
  const { user } = useAuthenticator();
  const router = useRouter();

  useEffect(() => {
    if (user) {
      router.push('/dashboard');
    }
  }, [user, router]);

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full space-y-8">
        <div>
          <h1 className="text-4xl font-bold text-center text-gray-900">
            Thư viện Online
          </h1>
          <p className="mt-2 text-center text-sm text-gray-600">
            Đăng nhập để truy cập thư viện của bạn
          </p>
        </div>
        
        <Authenticator
          initialState="signIn"
          components={{
            SignIn: {
              Header() {
                return (
                  <div className="text-center mb-6">
                    <h3 className="text-2xl font-semibold">Chào mừng trở lại</h3>
                  </div>
                );
              },
            },
          }}
        />
      </div>
    </div>
  );
}
```

---

## Xác minh & Kiểm thử

### Checklist

- CDK stack triển khai thành công
- Cognito User Pool được tạo với cấu hình đúng
- Người dùng có thể đăng ký với xác thực email
- Người dùng có thể đăng nhập và nhận JWT tokens
- Protected routes chuyển hướng người dùng chưa xác thực
- Admin group tồn tại trong Cognito
- Dashboard hiển thị thông tin người dùng

### Các vấn đề thường gặp & Giải pháp

**Vấn đề:** CDK deploy thất bại với lỗi permission
**Giải pháp:** Đảm bảo AWS CLI được cấu hình với quyền admin

**Vấn đề:** Không nhận được email xác thực Cognito
**Giải pháp:** Kiểm tra thư mục spam, xác minh cấu hình email trong Cognito console

**Vấn đề:** Authenticator UI không hiển thị
**Giải pháp:** Đảm bảo `@aws-amplify/ui-react/styles.css` đã được import

---

## Dọn dẹp (Tùy chọn)

```bash
# Hủy stack
cdk destroy
```

---

## Bước tiếp theo

Chúc mừng! Bạn đã hoàn thành Module 1. Bây giờ bạn có:
- Một thiết lập hạ tầng CDK hoạt động
- Cognito User Pool với xác thực email
- Các luồng xác thực được bảo vệ
- Một dashboard cơ bản

**Tiếp tục đến:** [Module 2: Hạ tầng Upload](../5.2-module2/)

---

## Tài nguyên bổ sung

- [Tài liệu AWS CDK](https://docs.aws.amazon.com/cdk/)
- [Hướng dẫn phát triển Amazon Cognito](https://docs.aws.amazon.com/cognito/)
- [Tài liệu Amplify UI React](https://ui.docs.amplify.aws/)
