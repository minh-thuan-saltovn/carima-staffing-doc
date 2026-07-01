# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-USER-004 |
| Tên màn hình | Khởi tạo lại mật khẩu |
| Tên tiếng Nhật | パスワード初期化 |
| Module | User Management |
| Chức năng | Khởi tạo lại mật khẩu của tài khoản quản trị viên Platform bằng cách gửi email thiết lập mật khẩu |
| Actor | Platform SaaS Admin |
| URL | /admin/users/password-reset |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# 2. Mục đích màn hình

Cho phép quản trị viên Platform khởi tạo lại mật khẩu cho một tài khoản quản trị viên khác khi họ quên mật khẩu hoặc cần thiết lập lại.

Sau khi thực hiện thành công:
- Mật khẩu cũ bị vô hiệu hóa.
- Hệ thống gửi email chứa liên kết thiết lập mật khẩu mới đến email của tài khoản được chọn.
- Hiển thị thông tin kết quả reset và thời gian thực hiện.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống Platform SaaS Admin.
- Có quyền khởi tạo lại mật khẩu (platform.user.password_reset.reset).
- Đã chọn một quản trị viên từ màn hình danh sách (PA-USER-001).

## Điều kiện sau

- Gửi yêu cầu khởi tạo lại mật khẩu thành công.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình |
| --- | --- |
| PA-USER-001 | Platform User List |

---

## Màn hình đích

| Action | Screen ID | Tên màn hình |
| --- | --- | --- |
| Reset thành công | PA-USER-001 | Platform User List |
| Hủy bỏ | PA-USER-001 | Platform User List |

---

# 5. UI/UX Layout

{image}

---

# 6. Định nghĩa Item màn hình

## Khu vực hiển thị thông tin (Readonly)

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Login ID | Label | varchar | No | Mã đăng nhập tài khoản mục tiêu |
| 2 | Name | Label | varchar | No | Tên quản trị viên tài khoản mục tiêu |
| 3 | Email | Label | varchar | No | Email nhận liên kết reset |

## Khu vực thao tác

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 4 | Reset | Button | Action | Yes | Xác nhận khởi tạo lại mật khẩu |
| 5 | Cancel | Button | Action | Yes | Hủy bỏ và quay lại |

---

# 7. Validation

[Reference Link](https://app.notion.com/p/Validation-Rule-378f02c407dd805aae8acbb637c995d5?source=copy_link)

---

# 8. Event Definition

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| api | Initial Load | Mở màn hình | platform.user.password_reset.reset | 1. Nhận ID người dùng từ màn hình PA-USER-001.<br>2. Gọi API GET `/api/v1/admin/users/{id}` để lấy thông tin chi tiết.<br>3. Hiển thị thông tin tài khoản (Login ID, Name, Email) dạng readonly. |
| screen | Cancel | Click Cancel button | platform.user.password_reset.reset | Quay lại màn hình danh sách PA-USER-001. |
| api | Reset Password | Click Reset button | platform.user.password_reset.reset | 1. Hiển thị popup xác nhận<br>2. Gọi API POST `/api/v1/admin/users/{id}/password-reset`.<br>3. Hệ thống gửi email chứa liên kết thiết lập lại mật khẩu đến email đích.<br>4. Hiển thị Toast thông báo thành công.<br>5. Chuyển hướng người dùng quay lại màn hình PA-USER-001. |

---

# 9. API Mapping

## 1. Get Platform User Detail

### Endpoint

```
GET /api/v1/admin/users/{id}
```

Response

```json
{
  "data": {
    "id": 1002,
    "login_id": "admin002",
    "full_name": "山田 太郎",
    "email": "yamada@platform-admin.jp"
  }
}
```

---

## 2. Password Reset

### Endpoint

```
POST /api/v1/admin/users/{id}/password-reset
```

Request

```json
{
  "id": 1002
}
```

Response

```json
{
  "message": "パスワード初期化メールを送信しました",
  "data": {
    "reset_at": "2026-06-30T09:30:00Z"
  }
}
```

---

# 10. Message Definition

[Reference Link](https://app.notion.com/p/Message-list-374f02c407dd8037808eea01e93be8aa?source=copy_link)

---

# 11. Error Handling

[Reference Link](https://app.notion.com/p/Common-Error-Handling-37af02c407dd802093eac2ec6dd5a000?source=copy_link)

---

# 12. Related Documents

- Business Flow Diagram
- ERD
- API Specification
- Role Matrix
- Wireframe
- NFR
