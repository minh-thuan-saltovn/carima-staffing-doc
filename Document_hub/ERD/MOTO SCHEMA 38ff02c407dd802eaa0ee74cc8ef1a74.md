# MOTO SCHEMA

## Nhóm 1: Master Cơ cấu Tổ chức MOTO (Organization)

### `tmp_moto_setup_credential` (Khởi tạo tài khoản MOTO lần đầu)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope trỏ về Platform |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_user(id)` |
| `token_hash` | VARCHAR(255) | No |  | **UK** | Token xác thực (Hash) |
| `expires_at` | TIMESTAMP | No |  | IDX | Ngày giờ hết hạn Token |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm credential đã được sử dụng |
| `revoked_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm credential bị thu hồi |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, USED, EXPIRED, REVOKED |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `mst_moto_company` (Thông tin pháp nhân MOTO & Mã số thuế)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_code` | VARCHAR(50) | No |  | **UK** | Mã công ty (Agency Code) |
| `company_name` | VARCHAR(255) | No |  |  | Tên chính thức in trên hợp đồng (正式企業名). Đồng nhất với SAKI/Platform (VARCHAR 255 — pháp nhân JP có thể dài: VD "株式会社○○○ホールディングスマネジメントジャパン") |
| `company_name_kana` | VARCHAR(255) | No |  |  | Tên chính thức Katakana (正式企業名（カナ）) |
| `company_name_en` | VARCHAR(255) | Yes | NULL |  | Tên chính thức tiếng Anh (正式企業名（英字）) |
| `display_name` | VARCHAR(100) | No |  |  | Tên hiển thị trên hệ thống UI (システム表示企業名) |
| `display_name_en` | VARCHAR(100) | Yes | NULL |  | Tên hiển thị tiếng Anh (システム表示企業名（英字）) |
| `representative_name` | VARCHAR(100) | Yes | NULL |  | Tên người đại diện pháp luật |
| `postal_code` | VARCHAR(8) | Yes | NULL |  | Mã bưu chính (郵便番号) |
| `address` | VARCHAR(255) | Yes | NULL |  | Trụ sở chính (住所1) |
| `dispatch_license_no` | VARCHAR(50) | Yes | NULL |  | Giấy phép phái cử lao động (VD: 派13-300000) |
| `invoice_registration_no` | VARCHAR(50) | Yes | NULL |  | Mã số đăng ký Hóa đơn đủ điều kiện (適格請求書発行事業者登録番号 - T+13 số) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `mst_moto_office` (Văn phòng / Chi nhánh của MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `office_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã chi nhánh |
| `office_name` | VARCHAR(100) | No |  |  | Tên chính thức (正式事業所名) |
| `office_name_en` | VARCHAR(100) | Yes | NULL |  | Tên chính thức tiếng Anh (正式事業所名（英字）) |
| `display_name` | VARCHAR(100) | No |  |  | Tên hiển thị hệ thống (システム表示事業所名) |
| `display_name_en` | VARCHAR(100) | Yes | NULL |  | Tên hiển thị tiếng Anh (システム表示事業所名（英字）) |
| `dispatch_license_no` | VARCHAR(50) | Yes | NULL |  | Giấy phép phái cử riêng theo office (khi 事業所ごとに派遣許可番号が異なる) |
| `address` | VARCHAR(255) | Yes | NULL |  | Địa chỉ văn phòng |
| `status` | SMALLINT | No | 1 | IDX | 1: Đang hoạt động, 0: Ngưng |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_office_code: (company_id, office_code) WHERE deleted_at IS NULL

### `mst_moto_department` (Phòng ban của MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `office_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu `mst_moto_office(id)` (NULL nếu thuộc tổng công ty) |
| `department_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã phòng ban |
| `department_name` | VARCHAR(100) | No |  |  | Tên chính thức (正式部署名) |
| `department_name_en` | VARCHAR(100) | Yes | NULL |  | Tên chính thức tiếng Anh (正式部署名（英字）) |
| `display_name` | VARCHAR(100) | No |  |  | Tên hiển thị hệ thống (システム表示部署名) |
| `display_name_en` | VARCHAR(100) | Yes | NULL |  | Tên hiển thị tiếng Anh (システム表示部署名（英字）) |
| `phone_number` | VARCHAR(50) | Yes | NULL |  | Số điện thoại phòng ban (電話番号) |
| `status` | SMALLINT | No | 1 | IDX | 1: Đang hoạt động, 0: Ngưng |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_dept_code: (company_id, department_code) WHERE deleted_at IS NULL

---

### `mst_moto_company_contact` (Thông tin liên hệ của công ty MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contact_name` | VARCHAR(100) | No |  |  | Tên người/bộ phận liên hệ |
| `department_name` | VARCHAR(100) | Yes | NULL |  | Tên phòng ban liên hệ |
| `email` | VARCHAR(255) | Yes | NULL |  | Email liên hệ chính |
| `phone_number` | VARCHAR(50) | Yes | NULL |  | Số điện thoại |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `mst_moto_contract_company` (Pháp nhân / đơn vị đại diện ký hợp đồng phía MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_company_name` | VARCHAR(255) | No |  |  | Tên pháp nhân đại diện ký (đồng nhất với company_name VARCHAR(255)) |
| `representative_name` | VARCHAR(100) | Yes | NULL |  | Tên người đại diện pháp luật |
| `address` | VARCHAR(255) | Yes | NULL |  | Địa chỉ pháp nhân |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 2: Người dùng, Phân quyền MOTO (Auth & Roles)

## Nhóm 2: Người dùng, Phân quyền MOTO (Auth & Roles)

### `mst_moto_permission` (Danh mục Quyền hệ thống MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `permission_key` | VARCHAR(100) | No |  | **UK** | Mã quyền (VD: `moto.invoice.edit`) |
| `parent_permission_id` | BIGINT | Yes | NULL | FK, IDX | Quyền cha để dựng cây menu/chức năng |
| `permission_type` | VARCHAR(20) | No | 'ACTION' | IDX | MENU, ACTION, DATA_SCOPE, LOGIN_ACCESS |
| `display_order` | INTEGER | No | 0 |  | Thứ tự hiển thị khi import/dựng màn hình phân quyền |
| `external_permission_no` | VARCHAR(50) | Yes | NULL | IDX | Mã/No quyền tương ứng e-staffing nếu cần mapping/import |
| `description` | VARCHAR(255) | Yes | NULL |  | Mô tả quyền |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `mst_moto_role` (Vai trò người dùng MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `role_name` | VARCHAR(100) | No |  |  | Tên vai trò |
| `is_system` | SMALLINT | No | 0 |  | 1: Role mặc định không được xóa |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_role_name: (company_id, role_name) WHERE deleted_at IS NULL

### `mst_moto_permission_role` (Bảng Pivot phân quyền)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `role_id` | BIGINT | No |  | **PK**, FK | Tham chiếu `mst_moto_role(id)` |
| `permission_id` | BIGINT | No |  | **PK**, FK | Tham chiếu `mst_moto_permission(id)` |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `mst_moto_user` (Tài khoản Nhân viên nội bộ MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `login_id` | VARCHAR(100) | No |  | **UNIQUE** uq_moto_login_id: (company_id, login_id) | Tên đăng nhập |
| `password_hash` | VARCHAR(255) | No |  |  | Mật khẩu mã hóa |
| `role_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_role(id)` |
| `office_id` | BIGINT | Yes | NULL | FK, IDX | Thuộc chi nhánh nào |
| `department_id` | BIGINT | Yes | NULL | FK, IDX | Thuộc phòng ban nào |
| `full_name` | VARCHAR(100) | No |  |  | Họ và tên (ユーザー氏名) |
| `full_name_kana` | VARCHAR(100) | No |  |  | Họ tên Katakana (ユーザー氏名（カナ）) |
| `email` | VARCHAR(255) | Yes | NULL |  | Email liên hệ |
| `notification_email_lang` | VARCHAR(10) | No | 'ja' |  | Ngôn ngữ email thông báo hệ thống (システムからの通知メール言語); bắt buộc theo 企業設定-派遣元 |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Blocked |
| `auth_provider` | VARCHAR(50) | No | 'LOCAL' | IDX | LOCAL, AZURE_AD, OKTA (Chuẩn bị cho SSO) |
| `sso_id` | VARCHAR(255) | Yes | NULL | IDX | ID định danh từ hệ thống SSO (Chuẩn bị cho SSO) |
| `login_fail_count` | SMALLINT | No | 0 |  | Số lần đăng nhập sai liên tiếp; lock sau 3 lần theo policy mặc định |
| `account_locked_flg` | SMALLINT | No | 0 | IDX | 1: Tài khoản đang bị khóa |
| `locked_at` | TIMESTAMP | Yes | NULL |  | Thời điểm bị khóa gần nhất |
| `locked_until` | TIMESTAMP | Yes | NULL | IDX | Thời điểm tự mở khóa nếu áp dụng temporary lock |
| `pwd_change_required_flg` | SMALLINT | No | 0 |  | 1: Bắt buộc đổi mật khẩu ở lần đăng nhập tiếp theo |
| `last_login_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm đăng nhập thành công gần nhất |
| `password_changed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm đổi/reset mật khẩu gần nhất |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_moto_user_2fa` (Trạng thái xác thực 2 bước của User MOTO)

> 2FA email-code + backup code (tham khảo 2段階認証マニュアル). Ngưỡng số lần/thời lượng khóa là tham số cấu hình (tài liệu dùng "一定"). Khóa email-code theo thời gian (`auth_code_locked_until`); khóa backup-code tới khi update (`backup_code_locked_flg`).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `mst_moto_user(id)` (1:1) |
| `two_factor_enabled_flg` | SMALLINT | No | 0 | IDX | 1: User đã bật 2FA (opt-in, cần đăng ký trước) |
| `email_for_2fa` | VARCHAR(255) | Yes | NULL |  | Email nhận mã xác thực |
| `auth_code_hash` | VARCHAR(255) | Yes | NULL |  | Hash mã xác thực email hiện hành |
| `auth_code_expires_at` | TIMESTAMP | Yes | NULL |  | Hết hạn mã email |
| `auth_code_attempt_count` | SMALLINT | No | 0 |  | Số lần nhập sai mã email |
| `auth_code_resend_count` | SMALLINT | No | 0 |  | Số lần gửi lại mã email |
| `auth_code_locked_until` | TIMESTAMP | Yes | NULL | IDX | Khóa luồng email-code theo thời gian (tự mở) |
| `backup_code_locked_flg` | SMALLINT | No | 0 |  | 1: Khóa luồng backup code tới khi user/admin update |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **RLS policy** — p_moto_2fa_tenant: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_moto_2fa_backup_code` (Backup code 2FA của User MOTO)

> Mỗi backup code dùng một lần (`used_at`); sau khi dùng tự làm mới cả bộ. User tự làm mới bất kỳ lúc nào; admin chỉ làm mới khi đang khóa.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_user(id)` |
| `code_hash` | VARCHAR(255) | No |  | IDX | Hash của backup code |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm code đã dùng (NULL = còn hiệu lực) |
| `generation` | INTEGER | No | 1 |  | Phiên bản bộ backup code (tăng mỗi lần làm mới) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **RLS policy** — p_moto_2fa_backup_tenant: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `mst_moto_user_scope` (Cấu hình Phạm vi dữ liệu của User MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_user(id)` |
| `scope_type` | VARCHAR(50) | No |  |  | Loại: OFFICE, DEPT, CLIENT, ALL |
| `target_id` | BIGINT | Yes | NULL | IDX | ID đối tượng (nếu scope là cụ thể) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `tmp_moto_pwd_reset_token` (Token quên mật khẩu User MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_user(id)` |
| `token_hash` | VARCHAR(255) | No |  | **UK** | Token băm |
| `expires_at` | TIMESTAMP | No |  | IDX | Ngày hết hạn |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token đã được sử dụng |
| `revoked_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token bị thu hồi/hủy |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, USED, EXPIRED, REVOKED |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 3: Thông tin Khách hàng SAKI (Cross-Tenant Mapping)

### `cfg_moto_client_company` (Master Khách hàng SAKI đang giao dịch)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Tham chiếu ID Tenant SAKI tại Platform |
| `saki_company_code` | VARCHAR(50) | No |  | **Logical FK** , IDX | Mã Business Code SAKI. Cốt lõi RLS. |
| `client_code_internal` | VARCHAR(50) | Yes | NULL |  | Mã khách hàng nội bộ do MOTO tự quy định |
| `approver_domain_payload` | JSONB | Yes | NULL |  | Danh sách domain email hợp lệ cho 勤怠承認者 của khách hàng này (ドメインマスタ); chặn set người duyệt ngoài domain |
| `status` | SMALLINT | No | 1 | IDX | 1: Đang giao dịch, 0: Dừng |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_client_saki_tenant: (company_id, saki_tenant_id) WHERE deleted_at IS NULL (Note: Application Rule BẮT BUỘC ưu tiên Restore bản ghi cũ thay vì tạo mới)

### `cfg_moto_client_company_user` (Phân công người phụ trách Khách hàng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `client_id` | BIGINT | No |  | FK, IDX | Tham chiếu `cfg_moto_client_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_user(id)` được giao chăm sóc khách hàng này |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 4: Quản lý Nhân sự Phái cử (Staff - Core Profile)

### `mst_staff` (Hồ sơ Nhân viên Phái cử - Trọng tâm hệ thống)

*Cập nhật trạng thái Staff theo `スタッフステータス.csv`*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã nhân viên e-staffing |
| `last_name` | VARCHAR(50) | No |  |  | Họ |
| `first_name` | VARCHAR(50) | No |  |  | Tên |
| `kana_last_name` | VARCHAR(50) | No |  |  | Họ (Katakana) |
| `kana_first_name` | VARCHAR(50) | No |  |  | Tên (Katakana) |
| `gender` | SMALLINT | No |  |  | 1: Nam, 2: Nữ, 3: Khác |
| `birth_date` | DATE | Yes | NULL |  | Ngày sinh (Phục vụ check độ tuổi lao động) |
| `employment_status` | SMALLINT | No | 1 | IDX | 1: Hợp đồng có thời hạn, 2: Vô thời hạn (無期雇用) |
| `work_status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE (稼働中), INACTIVE (辞退/抹消), PENDING |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_staff_code: (company_id, staff_code) WHERE deleted_at IS NULL

### `mst_staff_skill` (Kỹ năng của Nhân viên - Phục vụ Matching)

*Phục vụ matching với yêu cầu từ `t_dispatch_inquiry` của SAKI.*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `global_skill_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | Tham chiếu logic `global_master.mst_skill(id)` trong cùng DB environment |
| `global_skill_code` | VARCHAR(50) | No |  | IDX | Stable code tham chiếu `global_master.mst_skill.skill_code`, dùng cho export/import/cross-environment |
| `skill_level` | SMALLINT | No |  |  | 1: Sơ cấp, 2: Trung cấp, 3: Cao cấp |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_staff_skill: (company_id, staff_id, global_skill_code) WHERE deleted_at IS NULL

### `mst_staff_qualification` (Chứng chỉ/Bằng cấp Pháp lý của Staff)

*Bảng mới phát hiện từ `資格マスター.csv` (Ví dụ: Kế toán, Y tá, IT)*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `global_qualification_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | Tham chiếu logic `global_master.mst_qualification(id)` trong cùng DB environment |
| `global_qualification_code` | VARCHAR(50) | No |  | IDX | Stable code tham chiếu `global_master.mst_qualification.qualification_code`, dùng cho export/import/cross-environment |
| `acquired_date` | DATE | Yes | NULL |  | Ngày lấy chứng chỉ |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_staff_qualification: (company_id, staff_id, global_qualification_code) WHERE deleted_at IS NULL

### `mst_staff_auth_credential` (Tài khoản Staff Web)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, **UK** | Quan hệ 1:1 với `mst_staff` |
| `login_id` | VARCHAR(100) | No |  | Composite UK | ID đăng nhập Staff Web |
| `password_hash` | VARCHAR(255) | No |  |  | Mật khẩu đã mã hóa |
| `email` | VARCHAR(255) | Yes | NULL | IDX | Email đăng ký Staff Web; nếu có thì được dùng cho forgot password OTP/email |
| `email_verified_at` | TIMESTAMP | Yes | NULL |  | Thời điểm xác minh email |
| `account_locked_flg` | SMALLINT | No | 0 |  | 1: Bị khóa do đăng nhập sai |
| `login_fail_count` | SMALLINT | No | 0 |  | Số lần nhập sai liên tiếp |
| `locked_at` | TIMESTAMP | Yes | NULL |  | Thời điểm bị khóa gần nhất |
| `locked_until` | TIMESTAMP | Yes | NULL | IDX | Thời điểm tự mở khóa nếu áp dụng temporary lock |
| `pwd_change_required_flg` | SMALLINT | No | 0 |  | 1: Bắt buộc đổi mật khẩu ở lần đăng nhập tiếp theo |
| `last_login_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm đăng nhập thành công gần nhất |
| `password_changed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm đổi/reset mật khẩu gần nhất |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_staff_credential_login: (company_id, login_id)

### `tmp_staff_pwd_reset_token` (Token quên mật khẩu Staff - nhánh có Email)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_credential_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff_auth_credential(id)` |
| `token_hash` | VARCHAR(255) | No |  | **UK** | Token/OTP đã băm |
| `expires_at` | TIMESTAMP | No |  | IDX | Hạn sử dụng token |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token đã được sử dụng |
| `revoked_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token bị thu hồi/hủy |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, USED, EXPIRED, REVOKED |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_staff_password_reset_request` (Yêu cầu reset mật khẩu Staff - nhánh không có Email)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_credential_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff_auth_credential(id)` |
| `request_status` | VARCHAR(20) | No | 'REQUESTED' | IDX | REQUESTED, APPROVED, REJECTED, COMPLETED, CANCELLED |
| `request_reason` | TEXT | Yes | NULL |  | Nội dung Staff nhập hoặc ghi chú CS/admin |
| `handled_by_user_id` | BIGINT | Yes | NULL | FK, IDX | User MOTO xử lý reset thủ công |
| `handled_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xử lý |
| `admin_note` | TEXT | Yes | NULL |  | Ghi chú của MOTO admin khi reset/reject |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_moto_login_history` (Lịch sử đăng nhập User MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu `mst_moto_user(id)`; NULL nếu login_id không tồn tại |
| `login_id` | VARCHAR(100) | No |  | IDX | Login ID được nhập |
| `auth_provider` | VARCHAR(50) | No | 'LOCAL' | IDX | LOCAL, AZURE_AD, OKTA |
| `result` | VARCHAR(20) | No |  | IDX | SUCCESS, FAILED, LOCKED, MFA_REQUIRED, MFA_FAILED |
| `failure_reason` | VARCHAR(100) | Yes | NULL |  | PASSWORD_MISMATCH, USER_NOT_FOUND, ACCOUNT_LOCKED, TOKEN_EXPIRED... |
| `ip_address` | VARCHAR(45) | Yes | NULL | IDX | IPv4/IPv6 của client hoặc gateway |
| `user_agent` | TEXT | Yes | NULL |  | User-Agent tại thời điểm đăng nhập |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm ghi nhận login attempt |

### `t_staff_login_history` (Lịch sử đăng nhập Staff Web)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_credential_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu `mst_staff_auth_credential(id)`; NULL nếu login_id không tồn tại |
| `login_id` | VARCHAR(100) | No |  | IDX | Login ID được nhập |
| `result` | VARCHAR(20) | No |  | IDX | SUCCESS, FAILED, LOCKED |
| `failure_reason` | VARCHAR(100) | Yes | NULL |  | PASSWORD_MISMATCH, STAFF_NOT_FOUND, ACCOUNT_LOCKED, TOKEN_EXPIRED... |
| `ip_address` | VARCHAR(45) | Yes | NULL | IDX | IPv4/IPv6 của client hoặc gateway |
| `user_agent` | TEXT | Yes | NULL |  | User-Agent tại thời điểm đăng nhập |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm ghi nhận login attempt |

### `t_moto_auth_session` (Phiên đăng nhập / Refresh Token User MOTO)

> BF-M01 enterprise short session policy: DB là source of truth, Redis chỉ dùng cache/denylist/rate-limit. Access token JWT TTL mặc định 15 phút; refresh token absolute lifetime mặc định 8 giờ; idle timeout mặc định 60 phút; không có remember-me.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global tenant_id từ Platform |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `session_id` | UUID | No |  | **UK** | ID phiên đưa vào JWT claim để revoke/cache |
| `token_family_id` | UUID | No |  | IDX | Nhóm refresh token rotation; reuse token cũ sẽ revoke cả family |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_user(id)` |
| `refresh_token_hash` | VARCHAR(255) | No |  | **UK** | Hash của refresh token; tuyệt đối không lưu raw token |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, ROTATED, REVOKED, EXPIRED |
| `issued_at` | TIMESTAMP | No | NOW() |  | Thời điểm phát hành |
| `expires_at` | TIMESTAMP | No |  | IDX | Hết hạn tuyệt đối, mặc định 8 giờ từ lúc login |
| `idle_expires_at` | TIMESTAMP | No |  | IDX | Hết hạn do không hoạt động, mặc định 60 phút sau lần dùng cuối |
| `last_used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm refresh/session được dùng gần nhất |
| `rotated_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token được rotate |
| `revoked_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm revoke |
| `revoked_reason` | VARCHAR(50) | Yes | NULL | IDX | LOGOUT, PASSWORD_CHANGED, ADMIN_LOCK, TENANT_SUSPENDED, TOKEN_REUSE, EXPIRED |
| `replaced_by_session_id` | UUID | Yes | NULL | IDX | Session/refresh token mới sau rotation nếu có |
| `ip_address` | VARCHAR(45) | Yes | NULL | IDX | IP lúc phát hành hoặc dùng gần nhất |
| `user_agent` | TEXT | Yes | NULL |  | User-Agent |
| `device_id` | VARCHAR(100) | Yes | NULL | IDX | ID thiết bị/browser nếu frontend cung cấp |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

**Rule:** Refresh token rotation là bắt buộc. Nếu refresh token đã `ROTATED`/`REVOKED` bị dùng lại, revoke toàn bộ `token_family_id` và ghi audit/security event.

### `t_staff_auth_session` (Phiên đăng nhập / Refresh Token Staff Web)

> BF-M01 enterprise short session policy: DB là source of truth, Redis chỉ dùng cache/denylist/rate-limit. Access token JWT TTL mặc định 15 phút; refresh token absolute lifetime mặc định 8 giờ; idle timeout mặc định 60 phút; không có remember-me.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global tenant_id từ Platform |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `session_id` | UUID | No |  | **UK** | ID phiên đưa vào JWT claim để revoke/cache |
| `token_family_id` | UUID | No |  | IDX | Nhóm refresh token rotation; reuse token cũ sẽ revoke cả family |
| `staff_credential_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff_auth_credential(id)` |
| `refresh_token_hash` | VARCHAR(255) | No |  | **UK** | Hash của refresh token; tuyệt đối không lưu raw token |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, ROTATED, REVOKED, EXPIRED |
| `issued_at` | TIMESTAMP | No | NOW() |  | Thời điểm phát hành |
| `expires_at` | TIMESTAMP | No |  | IDX | Hết hạn tuyệt đối, mặc định 8 giờ từ lúc login |
| `idle_expires_at` | TIMESTAMP | No |  | IDX | Hết hạn do không hoạt động, mặc định 60 phút sau lần dùng cuối |
| `last_used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm refresh/session được dùng gần nhất |
| `rotated_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token được rotate |
| `revoked_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm revoke |
| `revoked_reason` | VARCHAR(50) | Yes | NULL | IDX | LOGOUT, PASSWORD_CHANGED, ADMIN_LOCK, STAFF_MANUAL_RESET, TENANT_SUSPENDED, TOKEN_REUSE, EXPIRED |
| `replaced_by_session_id` | UUID | Yes | NULL | IDX | Session/refresh token mới sau rotation nếu có |
| `ip_address` | VARCHAR(45) | Yes | NULL | IDX | IP lúc phát hành hoặc dùng gần nhất |
| `user_agent` | TEXT | Yes | NULL |  | User-Agent |
| `device_id` | VARCHAR(100) | Yes | NULL | IDX | ID thiết bị/browser nếu frontend cung cấp |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

**Rule:** Refresh token rotation là bắt buộc. Nếu refresh token đã `ROTATED`/`REVOKED` bị dùng lại, revoke toàn bộ `token_family_id` và ghi audit/security event.

### `mst_staff_conflict_date` (Theo dõi Ngày giới hạn Phái cử của Staff - 個人抵触日)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `conflict_date` | DATE | No |  | IDX | Ngày giới hạn 3 năm (Luật phái cử Nhật) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 5: Cấu hình Khung Nghiệp Vụ MOTO (Settings)

### `mst_moto_shift` (Master Định nghĩa Ca Làm Việc)

*Hỗ trợ ràng buộc chặt chẽ từ tài liệu `シフトマスタアップロード`*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `shift_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã ca làm việc (VD: S001) |
| `shift_name` | VARCHAR(100) | No |  |  | Tên ca (Ca sáng, Ca đêm) |
| `start_time` | SMALLINT | No |  |  | Phút từ 00:00. Bắt buộc < `end_time` |
| `end_time` | SMALLINT | No |  |  | Phút từ 00:00 (Hỗ trợ qua ngày: VD 30:00 = 1800 phút) |
| `break_time_minutes` | SMALLINT | No | 0 |  | Tổng thời gian nghỉ. Bắt buộc < Tổng giờ làm |
| `midnight_break_minutes` | SMALLINT | No | 0 |  | Thời gian nghỉ trong khung giờ đêm. Bắt buộc <= `break_time_minutes` |
| `shift_name_en` | VARCHAR(100) | Yes | NULL |  | シフト名 (英語) |
| `short_name_ja` | VARCHAR(50) | Yes | NULL |  | シフト短縮名 (日本語) |
| `short_name_en` | VARCHAR(50) | Yes | NULL |  | シフト短縮名 (英語) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_shift_code: (company_id, shift_code) WHERE deleted_at IS NULL

### `mst_moto_attendance_category` (Mã loại Chấm công: Nghỉ phép, Đi làm)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `category_code` | VARCHAR(20) | No |  | Composite Partial UK | Mã (WORK, PTO, ABSENT) |
| `category_name` | VARCHAR(100) | No |  |  | Tên (Làm việc, Nghỉ phép năm) |
| `is_paid` | SMALLINT | No | 1 |  | 1: Tính lương, 0: Không tính |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_category_code: (company_id, category_code) WHERE deleted_at IS NULL

### `mst_moto_workplace_type` (Master Phân loại Nơi làm việc - 就業場所区分)

*Thiết lập danh mục địa điểm (Văn phòng, Làm ở nhà, Vệ tinh) để Staff chọn khi chấm công.*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `workplace_code` | VARCHAR(20) | No |  | Composite Partial UK | Mã loại (VD: OFFICE, HOME, SATELLITE, OTHER) |
| `pc_display_name` | VARCHAR(50) | No |  |  | Tên hiển thị trên PC (VD: オフィス, 自宅) |
| `mobile_display_name` | VARCHAR(20) | No |  |  | Tên hiển thị trên Mobile (VD: オ, 自) |
| `pc_display_name_en` | VARCHAR(50) | Yes | NULL |  | Tên hiển thị PC (Tiếng Anh) |
| `mobile_display_name_en` | VARCHAR(20) | Yes | NULL |  | Tên hiển thị Mobile (Tiếng Anh) |
| `valid_from_month` | VARCHAR(7) | Yes | NULL |  | Tháng bắt đầu áp dụng (YYYY-MM) |
| `valid_to_month` | VARCHAR(7) | Yes | NULL |  | Tháng kết thúc áp dụng (YYYY-MM) |
| `use_fax_flg` | SMALLINT | No | 0 |  | Bật/tắt cho hệ thống FAX (1: Có, 0: Không) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_workplace_type: (company_id, workplace_code) WHERE deleted_at IS NULL

### `cfg_moto_contract_default` (Cấu hình Hợp đồng Mặc định)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Quan hệ 1:1 với company |
| `default_payment_terms` | VARCHAR(255) | Yes | NULL |  | Điều khoản thanh toán in trên hợp đồng |
| `show_unit_price_flg` | SMALLINT | No | 1 |  | 1: Hiện đơn giá, 0: Ẩn |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_invoice_calculation` (Quy tắc tính toán Hóa đơn - Vá Lỗi Thuế)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Quan hệ 1:1 |
| `tax_calculation_timing` | SMALLINT | No | 2 |  | 1: Tính thuế từng dòng, 2: Cộng tổng rồi mới tính thuế (Quy định bắt buộc của hóa đơn mới) |
| `default_tax_rate` | DECIMAL(5,2) | No | 10.00 |  | Mức thuế suất mặc định (10%) |
| `overtime_rate` | DECIMAL(5,2) | No | 1.25 |  | Hệ số làm thêm giờ (25%) |
| `night_rate` | DECIMAL(5,2) | No | 1.25 |  | Hệ số ca đêm (25%) |
| `holiday_rate` | DECIMAL(5,2) | No | 1.35 |  | Hệ số làm ngày nghỉ pháp định (35%) |
| `work_time_calc_method` | SMALLINT | No | 1 |  | 就業時間計算方法: 1=契約内/契約外 2=基準内/基準外 |
| `consumption_tax_external_flg` | SMALLINT | No | 1 |  | 消費税(外税)計算: 0/1 |
| `consumption_tax_rate_reduced` | SMALLINT | Yes | 8 |  | 軽減税率 (%) |
| `transport_convert_flg` | SMALLINT | No | 0 |  | 交通費(内税→外税)換算: 0/1 |
| `rounding_config` | JSONB | Yes | NULL |  | Quy tắc 端数処理 theo từng khoản (契約内/時間外/休出/深夜/控除/消費税/交通費 → 切捨/切上/四捨五入) — config theo company, không join → JSONB |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_invoice_fraction` (Quy tắc làm tròn số)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Quan hệ 1:1 |
| `time_rounding_type` | SMALLINT | No | 1 |  | Thời gian: 1: Lên, 2: Xuống, 3: Gần nhất |
| `time_rounding_unit` | SMALLINT | No | 15 |  | Đơn vị làm tròn (VD: 15 phút) |
| `money_rounding_type` | SMALLINT | No | 2 |  | Tiền: 1: Lên, 2: Cắt bỏ (Xuống), 3: Làm tròn |
| `tax_rounding_type` | SMALLINT | No | 2 |  | Thuế: 2: Cắt bỏ (Xuống) - Theo quy định thuế |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_36_agreement` (Cấu hình Hiệp định 36 - 36協定) [M10 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `agreement_name` | VARCHAR(100) | No |  |  | Tên hiệp định (VD: Hiệp định IT) |
| `valid_from` | DATE | No |  |  | Hiệu lực từ |
| `valid_to` | DATE | No |  |  | Hiệu lực đến |
| `max_overtime_month` | INT | No | 45 |  | Giới hạn OT thường 1 tháng (Giờ) |
| `max_overtime_year` | INT | No | 360 |  | Giới hạn OT thường 1 năm (Giờ) |
| `special_clause_month` | INT | Yes | NULL |  | Giới hạn OT Điều khoản đặc biệt (tháng) |
| `special_clause_year` | INT | Yes | NULL |  | Giới hạn OT Điều khoản đặc biệt (năm) |
| `spec_flag` | SMALLINT | No | 1 |  | 様式: 1=新様式 2=旧様式/適用除外 |
| `annual_start_date` | DATE | Yes | NULL |  | 1年間の起算日 |
| `monthly_start_day` | SMALLINT | Yes | NULL |  | 1ヶ月の起算日 (ngày trong tháng; default 1) |
| `weekly_start_dow` | SMALLINT | Yes | NULL |  | 週の起算日 (1=日…7=土; default 1=日) |
| `special_clause_count_limit` | SMALLINT | Yes | NULL |  | 特別条項 限度回数/năm (≤6) |
| `exemption_flg` | SMALLINT | No | 0 |  | 適用除外: 0=除外しない 1=届出あり 2=届出なし |
| `auto_extend_flg` | SMALLINT | No | 0 |  | 有効期間自動延長: 0/1 |
| `auto_reflect_contract_flg` | SMALLINT | No | 0 |  | 個別契約へ自動反映: 0/1 (chỉ 1 bản/company) |
| `current_document_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ về `t_36_agreement_document(id)` bản PDF hiện hành (latest version). Khi gia hạn → update sang document mới **[M10 NEW]** |
| `last_filing_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ về `t_36_agreement_filing(id)` lần 届出 gần nhất tới 労働基準監督署 **[M10 NEW]** |
| `expiration_warning_sent_at` | TIMESTAMP | Yes | NULL |  | Thời điểm cảnh báo gia hạn 60 ngày trước hết hạn đã gửi — BR-1505. NULL = chưa gửi. Dùng để tránh spam cảnh báo **[M10 NEW]** |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_legal_holiday` (Ngày nghỉ pháp định - Phục vụ tính lương 1.35)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `rule_type` | SMALLINT | No | 1 |  | 1=特定曜日, 2=毎週1回, 3=毎週1回または4週4回 |
| `specified_day_of_week` | SMALLINT | Yes | NULL |  | Nếu type=1, lưu Thứ (0:CN, 1:T2... 6:T7) |
| `week4_start_date` | DATE | Yes | NULL |  | 4週4日 起算日 (khi rule_type=3) |
| `determination_method` | SMALLINT | Yes | NULL |  | 法定休日出勤判定方法: 1=シフト出勤予定日以外 2=契約勤務日以外 3=起算曜日から最も遠い |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_bank_account` (Tài khoản ngân hàng của MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `bank_name` | VARCHAR(100) | No |  |  | Tên ngân hàng |
| `branch_name` | VARCHAR(100) | No |  |  | Tên chi nhánh |
| `account_type` | SMALLINT | No | 1 |  | 1: Thường (普通), 2: Hiện tại (当座) |
| `account_number` | VARCHAR(50) | No |  |  | Số tài khoản |
| `account_holder` | VARCHAR(100) | No |  |  | Tên người nhận (Katakana) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_alarm_setting` (Cấu hình Cảnh báo chung) [M08 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `alarm_type` | VARCHAR(50) | No |  | **UK** | Mã: 36_AGREEMENT, CONFLICT_DATE, TIMECARD |
| `is_enabled` | SMALLINT | No | 1 |  | 1: Bật, 0: Tắt |
| `alert_days_before` | INT | Yes | NULL |  | Báo trước N ngày |
| `alarm_subtype` | VARCHAR(30) | Yes | NULL |  | Loại 36協定 alarm: SINGLE_MONTH(単月) / MULTI_MONTH_AVG(複数月平均) / ANNUAL(年間) |
| `threshold_config` | JSONB | Yes | NULL |  | Cấu hình ngưỡng cảnh báo: list {percent: 50/60/70/80/90/100, frequency: ONCE(1回)/DAILY(毎日)} (list config, không join → JSONB hợp lý) |
| `channel_email_flg` | SMALLINT | No | 1 |  | 1=Gửi email / 0=Không (BR-101) **[M08 NEW]** |
| `channel_inapp_flg` | SMALLINT | No | 1 |  | 1=Hiển thị in-app / 0=Không. Với is_legal_alert=1 phải = 1 (BR-101/126) **[M08 NEW]** |
| `is_legal_alert` | SMALLINT | No | 0 | IDX | 1=Cảnh báo pháp lý (36協定/抵触日) — app layer chặn UI tắt in-app **[M08 NEW]** |
| `remind_frequency` | SMALLINT | No | 1 |  | 1=ONCE / 2=DAILY / 3=WEEKLY (BR-130) **[M08 NEW]** |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_moto_alarm_channels: channel_email_flg IN (0, 1) AND channel_inapp_flg IN (0, 1)
- **CHECK** chk_moto_alarm_is_enabled: is_enabled IN (0, 1)
- **CHECK** chk_moto_alarm_legal: is_legal_alert IN (0, 1)
- **CHECK** chk_moto_alarm_remind_freq: remind_frequency IN (1, 2, 3)
- **CHECK** chk_moto_alarm_legal_inapp: is_legal_alert = 0 OR channel_inapp_flg = 1 (BR-101: Cảnh báo pháp lý không thể tắt in-app)

### `cfg_moto_alarm_recipient` (Người nhận cảnh báo MOTO) [M08 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `alarm_setting_id` | BIGINT | No |  | FK, IDX | Tham chiếu cấu hình alarm |
| `recipient_type` | SMALLINT | No | 2 | IDX | 1=ROLE_BASED (gửi cho tất cả user thuộc role)/ 2=SPECIFIC_USER (gửi user cụ thể) **[M08 NEW]** |
| `user_id` | BIGINT | Yes | NULL | FK, IDX | Bắt buộc khi recipient_type=2; NULL khi recipient_type=1 **[M08 NULL changed]** |
| `recipient_role` | VARCHAR(30) | Yes | NULL |  | Bắt buộc khi recipient_type=1: SALES_REP/AGENCY_MANAGER/COMPLAINT_CONTACT |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_moto_alarm_recipient_type: recipient_type IN (1, 2)
- **CHECK** chk_moto_alarm_recipient_consistency: (recipient_type = 1 AND recipient_role IS NOT NULL AND user_id IS NULL) OR (recipient_type = 2 AND user_id IS NOT NULL)

---

### `cfg_moto_contract_additional_text` (Văn bản bổ sung mặc định trên hợp đồng MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `additional_text_content` | TEXT | No |  |  | Nội dung văn bản bổ sung mặc định |
| `status` | SMALLINT | No | 1 |  | 1: Đang sử dụng, 0: Ngưng sử dụng |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_contract_text_per_client` (Văn bản hợp đồng tùy biến theo khách hàng SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_contract_company` |
| `client_id` | BIGINT | No |  | FK, IDX | Tham chiếu Khách hàng `cfg_moto_client_company(id)` |
| `custom_text_content` | TEXT | No |  |  | Nội dung văn bản tùy chỉnh riêng cho khách hàng này |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_contract_party` (契約先企業マスタ — Alias tên hiển thị SAKI trên PDF hợp đồng)

> **[Phase 2 NEW — 2026-06-29]** Theo `操作マニュアル企業設定-派遣元` pages 62-63: MOTO định nghĩa N entries per SAKI customer; mỗi entry có `契約先コード` + `契約先企業名(JP)` + `契約先企業名(英字)` được dùng khi t_contract reference qua `contract_party_code`. Nếu không reference → fallback dùng tên gốc SAKI từ Platform. Spec đầy đủ: `docs/superpowers/specs/2026-06-29-cfg-moto-contract-party-design.md`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `client_company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `cfg_moto_client_company(id)` — SAKI parent |
| `contract_party_code` | VARCHAR(16) | No |  |  | 契約先コード (vd `C001-001`) — match với `t_contract.contract_party_code` snapshot |
| `display_name_jp` | VARCHAR(255) | No |  |  | 契約先企業名 (Japanese) — name sẽ in lên PDF hợp đồng |
| `display_name_en` | VARCHAR(255) | No |  |  | 契約先企業名（英字）— English variant, bắt buộc theo UI e-staffing |
| `status` | SMALLINT | No | 1 | IDX | 1=有効 (active), 0=無効 (inactive) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_contract_party_code: (company_id, contract_party_code) WHERE deleted_at IS NULL
- **FK** fk_contract_party_company: (company_id) → mst_moto_company(id)
- **FK** fk_contract_party_client: (client_company_id) → cfg_moto_client_company(id)
- **INDEX** idx_moto_contract_party_client: (client_company_id)

### `cfg_moto_invoice_default` (Cấu hình mặc định cho Hóa đơn phía MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `mst_moto_company(id)` (1:1) |
| `default_billing_method` | SMALLINT | No | 1 |  | 1: Chuyển khoản, 2: Séc, 3: Tiền mặt |
| `default_payment_terms` | VARCHAR(255) | Yes | NULL |  | Điều khoản thanh toán mặc định in trên hóa đơn |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_seal` (Cấu hình Con dấu dùng trên chứng từ MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `seal_name` | VARCHAR(100) | No |  |  | Tên gọi của con dấu (VD: Dấu tròn công ty) |
| `seal_image_url` | VARCHAR(255) | No |  |  | Đường dẫn lưu ảnh con dấu trên S3 |
| `default_flg` | SMALLINT | No | 0 |  | 1: Làm con dấu mặc định |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_seal_per_client` (Cấu hình Con dấu tùy biến theo từng khách hàng SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `client_id` | BIGINT | No |  | FK, IDX | Tham chiếu Khách hàng `cfg_moto_client_company(id)` |
| `seal_id` | BIGINT | No |  | FK, IDX | Tham chiếu Con dấu `cfg_moto_seal(id)` |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_advance_payment_screen` (Cấu hình Màn hình thanh toán tạm ứng / 立替)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `client_company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `cfg_moto_client_company(id)`; cấu hình theo từng khách hàng SAKI (クライアントコード) |
| `screen_type` | VARCHAR(50) | No |  |  | Loại màn hình nhập tạm ứng: INVOICE_APPLICABLE / INVOICE_NON_APPLICABLE (立替金入力画面の種類) |
| `display_advance_payment_flg` | SMALLINT | No | 1 |  | 1: Hiển thị màn hình nhập chi phí tạm ứng |
| `apply_period_start` | DATE | Yes | NULL |  | Bắt đầu kỳ áp dụng (適用期間); bắt buộc khi INVOICE_APPLICABLE |
| `apply_period_end` | DATE | Yes | NULL |  | Kết thúc kỳ áp dụng |
| `screen_description` | TEXT | Yes | NULL |  | Mô tả hướng dẫn nhập liệu trên UI |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_advance_payment_client: (company_id, client_company_id) WHERE deleted_at IS NULL

### `cfg_moto_file_layout` (Tùy biến layout file DL/UL — ファイルレイアウト)

> Cấu hình cột & thứ tự cột khi download/upload. 1 layout/loại/công ty; áp dụng cho mọi user trong công ty. API liên quan: `MO-SET-011`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `layout_type` | VARCHAR(50) | No |  | Composite Partial UK | CONTRACT_CSV_DOWNLOAD / CONTRACT_UPLOAD / INVOICE_UPLOAD_HEADER / INVOICE_UPLOAD_DETAIL / INVOICE_UPLOAD_ATTENDANCE / REIMBURSEMENT_HEADER / REIMBURSEMENT_DETAIL |
| `column_config` | JSONB | No |  |  | Danh sách cột đã chọn + thứ tự (tối thiểu 1 cột) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_file_layout: (company_id, layout_type) WHERE deleted_at IS NULL
- **RLS policy** — p_cfg_file_layout_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `cfg_moto_editable_user_field` (Cấu hình trường user tự sửa được — ユーザー編集可能項目)

> Quy định những field nào user được tự sửa trong màn hình ユーザー情報編集. API liên quan: `MO-SET-021`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `field_key` | VARCHAR(50) | No |  | Composite Partial UK | full_name / full_name_kana / tel / email / display_lang |
| `is_editable` | SMALLINT | No | 1 |  | 1: Cho phép user tự sửa |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_moto_editable_user_field: (company_id, field_key) WHERE deleted_at IS NULL
- **RLS policy** — p_cfg_editable_field_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `cfg_moto_36_agreement_free` (Cấu hình Hiệp định 36 dạng tự do)

*Phục vụ các trường hợp linh hoạt không giới hạn cứng bằng số.*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `agreement_content` | TEXT | No |  |  | Nội dung hiệp định 36 nhập tự do |
| `valid_from` | DATE | No |  |  | Ngày bắt đầu hiệu lực |
| `valid_to` | DATE | No |  |  | Ngày kết thúc hiệu lực |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_36_overtime_start` (Cấu hình mốc bắt đầu tính giờ làm thêm theo Hiệp định 36)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `mst_moto_company(id)` (1:1) |
| `overtime_start_minutes` | SMALLINT | No | 480 |  | Số phút tiêu chuẩn trước khi bắt đầu tính OT (VD: 480 = 8 tiếng) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_compensatory_holiday` (Cấu hình ngày nghỉ bù / 振替休日 của MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `mst_moto_company(id)` (1:1) |
| `allow_compensatory_holiday_flg` | SMALLINT | No | 1 |  | 1: Cho phép sử dụng ngày nghỉ bù |
| `expire_days` | SMALLINT | No | 30 |  | Hạn sử dụng ngày nghỉ bù (VD: 30 ngày) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_moto_alarm_conflict` (Cấu hình cảnh báo xung đột ngày / 抵触日 của MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `mst_moto_company(id)` (1:1) |
| `enabled_flg` | SMALLINT | No | 1 |  | 1: Bật cảnh báo, 0: Tắt |
| `alarm_days_before` | SMALLINT | No | 30 |  | Cảnh báo trước bao nhiêu ngày khi chạm mức giới hạn pháp lý 3 năm |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 6: Yêu cầu Phái cử & Báo giá (Inquiry & Estimate)

### `t_dispatch_inquiry_received` (Yêu cầu MOTO nhận được - Immutable Revisions)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**  , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**  , IDX | ID Tenant SAKI gửi yêu cầu |
| `saki_company_code` | VARCHAR(50) | No |  | **Logical FK**  , IDX | Mã Business Code SAKI gửi |
| `client_id` | BIGINT | No |  | FK, IDX | Tham chiếu Khách hàng nội bộ MOTO |
| `saki_inquiry_id` | BIGINT | No |  | **Logical FK**  , IDX | ID gốc của Yêu cầu bên schema SAKI |
| `saki_inquiry_revision_no` | SMALLINT | No | 1 |  | Số Revision nhận từ SAKI (Tăng khi SAKI sửa đổi) |
| `inquiry_payload` | JSONB | No |  |  | Data Snapshot tại đúng Revision này |
| `status` | VARCHAR(20) | No | 'RECEIVED' | IDX | RECEIVED, VIEWED, RESPONDED, DECLINED, EXPIRED, SUPERSEDED |
| `superseded_by_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ tới ID của Revision mới thay thế nó |
| `deadline_at` | TIMESTAMP | Yes | NULL |  | Hạn chót SAKI cần phản hồi |
| `idempotency_key` | VARCHAR(100) | No |  | Composite UK | Chống duplicate từ Event Bus |
| `correlation_id` | VARCHAR(100) | No |  | IDX | Trace chuỗi Event |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | User tạo |
| `updated_by` | BIGINT | Yes | NULL |  | User cập nhật |
- **UNIQUE** uq_inquiry_received_revision: (company_id, saki_tenant_id, saki_inquiry_id, saki_inquiry_revision_no)
- **UNIQUE** uq_inquiry_received_idem: (company_id, saki_tenant_id, idempotency_key)
- **UNIQUE (Partial)** uq_inquiry_received_active: (company_id, saki_tenant_id, saki_inquiry_id) WHERE status <> 'SUPERSEDED' AND deleted_at IS NULL
- **FK** fk_inquiry_superseded: (superseded_by_id) → t_dispatch_inquiry_received(id)

### `t_dispatch_estimate_response` (Báo giá của MOTO gửi lại SAKI - Dạng Immutable)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO tenant) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI gửi thông báo |
| `status` | VARCHAR(20) | No | 'DRAFT' | IDX | DRAFT → SENT → VIEWED (F-0501/F-0502 filter, gate check F-0507) |
| `treatment_decision_method` | SMALLINT | No |  | IDX | 1=労使協定方式 / 2=派遣先均等・均衡方式 (F-0501/F-0502 filter, BR-052) |
| `version` | SMALLINT | No | 1 |  | Phiên bản thông báo — tăng khi SAKI cần cập nhật thông tin đối ngộ (BR-054) |
| `superseded_by_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ tới version mới thay thế (BR-054 immutability pattern) |
| `notification_date` | DATE | No |  | IDX | Ngày SAKI phát hành thông báo chính thức |
| `payload` | JSONB | No |  |  | Chi tiết thông tin đãi ngộ (display-only: lương/phụ cấp/phúc lợi/đào tạo; schema phân nhánh theo treatment_decision_method; không filter/join/calc → hợp lệ D3.3) |
| `viewed_at` | TIMESTAMPTZ | Yes | NULL |  | Thời điểm MOTO đầu tiên mở xem (trigger status → VIEWED) |
| `viewed_by_moto_user_id` | BIGINT | Yes | NULL |  | Logical FK — user MOTO đã xem đầu tiên |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
| `draft_expires_at` | TIMESTAMP | Yes | NULL | IDX | Hạn lưu nháp (D18.10) = thời điểm save + TTL; NULL khi không ở trạng thái draft. Set lúc save-draft, clear (NULL) khi submit. |
| `expiration_status` | VARCHAR(10) | Yes | NULL |  | normal / warning / expired (D18.10); NULL khi không phải draft; batch hằng ngày maintain tier. |
- **UNIQUE** uq_estimate_version: (company_id, inquiry_received_id, estimate_group_id, version)
- **UNIQUE (Partial)** uq_estimate_idempotency: (company_id, idempotency_key) WHERE idempotency_key IS NOT NULL
- **UNIQUE (Partial)** uq_estimate_single_draft: (company_id, inquiry_received_id, estimate_group_id) WHERE status = 'DRAFT' AND deleted_at IS NULL
- **UNIQUE (Partial)** uq_estimate_single_active: (company_id, inquiry_received_id, estimate_group_id) WHERE status = 'SUBMITTED' AND deleted_at IS NULL
- **FK** fk_estimate_superseded: (superseded_by_id) → t_dispatch_estimate_response(id)
- **INDEX (Partial)** idx_dispatch_estimate_response_draft_cleanup: (status, draft_expires_at) WHERE deleted_at IS NULL

---

### `t_dispatch_estimate_staff_profile` (Profile Ẩn danh đính kèm Báo giá)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `estimate_id` | BIGINT | No |  | FK, IDX | Tham chiếu `t_dispatch_estimate_response(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` (Dùng map nội bộ MOTO) |
| `anonymous_name` | VARCHAR(50) | No |  |  | Tên ẩn danh (VD: Aさん, Kỹ sư B) |
| `age_group` | VARCHAR(20) | No |  |  | Độ tuổi (VD: 20代, 30代) |
| `gender` | SMALLINT | Yes | NULL |  | 1: Nam, 2: Nữ |
| `skills_snapshot` | JSONB | No |  |  | Lưu mảng JSON chứa các kỹ năng của Staff này |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 7: Quản lý Hợp đồng Phái cử (Contracts - Anti-Mutation Applied)

*Theo tài liệu, Hợp đồng là trung tâm kết nối giữa Staff, MOTO và SAKI. MOTO là bên sở hữu vòng đời hợp đồng.*

### `t_contract` (Header Hợp đồng Phái cử)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `contract_no` | VARCHAR(50) | No |  | **PK** | Khóa chính (Mã hợp đồng tự sinh, không dùng IDENTITY) |
| `original_contract_no` | VARCHAR(50) | Yes | NULL | FK, IDX | Hợp đồng gốc (Phục vụ truy vết Gia hạn/Sửa đổi) trỏ về contract_no |
| `tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Tham chiếu Tenant SAKI |
| `saki_company_code` | VARCHAR(50) | No |  | **Logical FK** , IDX | Mã Business Code SAKI. Bắt buộc Index. |
| `client_id` | BIGINT | No |  | FK, IDX | Tham chiếu cfg_moto_client_company(id) |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_staff(id) |
| `contract_type` | SMALLINT | No | 1 |  | 1: Tạo mới, 2: Gia hạn, 3: Sửa, 4: Kết thúc |
| `initial_start_date` | DATE | No |  |  | Ngày bắt đầu làm việc của chu kỳ hợp đồng gốc đầu tiên |
| `contract_start_date` | DATE | No |  | IDX | Ngày bắt đầu hợp đồng của chu kỳ này |
| `contract_end_date` | DATE | No |  | IDX | Ngày kết thúc hợp đồng của chu kỳ này |
| `manufacturing_flg` | SMALLINT | No | 0 |  | 1: Hợp đồng nghiệp vụ sản xuất |
| `manufacturing_worker_le50_flg` | SMALLINT | No | 0 |  | 1: Số LĐ làm SX ≤50 người (đăng ký theo 派遣先責任者). Khác `manufacturing_flg` |
| `temp_to_perm_flg` | SMALLINT | No | 0 |  | 1: Hợp đồng phái cử dự kiến giới thiệu (紹介予定派遣) |
| `client_contract_no` | VARCHAR(50) | Yes | NULL | IDX | 契約番号 phía khách hàng (vd Job001), khác `contract_no` nội bộ e-staffing |
| `contract_party_code` | VARCHAR(16) | Yes | NULL |  | 契約先コード (alias hiển thị tên SAKI trên PDF hợp đồng, snapshot text từ `cfg_moto_contract_party.contract_party_code`; NULL → fallback dùng tên SAKI gốc từ Platform `mst_saki_company`). KHÔNG hard FK để giữ immutability khi master alias bị sửa/xóa sau khi contract đã chốt. |
| `branch_no` | VARCHAR(3) | No | '000' |  | 枝番 (3 số) — phiên bản sửa của cùng contract_no |
| `superseded_by_id` | VARCHAR(50) | Yes | NULL | FK, IDX | Trỏ tới `contract_no` của version thay thế (D18.5 Model A). NULL = version hiện hành. |
| `requestor_user_id` | BIGINT | Yes | NULL | FK, IDX | Người tạo yêu cầu hợp đồng / 契約依頼者 (Tham chiếu mst_moto_user); giữ ở header cho data-scope |
| `confirmor_user_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | Người xác nhận hợp đồng / 契約確認者 (Tham chiếu ID User SAKI); giữ ở header cho data-scope |
| `status` | VARCHAR(20) | No | 'DRAFT' | IDX | DRAFT, SUBMITTED, SAKI_CONFIRMING, SAKI_APPROVING, SENT_BACK_TO_AGENCY, WITHDRAWN, ACTIVE, AMENDING, CANCELLED, TERMINATED (vòng đời 2 cấp duyệt SAKI: 確認者→承認者) |
| `submitted_at` | TIMESTAMP | Yes | NULL |  | Thời điểm MOTO bấm nộp |
| `confirmed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm SAKI duyệt xác lập |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
| `draft_expires_at` | TIMESTAMP | Yes | NULL | IDX | Hạn lưu nháp (D18.10) = thời điểm save + TTL; NULL khi không ở trạng thái draft. Set lúc save-draft, clear (NULL) khi submit. |
| `expiration_status` | VARCHAR(10) | Yes | NULL |  | normal / warning / expired (D18.10); NULL khi không phải draft; batch hằng ngày maintain tier. |
- **PK** : (contract_no)
- **FK** fk_contract_original: (original_contract_no) → t_contract (contract_no)
- **FK** fk_contract_superseded: (superseded_by_id) → t_contract (contract_no)
- **UNIQUE (Partial)** uq_contract_single_active: (COALESCE(original_contract_no, contract_no)) WHERE status IN ('ACTIVE','AMENDING') AND deleted_at IS NULL
- **FK** fk_contract_company: (company_id) → mst_moto_company (id)
- **FK** fk_contract_staff: (staff_id) → mst_staff (id)
- **FK** fk_contract_requestor: (requestor_user_id) → mst_moto_user (id)
- **INDEX** idx_contract_saki_tenant: (saki_tenant_id)
- **INDEX** idx_contract_saki_code: (saki_company_code)
- **INDEX** idx_contract_confirmor: (confirmor_user_id)
- **INDEX** idx_contract_status: (status)
- **RLS policy** — p_contract_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true))
- **RLS policy** — p_contract_moto_owner: USING (current_setting('app.current_role', true) = 'moto_owner') WITH CHECK (current_setting('app.current_role', true) = 'moto_owner')
- **Generation Rule (Phase 3 Invariant):** Cột `contract_no` BẮT BUỘC do Sequence Service cấp phát theo định dạng `{tenant_prefix}-{company_prefix}-C{YYMM}-{seq}`. Cấm Client tự nhập.
- **INDEX (Partial)** idx_contract_draft_cleanup: (status, draft_expires_at) WHERE deleted_at IS NULL

---

### `t_contract_moto_detail` (Chi tiết Điều kiện Hợp đồng của MOTO)

*Bảng này sử dụng cơ chế Snapshot (lưu cứng dữ liệu text) để đáp ứng luật kiểm toán: Nếu Master thay đổi, Hợp đồng cũ không bị đổi theo.*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `contract_no` | VARCHAR(50) | No |  | FK, **UK** | Tham chiếu t_contract(contract_no) (1:1) |
| `staff_name_snapshot` | VARCHAR(100) | No |  |  | Lưu cứng Tên nhân viên tại thời điểm ký |
| `client_name_snapshot` | VARCHAR(100) | No |  |  | Lưu cứng Tên công ty khách hàng |
| `health_insurance_status` | SMALLINT | Yes | NULL |  | 健康保険: 1=加入済 2=手続中 3=対象外 |
| `welfare_pension_status` | SMALLINT | Yes | NULL |  | 厚生年金 status |
| `employment_insurance_status` | SMALLINT | Yes | NULL |  | 雇用保険 status |
| `treatment_decision_method` | SMALLINT | Yes | NULL | IDX | 待遇決定方式: 1=労使協定方式 2=派遣先均等・均衡方式 — filter compliance |
| `period_limit_exempt_flg` | SMALLINT | No | 0 |  | 期間制限の対象外: 0/1 |
| `personal_conflict_date` | DATE | Yes | NULL | IDX | 個人抵触日 (snapshot) — M08 抵触日 alert |
| `office_conflict_date` | DATE | Yes | NULL | IDX | 事業所抵触日 (snapshot) — M08 alert |
| `gender_snapshot` | SMALLINT | Yes | NULL |  | Giới tính staff lúc ký (nguồn `mst_staff.gender`): 1=男 2=女 — cho 通知書 |
| `age_category_snapshot` | SMALLINT | Yes | NULL |  | 年齢区分: 1=18歳未満 2=18-44 3=45-59 4=60歳以上 |
| `actual_age_snapshot` | SMALLINT | Yes | NULL |  | 実年齢 (khi 区分=1) |
| `agreement_target_flg` | SMALLINT | Yes | NULL |  | 協定対象派遣労働者該当 (通知書): 0/1 |
| `timecard_closing_type` | SMALLINT | Yes | NULL | IDX | 締め日: 1=15日/末日 2=15日/20日/末日 3=末日 — hot cho tổng hợp chấm công M04 + hóa đơn M05 |
| `mobile_use_flg` | SMALLINT | No | 0 |  | モバイル利用: 1/0 (Staff Web mobile) |
| `stamp_use_flg` | SMALLINT | No | 0 |  | 打刻使用: 1/0 |
| `gps_location_flg` | SMALLINT | No | 0 |  | 位置情報取得 (GPS khi chấm qua mobile): 1/0 |
| `staff_can_choose_approver_flg` | SMALLINT | No | 0 |  | スタッフによる承認者選択: 1=cho phép 0=không |
| `insurance_payload` | JSONB | Yes | NULL |  | 社会保険補足 (chỉ in 通知書, không join): 3 loại × {手続中日数, 対象外 cờ: 期間/時間/年齢/学生/賃金<88k/規模501-101/他社加入/週20h} |
| `period_limit_payload` | JSONB | Yes | NULL |  | 期間制限 detail (compliance display): 5 lý do 対象外 + 日数限定 + 代替要員(氏名/業務/休業期間) + 限定の別 + 派遣元雇用形態 + 事業所名称選択/名称 + 抵触日適用事業所code |
| `treatment_payload` | JSONB | Yes | NULL |  | 待遇 (text dài, không join): 比較対象労働者待遇情報, 協定対象限定の別 |
| `benefit_payload` | JSONB | Yes | NULL |  | 便宜供与 (checkbox/text in HĐ): 教育訓練+OJT(手順/業界/システム/その他), 給食/休憩/更衣/診療/制服/保養/物販 |
| `remark_payload` | JSONB | Yes | NULL |  | 備考 (display, cấu hình): 契約書/通知書備考, 備考1-10, 備考コード1-5, 選択式備考1-3, 自社共有用1-3(internal_only), 台帳備考 |
| `intro_to_hire_payload` | JSONB | Yes | NULL |  | 紹介予定派遣 (optional informational): 派遣期間後雇用形態, 年休退職金算入, 試用期間 (chỉ khi temp_to_perm_flg=1) |
| `job_posting_payload` | JSONB | Yes | NULL |  | Toàn bộ 求人票 (~40 field optional, chỉ in): 会社情報/勤務条件/加入保険/待遇/募集要件/求人者向け — chỉ khi 求人票フラグ=1 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

> Phân loại (đã đánh giá đánh đổi): **cột hot** cho field bị filter/calc/alert (hóa đơn/抵触日/compliance/通知書/đếm); **7 JSONB** CHỈ cho dữ liệu informational/display nhiều field không join (insurance補足, period_limit detail, treatment text, benefit, remark, intro_to_hire, job_posting). Dữ liệu cần join/calc (lịch làm việc, đơn giá, người phụ trách, 36協定, 台帳職種, 法定休日) tách thành **bảng con quan hệ** (xem dưới).
> 

### `t_contract_saki_detail` (Chi tiết Hợp đồng phía SAKI nhận được)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, **UK** | Tham chiếu `t_contract(contract_no)` (1:1) |
| `saki_office_name_snapshot` | VARCHAR(100) | No |  |  | Tên chi nhánh SAKI tiếp nhận (就業先事業所) |
| `saki_dept_name_snapshot` | VARCHAR(100) | No |  |  | Tên phòng ban SAKI tiếp nhận (就業先部署) |
| `manager_name_snapshot` | VARCHAR(100) | Yes | NULL |  | Tên người quản lý phía SAKI |
| `cost_center_code` | VARCHAR(20) | Yes | NULL | IDX | コストセンターコード — báo cáo chi phí nội bộ SAKI |
| `cost_center_name_snapshot` | VARCHAR(100) | Yes | NULL |  | **[Reviewer Fix 6]** Snapshot tên Cost Center từ `mst_saki_cost_center.cost_center_name` tại thời điểm ký HĐ — in lên 派遣先管理台帳 (Sổ cái quản lý phái cử). Đảm bảo immutability nếu SAKI đổi tên master. |
| `org_unit_name_snapshot` | VARCHAR(400) | Yes | NULL |  | 組織単位 (snapshot từ 部署マスタ SAKI) |
| `org_head_title_snapshot` | VARCHAR(200) | Yes | NULL |  | 組織の長の職名 |
| `saki_confirm_payload` | JSONB | Yes | NULL |  | Chỉ dữ liệu informational SAKI: コストセンターコメント; SAKI 備考; 派遣先管理台帳備考. (Person roles SAKI gồm 派遣先責任者/指揮命令者/苦情申出先/請求先担当者 + 勤怠承認者1-3 → chuyển sang bảng `t_contract_person_role` actor_side=SAKI để filter/routing) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

> Bảng này do hệ thống ghi qua **cross-tenant workflow** (SAKI xác nhận → ghi vào schema MOTO). 勤怠承認者 đọc theo contract_no khi duyệt chấm công (point lookup); tính năng 承認者一括修正 cập nhật JSONB.
> 

### `t_contract_work_schedule` (Điều kiện/lịch làm việc Hợp đồng — 1:1)

> Tách bảng vì M04 (tính chấm công) + M05 (hóa đơn) đọc 勤務日/時間/休憩/OT để tính toán/validate.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** |  |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, **UK** | `t_contract(contract_no)` (1:1) |
| `business_content` | VARCHAR(2050) | Yes | NULL |  | 業務内容 (≤25 dòng) |
| `responsibility_degree` | VARCHAR(510) | Yes | NULL |  | 責任の程度 |
| `safety_hygiene_content` | VARCHAR(500) | Yes | NULL |  | 安全及び衛生内容 |
| `work_mon` | SMALLINT | No | 0 |  | 勤務日 月 (1/0) |
| `work_tue` | SMALLINT | No | 0 |  | 火 |
| `work_wed` | SMALLINT | No | 0 |  | 水 |
| `work_thu` | SMALLINT | No | 0 |  | 木 |
| `work_fri` | SMALLINT | No | 0 |  | 金 |
| `work_sat` | SMALLINT | No | 0 |  | 土 |
| `work_sun` | SMALLINT | No | 0 |  | 日 |
| `work_holiday` | SMALLINT | No | 0 |  | 祝 |
| `shift_flg` | SMALLINT | No | 0 |  | シフト制: 1/0 |
| `holiday_sat` | SMALLINT | No | 0 |  | 休日土 |
| `holiday_sun` | SMALLINT | No | 0 |  | 休日日 |
| `holiday_public` | SMALLINT | No | 0 |  | 休日祝 |
| `holiday_designated` | SMALLINT | No | 0 |  | 指定休日 |
| `holiday_free_text` | VARCHAR(58) | Yes | NULL |  | 休日フリーテキスト |
| `work_start_time` | VARCHAR(4) | Yes | NULL |  | 勤務開始時刻 HHMM (0000-4759) |
| `work_end_time` | VARCHAR(4) | Yes | NULL |  | 勤務終了時刻 HHMM |
| `break_start_1` | VARCHAR(4) | Yes | NULL |  | 休憩開始1 HHMM |
| `break_start_2` | VARCHAR(4) | Yes | NULL |  | 休憩開始2 |
| `break_start_3` | VARCHAR(4) | Yes | NULL |  | 休憩開始3 |
| `break_dur_1` | SMALLINT | Yes | NULL |  | 休憩時間1 (phút 0-180) |
| `break_dur_2` | SMALLINT | Yes | NULL |  | 休憩時間2 |
| `break_dur_3` | SMALLINT | Yes | NULL |  | 休憩時間3 |
| `overtime_work_flg` | SMALLINT | No | 0 |  | 時間外労働: 1/0 |
| `holiday_work_flg` | SMALLINT | No | 0 |  | 休日労働: 1/0 |
| `renewal_count` | SMALLINT | Yes | NULL |  | 更新数 (1-6) |
| `renewal_unit` | SMALLINT | Yes | NULL |  | 更新単位: 1=ヶ月 2=年 |
| `last_extendable_date` | DATE | Yes | NULL |  | スタッフ最終延長可能日 |
| `other_work_condition_comment` | VARCHAR(500) | Yes | NULL |  | その他就業条件コメント |
| `created_at` | TIMESTAMP | No | NOW() |  |  |
| `updated_at` | TIMESTAMP | No | NOW() |  |  |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  |  |
| `updated_by` | BIGINT | Yes | NULL |  |  |
- **RLS policy** — p_contract_work_schedule_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_contract_billing` (Đơn giá / điều kiện thanh toán Hợp đồng — 1:1)

> Tách bảng vì M05 (hóa đơn) đọc đơn giá để tính toán.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** |  |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, **UK** | (1:1) |
| `unit_type` | SMALLINT | No | 1 |  | 単価単位: 1=時給 2=日給 3=月給 |
| `billing_unit_price` | INT | No |  |  | 請求単価 |
| `payment_terms` | VARCHAR(255) | Yes | NULL |  | Điều khoản thanh toán |
| `overtime_unit_price` | INT | Yes | NULL |  | 時間外単価 |
| `legal_holiday_unit_price` | INT | Yes | NULL |  | 法定休出単価 |
| `latenight_unit_price` | INT | Yes | NULL |  | 深夜割増単価 |
| `premium_show_flg` | SMALLINT | No | 1 |  | 割増単価・契約書表示: 0/1 |
| `monthly_lower_hours` | NUMERIC(5,2) | Yes | NULL |  | 月額適用下限時間 (0-999.99) |
| `monthly_upper_hours` | NUMERIC(5,2) | Yes | NULL |  | 月額適用上限時間 |
| `shortage_unit_price` | INT | Yes | NULL |  | 不足時間単価 |
| `excess_unit_price` | INT | Yes | NULL |  | 超過時間単価 |
| `monthly_settle_show_flg` | SMALLINT | No | 0 |  | 月額精算・契約書表示: 0/1 |
| `commute_allowance` | VARCHAR(510) | Yes | NULL |  | 交通費相当額 |
| `commute_show_flg` | SMALLINT | No | 0 |  | 交通費・契約書表示: 0/1 |
| `fee_show_flg` | SMALLINT | No | 1 |  | 派遣料金・契約書表示: 0/1 |
| `other_billing_remark` | VARCHAR(408) | Yes | NULL |  | その他請求に関する備考 |
| `other_billing_show_flg` | SMALLINT | No | 0 |  | 0/1 |
| `created_at` | TIMESTAMP | No | NOW() |  |  |
| `updated_at` | TIMESTAMP | No | NOW() |  |  |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  |  |
| `updated_by` | BIGINT | Yes | NULL |  |  |
- **RLS policy** — p_contract_billing_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_contract_person_role` (Người phụ trách Hợp đồng — 1:N)

> Tách bảng vì 営業担当 dùng cho 参照範囲 scope/filter; 勤怠承認者 dùng routing duyệt chấm công M04. `requestor_user_id`/`confirmor_user_id` giữ ở header `t_contract`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** |  |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX |  |
| `role_type` | VARCHAR(30) | No |  | IDX | SALES_REP, AGENCY_MANAGER(派遣元責任者), AGENCY_COMPLAINT, CLIENT_MANAGER(派遣先責任者), COMMANDER(指揮命令者), BILLING_CONTACT, COMPLAINT_CONTACT, TIMECARD_APPROVER_1/2/3 |
| `actor_side` | VARCHAR(10) | No |  |  | MOTO / SAKI |
| `user_id` | BIGINT | Yes | NULL | IDX | logical FK (MOTO/SAKI user) |
| `dept_id` | BIGINT | Yes | NULL |  | logical FK phòng ban |
| `name_snapshot` | VARCHAR(100) | No |  |  | 氏名 |
| `role_title_snapshot` | VARCHAR(50) | Yes | NULL |  | 役職 |
| `tel_snapshot` | VARCHAR(15) | Yes | NULL |  | TEL |
| `dept_name_snapshot` | VARCHAR(100) | Yes | NULL |  | 部署名 |
| `email_snapshot` | VARCHAR(256) | Yes | NULL | IDX | Email (cho 勤怠承認者 — set/validate theo domain) |
| `created_at` | TIMESTAMP | No | NOW() |  |  |
| `updated_at` | TIMESTAMP | No | NOW() |  |  |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  |  |
| `updated_by` | BIGINT | Yes | NULL |  |  |
- **UNIQUE (Partial)** uq_contract_person_role: (contract_no, role_type) WHERE deleted_at IS NULL
- **RLS policy** — p_contract_person_role_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_contract_36_agreement` (Link Hợp đồng ↔ 36協定 — 1:N ≤4)

> Tách bảng vì M04 giám sát giới hạn OT cần join tới `cfg_moto_36_agreement`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** |  |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX |  |
| `agreement_id` | BIGINT | No |  | FK, IDX | `cfg_moto_36_agreement(id)` |
| `slot_no` | SMALLINT | No |  |  | 1-4 |
| `overtime_limit_snapshot` | VARCHAR(500) | Yes | NULL |  | 上限残業 snapshot lúc ký |
| `created_at` | TIMESTAMP | No | NOW() |  |  |
| `updated_at` | TIMESTAMP | No | NOW() |  |  |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  |  |
| `updated_by` | BIGINT | Yes | NULL |  |  |
- **UNIQUE (Partial)** uq_contract_36: (contract_no, slot_no) WHERE deleted_at IS NULL
- **RLS policy** — p_contract_36_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_contract_job_category` (Link Hợp đồng ↔ 台帳職種 — 1:N ≤3)

> Tách bảng vì 派遣先管理台帳 report/filter theo 職種.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** |  |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX |  |
| `category_code` | VARCHAR(3) | No |  | IDX | logical ref `global_master.mst_ledger_job_category(category_code)` |
| `category_name_snapshot` | VARCHAR(160) | No |  |  | snapshot tên 業務 |
| `slot_no` | SMALLINT | No |  |  | 1-3 (No1 cho phép 901-903/999) |
| `created_at` | TIMESTAMP | No | NOW() |  |  |
| `updated_at` | TIMESTAMP | No | NOW() |  |  |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  |  |
| `updated_by` | BIGINT | Yes | NULL |  |  |
- **UNIQUE (Partial)** uq_contract_job_category: (contract_no, slot_no) WHERE deleted_at IS NULL
- **RLS policy** — p_contract_job_category_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_contract_legal_holiday` (Quy định ngày nghỉ pháp định per-Hợp đồng — 1:1)

> Tách bảng vì M04 tính giờ làm ngày nghỉ pháp định đọc quy định này (khác `cfg_moto_legal_holiday` cấp company).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** |  |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, **UK** | (1:1) |
| `rule_type` | SMALLINT | No |  |  | 1=特定曜日 2=1週1日 3=4週4日 |
| `specified_weekday` | SMALLINT | Yes | NULL |  | 1=日…7=土 (rule=1) |
| `specified_start_weekday` | SMALLINT | Yes | NULL |  | 特定曜日起算 (rule=1) |
| `week1day_start_weekday` | SMALLINT | Yes | NULL |  | 1週1日起算曜日 (rule=2) |
| `week4day_start_date` | DATE | Yes | NULL |  | 4週4日起算日 (rule=3) |
| `determine_by_non_shift` | SMALLINT | No | 0 |  | 判定: シフト出勤予定日以外 |
| `determine_by_non_workday` | SMALLINT | No | 0 |  | 判定: 契約勤務日以外 |
| `created_at` | TIMESTAMP | No | NOW() |  |  |
| `updated_at` | TIMESTAMP | No | NOW() |  |  |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  |  |
| `updated_by` | BIGINT | Yes | NULL |  |  |
- **RLS policy** — p_contract_legal_holiday_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_contract_approval` (Lịch sử SAKI phê duyệt Hợp đồng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu `t_contract(contract_no)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI thực hiện phê duyệt |
| `saki_company_code` | VARCHAR(50) | No |  | **Logical FK**, IDX | Business code SAKI snapshot |
| `saki_user_id` | BIGINT | No |  | **Logical FK**, IDX | Tham chiếu ID người duyệt bên Schema SAKI (actor — người thực sự bấm nút duyệt) |
| `proxy_for_user_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | **[Reviewer Fix 1]** ID user SAKI có thẩm quyền gốc (代理処理 — duyệt thay). NULL = tự duyệt; NOT NULL = saki_user_id duyệt thay cho proxy_for_user_id. Bắt buộc cho thanh tra lao động giải trình. |
| `proxy_reason` | VARCHAR(500) | Yes | NULL |  | **[Reviewer Fix 1]** Lý do duyệt thay (vd "Giám đốc đi công tác"). Bắt buộc khi proxy_for_user_id IS NOT NULL — enforce ở App-layer (D13). |
| `action` | VARCHAR(20) | No |  |  | APPROVED, REJECTED |
| `comments` | TEXT | Yes | NULL |  | Lý do từ chối (nếu có) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_contract_status_history` (Nhật ký thay đổi trạng thái Hợp đồng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu t_contract(contract_no) |
| `old_status` | VARCHAR(20) | Yes | NULL |  | Trạng thái cũ |
| `new_status` | VARCHAR(20) | No |  |  | Trạng thái mới |
| `change_reason` | VARCHAR(500) | Yes | NULL |  | Lý do thay đổi (Từ chối, Rút lại, Hủy) |
| `actor_type` | VARCHAR(20) | No |  |  | MOTO_USER, SAKI_USER, SYSTEM |
| `actor_id` | BIGINT | Yes | NULL | IDX | ID người thực hiện thay đổi |
| `actor_tenant_id` | BIGINT | Yes | NULL | IDX | Tenant của actor; bắt buộc nếu actor_type = SAKI_USER |
| `actor_company_code` | VARCHAR(50) | Yes | NULL | IDX | Business code công ty của actor nếu có |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID của HTTP Request (từ API Gateway) |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết Cross-tenant Event / Workflow |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_contract_status_cross_actor: actor_type <> 'SAKI_USER' OR (actor_tenant_id IS NOT NULL AND actor_company_code IS NOT NULL)

### `t_contract_termination_detail` (Chi tiết Chấm dứt Hợp đồng — 1:1 với contract_type=4)

> Gắn 1:1 với bản ghi `t_contract` có `contract_type=4`. Lưu thông tin pháp lý bắt buộc khi chấm dứt. `notice_date` nullable (双方合意không cần) nhưng bắt buộc nếu MOTO đơn phương (type=2) — phản ánh nghĩa vụ 30日前通知 trong hợp đồng phái cử và được enforce bằng CHECK constraint tại DB layer.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, **UK** | Tham chiếu `t_contract(contract_no)` (1:1, bắt buộc contract_type=4) |
| `termination_reason_type` | SMALLINT | No |  | IDX | 1=双方合意 / 2=一方的解除(MOTO) / 3=一方的解除(SAKI) / 4=期間満了 / 5=その他 |
| `termination_reason_detail` | TEXT | Yes | NULL |  | Lý do chi tiết (bắt buộc khi termination_reason_type=5) |
| `notice_date` | DATE | Yes | NULL |  | Ngày gửi thông báo báo trước (NULL hợp lệ khi 双方合意; bắt buộc khi type=2 — enforce bằng CHECK) |
| `agreed_termination_date` | DATE | No |  |  | Ngày chấm dứt đã thỏa thuận (bản sao `contract_end_date` tại thời điểm ký — phục vụ audit) |
| `moto_submitted_at` | TIMESTAMPTZ | Yes | NULL |  | Thời điểm MOTO nộp yêu cầu chấm dứt |
| `saki_approved_at` | TIMESTAMPTZ | Yes | NULL |  | Thời điểm SAKI đồng ý chấm dứt |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **FK** fk_termination_contract: (contract_no) → t_contract(contract_no)
- **CHECK** chk_termination_reason_type: termination_reason_type IN (1, 2, 3, 4, 5)
- **CHECK** chk_termination_notice_required: termination_reason_type != 2 OR notice_date IS NOT NULL
- **CHECK** chk_termination_reason_detail: termination_reason_type != 5 OR (termination_reason_detail IS NOT NULL AND trim(termination_reason_detail) <> '')
- **RLS policy** — p_contract_termination_detail_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **Trigger** trg_contract_termination_detail_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_contract_document_artifact` (Metadata file Hợp đồng trên S3)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu `t_contract(contract_no)` |
| `storage_key` | VARCHAR(255) | No |  |  | Đường dẫn Object Key trên AWS S3 |
| `file_hash` | VARCHAR(255) | No |  |  | Mã băm file PDF chống giả mạo |
| `version` | SMALLINT | No | 1 |  | Phiên bản tài liệu |
| `locked_flg` | SMALLINT | No | 0 |  | 1: Bản chốt không thể xóa |
| `generated_at` | TIMESTAMP | No |  |  | Thời điểm thực tế hệ thống render file PDF |
| `source_status` | VARCHAR(20) | No |  |  | Trạng thái của nguồn dữ liệu lúc xuất file (VD: DRAFT, ACTIVE) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_contract_artifact_version: (company_id, contract_no, version)

### `t_moto_contract_upload_history` (Lịch sử MOTO Upload file Hợp đồng hàng loạt)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu User MOTO upload |
| `file_name` | VARCHAR(255) | No |  |  | Tên file gốc (CSV/TSV) |
| `status` | VARCHAR(20) | No | 'PROCESSING' |  | Trạng thái xử lý batch |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_moto_shift_upload_history` (Lịch sử MOTO Upload Lịch ca làm việc — シフト表アップロード/ダウンロード) [Follow-up A NEW]

> MOTO upload シフト表 TSV/CSV để lên kế hoạch ca cho staff (e-staffing pattern). Cũng track DOWNLOAD operation (`operation_type`) vì BR-1547 yêu cầu audit cho cả Download cho compliance. Khác `t_moto_contract_upload_history` về schema validation: shift có period_year_month + staff_count thay vì contract_no. Downstream INSERT vào `t_moto_shift_schedule`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | User MOTO thực hiện upload/download |
| `operation_type` | SMALLINT | No |  | IDX | 1=UPLOAD / 2=DOWNLOAD — BR-1547 audit cả 2 chiều |
| `file_name` | VARCHAR(255) | No |  |  | Tên file (TSV/CSV) |
| `s3_storage_key` | VARCHAR(500) | Yes | NULL |  | S3 key — bắt buộc với UPLOAD, NULL với DOWNLOAD (chỉ log thao tác) |
| `file_hash` | VARCHAR(64) | Yes | NULL |  | SHA-256 hash — chống giả mạo |
| `target_period_year_month` | VARCHAR(7) | No |  | IDX | Kỳ ca làm việc (YYYY-MM) |
| `total_records` | INTEGER | No | 0 |  | Tổng số dòng shift |
| `success_records` | INTEGER | No | 0 |  | Dòng xử lý thành công (UPLOAD); = total cho DOWNLOAD |
| `failed_records` | INTEGER | No | 0 |  | Dòng lỗi (chỉ UPLOAD) |
| `affected_staff_count` | INTEGER | Yes | NULL |  | Số staff bị ảnh hưởng — phục vụ dashboard MOTO |
| `status` | VARCHAR(20) | No | 'PROCESSING' | IDX | PROCESSING, VALIDATING, COMPLETED, FAILED |
| `started_at` | TIMESTAMP | No | NOW() |  | Thời điểm bắt đầu |
| `completed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm hoàn thành |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_moto_shift_upload_period: (company_id, target_period_year_month, started_at DESC)
- **INDEX** idx_moto_shift_upload_status: (status) WHERE status IN ('PROCESSING', 'VALIDATING') AND deleted_at IS NULL
- **INDEX** idx_moto_shift_upload_operation: (operation_type, started_at DESC)
- **FK** fk_moto_shift_upload_user: (user_id) → mst_moto_user(id)
- **CHECK** chk_moto_shift_upload_operation_type: operation_type IN (1, 2)
- **CHECK** chk_moto_shift_upload_status: status IN ('PROCESSING', 'VALIDATING', 'COMPLETED', 'FAILED')
- **CHECK** chk_moto_shift_upload_records: total_records >= 0 AND success_records >= 0 AND failed_records >= 0 AND success_records + failed_records <= total_records
- **CHECK** chk_moto_shift_upload_period_format: target_period_year_month ~ '^[0-9]{4}-(0[1-9]|1[0-2])$'
- **CHECK** chk_moto_shift_upload_s3_consistency: operation_type = 2 OR s3_storage_key IS NOT NULL (UPLOAD bắt buộc có S3 key; DOWNLOAD không bắt buộc)
- **Trigger** trg_moto_shift_upload_updated_at: BEFORE UPDATE → fn_set_updated_at()

---

## Nhóm 8: Thông báo Đãi ngộ & Xung đột pháp lý (Từ SAKI)

### `t_treatment_notification` (Thông báo điều kiện đãi ngộ do SAKI cung cấp)

> Immutable sau khi `status='SENT'`: cập nhật bằng cách tạo version mới (`version`+1), set `superseded_by_id` trên version cũ. Tuân thủ BR-054 + 電子帳簿保存法.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO tenant) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI gửi thông báo |
| `status` | VARCHAR(20) | No | 'DRAFT' | IDX | DRAFT → SENT → VIEWED (F-0501/F-0502 filter, gate check F-0507) |
| `treatment_decision_method` | SMALLINT | No |  | IDX | 1=労使協定方式 / 2=派遣先均等・均衡方式 (F-0501/F-0502 filter, BR-052) |
| `version` | SMALLINT | No | 1 |  | Phiên bản thông báo — tăng khi SAKI cần cập nhật thông tin đối ngộ (BR-054) |
| `superseded_by_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ tới version mới thay thế (BR-054 immutability pattern) |
| `notification_date` | DATE | No |  | IDX | Ngày SAKI phát hành thông báo chính thức |
| `payload` | JSONB | No |  |  | Chi tiết thông tin đãi ngộ (display-only: lương/phụ cấp/phúc lợi/đào tạo; schema phân nhánh theo treatment_decision_method; không filter/join/calc → hợp lệ D3.3) |
| `viewed_at` | TIMESTAMPTZ | Yes | NULL |  | Thời điểm MOTO đầu tiên mở xem (trigger status → VIEWED) |
| `viewed_by_moto_user_id` | BIGINT | Yes | NULL |  | Logical FK — user MOTO đã xem đầu tiên |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **FK** fk_treatment_superseded: (superseded_by_id) → t_treatment_notification(id)
- **INDEX** idx_treatment_saki_tenant: (saki_tenant_id)
- **INDEX** idx_treatment_status: (status)
- **INDEX** idx_treatment_method: (treatment_decision_method)
- **INDEX** idx_treatment_notification_date: (notification_date)
- **CHECK** chk_treatment_status: status IN ('DRAFT', 'SENT', 'VIEWED')
- **CHECK** chk_treatment_method: treatment_decision_method IN (1, 2)
- **RLS policy** — p_treatment_notification_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)
- **RLS policy** — p_treatment_notification_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **Trigger** trg_treatment_notification_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_treatment_notification_contract` (Link Thông báo Đối ngộ ↔ Hợp đồng — M:N)

> BR-055: Một thông báo đối ngộ có thể liên kết với nhiều hợp đồng của cùng cặp SAKI-MOTO. F-0507 gate check dùng bảng này để verify hợp đồng X đã có thông tin đối ngộ hợp lệ.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `treatment_notification_id` | BIGINT | No |  | FK, IDX | Tham chiếu `t_treatment_notification(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu `t_contract(contract_no)` |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_treatment_notification_contract: (treatment_notification_id, contract_no) WHERE deleted_at IS NULL
- **FK** fk_tnc_notification: (treatment_notification_id) → t_treatment_notification(id)
- **FK** fk_tnc_contract: (contract_no) → t_contract(contract_no)
- **RLS policy** — p_treatment_notification_contract_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **Trigger** trg_treatment_notification_contract_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_notification_office_conflict` (Chi tiết giới hạn phái cử theo Office SAKI — 事業所単位)

> Audit record — SAKI đã thông báo ngày xung đột cấp sự nghiệp cho MOTO. Immutable sau khi tạo. KHÁC với `t_office_conflict_tracking` (batch-computed live state).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | DENORM từ `t_treatment_notification` — phục vụ RLS SAKI |
| `treatment_notification_id` | BIGINT | No |  | FK, IDX | Tham chiếu `t_treatment_notification(id)` |
| `saki_office_id` | BIGINT | No |  | **Logical FK**, IDX | 派遣先事業所 — logical FK → saki schema |
| `conflict_date` | DATE | No |  |  | 事業所単位抵触日 được SAKI thông báo |
| `notification_level` | SMALLINT | No | 1 | IDX | Mức cảnh báo tại thời điểm thông báo: 1=INFO(6ヶ月前) / 2=WARNING(3ヶ月前) / 3=CRITICAL(1ヶ月前) / 4=EXPIRED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **FK** fk_office_conflict_notification: (treatment_notification_id) → t_treatment_notification(id)
- **INDEX** idx_office_conflict_saki_tenant: (saki_tenant_id)
- **INDEX** idx_office_conflict_office: (saki_office_id)
- **CHECK** chk_office_conflict_level: notification_level IN (1, 2, 3, 4)
- **RLS policy** — p_office_conflict_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **RLS policy** — p_office_conflict_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)
- **Trigger** trg_notification_office_conflict_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_notification_individual_conflict` (Chi tiết giới hạn phái cử theo cá nhân — 個人単位)

> Audit record — SAKI đã thông báo ngày xung đột cấp cá nhân cho MOTO (BR-068). Immutable sau khi tạo. KHÁC với `t_individual_conflict_tracking` (batch-computed live state).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | DENORM từ `t_treatment_notification` — phục vụ RLS SAKI |
| `treatment_notification_id` | BIGINT | No |  | FK, IDX | Tham chiếu `t_treatment_notification(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `saki_org_unit_id` | BIGINT | No |  | **Logical FK**, IDX | 組織単位 (部署) của SAKI — BR-062: SAKI đăng ký danh sách |
| `conflict_date` | DATE | No |  |  | 個人単位抵触日 được SAKI thông báo |
| `notification_level` | SMALLINT | No | 1 | IDX | Mức cảnh báo: 1=INFO(6ヶ月前) / 2=WARNING(3ヶ月前) / 3=CRITICAL(1ヶ月前) / 4=EXPIRED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_individual_conflict_notification: (treatment_notification_id, staff_id, saki_org_unit_id) WHERE deleted_at IS NULL
- **FK** fk_individual_conflict_notification: (treatment_notification_id) → t_treatment_notification(id)
- **FK** fk_individual_conflict_staff: (staff_id) → mst_staff(id)
- **INDEX** idx_individual_conflict_saki_tenant: (saki_tenant_id)
- **INDEX** idx_individual_conflict_staff: (staff_id)
- **INDEX** idx_individual_conflict_org_unit: (saki_org_unit_id)
- **CHECK** chk_individual_conflict_level: notification_level IN (1, 2, 3, 4)
- **RLS policy** — p_individual_conflict_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **RLS policy** — p_individual_conflict_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)
- **Trigger** trg_notification_individual_conflict_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_office_conflict_tracking` (Theo dõi 抵触日 cấp sự nghiệp — Batch-computed Live State)

> Batch job chạy 1x/ngày (EventBridge). Ghi nhận trạng thái live của mốc 3 năm per saki_office. Cooling Period reset → INSERT row mới (reset_flg=0), set row cũ reset_flg=1 (SCD Type 2 — giữ audit trail đầy đủ). KHÁC với `t_notification_office_conflict` (audit record thông báo đã gửi).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI — cross-tenant RLS |
| `saki_office_id` | BIGINT | No |  | **Logical FK**, IDX | 派遣先事業所 — logical FK → saki schema |
| `period_start_date` | DATE | No |  |  | Ngày bắt đầu chu kỳ 3 năm hiện tại (ngày dispatch đầu tiên tại office này trong chu kỳ) |
| `conflict_date` | DATE | No |  | IDX | 事業所単位抵触日 = period_start_date + 3 năm |
| `extended_conflict_date` | DATE | Yes | NULL | IDX | Ngày xung đột sau gia hạn (update khi có bản ghi `t_office_dispatch_extension` hợp lệ) |
| `warning_level` | SMALLINT | No | 0 | IDX | 0=安全 / 1=INFO(6ヶ月前) / 2=WARNING(3ヶ月前) / 3=CRITICAL(1ヶ月前) / 4=EXPIRED |
| `reset_flg` | SMALLINT | No | 0 | IDX | 0=chu kỳ đang active / 1=đã reset do Cooling Period (row lịch sử — SCD Type 2) |
| `cooling_period_start_date` | DATE | Yes | NULL |  | Ngày bắt đầu Cooling Period (ngày kết thúc dispatch cuối cùng tại office này) |
| `aggregated_at` | TIMESTAMPTZ | Yes | NULL | IDX | Thời điểm batch job cập nhật gần nhất (1x/ngày; không realtime — D11: 36協定 pattern) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_office_conflict_tracking_active: (company_id, saki_office_id) WHERE reset_flg = 0 AND deleted_at IS NULL
- **INDEX** idx_office_conflict_tracking_saki_tenant: (saki_tenant_id)
- **INDEX** idx_office_conflict_tracking_office: (saki_office_id)
- **CHECK** chk_office_conflict_warning: warning_level IN (0, 1, 2, 3, 4)
- **RLS policy** — p_office_conflict_tracking_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **RLS policy** — p_office_conflict_tracking_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)
- **Trigger** trg_office_conflict_tracking_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_individual_conflict_tracking` (Theo dõi 抵触日 cấp cá nhân — Batch-computed Live State)

> Batch job chạy 1x/ngày. Track per staff+saki_org_unit. Cooling Period reset → INSERT row mới, set reset_flg=1 trên row cũ (SCD Type 2). Cấp cá nhân KHÔNG có gia hạn (BR-061) — không có extended_conflict_date.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI — cross-tenant RLS |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `saki_office_id` | BIGINT | No |  | **Logical FK**, IDX | 派遣先事業所 |
| `saki_org_unit_id` | BIGINT | No |  | **Logical FK**, IDX | 組織単位 (部署) của SAKI — BR-062: SAKI đăng ký danh sách trong hệ thống |
| `period_start_date` | DATE | No |  |  | Ngày staff bắt đầu làm tại saki_org_unit này (chu kỳ hiện tại) |
| `conflict_date` | DATE | No |  | IDX | 個人単位抵触日 = period_start_date + 3 năm |
| `warning_level` | SMALLINT | No | 0 | IDX | 0=安全 / 1=INFO(6ヶ月前) / 2=WARNING(3ヶ月前) / 3=CRITICAL(1ヶ月前) / 4=EXPIRED |
| `reset_flg` | SMALLINT | No | 0 | IDX | 0=chu kỳ đang active / 1=đã reset do Cooling Period (row lịch sử) |
| `cooling_period_start_date` | DATE | Yes | NULL |  | Ngày kết thúc dispatch tại org_unit này (bắt đầu đếm Cooling Period — BR-064: calendar months) |
| `aggregated_at` | TIMESTAMPTZ | Yes | NULL | IDX | Thời điểm batch job cập nhật gần nhất |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_individual_conflict_tracking_active: (company_id, staff_id, saki_office_id, saki_org_unit_id) WHERE reset_flg = 0 AND deleted_at IS NULL
- **INDEX** idx_individual_conflict_tracking_saki_tenant: (saki_tenant_id)
- **INDEX** idx_individual_conflict_tracking_staff: (staff_id)
- **INDEX** idx_individual_conflict_tracking_org_unit: (saki_org_unit_id)
- **CHECK** chk_individual_conflict_warning: warning_level IN (0, 1, 2, 3, 4)
- **RLS policy** — p_individual_conflict_tracking_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **RLS policy** — p_individual_conflict_tracking_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)
- **Trigger** trg_individual_conflict_tracking_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_office_dispatch_extension` (Gia hạn 抵触日 cấp sự nghiệp — 過半数代表者 記録)

> BR-057, BR-058: Gia hạn cấp sự nghiệp BẮT BUỘC ghi nhận ý kiến 過半数代表者 trước ngày xung đột. Vô hiệu nếu không có bản ghi này hoặc result=0. Immutable sau khi tạo.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Nhân viên áp dụng |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Hợp đồng gốc |
| `application_month` | VARCHAR(7) | No |  | IDX | Tháng xin vượt trần (YYYY-MM) |
| `reason` | TEXT | No |  |  | Lý do công việc đột xuất |
| `period_type` | SMALLINT | No | 1 | IDX | 1=月間 2=年間 |
| `status` | VARCHAR(20) | No | 'NOT_APPLIED' | IDX | NOT_REQUESTED(未申請), NOT_APPLIED(未適用), APPLIED(適用), DEFERRED(見送り), OTHER_COMPANY_REQUESTING(他社申請中) |
| `requested_overtime_minutes` | INT | Yes | NULL |  | 申請時間外労働時間 (dự kiến) |
| `applied_count` | SMALLINT | No | 0 |  | 適用回数 (trong tháng — bị áp dụng bao nhiêu ngày) |
| `approval_status` | SMALLINT | No | 1 | IDX | **[M10 NEW]** 1=PENDING / 2=APPROVED / 3=REJECTED / 4=AUTO_APPROVED (system auto-approve nếu hệ thống policy cho phép) |
| `approved_by_moto_user_id` | BIGINT | Yes | NULL | FK | **[M10 NEW]** MOTO user phê duyệt application — NULL nếu auto/system |
| `approved_at` | TIMESTAMP | Yes | NULL |  | **[M10 NEW]** Thời điểm phê duyệt |
| `rejection_reason` | TEXT | Yes | NULL |  | **[M10 NEW]** Bắt buộc nếu approval_status=REJECTED |
| `annual_application_sequence` | SMALLINT | No | 0 |  | **[M10 NEW]** Thứ tự lần thứ N trong năm áp dụng 特別条項 (1, 2, 3...). CHECK ≤6 — BR-1504 |
| `expiration_date` | DATE | Yes | NULL |  | **[M10 NEW]** Hạn hiệu lực của application này (typically end of application_month) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
- **FK** fk_office_extension_tracking: (office_conflict_tracking_id) → t_office_conflict_tracking(id)
- **INDEX** idx_office_extension_saki_tenant: (saki_tenant_id)
- **INDEX** idx_office_extension_office: (saki_office_id)
- **CHECK** chk_office_extension_result: result IN (0, 1)
- **CHECK** chk_office_extension_representative: representative_type IN (1, 2)
- **CHECK** chk_office_extension_new_date: result = 0 OR new_conflict_date IS NOT NULL
- **CHECK** chk_office_extension_period: extension_period_months IS NULL OR (extension_period_months BETWEEN 1 AND 36)
- **RLS policy** — p_office_dispatch_extension_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **RLS policy** — p_office_dispatch_extension_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)
- **Trigger** trg_office_dispatch_extension_updated_at: BEFORE UPDATE → fn_set_updated_at()

---

## Nhóm 9: Quản lý Chấm công (Attendance - Volume cực lớn tới 10M records)

### `t_attendance_daily` (Chấm công hàng ngày của Staff)

*Được phi chuẩn hóa (Denormalized) bằng cách copy `saki_tenant_id` từ Hợp đồng sang để ORM Query nhanh hơn khi SAKI xem chấm công.*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu t_contract(contract_no) |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_staff(id) |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Phục vụ Row-Level Security cho SAKI |
| `attendance_category_id` | BIGINT | No |  | FK | Loại chấm công (Đi làm, Nghỉ phép) |
| `workplace_type_id` | BIGINT | Yes | NULL | FK | Trỏ tới mst_moto_workplace_type. Ghi nhận hôm đó Staff làm ở đâu (Công ty hay Tại nhà). |
| `work_date` | DATE | No |  | IDX | Ngày làm việc thực tế |
| `start_time` | SMALLINT | Yes | NULL |  | Giờ vào ca (Tính bằng phút từ 00:00) |
| `end_time` | SMALLINT | Yes | NULL |  | Giờ ra ca (Tính bằng phút từ 00:00) |
| `break_time_minutes` | SMALLINT | No | 0 |  | Tổng số phút nghỉ giữa ca |
| `regular_work_minutes` | SMALLINT | No | 0 |  | Giờ hành chính hợp lệ (Khôi phục để tính lương) |
| `overtime_minutes` | SMALLINT | No | 0 |  | Giờ làm thêm (Khôi phục) |
| `midnight_work_minutes` | SMALLINT | No | 0 |  | Giờ ca đêm (Khôi phục) |
| `holiday_work_minutes` | SMALLINT | No | 0 |  | Giờ làm ngày nghỉ (Khôi phục) |
| `status` | VARCHAR(20) | No | 'DRAFT' | IDX | 申請中(SUBMITTED), 承認済(APPROVED), 引戻し(WITHDRAWN), 却下(REJECTED), 承認取消(APPROVAL_CANCELLED), 代理申請(PROXY), DRAFT |
| `punch_in_minutes` | SMALLINT | Yes | NULL |  | 打刻出勤時刻 (phút từ 00:00) |
| `punch_out_minutes` | SMALLINT | Yes | NULL |  | 打刻退勤時刻 (phút; cho phép >1440 nếu ca qua đêm) |
| `break_time_minutes_2` | SMALLINT | Yes | NULL |  | 休憩時間2 (break_time_minutes hiện có = 休憩1) |
| `break_time_minutes_3` | SMALLINT | Yes | NULL |  | 休憩時間3 |
| `midnight_break_minutes` | SMALLINT | Yes | NULL |  | 深夜休憩 (phút) |
| `substitute_holiday_date` | DATE | Yes | NULL | IDX | 振替休日 (ngày nghỉ bù; cho 区分=振替出勤; self-ref logic tới ngày làm bù) |
| `legal_holiday_work_flg` | SMALLINT | No | 0 |  | 法定休日出勤判定: 0/1 |
| `remark` | VARCHAR(500) | Yes | NULL |  | 備考 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
- **UNIQUE** uq_attendance_daily: (company_id, contract_no, work_date)
- **INDEX** idx_daily_saki_tenant: (saki_tenant_id)
- **FK** fk_daily_workplace: (workplace_type_id) → mst_moto_workplace_type (id)
- **RLS policy** — p_att_daily_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true))

### `t_attendance_daily_approval` (Lịch sử SAKI duyệt chấm công ngày)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `attendance_daily_id` | BIGINT | No |  | FK, IDX | Tham chiếu bản ghi chấm công ngày |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI thực hiện duyệt |
| `saki_company_code` | VARCHAR(50) | No |  | **Logical FK**, IDX | Business code SAKI snapshot |
| `saki_user_id` | BIGINT | No |  | **Logical FK**, IDX | Người SAKI thực hiện duyệt (actor) |
| `proxy_for_user_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | **[Reviewer Fix 1]** ID user SAKI có thẩm quyền gốc khi 代理処理. NULL = tự duyệt. |
| `proxy_reason` | VARCHAR(500) | Yes | NULL |  | **[Reviewer Fix 1]** Lý do duyệt thay. |
| `status` | VARCHAR(20) | No |  |  | APPROVED, REJECTED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_attendance_close` (Chốt công cuối kỳ / 勤怠締め)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | DENORMALIZE từ t_contract. RLS key. |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu `t_contract(contract_no)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `closing_month` | VARCHAR(7) | No |  | IDX | Tháng chốt công (YYYY-MM) |
| `total_work_minutes` | INT | No | 0 |  | Tổng số phút làm việc trong kỳ |
| `status` | VARCHAR(20) | No | 'SUBMITTED' | IDX | SUBMITTED, APPROVED_SAKI, UNLOCKED |
| `work_days` | NUMERIC(4,1) | Yes | NULL |  | 出勤日数 |
| `absence_days` | NUMERIC(4,1) | Yes | NULL |  | 欠勤日数 |
| `paid_leave_days` | NUMERIC(4,1) | Yes | NULL |  | 年休日数 (cho phép 0.5) |
| `holiday_work_days` | NUMERIC(4,1) | Yes | NULL |  | 休出日数 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
- **UNIQUE** uq_attendance_close: (company_id, contract_no, closing_month)
- **INDEX** idx_close_saki_tenant: (saki_tenant_id)
- **RLS policy** — p_att_close_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true))

### `t_attendance_close_approval` (Lịch sử SAKI duyệt chốt kỳ)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `attendance_close_id` | BIGINT | No |  | FK, IDX | Tham chiếu bảng chốt kỳ |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI thực hiện duyệt chốt |
| `saki_company_code` | VARCHAR(50) | No |  | **Logical FK**, IDX | Business code SAKI snapshot |
| `saki_user_id` | BIGINT | No |  | **Logical FK**, IDX | Người SAKI duyệt chốt (actor) |
| `proxy_for_user_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | **[Reviewer Fix 1]** ID user SAKI có thẩm quyền gốc khi 代理処理. NULL = tự duyệt. |
| `proxy_reason` | VARCHAR(500) | Yes | NULL |  | **[Reviewer Fix 1]** Lý do duyệt thay. |
| `status` | VARCHAR(20) | No |  |  | APPROVED, REJECTED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_working_time_monthly` (Tổng hợp Thời gian làm việc theo Tháng)

*Cực kỳ quan trọng: Bảng này Aggregate (Gộp) theo `staff_id` chứ không phải hợp đồng, để tính giới hạn tăng ca của con người theo luật lao động Nhật Bản.*

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `target_month` | VARCHAR(7) | No |  | IDX | Tháng tổng hợp (YYYY-MM) |
| `total_regular_minutes` | INT | No | 0 |  | Tổng phút giờ hành chính |
| `total_overtime_minutes` | INT | No | 0 |  | Tổng phút tăng ca (OT) |
| `total_midnight_minutes` | INT | No | 0 |  | Tổng phút ca đêm |
| `total_holiday_minutes` | INT | No | 0 |  | Tổng phút làm ngày nghỉ |
| `aggregation_start_date` | DATE | Yes | NULL |  | 集計対象期間開始日 (theo 起算日) |
| `aggregation_end_date` | DATE | Yes | NULL |  | 集計対象期間終了日 |
| `spec_flag` | SMALLINT | No | 1 |  | 様式: 1=新様式 2=旧様式 (quyết định 時間外 có gộp 法定休日 không) |
| `statutory_inside_minutes` | SMALLINT | Yes | NULL |  | 法定内労働時間 |
| `over_8h_daily_minutes` | INT | Yes | NULL |  | 1日8時間超 |
| `over_40h_weekly_minutes` | INT | Yes | NULL |  | 週40時間超 |
| `over_60h_monthly_minutes` | INT | Yes | NULL |  | 月60時間超 |
| `overtime_minutes_month` | INT | Yes | NULL |  | 時間外労働時間 (=8h超+40h超, KHÔNG gồm 法定休日) |
| `legal_holiday_work_minutes` | INT | Yes | NULL |  | 法定休日労働時間 |
| `monthly_overtime_plus_holiday` | INT | Yes | NULL |  | 1ヶ月時間外 (=時間外+法定休日; check trần <100h) |
| `overtime_plus_holiday_safety` | INT | Yes | NULL |  | 時間外+休日労働時間 (面接指導式: 総就業 −(暦日数/7)×40) |
| `work_days` | NUMERIC(4,1) | Yes | NULL |  | 出勤日数 |
| `absence_days` | NUMERIC(4,1) | Yes | NULL |  | 欠勤日数 |
| `paid_leave_days` | NUMERIC(4,1) | Yes | NULL |  | 年休日数 |
| `avg_2month_minutes` | INT | Yes | NULL |  | 複数月平均 2ヶ月 (時間外+法定休日, trần ≤80h) |
| `avg_3month_minutes` | INT | Yes | NULL |  | 複数月平均 3ヶ月 |
| `avg_4month_minutes` | INT | Yes | NULL |  | 複数月平均 4ヶ月 |
| `avg_5month_minutes` | INT | Yes | NULL |  | 複数月平均 5ヶ月 |
| `avg_6month_minutes` | INT | Yes | NULL |  | 複数月平均 6ヶ月 |
| `remaining_to_limit_multimonth` | INT | Yes | NULL |  | 上限までの残り時間 (複数月) |
| `overtime_minutes_prior_job` | INT | Yes | NULL |  | 前職分 時間外 (giờ làm công ty phái cử trước, cộng vào 36協定 năm) |
| `legal_holiday_minutes_prior_job` | INT | Yes | NULL |  | 前職分 法定休日労働 |
| `aggregated_at` | TIMESTAMP | Yes | NULL | IDX | 集計日時 (batch 1 lần/ngày, KHÔNG realtime; chỉ 当月+前月) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_working_time_monthly: (company_id, staff_id, target_month)

### `t_36_agreement_monthly` (Giám sát Hiệp định 36 theo Tháng) [M10 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `working_time_monthly_id` | BIGINT | No |  | FK, **UK** | Trỏ tới bảng tổng hợp tháng của Staff |
| `limit_hours` | INT | No |  |  | Giới hạn theo quy định (VD: 45) |
| `actual_hours` | DECIMAL(6,2) | No |  |  | Giờ OT thực tế đã làm |
| `warning_level` | SMALLINT | No | 0 | IDX | **[M10 Modified]** 0=NORMAL (<50%) / 1=50% / 2=60% / 3=70% / 4=80% CAUTION / 5=90% DANGER / 6=100%+ VIOLATION — BR-1513 phân tầng cảnh báo |
| `remaining_to_limit_minutes` | INT | Yes | NULL |  | 上限までの残り時間 |
| `over_limit_count` | SMALLINT | No | 0 |  | 超過回数 |
| `special_clause_limit_count` | SMALLINT | Yes | NULL |  | 特別条項限度回数 (≤6) |
| `special_clause_applied_count` | SMALLINT | No | 0 |  | 特別条項適用回数 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_36_monthly_warning_level: warning_level IN (0, 1, 2, 3, 4, 5, 6)

### `t_36_agreement_yearly` (Giám sát Hiệp định 36 theo Năm) [M10 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `target_year` | INT | No |  | IDX | Năm theo dõi |
| `limit_hours` | INT | No |  |  | Giới hạn năm (VD: 360 thường / 720 đặc biệt) |
| `actual_hours` | DECIMAL(6,2) | No | 0 |  | Tổng OT cộng dồn thực tế |
| `warning_level` | SMALLINT | No | 0 | IDX | **[M10 Modified]** 7 mức như monthly (0-6) cho phân tầng BR-1513 |
| `remaining_to_limit_minutes` | INT | Yes | NULL |  | 上限までの残り時間 |
| `over_limit_count` | SMALLINT | No | 0 |  | 超過回数 |
| `special_clause_count` | SMALLINT | No | 0 | IDX | **[M10 NEW]** Số tháng đã vượt 45h (áp dụng 特別条項) trong năm — BR-1504 ≤6, CHECK constraint enforce. Tính từ aggregate `t_special_clause_application` WHERE status=APPLIED |
| `annual_period_start_date` | DATE | Yes | NULL |  | **[M10 NEW]** Snapshot ngày bắt đầu năm tài chính từ `cfg_moto_36_agreement.annual_start_date` tại moment tính. Nếu NULL = năm dương lịch |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_36_yearly_warning_level: warning_level IN (0, 1, 2, 3, 4, 5, 6)
- **CHECK** chk_36_yearly_special_clause_count: special_clause_count >= 0 AND special_clause_count <= 6 (BR-1504 tuyệt đối)

### `t_special_clause_application` (Đơn xin áp dụng Điều khoản đặc biệt 36協定) [M10 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Nhân viên áp dụng |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Hợp đồng gốc |
| `application_month` | VARCHAR(7) | No |  | IDX | Tháng xin vượt trần (YYYY-MM) |
| `reason` | TEXT | No |  |  | Lý do công việc đột xuất |
| `period_type` | SMALLINT | No | 1 | IDX | 1=月間 2=年間 |
| `status` | VARCHAR(20) | No | 'NOT_APPLIED' | IDX | NOT_REQUESTED(未申請), NOT_APPLIED(未適用), APPLIED(適用), DEFERRED(見送り), OTHER_COMPANY_REQUESTING(他社申請中) |
| `requested_overtime_minutes` | INT | Yes | NULL |  | 申請時間外労働時間 (dự kiến) |
| `applied_count` | SMALLINT | No | 0 |  | 適用回数 (trong tháng — bị áp dụng bao nhiêu ngày) |
| `approval_status` | SMALLINT | No | 1 | IDX | **[M10 NEW]** 1=PENDING / 2=APPROVED / 3=REJECTED / 4=AUTO_APPROVED (system auto-approve nếu hệ thống policy cho phép) |
| `approved_by_moto_user_id` | BIGINT | Yes | NULL | FK | **[M10 NEW]** MOTO user phê duyệt application — NULL nếu auto/system |
| `approved_at` | TIMESTAMP | Yes | NULL |  | **[M10 NEW]** Thời điểm phê duyệt |
| `rejection_reason` | TEXT | Yes | NULL |  | **[M10 NEW]** Bắt buộc nếu approval_status=REJECTED |
| `annual_application_sequence` | SMALLINT | No | 0 |  | **[M10 NEW]** Thứ tự lần thứ N trong năm áp dụng 特別条項 (1, 2, 3...). CHECK ≤6 — BR-1504 |
| `expiration_date` | DATE | Yes | NULL |  | **[M10 NEW]** Hạn hiệu lực của application này (typically end of application_month) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_special_clause_approval_status: approval_status IN (1, 2, 3, 4)
- **CHECK** chk_special_clause_annual_seq: annual_application_sequence >= 0 AND annual_application_sequence <= 6 (BR-1504)
- **CHECK** chk_special_clause_rejection_reason: approval_status <> 3 OR (rejection_reason IS NOT NULL AND trim(rejection_reason) <> '')
- **CHECK** chk_special_clause_approved_consistency: approval_status NOT IN (2, 3, 4) OR (approved_at IS NOT NULL)
- **CHECK** chk_special_clause_application_month_format: application_month ~ '^[0-9]{4}-(0[1-9]|1[0-2])$'

### `t_attendance_email_history` (Lịch sử gửi email Chấm công/Nhắc nhở) [M08 DEPRECATED]

> **DEPRECATED M08:** Bảng này chỉ phục vụ lịch sử gửi email **liên quan tới Chấm công** (ALERT_36, APPROVAL_REMINDER). Từ M08, mọi email mới (universal) ghi vào `t_moto_email_send_history` để có universal tracking SES status + retry + audit (BR-109/110/111). Dữ liệu cũ trong bảng này **giữ nguyên read-only** (không migrate, không xóa) để đáp ứng audit retention 1 năm + tránh rủi ro migration sai lệch. Bảng sẽ được xóa sau khi dữ liệu cũ hết retention (1 năm sau ngày M08 deploy).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | Yes | NULL | FK, IDX | Hợp đồng liên quan |
| `email_type` | VARCHAR(50) | No |  |  | Loại: ALERT_36, APPROVAL_REMINDER |
| `recipient_email` | VARCHAR(255) | No |  |  | Email người nhận |
| `sent_at` | TIMESTAMP | No | NOW() |  | Thời gian hệ thống gửi |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 10: Quản lý Hóa đơn (Invoices - Qualified Invoices Support)

*Đã tái cấu trúc để hỗ trợ "Hóa đơn đủ điều kiện" (適格請求書): Thuế 10% và 8% phải được cộng tổng trước, sau đó mới nhân tỷ lệ thuế và làm tròn 1 lần duy nhất.*

### `t_invoice` (Header Hóa đơn MOTO gửi SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Tham chiếu Tenant SAKI |
| `saki_company_code` | VARCHAR(50) | No |  | **Logical FK** , IDX | Mã Business Code SAKI nhận hóa đơn (RLS vẫn theo saki_tenant_id) |
| `client_id` | BIGINT | No |  | FK, IDX | Tham chiếu cfg_moto_client_company(id) |
| `invoice_no` | VARCHAR(50) | No |  | **UNIQUE** uq_moto_invoice_no: (company_id, invoice_no) | Số hóa đơn duy nhất (Internal ID) |
| `invoice_code` | VARCHAR(50) | No |  | IDX | Mã hóa đơn hiển thị cho khách hàng (Khôi phục) |
| `invoice_group_id` | BIGINT | No |  | IDX | ID nhóm cố định cho toàn bộ chuỗi version/reissue của hóa đơn này |
| `version` | SMALLINT | No | 1 |  | Phiên bản hóa đơn trong cùng `invoice_group_id`; tăng khi MOTO reissue |
| `superseded_by_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ tới ID của version mới thay thế nó khi reissue |
| `billing_month` | VARCHAR(7) | No |  | IDX | Tháng tính cước (YYYY-MM) |
| `billing_start_date` | DATE | No |  |  | Ngày bắt đầu kỳ tính cước (Khôi phục) |
| `billing_end_date` | DATE | No |  |  | Ngày kết thúc kỳ tính cước (Khôi phục) |
| `total_amount_inc` | DECIMAL(12,2) | No | 0 |  | Tổng tiền sau thuế yêu cầu thanh toán (grand total toàn hóa đơn — tổng tất cả nhóm thuế) |
| `payment_due_date` | DATE | No |  |  | Hạn thanh toán |
| `source_status` | VARCHAR(20) | No | 'DRAFT' | IDX | Trạng thái nguồn từ MOTO: DRAFT, ISSUED, SUPERSEDED, WITHDRAWN, CANCELLED |
| `decision_status` | VARCHAR(30) | No | 'PENDING' | IDX | Quyết định của SAKI: PENDING, APPROVED, REJECTED, CORRECTION_REQUESTED |
| `payment_status` | VARCHAR(20) | No | 'UNPAID' | IDX | Trạng thái thanh toán: UNPAID, PARTIAL, PAID |
| `issued_at` | TIMESTAMP | Yes | NULL |  | Thời điểm MOTO phát hành hóa đơn sang SAKI |
| `approved_at` | TIMESTAMP | Yes | NULL |  | Thời điểm SAKI approve hóa đơn |
| `reissued_from_invoice_id` | BIGINT | Yes | NULL | FK, IDX | Nếu là hóa đơn reissue, trỏ về version trước đó |
| `correction_reason` | TEXT | Yes | NULL |  | Lý do SAKI yêu cầu sửa hoặc MOTO giải trình khi reissue |
| `issue_date` | DATE | Yes | NULL |  | 請求年月日 (ngày phát hành in trên hóa đơn) |
| `invoice_format_type` | SMALLINT | No | 1 | IDX | 0=従来請求書 1=適格請求書 |
| `layout_version` | VARCHAR(10) | Yes | NULL |  | レイアウトバージョン (請H_1/請H_2) |
| `registration_no_snapshot` | VARCHAR(20) | Yes | NULL |  | 適格請求書発行事業者登録番号 (T+13) snapshot bất biến trên hóa đơn |
| `recipient_code` | VARCHAR(16) | Yes | NULL |  | 請求書送付先コード |
| `recipient_name_snapshot` | VARCHAR(100) | Yes | NULL |  | 請求書送付先名 (snapshot) |
| `contact_name` | VARCHAR(40) | Yes | NULL |  | 問い合わせ先 |
| `contact_tel` | VARCHAR(15) | Yes | NULL |  | 問い合わせ先TEL |
| `transport_subtotal` | INT | Yes | NULL |  | 交通費相当額小計 (tổng giao thông — không phân theo thuế suất) |
| `transport_tax` | INT | Yes | NULL |  | 交通費相当額消費税 (適格để trống vì gộp vào 10%) |
| `transport_total` | INT | Yes | NULL |  | 交通費相当額合計 |
| `reimbursement_total` | INT | Yes | NULL |  | 立替金額計 (nhóm その他) |
| `other_add_deduct_total` | INT | Yes | NULL |  | その他加算・控除計 |
| `grand_total` | INT | Yes | NULL |  | 請求総計金額 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `draft_expires_at` | TIMESTAMP | Yes | NULL | IDX | Hạn lưu nháp (D18.10) = thời điểm save + TTL; NULL khi không ở trạng thái draft. Set lúc save-draft, clear (NULL) khi submit. |
| `expiration_status` | VARCHAR(10) | Yes | NULL |  | normal / warning / expired (D18.10); NULL khi không phải draft; batch hằng ngày maintain tier. |
- **RLS policy** — p_invoice_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true))
- **Generation Rule (Phase 3 Invariant):** Cột `invoice_no` BẮT BUỘC do Sequence Service cấp phát theo định dạng `{tenant_prefix}-{company_prefix}-I{YYMM}-{seq}`. Cấm Client tự nhập.
- **UNIQUE** uq_invoice_version: (company_id, client_id, invoice_group_id, version)
- **UNIQUE (Partial)** uq_invoice_single_draft: (company_id, client_id, invoice_group_id) WHERE source_status = 'DRAFT' AND deleted_at IS NULL
- **UNIQUE (Partial)** uq_invoice_single_active_issued: (company_id, client_id, invoice_group_id) WHERE source_status = 'ISSUED' AND decision_status IN ('PENDING', 'APPROVED', 'CORRECTION_REQUESTED') AND deleted_at IS NULL
- **FK** fk_invoice_superseded: (superseded_by_id) → t_invoice(id)
- **FK** fk_invoice_reissued_from: (reissued_from_invoice_id) → t_invoice(id)
- **Versioning Rule:** Hóa đơn là dữ liệu pháp lý bất biến. SAKI approve/reject/request correction không được ghi đè nội dung số tiền/chi tiết của version cũ. Khi MOTO reissue, hệ thống insert `version = N+1`, set version cũ `source_status = 'SUPERSEDED'`, và lưu `superseded_by_id`.
- **[R3/R4 Remediation 2026-06-28]** Cờ điều hành vận hành (lock/修正不可/月次締め/取消) đã chuyển sang `t_invoice_state` (1:1 per group, mutable). Tax grouping (10%/8%/非課税) đã chuyển sang `t_invoice_amount_summary` (1:N per version, immutable snapshot).
- **INDEX (Partial)** idx_invoice_draft_cleanup: (source_status, draft_expires_at) WHERE deleted_at IS NULL

### `t_invoice_approval` (Lịch sử SAKI approve/reject/request correction Hóa đơn)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_id` | BIGINT | No |  | FK, IDX | Version hóa đơn được SAKI thao tác |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI thực hiện thao tác |
| `saki_company_code` | VARCHAR(50) | No |  | **Logical FK**, IDX | Business code SAKI snapshot |
| `saki_user_id` | BIGINT | No |  | **Logical FK**, IDX | User SAKI thực hiện approve/reject/request correction (actor) |
| `proxy_for_user_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | **[Reviewer Fix 1]** ID user SAKI có thẩm quyền gốc khi 代理処理. NULL = tự duyệt. |
| `proxy_reason` | VARCHAR(500) | Yes | NULL |  | **[Reviewer Fix 1]** Lý do duyệt thay. Bắt buộc khi proxy_for_user_id IS NOT NULL. |
| `action` | VARCHAR(30) | No |  | IDX | APPROVE, REJECT, REQUEST_CORRECTION |
| `reason` | TEXT | Yes | NULL |  | Lý do reject/request correction hoặc ghi chú approve |
| `request_id` | VARCHAR(100) | No |  | IDX | Request ID từ API Gateway/service |
| `correlation_id` | VARCHAR(100) | No |  | IDX | ID truy vết workflow cross-tenant |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_invoice_approval_action: action IN ('APPROVE', 'REJECT', 'REQUEST_CORRECTION')
- **CHECK** chk_invoice_correction_reason: action NOT IN ('REJECT', 'REQUEST_CORRECTION') OR (reason IS NOT NULL AND trim(reason) <> '')

### `t_invoice_state` (Trạng thái vận hành của nhóm Hóa đơn — Mutable, 1:1 với invoice_group_id)

> **[R3 — Remediation 2026-06-28]** Tách ra khỏi `t_invoice` (immutable). Các cờ điều hành (lock/修正不可/月次締め/取消) thuộc về "nhóm hóa đơn" (group), không phải một version cụ thể. Khi MOTO reissue (version mới), lock state vẫn giữ nguyên theo group. Bảng này là MUTABLE — được UPDATE tự do; `t_invoice` là immutable — chỉ INSERT.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO tenant) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Tenant SAKI (RLS cross-tenant) |
| `invoice_group_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `t_invoice.invoice_group_id` (1:1 per group) |
| `locked_flg` | SMALLINT | No | 0 | IDX | SAKI ロック (lock khi download → MOTO không sửa/in được) |
| `locked_at` | TIMESTAMPTZ | Yes | NULL |  | Thời điểm SAKI lock |
| `locked_by_saki_user_id` | BIGINT | Yes | NULL |  | Logical FK user SAKI đã lock |
| `no_modify_flg` | SMALLINT | No | 0 |  | SAKI 修正不可 (cấm MOTO sửa vĩnh viễn) |
| `no_modify_set_at` | TIMESTAMPTZ | Yes | NULL |  | Thời điểm set 修正不可 |
| `no_modify_by_saki_user_id` | BIGINT | Yes | NULL |  | Logical FK user SAKI đã set |
| `monthly_closed_flg` | SMALLINT | No | 0 | IDX | SAKI 月次締め済 (chốt tháng) |
| `monthly_closed_at` | TIMESTAMPTZ | Yes | NULL |  | Thời điểm 月次締め |
| `monthly_closed_by_saki_user_id` | BIGINT | Yes | NULL |  | Logical FK user SAKI đã締め |
| `cancelled_flg` | SMALLINT | No | 0 | IDX | 取消 (MOTO hủy hóa đơn) |
| `cancelled_at` | TIMESTAMPTZ | Yes | NULL |  | Thời điểm hủy |
| `cancelled_by` | BIGINT | Yes | NULL |  | User MOTO đã hủy |
| `created_at` | TIMESTAMPTZ | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMPTZ | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `created_by` | BIGINT | Yes | NULL |  | ID người dùng tạo |
| `updated_by` | BIGINT | Yes | NULL |  | ID người dùng cập nhật |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
- **RLS policy** — p_invoice_state_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)
- **Trigger** trg_invoice_state_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_invoice_amount_summary` (Tổng tiền phân theo nhóm thuế suất — 適格請求書 tax grouping)

> **[R4 — Remediation 2026-06-28]** Thay thế flat columns tax trên `t_invoice`. Mỗi row = 1 nhóm thuế suất (10%/8%/非課税). Trong 適格請求書: tính thuế 1 lần/nhóm; 特別調整額 cũng thuộc nhóm. Bảng này là phần của snapshot immutable hóa đơn.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO tenant) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_id` | BIGINT | No |  | FK, IDX | Tham chiếu `t_invoice(id)` — version cụ thể |
| `tax_rate_category` | SMALLINT | No |  | IDX | 0=標準税率10% / 1=軽減税率8% / 2=非課税・不課税 |
| `display_order` | SMALLINT | No |  |  | Thứ tự hiển thị trên hóa đơn |
| `subtotal_ex_tax` | DECIMAL(12,2) | No | 0 |  | 小計 (税抜) — tổng trước thuế của nhóm này |
| `special_adjustment_1` | INT | Yes | NULL |  | 特別調整額1 (thuộc nhóm thuế này) |
| `special_adjustment_2` | INT | Yes | NULL |  | 特別調整額2 (thuộc nhóm thuế này) |
| `tax_amount` | DECIMAL(12,2) | No | 0 |  | 消費税額 — tính 1 lần/nhóm, làm tròn 1 lần |
| `total_inc_tax` | DECIMAL(12,2) | No | 0 |  | 請求合計 (税込) của nhóm này |
| `created_at` | TIMESTAMPTZ | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMPTZ | No | NOW() |  | Thời điểm cập nhật |
| `created_by` | BIGINT | Yes | NULL |  | ID người dùng tạo |
| `updated_by` | BIGINT | Yes | NULL |  | ID người dùng cập nhật |
- **UNIQUE** uq_invoice_amount_summary: (invoice_id, tax_rate_category)
- **CHECK** chk_tax_rate_category: tax_rate_category IN (0, 1, 2)
- **RLS policy** — p_invoice_amount_summary_saki: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **Note (Immutable):** Rows trong bảng này là snapshot bất biến của hóa đơn đã phát hành. Không UPDATE sau khi `t_invoice.source_status = 'ISSUED'`. Nếu MOTO reissue → INSERT row mới với `invoice_id` của version mới.

### `t_invoice_status_history` (Nhật ký trạng thái Hóa đơn)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_id` | BIGINT | No |  | FK, IDX | Version hóa đơn bị thay đổi trạng thái |
| `from_source_status` | VARCHAR(20) | Yes | NULL |  | Trạng thái nguồn trước khi đổi |
| `to_source_status` | VARCHAR(20) | Yes | NULL |  | Trạng thái nguồn sau khi đổi |
| `from_decision_status` | VARCHAR(30) | Yes | NULL |  | Trạng thái quyết định trước khi đổi |
| `to_decision_status` | VARCHAR(30) | Yes | NULL |  | Trạng thái quyết định sau khi đổi |
| `actor_type` | VARCHAR(20) | No |  | IDX | MOTO_USER, SAKI_USER, SYSTEM |
| `actor_id` | BIGINT | Yes | NULL | IDX | ID actor trong schema/tenant tương ứng |
| `actor_tenant_id` | BIGINT | No | 0 | IDX | Tenant của actor; 0 nếu SYSTEM |
| `actor_company_code` | VARCHAR(50) | Yes | NULL | IDX | Business code công ty của actor nếu có |
| `reason` | TEXT | Yes | NULL |  | Lý do thay đổi trạng thái |
| `request_id` | VARCHAR(100) | No |  | IDX | Request ID từ API Gateway/service |
| `correlation_id` | VARCHAR(100) | No |  | IDX | ID truy vết workflow cross-tenant |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_invoice_document_artifact` (Metadata file Hóa đơn PDF trên S3)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_id` | BIGINT | No |  | FK, IDX | Tham chiếu `t_invoice(id)` |
| `storage_key` | VARCHAR(255) | No |  |  | AWS S3 Object Key |
| `file_hash` | VARCHAR(255) | No |  |  | Mã băm file chống giả mạo |
| `version` | SMALLINT | No | 1 |  | Phiên bản PDF |
| `locked_flg` | SMALLINT | No | 0 |  | 1: PDF đã chốt/khóa |
| `generated_at` | TIMESTAMP | No |  |  | Thời điểm thực tế hệ thống render file PDF |
| `source_status` | VARCHAR(20) | No |  |  | Trạng thái của nguồn dữ liệu lúc xuất file (VD: DRAFT, ACTIVE) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_invoice_artifact_version: (company_id, invoice_id, version)

### `t_invoice_bank_account` (Snapshot tài khoản ngân hàng trên Hóa đơn)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_id` | BIGINT | No |  | FK, IDX | Tham chiếu Header Hóa đơn |
| `bank_name` | VARCHAR(100) | No |  |  | Tên ngân hàng (Snapshot chết) |
| `branch_name` | VARCHAR(100) | No |  |  | Tên chi nhánh |
| `account_number` | VARCHAR(50) | No |  |  | Số tài khoản |
| `account_type` | SMALLINT | Yes | NULL |  | 口座種別: 1=普通 2=当座 |
| `account_holder` | VARCHAR(100) | Yes | NULL |  | 口座名義 |
| `display_order` | SMALLINT | Yes | NULL |  | Thứ tự (≤6 dòng/hóa đơn) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_invoice_detail` (Chi tiết Hóa đơn theo Staff/Hợp đồng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_id` | BIGINT | No |  | FK, IDX | Tham chiếu Header Hóa đơn |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu Hợp đồng |
| `staff_id` | BIGINT | No |  | FK, IDX | Nhân sự thực hiện |
| `regular_amount` | DECIMAL(12,2) | No | 0 |  | Tiền công giờ hành chính |
| `overtime_amount` | DECIMAL(12,2) | No | 0 |  | Tiền công giờ làm thêm |
| `midnight_amount` | DECIMAL(12,2) | No | 0 |  | Tiền công giờ ca đêm |
| `detail_total_amount` | DECIMAL(12,2) | No | 0 |  | Tổng tiền dòng này (gộp vào nhóm thuế tương ứng trong `t_invoice_amount_summary`) |
| `detail_code` | VARCHAR(16) | No |  | IDX | 請求明細コード (unique trong 1 hóa đơn; thứ tự hiển thị) |
| `estaffing_contract_no` | VARCHAR(20) | Yes | NULL | IDX | e-staffing契約No snapshot — có thể dùng 5-digit 枝番 (legacy). KHÔNG nhầm với `contract_no` nội bộ (3-digit). Nếu import từ e-staffing cũ: lưu raw value tại đây. |
| `staff_code_snapshot` | VARCHAR(15) | Yes | NULL |  | スタッフコード snapshot |
| `staff_name_snapshot` | VARCHAR(64) | Yes | NULL |  | スタッフ名 (in trên hóa đơn) |
| `job_code` | VARCHAR(16) | Yes | NULL |  | Jobコード |
| `saki_custom_job_name_snapshot` | VARCHAR(100) | Yes | NULL |  | **[Reviewer Fix 3]** Snapshot tên công việc nội bộ SAKI tại thời điểm xuất hóa đơn — lấy từ `mst_saki_custom_job_type.custom_job_name`. Lưu cứng để hóa đơn lịch sử không mất tên hiển thị nếu SAKI xóa master sau này. KHÔNG hard FK (snapshot pattern). |
| `saki_cost_center_name_snapshot` | VARCHAR(100) | Yes | NULL |  | **[Reviewer Fix 6]** Snapshot tên Cost Center SAKI tại thời điểm xuất hóa đơn — lấy từ `mst_saki_cost_center.cost_center_name`. Đảm bảo immutability chứng từ kế toán (Sổ cái lịch sử không nhảy tên nếu SAKI đổi mst_saki_cost_center). KHÔNG hard FK (snapshot pattern). |
| `period_start` | DATE | Yes | NULL |  | 対象期間 từ |
| `period_end` | DATE | Yes | NULL |  | 対象期間 đến |
| `unit_price` | INT | Yes | NULL |  | 請求単価 |
| `unit_type` | SMALLINT | Yes | NULL |  | 請求単位: 1=時 2=日 3=月 |
| `contract_internal_minutes` | INT | Yes | NULL |  | 契約内 時間(phút) |
| `overtime_minutes` | INT | Yes | NULL |  | 時間外 時間(phút) |
| `holiday_work_minutes` | INT | Yes | NULL |  | 休出 時間(phút) |
| `holiday_work_amount` | INT | Yes | NULL |  | 休出金額 |
| `contract_external_minutes` | INT | Yes | NULL |  | 契約外小計 時間(phút) |
| `contract_external_amount` | INT | Yes | NULL |  | 契約外金額 |
| `midnight_minutes` | INT | Yes | NULL |  | 深夜 時間(phút) |
| `deduction_minutes` | INT | Yes | NULL |  | 控除 時間(phút) |
| `deduction_amount_time` | INT | Yes | NULL |  | 控除金額(時間) |
| `deduction_days` | NUMERIC(4,1) | Yes | NULL |  | 控除 日数 |
| `deduction_amount_days` | INT | Yes | NULL |  | 控除金額(日) |
| `deduction_total` | INT | Yes | NULL |  | 控除合計金額 |
| `adjustment_1` | INT | Yes | NULL |  | 調整額1 |
| `legal_holiday_amount` | INT | Yes | NULL |  | 法定休日 (調整2 con) |
| `other_adjustment` | INT | Yes | NULL |  | その他調整額 |
| `adjustment_2_total` | INT | Yes | NULL |  | 調整額2計 (=法定休日+その他) |
| `reimbursement_amount` | INT | Yes | NULL |  | 立替金額 (gắn明細) |
| `no_contract_flg` | SMALLINT | No | 0 |  | 契約無フラグ: 1=明細không gắn HĐ e-staffing |
| `transport_ex_tax` | INT | Yes | NULL |  | 交通費相当額(外税) |
| `transport_in_tax` | INT | Yes | NULL |  | 交通費相当額(内税) |
| `timecard_reimbursement_import_flg` | SMALLINT | No | 0 |  | タイムカード立替金取込フラグ |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_invoice_detail_attendance_daily` (Snapshot Chấm công per-ngày trong Hóa đơn — Immutable)

> **[R5 — Remediation 2026-06-28]** Bảng này là **snapshot bất biến** (không phải live mapping). Khi hóa đơn được phát hành (`source_status = 'ISSUED'`), dữ liệu chấm công hàng ngày được copy vào đây để bảo toàn tính pháp lý (電子帳簿保存法 12y4m). Sau khi invoice ISSUED, rows này không được UPDATE. Nếu MOTO reissue → copy lại sang `invoice_id` của version mới.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_detail_id` | BIGINT | No |  | FK, IDX | Dòng hóa đơn |
| `attendance_daily_id` | BIGINT | No |  | FK, IDX | ID Chấm công được snapshot vào dòng hóa đơn |
| `timecard_period_type` | SMALLINT | Yes | NULL |  | タイムカード期間種別: 1/2/3 (tờ thứ mấy trong tháng) |
| `closing_target_date` | DATE | Yes | NULL |  | 締め対象日 |
| `work_days` | NUMERIC(4,1) | Yes | NULL |  | 出勤日数 (cấp kỳ) |
| `absence_days` | NUMERIC(4,1) | Yes | NULL |  | 欠勤日数 |
| `paid_leave_days` | NUMERIC(4,1) | Yes | NULL |  | 年休日数 (0.5) |
| `work_date_snapshot` | DATE | Yes | NULL |  | 就業年月日 (snapshot per-ngày) |
| `category_snapshot` | SMALLINT | Yes | NULL |  | 区分 snapshot (0振替休日/1通常/2年休/3休出/4欠勤/5半休/6振替出勤) |
| `start_minutes_snapshot` | SMALLINT | Yes | NULL |  | 開始時刻 (phút) |
| `end_minutes_snapshot` | SMALLINT | Yes | NULL |  | 終了時刻 (phút, có thể >1440) |
| `break_minutes_snapshot` | SMALLINT | Yes | NULL |  | 休憩時間 |
| `midnight_break_minutes_snapshot` | SMALLINT | Yes | NULL |  | 深夜休憩 |
| `contract_internal_minutes_snapshot` | INT | Yes | NULL |  | 契約内時間 |
| `contract_external_minutes_snapshot` | INT | Yes | NULL |  | 契約外時間 |
| `workplace_type_snapshot` | SMALLINT | Yes | NULL |  | 就業場所区分 (1-8) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_invoice_detail_attendance: (company_id, invoice_detail_id, attendance_daily_id) WHERE deleted_at IS NULL
- **Application Rule:** Không dùng unique tuyệt đối trên `attendance_daily_id` vì MOTO reissue invoice bằng immutable version mới cần được phép snapshot lại cùng ngày công. Service layer phải chặn double billing bằng cách kiểm tra không tồn tại invoice active khác cho cùng `attendance_daily_id` trong các invoice có `source_status = 'ISSUED'` và `decision_status` chưa bị superseded/rejected/cancelled.

### `t_invoice_adjustment_detail` (Điều chỉnh Cộng/Trừ Đặc biệt - 特別調整額)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_detail_id` | BIGINT | No |  | FK, IDX | Thuộc dòng chi tiết nào |
| `adjustment_reason` | VARCHAR(255) | No |  |  | Lý do (VD: Chiết khấu, Thưởng) |
| `amount` | DECIMAL(12,2) | No |  |  | Số tiền (Âm hoặc Dương) |
| `tax_rate_percent` | INT | No | 10 |  | Mức thuế suất áp dụng (8 hoặc 10) |
| `breakdown_adjustment_type` | SMALLINT | No | 0 | IDX | 内訳調整額区分: 0=調整額1 1=その他 2=特別調整額1 3=特別調整額2 |
| `occurrence_date` | DATE | Yes | NULL |  | 内訳発生年月(日) |
| `tax_category` | SMALLINT | Yes | NULL |  | 消費税区分: 0=標準10% 1=軽減8% (nếu bảng đã có tax_rate_percent thì GIỮ, thêm cột này) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_invoice_reimbursement_header` (Header Thanh toán hộ/Tạm ứng - 立替金)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `invoice_detail_id` | BIGINT | No |  | FK, IDX | Thuộc dòng chi tiết nào |
| `total_reimbursement` | DECIMAL(12,2) | No | 0 |  | Tổng tiền thanh toán hộ |
| `business_transport_total` | INT | Yes | NULL |  | 業務交通費合計 |
| `with_tax_10_total` | INT | Yes | NULL |  | 消費税額入力あり 10%対象 |
| `with_tax_10_tax` | INT | Yes | NULL |  | 消費税額入力あり 10%消費税額 |
| `with_tax_8_total` | INT | Yes | NULL |  | 消費税額入力あり 8%対象 |
| `with_tax_8_tax` | INT | Yes | NULL |  | 消費税額入力あり 8%消費税額 |
| `without_tax_10_total` | INT | Yes | NULL |  | 消費税額入力なし 10%対象 |
| `without_tax_8_total` | INT | Yes | NULL |  | 消費税額入力なし 8%対象 |
| `without_tax_nontaxable_total` | INT | Yes | NULL |  | 消費税額入力なし 不課税非課税免税 |
| `nontaxable_reimbursement_total` | INT | Yes | NULL |  | 非課税立替金合計 |
| `out_of_scope_reimbursement_total` | INT | Yes | NULL |  | 課税対象外立替金合計 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_invoice_reimbursement_detail` (Chi tiết Thanh toán hộ theo hạng mục)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `reimbursement_header_id` | BIGINT | No |  | FK, IDX | Thuộc header thanh toán hộ nào |
| `expense_name_snapshot` | VARCHAR(100) | No |  |  | Snapshot tên chi phí (VD: Phí taxi) |
| `amount` | DECIMAL(12,2) | No |  |  | Số tiền |
| `tax_rate_percent` | INT | No | 10 |  | Mức thuế (0, 8, 10) |
| `reimbursement_type` | SMALLINT | No | 1 | IDX | 立替金種別: 1=業務交通費 2=その他立替金 3=非課税・課税対象外立替金 |
| `payment_date` | DATE | Yes | NULL |  | 支払日 |
| `from_location` | VARCHAR(40) | Yes | NULL |  | 発 (điểm đi) |
| `to_location` | VARCHAR(40) | Yes | NULL |  | 着 (điểm đến) |
| `transport_means` | SMALLINT | Yes | NULL |  | 交通手段: 1=鉄道 2=バス 3=船舶 |
| `fare_amount` | INT | Yes | NULL |  | 運賃金額 |
| `content` | VARCHAR(100) | Yes | NULL |  | 内容 |
| `expense_category` | VARCHAR(2) | Yes | NULL | IDX | 費用区分 (01-94: 01タクシー/02駐車場高速/.../07物品購入軽減8%/08物品購入10%/09切手/10収入印紙/14-16海外出張/92非課税/93不課税/94免税...) |
| `consumption_tax` | INT | Yes | NULL |  | 消費税額 |
| `registration_no` | VARCHAR(13) | Yes | NULL |  | 適格請求書発行事業者登録番号 (13 số, không tiền tố T) |
| `issuer_name` | VARCHAR(100) | Yes | NULL |  | 適格請求書発行事業者事業者名 |
| `nontaxable_type` | SMALLINT | Yes | NULL |  | 非課税種別: 1=非課税立替金 2=課税対象外立替金 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_invoice_upload_history` (Lịch sử MOTO Upload CSV Hóa đơn)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | User MOTO thực hiện upload |
| `file_name` | VARCHAR(255) | No |  |  | Tên file |
| `status` | VARCHAR(20) | No | 'PROCESSING' |  | PROCESSING, COMPLETED, FAILED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 11: Quản lý Lịch làm việc (Shift Schedule)

### `t_work_schedule` (Lịch làm việc dự kiến - 勤務予定, KHÁC シフト表)

> Lịch công dự kiến đưa vào chấm công; có 区分 + vòng đời 申請/承認; phạm vi 3 tháng. Khác `t_moto_shift_schedule` (シフト表 = lịch ca, gắn hợp đồng).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu `t_contract(contract_no)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `work_date` | DATE | No |  | IDX | Ngày làm việc dự kiến |
| `category` | SMALLINT | No |  |  | 区分: 1=通常 2=年休 3=休出 4=欠勤 |
| `start_minutes` | SMALLINT | Yes | NULL |  | Giờ vào ca dự kiến (phút từ 00:00) |
| `end_minutes` | SMALLINT | Yes | NULL |  | Giờ ra ca dự kiến (phút) |
| `break_minutes` | SMALLINT | Yes | NULL |  | 休憩時間 (phút) |
| `midnight_break_minutes` | SMALLINT | Yes | NULL |  | 深夜休憩 (phút) |
| `status` | VARCHAR(20) | No |  | IDX | 申請中/承認済/承認待ち/代理申請/再申請/代理再申請/申請取消/却下 |
| `cancel_request_flg` | SMALLINT | No | 0 |  | Cờ yêu cầu hủy: 0/1 |
| `closing_unit_ref` | VARCHAR(20) | Yes | NULL |  | 締め単位 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
- **UNIQUE (Partial)** uq_work_schedule: (contract_no, staff_id, work_date) WHERE deleted_at IS NULL
- **RLS policy** — p_work_schedule_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_attendance_reimbursement` (Tiền tạm ứng theo ngày - 立替金, header)

> Entry 立替金 theo ngày của staff; gom vào hóa đơn M05. Khác `t_invoice_reimbursement_*`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_staff(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Tham chiếu `t_contract(contract_no)` |
| `work_date` | DATE | No |  | IDX | Ngày phát sinh chi phí |
| `attendance_daily_id` | BIGINT | Yes | NULL | IDX | Logical ref tới `t_attendance_daily(id)` |
| `total_amount` | INT | Yes | NULL |  | Tổng tiền |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **RLS policy** — p_att_reimbursement_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_attendance_reimbursement_detail` (Chi tiết tiền tạm ứng theo ngày - 1:N)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `reimbursement_id` | BIGINT | No |  | FK, IDX | Trỏ tới `t_attendance_reimbursement(id)` |
| `expense_type` | SMALLINT | No |  |  | 1=業務交通費 2=その他立替金 |
| `from_location` | VARCHAR(100) | Yes | NULL |  | 出発地 |
| `to_location` | VARCHAR(100) | Yes | NULL |  | 到着地 |
| `transport_means` | VARCHAR(50) | Yes | NULL |  | 交通手段 |
| `content` | VARCHAR(255) | Yes | NULL |  | Nội dung (その他) |
| `amount` | INT | No |  |  | Số tiền |
| `tax_category` | SMALLINT | Yes | NULL |  | 課税区分: 1=課税 2=非課税 3=課税対象外 |
| `tax_rate_percent` | SMALLINT | Yes | NULL |  | 0/8/10 |
| `attachment_file_key` | VARCHAR(255) | Yes | NULL |  | File đính kèm S3 key |
| `invoice_registration_no` | VARCHAR(50) | Yes | NULL |  | 適格請求書発行事業者登録番号 (インボイス) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **RLS policy** — p_att_reimbursement_detail_moto: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)

### `t_moto_shift_schedule` (Lịch làm ca của Staff - シフト表)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `saki_tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Dùng để SAKI có thể xem lịch của Staff |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Hợp đồng liên quan |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu Staff |
| `shift_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu Master Ca làm việc |
| `work_date` | DATE | No |  | IDX | Ngày làm việc |
| `start_time` | SMALLINT | Yes | NULL |  | Giờ vào ca dự kiến (Phút) |
| `end_time` | SMALLINT | Yes | NULL |  | Giờ ra ca dự kiến (Phút) |
| `break_time_minutes` | SMALLINT | Yes | NULL |  | 休憩時間 (phút) |
| `midnight_break_minutes` | SMALLINT | Yes | NULL |  | 深夜休憩 (phút) |
| `shift_master_ref_flg` | SMALLINT | No | 0 |  | シフトマスタ引用フラグ: 0=nhập trực tiếp 1=tham chiếu master |
| `modified_flg` | SMALLINT | No | 0 |  | 変更あり(1)/なし(0) so với master |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
- **INDEX** idx_shift_saki_tenant: (saki_tenant_id)
- **RLS policy** — p_shift_schedule_saki: USING (saki_tenant_id = current_setting('app.current_saki_tenant_id', true)) WITH CHECK (saki_tenant_id = current_setting('app.current_saki_tenant_id', true))

---

## Nhóm 12: Hệ thống, Security, Audit & Async Event Queue

### `t_document_download_audit` (Bảo mật: Audit User tải tài liệu nhạy cảm)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `document_type` | VARCHAR(50) | No |  |  | INVOICE, CONTRACT |
| `document_artifact_id` | BIGINT | No |  | **Logical FK** | Trỏ tới bảng Artifact file PDF |
| `document_storage_key` | VARCHAR(255) | Yes | NULL |  | Lưu cứng S3 Key phòng khi Artifact bị Archive |
| `artifact_version` | SMALLINT | Yes | NULL |  | Version của tài liệu tại thời điểm tải |
| `actor_type` | VARCHAR(20) | No |  |  | MOTO_USER, SAKI_USER |
| `actor_tenant_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | ID Tenant của người tải (Bắt buộc nếu actor_type = SAKI_USER để trace SIEM) |
| `actor_company_code` | VARCHAR(50) | Yes | NULL | **Logical FK**, IDX | Mã Business Code của Company người tải |
| `actor_id` | BIGINT | No |  | IDX | ID người tải |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID của HTTP Request (từ API Gateway) |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết Cross-tenant (Map với SAKI request) |
| `user_agent` | VARCHAR(500) | Yes | NULL |  | Trình duyệt/Thiết bị |
| `ip_address` | VARCHAR(50) | Yes | NULL |  | Địa chỉ IP |
| `downloaded_at` | TIMESTAMP | No | NOW() |  | Giờ tải |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID user tạo |
| `updated_by` | BIGINT | Yes | NULL |  | ID user cập nhật |
- **CHECK** chk_document_download_cross_actor: actor_type <> 'SAKI_USER' OR (actor_tenant_id IS NOT NULL AND actor_company_code IS NOT NULL)

### `t_staff_notification_read` (Trạng thái Staff đọc Thông báo từ Staff Web)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Tham chiếu Staff |
| `notification_id` | BIGINT | No |  | **Logical FK**, IDX | ID thông báo chung từ Platform |
| `read_at` | TIMESTAMP | No | NOW() |  | Thời điểm đọc |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_moto_notification_read` (Trạng thái User MOTO đọc thông báo)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu User MOTO |
| `notification_id` | BIGINT | No |  | **Logical FK**, IDX | ID thông báo chung từ Platform |
| `read_at` | TIMESTAMP | No | NOW() |  | Thời điểm đọc |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_moto_batch_execution_history` (Nhật ký chạy Job Worker của MOTO)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `job_name` | VARCHAR(100) | No |  |  | Tên Job (VD: AGGREGATE_MONTHLY_ATTENDANCE) |
| `status` | VARCHAR(20) | No | 'RUNNING' |  | RUNNING, SUCCESS, FAILED |
| `error_log` | TEXT | Yes | NULL |  | Chi tiết Exception nếu lỗi |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_moto_audit_log` (Log thao tác hệ thống MOTO - Yêu cầu lưu trữ 6 tháng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_company(id) |
| `actor_type` | VARCHAR(20) | No | 'MOTO_USER' |  | SYSTEM, MOTO_USER, SAKI_USER, STAFF |
| `actor_tenant_id` | BIGINT | Yes | NULL | IDX | ID Tenant của người thao tác (Phục vụ SIEM cross-tenant) |
| `actor_company_code` | VARCHAR(50) | Yes | NULL | IDX | Mã Business Code của công ty người thao tác |
| `actor_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu User thao tác (Logical FK nếu là SAKI/Staff) |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID của HTTP Request (từ API Gateway) |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết Cross-tenant Event / Workflow |
| `action` | VARCHAR(100) | No |  | IDX | API Endpoint / Tên hành động |
| `entity_table` | VARCHAR(50) | No |  | IDX | Bảng bị tác động |
| `entity_id` | VARCHAR(100) | No |  | IDX | ID bị tác động |
| `before_data` | JSONB | Yes | NULL |  | Dữ liệu cũ (đã mask password) |
| `after_data` | JSONB | Yes | NULL |  | Dữ liệu mới |
| `ip_address` | VARCHAR(50) | Yes | NULL |  | IP Address của người dùng |
| `user_agent` | VARCHAR(500) | Yes | NULL |  | Trình duyệt/Thiết bị |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
- **CHECK** chk_moto_audit_cross_actor: actor_type NOT IN ('SAKI_USER', 'STAFF') OR (actor_tenant_id IS NOT NULL AND actor_company_code IS NOT NULL)

### `t_moto_inbox_event` (Hàng đợi nhận Event từ SAKI/Platform)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `source_tenant_id` | BIGINT | No | 0 | IDX | Nguồn phát Event (0 = SYSTEM/PLATFORM) |
| `source_tenant_type` | VARCHAR(20) | No |  |  | PLATFORM, SAKI, SYSTEM |
| `aggregate_type` | VARCHAR(50) | No |  | IDX | Loại thực thể (VD: INQUIRY) |
| `aggregate_id` | VARCHAR(100) | No |  | IDX | ID thực thể |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `event_type` | VARCHAR(100) | No |  | IDX | VD: SAKI_APPROVED_CONTRACT |
| `payload` | JSONB | No |  |  | Data từ SAKI gửi sang |
| `idempotency_key` | VARCHAR(100) | No |  | Composite UK | ID do nguồn sinh ra |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết luồng Log (Khớp với hệ thống) |
| `retry_count` | SMALLINT | No | 0 |  | Số lần Job thực hiện thử lại |
| `last_error` | TEXT | Yes | NULL |  | Chi tiết Exception lỗi cuối cùng |
| `processed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm xử lý xong |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm lên lịch thử lại tiếp theo |
| `status` | VARCHAR(20) | No | 'PENDING' | IDX | PENDING, PROCESSING, COMPLETED, FAILED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_moto_inbox_idem: (company_id, source_tenant_type, source_tenant_id, idempotency_key)
- **INDEX (composite)** idx_moto_inbox_aggregate: (aggregate_type, aggregate_id, created_at)
- **INDEX (composite)** idx_moto_inbox_worker: (status, next_retry_at)

### `t_moto_outbox_event` (Hàng đợi phát Event đi SAKI/Platform)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `target_tenant_id` | BIGINT | No | 0 | IDX | Đích đến Event (0 = PLATFORM/BROADCAST) |
| `target_tenant_type` | VARCHAR(20) | No |  |  | PLATFORM, SAKI, BROADCAST |
| `aggregate_type` | VARCHAR(50) | No |  | IDX | Loại thực thể |
| `aggregate_id` | VARCHAR(100) | No |  | IDX | ID thực thể |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `event_type` | VARCHAR(100) | No |  | IDX | VD: MOTO_SUBMITTED_ESTIMATE |
| `payload` | JSONB | No |  |  | Data MOTO gửi đi |
| `idempotency_key` | VARCHAR(100) | No |  | Composite UK | ID do MOTO sinh ra |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết luồng Log (Khớp với hệ thống) |
| `retry_count` | SMALLINT | No | 0 |  | Số lần Job thực hiện thử lại |
| `last_error` | TEXT | Yes | NULL |  | Chi tiết Exception lỗi cuối cùng |
| `processed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm xử lý xong |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm lên lịch thử lại tiếp theo |
| `status` | VARCHAR(20) | No | 'PENDING' | IDX | PENDING, SENDING, SENT, FAILED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_moto_outbox_idem: (company_id, target_tenant_type, target_tenant_id, idempotency_key)
- **INDEX (composite)** idx_moto_outbox_aggregate: (aggregate_type, aggregate_id, created_at)
- **INDEX (composite)** idx_moto_outbox_worker: (status, next_retry_at)

### `t_moto_upload_file_result` (Lưu chi tiết lỗi File Upload CSV)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `upload_history_id` | BIGINT | No |  | **Logical FK** | Trỏ tới bảng Upload Hợp đồng/Hóa đơn |
| `error_summary` | TEXT | Yes | NULL |  | Tổng hợp lỗi |
| `result_file_key` | VARCHAR(255) | Yes | NULL |  | S3 Key chứa file CSV báo chi tiết lỗi từng dòng |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_moto_notification_log` (Sổ Cái Thông báo MOTO — 通知監査ログ, 3 năm retention) [M08 NEW]

> **Audit master record** cho mọi event-driven notification gửi tới User MOTO **và Staff** (cùng tenant MOTO). Pure-immutable insert-only. Lưu snapshot subject/body rendered. Cùng schema MOTO vì Staff thuộc quyền quản lý MOTO (data ownership). Phân biệt recipient qua bảng inbox (MOTO user vs Staff). Retention 3 năm cho compliance/dispute resolution.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `notification_type_code` | VARCHAR(20) | No |  | IDX | Logical FK → `platform.mst_notification_template(notification_type_code)` |
| `category` | SMALLINT | No |  | IDX | 1=CONTRACT/2=ATTENDANCE/3=INVOICE/4=SYSTEM/5=ANNOUNCEMENT |
| `priority` | SMALLINT | No | 1 | IDX | 1=NORMAL/2=IMPORTANT/3=URGENT |
| `audience_scope` | SMALLINT | No |  | IDX | 1=MOTO_USER_ONLY / 2=STAFF_ONLY / 3=BOTH — phân biệt fan-out đích |
| `subject_snapshot` | VARCHAR(500) | No |  |  | Subject rendered (Mustache → text) — D5 snapshot |
| `body_snapshot` | TEXT | No |  |  | Body rendered — 電子帳簿保存法 + BR-109 |
| `template_variables_payload` | JSONB | Yes | NULL |  | Variables snapshot dùng render: {contract_no, staff_name, ...} — display-only, ≥5 fields |
| `action_url` | VARCHAR(500) | Yes | NULL |  | Deep-link URL rendered |
| `sender_type` | SMALLINT | No |  |  | 1=SYSTEM (batch/event)/2=PLATFORM_ADMIN/3=USER (manual) |
| `sender_user_id` | BIGINT | Yes | NULL |  | NULL nếu sender_type=SYSTEM |
| `related_entity_type` | VARCHAR(50) | Yes | NULL | IDX | "CONTRACT"/"ATTENDANCE"/"INVOICE"/"STAFF"... |
| `related_entity_id` | BIGINT | Yes | NULL | IDX | ID entity gốc |
| `is_legal_alert` | SMALLINT | No | 0 | IDX | Đóng băng tại moment (copy từ template) |
| `total_recipient_count` | INT | No | 0 |  | Số recipient fan-out |
| `triggered_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm event |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (chỉ Platform Admin sau 3 năm) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_moto_notif_log_type: (notification_type_code, triggered_at DESC)
- **INDEX** idx_moto_notif_log_entity: (related_entity_type, related_entity_id)
- **INDEX** idx_moto_notif_log_legal: (is_legal_alert, triggered_at DESC) WHERE is_legal_alert = 1
- **INDEX** idx_moto_notif_log_audience: (audience_scope, triggered_at DESC)
- **CHECK** chk_moto_notif_log_category: category IN (1, 2, 3, 4, 5)
- **CHECK** chk_moto_notif_log_priority: priority IN (1, 2, 3)
- **CHECK** chk_moto_notif_log_audience: audience_scope IN (1, 2, 3)
- **CHECK** chk_moto_notif_log_sender_type: sender_type IN (1, 2, 3)
- **CHECK** chk_moto_notif_log_legal: is_legal_alert IN (0, 1)
- **CHECK** chk_moto_notif_log_recipient_count: total_recipient_count >= 0
- **CHECK** chk_moto_notif_log_type_format: notification_type_code ~ '^NTF-[A-Z]{1}[0-9]{2}$'
- **CHECK** chk_moto_notif_log_sender_user_consistency: sender_type = 1 OR sender_user_id IS NOT NULL
- **Trigger** trg_moto_notif_log_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_moto_inbox_notification` (Hộp thư MOTO — Per-User Inbox, 90 ngày) [M08 NEW]

> Fan-out per **MOTO User** — 1 row mỗi recipient. Tối ưu query unread badge: `WHERE user_id=X AND is_read=0`. Retention 90 ngày, batch hard-delete (BR-105). Denormalize subject_cache/category/priority cho list view không JOIN.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Recipient MOTO user. RLS scope |
| `notification_log_id` | BIGINT | No |  | FK, IDX | Trỏ về `t_moto_notification_log(id)` |
| `notification_type_code` | VARCHAR(20) | No |  |  | Cached cho list filter |
| `category` | SMALLINT | No |  | IDX | Cached |
| `priority` | SMALLINT | No | 1 | IDX | Cached |
| `subject_cache` | VARCHAR(500) | No |  |  | Cached cho list rendering |
| `action_url` | VARCHAR(500) | Yes | NULL |  | Cached deep-link |
| `is_read` | SMALLINT | No | 0 | IDX | 0=Unread / 1=Read. Hot column |
| `read_at` | TIMESTAMP | Yes | NULL |  | Thời điểm read |
| `is_starred` | SMALLINT | No | 0 |  | 1=User pin |
| `delivered_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm fan-out — dùng cleanup 90d |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (user dismiss) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_moto_inbox_user_unread: (user_id, is_read, delivered_at DESC) WHERE is_read = 0 AND deleted_at IS NULL — badge count
- **INDEX** idx_moto_inbox_user_list: (user_id, delivered_at DESC) WHERE deleted_at IS NULL — list view
- **INDEX** idx_moto_inbox_log: (notification_log_id)
- **INDEX** idx_moto_inbox_cleanup: (delivered_at) WHERE deleted_at IS NULL
- **UNIQUE** uq_moto_inbox_log_user: (notification_log_id, user_id)
- **FK** fk_moto_inbox_log: (notification_log_id) → t_moto_notification_log(id)
- **CHECK** chk_moto_inbox_is_read: is_read IN (0, 1)
- **CHECK** chk_moto_inbox_is_starred: is_starred IN (0, 1)
- **CHECK** chk_moto_inbox_category: category IN (1, 2, 3, 4, 5)
- **CHECK** chk_moto_inbox_priority: priority IN (1, 2, 3)
- **CHECK** chk_moto_inbox_read_consistency: (is_read = 0 AND read_at IS NULL) OR (is_read = 1 AND read_at IS NOT NULL)
- **Trigger** trg_moto_inbox_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_staff_inbox_notification` (Hộp thư Staff Web — Per-Staff Inbox, 90 ngày) [M08 NEW]

> Fan-out per **Staff** (派遣スタッフ). Staff KHÔNG đăng nhập System User, dùng Staff Web/WebTimeCard độc lập. RLS scope theo `staff_id` (không phải user_id) — Staff chỉ thấy noti của chính mình (SELF_ONLY isolation). Cùng schema MOTO vì Staff thuộc data ownership của tenant MOTO. Cùng `notification_log_id` reference với MOTO user inbox khi audience_scope=BOTH.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Recipient Staff. RLS scope (staff_id-aware, KHÔNG dùng user_id) |
| `notification_log_id` | BIGINT | No |  | FK, IDX | Trỏ về `t_moto_notification_log(id)` |
| `notification_type_code` | VARCHAR(20) | No |  |  | Cached |
| `category` | SMALLINT | No |  | IDX | Cached |
| `priority` | SMALLINT | No | 1 | IDX | Cached |
| `subject_cache` | VARCHAR(500) | No |  |  | Cached cho Staff App list rendering |
| `action_url` | VARCHAR(500) | Yes | NULL |  | Cached deep-link (Staff Web URL) |
| `is_read` | SMALLINT | No | 0 | IDX | 0=Unread / 1=Read |
| `read_at` | TIMESTAMP | Yes | NULL |  |  |
| `delivered_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm fan-out |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_staff_inbox_unread: (staff_id, is_read, delivered_at DESC) WHERE is_read = 0 AND deleted_at IS NULL
- **INDEX** idx_staff_inbox_list: (staff_id, delivered_at DESC) WHERE deleted_at IS NULL
- **INDEX** idx_staff_inbox_log: (notification_log_id)
- **INDEX** idx_staff_inbox_cleanup: (delivered_at) WHERE deleted_at IS NULL
- **UNIQUE** uq_staff_inbox_log_staff: (notification_log_id, staff_id)
- **FK** fk_staff_inbox_log: (notification_log_id) → t_moto_notification_log(id)
- **FK** fk_staff_inbox_staff: (staff_id) → mst_staff(id)
- **CHECK** chk_staff_inbox_is_read: is_read IN (0, 1)
- **CHECK** chk_staff_inbox_category: category IN (1, 2, 3, 4, 5)
- **CHECK** chk_staff_inbox_priority: priority IN (1, 2, 3)
- **CHECK** chk_staff_inbox_read_consistency: (is_read = 0 AND read_at IS NULL) OR (is_read = 1 AND read_at IS NOT NULL)
- **Trigger** trg_staff_inbox_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_moto_email_send_history` (Lịch sử Gửi Email MOTO — Universal SES Delivery, 1 năm retention BR-109) [M08 NEW]

> Universal email send history cho MOTO tenant — thay thế việc mở rộng `t_attendance_email_history` (đã DEPRECATED). Cover mọi loại email: alarm, contract approval, invoice, treatment notification, attendance reminder, staff onboarding. Tracking SES delivery (sent/failed/bounced/complained/pending) + retry 3 lần (BR-110). Snapshot subject/body — D5 + 電子帳簿保存法.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `notification_log_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ `t_moto_notification_log(id)` nếu email là output event-driven noti. NULL với system email không gắn noti (vd: reset password Staff) |
| `notification_type_code` | VARCHAR(20) | No |  | IDX | NTF-* hoặc SYS-* cho hệ thống |
| `recipient_type` | SMALLINT | No |  |  | 1=MOTO_USER / 2=STAFF / 3=EXTERNAL (vd: SAKI contact) — phân biệt FK đích |
| `recipient_user_id` | BIGINT | Yes | NULL | FK, IDX | NULL nếu recipient_type ≠ 1 |
| `recipient_staff_id` | BIGINT | Yes | NULL | FK, IDX | NULL nếu recipient_type ≠ 2 |
| `recipient_email` | VARCHAR(255) | No |  | IDX | Snapshot email tại moment gửi |
| `subject_snapshot` | VARCHAR(500) | No |  |  | Subject snapshot — BR-109 |
| `body_snapshot` | TEXT | No |  |  | Body snapshot — BR-109 + 電子帳簿保存法 |
| `status` | SMALLINT | No | 1 | IDX | 1=PENDING/2=SENDING/3=SENT/4=FAILED/5=BOUNCED/6=COMPLAINED |
| `ses_message_id` | VARCHAR(100) | Yes | NULL | IDX | SES Message ID — webhook lookup |
| `retry_count` | INT | No | 0 |  | Số lần retry (max 3 — BR-110) |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Batch worker scan |
| `last_attempted_at` | TIMESTAMP | Yes | NULL |  | Lần gửi cuối |
| `sent_at` | TIMESTAMP | Yes | NULL |  | SES accept |
| `error_message` | TEXT | Yes | NULL |  | Chi tiết lỗi SES |
| `related_entity_type` | VARCHAR(50) | Yes | NULL | IDX | "CONTRACT"/"INVOICE"/"ATTENDANCE"... |
| `related_entity_id` | BIGINT | Yes | NULL | IDX |  |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_moto_email_status_retry: (status, next_retry_at) WHERE status = 4 AND deleted_at IS NULL — batch retry worker
- **INDEX** idx_moto_email_ses_msg: (ses_message_id) WHERE ses_message_id IS NOT NULL — webhook lookup
- **INDEX** idx_moto_email_recipient_email: (recipient_email, created_at DESC)
- **INDEX** idx_moto_email_log: (notification_log_id) WHERE notification_log_id IS NOT NULL
- **INDEX** idx_moto_email_entity: (related_entity_type, related_entity_id)
- **FK** fk_moto_email_log: (notification_log_id) → t_moto_notification_log(id)
- **CHECK** chk_moto_email_recipient_type: recipient_type IN (1, 2, 3)
- **CHECK** chk_moto_email_recipient_consistency: (recipient_type = 1 AND recipient_user_id IS NOT NULL AND recipient_staff_id IS NULL) OR (recipient_type = 2 AND recipient_staff_id IS NOT NULL AND recipient_user_id IS NULL) OR (recipient_type = 3 AND recipient_user_id IS NULL AND recipient_staff_id IS NULL)
- **CHECK** chk_moto_email_status: status IN (1, 2, 3, 4, 5, 6)
- **CHECK** chk_moto_email_retry_count: retry_count >= 0 AND retry_count <= 3
- **CHECK** chk_moto_email_sent_consistency: status <> 3 OR sent_at IS NOT NULL
- **CHECK** chk_moto_email_failed_consistency: status NOT IN (4, 5, 6) OR error_message IS NOT NULL
- **Trigger** trg_moto_email_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_36_agreement_document` (Tài liệu 36協定 PDF — Immutable Versioning) [M10 NEW]

> CMP-001 upload 36協定 PDF (max 10MB). Mỗi `cfg_moto_36_agreement` có thể có **nhiều version document** qua các năm gia hạn → tách bảng riêng (1:N). Khi finalize → tạo SHA-256 hash + gửi TSA qua `t_evidence_timestamp` (Platform schema). BR-1554 SHA-256+, BR-1548 timestamp trong 7 ngày, BR-1551 không xóa trước hạn.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `agreement_id` | BIGINT | No |  | FK, IDX | Trỏ về `cfg_moto_36_agreement(id)` |
| `document_version` | INTEGER | No | 1 |  | Phiên bản tăng dần per agreement_id (1=ban đầu, 2=gia hạn năm 2, ...) |
| `file_name` | VARCHAR(255) | No |  |  | Tên file gốc upload |
| `s3_storage_key` | VARCHAR(500) | No |  |  | S3 key chứa PDF — bắt buộc immutable |
| `file_size_bytes` | BIGINT | No |  |  | Kích thước file (max 10MB validation ở app layer) |
| `file_hash_sha256` | VARCHAR(64) | No |  | IDX | SHA-256 hash hex 64 ký tự — BR-1554 |
| `mime_type` | VARCHAR(100) | No | 'application/pdf' |  | Loại file (PDF only theo BR) |
| `evidence_timestamp_id` | BIGINT | Yes | NULL |  | Logical FK → `platform.t_evidence_timestamp(id)` cross-schema cho TSA signing — BR-1548 |
| `is_finalized` | SMALLINT | No | 0 | IDX | 1=Finalized (đã TSA sign — không sửa/xóa); 0=Draft. BR-1551 |
| `finalized_at` | TIMESTAMP | Yes | NULL |  | Thời điểm finalize (đã có evidence_timestamp_id) |
| `uploaded_by_moto_user_id` | BIGINT | No |  | FK | MOTO user upload |
| `status` | SMALLINT | No | 1 | IDX | 1=DRAFT 下書き / 2=ACTIVE 有効 / 3=EXPIRING_SOON 期限切れ間近 / 4=EXPIRED 期限切れ / 5=SUPERSEDED (đã có version mới hơn) |
| `superseded_by_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ về document version mới hơn (cùng agreement_id) khi gia hạn |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (TUYỆT ĐỐI KHÔNG dùng nếu is_finalized=1) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `draft_expires_at` | TIMESTAMP | Yes | NULL | IDX | Hạn lưu nháp (D18.10) = thời điểm save + TTL; NULL khi không ở trạng thái draft. Set lúc save-draft, clear (NULL) khi submit. |
| `expiration_status` | VARCHAR(10) | Yes | NULL |  | normal / warning / expired (D18.10); NULL khi không phải draft; batch hằng ngày maintain tier. |
- **PK**: (id)
- **UNIQUE** uq_36_doc_version: (agreement_id, document_version)
- **INDEX** idx_36_doc_agreement_status: (agreement_id, status, document_version DESC)
- **INDEX** idx_36_doc_finalized: (is_finalized) WHERE is_finalized = 1
- **FK** fk_36_doc_agreement: (agreement_id) → cfg_moto_36_agreement(id)
- **FK** fk_36_doc_user: (uploaded_by_moto_user_id) → mst_moto_user(id)
- **FK** fk_36_doc_superseded: (superseded_by_id) → t_36_agreement_document(id)
- **CHECK** chk_36_doc_version: document_version >= 1
- **CHECK** chk_36_doc_hash_format: file_hash_sha256 ~ '^[A-Fa-f0-9]{64}$'
- **CHECK** chk_36_doc_status: status IN (1, 2, 3, 4, 5)
- **CHECK** chk_36_doc_finalized_consistency: (is_finalized = 0 AND finalized_at IS NULL) OR (is_finalized = 1 AND finalized_at IS NOT NULL AND evidence_timestamp_id IS NOT NULL)
- **CHECK** chk_36_doc_size: file_size_bytes > 0 AND file_size_bytes <= 10485760 (max 10MB BR)
- **Trigger** trg_36_doc_updated_at: BEFORE UPDATE → fn_set_updated_at()
- **INDEX (Partial)** idx_36_agreement_document_draft_cleanup: (status, draft_expires_at) WHERE deleted_at IS NULL

### `t_36_agreement_filing` (Lịch sử Nộp 36協定 lên 労働基準監督署 — Government Filing) [M10 NEW]

> Tracking 届出 government submission cho mỗi version 36協定. SAKI/MOTO phải nộp 36協定 cho Sở Lao động khi tạo mới hoặc gia hạn. Đáp ứng compliance audit BR-1502 (giới hạn pháp lý), BR-1506 (hết hạn = chặn OT). Status PENDING → SUBMITTED → ACKNOWLEDGED (có biên nhận) hoặc REJECTED (cần resubmit).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `agreement_id` | BIGINT | No |  | FK, IDX | Trỏ về `cfg_moto_36_agreement(id)` |
| `document_id` | BIGINT | No |  | FK, IDX | Trỏ về `t_36_agreement_document(id)` — version PDF được nộp |
| `submission_date` | DATE | Yes | NULL | IDX | Ngày nộp |
| `labor_office_name` | VARCHAR(200) | No |  |  | Tên 労働基準監督署 nhận (vd: "東京中央労働基準監督署") |
| `labor_office_jurisdiction_code` | VARCHAR(20) | Yes | NULL |  | Mã thẩm quyền 監督署 (nếu có) |
| `receipt_number` | VARCHAR(100) | Yes | NULL | IDX | Số biên nhận sau khi nộp thành công |
| `receipt_date` | DATE | Yes | NULL |  | Ngày 監督署 cấp biên nhận |
| `submission_status` | SMALLINT | No | 1 | IDX | 1=PENDING (đang chuẩn bị) / 2=SUBMITTED (đã nộp, chờ acknowledge) / 3=ACKNOWLEDGED (đã có receipt) / 4=REJECTED (bị từ chối, cần resubmit) / 5=WITHDRAWN (rút yêu cầu) |
| `rejection_reason` | TEXT | Yes | NULL |  | Lý do từ chối (bắt buộc khi status=REJECTED) |
| `resubmitted_filing_id` | BIGINT | Yes | NULL | FK | Trỏ về filing mới (sau khi resubmit) |
| `submitted_by_moto_user_id` | BIGINT | Yes | NULL | FK | MOTO user thực hiện nộp |
| `submission_method` | SMALLINT | No |  |  | 1=ONLINE (e-Gov) / 2=PAPER (nộp giấy) / 3=POSTAL (gửi bưu điện) |
| `notes` | TEXT | Yes | NULL |  | Ghi chú vận hành |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_36_filing_agreement: (agreement_id, submission_date DESC)
- **INDEX** idx_36_filing_status: (submission_status, submission_date DESC)
- **FK** fk_36_filing_agreement: (agreement_id) → cfg_moto_36_agreement(id)
- **FK** fk_36_filing_document: (document_id) → t_36_agreement_document(id)
- **FK** fk_36_filing_user: (submitted_by_moto_user_id) → mst_moto_user(id)
- **FK** fk_36_filing_resubmitted: (resubmitted_filing_id) → t_36_agreement_filing(id) self-reference
- **CHECK** chk_36_filing_status: submission_status IN (1, 2, 3, 4, 5)
- **CHECK** chk_36_filing_method: submission_method IN (1, 2, 3)
- **CHECK** chk_36_filing_acknowledged_consistency: submission_status <> 3 OR (receipt_number IS NOT NULL AND receipt_date IS NOT NULL)
- **CHECK** chk_36_filing_rejection_reason: submission_status <> 4 OR (rejection_reason IS NOT NULL AND trim(rejection_reason) <> '')
- **Trigger** trg_36_filing_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_36_alarm_threshold_state` (Lịch sử Chuyển Tầng Cảnh báo 36協定 — Tier Transition Log) [M10 NEW]

> BR-1513: 注意 (80%) gửi 1 lần/kỳ; 危険 (90%) gửi hàng ngày. Cần biết "đã từng gửi tier nào cho staff X tháng Y chưa" để tránh spam notification. Bảng này log mỗi lần threshold transition lên 1 tier mới. Phân biệt với `t_36_agreement_monthly.warning_level` (snapshot mức hiện tại) — bảng này là **history** của các transitions.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `staff_id` | BIGINT | No |  | FK, IDX | Staff cụ thể |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Hợp đồng liên quan |
| `period_type` | SMALLINT | No |  | IDX | 1=MONTHLY / 2=YEARLY — phân biệt monthly threshold vs yearly threshold |
| `target_period` | VARCHAR(7) | No |  | IDX | YYYY-MM cho monthly, YYYY cho yearly |
| `tier_reached` | SMALLINT | No |  | IDX | 1=50% / 2=60% / 3=70% / 4=80% CAUTION / 5=90% DANGER / 6=100%+ VIOLATION |
| `actual_hours_at_transition` | DECIMAL(6,2) | No |  |  | Giờ OT tại thời điểm transition lên tier này |
| `limit_hours_at_transition` | INT | No |  |  | Limit tại moment (snapshot từ agreement) |
| `percentage_at_transition` | DECIMAL(5,2) | No |  |  | % tại moment (50.00, 60.00, ...) |
| `first_reached_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm đầu tiên đạt tier này |
| `last_notification_sent_at` | TIMESTAMP | Yes | NULL |  | Lần cuối gửi notification cho tier này |
| `notification_count` | INTEGER | No | 0 |  | Số lần đã gửi notification (cho tracking BR-1513) |
| `is_active` | SMALLINT | No | 1 | IDX | 1=Tier đang active (chưa lên tier cao hơn hoặc xuống thấp hơn); 0=Đã transition sang tier khác |
| `transitioned_out_at` | TIMESTAMP | Yes | NULL |  | Thời điểm chuyển tier (lên cao hoặc xuống thấp). NULL nếu vẫn active |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_36_threshold_state: (staff_id, contract_no, period_type, target_period, tier_reached) — chống insert duplicate cùng tier per period
- **INDEX** idx_36_threshold_active: (staff_id, period_type, target_period, is_active) WHERE is_active = 1
- **INDEX** idx_36_threshold_pending_notify: (tier_reached, last_notification_sent_at) WHERE is_active = 1 — batch scan gửi notification daily cho tier 5/6
- **FK** fk_36_threshold_staff: (staff_id) → mst_staff(id)
- **CHECK** chk_36_threshold_period_type: period_type IN (1, 2)
- **CHECK** chk_36_threshold_tier: tier_reached IN (1, 2, 3, 4, 5, 6)
- **CHECK** chk_36_threshold_percentage: percentage_at_transition >= 0 AND percentage_at_transition <= 999.99
- **CHECK** chk_36_threshold_is_active: is_active IN (0, 1)
- **CHECK** chk_36_threshold_notification_count: notification_count >= 0
- **CHECK** chk_36_threshold_active_consistency: (is_active = 1 AND transitioned_out_at IS NULL) OR (is_active = 0 AND transitioned_out_at IS NOT NULL)
- **CHECK** chk_36_threshold_target_period_format: (period_type = 1 AND target_period ~ '^[0-9]{4}-(0[1-9]|1[0-2])$') OR (period_type = 2 AND target_period ~ '^[0-9]{4}$')
- **Trigger** trg_36_threshold_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_moto_ledger_manual_data` (Dữ liệu Nhập Tay cho 派遣元管理台帳 — MOTO Manual Ledger Fields) [M10 NEW]

> 派遣元管理台帳 (Article 37 派遣法) — 12 trường bắt buộc, trong đó các trường "nhập tay" KHÔNG có trong tables khác (training records, career consulting, complaint handling, capability evaluation, health check) được lưu ở đây. Khi export 台帳 (CMP-012) → app layer JOIN với contract/staff/attendance/treatment để compile bản đầy đủ. Retention 3 năm theo BR-1542 (派遣法 Article 37); bản PDF export tĩnh trên S3 giữ 12年4ヶ月 theo 電子帳簿保存法 (Q1 user decision).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_moto_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | FK, IDX | Hợp đồng liên quan |
| `staff_id` | BIGINT | No |  | FK, IDX | Staff |
| `training_records_payload` | JSONB | Yes | NULL |  | 教育訓練実施記録: list [{date, type, hours, content, instructor}]. Display-only audit, không filter individual. ≥5 fields per item |
| `career_consulting_records_payload` | JSONB | Yes | NULL |  | キャリアコンサルティング実施記録: list [{date, counselor, topic, outcome, follow_up}]. Display-only |
| `complaint_handling_payload` | JSONB | Yes | NULL |  | 苦情処理状況 (MOTO-side): list [{received_date, source, category, content, action_taken, resolved_date}]. Display-only |
| `capability_evaluation_payload` | JSONB | Yes | NULL |  | 業務遂行能力評価: list [{evaluation_date, evaluator, rating, comments}]. Display-only |
| `health_check_records_payload` | JSONB | Yes | NULL |  | 健康診断実施状況: list [{check_date, type, result_summary, next_check_due}]. Display-only |
| `notes` | TEXT | Yes | NULL |  | Ghi chú tự do |
| `last_updated_for_ledger_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm cuối cập nhật phục vụ ledger (≠ updated_at — có thể update bản ghi nhưng không phải cho ledger) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (KHÔNG dùng trước retention 3 năm) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_moto_ledger_manual: (contract_no, staff_id) WHERE deleted_at IS NULL — 1 row per (contract, staff)
- **INDEX** idx_moto_ledger_staff: (staff_id, contract_no)
- **INDEX** idx_moto_ledger_company_time: (company_id, last_updated_for_ledger_at DESC)
- **Trigger** trg_moto_ledger_updated_at: BEFORE UPDATE → fn_set_updated_at()

---

### `t_moto_company_scheduled_change` (Thay đổi Company Master đã lên lịch — 適用日予約)

> Lưu MỘT thay đổi `mst_moto_company` đã lên lịch hiệu lực tương lai (e-staffing `適用日を予約する`). Batch hằng ngày (họ D16) tới `effective_date` thì apply `payload` vào `mst_moto_company`, set `status='APPLIED'`, ghi before/after vào `t_moto_audit_log` (= 変更履歴参照). Tối đa 1 PENDING/công ty. History dùng audit-log, KHÔNG versioning master row (D14). "Áp ngay" = update in-place thường, không tạo dòng ở đây. `lock_version` theo D18.2.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (MOTO) |
| `company_id` | BIGINT | No |  | FK, IDX | Công ty bị thay đổi → `mst_moto_company(id)` |
| `payload` | JSONB | No |  |  | Bộ giá trị company-master mới sẽ áp dụng (snapshot field đã sửa). Apply-only, không filter từng field — D3.3 JSONB OK |
| `effective_date` | DATE | No |  | IDX | Ngày thay đổi có hiệu lực |
| `status` | VARCHAR(20) | No | 'PENDING' | IDX | PENDING, APPLIED, CANCELLED |
| `applied_at` | TIMESTAMP | Yes | NULL |  | Thời điểm batch áp dụng |
| `requested_by` | BIGINT | Yes | NULL | FK, IDX | User đặt lịch → `mst_moto_user(id)` |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — admin có thể sửa/huỷ trước ngày apply |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID người tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID người cập nhật bản ghi |
- **PK**: (id)
- **FK** fk_moto_company_sched_company: (company_id) → mst_moto_company (id)
- **FK** fk_moto_company_sched_user: (requested_by) → mst_moto_user (id)
- **CHECK** chk_moto_company_sched_status: status IN ('PENDING', 'APPLIED', 'CANCELLED')
- **UNIQUE (Partial)** uq_moto_company_scheduled_pending: (company_id) WHERE status = 'PENDING' AND deleted_at IS NULL
- **INDEX (Partial)** idx_moto_company_sched_batch: (status, effective_date) WHERE deleted_at IS NULL
- **RLS policy** — p_moto_company_sched: USING (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_moto_tenant_id', true)::bigint)
- **Trigger** trg_moto_company_sched_updated_at: BEFORE UPDATE → fn_set_updated_at()

---

---