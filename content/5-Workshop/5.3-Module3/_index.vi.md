---
title: "Đăng ký & Xác thực người dùng"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Khởi tạo ứng dụng React

Tạo ứng dụng React mới và thêm Amplify:

```bash
npx create-react-app auth-workshop
cd auth-workshop
npm install aws-amplify @aws-amplify/ui-react
```

## Khởi tạo dự án Amplify

```bash
amplify init
# Làm theo hướng dẫn:
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

## Thêm xác thực

```bash
amplify add auth
# Chọn: Default configuration
# Sign-in method: Email
# Advanced settings: No
```

## Triển khai Backend

```bash
amplify push
```

## Xây dựng giao diện đăng ký

Tạo file `src/components/Register.js`:

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
      console.error('Lỗi:', error);
    }
  };

  const handleVerify = async (e) => {
    e.preventDefault();
    try {
      await Auth.confirmSignUp(email, code);
      alert('Xác thực email thành công!');
    } catch (error) {
      console.error('Lỗi:', error);
    }
  };

  return step === 'register' ? (
    <form onSubmit={handleRegister}>
      <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Mật khẩu" required />
      <button type="submit">Đăng ký</button>
    </form>
  ) : (
    <form onSubmit={handleVerify}>
      <input type="text" value={code} onChange={(e) => setCode(e.target.value)} placeholder="Mã xác thực" required />
      <button type="submit">Xác thực</button>
    </form>
  );
}

export default Register;
```

## Kiểm tra đăng ký

1. Khởi động ứng dụng: `npm start`
2. Điều hướng đến trang đăng ký
3. Nhập email và mật khẩu
4. Kiểm tra email để lấy mã xác thực
5. Nhập mã để xác thực tài khoản