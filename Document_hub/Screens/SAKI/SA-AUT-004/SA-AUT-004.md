# **SCREEN SPECIFICATION**

---

# **1. Thông tin màn hình**

| Item | Nội dung |
| --- | --- |
| Screen ID | SA-AUT-004 |
| Tên màn hình | Cấp lại mật khẩu |
| Tên tiếng Nhật | パスワード再発行 |
| Module | Authentication |
| Chức năng | Gửi yêu cầu cấp lại mật khẩu qua email khi quên mật khẩu |
| Actor | Common User |
| URL | /saki/password-reset |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# **2. Mục đích màn hình**

Cho phép người dùng gửi yêu cầu cấp lại mật khẩu bằng cách nhập thông tin tài khoản (Mã doanh nghiệp, User ID, Email). Hệ thống sẽ xác thực và gửi liên kết đặt lại mật khẩu mới tới email của người dùng.

---

# **3. Điều kiện truy cập**

## **Điều kiện trước**

- Người dùng chưa đăng nhập hoặc đã click vào liên kết quên mật khẩu từ màn hình đăng nhập.

## **Điều kiện sau**

- Hệ thống xác thực thông tin thành công, gửi email chứa liên kết khôi phục mật khẩu và quay lại màn hình đăng nhập.

---

# **4. Di chuyển màn hình**

## **Màn hình nguồn**

| Screen ID | Tên màn hình |
| --- | --- |
| SA-AUTH-001 | Login |

## **Màn hình đích**

| Screen ID | Tên màn hình |
| --- | --- |
| SA-AUTH-001 | Login |

---

# **5. UI/UX Layout**


---

# **6. Danh sách Item màn hình**

## **Khu vực nhập liệu**

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Company ID | Textbox | varchar | Yes | Mã doanh nghiệp của người dùng |
| 2 | User ID | Textbox | varchar | Yes | ID đăng nhập của người dùng |
| 3 | Email | Textbox | varchar | Yes | Địa chỉ email đăng ký tài khoản |
| 4 | Submit Button | Button | Action | Yes | Gửi yêu cầu cấp lại mật khẩu |
| 5 | Cancel Button | Button | Action | Yes | Hủy yêu cầu và quay lại màn hình đăng nhập |
| 6 | Error Message | Label | Text | No | Hiển thị thông điệp lỗi hệ thống hoặc xác thực |

---

# **7. Validation**

| Item | Rule | Message |
| --- | --- | --- |
| Company ID | Required | 企業IDを入力してください |
| User ID | Required | ユーザーIDを入力してください |
| Email | Required | メールアドレスを入力してください |
| Email | Email Format | メールアドレスの形式が正しくありません |
| Submit | Tài khoản hoặc email không tồn tại | 入力された情報に該当するアカウントが見つかりません |

---

# **8. Event Definition**

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| api | Submit Request | Click 送信 button | - | 1. Validate Company ID, User ID, Email<br>2. Gọi API POST `/api/v1/saki/auth/password-reset-request`<br>3. Hệ thống gửi email khôi phục mật khẩu và hiển thị popup thông báo gửi thành công<br>4. Chuyển hướng người dùng về màn hình Login |
| screen | Cancel | Click キャンセル button | - | 1. Chuyển hướng người dùng quay lại màn hình Login |

---

# **9. API Mapping**

## **Yêu cầu cấp lại mật khẩu**

### **Endpoint**

```
POST /api/v1/saki/auth/password-reset-request
```

Request

```json
{
  "company_id": "comp001",
  "user_id": "user001",
  "email": "user001@example.com"
}
```

Response

```json
{
  "message": "パスワード再設定用のメールを送信しました"
}
```

---

# **10. Message Definition**

Reference Link

---

# **11. Error Handling**

Reference Link

---

# **12. Related Documents**

- Business Flow Diagram
- ERD
- API Specification
- Wireframe
- Message List
- Error Handling
- Audit Log
- Notification Flow
