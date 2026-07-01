# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | SA-AUTH-004 |
| Tên màn hình | Cấp lại mật khẩu |
| Tên tiếng Nhật | パスワード再発行 |
| Module | Authentication |
| Chức năng | Khởi tạo và cấp lại mật khẩu cho tài khoản người dùng SAKI trên Popup/Modal |
| Actor | SAKI Admin |
| URL | Không có (Popup/Modal trên màn hình danh sách người dùng SAKI) |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# 2. Mục đích

Cho phép quản trị viên SAKI (SAKI Admin) thực hiện cấp lại mật khẩu cho người dùng trực thuộc bằng 3 phương thức lựa chọn: gửi email đặt lại link, tự động sinh mật khẩu tạm thời, hoặc tự chỉ định mật khẩu tạm thời.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống quản trị SAKI.
- Có quyền chỉnh sửa thông tin người dùng (saki.settings.user_master_update.edit).
- Đang ở màn hình Chi tiết người dùng SAKI (SA-USER-003) và click vào nút `パスワードリセット` (Password Reset) ở menu Thao tác bên phải.

## Điều kiện sau

- Cấp lại mật khẩu thành công và gửi thông tin tương ứng cho người dùng.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình |
| --- | --- |
| SA-USER-003 | Chi tiết người dùng SAKI |

---

## Màn hình đích

| Action | Screen ID | Tên màn hình |
| --- | --- | --- |
| Xác nhận thành công | SA-USER-003 | Chi tiết người dùng SAKI (Reload) |
| Hủy bỏ (Cancel) | SA-USER-003 | Chi tiết người dùng SAKI |

---

# 5. UI/UX Layout

{image}

---

# 6. Định nghĩa Item màn hình

## Khu vực lựa chọn phương thức đặt lại (Reset Method)

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Send Reset Email | Radio | smallint | Yes | Gửi email chứa link đặt lại mật khẩu (Giá trị = 1) |
| 2 | Issue Temporary Password | Radio | smallint | Yes | Hệ thống tự động tạo mật khẩu tạm thời (Giá trị = 2) |
| 3 | Specify Temporary Password | Radio | smallint | Yes | Quản trị viên tự nhập mật khẩu tạm thời (Giá trị = 3) |
| 4 | Manual Password | Textbox | password | Yes | Nhập mật khẩu tạm thời thủ công (Chỉ hiển thị khi chọn Item 3) |

## Khu vực tùy chọn nâng cao (Options)

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 5 | Force Password Change | Checkbox | boolean | No | Bắt buộc người dùng đổi mật khẩu ở lần đăng nhập đầu tiên |
| 6 | Send Notification Email | Checkbox | boolean | No | Gửi email thông báo thiết lập cho người dùng |
| 7 | Record Comment | Checkbox | boolean | No | Ghi nhận ý kiến lý do vào lịch sử thao tác |
| 8 | Admin Comment | Textarea | varchar | No | Nhập lý do hoặc thông tin bổ sung (Chỉ hiển thị khi chọn Item 7) |

## Thao tác điều khiển

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 9 | Submit Reset | Button | Action | Yes | Thực hiện đặt lại mật khẩu |
| 10 | Cancel | Button | Action | Yes | Đóng Modal và quay lại |

---

# 7. Validation

[Reference Link](https://app.notion.com/p/Validation-Rule-378f02c407dd805aae8acbb637c995d5?source=copy_link)

---

# 8. Event Definition

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| api | Initial Load | Mở Popup | saki.settings.user_master_update.edit | 1. Nhận thông tin ID người dùng từ màn hình Chi tiết người dùng SAKI (SA-USER-003).<br>2. Hiển thị Popup với phương thức mặc định là "Send Reset Email" (Item 1). |
| screen | Toggle Method | Thay đổi lựa chọn Radio | saki.settings.user_master_update.edit | 1. Nếu chọn "Specify Temporary Password" (Item 3), hiển thị trường nhập mật khẩu (Item 4). Ngược lại ẩn đi.<br>2. Nếu chọn Checkbox "Record Comment" (Item 7), hiển thị Textarea nhập lý do (Item 8). Ngược lại ẩn đi. |
| screen | Cancel | Click Cancel button | - | Đóng Modal và quay lại màn hình Chi tiết người dùng SAKI (SA-USER-003). |
| api | Submit Reset | Click Submit button | saki.settings.user_master_update.edit | 1. Validate dữ liệu đầu vào (nếu chọn phương thức thủ công, bắt buộc nhập mật khẩu tạm thời).<br>2. Gọi API POST `/api/v1/saki/users/{id}/password-reset`. Payload truyền các tùy chọn đã chọn.<br>3. Nếu chọn tự động sinh mật khẩu, hiển thị popup thông báo mật khẩu tạm thời vừa sinh cho quản trị viên sao chép.<br>4. Đóng Modal và reload màn hình Chi tiết người dùng SAKI (SA-USER-003). |

---

# 9. API Mapping

## Password Reset

### Endpoint

```
POST /api/v1/saki/users/{id}/password-reset
```

Request

```json
{
  "reset_method": 2,
  "temporary_password": "",
  "force_change_on_first_login": true,
  "send_notification_email": true,
  "record_comment": true,
  "admin_comment": "Khởi tạo lại mật khẩu định kỳ theo yêu cầu bảo mật."
}
```

Response

*Trường hợp 1: Chọn phương thức 1 (Gửi email) hoặc 3 (Tự chỉ định mật khẩu)*

```json
{
  "message": "パスワードの再設定が完了しました"
}
```

*Trường hợp 2: Chọn phương thức 2 (Hệ thống tự động sinh mật khẩu tạm thời)*

```json
{
  "data": {
    "temporary_password": "kG9$mP2_tX7#",
    "reset_at": "2026-06-30T10:30:00Z"
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
