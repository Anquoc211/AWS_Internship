---
title: "Đăng nhập & Quản lý phiên"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Xây dựng component đăng nhập

Tạo file `src/components/Login.js`:

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
      console.log('Đăng nhập thành công:', user);
    } catch (error) {
      console.error('Lỗi đăng nhập:', error);
      alert(error.message);
    }
  };

  const handleLogout = async () => {
    try {
      await Auth.signOut();
      setUser(null);
      console.log('Đăng xuất thành công');
    } catch (error) {
      console.error('Lỗi đăng xuất:', error);
    }
  };

  return (
    <div>
      {!user ? (
        <form onSubmit={handleLogin}>
          <h2>Đăng nhập</h2>
          <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
          <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Mật khẩu" required />
          <button type="submit">Đăng nhập</button>
        </form>
      ) : (
        <div>
          <h2>Chào mừng, {user.attributes.email}</h2>
          <button onClick={handleLogout}>Đăng xuất</button>
        </div>
      )}
    </div>
  );
}

export default Login;
```

## Kiểm tra trạng thái xác thực

Thêm vào `src/App.js`:

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

  if (loading) return <div>Đang tải...</div>;

  return (
    <div className="App">
      {user ? <h1>Chào mừng trở lại!</h1> : <Login />}
    </div>
  );
}

export default App;
```

## Quản lý phiên

Cognito tự động quản lý phiên:

- **Access Token**: Có hiệu lực 1 giờ, dùng để ủy quyền API
- **ID Token**: Chứa thuộc tính người dùng, hết hạn sau 1 giờ
- **Refresh Token**: Có hiệu lực 30 ngày (có thể cấu hình), làm mới access/ID tokens

### Tự động làm mới phiên

Amplify tự động làm mới tokens:

```javascript
// Lấy phiên hiện tại
const session = await Auth.currentSession();
console.log('Access token:', session.getAccessToken().getJwtToken());

// Amplify tự động xử lý làm mới token
// Không cần làm mới thủ công
```

## Routes được bảo vệ

Tạo routes được bảo vệ trong ứng dụng:

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