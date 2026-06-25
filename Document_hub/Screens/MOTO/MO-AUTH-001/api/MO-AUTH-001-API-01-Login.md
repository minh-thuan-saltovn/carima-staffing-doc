# API Detail Design

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design |
| API ID | MO-AUTH-001-API-01-Login |
| API Name | Login |
| Business Flow áp dụng | BF-001 Login Flow |
| Portal áp dụng | MOTO Portal |
| Screen ID liên quan | MO-AUTH-001 |
| Version | v0.1 |
| Ngày tạo | YYYY/MM/DD |
| Ngày cập nhật | YYYY/MM/DD |
| Người tạo | TBD |
| Người review | TBD |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v0.1 | YYYY/MM/DD | TBD | Tạo bản đầu tiên |

---

## 3. Tổng quan API

### 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | MO-AUTH-001-API-01-Login |
| Tên API | Login |
| Tên tiếng Nhật | ログイン |
| Business Flow liên quan | BF-001 Login Flow |
| Screen ID liên quan | MO-AUTH-001 |
| Chức năng liên quan | Đăng nhập MOTO Portal |
| Mục đích API | Xác thực người dùng MOTO Portal và phát hành Access Token |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/moto/auth/login |
| Authentication | Không yêu cầu |
| Authorization | Không yêu cầu trước login |
| Phạm vi Tenant | Tenant được xác định theo domain/subdomain hoặc context request |
| Có Transaction không | Có |
| Xử lý file | Không |
| Response format | JSON |
| Trạng thái | Draft |

### 3.2 Mô tả API

API này dùng để xác thực User ID và Password của người dùng MOTO Portal.

Người dùng nhập thông tin đăng nhập tại màn hình Login. Backend xác định Tenant hiện tại, tìm user trong tenant DB, kiểm tra mật khẩu, trạng thái tài khoản và chính sách bảo mật liên quan.

Nếu đăng nhập thành công, hệ thống phát hành Access Token, Refresh Token, ghi Login History, ghi Audit Log và trả về thông tin user hiện tại.

Nếu đăng nhập thất bại, hệ thống trả về lỗi tương ứng và ghi nhận Audit Log / Login Failed History theo chính sách bảo mật.

### 3.3 Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User bấm nút 「ログイン」 |
| Actor | MOTO User |
| Trước khi gọi API | User đang ở màn hình Login |
| Sau khi gọi API thành công | User được chuyển tới Dashboard |
| Sau khi gọi API thất bại | Hiển thị message lỗi trên màn hình Login |
| Thay đổi trạng thái liên quan | Guest → Authenticated |

---

## 4. Định nghĩa Endpoint

### 4.1 Endpoint

POST /api/v1/moto/auth/login

### 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept | Có | application/json | Loại dữ liệu response mong muốn |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |
| User-Agent | Không | Mozilla/5.0 | Thông tin trình duyệt / thiết bị |
| X-Forwarded-For | Không | 192.168.1.1 | IP thật nếu đi qua proxy/load balancer |

### 4.3 Path Parameters

Không có

### 4.4 Query Parameters

Không có

---

## 5. Request Body

### 5.1 Request Body JSON

```json
{
  "user_id": "moto001",
  "password": "password123",
  "remember_me": true
}
```

### 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| user_id | string | Có | 100 | Required, trim | User ID đăng nhập |
| password | string | Có | 255 | Required | Mật khẩu đăng nhập |
| remember_me | boolean | Không | - | true / false | Ghi nhớ đăng nhập |

### 5.3 Ghi chú request

| Hạng mục | Nội dung |
| --- | --- |
| Tenant Code có truyền trong body không | Không |
| Cách xác định Tenant | Domain/subdomain hoặc request context |
| Có truyền email không | Không |
| Có truyền company_id không | Không |
| Có truyền role không | Không |
| Có truyền token không | Không |

---

## 6. Response Definition

### 6.1 Success Response

HTTP Status: 200 OK

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxxx.yyyyy",
    "refresh_token": "refresh_token_sample",
    "token_type": "Bearer",
    "expires_in": 3600,
    "expires_at": "2026-01-01T10:00:00+09:00",
    "user": {
      "user_id": "moto001",
      "company_id": "MOTO001",
      "last_name_ja": "山田",
      "first_name_ja": "太郎",
      "email": "yamada@example.com",
      "role_id": "ROLE_MOTO_ADMIN",
      "role_name": "MOTO Admin"
    },
    "tenant": {
      "tenant_id": "tenant_moto_001",
      "tenant_code": "MOTO001",
      "tenant_name": "MOTO Company"
    },
    "permissions": [
      "DASHBOARD_VIEW",
      "USER_VIEW",
      "USER_UPDATE"
    ]
  },
  "message": "ログインに成功しました"
}
```

### 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| success | boolean | Kết quả xử lý |
| data.access_token | string | Access Token dùng cho API sau login |
| data.refresh_token | string | Refresh Token dùng để cấp lại Access Token |
| data.token_type | string | Loại token, mặc định Bearer |
| data.expires_in | integer | Thời gian hết hạn Access Token, đơn vị giây |
| data.expires_at | string | Thời điểm hết hạn Access Token |
| data.user.user_id | string | User ID |
| data.user.company_id | string | Company ID của user |
| data.user.last_name_ja | string | Họ tiếng Nhật |
| data.user.first_name_ja | string | Tên tiếng Nhật |
| data.user.email | string | Email user |
| data.user.role_id | string | Role ID |
| data.user.role_name | string | Tên role |
| data.tenant.tenant_id | string | Tenant ID |
| data.tenant.tenant_code | string | Tenant Code |
| data.tenant.tenant_name | string | Tên Tenant |
| data.permissions | array[string] | Danh sách quyền của user |
| message | string | Message trả về |

### 6.3 Error Response - Validation Error

### 6.4 Error Response - Invalid Credentials

HTTP Status: 401 Unauthorized

### 6.5 Error Response - Account Locked

HTTP Status: 403 Forbidden

### 6.6 Error Response - Account Inactive

### 6.7 Error Response - Tenant Not Found

HTTP Status: 404 Not Found

### 6.8 Error Response - System Error

HTTP Status: 500 Internal Server Error

### 6.9 Error Response - Bad Request

HTTP Status: 400 Bad Request

### 6.10 Error Response - Method Not Allowed

HTTP Status: 405 Method Not Allowed

### 6.11 Error Response - Too Many Requests

HTTP Status: 429 Too Many Requests

### 6.12 Error Response - Service Unavailable

HTTP Status: 503 Service Unavailable

### 6.13 Error Response - Gateway Timeout

HTTP Status: 504 Gateway Timeout

Untitled

## 7. Validation Rules

[Message list](https://app.notion.com/p/sokucom/Message-list-374f02c407dd8037808eea01e93be8aa?source=copy_link)

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | user_id | required | - | required |
| 2 | user_id | max | 100 | max |
| 3 | password | required | - | required |
| 4 | password | max | 255 | max |
| 5 | remember_me | boolean | - | boolean |

### 7.1 Validation xử lý trước Business Rule

| Step | Nội dung |
| --- | --- |
| 1 | Kiểm tra Content-Type là application/json |
| 2 | Parse JSON request body |
| 3 | Validate required field |
| 4 | Validate type / max length |
| 5 | Trim user_id |
| 6 | Chuyển sang xử lý Business Rule |

## 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant Resolution | Backend xác định Tenant dựa trên domain/subdomain hoặc request context |
| BR-002 | Tenant Active Check | Chỉ Tenant đang Active mới cho phép login |
| BR-003 | Tenant Isolation | User chỉ được tìm trong DB/schema của Tenant hiện tại |
| BR-004 | User Existence Check | User ID phải tồn tại trong tenant_db.mst_moto_user |
| BR-005 | Password Verification | Password request phải khớp với password_hash đã lưu |
| BR-006 | User Status Check | Chỉ user có status Active mới được login |
| BR-007 | Account Lock Check | User bị lock không được login |
| BR-008 | Failed Login Count | Nếu login fail, tăng số lần login thất bại theo policy |
| BR-009 | Account Lock Policy | Nếu vượt quá số lần sai mật khẩu cho phép, khóa tài khoản |
| BR-010 | Token Issue | Login thành công sẽ phát hành Access Token và Refresh Token |
| BR-011 | Token Expiration | Nếu remember_me = false thì token ngắn hạn, nếu true thì token dài hơn theo policy |
| BR-012 | Login History | Ghi lịch sử login thành công/thất bại |
| BR-013 | Audit Log | Ghi Audit Log cho login success/fail/locked |
| BR-014 | Sensitive Information | Không trả password_hash, salt, failed_count, internal token secret ra response |
| BR-015 | Permission Scope | Response trả danh sách permission theo role của user |

---

## 9. Permission / Authorization

### 9.1 Authentication

| Hạng mục | Nội dung |
| --- | --- |
| API yêu cầu Access Token không | Không |
| Lý do | Đây là API dùng để đăng nhập |
| User chưa login có được gọi không | Có |
| User đã login có được gọi không | Có, nhưng token cũ sẽ xử lý theo policy hệ thống |

### 9.2 Authorization

| Role | Cho phép login MOTO Portal | Ghi chú |
| --- | --- | --- |
| MOTO Admin | Có | Nếu thuộc Tenant hiện tại |
| MOTO User | Có | Nếu thuộc Tenant hiện tại |
| SAKI Admin | Không | Không thuộc MOTO Portal |
| SAKI User | Không | Không thuộc MOTO Portal |
| Platform Admin | Không | Dùng Platform Admin Portal riêng |
| Staff | Không | Không thuộc scope API này |

### 9.3 Logic kiểm tra quyền sau khi xác thực

| Step | Nội dung |
| --- | --- |
| 1 | Xác định Tenant hiện tại |
| 2 | Tìm user trong tenant_db.mst_moto_user |
| 3 | Kiểm tra user thuộc MOTO Portal |
| 4 | Kiểm tra trạng thái user |
| 5 | Kiểm tra role của user |
| 6 | Load permission theo role |
| 7 | Trả permissions trong response |

---

## 10. Data Mapping

### 10.1 Tenant Resolution Mapping

| Source | DB | Table | Column | Mô tả |
| --- | --- | --- | --- | --- |
| Request domain/subdomain | central_db | mst_tenant | tenant_id | Xác định Tenant ID |
| Request domain/subdomain | central_db | mst_tenant | tenant_code | Xác định Tenant Code |
| Request domain/subdomain | central_db | mst_tenant | tenant_name | Tên Tenant |
| Request domain/subdomain | central_db | mst_tenant | db_name / schema_name | Xác định tenant DB/schema |
| Request domain/subdomain | central_db | mst_tenant | status | Kiểm tra Tenant đang active |

### 10.2 Request To DB Mapping

| Request Field | DB | Table | Column | Process | Mô tả |
| --- | --- | --- | --- | --- | --- |
| user_id | tenant_db | mst_moto_user | user_id | WHERE | Tìm user đăng nhập |
| password | tenant_db | mst_moto_user | password_hash | Verify only | So sánh password với hash, không lưu plain text |
| remember_me | tenant_db | TBD: access_token | expires_at | Calculate | Tính thời gian hết hạn token |
| Header.User-Agent | tenant_db | TBD: login_history | user_agent | Insert | Lưu user agent |
| Client IP | tenant_db | TBD: login_history | ip_address | Insert | Lưu IP đăng nhập |
| Login Result | tenant_db | TBD: login_history | login_result | Insert | success / failed / locked |

### 10.3 DB To Response Mapping

| DB | Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | mst_moto_user | user_id | data.user.user_id | User ID |
| tenant_db | mst_moto_user | company_id | data.user.company_id | Company ID |
| tenant_db | mst_moto_user | last_name_ja | data.user.last_name_ja | Họ tiếng Nhật |
| tenant_db | mst_moto_user | first_name_ja | data.user.first_name_ja | Tên tiếng Nhật |
| tenant_db | mst_moto_user | email | data.user.email | Email |
| tenant_db | mst_moto_user | role_id | data.user.role_id | Role ID |
| tenant_db | TBD: mst_role | role_name | data.user.role_name | Role Name |
| central_db | mst_tenant | tenant_id | data.tenant.tenant_id | Tenant ID |
| central_db | mst_tenant | tenant_code | data.tenant.tenant_code | Tenant Code |
| central_db | mst_tenant | tenant_name | data.tenant.tenant_name | Tenant Name |
| tenant_db | TBD: access_token | token | data.access_token | Access Token |
| tenant_db | TBD: refresh_token | token | data.refresh_token | Refresh Token |
| tenant_db | TBD: access_token | expires_in | data.expires_in | Token lifetime |
| tenant_db | TBD: access_token | expires_at | data.expires_at | Token expired datetime |
| tenant_db | TBD: role_permission | permission_code | data.permissions[] | Permission list |

### 10.4 Insert Access Token Mapping

| DB | Table | Column | Value | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: access_token | token_id | Generated UUID | Token ID |
| tenant_db | TBD: access_token | user_id | mst_moto_user.user_id | User ID |
| tenant_db | TBD: access_token | access_token_hash | Hash(access_token) | Không lưu raw token nếu có thể |
| tenant_db | TBD: access_token | refresh_token_hash | Hash(refresh_token) | Không lưu raw refresh token nếu có thể |
| tenant_db | TBD: access_token | token_type | Bearer | Token type |
| tenant_db | TBD: access_token | expires_at | Calculated datetime | Hạn Access Token |
| tenant_db | TBD: access_token | refresh_expires_at | Calculated datetime | Hạn Refresh Token |
| tenant_db | TBD: access_token | revoked_at | null | Token chưa bị revoke |
| tenant_db | TBD: access_token | created_at | Current datetime | Ngày tạo |
| tenant_db | TBD: access_token | created_by | system | Người tạo |

### 10.5 Insert Login History Mapping

| DB | Table | Column | Value | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: login_history | login_history_id | Generated UUID | ID lịch sử login |
| tenant_db | TBD: login_history | user_id | mst_moto_user.user_id | User ID |
| tenant_db | TBD: login_history | login_at | Current datetime | Thời điểm login |
| tenant_db | TBD: login_history | logout_at | null | Chưa logout |
| tenant_db | TBD: login_history | ip_address | Client IP | IP đăng nhập |
| tenant_db | TBD: login_history | user_agent | Header.User-Agent | Trình duyệt/thiết bị |
| tenant_db | TBD: login_history | login_result | success / failed / locked | Kết quả login |
| tenant_db | TBD: login_history | failure_reason | null / invalid_credentials / locked | Lý do fail |
| tenant_db | TBD: login_history | created_at | Current datetime | Ngày tạo |

### 10.6 Update User Mapping

| DB | Table | Column | Timing | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | mst_moto_user | last_login_at | Login success | Cập nhật lần login cuối |
| tenant_db | mst_moto_user | failed_login_count | Login failed | Tăng số lần login thất bại |
| tenant_db | mst_moto_user | failed_login_count | Login success | Reset về 0 |
| tenant_db | mst_moto_user | locked_at | Khi vượt quá policy | Set thời điểm lock |
| tenant_db | mst_moto_user | updated_at | Khi update user | Ngày cập nhật |

### 10.7 Index cần thiết

| DB | Table | Index | Column | Mục đích |
| --- | --- | --- | --- | --- |
| central_db | mst_tenant | idx_mst_tenant_domain | domain / subdomain | Tìm Tenant theo domain |
| central_db | mst_tenant | idx_mst_tenant_status | status | Kiểm tra Tenant active |
| tenant_db | mst_moto_user | idx_mst_moto_user_user_id | user_id | Tìm user login |
| tenant_db | mst_moto_user | idx_mst_moto_user_status | status | Kiểm tra user active |
| tenant_db | TBD: access_token | idx_access_token_user_id | user_id | Tìm token theo user |
| tenant_db | TBD: login_history | idx_login_history_user_id | user_id | Tra cứu lịch sử login |
| tenant_db | TBD: login_history | idx_login_history_login_at | login_at | Sort lịch sử login |

---

## 11. Database Transaction

### 11.1 Phạm vi transaction khi Login Success

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request | Ngoài transaction |
| 2 | Resolve Tenant | Ngoài transaction |
| 3 | Check Tenant status | Ngoài transaction |
| 4 | Find user | Ngoài transaction |
| 5 | Verify password | Ngoài transaction |
| 6 | Check user status / lock | Ngoài transaction |
| 7 | Begin transaction | Trong transaction |
| 8 | Create Access Token / Refresh Token | Trong transaction |
| 9 | Insert Login History success | Trong transaction |
| 10 | Update mst_moto_user.last_login_at | Trong transaction |
| 11 | Reset failed_login_count | Trong transaction |
| 12 | Insert Audit Log success | Trong transaction |
| 13 | Commit transaction | Trong transaction |

### 11.2 Phạm vi transaction khi Login Failed

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request | Ngoài transaction |
| 2 | Resolve Tenant | Ngoài transaction |
| 3 | Find user | Ngoài transaction |
| 4 | Verify password failed | Ngoài transaction |
| 5 | Begin transaction | Trong transaction |
| 6 | Update failed_login_count | Trong transaction |
| 7 | Lock account nếu vượt quá policy | Trong transaction |
| 8 | Insert Login History failed | Trong transaction |
| 9 | Insert Audit Log failed | Trong transaction |
| 10 | Commit transaction | Trong transaction |

### 11.3 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Insert Access Token thất bại | Rollback |
| 2 | Insert Login History thất bại | Rollback |
| 3 | Update mst_moto_user thất bại | Rollback |
| 4 | Insert Audit Log thất bại | Rollback |
| 5 | Lock account update thất bại | Rollback |

---

## 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Guest | Login success | Authenticated | User/password hợp lệ, account active |
| Guest | Login failed | Guest | Sai User ID hoặc Password |
| Guest | Login failed nhiều lần | Locked | Vượt quá số lần sai mật khẩu cho phép |
| Locked | Login | Locked | Account đang bị khóa |
| Inactive | Login | Inactive | Account không còn hoạt động |
| Authenticated | Logout | Guest | User logout hoặc token revoked |
| Authenticated | Token expired | Guest | Access Token hết hạn |

---

## 13. Sequence / Processing Flow

### 13.1 Login Success Flow

1. Frontend gọi API Login
2. Backend validate request body
3. Backend xác định Tenant từ domain/subdomain hoặc request context
4. Backend kiểm tra Tenant tồn tại
5. Backend kiểm tra Tenant active
6. Backend kết nối tenant DB/schema tương ứng
7. Backend tìm user theo user_id trong mst_moto_user
8. Backend kiểm tra user tồn tại
9. Backend kiểm tra trạng thái user
10. Backend kiểm tra account lock
11. Backend verify password
12. Backend load role / permission của user
13. Backend bắt đầu transaction
14. Backend tạo Access Token
15. Backend tạo Refresh Token
16. Backend insert Login History success
17. Backend update last_login_at
18. Backend reset failed_login_count
19. Backend insert Audit Log success
20. Backend commit transaction
21. Backend trả Success Response

### 13.2 Login Failed Flow

1. Frontend gọi API Login
2. Backend validate request body
3. Backend xác định Tenant
4. Backend tìm user theo user_id
5. Backend verify password thất bại hoặc user không hợp lệ
6. Backend bắt đầu transaction
7. Backend tăng failed_login_count nếu xác định được user
8. Backend lock account nếu vượt quá policy
9. Backend insert Login History failed
10. Backend insert Audit Log failed
11. Backend commit transaction
12. Backend trả Error Response

---

## 14. Notification

| Hạng mục | Nội dung |
| --- | --- |
| Có cần notification không | Không |
| Thời điểm gửi | - |
| Đối tượng nhận | - |
| Kênh gửi | - |
| Template ID | - |

### 14.1 Ghi chú

| No. | Nội dung |
| --- | --- |
| 1 | Login thành công không gửi notification |
| 2 | Login thất bại không gửi notification |
| 3 | Account bị lock có thể notify admin nếu NFR/Security Policy yêu cầu |
| 4 | Nếu cần notify account locked, tạo API/Job riêng hoặc event riêng |

---

## 15. Audit Log

### 15.1 Audit Log Requirement

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | LOGIN |
| Bảng target | mst_moto_user |
| ID record target | user_id |
| Thời điểm ghi log | Login success, login failed, account locked |
| Chính sách lưu trữ | Theo NFR |

### 15.2 Audit Log Mapping

| DB | Table | Column | Value | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: audit_log | audit_log_id | Generated UUID | Audit Log ID |
| tenant_db | TBD: audit_log | action_type | LOGIN | Loại thao tác |
| tenant_db | TBD: audit_log | target_table | mst_moto_user | Bảng target |
| tenant_db | TBD: audit_log | target_id | user_id | User ID |
| tenant_db | TBD: audit_log | actor_user_id | user_id / null | User thực hiện |
| tenant_db | TBD: audit_log | ip_address | Client IP | IP |
| tenant_db | TBD: audit_log | user_agent | Header.User-Agent | Trình duyệt/thiết bị |
| tenant_db | TBD: audit_log | result | success / failed / locked | Kết quả |
| tenant_db | TBD: audit_log | detail | JSON object | Nội dung chi tiết |
| tenant_db | TBD: audit_log | created_at | Current datetime | Ngày tạo |

### 15.3 Audit Log Detail JSON - Success

```json
{
  "login_result": "success",
  "user_id": "moto001",
  "tenant_id": "tenant_moto_001",
  "ip_address": "192.168.1.1"
}
```

### 15.4 Audit Log Detail JSON - Failed

```json
{
  "login_result": "failed",
  "user_id": "moto001",
  "tenant_id": "tenant_moto_001",
  "failure_reason": "invalid_credentials",
  "ip_address": "192.168.1.1"
}
```

### 15.5 Audit Log Detail JSON - Locked

```json
{
  "login_result": "locked",
  "user_id": "moto001",
  "tenant_id": "tenant_moto_001",
  "failure_reason": "account_locked",
  "ip_address": "192.168.1.1"
}
```

## 16. Xử lý file

### 16.1 Upload

| Hạng mục | Nội dung |
| --- | --- |
| Có upload file không | Không |
| Max file size | - |
| Định dạng cho phép | - |
| Nơi lưu trữ | - |
| Virus Scan | - |
| Quy tắc đặt tên file | - |
| Bảng lưu metadata | - |

### 16.2 Download

| Hạng mục | Nội dung |
| --- | --- |
| Có download file không | Không |
| Permission Check | - |
| Phương thức download | - |
| Thời gian hết hạn | - |
| Audit Log | - |

---

## 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication | API Login không yêu cầu Access Token |
| 2 | Authorization | Chỉ xác định quyền sau khi login thành công |
| 3 | Tenant Isolation | Không cho phép Cross Tenant |
| 4 | Tenant Resolution | Tenant phải được xác định từ domain/subdomain hoặc request context đáng tin cậy |
| 5 | Password Handling | Không lưu password plain text |
| 6 | Password Verification | Verify bằng password_hash |
| 7 | Sensitive Data Masking | Không ghi password, token raw, password_hash vào log |
| 8 | Token Storage | DB nên lưu token dạng hash, không lưu raw token nếu có thể |
| 9 | Token Expiration | Access Token phải có expires_at |
| 10 | Refresh Token | Refresh Token phải có hạn và có thể revoke |
| 11 | Account Lock | Áp dụng lock policy khi login fail nhiều lần |
| 12 | Rate Limit | Áp dụng rate limit theo IP và user_id |
| 13 | SQL Injection Prevention | Sử dụng parameterized query / ORM binding |
| 14 | Brute Force Prevention | Không trả message phân biệt user không tồn tại và password sai |
| 15 | Audit Log | Bắt buộc ghi log login success/fail/locked |
| 16 | HTTPS | Chỉ cho phép gọi API qua HTTPS |
| 17 | CORS | Chỉ cho phép origin hợp lệ |
| 18 | Session Fixation | Tạo token mới sau khi login thành công |

### 17.1 Security Response Policy

| Case | Message Policy |
| --- | --- |
| User không tồn tại | Trả INVALID_CREDENTIALS |
| Password sai | Trả INVALID_CREDENTIALS |
| Account locked | Trả ACCOUNT_LOCKED |
| Tenant không tồn tại | Trả TENANT_NOT_FOUND hoặc generic error theo policy |
| Tenant inactive | Trả TENANT_INACTIVE hoặc generic error theo policy |

---

## 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | ≤ 2 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Không |
| Index cần thiết | Có |
| Cache cần thiết | Không bắt buộc |
| Bulk Processing | Không |
| Async Processing | Không |
| Transaction scope | Ngắn, chỉ bao gồm insert/update cần thiết |
| Password hash cost | Theo Security Policy, cần cân bằng bảo mật và performance |
| Rate limit | Có |

### 18.1 Performance Notes

| No. | Nội dung |
| --- | --- |
| 1 | Cần index user_id trong mst_moto_user |
| 2 | Cần index domain/subdomain trong mst_tenant |
| 3 | Không nên load dữ liệu user không cần thiết |
| 4 | Không nên ghi log đồng bộ quá nặng trong transaction |
| 5 | Nếu Audit Log lớn, có thể tách async theo NFR, nhưng Login History nên ghi đồng bộ |

---

## 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 Login Flow |
| Wireframe | MO-AUTH-001 |
| Screen Detail Design | MO-AUTH-001 |
| ERD | Carima Data Dictionary |
| Data Dictionary | central_db.mst_tenant, tenant_db.mst_moto_user |
| Permission Matrix | TBD |
| Validation Rule | TBD |
| Security Policy | TBD |
| NFR | TBD |

---

## 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| 1 | Chưa xác nhận cách xác định Tenant là domain, subdomain hay tenant context | High | TBD | TBD | Open |
| 2 | Chưa xác định bảng Access Token chính thức trong Data Dictionary | High | TBD | TBD | Open |
| 3 | Chưa xác định bảng Refresh Token chính thức trong Data Dictionary | High | TBD | TBD | Open |
| 4 | Chưa xác định bảng Login History chính thức trong Data Dictionary | High | TBD | TBD | Open |
| 5 | Chưa xác định bảng Audit Log chính thức trong Data Dictionary | Medium | TBD | TBD | Open |
| 6 | Chưa xác định column password_hash chính thức của mst_moto_user | High | TBD | TBD | Open |
| 7 | Chưa xác định column failed_login_count / locked_at chính thức | Medium | TBD | TBD | Open |
| 8 | Chưa xác định Account Lock Policy: số lần fail, thời gian khóa, unlock method | Medium | TBD | TBD | Open |
| 9 | Chưa xác định thời gian hết hạn Access Token / Refresh Token | Medium | TBD | TBD | Open |
| 10 | Chưa xác định role / permission table chính thức | Medium | TBD | TBD | Open |

---

## 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | API ID đã được định nghĩa | Done |
| 2 | Endpoint đã được định nghĩa | Done |
| 3 | Method đã được định nghĩa | Done |
| 4 | Request header đã được định nghĩa | Done |
| 5 | Request body đã được định nghĩa bằng JSON chuẩn | Done |
| 6 | Success response đã được định nghĩa bằng JSON chuẩn | Done |
| 7 | Error response đã được định nghĩa bằng JSON chuẩn | Done |
| 8 | Validation rule đã được định nghĩa | Done |
| 9 | Business rule đã được định nghĩa | Done |
| 10 | Permission / Authorization đã được định nghĩa | Done |
| 11 | Tenant isolation rule đã được định nghĩa | Done |
| 12 | Data mapping đã được định nghĩa | Done |
| 13 | DB mapping cho request đã được định nghĩa | Done |
| 14 | DB mapping cho response đã được định nghĩa | Done |
| 15 | DB mapping cho token đã được định nghĩa | Done |
| 16 | DB mapping cho login history đã được định nghĩa | Done |
| 17 | Transaction scope đã được định nghĩa | Done |
| 18 | Rollback condition đã được định nghĩa | Done |
| 19 | Status transition đã được định nghĩa | Done |
| 20 | Sequence flow đã được định nghĩa | Done |
| 21 | Notification rule đã được định nghĩa | Done |
| 22 | Audit Log đã được định nghĩa | Done |
| 23 | File handling đã được định nghĩa | Done |
| 24 | Security considerations đã được định nghĩa | Done |
| 25 | Performance considerations đã được định nghĩa | Done |
| 26 | Tài liệu liên quan đã được định nghĩa | Done |
| 27 | Open issues đã được định nghĩa | Done |
| 28 | Các bảng chưa có trong Data Dictionary đã được đánh dấu TBD | Done |