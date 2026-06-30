# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-USER-005 |
| Tên màn hình | Khóa/Mở khóa tài khoản |
| Tên tiếng Nhật | ロック解除 |
| Module | User Management |
| Chức năng | Xem trạng thái khóa tài khoản, khóa và mở khóa tài khoản quản trị viên Platform |
| Actor | Platform SaaS Admin |
| URL | /admin/users/lock-management |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# 2. Mục đích màn hình

Cho phép quản trị viên Platform kiểm tra số lần đăng nhập sai và thực hiện thao tác khóa hoặc mở khóa cho một tài khoản quản trị viên khác để kiểm soát an ninh truy cập.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống Platform SaaS Admin.
- Có quyền chỉnh sửa người dùng (platform.user.platform_user_list.edit).
- Đã chọn một quản trị viên từ màn hình danh sách (PA-USER-001).

## Điều kiện sau

- Thay đổi trạng thái khóa/mở khóa của tài khoản thành công.

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
| Thay đổi thành công | PA-USER-001 | Platform User List |
| Hủy bỏ (Cancel) | PA-USER-001 | Platform User List |

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
| 3 | Email | Label | varchar | No | Email tài khoản mục tiêu |
| 4 | Lock Status | Label | smallint | No | Trạng thái khóa hiện tại (Locked / Unlocked) |
| 5 | Failed Login Count | Label | smallint | No | Số lần đăng nhập sai liên tiếp hiện tại |

## Khu vực thao tác

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 6 | Lock Account | Button | Action | Yes | Thực hiện khóa tài khoản (Hiển thị khi đang Unlocked) |
| 7 | Unlock Account | Button | Action | Yes | Thực hiện mở khóa tài khoản (Hiển thị khi đang Locked) |
| 8 | Cancel | Button | Action | Yes | Hủy bỏ và quay lại |

---

# 7. Validation

Reference Link

---

# 8. Event Definition

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| api | Initial Load | Mở màn hình | platform.user.platform_user_list.edit | 1. Nhận ID người dùng từ màn hình PA-USER-001.<br>2. Gọi API GET `/api/v1/admin/users/{id}` để lấy thông tin chi tiết tài khoản.<br>3. Gọi API GET `/api/v1/admin/users/{id}/lock-status` để lấy trạng thái khóa và số lần đăng nhập sai.<br>4. Hiển thị thông tin tương ứng. |
| screen | Cancel | Click Cancel button | platform.user.platform_user_list.edit | Quay lại màn hình danh sách PA-USER-001. |
| api | Lock Account | Click Lock Account button | platform.user.platform_user_list.edit | 1. Hiển thị popup xác nhận:<br>アカウントをロックします。よろしいですか。<br>2. Gọi API POST `/api/v1/admin/users/{id}/lock`.<br>3. Hiển thị Toast thông báo thành công và chuyển hướng về màn hình PA-USER-001. |
| api | Unlock Account | Click Unlock Account button | platform.user.platform_user_list.edit | 1. Hiển thị popup xác nhận:<br>アカウントのロックを解除します。よろしいですか。<br>2. Gọi API POST `/api/v1/admin/users/{id}/unlock`.<br>3. Hiển thị Toast thông báo thành công và chuyển hướng về màn hình PA-USER-001. |

---

# 9. API Mapping

## 1. Get Platform User Detail (Initial Load)

### Endpoint

```
GET /api/v1/admin/users/{id}
```

Response

```json
{
  "data": {
    "admin_user_id": "01H2MX7Z...",
    "login_id": "admin002",
    "admin_name": "山田 太郎",
    "email": "yamada@platform-admin.jp"
  }
}
```

---

## 2. Get Account Lock Status

### Endpoint

```
GET /api/v1/admin/users/{id}/lock-status
```

Response

```json
{
  "data": {
    "admin_user_id": "01H2MX7Z...",
    "lock_status": 1,
    "failed_login_count": 5
  }
}
```

---

## 3. Lock Platform User

### Endpoint

```
POST /api/v1/admin/users/{id}/lock
```

Response

```json
{
  "message": "アカウントをロックしました"
}
```

---

## 4. Unlock Platform User

### Endpoint

```
POST /api/v1/admin/users/{id}/unlock
```

Response

```json
{
  "message": "アカウントのロックを解除しました"
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
