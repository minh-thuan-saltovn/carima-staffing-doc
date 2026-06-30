# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-USER-002 |
| Tên màn hình | Tạo quản trị viên Platform mới |
| Tên tiếng Nhật | 管理ユーザー登録 |
| Module | User Management |
| Chức năng | Thêm mới một tài khoản quản trị viên Platform |
| Actor | Platform SaaS Admin |
| URL | /admin/users/create |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# 2. Mục đích màn hình

Cho phép quản trị viên Platform tạo mới tài khoản quản trị viên khác. Hệ thống sẽ tự động gửi email thông báo thiết lập mật khẩu đến địa chỉ email đăng ký để kích hoạt tài khoản.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống Platform SaaS Admin.
- Có quyền tạo mới người dùng (platform.user.create).

## Điều kiện sau

- Tạo mới tài khoản quản trị viên thành công.

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
| Đăng ký thành công | PA-USER-001 | Platform User List |
| Hủy bỏ (Cancel) | PA-USER-001 | Platform User List |

---

# 5. UI/UX Layout

{image}

---

# 6. Định nghĩa Item màn hình

## Khu vực nhập liệu

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Login ID | Textbox | varchar | Yes | Mã đăng nhập tài khoản mới |
| 2 | Name | Textbox | varchar | Yes | Tên quản trị viên |
| 3 | Email | Textbox | varchar | Yes | Địa chỉ email nhận thông tin kích hoạt |
| 4 | Role | Dropdown | smallint | Yes | Vai trò (1: Super Admin, 2: Operator) |
| 5 | Register | Button | Action | Yes | Thực hiện tạo mới |
| 6 | Cancel | Button | Action | Yes | Hủy bỏ và quay lại |

---

# 7. Validation

Reference Link

---

# 8. Event Definition

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| api | Initial Load | Mở màn hình | platform.user.create | Load danh sách Vai trò (Role) từ Database vào Dropdown Role. |
| screen | Cancel | Click Cancel button | platform.user.create | Quay lại màn hình danh sách PA-USER-001. |
| api | Submit Create | Click Register button | platform.user.create | 1. Thực hiện validate form nhập liệu.<br>2. Gọi API POST `/api/v1/admin/users`.<br>3. Tài khoản được tạo với `status = 0` (chưa kích hoạt).<br>4. Hệ thống gửi email chứa link thiết lập mật khẩu lần đầu đến địa chỉ email đã nhập.<br>5. Sau khi người dùng click link và đặt mật khẩu thành công, `status` tự động chuyển sang `1` (hoạt động).<br>6. Quay lại màn hình danh sách PA-USER-001 và hiển thị Toast thông báo thành công. |

---

# 9. API Mapping

## Create Platform User

### Endpoint

```
POST /api/v1/admin/users
```

Request

```json
{
  "login_id": "admin002",
  "admin_name": "山田 太郎",
  "email": "yamada@platform-admin.jp",
  "role": 2
}
```

Response

```json
{
  "data": {
    "admin_user_id": "01H2MX7Z...",
    "login_id": "admin002",
    "admin_name": "山田 太郎",
    "email": "yamada@platform-admin.jp",
    "role": 2,
    "status": 0
  }
}
```

---

# 10. Message Definition

Reference Link

---

# 11. Error Handling

Reference Link

---

# 12. Related Documents

- Business Flow Diagram
- ERD
- API Specification
- Role Matrix
- Wireframe
- NFR