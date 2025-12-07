---
title: "Tính năng nâng cao & Dọn dẹp"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Xác thực đa yếu tố (MFA)

Kích hoạt MFA dựa trên TOTP:

```bash
amplify update auth
# Chọn: Walkthrough all the auth configurations
# MFA: Optional
# MFA type: TOTP (Time-based One-Time Password)
amplify push
```

Triển khai MFA trong ứng dụng:

```javascript
import { Auth } from 'aws-amplify';

// Thiết lập MFA cho người dùng
async function setupMFA() {
  try {
    const user = await Auth.currentAuthenticatedUser();
    const code = await Auth.setupTOTP(user);
    
    // Hiển thị mã QR cho người dùng
    const qrCodeUrl = `otpauth://totp/AWSCognito:${user.username}?secret=${code}&issuer=AuthWorkshop`;
    console.log('QR Code URL:', qrCodeUrl);
    
    // Xác thực token TOTP
    const token = prompt('Nhập mã TOTP từ ứng dụng xác thực:');
    await Auth.verifyTotpToken(user, token);
    await Auth.setPreferredMFA(user, 'TOTP');
    
    alert('Kích hoạt MFA thành công!');
  } catch (error) {
    console.error('Lỗi thiết lập MFA:', error);
  }
}
```

## Đăng nhập mạng xã hội

Thêm xác thực Google/Facebook:

```bash
amplify update auth
# Chọn: Walkthrough all the auth configurations
# Social providers: Google, Facebook
# Làm theo hướng dẫn để cấu hình OAuth
amplify push
```

Thêm nút đăng nhập mạng xã hội:

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
      <button onClick={handleGoogleLogin}>Đăng nhập với Google</button>
      <button onClick={handleFacebookLogin}>Đăng nhập với Facebook</button>
    </div>
  );
}
```

## Nhóm người dùng & Vai trò

Tạo nhóm người dùng trong Cognito console:

1. Truy cập Cognito User Pools
2. Chọn user pool của bạn
3. Điều hướng đến "Users and groups"  "Groups"
4. Tạo nhóm: `Admin`, `User`

Gán người dùng vào nhóm qua code:

```javascript
// Code phía server (Lambda)
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

## Triển khai lên Amplify Hosting

Triển khai ứng dụng React:

```bash
amplify add hosting
# Chọn: Amplify Console
# Type: Manual deployment

amplify publish
```

Ứng dụng của bạn sẽ được triển khai lên URL trực tiếp.

## Dọn dẹp tài nguyên

Xóa tất cả tài nguyên AWS:

```bash
# Xóa backend Amplify
amplify delete

# Xác nhận xóa
# Lệnh này sẽ xóa:
# - Cognito User Pool
# - Tất cả tài nguyên xác thực
# - CloudFormation stacks
```

Xóa file local:

```bash
# Xóa file dự án Amplify
Remove-Item -Recurse -Force amplify
Remove-Item -Force aws-exports.js
```

## Tối ưu chi phí

- **Cognito User Pools**: Gói miễn phí hỗ trợ 50,000 MAUs (Người dùng hoạt động hàng tháng)
- **Amplify Hosting**: Phút build và truyền dữ liệu có giới hạn gói miễn phí
- **Thực hành tốt nhất**: Luôn xóa tài nguyên không sử dụng

## Chúc mừng! 

Bạn đã hoàn thành Workshop Xác thực bao gồm:
-  Đăng ký người dùng và xác thực email
-  Đăng nhập và quản lý phiên
-  Quy trình đặt lại mật khẩu
-  MFA với TOTP
-  Xác thực mạng xã hội
- Nhóm người dùng và vai trò
- Triển khai production