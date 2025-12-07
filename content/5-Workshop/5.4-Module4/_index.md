---
title: "Login & Session Management"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Build Login Component

Create `src/components/Login.js`:

```javascript
import { Auth } from 'aws-amplify';
import { useState } from 'react';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [user, setUser] = useState(null);

  const handleLogin = async (e) => {
    e.preventDefault();
    try {
      const user = await Auth.signIn(email, password);
      setUser(user);
      console.log('Login successful:', user);
    } catch (error) {
      console.error('Error signing in:', error);
      alert(error.message);
    }
  };

  const handleLogout = async () => {
    try {
      await Auth.signOut();
      setUser(null);
      console.log('Logged out successfully');
    } catch (error) {
      console.error('Error signing out:', error);
    }
  };

  return (
    <div>
      {!user ? (
        <form onSubmit={handleLogin}>
          <h2>Login</h2>
          <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
          <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
          <button type="submit">Login</button>
        </form>
      ) : (
        <div>
          <h2>Welcome, {user.attributes.email}</h2>
          <button onClick={handleLogout}>Logout</button>
        </div>
      )}
    </div>
  );
}

export default Login;
```

## Check Authentication State

Add to `src/App.js`:

```javascript
import { Amplify } from 'aws-amplify';
import { Auth } from 'aws-amplify';
import { useEffect, useState } from 'react';
import awsconfig from './aws-exports';
import Login from './components/Login';

Amplify.configure(awsconfig);

function App() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    checkUser();
  }, []);

  const checkUser = async () => {
    try {
      const user = await Auth.currentAuthenticatedUser();
      setUser(user);
    } catch (error) {
      setUser(null);
    }
    setLoading(false);
  };

  if (loading) return <div>Loading...</div>;

  return (
    <div className="App">
      {user ? <h1>Welcome back!</h1> : <Login />}
    </div>
  );
}

export default App;
```

## Session Management

Cognito automatically manages sessions:

- **Access Token**: Valid for 1 hour, used for API authorization
- **ID Token**: Contains user attributes, expires after 1 hour
- **Refresh Token**: Valid for 30 days (configurable), refreshes access/ID tokens

### Auto-refresh Sessions

Amplify automatically refreshes tokens:

```javascript
// Get current session
const session = await Auth.currentSession();
console.log('Access token:', session.getAccessToken().getJwtToken());

// Amplify handles token refresh automatically
// No manual refresh needed
```

## Protected Routes

Create protected routes in your app:

```javascript
import { Route, Navigate } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      await Auth.currentAuthenticatedUser();
      setUser(true);
    } catch {
      setUser(false);
    }
    setLoading(false);
  };

  if (loading) return <div>Đang tải...</div>;
  return user ? children : <Navigate to="/login" />;
}

export default ProtectedRoute;
```