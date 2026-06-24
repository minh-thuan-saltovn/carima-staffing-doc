# MO-AUTH-002-API-01-Logout

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Logout |
| Description | Đăng xuất và vô hiệu hóa phiên/token hiện tại. |
| Endpoint | `/api/v1/moto/auth/logout` |
| Menu | Company Settings |
| Method | GET |
| Related Tables | central_db.mst_tenant, tenant_db.mst_moto_user, tenant_db.[TBD: login_history / access_token] |
| Screen list | 企業マスタ (MO-SET-001) - Company Master |
| System | MOTO Portal |
| 説明 | ログアウトし、現在のセッション／トークンを無効化する。. |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design |
| API ID | MO-AUTH-002-API-01-Logout |
| API Name | Logout |
| Business Flow áp dụng | BF-001 Login Flow |
| Portal áp dụng | MOTO Portal |
| Screen ID liên quan | MO-AUTH-002 |
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
| API ID | MO-AUTH-002-API-01-Logout |
| Tên API | Logout |
| Tên tiếng Nhật | ログアウト |
| Business Flow liên quan | BF-001 Login Flow |
| Screen ID liên quan | MO-AUTH-002 |
| Chức năng liên quan | Đăng xuất khỏi MOTO Portal |
| Mục đích API | Hủy phiên đăng nhập hiện tại, revoke Access Token / Refresh Token và ghi nhận thời điểm logout |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/moto/auth/logout |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant được xác định từ Access Token / request context |
| Có Transaction không | Có |
| Xử lý file | Không |
| Response format | JSON |
| Trạng thái | Draft |

### 3.2 Mô tả API

API này dùng để đăng xuất người dùng MOTO Portal khỏi hệ thống.

Frontend gọi API Logout khi user bấm nút 「ログアウト」. Backend xác thực Access Token hiện tại, xác định Tenant và user tương ứng, sau đó revoke token, cập nhật Login History logout_at và ghi Audit Log.

Sau khi logout thành công, token hiện tại không còn được sử dụng cho các API yêu cầu Authentication.

### 3.3 Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User bấm nút 「ログアウト」 |
| Actor | MOTO User |
| Trước khi gọi API | User đang đăng nhập MOTO Portal |
| Sau khi gọi API thành công | User quay về màn hình Login |
| Sau khi gọi API thất bại | Frontend xóa token local và chuyển về Login theo policy |
| Thay đổi trạng thái liên quan | Authenticated → Guest |

---

## 4. Định nghĩa Endpoint

### 4.1 Endpoint

POST /api/v1/moto/auth/logout

### 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept | Có | application/json | Loại dữ liệu response mong muốn |
| Authorization | Có | Bearer eyJhbGciOi... | Access Token hiện tại |
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
  "logout_all_devices": false
}
```

### 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Default | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| logout_all_devices | boolean | Không | false | true / false | true: logout tất cả thiết bị, false: chỉ logout token hiện tại |

### 5.3 Ghi chú request

| Hạng mục | Nội dung |
| --- | --- |
| Có thể gửi body rỗng không | Có |
| Nếu body rỗng | Xử lý như logout_all_devices = false |
| User ID có truyền trong body không | Không |
| Tenant Code có truyền trong body không | Không |
| Token lấy từ đâu | Authorization Header |
| Tenant xác định từ đâu | Access Token / token record / request context |

### 5.4 Request Body rỗng hợp lệ

```json
{}
```

---

## 6. Response Definition

### 6.1 Success Response

HTTP Status: 200 OK

```json
{
  "success": true,
  "data": {
    "logged_out": true,
    "logout_all_devices": false,
    "logout_at": "2026-01-01T18:00:00+09:00"
  },
  "message": "ログアウトしました"
}
```

### 6.2 Success Response khi logout tất cả thiết bị

HTTP Status: 200 OK

```json
{
  "success": true,
  "data": {
    "logged_out": true,
    "logout_all_devices": true,
    "revoked_token_count": 3,
    "logout_at": "2026-01-01T18:00:00+09:00"
  },
  "message": "ログアウトしました"
}
```

### 6.3 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| success | boolean | Kết quả xử lý |
| data.logged_out | boolean | Kết quả logout |
| data.logout_all_devices | boolean | Có logout tất cả thiết bị hay không |
| data.revoked_token_count | integer | Số lượng token bị revoke, chỉ trả khi logout_all_devices = true |
| data.logout_at | string | Thời điểm logout |
| message | string | Message trả về |

### 6.4 Error Response - Unauthorized

HTTP Status: 401 Unauthorized

```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "認証が必要です"
  }
}
```

### 6.5 Error Response - Token Expired

HTTP Status: 401 Unauthorized

```json
{
  "success": false,
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "セッションの有効期限が切れています"
  }
}
```

### 6.6 Error Response - Token Revoked

HTTP Status: 401 Unauthorized

```json
{
  "success": false,
  "error": {
    "code": "TOKEN_REVOKED",
    "message": "セッションは既に無効です"
  }
}
```

### 6.7 Error Response - Validation Error

HTTP Status: 422 Unprocessable Entity

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "入力内容を確認してください",
    "details": [
      {
        "field": "logout_all_devices",
        "code": "INVALID_TYPE",
        "message": "logout_all_devicesの形式が正しくありません"
      }
    ]
  }
}
```

### 6.8 Error Response - System Error

HTTP Status: 500 Internal Server Error

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "システムエラーが発生しました"
  }
}
```

---

## 7. Validation Rules

| No. | Field | Rule | Error Code | Error Message |
| --- | --- | --- | --- | --- |
| 1 | Authorization | Bắt buộc | CMS-VAL-23 | Authorizationを入力してください。 |
| 2 | Authorization | Format Bearer token | INVALID_TOKEN_FORMAT | 認証情報の形式が正しくありません |
| 3 | logout_all_devices | Nếu truyền thì phải là boolean | CMS-VAL-58 | logout_all_devicesは、trueかfalseを指定してください。 |
| 4 | logout_all_devices | Nếu không truyền thì default false | - | - |

### 7.1 Validation xử lý trước Business Rule

| Step | Nội dung |
| --- | --- |
| 1 | Kiểm tra Authorization Header |
| 2 | Kiểm tra Bearer token format |
| 3 | Parse JSON request body nếu có |
| 4 | Validate logout_all_devices nếu có truyền |
| 5 | Nếu body rỗng thì set logout_all_devices = false |
| 6 | Chuyển sang xử lý Authentication / Business Rule |

## 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Authentication Required | API Logout yêu cầu Access Token hợp lệ |
| BR-002 | Token Validation | Access Token phải tồn tại, chưa hết hạn và chưa bị revoke |
| BR-003 | Tenant Resolution | Backend xác định Tenant từ token hoặc request context |
| BR-004 | Tenant Active Check | Chỉ xử lý logout trong Tenant hợp lệ |
| BR-005 | Tenant Isolation | Không cho phép revoke token khác Tenant |
| BR-006 | User Existence Check | User của token phải tồn tại trong tenant_db.mst_moto_user |
| BR-007 | Current Device Logout | Nếu logout_all_devices = false, chỉ revoke token hiện tại |
| BR-008 | All Devices Logout | Nếu logout_all_devices = true, revoke toàn bộ active token của user trong Tenant hiện tại |
| BR-009 | Refresh Token Revoke | Khi logout, refresh token liên quan cũng phải bị revoke |
| BR-010 | Login History Update | Cập nhật logout_at cho login history tương ứng |
| BR-011 | Audit Log | Ghi Audit Log khi logout thành công hoặc thất bại |
| BR-012 | Idempotent Policy | Nếu token đã expired/revoked, response theo Authentication Policy |
| BR-013 | Sensitive Information | Không trả raw token, token hash, refresh token ra response |
| BR-014 | Frontend Cleanup | Sau khi nhận response success, frontend xóa token local storage/session storage |

---

## 9. Permission / Authorization

### 9.1 Authentication

| Hạng mục | Nội dung |
| --- | --- |
| API yêu cầu Access Token không | Có |
| Token type | Bearer |
| Token lấy từ đâu | Authorization Header |
| Token hết hạn có được logout không | Không, trả TOKEN_EXPIRED hoặc UNAUTHORIZED theo policy |
| Token đã revoke có được logout không | Không, trả TOKEN_REVOKED hoặc UNAUTHORIZED theo policy |

### 9.2 Authorization

| Role | Cho phép logout MOTO Portal | Ghi chú |
| --- | --- | --- |
| MOTO Admin | Có | Nếu token thuộc Tenant hiện tại |
| MOTO User | Có | Nếu token thuộc Tenant hiện tại |
| SAKI Admin | Không | Không thuộc MOTO Portal |
| SAKI User | Không | Không thuộc MOTO Portal |
| Platform Admin | Không | Dùng Platform Admin Portal riêng |
| Staff | Không | Không thuộc scope API này |

### 9.3 Logic kiểm tra quyền

| Step | Nội dung |
| --- | --- |
| 1 | Kiểm tra Access Token |
| 2 | Xác định Tenant từ token |
| 3 | Tìm token record trong tenant DB |
| 4 | Kiểm tra token chưa revoke |
| 5 | Kiểm tra token chưa expired |
| 6 | Tìm user tương ứng |
| 7 | Kiểm tra user thuộc MOTO Portal |
| 8 | Cho phép logout |

---

## 10. Data Mapping

### 10.1 Token To Tenant Mapping

| Source | DB | Table | Column | Mô tả |
| --- | --- | --- | --- | --- |
| Authorization Token | tenant_db | TBD: access_token | access_token_hash | Tìm token hiện tại |
| Authorization Token | tenant_db | TBD: access_token | tenant_id | Xác định Tenant |
| Authorization Token | tenant_db | TBD: access_token | user_id | Xác định user |
| Authorization Token | tenant_db | TBD: access_token | expires_at | Kiểm tra token hết hạn |
| Authorization Token | tenant_db | TBD: access_token | revoked_at | Kiểm tra token đã revoke |
| Token tenant_id | central_db | mst_tenant | tenant_id | Xác nhận Tenant tồn tại |
| Token tenant_id | central_db | mst_tenant | status | Xác nhận Tenant active |

### 10.2 Request To DB Mapping

| Request Field | DB | Table | Column | Process | Mô tả |
| --- | --- | --- | --- | --- | --- |
| Authorization | tenant_db | TBD: access_token | access_token_hash | WHERE | Tìm token hiện tại |
| logout_all_devices | tenant_db | TBD: access_token | revoked_at | UPDATE | Nếu true, revoke toàn bộ active token của user |
| logout_all_devices | tenant_db | TBD: refresh_token | revoked_at | UPDATE | Nếu true, revoke toàn bộ refresh token của user |
| Header.User-Agent | tenant_db | TBD: audit_log | user_agent | INSERT | Lưu user agent |
| Client IP | tenant_db | TBD: audit_log | ip_address | INSERT | Lưu IP |
| Current datetime | tenant_db | TBD: login_history | logout_at | UPDATE | Lưu thời điểm logout |

### 10.3 DB To Response Mapping

| DB | Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: access_token | revoked_at | data.logged_out | Nếu update thành công thì true |
| Request | - | logout_all_devices | data.logout_all_devices | Giá trị request hoặc default false |
| System | - | current_datetime | data.logout_at | Thời điểm logout |
| tenant_db | TBD: access_token | count | data.revoked_token_count | Số token đã revoke khi logout_all_devices = true |

### 10.4 Update Access Token Mapping - Current Device Logout

| DB | Table | Column | Value | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: access_token | revoked_at | Current datetime | Revoke Access Token hiện tại |
| tenant_db | TBD: access_token | revoked_reason | logout | Lý do revoke |
| tenant_db | TBD: access_token | updated_at | Current datetime | Ngày cập nhật |
| tenant_db | TBD: access_token | updated_by | user_id | User thực hiện logout |

### 10.5 Update Refresh Token Mapping - Current Device Logout

| DB | Table | Column | Value | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: refresh_token | revoked_at | Current datetime | Revoke Refresh Token liên quan |
| tenant_db | TBD: refresh_token | revoked_reason | logout | Lý do revoke |
| tenant_db | TBD: refresh_token | updated_at | Current datetime | Ngày cập nhật |
| tenant_db | TBD: refresh_token | updated_by | user_id | User thực hiện logout |

### 10.6 Update Access Token Mapping - All Devices Logout

| DB | Table | Column | Value | Điều kiện |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: access_token | revoked_at | Current datetime | user_id = current_user_id AND revoked_at IS NULL |
| tenant_db | TBD: access_token | revoked_reason | logout_all_devices | user_id = current_user_id AND revoked_at IS NULL |
| tenant_db | TBD: access_token | updated_at | Current datetime | user_id = current_user_id |
| tenant_db | TBD: access_token | updated_by | user_id | user_id = current_user_id |

### 10.7 Update Login History Mapping

| DB | Table | Column | Value | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: login_history | logout_at | Current datetime | Thời điểm logout |
| tenant_db | TBD: login_history | updated_at | Current datetime | Ngày cập nhật |
| tenant_db | TBD: login_history | updated_by | user_id | User thực hiện logout |

### 10.8 Audit Log Mapping

| DB | Table | Column | Value | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: audit_log | audit_log_id | Generated UUID | Audit Log ID |
| tenant_db | TBD: audit_log | action_type | LOGOUT | Loại thao tác |
| tenant_db | TBD: audit_log | target_table | mst_moto_user | Bảng target |
| tenant_db | TBD: audit_log | target_id | user_id | User ID |
| tenant_db | TBD: audit_log | actor_user_id | user_id | User thực hiện |
| tenant_db | TBD: audit_log | ip_address | Client IP | IP |
| tenant_db | TBD: audit_log | user_agent | Header.User-Agent | Trình duyệt/thiết bị |
| tenant_db | TBD: audit_log | result | success / failed | Kết quả |
| tenant_db | TBD: audit_log | detail | JSON object | Nội dung chi tiết |
| tenant_db | TBD: audit_log | created_at | Current datetime | Ngày tạo |

### 10.9 Index cần thiết

| DB | Table | Index | Column | Mục đích |
| --- | --- | --- | --- | --- |
| tenant_db | TBD: access_token | idx_access_token_hash | access_token_hash | Tìm token hiện tại |
| tenant_db | TBD: access_token | idx_access_token_user_id | user_id | Revoke token theo user |
| tenant_db | TBD: access_token | idx_access_token_revoked_at | revoked_at | Lọc token active |
| tenant_db | TBD: refresh_token | idx_refresh_token_access_token_id | access_token_id | Revoke refresh token liên quan |
| tenant_db | TBD: login_history | idx_login_history_user_id | user_id | Tìm login history |
| tenant_db | TBD: login_history | idx_login_history_token_id | token_id | Tìm login history theo token |
| tenant_db | mst_moto_user | idx_mst_moto_user_user_id | user_id | Tìm user |

---

## 11. Database Transaction

### 11.1 Phạm vi transaction khi Logout Current Device

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate Authorization Header | Ngoài transaction |
| 2 | Verify Access Token | Ngoài transaction |
| 3 | Resolve Tenant | Ngoài transaction |
| 4 | Check user | Ngoài transaction |
| 5 | Begin transaction | Trong transaction |
| 6 | Revoke current Access Token | Trong transaction |
| 7 | Revoke related Refresh Token | Trong transaction |
| 8 | Update Login History logout_at | Trong transaction |
| 9 | Insert Audit Log success | Trong transaction |
| 10 | Commit transaction | Trong transaction |

### 11.2 Phạm vi transaction khi Logout All Devices

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate Authorization Header | Ngoài transaction |
| 2 | Verify Access Token | Ngoài transaction |
| 3 | Resolve Tenant | Ngoài transaction |
| 4 | Check user | Ngoài transaction |
| 5 | Begin transaction | Trong transaction |
| 6 | Revoke all active Access Tokens of current user | Trong transaction |
| 7 | Revoke all active Refresh Tokens of current user | Trong transaction |
| 8 | Update active Login History logout_at | Trong transaction |
| 9 | Insert Audit Log success | Trong transaction |
| 10 | Commit transaction | Trong transaction |

### 11.3 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Revoke Access Token thất bại | Rollback |
| 2 | Revoke Refresh Token thất bại | Rollback |
| 3 | Update Login History thất bại | Rollback |
| 4 | Insert Audit Log thất bại | Rollback |

---

## 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Authenticated | Logout current device | Guest | Token hiện tại được revoke |
| Authenticated | Logout all devices | Guest | Toàn bộ token active của user được revoke |
| Authenticated | Token expired | Guest | Token hết hạn |
| Authenticated | Token revoked | Guest | Token đã bị revoke |
| Guest | Logout | Guest | Không có token hợp lệ |
| Locked | Logout | Guest | Nếu token còn hợp lệ, vẫn cho phép logout theo policy |

---

## 13. Sequence / Processing Flow

### 13.1 Logout Current Device Flow

1. Frontend gọi API Logout
2. Backend kiểm tra Authorization Header
3. Backend parse Bearer token
4. Backend tìm token record từ access_token_hash
5. Backend kiểm tra token tồn tại
6. Backend kiểm tra token chưa expired
7. Backend kiểm tra token chưa revoked
8. Backend xác định Tenant từ token
9. Backend kiểm tra Tenant hợp lệ
10. Backend tìm user trong tenant_db.mst_moto_user
11. Backend kiểm tra user tồn tại
12. Backend bắt đầu transaction
13. Backend revoke Access Token hiện tại
14. Backend revoke Refresh Token liên quan
15. Backend cập nhật Login History logout_at
16. Backend ghi Audit Log success
17. Backend commit transaction
18. Backend trả Success Response
19. Frontend xóa token local
20. Frontend chuyển về màn hình Login

### 13.2 Logout All Devices Flow

1. Frontend gọi API Logout với logout_all_devices = true
2. Backend kiểm tra Authorization Header
3. Backend xác thực token hiện tại
4. Backend xác định Tenant và user
5. Backend bắt đầu transaction
6. Backend revoke toàn bộ active Access Token của user
7. Backend revoke toàn bộ active Refresh Token của user
8. Backend cập nhật logout_at cho Login History đang active
9. Backend ghi Audit Log success
10. Backend commit transaction
11. Backend trả Success Response kèm revoked_token_count
12. Frontend xóa token local
13. Frontend chuyển về màn hình Login

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
| 1 | Logout thông thường không gửi notification |
| 2 | Logout all devices không gửi notification |
| 3 | Nếu hệ thống yêu cầu cảnh báo bảo mật khi logout all devices, cần định nghĩa thêm policy riêng |

---

## 15. Audit Log

### 15.1 Audit Log Requirement

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | LOGOUT |
| Bảng target | mst_moto_user |
| ID record target | user_id |
| Thời điểm ghi log | Logout success / logout failed |
| Chính sách lưu trữ | Theo NFR |

### 15.2 Audit Log Detail JSON - Logout Current Device

```json
{
  "logout_result": "success",
  "logout_type": "current_device",
  "user_id": "moto001",
  "tenant_id": "tenant_moto_001",
  "ip_address": "192.168.1.1",
  "logout_at": "2026-01-01T18:00:00+09:00"
}
```

### 15.3 Audit Log Detail JSON - Logout All Devices

```json
{
  "logout_result": "success",
  "logout_type": "all_devices",
  "user_id": "moto001",
  "tenant_id": "tenant_moto_001",
  "revoked_token_count": 3,
  "ip_address": "192.168.1.1",
  "logout_at": "2026-01-01T18:00:00+09:00"
}
```

### 15.4 Audit Log Detail JSON - Failed

```json
{
  "logout_result": "failed",
  "failure_reason": "token_expired",
  "tenant_id": "tenant_moto_001",
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
| 1 | Authentication | API Logout yêu cầu Access Token |
| 2 | Authorization | Chỉ user đang đăng nhập mới được logout |
| 3 | Tenant Isolation | Không cho phép revoke token khác Tenant |
| 4 | Token Validation | Token phải tồn tại, chưa expired, chưa revoked |
| 5 | Token Revocation | Logout phải revoke Access Token và Refresh Token |
| 6 | Token Storage | DB nên lưu token dạng hash, không lưu raw token nếu có thể |
| 7 | Sensitive Data Masking | Không ghi raw token vào log |
| 8 | Replay Prevention | Token đã revoke không được sử dụng lại |
| 9 | Logout All Devices | Chỉ revoke token của chính user hiện tại trong Tenant hiện tại |
| 10 | Audit Log | Bắt buộc ghi log logout |
| 11 | HTTPS | Chỉ cho phép gọi API qua HTTPS |
| 12 | CORS | Chỉ cho phép origin hợp lệ |
| 13 | CSRF | Nếu dùng cookie-based auth thì cần CSRF protection |
| 14 | Local Token Cleanup | Frontend phải xóa Access Token / Refresh Token sau logout |

### 17.1 Security Response Policy

| Case | Message Policy |
| --- | --- |
| Token không tồn tại | Trả UNAUTHORIZED |
| Token hết hạn | Trả TOKEN_EXPIRED hoặc UNAUTHORIZED theo policy |
| Token đã revoke | Trả TOKEN_REVOKED hoặc UNAUTHORIZED theo policy |
| User không tồn tại | Trả UNAUTHORIZED |
| Tenant không hợp lệ | Trả UNAUTHORIZED hoặc TENANT_NOT_FOUND theo policy |
| Logout thành công | Trả success true |

---

## 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | ≤ 2 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Không |
| Index cần thiết | Có |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |
| Transaction scope | Ngắn, chỉ bao gồm revoke token, update login history, insert audit log |
| Rate limit | Có thể áp dụng nhưng không bắt buộc nghiêm ngặt như login |

### 18.1 Performance Notes

| No. | Nội dung |
| --- | --- |
| 1 | Cần index access_token_hash để tìm token nhanh |
| 2 | Cần index user_id để logout all devices nhanh |
| 3 | Không nên decode/verify token nhiều lần |
| 4 | Không trả dữ liệu user lớn trong response logout |
| 5 | Audit Log nên ghi tối giản để transaction không kéo dài |
| 6 | Logout all devices có thể update nhiều record nhưng chỉ trong phạm vi user hiện tại |

---

## 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 Login Flow |
| Wireframe | MO-AUTH-002 |
| Screen Detail Design | MO-AUTH-002 |
| API Login | MO-AUTH-001-API-01-Login |
| ERD | Carima Data Dictionary |
| Data Dictionary | central_db.mst_tenant, tenant_db.mst_moto_user |
| Token Table | TBD: access_token / refresh_token |
| Login History Table | TBD: login_history |
| Audit Log Table | TBD: audit_log |
| Permission Matrix | TBD |
| Security Policy | TBD |
| NFR | TBD |

---

## 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| 1 | Chưa xác định bảng Access Token chính thức trong Data Dictionary | High | TBD | TBD | Open |
| 2 | Chưa xác định bảng Refresh Token chính thức trong Data Dictionary | High | TBD | TBD | Open |
| 3 | Chưa xác định bảng Login History chính thức trong Data Dictionary | High | TBD | TBD | Open |
| 4 | Chưa xác định bảng Audit Log chính thức trong Data Dictionary | Medium | TBD | TBD | Open |
| 5 | Chưa xác định column liên kết token_id với login_history | Medium | TBD | TBD | Open |
| 6 | Chưa xác định policy token expired khi logout | Medium | TBD | TBD | Open |
| 7 | Chưa xác định có hỗ trợ logout_all_devices trên UI hay không | Low | TBD | TBD | Open |
| 8 | Chưa xác định token lưu raw hay hash trong DB | High | TBD | TBD | Open |
| 9 | Chưa xác định thời gian giữ Login History / Audit Log | Medium | TBD | TBD | Open |

---

## 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | API ID đã được định nghĩa | Done |
| 2 | Endpoint đã được định nghĩa | Done |
| 3 | Method đã được định nghĩa | Done |
| 4 | Request header đã được định nghĩa | Done |
| 5 | Request body đã được định nghĩa bằng JSON chuẩn | Done |
| 6 | Request body rỗng hợp lệ đã được định nghĩa | Done |
| 7 | Success response đã được định nghĩa bằng JSON chuẩn | Done |
| 8 | Error response đã được định nghĩa bằng JSON chuẩn | Done |
| 9 | Validation rule đã được định nghĩa | Done |
| 10 | Authentication rule đã được định nghĩa | Done |
| 11 | Authorization rule đã được định nghĩa | Done |
| 12 | Business rule đã được định nghĩa | Done |
| 13 | Tenant isolation rule đã được định nghĩa | Done |
| 14 | Data mapping đã được định nghĩa | Done |
| 15 | DB mapping cho token đã được định nghĩa | Done |
| 16 | DB mapping cho login history đã được định nghĩa | Done |
| 17 | DB mapping cho audit log đã được định nghĩa | Done |
| 18 | Transaction scope đã được định nghĩa | Done |
| 19 | Rollback condition đã được định nghĩa | Done |
| 20 | Status transition đã được định nghĩa | Done |
| 21 | Sequence flow đã được định nghĩa | Done |
| 22 | Notification rule đã được định nghĩa | Done |
| 23 | Audit Log đã được định nghĩa | Done |
| 24 | File handling đã được định nghĩa | Done |
| 25 | Security considerations đã được định nghĩa | Done |
| 26 | Performance considerations đã được định nghĩa | Done |
| 27 | Tài liệu liên quan đã được định nghĩa | Done |
| 28 | Open issues đã được định nghĩa | Done |
| 29 | Các bảng chưa có trong Data Dictionary đã được đánh dấu TBD | Done |