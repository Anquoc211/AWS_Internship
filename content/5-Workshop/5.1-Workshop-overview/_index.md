---
title: "Workshop Overview"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Goals

Introduce the authentication workshop: set up AWS Amplify CLI, create a React application with complete authentication flow using Amazon Cognito, implement user registration, login, password management, MFA, and social sign-in. Emphasize rapid implementation of secure authentication without managing infrastructure.

## Architecture at a glance

- Amazon Cognito User Pool manages user directory and authentication.  
- AWS Amplify provides CLI tooling and frontend libraries for seamless integration.  
- React application uses Amplify UI components for authentication UI.  
- JWT tokens handle session management and API authorization.

## Expected outcomes

- React application with user registration and email verification.  
- Login/logout functionality with session persistence.  
- Password reset and change password features.  
- Multi-factor authentication (MFA) with TOTP.  
- Social sign-in integration (Google/Facebook).  
- Protected routes with role-based access control.

## Timing & structure

- Module 1: Amplify setup and initial configuration.  
- Module 2: User registration and email verification flow.  
- Module 3: Login, session management, and logout.  
- Module 4: Password management (forgot/reset/change).  
- Module 5: Advanced features (MFA and social login).  
- Module 6: Production deployment and cleanup.