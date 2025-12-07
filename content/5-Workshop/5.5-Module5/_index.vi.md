---
title: "Quản lý mật khẩu"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Quy trình quên mật khẩu

Tạo file `src/components/ForgotPassword.js`:

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
      alert('Mã xác thực đã được gửi đến email của bạn');
    } catch (error) {
      console.error('Lỗi:', error);
      alert(error.message);
    }
  };

  const handleResetPassword = async (e) => {
    e.preventDefault();
    try {
      await Auth.forgotPasswordSubmit(email, code, newPassword);
      alert('Đặt lại mật khẩu thành công!');
      setStep('complete');
    } catch (error) {
      console.error('Lỗi:', error);
      alert(error.message);
    }
  };

  if (step === 'request') {
    return (
      <form onSubmit={handleRequestReset}>
        <h2>Quên mật khẩu</h2>
        <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
        <button type="submit">Gửi mã</button>
      </form>
    );
  }

  if (step === 'reset') {
    return (
      <form onSubmit={handleResetPassword}>
        <h2>Đặt lại mật khẩu</h2>
        <input type="text" value={code} onChange={(e) => setCode(e.target.value)} placeholder="Mã xác thực" required />
        <input type="password" value={newPassword} onChange={(e) => setNewPassword(e.target.value)} placeholder="Mật khẩu mới" required />
        <button type="submit">Đặt lại mật khẩu</button>
      </form>
    );
  }

  return <div>Đặt lại mật khẩu hoàn tất! <a href="/login">Đi đến Đăng nhập</a></div>;
}

export default ForgotPassword;
```

## Đổi mật khẩu

Tạo file `src/components/ChangePassword.js`:

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
      alert('Đổi mật khẩu thành công!');
      setOldPassword('');
      setNewPassword('');
    } catch (error) {
      console.error('Lỗi:', error);
      alert(error.message);
    }
  };

  return (
    <form onSubmit={handleChangePassword}>
      <h2>Đổi mật khẩu</h2>
      <input type="password" value={oldPassword} onChange={(e) => setOldPassword(e.target.value)} placeholder="Mật khẩu hiện tại" required />
      <input type="password" value={newPassword} onChange={(e) => setNewPassword(e.target.value)} placeholder="Mật khẩu mới" required />
      <button type="submit">Đổi mật khẩu</button>
    </form>
  );
}

export default ChangePassword;
```

## Chính sách mật khẩu

Cấu hình yêu cầu mật khẩu trong Amplify:

```bash
amplify update auth
# Chọn: Walkthrough all the auth configurations
# Password policy: Custom
# Minimum length: 8
# Require lowercase: Yes
# Require uppercase: Yes
# Require numbers: Yes
# Require symbols: Yes
```

Triển khai thay đổi:

```bash
amplify push
```

## Xác thực mật khẩu

Thêm xác thực phía client:

```javascript
const validatePassword = (password) => {
  const minLength = 8;
  const hasUpperCase = /[A-Z]/.test(password);
  const hasLowerCase = /[a-z]/.test(password);
  const hasNumbers = /\d/.test(password);
  const hasSymbols = /[!@#$%^&*(),.?":{}|<>]/.test(password);

  if (password.length < minLength) {
    return 'Mật khẩu phải có ít nhất 8 ký tự';
  }
  if (!hasUpperCase) {
    return 'Mật khẩu phải chứa chữ in hoa';
  }
  if (!hasLowerCase) {
    return 'Mật khẩu phải chứa chữ thường';
  }
  if (!hasNumbers) {
    return 'Mật khẩu phải chứa số';
  }
  if (!hasSymbols) {
    return 'Mật khẩu phải chứa ký tự đặc biệt';
  }
  return null;
};
```