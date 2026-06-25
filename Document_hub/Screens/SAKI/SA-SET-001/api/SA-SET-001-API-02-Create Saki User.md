# SA-SET-001-API-02-Create Saki User

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Create Saki User |
| Description | Tạo mới tài khoản người dùng SAKI |
| Endpoint | /api/v1/saki/settings/users |
| Menu | Settings |
| Method | POST |
| Related Tables | tenant_db.mst_saki_user |
| Screen list | 企業マスタ SA-SET-001 SAKI Portal Company Master |
| System | SAKI Portal |
| 説明 | Empty |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design |
| API ID | SA-SET-001-API-02-Create Saki User |
| API Name | Create Saki User |
| Business Flow áp dụng | BF-001 Tenant setup flow / User management |
| Portal áp dụng | SAKI Portal |
| Screen ID liên quan | SA-SET-001 |
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
| v1.0 | 2026/06/23 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Saki Create User. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-SET-001-API-02-Create Saki User |
| Tên API | Create Saki User |
| Tên tiếng Nhật | ユーザー作成 |
| Business Flow liên quan | BF-001 Tenant setup flow / User management |
| Screen ID liên quan | SA-SET-001 |
| Chức năng liên quan | Thêm mới người dùng hệ thống SAKI Portal |
| Mục đích API | Đăng ký thông tin người dùng mới, thiết lập vai trò, phạm vi quyền hạn và gửi email kích hoạt tài khoản. |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/saki/settings/users |
| Authentication | Có |
| Authorization | Có (Yêu cầu quyền quản trị viên) |
| Phạm vi Tenant | Tenant được xác định từ Access Token / request context |
| Có Transaction không | Có |
| Xử lý file | Không |
| Response format | JSON |
| Trạng thái | VN Review |

## 3.2 Mô tả API

API này dùng để đăng ký tài khoản người dùng mới cho SAKI Portal.

Hệ thống xác thực token và kiểm tra quyền admin của người gửi yêu cầu. Khi các thông tin nhập hợp lệ, Backend thực hiện thêm mới bản ghi vào bảng mst_saki_user trong tenant DB, đồng thời tạo bản ghi scope phân quyền chi tiết (nếu có) và thông tin xác thực mật khẩu khởi tạo.

Nếu chọn gửi email mời, hệ thống sẽ đẩy tác vụ gửi email chứa liên kết đăng nhập và mật khẩu tạm thời vào hàng đợi (Queue) để xử lý ngầm.

## 3.3 Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Admin nhấn nút 「招待メール送信」 hoặc 「下書き保存」 |
| Actor | SAKI Admin |
| Trước khi gọi API | Admin đang ở màn hình Tạo người dùng và điền các trường dữ liệu |
| Sau khi gọi API thành công | Chuyển hướng về màn hình Danh sách người dùng SAKI (SA-SET-001) |
| Sau khi gọi API thất bại | Hiển thị thông báo lỗi cụ thể trên giao diện tạo |
| Thay đổi trạng thái liên quan | Bản ghi user mới được thêm vào DB |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/saki/settings/users
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept | Có | application/json | Dữ liệu response mong muốn |
| Authorization | Có | Bearer eyJhbGciOi... | Access Token của Admin đang đăng nhập |
| Accept-Language | Không | ja | Ngôn ngữ câu thông báo lỗi |

## 4.3 Path Parameters

Không có

## 4.4 Query Parameters

Không có

---

# 5. Request Body

## 5.1 Request Body JSON

```json
{
  "last_name_ja": "山田",
  "first_name_ja": "太郎",
  "last_name_kana": "ヤマダ",
  "first_name_kana": "タロウ",
  "user_id": "saki_user_09",
  "email": "yamada@saki-company.co.jp",
  "tel": "03-1234-5678",
  "mail_language": "ja",
  "company_id": "SAKI0000000000000001",
  "office_id": "OFF00000000000000001",
  "department_id": "DEP00000000000000001",
  "position_ja": "マネージャー",
  "attendance_approver1_id": "saki_admin_01",
  "attendance_approver2_id": null,
  "attendance_approver3_id": null,
  "execution_role_id": "ROLE_SAKI_STAFF",
  "reference_scope": 6,
  "custom_scopes": [
    {
      "office_id": "OFF00000000000000001",
      "department_id": "DEP00000000000000001"
    }
  ],
  "edit_on_approval_flg": 0,
  "view_group_company_flg": 0,
  "edit_contract_dept_flg": 0,
  "status": 1,
  "password_generation_type": 1,
  "password": null,
  "require_password_change_flg": 1,
  "two_factor_auth_flg": 0,
  "send_invitation_email": true,
  "email_subject": "【SAKI Portal】システムアカウント招待のご案内",
  "email_body": "SAKI Portalへようこそ。\nアカウントが作成されました。\nログインURL: https://portal.saki.jp/login"
}
```

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Default | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| last_name_ja | string | Có | - | Required, max 24 | Họ tiếng Nhật |
| first_name_ja | string | Có | - | Required, max 24 | Tên tiếng Nhật |
| last_name_kana | string | Có | - | Required, max 24 | Họ Katakana |
| first_name_kana | string | Có | - | Required, max 24 | Tên Katakana |
| user_id | string | Có | - | Required, max 100, unique | ID đăng nhập |
| email | string | Có | - | Required, email format, unique | Email liên hệ |
| tel | string | Có | - | Required, max 15 | Số điện thoại |
| mail_language | string | Không | ja | ja / en | Ngôn ngữ thông báo |
| company_id | string | Có | - | Required, max 20 | Mã công ty liên kết |
| office_id | string | Có | - | Required, max 20 | Mã sự nghiệp sở |
| department_id | string | Có | - | Required, max 20 | Mã phòng ban |
| position_ja | string | Không | - | Max 50 | Chức vụ tiếng Nhật |
| attendance_approver1_id | string | Không | - | Max 100 | Người duyệt chấm công 1 |
| attendance_approver2_id | string | Không | - | Max 100 | Người duyệt chấm công 2 |
| attendance_approver3_id | string | Không | - | Max 100 | Người duyệt chấm công 3 |
| execution_role_id | string | Có | - | Required, max 64 | Mã vai trò phân quyền |
| reference_scope | integer | Có | - | Required, 1-6 | Phạm vi tham chiếu dữ liệu |
| custom_scopes | array | Không | - | Array of objects | Danh sách scope chi tiết (nếu scope = 6) |
| custom_scopes[].office_id | string | Có | - | Required (nếu custom_scopes có truyền) | Mã sự nghiệp sở được phân quyền |
| custom_scopes[].department_id | string | Không | - | Max 20 | Mã phòng ban được phân quyền |
| edit_on_approval_flg | integer | Không | 0 | 0 / 1 | Cờ cho sửa thông tin khi duyệt |
| view_group_company_flg | integer | Không | 0 | 0 / 1 | Cờ tham chiếu nhóm công ty |
| edit_contract_dept_flg | integer | Không | 0 | 0 / 1 | Cờ cho sửa phòng ban hợp đồng |
| status | integer | Không | 1 | 0: vô hiệu, 1: hoạt động | Trạng thái tài khoản |
| password_generation_type | integer | Không | 1 | 1: tự sinh, 2: chỉ định | Cách thức đặt mật khẩu |
| password | string | Không | - | Bắt buộc nếu type = 2, min 8, max 255 | Mật khẩu do admin chỉ định |
| require_password_change_flg | integer | Không | 1 | 0 / 1 | Yêu cầu đổi mật khẩu ở lần sau |
| two_factor_auth_flg | integer | Không | 0 | 0 / 1 | Yêu cầu xác thực 2 yếu tố |
| send_invitation_email | boolean | Không | true | true / false | Gửi email mời ngay hay không |
| email_subject | string | Không | - | Required nếu send_invitation_email = true | Tiêu đề email mời |
| email_body | string | Không | - | Required nếu send_invitation_email = true | Nội dung email mời |

## 5.3 Ghi chú request

* Nếu password_generation_type = 1, Backend tự sinh ngẫu nhiên mật khẩu mạnh (chứa chữ hoa, chữ thường, chữ số, ký tự đặc biệt, độ dài tối thiểu 12 ký tự) và băm để lưu.
* Nếu reference_scope = 6, custom_scopes bắt buộc phải truyền ít nhất 1 bản ghi.

---

# 6. Response Definition

## 6.1 Success Response

HTTP Status: 201 Created

```json
{
  "success": true,
  "data": {
    "user_id": "saki_user_09",
    "company_id": "SAKI0000000000000001",
    "email": "yamada@saki-company.co.jp",
    "status": 1,
    "created_at": "2026-06-23T16:00:00+09:00"
  },
  "message": "ユーザーを作成しました"
}
```

## 6.2 Error Response - Validation Error

HTTP Status: 422 Unprocessable Entity

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "入力内容を確認してください",
    "details": [
      {
        "field": "user_id",
        "code": "DUPLICATE",
        "message": "ユーザーIDは既に登録されています"
      },
      {
        "field": "email",
        "code": "DUPLICATE",
        "message": "メールアドレスは既に登録されています"
      }
    ]
  }
}
```
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


# 7. Validation Rules

[Message list](https://app.notion.com/p/sokucom/Message-list-374f02c407dd8037808eea01e93be8aa?source=copy_link) 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | user_id | required | - | required |
| 2 | user_id | max | 100 | max |
| 3 | user_id | string | - | string |
| 4 | user_id | unique | - | unique |
| 5 | last_name_ja | required | - | required |
| 6 | last_name_ja | max | 24 | max |
| 7 | first_name_ja | required | - | required |
| 8 | first_name_ja | max | 24 | max |
| 9 | last_name_kana | required | - | required |
| 10 | last_name_kana | regex | /^[ァ-ヶー]+$/u | regex |
| 11 | last_name_kana | max | 24 | max |
| 12 | first_name_kana | required | - | required |
| 13 | first_name_kana | regex | /^[ァ-ヶー]+$/u | regex |
| 14 | first_name_kana | max | 24 | max |
| 15 | email | required | - | required |
| 16 | email | email | - | email |
| 17 | email | unique | - | unique |
| 18 | tel | required | - | required |
| 19 | tel | regex | /^[0-9-]+$/ | regex |
| 20 | tel | max | 15 | max |
| 21 | office_id | required | - | required |
| 22 | office_id | exists | mst_saki_office,office_id | exists |
| 23 | department_id | required | - | required |
| 24 | department_id | exists | mst_saki_department,department_id | exists |
| 25 | reference_scope | required | - | required |
| 26 | reference_scope | in | 1, 2, 3, 4, 5, 6 | in |
| 27 | custom_scopes | required | - | required |
| 28 | password | required | - | required |
| 29 | password | min | 8 | min |
| 30 | password | max | 255 | max |
| 31 | password | regex | /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]+$/ | regex |

## 7.1 Validation xử lý trước Business Rule

| Step | Nội dung |
| --- | --- |
| 1 | Kiểm tra Authorization Header (Token hợp lệ và chưa hết hạn) |
| 2 | Kiểm tra Content-Type là application/json |
| 3 | Parse JSON request body |
| 4 | Validate các trường bắt buộc và định dạng (User ID, Email, Tel...) |
| 5 | Trim khoảng trắng đối với user_id |
| 6 | Chuyển sang xử lý Business Rule |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Authorization Check | Chỉ người dùng có quyền quản trị user (như SAKI Admin) mới được phép gọi API này |
| BR-002 | Tenant Isolation | Người dùng mới bắt buộc phải được tạo trong đúng tenant schema của Admin đang gọi API |
| BR-003 | User ID Uniqueness | user_id phải là duy nhất, không được trùng lặp trong bảng mst_saki_user |
| BR-004 | Email Uniqueness | email phải là duy nhất, không được trùng lặp trong bảng mst_saki_user |
| BR-005 | Password Generation | Nếu password_generation_type = 1, hệ thống tự động sinh ngẫu nhiên mật khẩu mạnh. Mật khẩu thô tuyệt đối không được ghi log |
| BR-006 | Password Hashing | Mật khẩu phải được băm bằng thuật toán an toàn (Bcrypt) trước khi lưu vào bảng xác thực tài khoản |
| BR-007 | Scope Synchronization | Nếu reference_scope = 6, ghi nhận danh sách chi tiết vào mst_saki_user_scope. Nếu khác 6, không tạo bản ghi scope liên quan |
| BR-008 | Queue Invitation Mail | Nếu send_invitation_email = true, đẩy tác vụ gửi mail mời kèm thông tin đăng nhập tạm thời vào hàng đợi (Queue Job) để thực hiện bất đồng bộ |
| BR-009 | Sensitive Data Masking | Không trả mật khẩu thô hoặc thông tin băm mật khẩu ra Success Response |

---

# 9. Permission / Authorization

## 9.1 Authentication

| Hạng mục | Nội dung |
| --- | --- |
| API yêu cầu Access Token không | Có |
| Token type | Bearer |

## 9.2 Authorization

| Role | Cho phép tạo SAKI User | Ghi chú |
| --- | --- | --- |
| SAKI Admin | Có | Chỉ được tạo user trong cùng Tenant của mình |
| SAKI User / Staff | Không | Không có quyền quản trị user |
| MOTO Portal Users | Không | Không thuộc scope phân quyền của SAKI Portal |
| Platform Admin | Không | Cổng quản trị Platform Admin riêng biệt |

## 9.3 Logic kiểm tra quyền

| Step | Nội dung |
| --- | --- |
| 1 | Xác thực token hiện tại, lấy tenant_id và user_id của actor |
| 2 | Truy cập tenant schema tương ứng |
| 3 | Truy vấn bảng mst_saki_user để lấy vai trò của actor |
| 4 | Kiểm tra vai trò có quyền quản lý user (ROLE_SAKI_ADMIN) |
| 5 | Cho phép thực thi nếu hợp lệ, ngược lại trả về 403 Forbidden |

---

# 10. Data Mapping

## 10.1 Request To DB Mapping

| Request Field | DB | Table | Column | Process | Mô tả |
| --- | --- | --- | --- | --- | --- |
| user_id | tenant_db | mst_saki_user | user_id | INSERT | ID người dùng |
| company_id | tenant_db | mst_saki_user | company_id | INSERT | Mã công ty |
| office_id | tenant_db | mst_saki_user | office_id | INSERT | Mã sự nghiệp sở |
| department_id | tenant_db | mst_saki_user | department_id | INSERT | Mã phòng ban |
| last_name_ja | tenant_db | mst_saki_user | last_name_ja | INSERT | Họ tiếng Nhật |
| first_name_ja | tenant_db | mst_saki_user | first_name_ja | INSERT | Tên tiếng Nhật |
| last_name_kana | tenant_db | mst_saki_user | last_name_kana | INSERT | Họ Katakana |
| first_name_kana | tenant_db | mst_saki_user | first_name_kana | INSERT | Tên Katakana |
| email | tenant_db | mst_saki_user | email | INSERT | Email |
| tel | tenant_db | mst_saki_user | tel | INSERT | Điện thoại |
| mail_language | tenant_db | mst_saki_user | mail_language | INSERT | Ngôn ngữ thông báo |
| execution_role_id | tenant_db | mst_saki_user | execution_role_id | INSERT | Vai trò phân quyền |
| reference_scope | tenant_db | mst_saki_user | reference_scope | INSERT | Phạm vi tham chiếu dữ liệu |
| edit_on_approval_flg | tenant_db | mst_saki_user | edit_on_approval_flg | INSERT | Cờ sửa khi duyệt |
| view_group_company_flg | tenant_db | mst_saki_user | view_group_company_flg | INSERT | Cờ xem thông tin nhóm |
| edit_contract_dept_flg | tenant_db | mst_saki_user | edit_contract_dept_flg | INSERT | Cờ sửa bộ phận hợp đồng |
| status | tenant_db | mst_saki_user | status | INSERT | Trạng thái hoạt động |
| password (hashed) | tenant_db | TBD: saki_user_credential | password_hash | INSERT | Hash mật khẩu đăng nhập |
| require_password_change_flg | tenant_db | TBD: saki_user_credential | require_change_flg | INSERT | Bắt buộc đổi mật khẩu lần đầu |
| two_factor_auth_flg | tenant_db | TBD: saki_user_credential | two_factor_flg | INSERT | Bắt buộc xác thực 2 yếu tố |

## 10.2 Custom Scope Mapping (nếu reference_scope = 6)

| Request Field | DB | Table | Column | Value | Mô tả |
| --- | --- | --- | --- | --- | --- |
| user_id | tenant_db | mst_saki_user_scope | user_id | Request.user_id | Liên kết người dùng |
| custom_scopes[].office_id | tenant_db | mst_saki_user_scope | office_id | custom_scopes[].office_id | Sự nghiệp sở phân quyền |
| custom_scopes[].department_id | tenant_db | mst_saki_user_scope | department_id | custom_scopes[].department_id | Phòng ban phân quyền |

## 10.3 DB To Response Mapping

| DB | Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- | --- |
| tenant_db | mst_saki_user | user_id | data.user_id | ID người dùng vừa tạo |
| tenant_db | mst_saki_user | company_id | data.company_id | Mã công ty liên kết |
| tenant_db | mst_saki_user | email | data.email | Email liên hệ |
| tenant_db | mst_saki_user | status | data.status | Trạng thái tài khoản |
| tenant_db | mst_saki_user | created_at | data.created_at | Thời điểm tạo |

## 10.4 Index cần thiết

| DB | Table | Index Name | Column | Mục đích |
| --- | --- | --- | --- | --- |
| tenant_db | mst_saki_user | idx_saki_user_unique | user_id | Check trùng lặp ID nhanh chóng |
| tenant_db | mst_saki_user | idx_saki_user_email | email | Check trùng lặp Email nhanh chóng |
| tenant_db | mst_saki_user_scope | idx_saki_user_scope_user | user_id | Truy vấn danh sách scope liên quan |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction khi tạo mới thành công

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Xác định Tenant & Validate request body | Ngoài transaction |
| 2 | Kiểm tra quyền của Admin và check trùng lặp ID/Email | Ngoài transaction |
| 3 | Bắt đầu Database Transaction | Trong transaction |
| 4 | Insert thông tin cơ bản vào mst_saki_user | Trong transaction |
| 5 | Insert thông tin xác thực/bảo mật vào saki_user_credential | Trong transaction |
| 6 | Insert custom scopes vào mst_saki_user_scope (nếu reference_scope = 6) | Trong transaction |
| 7 | Ghi nhận Audit Log thành công | Trong transaction |
| 8 | Commit Transaction | Trong transaction |
| 9 | Trigger Event đẩy job gửi email mời vào hàng đợi (Queue) | Ngoài transaction |

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Thêm mới bản ghi mst_saki_user thất bại | Rollback toàn bộ transaction |
| 2 | Thêm mới bản ghi saki_user_credential thất bại | Rollback toàn bộ transaction |
| 3 | Thêm mới bản ghi mst_saki_user_scope thất bại | Rollback toàn bộ transaction |
| 4 | Ghi Audit log tạo mới thất bại | Rollback toàn bộ transaction |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Non-existent (Chưa tồn tại) | Đăng ký thành công | Active / Inactive | status được chỉ định (mặc định 1: hoạt động) |

---

# 13. Sequence / Processing Flow

## 13.1 Create User Success Flow

1. Frontend gửi request tạo mới user.
2. Backend xác thực token của Admin và xác định tenant DB.
3. Validate request data và kiểm tra tính duy nhất của user_id / email.
4. Hệ thống tự động sinh mật khẩu tạm thời (nếu chọn tự sinh).
5. Khởi tạo Database Transaction:
   - Insert bản ghi vào mst_saki_user.
   - Băm mật khẩu và insert thông tin vào bảng credential.
   - Insert custom scopes vào mst_saki_user_scope (nếu reference_scope = 6).
   - Ghi nhận Audit Log tạo mới thành công.
6. Commit Transaction thành công.
7. Đẩy job gửi email mời kích hoạt tài khoản vào hàng đợi (Queue Job).
8. Backend trả về response thành công HTTP 201 Created.

## 13.2 Create User Failed Flow

1. Frontend gửi request tạo mới user.
2. Backend validate thông tin nhập thất bại hoặc trùng lặp dữ liệu.
3. Hoặc transaction ghi dữ liệu vào cơ sở dữ liệu gặp sự cố kỹ thuật.
4. Hệ thống tự động rollback toàn bộ dữ liệu đã ghi tạm thời (nếu đã mở transaction).
5. Trả về Error Response chi tiết (HTTP 422/403/500).

---

# 14. Notification

| Hạng mục | Nội dung |
| --- | --- |
| Có cần notification không | Có (khi send_invitation_email = true) |
| Kênh gửi | Email |
| Đối tượng nhận | Email của user được tạo |
| Thời điểm gửi | Bất đồng bộ qua Queue sau khi commit transaction thành công |

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | CREATE |
| Bảng target | mst_saki_user |
| ID record target | user_id |
| Dữ liệu ghi log | user_id, email, role, status, actor_admin_id, ip_address |
| Chính sách lưu trữ | Ghi vào bảng log hoạt động chung của tenant schema |

---

# 16. Xử lý file

Không áp dụng.

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Mật khẩu an toàn | Mật khẩu khởi tạo tự sinh bắt buộc phải có độ phức tạp cao, mật khẩu thô không được ghi log, lưu băm Bcrypt. |
| 2 | Phân quyền truy cập | Bắt buộc kiểm tra quyền quản trị của actor đối với tenant schema, không cho phép can thiệp chéo tenant. |
| 3 | SQL Injection Prevention | Sử dụng Eloquent ORM / Parameterized Queries khi thao tác cơ sở dữ liệu. |
| 4 | Masking Sensitive Data | Không ghi mật khẩu hay mã băm mật khẩu ra file log hệ thống hoặc audit log. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian phản hồi | Dưới 2 giây |
| Xử lý email | Bắt buộc đẩy vào Queue xử lý bất đồng bộ để tránh nghẽn luồng HTTP Request. |
| Index cần thiết | Index theo user_id, email trong bảng mst_saki_user để kiểm tra trùng lặp nhanh. |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 Tenant setup flow / User management |
| Wireframe | SA-SET-001 SAKI Portal User Management |
| Screen Detail Design | SA-SET-001 |
| ERD | Carima-Staffing Data Dictionary / ERD|
| Permission Matrix | Portal Permission Matrix |
| Validation Rule | Validation Rules |
| NFR | Performance Requirement |
| API Specification | SA-SET-001-API-01-Saki user list |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| 1 | Chưa xác định bảng credential chính thức dành cho saki_user trong Data Dictionary (hiện đang giả định tên là saki_user_credential). | Low | PM / SA | TBD | Open |

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
| 8 | Tenant isolation rule đã được định nghĩa | Done |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done |
| 14 | File handling rule đã được định nghĩa nếu cần | Done |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
