---
title: "Workshop"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Authentication Workshop: AWS Amplify & Amazon Cognito

## Overview

Hands-on workshop introducing user authentication fundamentals with AWS Amplify and Amazon Cognito. Participants build a complete authentication system including user registration, email verification, login/logout, password management, multi-factor authentication (MFA), and social sign-in. Focus on implementing secure authentication quickly without managing infrastructure, following AWS best practices.

## Agenda

- Authentication foundations: why Amplify & Cognito, security best practices, use cases.  
- Getting started: install Amplify CLI, initialize React app, add authentication category.  
- User registration: build sign-up flow, email verification, custom attributes, password policies.  
- Login & sessions: implement authentication, handle JWT tokens, manage user sessions, logout properly.  
- Password management: forgot password flow, password reset, change password, security requirements.  
- Advanced features: enable MFA (TOTP), add social sign-in (Google/Facebook), user groups & roles.  
- Deployment: host with Amplify Hosting, configure CI/CD, custom domains, monitoring.

## What You'll Build

- React application with complete authentication UI (registration, login, profile).  
- Amazon Cognito User Pool with email verification and custom password policies.  
- Multi-factor authentication using TOTP (Time-based One-Time Password).  
- Social sign-in integration with Google and Facebook.  
- Protected routes with role-based access control (RBAC).

## Prerequisites

- AWS account with console access.  
- Node.js 18+ and npm installed.  
- Basic familiarity with JavaScript/React.  
- Amplify CLI installed (`npm install -g @aws-amplify/cli`).

## References

- Amplify CLI configuration steps  
- Cognito User Pool setup guide  
- Social identity provider integration  
- Cleanup checklist