---
title: "Prerequisites"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Account & Permissions

- An AWS account with permissions to create Cognito User Pools, IAM roles, and Amplify resources.  
- No VPC or advanced networking permissions required for this workshop.

## Tools

- **Node.js 18+** and npm installed on your local machine.  
- **AWS Amplify CLI**: Install with `npm install -g @aws-amplify/cli`  
- **A code editor** (VS Code, Sublime, or any preferred editor).  
- **Web browser** for testing the authentication flow.

## Quick Setup

1. **Configure Amplify CLI:**
   ```bash
   amplify configure
   ```
   Follow the prompts to set up an IAM user with appropriate permissions.

2. **Select a region** close to you (e.g., `us-east-1` or `ap-southeast-1`).

3. **Verify installations:**
   ```bash
   node --version  # Should be 18+
   npm --version
   amplify --version
   ```