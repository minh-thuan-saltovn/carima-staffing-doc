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
- Có quyền tạo mới người dùng (platform.user.platform_user.create).

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
| Hủy bỏ | PA-USER-001 | Platform User List |

---

# 5. UI/UX Layout

{image}

---

# 6. Định nghĩa Item màn hình

## Khu vực nhập liệu

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Login ID | Textbox | varchar | Yes | Tên đăng nhập của Admin |
| 2 | Name | Textbox | varchar | Yes | Tên quản trị viên |
| 3 | Email | Textbox | varchar | Yes | Địa chỉ email nhận thông tin kích hoạt |
| 4 | Role | Dropdown | bigint | Yes | Vai trò quản trị |
| 5 | Register | Button | Action | Yes | Thực hiện tạo mới |
| 6 | Cancel | Button | Action | Yes | Hủy bỏ và quay lại |

---

# 7. Validation

[Reference Link](https://app.notion.com/p/Validation-Rule-378f02c407dd805aae8acbb637c995d5?source=copy_link)

---

# 8. Event Definition

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| api | Initial Load | Mở màn hình | platform.user.platform_user.view | Load danh sách Vai trò từ Database vào Dropdown Role. |
| screen | Cancel | Click Cancel button | - | Quay lại màn hình danh sách PA-USER-001. |
| api | Submit Create | Click Register button | platform.user.platform_user.create | 1. Thực hiện validate form nhập liệu.<br>2. Gọi API POST `/api/v1/admin/users`.<br>3. Tài khoản được tạo với `status = 0` (chưa kích hoạt).<br>4. Hệ thống gửi email chứa link thiết lập mật khẩu lần đầu đến địa chỉ email đã nhập.<br>5. Sau khi người dùng click link và đặt mật khẩu thành công, `status` tự động chuyển sang `1` (hoạt động).<br>6. Quay lại màn hình danh sách PA-USER-001 và hiển thị Toast thông báo thành công. |

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
  "full_name": "山田 太郎",
  "email": "yamada@platform-admin.jp",
  "role_id": 2
}
```

Response

```json
{
  "data": {
    "id": 1002,
    "login_id": "admin002",
    "full_name": "山田 太郎",
    "email": "yamada@platform-admin.jp",
    "role_id": 2,
    "status": 0
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