## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Password Reset |
| Diagram / Flow áp dụng | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Portal áp dụng | Platform Manager |
| Version | v0.1 |
| Ngày tạo | 2026/07/01 |
| Ngày cập nhật | 2026/07/01 |
| Người tạo | Thuận |
| Người review | Sang |
| Trạng thái | VN Review |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v0.1 | 2026/07/01 | Thuận | Tạo bản đầu tiên |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-USER-004-API-01 |
| Tên API | Password Reset |
| Business Flow liên quan | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Screen ID liên quan | PA-USER-004 |
| Chức năng liên quan | Đặt lại mật khẩu (Modal) |
| Mục đích API | Đặt lại mật khẩu tài khoản quản trị viên Platform và gửi mail thông báo |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/admin/users/password-reset |
| Authentication | Có |
| Authorization | Có (platform.user.platform_user.password_reset) |
| Phạm vi Tenant | Platform Common (central_db) |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | VN Review |

---

## 3.2 Mô tả API

### Mục đích

API này tiếp nhận yêu cầu đặt lại mật khẩu cho một quản trị viên Platform cụ thể. Hệ thống sẽ sinh mật khẩu tạm thời hoặc áp dụng mật khẩu chỉ định, mã hóa mật khẩu và cập nhật vào trường `password_hash` của bảng `mst_platform_user`, đồng thời gửi thông tin kích hoạt qua email cho người dùng mục tiêu.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Admin bấm nút “Xác nhận đặt lại” |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Client hiển thị modal xác nhận và cho phép nhập lý do |
| Sau khi gọi API | Mật khẩu được cập nhật thành công, log lý do được ghi nhận, gửi email thông báo |
| Thay đổi trạng thái liên quan | Cập nhật password_hash, set pwd_change_required_flg = 1 |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/admin/users/password-reset
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token của Platform Admin |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ thông điệp lỗi trả về |

---

## 4.3 Path Parameters

Không áp dụng.

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "user_id": 1001,
  "reset_method": 1,
  "specified_password": null,
  "force_change_on_next_login": true,
  "send_notification_email": true,
  "record_comment_log": true,
  "admin_comment": "Nhập lý do đặt lại mật khẩu theo yêu cầu"
}
```

---

## 5.2 Định nghĩa field request

| Field | Column (DB) | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- | --- |
| user_id | mst_platform_user.id | number | Có | - | Phải tồn tại trong `mst_platform_user` | ID người dùng cần đặt lại mật khẩu |
| reset_method | - | number | Có | - | 1: Tự động, 2: Chỉ định | Phương thức đặt lại mật khẩu |
| specified_password | - | string | Không | 100 | Bắt buộc nếu reset_method = 2 | Mật khẩu mới tự chỉ định |
| force_change_on_next_login | mst_platform_user.pwd_change_required_flg | boolean | Có | - | true/false | Bắt buộc đổi mật khẩu ở lần đăng nhập tiếp theo |
| send_notification_email | - | boolean | Có | - | - | Gửi mail thông báo cho người dùng |
| record_comment_log | - | boolean | Có | - | - | Ghi nhận log ý kiến của Admin |
| admin_comment | - | string | Không | 1000 | Bắt buộc nếu record_comment_log = true | Lý do thực hiện đặt lại mật khẩu |

---

# 6. Response Definition

### 6.1 Response Body Schema

```json
{
  "data": {
    "temporary_password": "TEMP_PWD_ABC123"
  }
}
```

### 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.temporary_password | string | Mật khẩu tạm thời sinh tự động (trả về null nếu reset_method = 2) |

### 6.3 Handle response status

| HTTP Status | Error Code | Mô tả |
| --- | --- | --- |
| 200 OK | - | Đặt lại mật khẩu thành công |
| 400 Bad Request | VAL001 | Tham số đầu vào không đúng định dạng hoặc thiếu trường bắt buộc |
| 401 Unauthorized | AUTH001 | Token không hợp lệ hoặc đã hết hạn |
| 403 Forbidden | AUTH002 | Không có quyền thực hiện hành động đặt lại mật khẩu |
| 404 Not Found | USR001 | Không tìm thấy người dùng cần đặt lại mật khẩu |
| 500 Internal Server Error | SYS001 | Lỗi hệ thống |

---

## 7. Validation Rules

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | user_id | required | - | user_id_required |
| 2 | reset_method | required | - | reset_method_required |
| 3 | specified_password | required_if | reset_method = 2 | specified_password_required |
| 4 | specified_password | format | alphanumeric, special chars, min 8 | specified_password_invalid_format |
| 5 | force_change_on_next_login | required | - | force_change_on_next_login_required |
| 6 | admin_comment | required_if | record_comment_log = true | admin_comment_required |
| 7 | admin_comment | max | 1000 | admin_comment_too_long |

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Password hashing | Mật khẩu mới (tự sinh hay chỉ định) phải được mã hóa băm trước khi lưu vào cột `password_hash` của bảng `mst_platform_user`. |
| BR-002 | Temp password generation | Mật khẩu tự động phải đáp ứng độ mạnh tối thiểu (ví dụ: tối thiểu 12 ký tự, chứa cả chữ hoa, chữ thường, số và ký tự đặc biệt). |
| BR-003 | Send Email Notification | Nếu `send_notification_email` = true, hệ thống sẽ kích hoạt tiến trình gửi email chứa mật khẩu mới đến địa chỉ email của user. |
| BR-004 | Admin Comment Log | Nếu `record_comment_log` = true, lý do `admin_comment` phải được ghi nhận vào nhật ký vận hành hệ thống kèm theo ID của Admin thao tác. |
| BR-005 | Password Change Required | Cờ `pwd_change_required_flg` của người dùng mục tiêu bắt buộc được set về 1 (true) nếu `force_change_on_next_login` = true. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Super Admin | Có | Có | Có | Có | Không | Có |
| Platform Operator | Có | Không | Có | Không | Không | Không |
| Platform Security Admin | Có | Có | Có | Không | Không | Không |
| Tenant Admin/User | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Giải mã JWT Access Token từ Header Authorization.
2. Kiểm tra tính hợp lệ và thời hạn của token.
3. Lấy Platform Role ID của tài khoản Admin đang thao tác.
4. Kiểm tra quyền platform.user.platform_user.password_reset liên kết với Role.
5. Nếu không có quyền, từ chối yêu cầu và trả về lỗi 403 Forbidden.
```

---

# 10. Status Transition

Không áp dụng chuyển đổi trạng thái thực thể phức tạp cho API này.

---

# 11. Sequence / Processing Flow

```
1. Frontend gửi yêu cầu đặt lại mật khẩu (POST /api/v1/admin/users/password-reset).
2. Backend xác thực JWT token của quản trị viên Platform.
3. Backend kiểm tra quyền đặt lại mật khẩu.
4. Backend validate dữ liệu trong Request Body.
5. Backend kiểm tra sự tồn tại của user_id trong mst_platform_user.
6. Backend sinh mật khẩu tạm thời (nếu reset_method = 1) hoặc lấy mật khẩu chỉ định (nếu reset_method = 2).
7. Backend khởi tạo Transaction trên central_db.
8. Backend băm mật khẩu mới và cập nhật trường password_hash & pwd_change_required_flg trong mst_platform_user.
9. Backend chèn lý do admin_comment vào bảng log hoạt động (nếu record_comment_log = true).
10. Backend commit Transaction.
11. Backend kích hoạt gửi email thông báo mật khẩu mới cho user (nếu send_notification_email = true).
12. Backend trả Response thành công 200 OK kèm mật khẩu tạm thời vừa tạo.
```

---

# 12. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Wireframe | PA-USER-004 (image.png) |
| Screen Detail Design | [PA-USER-004.md](file:///Users/macbookpro/Thuan/work/carima_staffing/carima-staffing-doc/Document_hub/Screens/ADMIN/PA-USER-004/PA-USER-004.md) |
| ERD | [PLATFORM SCHEMA.md](file:///Users/macbookpro/Thuan/work/carima_staffing/carima-staffing-doc/Document_hub/ERD/PLATFORM%20SCHEMA.md) |
| Permission Matrix | Platform Role Matrix |
| Validation Rule | Validation Rule Document |
| NFR | NFR Document |

---

# 13. Open Issues

Không có.

---

# 14. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Not Started |
| 2 | Request body đã được định nghĩa | Not Started |
| 3 | Response body đã được định nghĩa | Not Started |
| 4 | Error response đã được định nghĩa | Not Started |
| 5 | Validation rule đã được định nghĩa | Not Started |
| 6 | Business rule đã được định nghĩa | Not Started |
| 7 | Permission check đã được định nghĩa | Not Started |
| 8 | DB mapping đã được định nghĩa | Not Started |
| 9 | Transaction scope đã được định nghĩa | Not Started |
| 10 | Audit log đã được định nghĩa | Not Started |
| 11 | Notification rule đã được định nghĩa | Not Started |
