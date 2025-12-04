---
title: "Day 46 - Authentication & Project Setup"
weight: 1
chapter: false
pre: "<b> 1.10.1. </b>"
---

**Date:** 2025-11-10 (Monday)  
**Status:** "Done"  

---

# **Lecture Notes**

## Project Context Review

### Online Library Architecture

**Frontend Stack:**
- Next.js 14 (App Router)
- AWS Amplify (Hosting + CI/CD)
- Amplify UI Components
- TailwindCSS for styling
- React Query for state management

**AWS Services Integration:**
- Amazon Cognito (Authentication)
- API Gateway (HTTP API endpoints)
- S3 (File storage via presigned URLs)
- CloudFront (Content delivery)
- DynamoDB (Metadata storage)

### Development Environment Setup

**Prerequisites:**
- Node.js 18+ and npm/yarn
- AWS CLI configured
- Git and GitHub account
- AWS account with appropriate permissions

**Project Structure:**
```
online-library/
├── src/
│   ├── app/              # Next.js App Router pages
│   ├── components/       # Reusable UI components
│   ├── lib/              # Utility functions & AWS SDK
│   ├── hooks/            # Custom React hooks
│   └── types/            # TypeScript definitions
├── public/               # Static assets
├── amplify/              # Amplify configuration
└── .env.local            # Environment variables
```

## Authentication Flow

### Cognito Integration Strategy

**User Journey:**
1. Registration → Email verification → Login
2. JWT token storage in localStorage/cookies
3. Auto-refresh token before expiration
4. Role-based access (User vs Admin)

**Key Components:**
- Login page with email/password
- Registration form with validation
- Email verification flow
- Password reset functionality
- Protected routes HOC

### Amplify UI Authentication

**Benefits:**
- Pre-built, customizable components
- Built-in form validation
- Automatic token management
- Responsive design out-of-the-box

**Configuration:**
```javascript
// amplify-config.js
const amplifyConfig = {
  Auth: {
    region: 'ap-southeast-1',
    userPoolId: process.env.NEXT_PUBLIC_USER_POOL_ID,
    userPoolWebClientId: process.env.NEXT_PUBLIC_USER_POOL_CLIENT_ID,
  },
  API: {
    endpoints: [
      {
        name: 'OnlineLibraryAPI',
        endpoint: process.env.NEXT_PUBLIC_API_ENDPOINT,
      }
    ]
  }
}
```

## Key Insights

- Amplify UI components significantly reduce auth implementation time
- Token refresh should happen automatically in background
- Store user role/groups in JWT claims for frontend authorization
- Environment variables critical for security - never commit sensitive data
- CI/CD setup in Amplify requires proper build settings

---

# **Tasks Completed**

1. **Project Initialization**
   - Created Next.js 14 project with TypeScript and TailwindCSS
   - Installed AWS Amplify and required dependencies
   - Set up project folder structure

2. **Amplify Configuration**
   - Configured Amplify with Cognito User Pool credentials
   - Set up API Gateway endpoint configuration
   - Created environment variable structure

3. **Authentication Pages**
   - Built login page with Amplify Authenticator
   - Created registration page with email verification
   - Implemented protected route HOC for authenticated pages

4. **User Management**
   - Created custom `useUser` hook for user context
   - Implemented JWT token handling
   - Set up admin role detection from Cognito groups

5. **CI/CD Setup**
   - Connected GitHub repository to Amplify Hosting
   - Configured build settings for Next.js deployment
   - Set up environment variables in Amplify Console

---

# **Challenges & Solutions**

**Challenge:** Amplify v6 API changes from previous versions  
**Solution:** Updated to use new Amplify.configure() syntax and Authenticator.Provider

**Challenge:** Token refresh timing causing 401 errors  
**Solution:** Implemented automatic token refresh 5 minutes before expiration

**Challenge:** Admin role not reflecting immediately after login  
**Solution:** Added token re-fetch mechanism after successful authentication
