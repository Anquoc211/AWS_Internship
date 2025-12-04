---
title: "Module 1: Foundation & Authentication"
date: "2025-01-15T09:00:00+07:00"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Overview

In this module, you'll set up the foundational infrastructure for the Online Library platform and implement user authentication using Amazon Cognito. You'll create a CDK project to manage infrastructure as code and build the authentication flow.

**Duration:** ~60 minutes

**Services Used:**
- AWS CDK
- Amazon Cognito
- AWS Amplify (for hosting)

---

## What You'll Learn

- Initialize an AWS CDK project with TypeScript
- Deploy Amazon Cognito User Pool with proper configuration
- Implement user registration and login flows
- Integrate Cognito with a Next.js frontend
- Configure email verification and password policies

---

## Architecture for This Module

```
User → Amplify Hosting → Cognito User Pool
                       ↓
                   JWT Tokens
```

---

## Step 1: Set Up AWS CDK Project

### 1.1 Install Prerequisites

```bash
# Install AWS CDK globally
npm install -g aws-cdk

# Verify installation
cdk --version

# Configure AWS CLI (if not already done)
aws configure
```

### 1.2 Create CDK Project

```bash
# Create project directory
mkdir online-library-cdk
cd online-library-cdk

# Initialize CDK project
cdk init app --language typescript

# Install dependencies
npm install
```

### 1.3 Project Structure

```
online-library-cdk/
├── bin/
│   └── online-library-cdk.ts    # App entry point
├── lib/
│   └── online-library-cdk-stack.ts  # Main stack
├── test/
├── cdk.json
├── package.json
└── tsconfig.json
```

---

## Step 2: Create Cognito User Pool

### 2.1 Install Cognito Construct Library

```bash
npm install @aws-cdk/aws-cognito
```

### 2.2 Define Cognito Stack

Edit `lib/online-library-cdk-stack.ts`:

```typescript
import * as cdk from 'aws-cdk-lib';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import { Construct } from 'constructs';

export class OnlineLibraryCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create Cognito User Pool
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
      removalPolicy: cdk.RemovalPolicy.DESTROY, // For dev only
    });

    // Create User Pool Client
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

    // Create Admin Group
    new cognito.CfnUserPoolGroup(this, 'AdminsGroup', {
      userPoolId: userPool.userPoolId,
      groupName: 'Admins',
      description: 'Administrator users with full access',
      precedence: 1,
    });

    // Output values for frontend
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

### 2.3 Deploy the Stack

```bash
# Bootstrap CDK (first time only)
cdk bootstrap

# Synthesize CloudFormation template
cdk synth

# Deploy the stack
cdk deploy

# Note the output values
```

---

## Step 3: Set Up Next.js Frontend

### 3.1 Create Next.js Project

```bash
# In a separate directory
npx create-next-app@latest online-library-frontend --typescript --tailwind --app

cd online-library-frontend
```

### 3.2 Install Dependencies

```bash
npm install aws-amplify @aws-amplify/ui-react
```

### 3.3 Configure Amplify

Create `src/lib/amplify-config.ts`:

```typescript
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

### 3.4 Create Environment File

Create `.env.local`:

```bash
NEXT_PUBLIC_AWS_REGION=ap-southeast-1
NEXT_PUBLIC_USER_POOL_ID=ap-southeast-1_xxxxxxxxx
NEXT_PUBLIC_USER_POOL_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## Step 4: Build Authentication Pages

### 4.1 Create Layout with Auth Provider

Edit `src/app/layout.tsx`:

```typescript
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
    <html lang="en">
      <body>
        <Authenticator.Provider>
          {children}
        </Authenticator.Provider>
      </body>
    </html>
  );
}
```

### 4.2 Create Login Page

Create `src/app/auth/login/page.tsx`:

```typescript
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
            Online Library
          </h1>
          <p className="mt-2 text-center text-sm text-gray-600">
            Sign in to access your library
          </p>
        </div>
        
        <Authenticator
          initialState="signIn"
          components={{
            SignIn: {
              Header() {
                return (
                  <div className="text-center mb-6">
                    <h3 className="text-2xl font-semibold">Welcome Back</h3>
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

### 4.3 Create Registration Page

Create `src/app/auth/register/page.tsx`:

```typescript
'use client';

import { Authenticator } from '@aws-amplify/ui-react';
import { useRouter } from 'next/navigation';

export default function RegisterPage() {
  const router = useRouter();

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full space-y-8">
        <div>
          <h1 className="text-4xl font-bold text-center text-gray-900">
            Create Account
          </h1>
          <p className="mt-2 text-center text-sm text-gray-600">
            Join our online library community
          </p>
        </div>
        
        <Authenticator
          initialState="signUp"
          components={{
            SignUp: {
              FormFields() {
                return (
                  <>
                    <Authenticator.SignUp.FormFields />
                    <div className="text-sm text-gray-600 mt-4">
                      By signing up, you agree to our Terms of Service
                    </div>
                  </>
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

## Step 5: Test Authentication Flow

### 5.1 Run Development Server

```bash
npm run dev
```

### 5.2 Test User Registration

1. Navigate to `http://localhost:3000/auth/register`
2. Fill in the registration form
3. Check email for verification code
4. Enter verification code
5. Verify user is created in Cognito console

### 5.3 Test User Login

1. Navigate to `http://localhost:3000/auth/login`
2. Enter credentials
3. Verify redirect to dashboard
4. Check JWT token in browser storage

---

## Step 6: Create Protected Routes

### 6.1 Create Auth Guard Component

Create `src/components/auth/ProtectedRoute.tsx`:

```typescript
'use client';

import { useAuthenticator } from '@aws-amplify/ui-react';
import { useRouter } from 'next/navigation';
import { useEffect } from 'react';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requireAdmin?: boolean;
}

export function ProtectedRoute({ 
  children, 
  requireAdmin = false 
}: ProtectedRouteProps) {
  const { user, authStatus } = useAuthenticator();
  const router = useRouter();

  useEffect(() => {
    if (authStatus === 'unauthenticated') {
      router.push('/auth/login');
    }

    if (requireAdmin && user) {
      const groups = user.signInUserSession?.accessToken.payload['cognito:groups'] || [];
      if (!groups.includes('Admins')) {
        router.push('/dashboard');
      }
    }
  }, [authStatus, user, requireAdmin, router]);

  if (authStatus === 'configuring' || authStatus === 'unauthenticated') {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-gray-900"></div>
      </div>
    );
  }

  return <>{children}</>;
}
```

### 6.2 Create Dashboard Page

Create `src/app/dashboard/page.tsx`:

```typescript
'use client';

import { ProtectedRoute } from '@/components/auth/ProtectedRoute';
import { useAuthenticator } from '@aws-amplify/ui-react';

export default function DashboardPage() {
  const { user, signOut } = useAuthenticator();

  return (
    <ProtectedRoute>
      <div className="min-h-screen bg-gray-100">
        <nav className="bg-white shadow-sm">
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div className="flex justify-between h-16">
              <div className="flex items-center">
                <h1 className="text-xl font-bold">Online Library</h1>
              </div>
              <div className="flex items-center space-x-4">
                <span className="text-sm text-gray-700">
                  {user?.signInUserSession?.idToken.payload.email}
                </span>
                <button
                  onClick={signOut}
                  className="px-4 py-2 text-sm font-medium text-white bg-red-600 rounded-md hover:bg-red-700"
                >
                  Sign Out
                </button>
              </div>
            </div>
          </div>
        </nav>

        <main className="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
          <div className="px-4 py-6 sm:px-0">
            <div className="border-4 border-dashed border-gray-200 rounded-lg h-96 flex items-center justify-center">
              <p className="text-2xl text-gray-400">
                Welcome to your library dashboard!
              </p>
            </div>
          </div>
        </main>
      </div>
    </ProtectedRoute>
  );
}
```

---

## Verification & Testing

### Checklist

- CDK stack deploys successfully
- Cognito User Pool created with correct configuration
- User can register with email verification
- User can login and receive JWT tokens
- Protected routes redirect unauthenticated users
- Admin group exists in Cognito
- Dashboard displays user information

### Common Issues & Solutions

**Issue:** CDK deploy fails with permission errors
**Solution:** Ensure your AWS CLI is configured with admin permissions

**Issue:** Cognito verification email not received
**Solution:** Check spam folder, verify email configuration in Cognito console

**Issue:** Authenticator UI not displaying
**Solution:** Ensure `@aws-amplify/ui-react/styles.css` is imported

---

## Clean Up (Optional)

```bash
# Destroy the stack
cdk destroy
```

---

## Next Steps

Congratulations! You've completed Module 1. You now have:
- A working CDK infrastructure setup
- Cognito User Pool with email verification
- Protected authentication flows
- A basic dashboard

**Continue to:** [Module 2: Upload Infrastructure](../5.2-module2/)

---

## Additional Resources

- [AWS CDK Documentation](https://docs.aws.amazon.com/cdk/)
- [Amazon Cognito Developer Guide](https://docs.aws.amazon.com/cognito/)
- [Amplify UI React Documentation](https://ui.docs.amplify.aws/)
