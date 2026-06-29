# **SCREEN SPECIFICATION**

---

# **1. Thông tin màn hình**

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-AU-001 |
| Tên màn hình | Đăng nhập |
| Tên tiếng Nhật | ログイン |
| Module | Authentication |
| Chức năng | Đăng nhập hệ thống quản trị Platform  |
| Actor | Platform Admin |
| URL | /admin/login |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# **2. Mục đích màn hình**

Cho phép quản trị viên Platform đăng nhập vào hệ thống quản trị Platform để quản lý Tenant, …

---

# **3. Điều kiện truy cập**

## **Điều kiện trước**

- Quản trị viên chưa đăng nhập hoặc phiên làm việc đã hết hạn.
- Có tài khoản hoạt động trong bảng admin_user của Platform schema.

## **Điều kiện sau**

- Đăng nhập thành công và chuyển hướng đến Dashboard của Platform Admin.
- Khởi tạo Session đăng nhập quản trị và lưu trạng thái đăng nhập nếu chọn Remember Login.

---

# **4. Di chuyển màn hình**

## **Màn hình nguồn**

| Screen ID | Tên màn hình |
| --- | --- |
| PA-AU-001 | Login |

## **Màn hình đích**

| Screen ID | Tên màn hình |
| --- | --- |
| PA-DB-001 | Dashboard |
| SA-AUT-004 | Password Reset |

---

# **5. UI/UX Layout**

!image.png

---

# **6. Danh sách Item màn hình**

## **Khu vực đăng nhập**

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | User ID | Textbox | varchar | Yes | ID đăng nhập của quản trị viên |
| 2 | Password | Password Textbox | varchar | Yes | Password đăng nhập |
| 3 | Show Password | Icon Button | Boolean | No | Hiển thị hoặc ẩn mật khẩu |
| 4 | Remember Login | Checkbox | Boolean | No | Lưu trạng thái đăng nhập |
| 5 | Forgot Password | Link | URL | No | Chuyển sang màn hình đặt lại mật khẩu |
| 6 | Login Button | Button | Action | Yes | Thực hiện đăng nhập hệ thống |
| 7 | Error Message | Label | Text | No | Hiển thị thông điệp lỗi đăng nhập |

---

# **7. Validation**

| Item | Rule | Message |
| --- | --- | --- |
| User ID | Required | User IDを入力してください |
| Password | Required | パスワードを入力してください |
| User ID | Max 255 ký tự | User IDは255文字以内で入力してください |
| Password | Max 255 ký tự | パスワードは255文字以内で入力してください |
| Login | Tài khoản hoặc mật khẩu không đúng | ユーザーIDまたはパスワードが正しくありません |
| Account Lock | Vượt quá số lần đăng nhập sai cho phép | アカウントがロックされています |

---

# **8. Event Definition**

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| api | Login | Click ログイン button | - | 1. Validate User ID, Password
2. Gọi API Login
3. Kiểm tra trạng thái tài khoản quản trị viên
4. Khởi tạo Session đăng nhập thành công
5. Chuyển hướng về trang Dashboard của Platform Admin |
| screen | Remember Login | Tick checkbox ログイン情報を保存 | - | 1. Hệ thống ghi nhận tùy chọn lưu trạng thái đăng nhập
2. Tự động phục hồi session trong các lần truy cập sau |
| screen | Forgot Password | Click パスワードの再設定? link | - | 1. Chuyển hướng sang màn hình Password Reset của Admin |

---

# **9. API Mapping**

## **Đăng nhập**

### **Endpoint**

```
POST /api/v1/admin/login
```

Request

```json
{
  "login_id": "admin001",
  "password": "password123",
  "remember_login": true
}
```

Response

```json
{
  "acess_token": "jwt-access_token-for-platform-admin",
  "refresh_token": "jwt-refresh_token-for-platform-admin",
  "admin": {
    "admin_user_id": "01H2MX7Z...",
    "login_id": "admin001",
    "admin_name": "Super Admin",
    "role": 1
  },
  "permissions": [
    "platform.dashboard.platform_dashboard.view",
    "platform.tenant.moto_tenant_list.view",
  ]
}
```

---

# **10. Message Definition**

Reference Link

---

# **11. Error Handling**

Reference Link

---

# 12. Related Documents

- Business Flow Diagram
- ERD
- API Specification
- Role Matrix
- Wireframe
- Message List
- Error Handling
- Audit Log
- Notification Flow