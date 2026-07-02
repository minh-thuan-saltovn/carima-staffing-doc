## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Lock Platform User |
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
| API ID | PA-USER-005-API-02 |
| Tên API | Lock Platform User |
| Business Flow liên quan | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Screen ID liên quan | PA-USER-005 |
| Chức năng liên quan | Khóa/Mở khóa tài khoản |
| Mục đích API | Thực hiện thao tác khóa tài khoản quản trị viên Platform |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/admin/users/{id}/lock |
| Authentication | Có |
| Authorization | Có (platform.user.account_lock_unlock.lock) |
| Phạm vi Tenant | Platform Common (central_db) |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | VN Review |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để chủ động khóa tài khoản của một quản trị viên Platform cụ thể, cập nhật cờ `account_locked_flg` = 1, ghi log hoạt động lý do khóa của Admin, và gửi thông báo email đến tài khoản mục tiêu.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Admin chọn lý do và bấm nút "Khóa tài khoản" |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Client hiển thị modal và điền lý do khóa |
| Sau khi gọi API | Tài khoản bị khóa, log lý do được ghi nhận, gửi email thông báo |
| Thay đổi trạng thái liên quan | Cập nhật account_locked_flg = 1, status = 0 (hoặc giữ status tùy nghiệp vụ) |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/admin/users/{id}/lock
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token của Platform Admin |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ thông điệp lỗi trả về |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | number | Có | ID của người dùng cần khóa tài khoản | 1001 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "lock_reason": "Yêu cầu khóa tài khoản do nghi ngờ vi phạm an ninh",
  "send_notification_email": true
}
```

---

## 5.2 Định nghĩa field request

| Field | Column (DB) | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- | --- |
| lock_reason | - | string | Có | 1000 | Không được rỗng | Lý do thực hiện khóa tài khoản |
| send_notification_email | - | boolean | Có | - | - | Gửi mail thông báo cho người dùng |

---

# 6. Response Definition

### 6.1 Response Body Schema

```json
{
  "data": null
}
```

### 6.2 Định nghĩa field response thành công

Không áp dụng (trả về null khi thành công).

### 6.3 Handle response status

| HTTP Status | Error Code | Mô tả |
| --- | --- | --- |
| 200 OK | - | Khóa tài khoản thành công |
| 400 Bad Request | VAL001 | Tham số đầu vào không đúng định dạng hoặc thiếu lý do khóa |
| 401 Unauthorized | AUTH001 | Token không hợp lệ hoặc đã hết hạn |
| 403 Forbidden | AUTH002 | Không có quyền thực hiện hành động khóa tài khoản |
| 404 Not Found | USR001 | Không tìm thấy người dùng cần khóa |
| 500 Internal Server Error | SYS001 | Lỗi hệ thống |

---

## 7. Validation Rules

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | lock_reason | required | - | lock_reason_required |
| 2 | lock_reason | max | 1000 | lock_reason_too_long |

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Lock status update | Set cờ `account_locked_flg` = 1 và `locked_at` = thời điểm hiện tại trong bảng `mst_platform_user`. |
| BR-002 | Audit Logging | Bắt buộc ghi nhận log lý do khóa (`lock_reason`) cùng ID của Admin thao tác vào bảng log vận hành hệ thống. |
| BR-003 | Email sending | Nếu `send_notification_email` = true, hệ thống tự động gửi email thông báo tài khoản bị khóa cho user mục tiêu. |
| BR-004 | Admin check | Quản trị viên Platform không được phép tự khóa chính tài khoản của mình. |

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
2. Kiểm tra quyền platform.user.account_lock_unlock.lock liên kết với Role.
3. Nếu không có quyền hoặc tự khóa chính mình, trả về lỗi 403 Forbidden.
```

---

# 10. Status Transition

Không áp dụng chuyển đổi trạng thái thực thể phức tạp cho API này.

---

# 11. Sequence / Processing Flow

```
1. Frontend gửi yêu cầu khóa (POST /api/v1/admin/users/{id}/lock).
2. Backend xác thực JWT token và kiểm tra quyền.
3. Backend validate thông tin lock_reason.
4. Backend kiểm tra sự tồn tại của user_id trong mst_platform_user.
5. Backend kiểm tra logic tự khóa chính mình.
6. Backend khởi tạo Transaction trên central_db.
7. Backend cập nhật account_locked_flg = 1, locked_at = CURRENT_TIMESTAMP trong mst_platform_user.
8. Backend chèn log lý do lock_reason vào bảng log hoạt động.
9. Backend commit Transaction.
10. Backend gửi mail thông báo cho user (nếu send_notification_email = true).
11. Backend trả Response thành công 200 OK.
```

---

# 12. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Wireframe | PA-USER-005 (image.png) |
| Screen Detail Design | [PA-USER-005.md](file:///Users/macbookpro/Thuan/work/carima_staffing/carima-staffing-doc/Document_hub/Screens/ADMIN/PA-USER-005/PA-USER-005.md) |
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
