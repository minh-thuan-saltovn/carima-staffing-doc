# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-USER-001 |
| Tên màn hình | Danh sách quản trị viên Platform |
| Tên tiếng Nhật | 管理ユーザー一覧 |
| Module | User Management |
| Chức năng | Hiển thị danh sách các tài khoản quản trị viên Platform |
| Actor | Platform SaaS Admin |
| URL | /admin/users |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# 2. Mục đích màn hình

Cho phép người dùng:

- Tìm kiếm các tài khoản quản trị viên Platform dựa trên các điều kiện lọc.
- Xem danh sách quản trị viên kèm thông tin cơ bản và trạng thái hoạt động.
- Điều hướng sang màn hình tạo mới (PA-USER-002), đặt lại mật khẩu (PA-USER-004), và khóa/mở khóa tài khoản (PA-USER-005).

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống Platform SaaS Admin.
- Có quyền xem danh sách người dùng (platform.user.platform_user_list.view).

## Điều kiện sau

- Hiển thị danh sách quản trị viên Platform theo điều kiện tìm kiếm.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình |
| --- | --- |
| PA-DB-001 | Dashboard |

---

## Màn hình đích

| Action | Screen ID | Tên màn hình |
| --- | --- | --- |
| Tạo mới | PA-USER-002 | Tạo quản trị viên Platform mới |
| Đặt lại mật khẩu | PA-USER-004 | Khởi tạo lại mật khẩu |
| Khóa/Mở khóa | PA-USER-005 | Khóa/Mở khóa tài khoản |

---

# 5. UI/UX Layout

{image}

---

# 6. Định nghĩa Item màn hình

## Khu vực tìm kiếm

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Login ID | Textbox | varchar | No | Tên đăng nhập của quản trị viên |
| 2 | Name | Textbox | varchar | No | Tên quản trị viên |
| 3 | Role | Dropdown | smallint | No | Vai trò quản trị (1: Super Admin, 2: Operator) |
| 4 | Status | Dropdown | smallint | No | Trạng thái (0: Vô hiệu, 1: Hoạt động) |
| 5 | Search | Button | Action | Yes | Thực hiện tìm kiếm |
| 6 | Clear | Button | Action | Yes | Xóa điều kiện tìm kiếm |

---

## Danh sách hiển thị

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 7 | Create User | Button | Action | Yes | Điều hướng sang màn hình tạo mới (PA-USER-002) |
| 8 | Login ID Column | Label | varchar | No | Mã đăng nhập tài khoản |
| 9 | Name Column | Label | varchar | No | Tên quản trị viên |
| 10 | Email Column | Label | varchar | No | Địa chỉ email đăng ký |
| 11 | Role Column | Label | smallint | No | Vai trò quản trị |
| 12 | Status Column | Label | smallint | No | Trạng thái hoạt động tài khoản |
| 13 | Last Login Column | Label | timestamptz | No | Thời gian đăng nhập gần nhất |
| 14 | Reset Password Action | Link | Action | No | Click để mở màn hình đặt lại mật khẩu (PA-USER-004) |
| 15 | Lock/Unlock Action | Link | Action | No | Click để mở màn hình khóa/mở khóa (PA-USER-005) |

---

# 7. Validation

Reference Link

---

# 8. Event Definition

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| api | Initial Load | Mở màn hình | platform.user.platform_user_list.view | 1. Load danh sách vai trò và trạng thái vào Dropdown lọc.<br>2. Gọi API GET `/api/v1/admin/users` để lấy danh sách mặc định. |
| api | Search | Click Search button | platform.user.platform_user_list.view | 1. Lấy điều kiện tìm kiếm từ Form.<br>2. Gọi API GET `/api/v1/admin/users` kèm các tham số lọc tìm kiếm.<br>3. Hiển thị kết quả lên bảng danh sách. |
| screen | Clear | Click Clear button | platform.user.platform_user_list.view | 1. Xóa toàn bộ điều kiện tìm kiếm trên Form.<br>2. Gọi API lấy danh sách mặc định ban đầu. |
| screen | Create User | Click Create User button | platform.user.create | Chuyển hướng sang màn hình PA-USER-002. |
| screen | Reset Password | Click Reset Password link | platform.user.password_reset.reset | Truyền thông tin ID và chuyển hướng sang màn hình PA-USER-004. |
| screen | Lock/Unlock | Click Lock/Unlock link | platform.user.platform_user_list.edit | Truyền thông tin ID và chuyển hướng sang màn hình PA-USER-005. |

---

# 9. API Mapping

## Search Platform Users

### Endpoint

```
GET /api/v1/admin/users
```

Request Param

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| login_id | string | No | Tên đăng nhập |
| admin_name | string | No | Tên quản trị viên |
| role | number | No | Vai trò quản trị |
| status | number | No | Trạng thái (0: Vô hiệu, 1: Hoạt động) |
| page | number | Yes | Số trang |
| limit | number | Yes | Kích thước trang |

Response

```json
{
  "data": [
    {
      "admin_user_id": "01H2MX7Z...",
      "login_id": "admin002",
      "admin_name": "山田 太郎",
      "email": "yamada@platform-admin.jp",
      "role": 2,
      "status": 1,
      "last_login_at": "2026-06-30T09:00:00Z"
    }
  ],
  "meta": {
    "total": 1,
    "page": 1,
    "limit": 10
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
