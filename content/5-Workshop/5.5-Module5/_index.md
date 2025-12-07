---
title: "Password Management"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Forgot Password Flow

Create `src/components/ForgotPassword.js`:

```javascript
import { Auth } from 'aws-amplify';
import { useState } from 'react';

function ForgotPassword() {
  const [email, setEmail] = useState('');
  const [code, setCode] = useState('');
  const [newPassword, setNewPassword] = useState('');
  const [step, setStep] = useState('request');

  const handleRequestReset = async (e) => {
    e.preventDefault();
    try {
      await Auth.forgotPassword(email);
      setStep('reset');
      alert('Verification code sent to your email');
    } catch (error) {
      console.error('Error:', error);
      alert(error.message);
    }
  };

  const handleResetPassword = async (e) => {
    e.preventDefault();
    try {
      await Auth.forgotPasswordSubmit(email, code, newPassword);
      alert('Password reset successfully!');
      setStep('complete');
    } catch (error) {
      console.error('Error:', error);
      alert(error.message);
    }
  };

  if (step === 'request') {
    return (
      <form onSubmit={handleRequestReset}>
        <h2>Forgot Password</h2>
        <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
        <button type="submit">Send Code</button>
      </form>
    );
  }

  if (step === 'reset') {
    return (
      <form onSubmit={handleResetPassword}>
        <h2>Reset Password</h2>
        <input type="text" value={code} onChange={(e) => setCode(e.target.value)} placeholder="Verification Code" required />
        <input type="password" value={newPassword} onChange={(e) => setNewPassword(e.target.value)} placeholder="New Password" required />
        <button type="submit">Reset Password</button>
      </form>
    );
  }

  return <div>Password reset complete! <a href="/login">Go to Login</a></div>;
}

export default ForgotPassword;
```

## Change Password

Create `src/components/ChangePassword.js`:

```javascript
import { Auth } from 'aws-amplify';
import { useState } from 'react';

function ChangePassword() {
  const [oldPassword, setOldPassword] = useState('');
  const [newPassword, setNewPassword] = useState('');

  const handleChangePassword = async (e) => {
    e.preventDefault();
    try {
      const user = await Auth.currentAuthenticatedUser();
      await Auth.changePassword(user, oldPassword, newPassword);
      alert('Password changed successfully!');
      setOldPassword('');
      setNewPassword('');
    } catch (error) {
      console.error('Error:', error);
      alert(error.message);
    }
  };

  return (
    <form onSubmit={handleChangePassword}>
      <h2>Change Password</h2>
      <input type="password" value={oldPassword} onChange={(e) => setOldPassword(e.target.value)} placeholder="Current Password" required />
      <input type="password" value={newPassword} onChange={(e) => setNewPassword(e.target.value)} placeholder="New Password" required />
      <button type="submit">Change Password</button>
    </form>
  );
}

export default ChangePassword;
```

## Password Policy

Configure password requirements in Amplify:

```bash
amplify update auth
# Choose: Walkthrough all the auth configurations
# Password policy: Custom
# Minimum length: 8
# Require lowercase: Yes
# Require uppercase: Yes
# Require numbers: Yes
# Require symbols: Yes
```

Deploy changes:

```bash
amplify push
```

## Password Validation

Add client-side validation:

```javascript
const validatePassword = (password) => {
  const minLength = 8;
  const hasUpperCase = /[A-Z]/.test(password);
  const hasLowerCase = /[a-z]/.test(password);
  const hasNumbers = /\d/.test(password);
  const hasSymbols = /[!@#$%^&*(),.?":{}|<>]/.test(password);

  if (password.length < minLength) {
    return 'Password must be at least 8 characters';
  }
  if (!hasUpperCase) {
    return 'Password must contain uppercase letter';
  }
  if (!hasLowerCase) {
    return 'Password must contain lowercase letter';
  }
  if (!hasNumbers) {
    return 'Password must contain number';
  }
  if (!hasSymbols) {
    return 'Password must contain special character';
  }
  return null;
};
```