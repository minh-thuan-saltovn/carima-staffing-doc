## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Create Platform User |
| Diagram / Flow áp dụng | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Portal áp dụng | Platform Manager |
| Version | v0.1 |
| Ngày tạo | 2026/07/01 |
| Ngày cập nhật | 2026/07/01 |
| Người tạo | Phúc |
| Người review | Sang |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v0.1 | 2026/07/01 | Phúc | Tạo bản đầu tiên |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-USER-002-API-01 |
| Tên API | Create Platform User |
| Business Flow liên quan | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Screen ID liên quan | PA-USER-002 |
| Chức năng liên quan | Thêm mới một tài khoản quản trị viên Platform |
| Mục đích API | Tạo mới một tài khoản quản trị viên Platform và gửi email kích hoạt |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/admin/users |
| Authentication | Có |
| Authorization | Có (platform.user.platform_user.create) |
| Phạm vi Tenant | Platform Common (central_db) |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | In progress |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để tạo mới tài khoản quản trị viên Platform vào bảng `mst_platform_user`, kiểm tra trùng lặp thông tin và kích hoạt quy trình gửi email chứa liên kết thiết lập mật khẩu lần đầu.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Admin bấm nút “Register” |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Client validate form cơ bản |
| Sau khi gọi API | Tài khoản lưu thành công, hệ thống gửi email mời |
| Thay đổi trạng thái liên quan | Tạo mới ở trạng thái status = 1 (hoặc 0 tùy cấu hình), pwd_change_required_flg = 1 |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/admin/users
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
  "full_name": "山田太郎",
  "full_name_kana": "ヤマダタロウ",
  "login_id": "yamada.t",
  "email": "yamada.t@moto.jp",
  "phone_number": "03-1234-5678",
  "mail_language": "ja",
  "company_id": "MOTO_STF_001",
  "department_id": "DEPT_101",
  "position": "マネージャー",
  "office_id": "OFFICE_201",
  "supervisor_id": "USER_1005",
  "start_date": "2026-07-01",
  "role_ids": [1, 2],
  "reference_scope": 1,
  "approval_permissions": [1, 2],
  "status": 2,
  "initial_password_method": 1,
  "specified_password": null,
  "pwd_change_required_flg": 1,
  "require_2fa_flg": 0,
  "send_invitation_email": true,
  "invitation_email_settings": {
    "subject": "【Carima】プラットフォームアカウント招待のご案内",
    "body": "招待メール本文を入力してください",
    "send_timing": 1
  }
}
```

---

## 5.2 Định nghĩa field request

| Field | Column (DB) | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- | --- |
| full_name | - | string | Có | 100 | Không được rỗng | Họ và tên |
| full_name_kana | - | string | Không | 100 | Katakana | Họ tên Kana |
| login_id | mst_platform_user.login_id | string | Có | 100 | Alphanumeric, dot, underscore. Unique. | Mã đăng nhập |
| email | mst_platform_user.email | string | Có | 255 | Format email. Unique. | Địa chỉ email |
| phone_number | - | string | Không | 20 | Định dạng số điện thoại | Số điện thoại |
| mail_language | - | string | Có | 5 |ja, en | Ngôn ngữ |
| company_id | - | string | Không | 50 | Tồn tại trong tenant | ID công ty/tenant |
| department_id | - | string | Không | 50 | Tồn tại trong master | ID bộ phận |
| position | - | string | Không | 100 | - | Chức vụ |
| office_id | - | string | Không | 50 | Tồn tại trong master | ID chi nhánh phụ trách |
| supervisor_id | - | string | Không | 50 | Tồn tại trong master | ID cấp trên trực tiếp |
| start_date | - | string | Không | - | YYYY-MM-DD | Ngày bắt đầu sử dụng |
| role_ids | - | array[bigint] | Có | - | Phải tồn tại vai trò | Danh sách vai trò |
| reference_scope | - | number | Có | - | 1, 2, 3, 4 | Phạm vi tham chiếu |
| approval_permissions | - | array[number] | Không | - | - | Danh sách quyền phê duyệt |
| status | mst_platform_user.status | number | Có | - | 0, 1, 2 | Trạng thái (2: 招待中) |
| initial_password_method | - | number | Có | - | 1: Tự động, 2: Chỉ định | Phương thức tạo mật khẩu ban đầu |
| specified_password | - | string | Không | 100 | Bắt buộc nếu method = 2 | Mật khẩu chỉ định |
| pwd_change_required_flg | mst_platform_user.pwd_change_required_flg | number | Có | - | 0 hoặc 1 | Bắt buộc đổi mật khẩu lần đầu |
| require_2fa_flg | - | number | Có | - | 0 hoặc 1 | Yêu cầu xác thực 2 yếu tố |
| send_invitation_email | - | boolean | Có | - | - | Gửi mail mời |
| invitation_email_settings.subject | - | string | Không | 200 | Bắt buộc nếu gửi mail | Tiêu đề email mời |
| invitation_email_settings.body | - | string | Không | 2000 | Bắt buộc nếu gửi mail | Nội dung email mời |
| invitation_email_settings.send_timing | - | number | Không | - | 1: Ngay, 2: Nháp | Thời điểm gửi |

---

# 6. Response Definition

### 6.1 Response Body Schema

```json
{
  "data": {
    "id": 1002,
    "login_id": "yamada.t",
    "full_name": "山田太郎",
    "email": "yamada.t@moto.jp",
    "status": 2,
    "created_at": "2026-07-01T14:50:00Z"
  }
}
```

### 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.id | number | ID tự tăng của tài khoản vừa tạo trong hệ thống |
| data.login_id | string | Mã đăng nhập |
| data.full_name | string | Họ và tên |
| data.email | string | Địa chỉ email |
| data.status | number | Trạng thái hoạt động |
| data.created_at | string | Thời gian tạo tài khoản |

### 6.3 Handle response status

| HTTP Status | Error Code | Mô tả |
| --- | --- | --- |
| 201 Created | - | Tạo tài khoản thành công |
| 400 Bad Request | VAL001 | Tham số đầu vào không đúng định dạng hoặc thiếu trường bắt buộc |
| 401 Unauthorized | AUTH001 | Token không hợp lệ hoặc đã hết hạn |
| 403 Forbidden | AUTH002 | Không có quyền thực hiện hành động tạo mới |
| 409 Conflict | DUP001 | login_id hoặc email đã tồn tại trong hệ thống |
| 500 Internal Server Error | SYS001 | Lỗi hệ thống |

---

## 7. Validation Rules

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | login_id | required | - | login_id_required |
| 2 | login_id | format | alphanumeric, dots, underscores | login_id_invalid_format |
| 3 | login_id | max | 100 | login_id_too_long |
| 4 | full_name | required | - | full_name_required |
| 5 | full_name | max | 100 | full_name_too_long |
| 6 | email | required | - | email_required |
| 7 | email | email | - | email_invalid_format |
| 8 | email | max | 255 | email_too_long |
| 9 | role_ids | required | - | role_ids_required |
| 10 | status | required | - | status_required |
| 11 | mail_language | required | - | mail_language_required |
| 12 | reference_scope | required | - | reference_scope_required |
| 13 | initial_password_method | required | - | initial_password_method_required |
| 14 | specified_password | required_if | initial_password_method = 2 | specified_password_required |
| 15 | invitation_email_settings.subject | required_if | send_invitation_email = true | email_subject_required |
| 16 | invitation_email_settings.body | required_if | send_invitation_email = true | email_body_required |

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Platform isolation | Quản trị viên Platform được lưu trong DB chung (`central_db`), không thuộc sở hữu của bất kỳ Tenant nào. |
| BR-002 | Permission check | Chỉ người dùng có quyền `platform.user.platform_user.create` mới được phép gọi API này. |
| BR-003 | Unique check | Phải kiểm tra sự tồn tại duy nhất của `login_id` và `email` trước khi thêm vào DB. |
| BR-004 | Audit log | Ghi lại hành động tạo tài khoản mới vào nhật ký vận hành hệ thống. |
| BR-005 | Invitation Link | Hệ thống tạo Token kích hoạt và gửi email chứa đường dẫn thiết lập mật khẩu lần đầu cho người dùng. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Super Admin | Có | Có | Có | Có | Không | Có |
| Platform Operator | Có | Có | Có | Không | Không | Không |
| Platform Security Admin | Có | Không | Không | Không | Không | Không |
| Tenant Admin/User | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Giải mã JWT Access Token từ Header Authorization.
2. Kiểm tra tính hợp lệ và thời hạn của token.
3. Lấy Platform Role ID của tài khoản đang thao tác.
4. Kiểm tra quyền platform.user.platform_user.create liên kết với Role.
5. Nếu không có quyền, từ chối yêu cầu và trả về lỗi 403 Forbidden.
```

---

# 10. Status Transition

Không áp dụng chuyển đổi trạng thái thực thể phức tạp cho API này.

---

# 11. Sequence / Processing Flow

```
1. Frontend gửi yêu cầu đăng ký (POST /api/v1/admin/users).
2. Backend xác thực JWT token của quản trị viên Platform.
3. Backend kiểm tra quyền tạo người dùng.
4. Backend validate dữ liệu trong Request Body.
5. Backend kiểm tra trùng lặp login_id và email trong bảng mst_platform_user.
6. Backend khởi tạo Transaction trên central_db.
7. Backend chèn bản ghi mới vào mst_platform_user.
8. Backend tạo Token kích hoạt và ghi nhận Audit Log.
9. Backend commit Transaction.
10. Backend gửi tác vụ email mời vào Queue.
11. Backend trả Response thành công 201 Created kèm dữ liệu tài khoản.
```

---

# 12. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Wireframe | PA-USER-002 (image-1.png) |
| Screen Detail Design | [PA-USER-002.md](file:///Users/macbookpro/Thuan/work/carima_staffing/carima-staffing-doc/Document_hub/Screens/ADMIN/PA-USER-002/PA-USER-002.md) |
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
