---
title: "Advanced Features & Cleanup"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Multi-Factor Authentication (MFA)

Enable TOTP-based MFA:

```bash
amplify update auth
# Choose: Walkthrough all the auth configurations
# MFA: Optional
# MFA type: TOTP (Time-based One-Time Password)
amplify push
```

Implement MFA in your app:

```javascript
import { Auth } from 'aws-amplify';

// Setup MFA for user
async function setupMFA() {
  try {
    const user = await Auth.currentAuthenticatedUser();
    const code = await Auth.setupTOTP(user);
    
    // Display QR code to user using code
    const qrCodeUrl = `otpauth://totp/AWSCognito:${user.username}?secret=${code}&issuer=AuthWorkshop`;
    console.log('QR Code URL:', qrCodeUrl);
    
    // Verify TOTP token
    const token = prompt('Enter TOTP code from authenticator app:');
    await Auth.verifyTotpToken(user, token);
    await Auth.setPreferredMFA(user, 'TOTP');
    
    alert('MFA enabled successfully!');
  } catch (error) {
    console.error('Error setting up MFA:', error);
  }
}
```

## Social Sign-In

Add Google/Facebook authentication:

```bash
amplify update auth
# Choose: Walkthrough all the auth configurations
# Social providers: Google, Facebook
# Follow prompts to configure OAuth
amplify push
```

Add social login buttons:

```javascript
import { Auth } from 'aws-amplify';

function SocialLogin() {
  const handleGoogleLogin = () => {
    Auth.federatedSignIn({ provider: 'Google' });
  };

  const handleFacebookLogin = () => {
    Auth.federatedSignIn({ provider: 'Facebook' });
  };

  return (
    <div>
      <button onClick={handleGoogleLogin}>Sign in with Google</button>
      <button onClick={handleFacebookLogin}>Sign in with Facebook</button>
    </div>
  );
}
```

## User Groups & Roles

Create user groups in Cognito console:

1. Go to Cognito User Pools
2. Select your user pool
3. Navigate to "Users and groups"  "Groups"
4. Create groups: `Admin`, `User`

Assign users to groups via code:

```javascript
// Server-side code (Lambda)
const AWS = require('aws-sdk');
const cognito = new AWS.CognitoIdentityServiceProvider();

async function addUserToGroup(username, groupName, userPoolId) {
  const params = {
    GroupName: groupName,
    UserPoolId: userPoolId,
    Username: username
  };
  
  await cognito.adminAddUserToGroup(params).promise();
}
```

## Deploy to Amplify Hosting

Deploy your React app:

```bash
amplify add hosting
# Choose: Amplify Console
# Type: Manual deployment

amplify publish
```

Your app will be deployed to a live URL.

## Cleanup Resources

Remove all AWS resources:

```bash
# Delete Amplify backend
amplify delete

# Confirm deletion
# This will remove:
# - Cognito User Pool
# - All authentication resources
# - CloudFormation stacks
```

Remove local files:

```bash
# Delete Amplify project files
Remove-Item -Recurse -Force amplify
Remove-Item -Force aws-exports.js
```

## Cost Optimization

- **Cognito User Pools**: Free tier covers 50,000 MAUs (Monthly Active Users)
- **Amplify Hosting**: Build minutes and data transfer have free tier limits
- **Best Practice**: Always delete unused resources

## Congratulations! 

You've completed the Authentication Workshop covering:
-  User registration and email verification
-  Login and session management
-  Password reset flows
-  MFA with TOTP
-  Social authentication
- User groups and roles
- Production deployment