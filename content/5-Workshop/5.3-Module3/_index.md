---
title: "User Registration & Verification"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Initialize React App

Create a new React application and add Amplify:

```bash
npx create-react-app auth-workshop
cd auth-workshop
npm install aws-amplify @aws-amplify/ui-react
```

## Initialize Amplify Project

```bash
amplify init
# Follow prompts:
# - Project name: authworkshop
# - Environment: dev
# - Default editor: Visual Studio Code
# - App type: javascript
# - Framework: react
# - Source directory: src
# - Distribution directory: build
# - Build command: npm run build
# - Start command: npm start
```

## Add Authentication

```bash
amplify add auth
# Choose: Default configuration
# Sign-in method: Email
# Advanced settings: No
```

## Deploy Backend

```bash
amplify push
```

## Build Registration UI

Create `src/components/Register.js`:

```javascript
import { Auth } from 'aws-amplify';
import { useState } from 'react';

function Register() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [code, setCode] = useState('');
  const [step, setStep] = useState('register');

  const handleRegister = async (e) => {
    e.preventDefault();
    try {
      await Auth.signUp({
        username: email,
        password: password,
        attributes: { email }
      });
      setStep('verify');
    } catch (error) {
      console.error('Error:', error);
    }
  };

  const handleVerify = async (e) => {
    e.preventDefault();
    try {
      await Auth.confirmSignUp(email, code);
      alert('Email verified successfully!');
    } catch (error) {
      console.error('Error:', error);
    }
  };

  return step === 'register' ? (
    <form onSubmit={handleRegister}>
      <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
      <button type="submit">Register</button>
    </form>
  ) : (
    <form onSubmit={handleVerify}>
      <input type="text" value={code} onChange={(e) => setCode(e.target.value)} placeholder="Verification Code" required />
      <button type="submit">Verify</button>
    </form>
  );
}

export default Register;
```

## Test Registration

1. Start the app: `npm start`
2. Navigate to the registration page
3. Enter email and password
4. Check email for verification code
5. Enter code to verify account