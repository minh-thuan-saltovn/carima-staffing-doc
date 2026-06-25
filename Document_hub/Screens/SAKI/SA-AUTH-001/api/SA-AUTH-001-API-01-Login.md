# SA-AUTH-001-API-01-Login

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Login |
| Description | Đăng nhập, xác thực tài khoản và cấp token truy cập. |
| Endpoint | /api/v1/saki/login |
| Menu | Common |
| Method | POST |
| Related Tables | central_db.mst_tenant, tenant_db.mst_saki_user, tenant_db.[TBD: login_history / access_token] |
| Screen list | ログイン (SA-AUTH-001) / Login |
| System | SAKI Portal |
| 説明 | ログイン認証を行い、アクセストークンを発行する。 |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design |
| Portal áp dụng | SAKI Portal |
| Version | v1.0 |
| Ngày tạo | 2026/06/23 |
| Ngày cập nhật | 2026/06/23 |
| Người tạo | Thuận |
| Người review | Sang |
| Trạng thái | VN Review |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/23 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Saki Login. |

---

## 3. Tổng quan API

### 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-AUTH-001-API-01-Login |
| Tên API | Login |
| Tên tiếng Nhật | ログイン |
| Business Flow liên quan | BF-001 Login Flow |
| Screen ID liên quan | SA-AUTH-001 |
| Chức năng liên quan | Đăng nhập SAKI Portal |
| Mục đích API | Xác thực tài khoản người dùng SAKI Portal và phát hành Access Token cùng danh sách quyền hạn. |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/saki/login |
| Authentication | Không yêu cầu |
| Authorization | Không yêu cầu trước login |
| Phạm vi Tenant | Tenant được xác định theo domain/subdomain hoặc context request |
| Có Transaction không | Có |
| Xử lý file | Không |
| Response format | JSON |
| Trạng thái | VN Review |

---

### 3.2 Mô tả API

API này xác thực người dùng SAKI Portal thông qua User ID và mật khẩu.

Hệ thống tự động xác định Tenant từ request, truy vấn thông tin trong bảng mst_saki_user, xác minh mật khẩu băm và chính sách bảo mật liên quan.

Khi đăng nhập thành công, hệ thống phát hành Access/Refresh Token, ghi nhận lịch sử và trả về thông tin user kèm danh sách quyền hạn. Nếu thất bại, ghi nhận lịch sử lỗi và trả về thông báo lỗi chung.

---

### 3.3 Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User bấm nút 「ログイン」 |
| Actor | SAKI User |
| Trước khi gọi API | User đang ở màn hình Login |
| Sau khi gọi API thành công | User được chuyển tới Dashboard |
| Sau khi gọi API thất bại | Hiển thị message lỗi trên màn hình Login |
| Thay đổi trạng thái liên quan | Guest → Authenticated |

---

## 4. Định nghĩa Endpoint

### 4.1 Endpoint

```
POST /api/v1/saki/login
```

### 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Content-Type | Có | application/json | Định dạng dữ liệu gửi lên |
| Accept | Có | application/json | Định dạng dữ liệu mong muốn nhận về |
| Accept-Language | Không | ja | Ngôn ngữ hiển thị câu thông báo lỗi |
| User-Agent | Không | Mozilla/5.0... | Thông tin trình duyệt/thiết bị của client |
| X-Forwarded-For | Không | 192.168.1.100 | IP thực của client |

---

### 4.3 Path Parameters

Không áp dụng.

---

### 4.4 Query Parameters

Không áp dụng.

---

## 5. Request Body

### 5.1 Request Body JSON

```json
{
  "user_id": "saki_user_01",
  "password": "password123",
  "remember_me": true
}
```

---

### 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| user_id | string | Có | 100 | Không rỗng, trim | ID tài khoản đăng nhập của user |
| password | string | Có | 255 | Không rỗng | Mật khẩu đăng nhập |
| remember_me | boolean | Không | - | boolean | Cờ ghi nhớ đăng nhập để điều chỉnh thời gian hạn token |

---

### 5.3 Ghi chú request

* Tenant Code không được gửi trực tiếp trong Request Body để tăng tính bảo mật và độc lập về định danh tenant.
* Hệ thống sẽ tự động phân giải Tenant ID từ domain/subdomain (ví dụ: clientA.saki-portal.com -> tenant của clientA) hoặc context của proxy/router.

---

## 6. Response Definition

### 6.1 Success Response

HTTP Status: 200 OK

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxxx.yyyyy",
    "refresh_token": "refresh_token_saki_sample",
    "token_type": "Bearer",
    "expires_in": 3600,
    "expires_at": "2026-06-23T15:35:00+09:00",
    "user": {
      "user_id": "saki_user_01",
      "company_id": "SAKI0000000000000001",
      "office_id": "OFF00000000000000001",
      "department_id": "DEP00000000000000001",
      "last_name_ja": "山田",
      "first_name_ja": "太郎",
      "email": "yamada@saki-company.co.jp",
      "role_id": "ROLE_SAKI_ADMIN",
      "role_name": "SAKI Admin",
      "reference_scope": 6
    },
    "tenant": {
      "tenant_id": "tenant_saki_001",
      "tenant_code": "SAKI001",
      "tenant_name": "SAKI Company Ltd"
    },
    "permissions": [
      "USER_VIEW",
      "USER_UPDATE"
    ]
  },
  "message": "Đăng nhập thành công."
}
```

---

### 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| success | boolean | Kết quả xử lý |
| data.access_token | string | JWT Access Token để gọi các API tiếp theo |
| data.refresh_token | string | Token dùng để làm mới Access Token |
| data.token_type | string | Loại token, mặc định Bearer |
| data.expires_in | integer | Thời gian sống của Access Token (giây) |
| data.expires_at | string | Thời điểm hết hạn của Access Token |
| data.user.user_id | string | ID người dùng |
| data.user.company_id | string | ID công ty SAKI liên kết |
| data.user.office_id | string | ID sự nghiệp sở trực thuộc của user |
| data.user.department_id | string | ID phòng ban trực thuộc của user |
| data.user.last_name_ja | string | Họ tiếng Nhật |
| data.user.first_name_ja | string | Tên tiếng Nhật |
| data.user.email | string | Địa chỉ email của user |
| data.user.role_id | string | Mã vai trò phân quyền thực thi |
| data.user.role_name | string | Tên vai trò phân quyền |
| data.user.reference_scope | integer | Phạm vi truy cập dữ liệu của user (1-6) |
| data.tenant.tenant_id | string | ID của Tenant |
| data.tenant.tenant_code | string | Mã Code của Tenant |
| data.tenant.tenant_name | string | Tên Tenant |
| data.permissions | array[string] | Danh sách quyền chi tiết của user |
| message | string | Thông báo kết quả |

---

### 6.3 Error Response - Validation Error

HTTP Status: 422 Unprocessable Entity

---

### 6.4 Error Response - Invalid Credentials

HTTP Status: 401 Unauthorized

---

### 6.5 Error Response - Account Locked

HTTP Status: 403 Forbidden

---

### 6.6 Error Response - Account Inactive

HTTP Status: 403 Forbidden

---

### 6.7 Error Response - Tenant Not Found

HTTP Status: 404 Not Found

---

### 6.8 Error Response - System Error

HTTP Status: 500 Internal Server Error

---


## 7. Validation Rules

Message list 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | user_id | required | - | required |
| 2 | user_id | max | 100 | max |
| 3 | password | required | - | required |
| 4 | password | max | 255 | max |
| 5 | remember_me | boolean | - | boolean |

---

### 7.1 Validation xử lý trước Business Rule

| Step | Nội dung |
| --- | --- |
| 1 | Kiểm tra Content-Type là application/json |
| 2 | Parse JSON request body |
| 3 | Validate required field |
| 4 | Validate type / max length |
| 5 | Trim user_id |
| 6 | Chuyển sang xử lý Business Rule |

---

## 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant Resolution | Xác định Tenant ID bằng cách tra cứu domain/subdomain trong central_db.mst_tenant. |
| BR-002 | Tenant Active Check | Chỉ cho phép đăng nhập nếu trạng thái tenant đang hoạt động (status = 1). |
| BR-003 | Tenant Isolation | Tìm kiếm tài khoản của user_id độc lập trong bảng mst_saki_user thuộc tenant schema tương ứng. |
| BR-004 | User Existence Check | Tìm kiếm sự tồn tại của user_id trong tenant DB. Nếu không tồn tại, trả lỗi INVALID_CREDENTIALS (không phân biệt rõ tài khoản hay mật khẩu sai để tránh brute-force). |
| BR-005 | Password Verification | So khớp password truyền lên với password_hash được lưu trong cơ sở dữ liệu. |
| BR-006 | User Status Check | Chỉ cho phép đăng nhập nếu tài khoản người dùng đang có hiệu lực (status = 1). Nếu status = 0, trả lỗi ACCOUNT_INACTIVE. |
| BR-007 | Account Lock Check | Kiểm tra cờ khóa tài khoản hoặc số lần đăng nhập thất bại. Nếu tài khoản đang bị khóa, trả lỗi ACCOUNT_LOCKED. |
| BR-008 | Failed Login Handling | Nếu mật khẩu sai: Tăng số lần đăng nhập sai (failed_login_count). Nếu failed_login_count vượt quá số lần cho phép (ví dụ: 5 lần), cập nhật trạng thái khóa tài khoản và thời điểm khóa. |
| BR-009 | Reset Failed Count | Nếu đăng nhập thành công: Reset failed_login_count về 0 và cập nhật last_login_at. |
| BR-010 | Token Generation | Sinh access_token và refresh_token. Nếu remember_me = true, tăng thời gian sống của refresh_token. |
| BR-011 | Ghi nhận Login History | Ghi nhận lịch sử đăng nhập vào bảng lịch sử chấm công/đăng nhập của tenant schema. |

---

## 9. Permission / Authorization

### 9.1 Authentication

* API này không yêu cầu Access Token để thực thi.
* Cho phép người dùng chưa xác thực (Guest) truy cập.

---

### 9.2 Authorization

| Role | Cho phép login SAKI Portal | Ghi chú |
| --- | --- | --- |
| SAKI Admin | Có | Phải thuộc tenant schema đang truy cập |
| SAKI User / Staff | Có | Phải thuộc tenant schema đang truy cập |
| MOTO Portal Users | Không | Không thuộc scope phân quyền của SAKI |
| Platform Admin | Không | Đăng nhập cổng quản trị SaaS riêng biệt |

---

### 9.3 Logic kiểm tra quyền sau khi xác thực

```
1. Tìm kiếm thông tin user theo user_id trong mst_saki_user.
2. Lấy role_id liên kết của user.
3. Truy vấn bảng phân quyền tương ứng để lấy toàn bộ danh sách mã quyền hạn (permission_code) gán cho role đó.
4. Gán danh sách quyền này vào Response Body trả về cho client để frontend điều hướng hiển thị menu/chức năng.
```

---

## 10. Data Mapping

### 10.1 Tenant Resolution Mapping

| Source | DB | Table | Column | Mô tả |
| --- | --- | --- | --- | --- |
| Request Subdomain / Host | central_db | mst_tenant | tenant_id | Lấy Tenant ID |
| Request Subdomain / Host | central_db | mst_tenant | tenant_code | Lấy Tenant Code |
| Request Subdomain / Host | central_db | mst_tenant | tenant_name | Lấy tên Tenant |
| Request Subdomain / Host | central_db | mst_tenant | db_schema | Xác định schema kết nối động |
| Request Subdomain / Host | central_db | mst_tenant | status | Kiểm tra trạng thái hoạt động |

---

### 10.2 Request To DB Mapping

| Request Field | DB | Table | Column | Process | Mô tả |
| --- | --- | --- | --- | --- | --- |
| user_id | tenant_db | mst_saki_user | user_id | WHERE | Khớp mã tài khoản |
| password | tenant_db | mst_saki_user | password_hash | Verify | So sánh mật khẩu bằng hash |
| remember_me | tenant_db | access_token | expires_at | Calculate | Tính hạn token |
| Header.User-Agent | tenant_db | login_history | user_agent | INSERT | Lưu thiết bị |
| Client IP | tenant_db | login_history | ip_address | INSERT | Lưu IP |

---

### 10.3 DB To Response Mapping

| DB | Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | mst_saki_user | user_id | data.user.user_id | ID người dùng |
| tenant_db | mst_saki_user | company_id | data.user.company_id | ID công ty SAKI |
| tenant_db | mst_saki_user | last_name_ja | data.user.last_name_ja | Họ tiếng Nhật |
| tenant_db | mst_saki_user | first_name_ja | data.user.first_name_ja | Tên tiếng Nhật |
| tenant_db | mst_saki_user | email | data.user.email | Email của user |
| tenant_db | mst_saki_user | execution_role_id | data.user.role_id | Mã vai trò phân quyền |
| central_db | mst_tenant | tenant_id | data.tenant.tenant_id | Tenant ID |
| central_db | mst_tenant | tenant_code | data.tenant.tenant_code | Tenant Code |
| central_db | mst_tenant | tenant_name | data.tenant.tenant_name | Tên Tenant |

---

### 10.4 Index cần thiết

| DB | Table | Index Name | Column | Mục đích |
| --- | --- | --- | --- | --- |
| central_db | mst_tenant | idx_tenant_domain | domain / subdomain | Tra cứu tenant theo tên miền truy cập |
| tenant_db | mst_saki_user | idx_saki_user_login | user_id, status | Tra cứu nhanh tài khoản đang hoạt động |

---

## 11. Database Transaction

### 11.1 Phạm vi transaction khi Login Success

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Xác định Tenant & Validate request | Ngoài transaction |
| 2 | Verify password & status | Ngoài transaction |
| 3 | Bắt đầu Database Transaction | Trong transaction |
| 4 | Tạo mới bản ghi token trong DB | Trong transaction |
| 5 | Ghi nhận Login History thành công | Trong transaction |
| 6 | Cập nhật last_login_at & reset failed_login_count | Trong transaction |
| 7 | Ghi Audit Log thành công | Trong transaction |
| 8 | Commit Transaction | Trong transaction |

---

### 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Ghi token thất bại | Rollback toàn bộ transaction |
| 2 | Cập nhật thông tin đăng nhập cuối của user lỗi | Rollback toàn bộ transaction |
| 3 | Ghi lịch sử đăng nhập/Audit log lỗi | Rollback toàn bộ transaction |

---

## 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Guest (Khách) | Xác thực thành công | Authenticated (Đăng nhập) | Điền đúng thông tin, tài khoản hoạt động |
| Guest (Khách) | Xác thực thất bại | Guest (Khách) | Sai thông tin đăng nhập |
| Guest (Khách) | Sai mật khẩu liên tiếp | Locked (Khóa) | Số lần đăng nhập sai vượt hạn mức cho phép |
| Authenticated | Logout | Guest (Khách) | Token bị thu hồi hoặc đăng xuất |

---

## 13. Sequence / Processing Flow

### 13.1 Login Success Flow

1. Frontend gọi API Login.
2. Backend xác định Tenant và kết nối DB schema tương ứng.
3. Tìm user trong mst_saki_user, kiểm tra trạng thái hoạt động và verify password.
4. Khởi tạo Database Transaction:
   - Tạo Access Token và Refresh Token.
   - Ghi nhận Login History thành công.
   - Cập nhật last_login_at và reset failed_login_count về 0.
   - Ghi Audit Log thành công.
5. Commit Transaction và trả về Success Response HTTP 200 OK.

### 13.2 Login Failed Flow

1. Frontend gọi API Login.
2. Backend xác định Tenant và truy vấn user.
3. Xác định các trường hợp thất bại (sai thông tin, tài khoản bị vô hiệu/khóa).
4. Khởi tạo Database Transaction:
   - Tăng failed_login_count và tự động khóa tài khoản nếu vượt giới hạn.
   - Ghi nhận Login History thất bại.
   - Ghi Audit Log thất bại tương ứng.
5. Commit Transaction và trả về Error Response thích hợp (HTTP 401/403/422).

---

## 14. Notification

Không áp dụng.

---

## 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | LOGIN |
| Bảng target | mst_saki_user |
| ID record target | user_id |
| Dữ liệu ghi log | user_id, ip_address, user_agent, kết quả đăng nhập (success/failed/locked) |
| Chính sách lưu trữ | Ghi vào bảng log hoạt động chung của tenant schema |

---

## 16. Xử lý file

Không áp dụng.

---

## 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Mật khẩu an toàn | Tuyệt đối không lưu mật khẩu thô trong DB, bắt buộc băm bằng thuật toán an toàn (Bcrypt/Argon2). |
| 2 | Brute-force Prevention | Không thông báo lỗi chi tiết "Sai mật khẩu" hoặc "Tài khoản không tồn tại", chỉ trả về thông báo chung "Tài khoản hoặc mật khẩu không chính xác." |
| 3 | Token Revocation | Refresh Token phải được quản lý hạn dùng và có cơ chế vô hiệu hóa khi người dùng chọn Đăng xuất. |
| 4 | SQL Injection Prevention | Sử dụng Eloquent ORM / Parameterized Queries khi truy vấn thông tin user. |
| 5 | Masking Sensitive Data | Không ghi mật khẩu hay mã băm mật khẩu ra file log hệ thống hoặc audit log. |

---

## 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian phản hồi | Dưới 2 giây (đã tính thời gian thực hiện băm mật khẩu) |
| Index cần thiết | Index theo user_id trong bảng mst_saki_user để truy vấn nhanh thông tin đăng nhập |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 Login Flow |
| Database Dictionary | Carima-Staffing Data Dictionary / ERD (mst_tenant, mst_saki_user) |
| API Specification | MO-AUTH-001-API-01-Login (đối ứng phía MOTO Portal) |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| - | Không áp dụng. | - | - | - | - |

---

## 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Done |
| 2 | Request body đã được định nghĩa | Done |
| 3 | Response body đã được định nghĩa | Done |
| 4 | Error response đã được định nghĩa | Done |
| 5 | Validation rule đã được định nghĩa | Done |
| 6 | Business rule đã được định nghĩa | Done |
| 7 | Permission check đã được định nghĩa | Done |
| 8 | Tenant isolation rule đã được định nghĩa | Done |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done |
| 14 | File handling rule đã được định nghĩa nếu cần | Done |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
