# PA-TEN-009-API-01-Create saki tenant

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Create saki tenant |
| Description | Tạo mới tenant SAKI. |
| Endpoint | `/api/v1/admin/saki-tenants` |
| Menu | Tenant Management |
| Method | POST |
| Related Tables | `platform.tenant_registry`, `tenant_db.mst_saki_company`, `tenant_db.mst_saki_user` |
| Screen list | 派遣先テナント作成 (PA-TEN-009) — Create SAKI Tenant |
| System | Platform SaaS Admin |
| 説明 | SAKIテナントを新規作成する。 |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Create saki tenant |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.0 |
| Ngày tạo | 2026/06/17 |
| Ngày cập nhật | 2026/06/17 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Create SAKI Tenant theo template, sơ đồ màn hình và cấu trúc DB thực tế |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-009-API-01 |
| Tên API | Create saki tenant |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-009 |
| Chức năng liên quan | Tenant Management (Create SAKI Tenant) |
| Mục đích API | Đăng ký một SAKI Tenant (doanh nghiệp tiếp nhận phái cử) mới vào hệ thống, tự động cấp phát schema/database PostgreSQL riêng và khởi tạo tài khoản Admin cho Tenant đó. |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/admin/saki-tenants |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này được sử dụng bởi quản trị viên hệ thống (Platform SaaS Admin) để đăng ký một Tenant SAKI mới. Khi đăng ký thành công, hệ thống sẽ tự động khởi tạo cơ sở dữ liệu riêng (PostgreSQL schema riêng), chạy migrations tạo cấu trúc bảng, thêm dữ liệu ban đầu cho công ty và tài khoản quản trị viên, và gửi email chứa liên kết đặt mật khẩu cho admin tenant.

### Ngữ cảnh nghiệp vụ

Quản trị viên điền đầy đủ thông tin vào form của màn hình "Tạo Tenant SAKI mới" (PA-TEN-009), sau đó nhấn nút "SAKIテナント発行" (Phát hành) để kích hoạt quá trình.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User nhấn nút Phát hành trên giao diện màn hình PA-TEN-009, xác nhận qua dialog `CMS-VAL-85` |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Chưa tồn tại bản ghi tenant trong database hệ thống |
| Sau khi gọi API | Tenant mới được tạo, có schema DB riêng chứa master company và admin user ban đầu; một email kích hoạt được tự động đẩy vào Queue ngầm để gửi đi |
| Thay đổi trạng thái liên quan | Tạo mới bản ghi, trạng thái hoạt động mặc định của Tenant là `status = 1` |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/admin/saki-tenants
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về khi có lỗi |

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
  "tenant_code": "SAKI-0001",
  "domain": "saki-factory",
  "plan_code": "STANDARD",
  "official_name_ja": "SAKI Partner Factory",
  "display_name_ja": "SAKI Factory",
  "postal_code": "2000002",
  "tel": "03-5678-1234",
  "address_ja": "東京都千代田区麹町1-1 SAKIビル2F",
  "admin_user": {
    "last_name_ja": "佐藤",
    "first_name_ja": "二郎",
    "email": "sato@saki-factory.jp",
    "tel": "080-1234-5678"
  }
}
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| tenant_code | string | Có | 50 | Bắt buộc, chỉ nhập chữ cái tiếng Anh nửa dòng và số, không khoảng trắng/ký tự đặc biệt. Duy nhất trong DB. | Mã Tenant định danh |
| domain | string | Có | 50 | Bắt buộc, định dạng subdomain (ký tự thường, số, dấu gạch ngang). Duy nhất trong DB. | Tên miền con định danh |
| plan_code | string | Có | 20 | Bắt buộc. Phải thuộc các gói hợp lệ: `LITE`, `STANDARD`, `PRO`, `ENTERPRISE` | Gói dịch vụ đăng ký |
| official_name_ja | string | Có | 100 | Bắt buộc. Tên chính thức công ty. | Tên chính thức doanh nghiệp |
| display_name_ja | string | Có | 24 | Bắt buộc. Tên hiển thị hệ thống. | Tên hiển thị rút gọn |
| postal_code | string | Có | 7 | Bắt buộc, định dạng 7 chữ số liên tục, không có dấu gạch ngang. | Mã bưu chính |
| tel | string | Có | 15 | Bắt buộc, đúng định dạng số điện thoại liên lạc. | Số điện thoại công ty |
| address_ja | string | Có | 100 | Bắt buộc. Địa chỉ trụ sở. | Địa chỉ văn phòng |
| admin_user | object | Có | - | Bắt buộc. | Thông tin tài khoản admin |
| admin_user.last_name_ja | string | Có | 24 | Bắt buộc. Họ quản trị viên. | Họ của quản trị viên |
| admin_user.first_name_ja | string | Có | 24 | Bắt buộc. Tên quản trị viên. | Tên của quản trị viên |
| admin_user.email | string | Có | 128 | Bắt buộc, định dạng email hợp lệ. Duy nhất trong DB. | Email tài khoản admin |
| admin_user.tel | string | Có | 15 | Bắt buộc, đúng định dạng số điện thoại. | SĐT liên lạc của admin |

---

# 6. Response Definition

## 6.1 Success Response

### HTTP Status

```
201 Created
```

### Response Body

```json
{
  "data": {
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "tenant_code": "SAKI-0001",
    "company_name": "SAKI Partner Factory",
    "created_at": "2026-06-17T14:46:00+09:00"
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant_id | string | ID định danh duy nhất cấp phát cho Tenant (ULID) |
| data.tenant_code | string | Mã Tenant đã đăng ký |
| data.company_name | string | Tên công ty chính thức của Tenant |
| data.created_at | string | Thời điểm tạo Tenant trong hệ thống |

---

## 6.3 Error Response

### Validation Error

```
422 Unprocessable Entity
```

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed.",
    "details": [
      {
        "field": "tenant_code",
        "message": "The tenant code has already been taken."
      }
    ]
  }
}
```

### Unauthorized

```
401 Unauthorized
```

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication is required."
  }
}
```

### Forbidden

```
403 Forbidden
```

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to perform this operation."
  }
}
```

### State Conflict / Duplicate Value

```
409 Conflict
```

```json
{
  "error": {
    "code": "DUPLICATE_RESOURCE",
    "message": "The domain name or tenant code already exists."
  }
}
```

### System Error

```
500 Internal Server Error
```

```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred during tenant database migration."
  }
}
```

---

# 7. Validation Rules

| No. | Field | Rule | Error Code | Error Message |
| --- | --- | --- | --- | --- |
| 1 | tenant_code | Bắt buộc nhập | CMS-VAL-23 | テナントコードを入力してください。 |
| 2 | tenant_code | Định dạng: Chỉ cho phép ký tự tiếng Anh nửa dòng và chữ số, không khoảng trắng, không ký tự đặc biệt | CMS-VAL-24 | テナントコードに正しい形式を指定してください。 |
| 3 | tenant_code | Tối đa 50 ký tự | CMS-VAL-6 | テナントコードは50文字以内で入力してください。 |
| 4 | tenant_code | Phải là duy nhất (không trùng lặp trong platform.tenant_registry) | CMS-VAL-11 | テナントコードの値は既に存在しています。 |
| 5 | domain | Bắt buộc nhập | CMS-VAL-23 | ドメインを入力してください。 |
| 6 | domain | Tối đa 50 ký tự | CMS-VAL-6 | ドメインは50文字以内で入力してください。 |
| 7 | domain | Định dạng: chỉ chứa ký tự thường a-z, số 0-9 và dấu gạch ngang - | CMS-VAL-24 | ドメインに正しい形式を指定してください。 |
| 8 | domain | Phải là duy nhất (không trùng lặp trong platform.tenant_registry) | CMS-VAL-11 | ドメインの値は既に存在しています。 |
| 9 | plan_code | Bắt buộc nhập | CMS-VAL-23 | プランコードを入力してください。 |
| 10 | plan_code | Phải thuộc danh sách: LITE, STANDARD, PRO, ENTERPRISE | CMS-VAL-41 | 選択されたプランコードは正しくありません。 |
| 11 | official_name_ja | Bắt buộc nhập | CMS-VAL-23 | 公式会社名（和文）を入力してください。 |
| 12 | official_name_ja | Tối đa 100 ký tự | CMS-VAL-6 | 公式会社名（和文）は100文字以内で入力してください。 |
| 13 | display_name_ja | Bắt buộc nhập | CMS-VAL-23 | 表示名（和文）を入力してください。 |
| 14 | display_name_ja | Tối đa 24 ký tự | CMS-VAL-6 | 表示名（和文）は24文字以内で入力してください。 |
| 15 | postal_code | Bắt buộc nhập | CMS-VAL-23 | 郵便番号を入力してください。 |
| 16 | postal_code | Đúng 7 chữ số | CMS-VAL-53 | 郵便番号は7文字で入力してください。 |
| 17 | tel | Bắt buộc nhập | CMS-VAL-23 | 電話番号を入力してください。 |
| 18 | tel | Đúng định dạng số điện thoại | CMS-VAL-9 | 電話番号は有効な電話番号を入力してください。 |
| 19 | address_ja | Bắt buộc nhập | CMS-VAL-23 | 住所（和文）を入力してください。 |
| 20 | address_ja | Tối đa 100 ký tự | CMS-VAL-6 | 住所（和文）は100文字以内で入力してください。 |
| 21 | admin_user | Bắt buộc nhập cấu trúc | CMS-VAL-23 | 管理者情報を入力してください。 |
| 22 | admin_user.last_name_ja | Bắt buộc nhập | CMS-VAL-23 | 管理者氏名（姓）を入力してください。 |
| 23 | admin_user.last_name_ja | Tối đa 24 ký tự | CMS-VAL-6 | 管理者氏名（姓）は24文字以内で入力してください。 |
| 24 | admin_user.first_name_ja | Bắt buộc nhập | CMS-VAL-23 | 管理者氏名（名）を入力してください。 |
| 25 | admin_user.first_name_ja | Tối đa 24 ký tự | CMS-VAL-6 | 管理者氏名（名）は24文字以内で入力してください。 |
| 26 | admin_user.email | Bắt buộc nhập | CMS-VAL-23 | 管理者メールアドレスを入力してください。 |
| 27 | admin_user.email | Đúng định dạng Email | CMS-VAL-48 | 管理者メールアドレスには、有効なメールアドレスを指定してください。 |
| 28 | admin_user.email | Phải là duy nhất trong schema/bảng quản trị | CMS-VAL-11 | 管理者メールアドレスの値は既に存在しています。 |
| 29 | admin_user.tel | Bắt buộc nhập | CMS-VAL-23 | 管理者電話番号を入力してください。 |
| 30 | admin_user.tel | Đúng định dạng số điện thoại | CMS-VAL-9 | 管理者電話番号は有効な電話番号を入力してください。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant type assignment | Toàn bộ các tenant tạo mới qua endpoint này mặc định được cấu hình trường `tenant_type = 'saki'` trong cơ sở dữ liệu dùng chung. |
| BR-002 | Schema and DB provisioning | Hệ thống tự động phát sinh tên schema độc lập (`schema_name` có tiền tố `tenant_saki_` kết hợp ngẫu nhiên để tránh đụng độ) và tạo PostgreSQL role tương ứng (`db_user`). Chạy tự động các lệnh migrations để tạo cấu trúc bảng trên schema mới này. |
| BR-003 | Default status assignment | Sau khi khởi tạo thành công, Tenant mới mặc định có trạng thái kích hoạt hoạt động (`status = 1`). |
| BR-004 | Authorization restriction | Chỉ người dùng thuộc nhóm Platform SaaS Admin có quyền `tenant.create` mới được phép truy cập và thực thi API này. |
| BR-005 | Asynchronous Email | Hành động gửi email kích hoạt cho Admin của tenant không được xử lý đồng bộ làm chậm phản hồi API. Hệ thống phải đẩy job gửi email kích hoạt vào hàng đợi (Queue Job) để xử lý ngầm. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create |
| --- | --- | --- |
| Platform SaaS Admin | Có | Có |
| Platform SaaS Staff | Có | Không |
| Tenant Admin (MOTO/SAKI) | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Validate JWT token
2. Xác định admin user từ token context
3. Kiểm tra user thuộc nhóm Platform SaaS Admin
4. Kiểm tra quyền tenant.create
5. Từ chối request nếu user không đủ quyền (HTTP 403)
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

**Bảng dùng chung quản trị hệ thống: `platform.tenant_registry`**

| Request Field | Table | Column | Logic mapping / Giá trị |
| --- | --- | --- | --- |
| - | platform.tenant_registry | tenant_id | ULID tự phát sinh của tenant |
| - | platform.tenant_registry | tenant_type | `'saki'` |
| tenant_code | platform.tenant_registry | company_id | `tenant_code` |
| official_name_ja | platform.tenant_registry | company_name | `official_name_ja` |
| - | platform.tenant_registry | schema_name | System Generated: `tenant_saki_{tenant_code_lowercase}` |
| - | platform.tenant_registry | db_user | System Generated: `dbuser_saki_{tenant_code_lowercase}` |
| - | platform.tenant_registry | status | `1` (Hoạt động) |
| plan_code | platform.tenant_registry | plan_code | `plan_code` |
| tel | platform.tenant_registry | phone_number | `tel` |
| - | platform.tenant_registry | contracted_at | `now()` (Ngày ký hợp đồng mặc định) |

**Bảng thông tin doanh nghiệp trong schema riêng của Tenant: `tenant_db.mst_saki_company`**

| Request Field | Table | Column | Logic mapping / Giá trị |
| --- | --- | --- | --- |
| - | tenant_db.mst_saki_company | company_id | Tự phát sinh theo format: `SAKI-{4 chữ số tự tăng}` |
| official_name_ja | tenant_db.mst_saki_company | official_name_ja | `official_name_ja` |
| - | tenant_db.mst_saki_company | official_name_kana | `official_name_ja` chuyển sang Hiragana/Katakana nếu cần |
| display_name_ja | tenant_db.mst_saki_company | display_name_ja | `display_name_ja` |
| postal_code | tenant_db.mst_saki_company | postal_code | `postal_code` |
| address_ja | tenant_db.mst_saki_company | address_ja | `address_ja` |
| - | tenant_db.mst_saki_company | created_by | ID của Platform Admin thực hiện |
| - | tenant_db.mst_saki_company | updated_by | ID của Platform Admin thực hiện |

**Bảng tài khoản quản trị ban đầu của Tenant: `tenant_db.mst_saki_user`**

| Request Field | Table | Column | Logic mapping / Giá trị |
| --- | --- | --- | --- |
| - | tenant_db.mst_saki_user | user_id | Tự phát sinh mã người dùng (VARCHAR định danh / email hoặc ULID) |
| - | tenant_db.mst_saki_user | company_id | Liên kết khóa ngoại `company_id` vừa tạo phía trên |
| - | tenant_db.mst_saki_user | office_id | Mặc định khởi tạo văn phòng chính (ví dụ: `OFFICE-0001`) |
| - | tenant_db.mst_saki_user | department_id | Mặc định khởi tạo phòng ban chính (ví dụ: `DEPT-0001`) |
| admin_user.last_name_ja | tenant_db.mst_saki_user | last_name_ja | `admin_user.last_name_ja` |
| admin_user.first_name_ja | tenant_db.mst_saki_user | first_name_ja | `admin_user.first_name_ja` |
| - | tenant_db.mst_saki_user | last_name_kana | Mặc định rỗng hoặc convert |
| - | tenant_db.mst_saki_user | first_name_kana | Mặc định rỗng hoặc convert |
| admin_user.tel | tenant_db.mst_saki_user | tel | `admin_user.tel` |
| admin_user.email | tenant_db.mst_saki_user | email | `admin_user.email` |
| - | tenant_db.mst_saki_user | mail_language | `'ja'` |
| - | tenant_db.mst_saki_user | reference_scope | `5` (Toàn quyền hiển thị) |
| - | tenant_db.mst_saki_user | execution_role_id | Vai trò admin mặc định trong cấu hình phân quyền SAKI |
| - | tenant_db.mst_saki_user | status | `1` (Hoạt động) |
| - | tenant_db.mst_saki_user | created_by | ID của Platform Admin thực hiện |
| - | tenant_db.mst_saki_user | updated_by | ID của Platform Admin thực hiện |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | data.tenant_id | ID khóa chính tự sinh (ULID) |
| platform.tenant_registry | company_id | data.tenant_code | Mã Tenant định danh |
| platform.tenant_registry | company_name | data.company_name | Tên công ty chính thức |
| platform.tenant_registry | created_at | data.created_at | Thời điểm tạo hệ thống |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request body đầu vào | Ngoài transaction |
| 2 | Kiểm tra trùng lặp `tenant_code` hoặc `domain` | Ngoài transaction |
| 3 | Khởi tạo transaction trên Platform DB | Trong transaction |
| 4 | INSERT bản ghi vào `platform.tenant_registry` | Trong transaction |
| 5 | Tạo cơ sở dữ liệu riêng: Thực thi DDL tạo schema PostgreSQL và cấu hình quyền sử dụng cho `db_user` | Trong transaction |
| 6 | Chạy migrations để thiết lập cấu trúc bảng trong schema của tenant | Trong transaction |
| 7 | INSERT bản ghi thông tin công ty vào `mst_saki_company` của schema mới | Trong transaction |
| 8 | INSERT bản ghi tài khoản admin ban đầu vào `mst_saki_user` của schema mới | Trong transaction |
| 9 | INSERT ghi nhận audit log vào `platform.tenant_provision_log` | Trong transaction |
| 10 | Commit transaction trên Platform DB và Tenant DB | Trong transaction |
| 11 | Gửi email kích hoạt tài khoản qua Queue Job | Sau commit (Ngoại vi) |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | INSERT vào `platform.tenant_registry` thất bại | Rollback toàn bộ |
| 2 | Tạo schema hoặc phân quyền cho `db_user` lỗi | Rollback toàn bộ, drop schema vừa tạo nếu có |
| 3 | Chạy migrations tạo cấu trúc bảng trên schema mới lỗi | Rollback toàn bộ, drop schema vừa tạo |
| 4 | INSERT thông tin công ty hoặc tài khoản admin lỗi | Rollback toàn bộ, drop schema vừa tạo |
| 5 | Ghi nhận audit log lỗi | Rollback toàn bộ, drop schema vừa tạo |
| 6 | Có bất kỳ exception hệ thống nào phát sinh | Rollback toàn bộ DB thay đổi và xóa bỏ tài nguyên phát sinh |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| (Không tồn tại) | POST /admin/saki-tenants | 1 (Hoạt động) | Đăng ký thành công |

---

# 13. Sequence / Processing Flow

```
1. Client gửi request POST /api/v1/admin/saki-tenants chứa đầy đủ dữ liệu đăng ký.
2. Middleware thực hiện kiểm tra Authentication và xác định JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization): Platform SaaS Admin có quyền "tenant.create".
4. Controller nhận request và thực hiện validate tham số request body đầu vào:
   - Kiểm tra định dạng bắt buộc, chiều dài chuỗi, định dạng email, số điện thoại, mã bưu điện.
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm chi tiết lỗi validate.
5. Controller gọi Service thực hiện nghiệp vụ đăng ký trong Database Transaction:
   - Kiểm tra trùng lặp `tenant_code` hoặc `domain` trong bảng `platform.tenant_registry`.
     - Nếu đã tồn tại: Throw Exception -> Trả về HTTP 409 Conflict.
   - Bắt đầu transaction.
   - Cấp phát ULID mới và thực hiện INSERT vào bảng `platform.tenant_registry`.
   - Thực thi tạo schema PostgreSQL mới (`tenant_saki_{code}`) và tạo user DB riêng.
   - Chạy lệnh migration hệ thống tự động để tạo các bảng nghiệp vụ trong schema mới.
   - INSERT dữ liệu doanh nghiệp mới vào bảng `mst_saki_company` thuộc schema vừa tạo.
   - INSERT dữ liệu tài khoản quản trị viên ban đầu vào bảng `mst_saki_user` thuộc schema mới.
   - INSERT ghi nhận audit log chi tiết hành động vào bảng `platform.tenant_provision_log`.
6. Transaction được Commit thành công.
7. Đẩy Job ngầm gửi email kích hoạt tài khoản admin (chứa token kích hoạt) vào Queue System.
8. Controller trả về thông tin Tenant vừa tạo kèm HTTP 201 Created.
```

---

# 14. Notification

| Hạng mục | Nội dung |
| --- | --- |
| Có cần notification không | Có |
| Thời điểm gửi | Sau khi commit transaction tạo tenant thành công |
| Đối tượng nhận | Email quản trị viên của Tenant SAKI mới (`admin_user.email`) |
| Kênh gửi | Email tự động |
| Template ID | platform.email.saki_admin_activation |

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | CREATE |
| Bảng target | platform.tenant_registry |
| ID record target | tenant_id |
| Dữ liệu ghi log | Chi tiết thông tin Tenant được khởi tạo (Mã tenant, domain, plan_code, email quản trị) |
| Chính sách lưu trữ | Theo NFR |

---

# 16. Xử lý file

## 16.1 Upload

| Hạng mục | Nội dung |
| --- | --- |
| Có upload file không | Không |
| Max file size | Không áp dụng |
| Định dạng cho phép | Không áp dụng |
| Nơi lưu trữ | Không áp dụng |
| Virus Scan | Không áp dụng |
| Quy tắc đặt tên file | Không áp dụng |
| Bảng lưu metadata | Không áp dụng |

---

## 16.2 Download

| Hạng mục | Nội dung |
| --- | --- |
| Có download file không | Không |
| Permission Check | Không áp dụng |
| Phương thức download | Không áp dụng |
| Thời gian hết hạn | Không áp dụng |
| Audit Log | Không áp dụng |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token hợp lệ và phân quyền Platform SaaS Admin (`tenant.create`). |
| 2 | Tenant Isolation (Schema level) | Đảm bảo mỗi tenant được cấp phát một schema PostgreSQL riêng biệt để cách ly dữ liệu triệt để, không cho phép truy cập chéo. |
| 3 | Input Sanitization | Thực hiện làm sạch dữ liệu đầu vào và sử dụng parameterized query/bindings khi thực thi các lệnh DB để phòng tránh SQL Injection, đặc biệt tại bước tạo schema động. |
| 4 | Secure Email Token | Mã token kích hoạt tài khoản đính kèm trong email gửi cho admin tenant phải có thời gian hết hạn (ví dụ: 24 giờ) và được mã hóa bảo mật. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 2.0 giây (do phải chạy migrations động khi tạo schema) |
| Lượng truy cập dự kiến | Thấp (chỉ sử dụng bởi Platform Admin) |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính `tenant_id` (ULID), index duy nhất trên `company_id` và `domain` |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Có (Tiến trình gửi email kích hoạt được xử lý không đồng bộ qua hàng đợi hệ thống) |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Wireframe | PA-TEN-009 Create SAKI Tenant |
| Screen Detail Design | PA-TEN-009 |
| API liên quan | PA-TEN-007-API-01 Saki tenant list |
| ERD | Carima-Staffing Data Dictionary / ERD |
| Permission Matrix | Platform Portal Permission Matrix |
| Validation Rule | Section 7. Validation Rules |
| NFR | Performance Requirement |

---

# 20. Open Issues

Không có.

---

# 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Done |
| 2 | Request body đã được định nghĩa | Done |
| 3 | Response body đã được định nghĩa | Done |
| 4 | Error response đã được định nghĩa | Done |
| 5 | Validation rule đã được định nghĩa | Done |
| 6 | Business rule đã được định nghĩa | Done |
| 7 | Permission check đã được định nghĩa | Done |
| 8 | Tenant isolation rule đã được định nghĩa | Done (Tách schema độc lập) |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
