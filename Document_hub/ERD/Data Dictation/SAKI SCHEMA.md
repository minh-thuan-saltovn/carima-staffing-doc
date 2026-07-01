# SAKI SCHEMA

## Nhóm 1: Master Cơ cấu Tổ chức SAKI (Organization)

### `tmp_saki_setup_credential` (Khởi tạo tài khoản SAKI lần đầu)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `token_hash` | VARCHAR(255) | No |  | **UK** | Token xác thực onboarding |
| `expires_at` | TIMESTAMP | No |  | IDX | Ngày hết hạn |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm credential đã được sử dụng |
| `revoked_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm credential bị thu hồi |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, USED, EXPIRED, REVOKED |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `mst_saki_company` (Thông tin Pháp nhân Công ty Khách hàng SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_code` | VARCHAR(50) | No |  | **UK** | Mã công ty (Client Code) |
| `company_name` | VARCHAR(255) | No |  |  | Tên chính thức (正式企業名) |
| `company_name_kana` | VARCHAR(255) | No |  |  | Tên chính thức Katakana (正式企業名（カナ）) |
| `company_name_en` | VARCHAR(255) | Yes | NULL |  | Tên chính thức tiếng Anh (正式企業名（英字）) |
| `display_name` | VARCHAR(255) | No |  |  | Tên hiển thị hệ thống (システム表示企業名) |
| `display_name_en` | VARCHAR(255) | Yes | NULL |  | Tên hiển thị tiếng Anh (システム表示企業名（英字）) |
| `representative_name` | VARCHAR(100) | Yes | NULL |  | Tên người đại diện |
| `postal_code` | VARCHAR(8) | Yes | NULL |  | Mã bưu chính (郵便番号) |
| `address` | VARCHAR(255) | Yes | NULL |  | Địa chỉ trụ sở (住所1) |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Inactive |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `mst_saki_company_contact` (Thông tin liên hệ / người phụ trách công ty SAKI)

> Đối xứng với `mst_moto_company_contact`. Lưu người phụ trách như e-staffing担当者1/2, đầu mối liên hệ. SAKI company không nhúng contact trực tiếp để giữ đối xứng MOTO.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `contact_type` | VARCHAR(50) | Yes | NULL | IDX | Loại liên hệ (VD: E_STAFFING_PIC1, E_STAFFING_PIC2, GENERAL) |
| `contact_name` | VARCHAR(100) | No |  |  | Tên người/bộ phận liên hệ |
| `department_name` | VARCHAR(100) | Yes | NULL |  | Tên phòng ban liên hệ |
| `email` | VARCHAR(255) | Yes | NULL |  | Email liên hệ |
| `phone_number` | VARCHAR(50) | Yes | NULL |  | Số điện thoại |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **RLS policy** — p_saki_company_contact: USING (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)

### `mst_saki_office` (Văn phòng / Chi nhánh / 事業所)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `office_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã văn phòng |
| `office_name` | VARCHAR(100) | No |  |  | Tên chính thức (正式事業所名) |
| `office_name_en` | VARCHAR(100) | Yes | NULL |  | Tên chính thức tiếng Anh (正式事業所名（英字）) |
| `display_name` | VARCHAR(100) | No |  |  | Tên hiển thị hệ thống (システム表示事業所名) |
| `display_name_en` | VARCHAR(100) | Yes | NULL |  | Tên hiển thị tiếng Anh (システム表示事業所名（英字）) |
| `conflict_application_office_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu `mst_saki_conflict_application_office(id)` (N:1); nếu set, office dùng 抵触日/待遇 của cơ sở áp dụng gộp |
| `address` | VARCHAR(255) | Yes | NULL |  | Địa chỉ |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Inactive |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **UNIQUE (Partial)** uq_saki_office_code: (company_id, office_code) WHERE deleted_at IS NULL

### `mst_saki_department` (Phòng ban / 部署)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `office_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu Văn phòng |
| `approval_group_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ tới nhóm duyệt mặc định của phòng |
| `department_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã phòng ban |
| `department_name` | VARCHAR(100) | No |  |  | Tên chính thức (正式部署名) |
| `department_name_en` | VARCHAR(100) | Yes | NULL |  | Tên chính thức tiếng Anh (正式部署名（英字）) |
| `display_name` | VARCHAR(100) | No |  |  | Tên hiển thị hệ thống (システム表示部署名) |
| `display_name_en` | VARCHAR(100) | Yes | NULL |  | Tên hiển thị tiếng Anh (システム表示部署名（英字）) |
| `phone_number` | VARCHAR(50) | Yes | NULL |  | Số điện thoại phòng ban (電話番号) |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Inactive |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **UNIQUE (Partial)** uq_saki_dept_code: (company_id, department_code) WHERE deleted_at IS NULL

### `mst_saki_conflict_date_office` (Ngày giới hạn phái cử theo đơn vị / 事業所抵触日)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `office_id` | BIGINT | No |  | FK, **UK** | Tham chiếu Văn phòng (1:1) |
| `conflict_date` | DATE | No |  | IDX | Ngày giới hạn pháp lý 3 năm (事業所抵触日) |
| `welfare_education_training` | TEXT | Yes | NULL |  | 待遇: 教育訓練 (đào tạo) — thông tin đãi ngộ đối chiếu cung cấp cho MOTO |
| `welfare_meal_facility` | TEXT | Yes | NULL |  | 待遇: 給食施設 (cơ sở ăn uống) |
| `welfare_break_room` | TEXT | Yes | NULL |  | 待遇: 休憩室 (phòng nghỉ) |
| `welfare_changing_room` | TEXT | Yes | NULL |  | 待遇: 更衣室 (phòng thay đồ) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `mst_saki_conflict_application_office` (抵触日適用事業所 — Cơ sở áp dụng ngày 抵触 gộp nhiều office)

> Dùng khi nhiều `mst_saki_office` chia sẻ chung một cơ sở bảo hiểm lao động (雇用保険適用事業所) hoặc cùng một địa điểm nhiều tầng. Nhiều office trỏ về một bản ghi này (N:1) để dùng chung 事業所抵触日 + 待遇 4 mục.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `application_office_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã 抵触日適用事業所 (採番 được) |
| `application_office_name` | VARCHAR(100) | No |  |  | Tên 抵触日適用事業所 |
| `address` | VARCHAR(255) | Yes | NULL |  | Địa chỉ |
| `conflict_date` | DATE | No |  | IDX | 事業所抵触日 chung cho nhóm |
| `welfare_education_training` | TEXT | Yes | NULL |  | 待遇: 教育訓練 |
| `welfare_meal_facility` | TEXT | Yes | NULL |  | 待遇: 給食施設 |
| `welfare_break_room` | TEXT | Yes | NULL |  | 待遇: 休憩室 |
| `welfare_changing_room` | TEXT | Yes | NULL |  | 待遇: 更衣室 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **UNIQUE (Partial)** uq_saki_conflict_app_office_code: (company_id, application_office_code) WHERE deleted_at IS NULL
- **RLS policy** — p_saki_conflict_app_office: USING (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)

### `mst_saki_business_partner` (Đối tác kinh doanh chung của SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `partner_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã đối tác |
| `partner_name` | VARCHAR(100) | No |  |  | Tên đối tác |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **UNIQUE (Partial)** uq_saki_partner_code: (company_id, partner_code) WHERE deleted_at IS NULL

---

## Người dùng, Vai trò & Scope Truy cập SAKI

### `mst_saki_permission` (Danh mục quyền hệ thống SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `permission_key` | VARCHAR(100) | No |  | **UK** | VD: `saki.inquiry.create` |
| `parent_permission_id` | BIGINT | Yes | NULL | FK, IDX | Quyền cha để dựng cây menu/chức năng |
| `permission_type` | VARCHAR(20) | No | 'ACTION' | IDX | MENU, ACTION, DATA_SCOPE, LOGIN_ACCESS |
| `display_order` | INTEGER | No | 0 |  | Thứ tự hiển thị khi import/dựng màn hình phân quyền |
| `external_permission_no` | VARCHAR(50) | Yes | NULL | IDX | Mã/No quyền tương ứng e-staffing nếu cần mapping/import |
| `description` | VARCHAR(255) | Yes | NULL |  | Mô tả |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `mst_saki_role` (Vai trò nội bộ SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `role_name` | VARCHAR(100) | No |  |  | Tên vai trò (VD: Quản lý bộ phận) |
| `is_system` | SMALLINT | No | 0 |  | 1: Không được xóa |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **UNIQUE (Partial)** uq_saki_role_name: (company_id, role_name) WHERE deleted_at IS NULL

### `mst_saki_permission_role` (Bảng Pivot gán quyền vào Role)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `role_id` | BIGINT | No |  | **PK**, FK | Tham chiếu `mst_saki_role(id)` |
| `permission_id` | BIGINT | No |  | **PK**, FK | Tham chiếu `mst_saki_permission(id)` |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `mst_saki_user` (Người dùng SAKI / Khách hàng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `office_id` | BIGINT | Yes | NULL | FK, IDX | Chi nhánh của User |
| `department_id` | BIGINT | Yes | NULL | FK, IDX | Phòng ban của User |
| `role_id` | BIGINT | No |  | FK, IDX | Vai trò |
| `login_id` | VARCHAR(100) | No |  | **UNIQUE** uq_saki_login_id: (company_id, login_id) | Tên đăng nhập |
| `password_hash` | VARCHAR(255) | No |  |  | Mật khẩu mã hóa |
| `full_name` | VARCHAR(100) | No |  |  | Họ tên (ユーザー氏名) |
| `full_name_kana` | VARCHAR(100) | No |  |  | Họ tên Katakana (ユーザー氏名（カナ）) |
| `email` | VARCHAR(255) | Yes | NULL |  | Email |
| `notification_email_lang` | VARCHAR(10) | No | 'ja' |  | Ngôn ngữ email thông báo hệ thống (システムからの通知メール言語); bắt buộc theo 企業設定-派遣先 tr.39 |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Locked |
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
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `tmp_saki_pwd_reset_token` (Token quên mật khẩu SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_user(id)` |
| `token_hash` | VARCHAR(255) | No |  | **UK** |  |
| `expires_at` | TIMESTAMP | No |  | IDX |  |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token đã được sử dụng |
| `revoked_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token bị thu hồi/hủy |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, USED, EXPIRED, REVOKED |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `t_saki_login_history` (Lịch sử đăng nhập User SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu `mst_saki_user(id)`; NULL nếu login_id không tồn tại |
| `login_id` | VARCHAR(100) | No |  | IDX | Login ID được nhập |
| `auth_provider` | VARCHAR(50) | No | 'LOCAL' | IDX | LOCAL, AZURE_AD, OKTA |
| `result` | VARCHAR(20) | No |  | IDX | SUCCESS, FAILED, LOCKED, MFA_REQUIRED, MFA_FAILED |
| `failure_reason` | VARCHAR(100) | Yes | NULL |  | PASSWORD_MISMATCH, USER_NOT_FOUND, ACCOUNT_LOCKED, TOKEN_EXPIRED... |
| `ip_address` | VARCHAR(45) | Yes | NULL | IDX | IPv4/IPv6 của client hoặc gateway |
| `user_agent` | TEXT | Yes | NULL |  | User-Agent tại thời điểm đăng nhập |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm ghi nhận login attempt |

### `t_saki_auth_session` (Phiên đăng nhập / Refresh Token User SAKI)

> BF-M01 enterprise short session policy: DB là source of truth, Redis chỉ dùng cache/denylist/rate-limit. Access token JWT TTL mặc định 15 phút; refresh token absolute lifetime mặc định 8 giờ; idle timeout mặc định 60 phút; không có remember-me.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Global tenant_id từ Platform |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `session_id` | UUID | No |  | **UK** | ID phiên đưa vào JWT claim để revoke/cache |
| `token_family_id` | UUID | No |  | IDX | Nhóm refresh token rotation; reuse token cũ sẽ revoke cả family |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_user(id)` |
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
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

**Rule:** Refresh token rotation là bắt buộc. Nếu refresh token đã `ROTATED`/`REVOKED` bị dùng lại, revoke toàn bộ `token_family_id` và ghi audit/security event.

### `t_saki_user_2fa` (Trạng thái xác thực 2 bước của User SAKI)

> 2FA email-code + backup code (tham khảo 2段階認証マニュアル). Ngưỡng số lần/thời lượng khóa là tham số cấu hình ở `cfg_platform_system_setting` hoặc tenant setting (tài liệu dùng "一定", không hardcode). Khóa email-code theo thời gian (`auth_code_locked_until`); khóa backup-code tới khi update (`backup_code_locked_flg`).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `mst_saki_user(id)` (1:1) |
| `two_factor_enabled_flg` | SMALLINT | No | 0 | IDX | 1: User đã bật 2FA (opt-in, cần đăng ký trước) |
| `email_for_2fa` | VARCHAR(255) | Yes | NULL |  | Email nhận mã xác thực |
| `auth_code_hash` | VARCHAR(255) | Yes | NULL |  | Hash mã xác thực email hiện hành |
| `auth_code_expires_at` | TIMESTAMP | Yes | NULL |  | Hết hạn mã email |
| `auth_code_attempt_count` | SMALLINT | No | 0 |  | Số lần nhập sai mã email |
| `auth_code_resend_count` | SMALLINT | No | 0 |  | Số lần gửi lại mã email |
| `auth_code_locked_until` | TIMESTAMP | Yes | NULL | IDX | Khóa luồng email-code theo thời gian (tự mở) |
| `backup_code_locked_flg` | SMALLINT | No | 0 |  | 1: Khóa luồng backup code tới khi user/admin update |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **RLS policy** — p_saki_2fa_tenant: USING (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)

### `t_saki_2fa_backup_code` (Backup code 2FA của User SAKI)

> Mỗi backup code dùng một lần (`used_at`); sau khi dùng tự làm mới cả bộ. User tự làm mới bất kỳ lúc nào; admin chỉ làm mới khi đang khóa.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_user(id)` |
| `code_hash` | VARCHAR(255) | No |  | IDX | Hash của backup code |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm code đã dùng (NULL = còn hiệu lực) |
| `generation` | INTEGER | No | 1 |  | Phiên bản bộ backup code (tăng mỗi lần làm mới) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **RLS policy** — p_saki_2fa_backup_tenant: USING (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)

### `mst_saki_user_scope` (Phạm vi dữ liệu Data Scope của User)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_user(id)` |
| `scope_type` | VARCHAR(50) | No |  |  | ASSIGNED_ONLY, DEPARTMENT, BRANCH, ALL |
| `target_id` | BIGINT | Yes | NULL | IDX | ID tương ứng nếu Custom Scope |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

---

## Nhóm 3: Cấu hình Luồng Duyệt (Approval Workflow)

### `mst_saki_approval_group` (Nhóm phê duyệt)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `group_name` | VARCHAR(100) | No |  |  | Tên nhóm phê duyệt |
| `description` | VARCHAR(255) | Yes | NULL |  |  |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `mst_saki_approval_group_user` (Thành viên nhóm phê duyệt)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `approval_group_id` | BIGINT | No |  | FK, IDX |  |
| `user_id` | BIGINT | No |  | FK, IDX |  |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

---

## Nhóm 4: Cấu hình Từ Điển / Nghiệp Vụ Riêng Của SAKI

### `mst_saki_custom_job_type` (Loại công việc định nghĩa riêng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `global_job_id` | BIGINT | Yes | NULL | **Logical FK** | Trỏ về Job Type chung trên Platform (Nếu có) |
| `custom_job_code` | VARCHAR(50) | No |  |  | Mã công việc nội bộ SAKI |
| `custom_job_name` | VARCHAR(100) | No |  |  | Tên công việc nội bộ SAKI |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `mst_saki_custom_job_type_skill` (Kỹ năng yêu cầu cho Job riêng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `custom_job_type_id` | BIGINT | No |  | FK, IDX |  |
| `global_skill_id` | BIGINT | Yes | NULL | **Logical FK** | Tham chiếu logic `global_master.mst_skill(id)` trong cùng DB environment |
| `global_skill_code` | VARCHAR(50) | No |  | IDX | Stable code tham chiếu `global_master.mst_skill.skill_code`, dùng cho export/import/cross-environment |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

---

## Nhóm 5: Cấu hình Khung Giao Dịch SAKI (Configurations)

### `cfg_saki_work_schedule_default` (Lịch làm việc mặc định)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Quan hệ 1:1 với SAKI Company |
| `start_time` | SMALLINT | Yes | NULL |  | Giờ vào ca mặc định (Phút) |
| `end_time` | SMALLINT | Yes | NULL |  | Giờ ra ca mặc định (Phút) |
| `break_time_minutes` | SMALLINT | No | 60 |  | Số phút nghỉ trưa mặc định |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `cfg_saki_treatment_info` (Thông tin Đãi ngộ - 待遇情報)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Quan hệ 1:1 |
| `treatment_content` | TEXT | No |  |  | Thông tin đãi ngộ mặc định cung cấp cho MOTO |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `cfg_saki_contract_item_control` (Kiểm soát hiển thị Hợp đồng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `item_key` | VARCHAR(50) | No |  |  | Mã Field trên UI hợp đồng |
| `is_visible` | SMALLINT | No | 1 |  | 1: Cho phép hiển thị |
| `is_editable` | SMALLINT | No | 1 |  | 1: Cho phép SAKI sửa |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `cfg_saki_contract_remarks_control` (Kiểm soát Ghi chú Hợp đồng)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `remark_key` | VARCHAR(50) | No |  |  | Mã khối ghi chú (VD: REMARK_1) |
| `remark_default_text` | TEXT | Yes | NULL |  | Text mặc định |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `cfg_saki_alarm_setting` (Cấu hình Cảnh báo chung) [M08 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `alarm_type` | VARCHAR(50) | No |  | **UK** | Loại báo động (ATTENDANCE_LATE, INVOICE_UNAPPROVED) |
| `is_enabled` | SMALLINT | No | 1 |  | 1: Bật, 0: Tắt |
| `channel_email_flg` | SMALLINT | No | 1 |  | 1=Gửi email / 0=Không (BR-101) **[M08 NEW]** |
| `channel_inapp_flg` | SMALLINT | No | 1 |  | 1=Hiển thị in-app / 0=Không. Với is_legal_alert=1 phải = 1 (BR-101/126) **[M08 NEW]** |
| `alert_days_before` | INT | Yes | NULL |  | Báo trước N ngày (parity với MOTO; áp dụng cho contract-end/treatment-deadline alarm) **[M08 NEW]** |
| `is_legal_alert` | SMALLINT | No | 0 | IDX | 1=Cảnh báo pháp lý (36協定/抵触日 sau M06) — app layer chặn UI tắt in-app **[M08 NEW]** |
| `remind_frequency` | SMALLINT | No | 1 |  | 1=ONCE 一回 / 2=DAILY 毎日 / 3=WEEKLY 毎週 (BR-130) **[M08 NEW]** |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |
- **CHECK** chk_saki_alarm_channels: channel_email_flg IN (0, 1) AND channel_inapp_flg IN (0, 1)
- **CHECK** chk_saki_alarm_is_enabled: is_enabled IN (0, 1)
- **CHECK** chk_saki_alarm_legal: is_legal_alert IN (0, 1)
- **CHECK** chk_saki_alarm_remind_freq: remind_frequency IN (1, 2, 3)
- **CHECK** chk_saki_alarm_legal_inapp: is_legal_alert = 0 OR channel_inapp_flg = 1 (BR-101: Cảnh báo pháp lý không thể tắt in-app)
- **CHECK** chk_saki_alarm_days_before: alert_days_before IS NULL OR alert_days_before > 0

### `cfg_saki_alarm_send_condition` (Điều kiện kích hoạt Cảnh báo)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `alarm_setting_id` | BIGINT | No |  | FK, IDX | Trỏ tới bảng cấu hình gốc |
| `condition_type` | VARCHAR(50) | No |  |  | VD: DAYS_BEFORE_DEADLINE |
| `threshold_value` | INT | No |  |  | Giá trị ngưỡng (VD: 3 ngày) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `cfg_saki_alarm_specified_user` (Người nhận Cảnh báo chỉ định)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `alarm_setting_id` | BIGINT | No |  | FK, IDX |  |
| `user_id` | BIGINT | No |  | FK, IDX | User nhận email/notify |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `cfg_saki_36_agreement_alarm` (Cảnh báo luật 36協定 dành cho SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, **UK** | Quan hệ 1:1 |
| `enabled_flg` | SMALLINT | No | 1 |  | 1: Bật cảnh báo quá giờ OT cho người quản lý SAKI |
| `alarm_hours_threshold` | INT | No | 40 |  | Ngưỡng báo động (VD: 40 giờ/tháng) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

### `cfg_saki_holiday` (Cấu hình Ngày nghỉ SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `rule_type` | SMALLINT | No | 1 |  | 1: Nghỉ cuối tuần, 2: Ngày cụ thể |
| `specified_day_of_week` | SMALLINT | Yes | NULL |  | Nếu type=1 (0: CN, 6: T7) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | Người tạo |
| `updated_by` | BIGINT | Yes | NULL |  | Người cập nhật |

---

## Nhóm 6: Cấu hình Ngày nghỉ SAKI

### `cfg_saki_specified_date` (Ngày chỉ định đặc biệt trong lịch SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `holiday_setting_id` | BIGINT | No |  | FK, IDX | Tham chiếu `cfg_saki_holiday(id)` |
| `specified_date` | DATE | No |  | IDX | Ngày cụ thể (VD: Ngày thành lập công ty) |
| `is_working_day` | SMALLINT | No | 0 |  | 1: Đi làm, 0: Nghỉ |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

## Nhóm 7: Yêu cầu Phái cử (Dispatch Inquiry)

### `t_saki_opinion_hearing` (Phiếu lấy ý kiến / Điều trần - 意見聴取)

> Bổ sung 11 cột pháp lý theo 派遣法 第40条の2 第4項 (事業所単位の期間制限延長 cần lấy ý kiến 過半数労働組合 hoặc 過半数代表者). Reference template: `instructions/意見聴取結果_20170611.pdf` + `導入手続きマニュアル派遣先`. Mỗi 事業所 cần 1 biên bản trước khi extend 派遣可能期間.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `target_office_id` | BIGINT | No |  | FK, IDX | **[Reviewer Fix 2]** 対象事業所 — link `mst_saki_office(id)`. 1 hearing áp dụng 1 事業所. |
| `representative_type` | SMALLINT | No |  |  | **[Reviewer Fix 2]** 1=過半数労働組合 / 2=過半数代表者 (nếu không có công đoàn quá bán) |
| `representative_name` | VARCHAR(200) | No |  |  | **[Reviewer Fix 2]** 組合名 hoặc 代表者氏名 (1 hoặc 2 tuỳ representative_type) |
| `selection_method` | SMALLINT | Yes | NULL |  | **[Reviewer Fix 2]** Cách thức bầu chọn (chỉ khi representative_type=2): 1=投票 2=挙手 3=話し合い 4=その他 |
| `selection_method_detail` | VARCHAR(500) | Yes | NULL |  | **[Reviewer Fix 2]** Chi tiết cách bầu chọn (khi selection_method=その他, hoặc giải thích thêm) |
| `notice_date` | DATE | No |  |  | **[Reviewer Fix 2]** 意見聴取通知日 — ngày gửi thông báo lấy ý kiến |
| `notice_content` | TEXT | No |  |  | **[Reviewer Fix 2]** 通知事項 — nội dung thông báo gửi 過半数代表者 (≤3774 byte per template) |
| `hearing_period_from` | DATE | Yes | NULL |  | **[Reviewer Fix 2]** 聴取期間開始 — khoảng thời gian cho phép góp ý |
| `hearing_period_to` | DATE | Yes | NULL |  | **[Reviewer Fix 2]** 聴取期間終了 |
| `hearing_date` | DATE | No |  |  | Ngày thực hiện lấy ý kiến (kết thúc 意見聴取) |
| `opinion_content` | TEXT | Yes | NULL |  | **[Reviewer Fix 2]** 意見内容 — nội dung ý kiến đã thu nhận (≤3774 byte) |
| `result_status` | VARCHAR(20) | No |  | IDX | Kết quả (AGREED, DISAGREED, NO_OPINION) |
| `publication_method` | VARCHAR(500) | Yes | NULL |  | **[Reviewer Fix 2]** 派遣先労働者への周知方法 — required by 派遣法 (vd "社内掲示", "イントラネット掲載") |
| `extension_target_period_from` | DATE | Yes | NULL |  | **[Reviewer Fix 2]** 延長後の派遣可能期間 from — áp dụng cho 事業所単位の期間制限 |
| `extension_target_period_to` | DATE | Yes | NULL |  | **[Reviewer Fix 2]** 延長後の派遣可能期間 to (vd +3 năm từ initial) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
- **FK** fk_opinion_office: (target_office_id) → mst_saki_office(id)
- **INDEX** idx_opinion_hearing_office_date: (target_office_id, hearing_date DESC) — lookup latest hearing per 事業所
- **CHECK** chk_representative_type: representative_type IN (1, 2)
- **CHECK** chk_selection_method: selection_method IS NULL OR selection_method IN (1, 2, 3, 4)
- **CHECK** chk_selection_consistency: (representative_type = 1) OR (representative_type = 2 AND selection_method IS NOT NULL) — nếu là 代表者 thì phải có cách bầu chọn
- **CHECK** chk_hearing_period: (hearing_period_from IS NULL AND hearing_period_to IS NULL) OR (hearing_period_to >= hearing_period_from)
- **CHECK** chk_extension_period: (extension_target_period_from IS NULL AND extension_target_period_to IS NULL) OR (extension_target_period_to > extension_target_period_from)

### `t_dispatch_inquiry_draft` (Bản nháp Yêu cầu Phái cử)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Người dùng SAKI đang soạn nháp |
| `inquiry_payload` | JSONB | No |  |  | Lưu tạm toàn bộ dữ liệu form |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `draft_expires_at` | TIMESTAMP | Yes | NULL | IDX | Hạn lưu nháp (D18.10) = thời điểm save + TTL; NULL khi không ở trạng thái draft. Set lúc save-draft, clear (NULL) khi submit. |
| `expiration_status` | VARCHAR(10) | Yes | NULL |  | normal / warning / expired (D18.10); NULL khi không phải draft; batch hằng ngày maintain tier. |

### `t_dispatch_inquiry` (Yêu cầu Phái cử chính thức - 派遣照会)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `inquiry_no` | VARCHAR(50) | No |  | **UK** | Mã yêu cầu phái cử |
| `inquiry_type` | SMALLINT | No | 2 | IDX | 1=見積依頼 (yêu cầu báo giá) 2=派遣照会 (yêu cầu phái cử) — dùng chung quy trình |
| `office_id` | BIGINT | No |  | FK, IDX | Văn phòng cần người |
| `department_id` | BIGINT | No |  | FK, IDX | Phòng ban cần người |
| `user_id` | BIGINT | No |  | FK, IDX | Người tạo yêu cầu (起票者) |
| `status` | VARCHAR(20) | No | 'SUBMITTED' | IDX | DRAFT, SUBMITTED, ESTIMATING(見積中), ESTIMATED(見積済), APPROVING(承認中), REJECTED(却下), WITHDRAWN(引戻し), CLOSED(クローズ) |
| `headcount_required` | INT | No | 1 |  | Số lượng nhân sự cần |
| `attached_files_payload` | JSONB | Yes | NULL |  | Files đính kèm SAKI upload (vd: 求人票 cho 紹介予定派遣). Display-only, ≥5 fields/item, không filter individual — D3 JSONB OK. Schema: `[{s3_key, filename, size_bytes, mime_type, uploaded_at, file_type: "JOB_POSTING"/"OTHER"}]`. Tối đa 5 files (validation ở app layer). **[Follow-up B NEW]** |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — bump +1 mỗi UPDATE; mọi UPDATE phải kèm `WHERE lock_version = ?`. Sửa bảng con bump version header. |
| `draft_expires_at` | TIMESTAMP | Yes | NULL | IDX | Hạn lưu nháp (D18.10) = thời điểm save + TTL; NULL khi không ở trạng thái draft. Set lúc save-draft, clear (NULL) khi submit. |
| `expiration_status` | VARCHAR(10) | Yes | NULL |  | normal / warning / expired (D18.10); NULL khi không phải draft; batch hằng ngày maintain tier. |
- **Generation Rule (Phase 3 Invariant):** Cột `inquiry_no` BẮT BUỘC do Sequence Service cấp phát theo định dạng `{tenant_prefix}-{company_prefix}-I{YYMM}-{seq}`. Cấm Client tự nhập.
- **INDEX (Partial)** idx_dispatch_inquiry_draft_cleanup: (status, draft_expires_at) WHERE deleted_at IS NULL

---

### `t_dispatch_inquiry_condition` (Điều kiện của Yêu cầu Phái cử)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_saki_company(id) |
| `inquiry_id` | BIGINT | No |  | FK, **UK** | Tham chiếu Yêu cầu phái cử (1:1) |
| `expected_start_date` | DATE | No |  |  | Ngày dự kiến bắt đầu |
| `expected_end_date` | DATE | Yes | NULL |  | Ngày dự kiến kết thúc |
| `work_location` | VARCHAR(200) | No |  |  | Địa điểm làm việc (Khôi phục từ bản cũ) |
| `work_time_start` | SMALLINT | No |  |  | Giờ bắt đầu làm (phút từ 00:00) |
| `work_time_end` | SMALLINT | No |  |  | Giờ kết thúc làm (phút từ 00:00) |
| `break_duration` | SMALLINT | No |  |  | Thời gian nghỉ (phút) |
| `overtime_flg` | SMALLINT | No | 0 |  | Cờ có làm thêm hay không (1: Có) |
| `work_mon`..`work_sun`,`work_holiday` | SMALLINT ×8 | No | 0 |  | 勤務日 bitset (月火水木金土日祝); 1=làm 0=nghỉ |
| `skill_requirements` | TEXT | Yes | NULL |  | Yêu cầu kỹ năng dạng text tự do |
| `hourly_rate_min` | INT | Yes | NULL |  | Ngân sách theo giờ (Min) |
| `hourly_rate_max` | INT | Yes | NULL |  | Ngân sách theo giờ (Max) |
| `condition_extra_payload` | JSONB | Yes | NULL |  | その他情報: 残業(0-30時間/月)+コメント, 更新(単位/数)+コメント, シフト+コメント, 依頼背景, 前任者, 引継ぎ有無, クライアント予定契約終了日, その他就業条件 (JSONB-first) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_dispatch_inquiry_job_type` (Loại công việc yêu cầu - 1:N)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `inquiry_id` | BIGINT | No |  | FK, IDX | Tham chiếu Yêu cầu phái cử |
| `job_code_snapshot` | VARCHAR(50) | No |  |  | Mã công việc lúc tạo |
| `job_name_snapshot` | VARCHAR(100) | No |  |  | Tên công việc lúc tạo |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_dispatch_inquiry_work_skill` (Kỹ năng nghiệp vụ yêu cầu - 1:N)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `inquiry_id` | BIGINT | No |  | FK, IDX | Tham chiếu Yêu cầu phái cử |
| `skill_code_snapshot` | VARCHAR(50) | No |  |  | Mã kỹ năng lúc tạo |
| `skill_name_snapshot` | VARCHAR(100) | No |  |  | Tên kỹ năng lúc tạo |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_dispatch_inquiry_lang_skill` (Kỹ năng Ngoại ngữ / PC yêu cầu - 1:N)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `inquiry_id` | BIGINT | No |  | FK, IDX | Tham chiếu Yêu cầu phái cử |
| `lang_skill_snapshot` | VARCHAR(100) | No |  |  | Tên ngôn ngữ / PC (VD: Tiếng Anh, Excel) |
| `required_level` | SMALLINT | No |  |  | 1: Cơ bản, 2: Trung cấp, 3: Nâng cao |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_dispatch_inquiry_office_env` (Môi trường văn phòng làm việc)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `inquiry_id` | BIGINT | No |  | FK, **UK** | Tham chiếu Yêu cầu phái cử (1:1) |
| `smoking_rule` | SMALLINT | No |  |  | 1: Cấm hút thuốc, 2: Có phòng hút thuốc |
| `dress_code` | SMALLINT | No |  |  | 1: Tự do, 2: Đồng phục, 3: Suit |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_dispatch_inquiry_target_company` (Danh sách công ty MOTO nhận yêu cầu)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `inquiry_id` | BIGINT | No |  | FK, IDX | Tham chiếu Yêu cầu phái cử |
| `moto_tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Tenant ID của MOTO nhận yêu cầu |
| `moto_company_code` | VARCHAR(50) | No |  | **Logical FK** , IDX | Mã Business Code của MOTO nhận thông báo |
| `status` | VARCHAR(20) | No | 'SENT' |  | SENT, VIEWED, RESPONDED, DECLINED, EXPIRED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **INDEX (composite)** idx_target_list_1: (company_id, inquiry_id, status)
- **INDEX (composite)** idx_target_list_2: (company_id, moto_tenant_id, status)

### `t_dispatch_inquiry_approval` (Luồng phê duyệt yêu cầu phái cử nội bộ SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `inquiry_id` | BIGINT | No |  | FK, IDX | Tham chiếu Yêu cầu phái cử |
| `approval_group_id` | BIGINT | No |  | FK, IDX | Nhóm duyệt thực hiện |
| `saki_user_id` | BIGINT | Yes | NULL | FK | Người duyệt cụ thể (actor — người thực sự bấm nút) |
| `proxy_for_user_id` | BIGINT | Yes | NULL | FK, IDX | **[Reviewer Fix 1]** ID user SAKI có thẩm quyền gốc khi 代理処理 (duyệt thay). NULL = tự duyệt. Bắt buộc cho 派遣事業所単位の期間制限延長 thanh tra. |
| `proxy_reason` | VARCHAR(500) | Yes | NULL |  | **[Reviewer Fix 1]** Lý do duyệt thay. Bắt buộc khi proxy_for_user_id IS NOT NULL — enforce ở App-layer (D13). |
| `status` | VARCHAR(20) | No | 'PENDING' |  | PENDING, APPROVED, REJECTED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_dispatch_estimate` (Báo giá phản hồi từ MOTO - View trên SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**  , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_saki_company(id) |
| `inquiry_id` | BIGINT | No |  | FK, IDX | Tham chiếu Yêu cầu phái cử |
| `based_on_saki_inquiry_revision_no` | SMALLINT | No |  | IDX | Số Revision của Yêu cầu mà báo giá này đáp ứng (Phục vụ truy vết Audit) |
| `target_company_id` | BIGINT | No |  | FK, IDX | Tham chiếu bảng Target Company |
| `moto_tenant_id` | BIGINT | No |  | **Logical FK**  , IDX | Tenant ID của MOTO (Phục vụ truy vết tuyệt đối) |
| `moto_company_code` | VARCHAR(50) | No |  | **Logical FK** , IDX | Company Code của MOTO |
| `moto_estimate_group_id` | BIGINT | No |  | **Logical FK** , IDX | ID Nhóm báo giá bên MOTO (Dùng để gom version) |
| `version` | SMALLINT | No | 1 |  | Phiên bản báo giá từ MOTO |
| `superseded_by_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ tới ID của version mới thay thế nó (Trên SAKI) |
| `correlation_id` | VARCHAR(100) | No |  | IDX | ID truy vết Workflow |
| `unit_price` | DECIMAL(12,0) | Yes | NULL | IDX | Đơn giá báo giá (Khôi phục từ bản cũ để Sort) |
| `unit_type` | SMALLINT | Yes | NULL |  | Đơn vị tính: 0:Giờ, 1:Ngày, 2:Tháng |
| `candidate_count` | SMALLINT | Yes | NULL |  | Số lượng ứng viên MOTO đề xuất |
| `estimate_payload` | JSONB | No |  |  | Data chi tiết Báo giá do MOTO gửi qua |
| `source_status` | VARCHAR(20) | No | 'RECEIVED' | IDX | Trạng thái nguồn từ MOTO: RECEIVED, WITHDRAWN, SUPERSEDED |
| `decision_status` | VARCHAR(20) | No | 'PENDING' | IDX | Quyết định của SAKI: PENDING, ACCEPTED, REJECTED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_saki_estimate_sync: (company_id, moto_tenant_id, moto_estimate_group_id, version)
- **UNIQUE (Partial)** uq_saki_estimate_active: (company_id, moto_tenant_id, moto_estimate_group_id) WHERE source_status = 'RECEIVED' AND deleted_at IS NULL
- **FK** fk_saki_estimate_superseded: (superseded_by_id) → t_dispatch_estimate(id)
- **INDEX (composite)** idx_saki_est_list_1: (company_id, inquiry_id, source_status)
- **INDEX (composite)** idx_saki_est_list_2: (company_id, target_company_id, source_status, version DESC)
- **INDEX (composite)** idx_saki_est_list_3: (company_id, moto_tenant_id, moto_estimate_group_id, version DESC)

### `t_dispatch_inquiry_revision_history` (Lịch sử chỉnh sửa Yêu cầu phái cử)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `inquiry_id` | BIGINT | No |  | FK, IDX | Tham chiếu Yêu cầu phái cử |
| `revision_number` | SMALLINT | No |  |  | Lần sửa đổi thứ N |
| `changed_by` | BIGINT | No |  | FK | Người SAKI thực hiện sửa |
| `change_summary` | TEXT | Yes | NULL |  | Tóm tắt thay đổi |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---

## Nhóm 8: Hệ thống, Security, Audit & Async Event Queue (SAKI)

### `t_saki_contract_upload_history` (Lịch sử SAKI Upload file Hợp đồng)

> **[Reviewer Fix 7 — 2026-06-29]** Bổ sung `upload_options_payload` JSONB để Job Worker chạy ngầm biết trạng thái 4 checkbox "マスタの最新情報で更新する" mà SAKI tick lúc upload. Ref: `instructions/【総合版】操作マニュアル派遣先_ver.1.7.9_26.3.8.pdf` page 48.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | User SAKI upload |
| `file_name` | VARCHAR(255) | No |  |  | Tên file (TSV/CSV) |
| `upload_options_payload` | JSONB | Yes | NULL |  | **[Reviewer Fix 7]** Trạng thái 4 checkbox "マスタの最新情報で更新する" tại thời điểm SAKI bấm Upload — truyền cho Background Job Worker. Schema: `{"overwrite_workplace_address": bool, "overwrite_pic_info": bool, "overwrite_office_conflict_date": bool, "overwrite_org_unit": bool}`. NULL = không overwrite (default behavior). Per page 48 操作マニュアル契約管理-派遣先. |
| `status` | VARCHAR(20) | No | 'PROCESSING' |  | PROCESSING, COMPLETED, FAILED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

> **Note:** Bảng này hiện đang **leaner** so với `t_saki_shift_upload_history` (newer table có thêm `s3_storage_key`, `file_hash`, `total/success/failed_records`, `started_at/completed_at`). Phase 3 polish nên cân nhắc parity hai upload history tables. Tạm thời chỉ fix gap nghiêm trọng (options payload).
> 

### `t_saki_shift_upload_history` (Lịch sử SAKI Upload/Download Lịch ca làm việc — シフト表アップロード/ダウンロード) [Follow-up A NEW + Reviewer Fix 8]

> SAKI upload/download シフト表 TSV/CSV: upload để confirm lịch ca của staff đang phái cử; download để tải dữ liệu về hệ thống lương nội bộ SAKI. **[Reviewer Fix 8 — 2026-06-29]** Bổ sung `operation_type` để track cả 2 chiều, đối xứng với `t_moto_shift_upload_history`. BR-1547 audit compliance yêu cầu lưu vết Download cả 2 phía (file chứa lịch trình cá nhân — privacy sensitive). Ref: `e-staffingシフト管理機能操作マニュアル派遣先`. Tách riêng khỏi `t_saki_contract_upload_history` vì có schema validation rules khác (period_year_month, shift_record_count) và downstream xử lý vào `t_moto_shift_schedule` (cross-tenant ghi). FK đến `t_saki_upload_file_result` cho error per-row.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | User SAKI thực hiện upload/download |
| `operation_type` | SMALLINT | No |  | IDX | **[Reviewer Fix 8]** 1=UPLOAD / 2=DOWNLOAD — BR-1547 audit cả 2 chiều. SAKI download シフト表 cho hệ thống lương nội bộ → bắt buộc log. |
| `file_name` | VARCHAR(255) | No |  |  | Tên file gốc (TSV/CSV) |
| `s3_storage_key` | VARCHAR(500) | Yes | NULL |  | **[Reviewer Fix 8]** S3 key — bắt buộc với UPLOAD (audit retention 1 năm), NULL với DOWNLOAD (chỉ log thao tác, không lưu file xuất ra). |
| `file_hash` | VARCHAR(64) | Yes | NULL |  | SHA-256 hash file gốc — chống giả mạo |
| `target_period_year_month` | VARCHAR(7) | No |  | IDX | Kỳ kế hoạch ca (YYYY-MM) — VD "2026-07" |
| `total_records` | INTEGER | No | 0 |  | Tổng số dòng shift trong file |
| `success_records` | INTEGER | No | 0 |  | Số dòng xử lý thành công (UPLOAD); = total cho DOWNLOAD |
| `failed_records` | INTEGER | No | 0 |  | Số dòng lỗi (chỉ UPLOAD) |
| `status` | VARCHAR(20) | No | 'PROCESSING' | IDX | PROCESSING, VALIDATING, COMPLETED, FAILED |
| `started_at` | TIMESTAMP | No | NOW() |  | Thời điểm bắt đầu xử lý |
| `completed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm hoàn thành |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_saki_shift_upload_period: (company_id, target_period_year_month, started_at DESC)
- **INDEX** idx_saki_shift_upload_status: (status) WHERE status IN ('PROCESSING', 'VALIDATING') AND deleted_at IS NULL — batch monitor
- **INDEX** idx_saki_shift_upload_operation: (operation_type, started_at DESC) — **[Reviewer Fix 8]** filter UPLOAD vs DOWNLOAD audit reports
- **FK** fk_saki_shift_upload_user: (user_id) → mst_saki_user(id)
- **CHECK** chk_saki_shift_upload_operation_type: operation_type IN (1, 2) — **[Reviewer Fix 8]**
- **CHECK** chk_saki_shift_upload_status: status IN ('PROCESSING', 'VALIDATING', 'COMPLETED', 'FAILED')
- **CHECK** chk_saki_shift_upload_records: total_records >= 0 AND success_records >= 0 AND failed_records >= 0 AND success_records + failed_records <= total_records
- **CHECK** chk_saki_shift_upload_period_format: target_period_year_month ~ '^[0-9]{4}-(0[1-9]|1[0-2])$'
- **CHECK** chk_saki_shift_upload_s3_consistency: operation_type = 2 OR s3_storage_key IS NOT NULL — **[Reviewer Fix 8]** UPLOAD bắt buộc có S3 key; DOWNLOAD không bắt buộc. Đối xứng với MOTO side.
- **Trigger** trg_saki_shift_upload_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_saki_batch_execution_history` (Nhật ký chạy Job Worker của SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | Yes | NULL | FK, IDX | Người kích hoạt job (nếu là export) |
| `job_name` | VARCHAR(100) | No |  |  | Tên Job |
| `status` | VARCHAR(20) | No | 'RUNNING' |  | RUNNING, SUCCESS, FAILED |
| `error_log` | TEXT | Yes | NULL |  | Chi tiết Exception |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_saki_audit_log` (Log thao tác hệ thống SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_saki_company(id) |
| `actor_type` | VARCHAR(20) | No | 'SAKI_USER' |  | SYSTEM, SAKI_USER, MOTO_USER (Cross-write) |
| `actor_tenant_id` | BIGINT | Yes | NULL | IDX | ID Tenant của người thao tác (Phục vụ SIEM cross-tenant) |
| `actor_company_code` | VARCHAR(50) | Yes | NULL | IDX | Mã Business Code của công ty người thao tác |
| `actor_id` | BIGINT | Yes | NULL | FK, IDX | User thao tác (Logical FK nếu là MOTO) |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID của HTTP Request (từ API Gateway) |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết Cross-tenant Event / Workflow |
| `action` | VARCHAR(100) | No |  |  | API Endpoint / Hành động |
| `entity_table` | VARCHAR(50) | No |  | IDX | Bảng bị tác động |
| `entity_id` | VARCHAR(100) | No |  | IDX | ID bị tác động |
| `before_data` | JSONB | Yes | NULL |  | Dữ liệu cũ |
| `after_data` | JSONB | Yes | NULL |  | Dữ liệu mới |
| `ip_address` | VARCHAR(50) | Yes | NULL |  | IP Address của người dùng |
| `user_agent` | VARCHAR(500) | Yes | NULL |  | Trình duyệt/Thiết bị |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
- **CHECK** chk_saki_audit_cross_actor: actor_type NOT IN ('MOTO_USER') OR (actor_tenant_id IS NOT NULL AND actor_company_code IS NOT NULL)

### `t_saki_inbox_event` (Hàng đợi nhận Event từ MOTO/Platform)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `source_tenant_id` | BIGINT | No | 0 | IDX | Nguồn phát Event (0 = SYSTEM/PLATFORM) |
| `source_tenant_type` | VARCHAR(20) | No |  |  | PLATFORM, MOTO, SYSTEM |
| `aggregate_type` | VARCHAR(50) | No |  | IDX | Loại thực thể (VD: INQUIRY) |
| `aggregate_id` | VARCHAR(100) | No |  | IDX | ID thực thể |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `event_type` | VARCHAR(100) | No |  | IDX | VD: MOTO_CONTRACT_SUBMITTED |
| `payload` | JSONB | No |  |  | Data từ MOTO gửi sang |
| `idempotency_key` | VARCHAR(100) | No |  | Composite UK | ID do nguồn sinh ra để chống xử lý trùng |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết luồng Log (Khớp với hệ thống) |
| `retry_count` | SMALLINT | No | 0 |  | Số lần Job thực hiện thử lại |
| `last_error` | TEXT | Yes | NULL |  | Chi tiết Exception lỗi cuối cùng |
| `processed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm xử lý xong |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm lên lịch thử lại tiếp theo |
| `status` | VARCHAR(20) | No | 'PENDING' |  | PENDING, PROCESSING, COMPLETED, FAILED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_saki_inbox_idem: (company_id, source_tenant_type, source_tenant_id, idempotency_key)
- **INDEX (composite)** idx_saki_inbox_aggregate: (aggregate_type, aggregate_id, created_at)
- **INDEX (composite)** idx_saki_inbox_worker: (status, next_retry_at)

### `t_saki_outbox_event` (Hàng đợi phát Event đi MOTO/Platform)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `target_tenant_id` | BIGINT | No | 0 | IDX | Đích đến Event (0 = PLATFORM/BROADCAST) |
| `target_tenant_type` | VARCHAR(20) | No |  |  | PLATFORM, MOTO, BROADCAST |
| `aggregate_type` | VARCHAR(50) | No |  | IDX | Loại thực thể |
| `aggregate_id` | VARCHAR(100) | No |  | IDX | ID thực thể |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `event_type` | VARCHAR(100) | No |  | IDX | VD: SAKI_INQUIRY_CREATED |
| `payload` | JSONB | No |  |  | Data SAKI gửi đi |
| `idempotency_key` | VARCHAR(100) | No |  | Composite UK | ID do SAKI sinh ra |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết luồng Log (Khớp với hệ thống) |
| `retry_count` | SMALLINT | No | 0 |  | Số lần Job thực hiện thử lại |
| `last_error` | TEXT | Yes | NULL |  | Chi tiết Exception lỗi cuối cùng |
| `processed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm xử lý xong |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm lên lịch thử lại tiếp theo |
| `status` | VARCHAR(20) | No | 'PENDING' |  | PENDING, SENDING, SENT, FAILED |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_saki_outbox_idem: (company_id, target_tenant_type, target_tenant_id, idempotency_key)
- **INDEX (composite)** idx_saki_outbox_aggregate: (aggregate_type, aggregate_id, created_at)
- **INDEX (composite)** idx_saki_outbox_worker: (status, next_retry_at)

### `t_saki_upload_file_result` (Chi tiết lỗi File Upload SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `upload_history_id` | BIGINT | No |  | FK, IDX | Trỏ tới bảng Upload History |
| `error_summary` | TEXT | Yes | NULL |  | Tổng hợp lỗi dòng nào sai |
| `result_file_key` | VARCHAR(255) | Yes | NULL |  | S3 Key file báo lỗi |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_saki_notification_read` (Trạng thái User SAKI đọc thông báo)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_user(id)` |
| `notification_id` | BIGINT | No |  | **Logical FK**, IDX | Thông báo chung từ Platform |
| `read_at` | TIMESTAMP | No | NOW() |  | Thời điểm đọc |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `mst_saki_cost_center` (Master Trung tâm chi phí SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `cost_center_code` | VARCHAR(50) | No |  | Composite Partial UK | Mã trung tâm chi phí |
| `cost_center_name` | VARCHAR(100) | No |  |  | Tên trung tâm chi phí |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_saki_cost_center: (company_id, cost_center_code) WHERE deleted_at IS NULL

### `t_saki_document_download_audit` (Audit User SAKI tải tài liệu nhạy cảm)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK** , IDX | Bảo mật Global Scope |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_saki_company(id) |
| `user_id` | BIGINT | No |  | FK, IDX | Người SAKI thực hiện tải |
| `document_type` | VARCHAR(50) | No |  |  | INVOICE, CONTRACT |
| `document_artifact_id` | BIGINT | No |  | **Logical FK** | Trỏ tới bảng Metadata PDF |
| `moto_tenant_id` | BIGINT | Yes | NULL | **Logical FK**, IDX | Tenant ID của MOTO sở hữu tài liệu này (Phục vụ SIEM) |
| `moto_company_code` | VARCHAR(50) | Yes | NULL | **Logical FK**, IDX | Company Code của MOTO sở hữu tài liệu |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID của HTTP Request (từ API Gateway) |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết luồng tải file sang hệ thống MOTO |
| `user_agent` | VARCHAR(500) | Yes | NULL |  | Trình duyệt/Thiết bị tải file |
| `downloaded_at` | TIMESTAMP | No | NOW() |  | Thời điểm tải |
| `ip_address` | VARCHAR(50) | Yes | NULL |  | Địa chỉ IP máy tính tải file |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_saki_document_download_moto_owner: moto_tenant_id IS NOT NULL AND moto_company_code IS NOT NULL

### `t_saki_notification_log` (Sổ Cái Thông báo SAKI — 通知監査ログ, 3 năm retention) [M08 NEW]

> **Audit master record** cho mọi event-driven notification gửi tới User SAKI. Pure-immutable insert-only (không versioning, không update sau khi tạo). Lưu **snapshot subject/body** rendered tại thời điểm noti — đáp ứng compliance 3 năm theo Performance/Compliance spec (Notification history 1-3 năm). Inbox UI 90 ngày fan-out riêng (`t_saki_inbox_notification`) — bảng này là **source of truth** cho lookup audit/dispute resolution.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `notification_type_code` | VARCHAR(20) | No |  | IDX | Logical FK → `platform.mst_notification_template(notification_type_code)` (cross-schema logical) |
| `category` | SMALLINT | No |  | IDX | 1=CONTRACT/2=ATTENDANCE/3=INVOICE/4=SYSTEM/5=ANNOUNCEMENT (copy từ template tại moment) |
| `priority` | SMALLINT | No | 1 | IDX | 1=NORMAL/2=IMPORTANT/3=URGENT |
| `subject_snapshot` | VARCHAR(500) | No |  |  | Subject rendered (Mustache → text) tại thời điểm tạo — D5 snapshot pattern |
| `body_snapshot` | TEXT | No |  |  | Body rendered — không reference template để render lại (vi phạm 電子帳簿保存法) |
| `template_variables_payload` | JSONB | Yes | NULL |  | Variables snapshot dùng render (audit replay): {contract_no: "...", staff_name: "..."} — display-only, ≥5 fields, không filter |
| `action_url` | VARCHAR(500) | Yes | NULL |  | Deep-link URL rendered |
| `sender_type` | SMALLINT | No |  |  | 1=SYSTEM (batch/event)/2=PLATFORM_ADMIN/3=USER (manual trigger) |
| `sender_user_id` | BIGINT | Yes | NULL |  | NULL nếu sender_type=SYSTEM; ngược lại là user gây ra event |
| `related_entity_type` | VARCHAR(50) | Yes | NULL | IDX | "CONTRACT"/"INVOICE"/"ATTENDANCE"/"DISPATCH_INQUIRY"... — trace back tới entity gốc |
| `related_entity_id` | BIGINT | Yes | NULL | IDX | ID entity (vd: [contract.id](http://contract.id/)) |
| `is_legal_alert` | SMALLINT | No | 0 | IDX | Đóng băng tại moment (copy từ template) — phục vụ audit "lúc đó có là cảnh báo pháp lý không" |
| `total_recipient_count` | INT | No | 0 |  | Số recipient fan-out — tổng cho summary view |
| `triggered_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm event xảy ra |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật (sẽ không cập nhật — immutable) |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (chỉ Platform Admin sau 3 năm) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_saki_notif_log_type: (notification_type_code, triggered_at DESC)
- **INDEX** idx_saki_notif_log_entity: (related_entity_type, related_entity_id)
- **INDEX** idx_saki_notif_log_legal: (is_legal_alert, triggered_at DESC) WHERE is_legal_alert = 1
- **CHECK** chk_saki_notif_log_category: category IN (1, 2, 3, 4, 5)
- **CHECK** chk_saki_notif_log_priority: priority IN (1, 2, 3)
- **CHECK** chk_saki_notif_log_sender_type: sender_type IN (1, 2, 3)
- **CHECK** chk_saki_notif_log_legal: is_legal_alert IN (0, 1)
- **CHECK** chk_saki_notif_log_recipient_count: total_recipient_count >= 0
- **CHECK** chk_saki_notif_log_type_format: notification_type_code ~ '^NTF-[A-Z]{1}[0-9]{2}$'
- **CHECK** chk_saki_notif_log_sender_user_consistency: sender_type = 1 OR sender_user_id IS NOT NULL (SYSTEM không cần user, các loại khác bắt buộc)
- **Trigger** trg_saki_notif_log_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_saki_inbox_notification` (Hộp thư SAKI — Per-User Inbox, 90 ngày retention) [M08 NEW]

> Fan-out per user — 1 row cho mỗi recipient của mỗi notification. Tối ưu cho query **badge unread**: `WHERE user_id=X AND is_read=0` (BR-104). Retention 90 ngày, batch hard-delete sau (BR-105). Reference `notification_log_id` cho audit replay; **denormalize** category/priority/subject_cache để tránh JOIN trên hot path (list view).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `user_id` | BIGINT | No |  | FK, IDX | Recipient SAKI user. RLS scope (user_id-aware) |
| `notification_log_id` | BIGINT | No |  | FK, IDX | Trỏ về `t_saki_notification_log(id)` cho audit replay |
| `notification_type_code` | VARCHAR(20) | No |  |  | Cached cho list filter (denormalized) |
| `category` | SMALLINT | No |  | IDX | Cached cho list filter |
| `priority` | SMALLINT | No | 1 | IDX | Cached cho list filter & ORDER BY |
| `subject_cache` | VARCHAR(500) | No |  |  | Cached cho list rendering — KHÔNG JOIN log mỗi lần render danh sách |
| `action_url` | VARCHAR(500) | Yes | NULL |  | Cached deep-link |
| `is_read` | SMALLINT | No | 0 | IDX | 0=Unread / 1=Read. Hot column — index để đếm badge nhanh |
| `read_at` | TIMESTAMP | Yes | NULL |  | Thời điểm read |
| `is_starred` | SMALLINT | No | 0 |  | 1=User pin (UX) |
| `delivered_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm fan-out tới user — dùng cho retention cleanup 90d |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (user dismiss) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_saki_inbox_user_unread: (user_id, is_read, delivered_at DESC) WHERE is_read = 0 AND deleted_at IS NULL — partial index cho badge count
- **INDEX** idx_saki_inbox_user_list: (user_id, delivered_at DESC) WHERE deleted_at IS NULL — main list view
- **INDEX** idx_saki_inbox_log: (notification_log_id) — reverse lookup
- **INDEX** idx_saki_inbox_cleanup: (delivered_at) WHERE deleted_at IS NULL — batch cleanup BR-105
- **UNIQUE** uq_saki_inbox_log_user: (notification_log_id, user_id) — chống duplicate fan-out
- **FK** fk_saki_inbox_log: (notification_log_id) → t_saki_notification_log(id)
- **CHECK** chk_saki_inbox_is_read: is_read IN (0, 1)
- **CHECK** chk_saki_inbox_is_starred: is_starred IN (0, 1)
- **CHECK** chk_saki_inbox_category: category IN (1, 2, 3, 4, 5)
- **CHECK** chk_saki_inbox_priority: priority IN (1, 2, 3)
- **CHECK** chk_saki_inbox_read_consistency: (is_read = 0 AND read_at IS NULL) OR (is_read = 1 AND read_at IS NOT NULL)
- **Trigger** trg_saki_inbox_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_saki_email_send_history` (Lịch sử Gửi Email SAKI — SES Delivery Tracking, 1 năm retention BR-109) [M08 NEW]

> Universal email send history cho SAKI tenant — mọi loại email (alarm, announcement, system) được track ở đây. 1 row per email send attempt. Lưu **subject_snapshot/body_snapshot** tại thời điểm gửi (D5 + 電子帳簿保存法). Tracking SES delivery status (sent/failed/bounced/complained/pending) cho retry logic (BR-110: 3 lần backoff 5min/30min/2h).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (RLS) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `notification_log_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ `t_saki_notification_log(id)` nếu email là output của event-driven noti. NULL với system email không gắn với notification_log (vd: reset password) |
| `notification_type_code` | VARCHAR(20) | No |  | IDX | Mã loại noti (NTF-C01...) hoặc SYS-* cho hệ thống |
| `recipient_user_id` | BIGINT | Yes | NULL | FK, IDX | NULL nếu gửi cho external email (không phải user SAKI) |
| `recipient_email` | VARCHAR(255) | No |  | IDX | Email người nhận (snapshot tại moment gửi — tránh mất nếu user đổi email) |
| `subject_snapshot` | VARCHAR(500) | No |  |  | Subject snapshot — audit BR-109 |
| `body_snapshot` | TEXT | No |  |  | Body snapshot — audit BR-109 + 電子帳簿保存法 |
| `status` | SMALLINT | No | 1 | IDX | 1=PENDING/2=SENDING/3=SENT/4=FAILED/5=BOUNCED/6=COMPLAINED |
| `ses_message_id` | VARCHAR(100) | Yes | NULL | IDX | Message ID từ SES — dùng tra cứu khi SES bounce webhook callback |
| `retry_count` | INT | No | 0 |  | Số lần retry (max 3 — BR-110) |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm retry tiếp theo — batch worker scan WHERE status=4 AND next_retry_at<=NOW |
| `last_attempted_at` | TIMESTAMP | Yes | NULL |  | Thời điểm thử gửi lần cuối |
| `sent_at` | TIMESTAMP | Yes | NULL |  | Thời điểm SES accept (status=3) |
| `error_message` | TEXT | Yes | NULL |  | Chi tiết lỗi (SES error code + message) |
| `related_entity_type` | VARCHAR(50) | Yes | NULL | IDX | Trace back: "CONTRACT"/"INVOICE"/... |
| `related_entity_id` | BIGINT | Yes | NULL | IDX |  |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_saki_email_status_retry: (status, next_retry_at) WHERE status = 4 AND deleted_at IS NULL — batch retry worker
- **INDEX** idx_saki_email_ses_msg: (ses_message_id) WHERE ses_message_id IS NOT NULL — SES bounce webhook lookup
- **INDEX** idx_saki_email_recipient: (recipient_email, created_at DESC)
- **INDEX** idx_saki_email_log: (notification_log_id) WHERE notification_log_id IS NOT NULL
- **INDEX** idx_saki_email_entity: (related_entity_type, related_entity_id)
- **FK** fk_saki_email_log: (notification_log_id) → t_saki_notification_log(id)
- **CHECK** chk_saki_email_status: status IN (1, 2, 3, 4, 5, 6)
- **CHECK** chk_saki_email_retry_count: retry_count >= 0 AND retry_count <= 3
- **CHECK** chk_saki_email_sent_consistency: (status <> 3 OR sent_at IS NOT NULL)
- **CHECK** chk_saki_email_failed_consistency: (status NOT IN (4, 5, 6) OR error_message IS NOT NULL)
- **Trigger** trg_saki_email_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_saki_ledger_manual_data` (Dữ liệu Nhập Tay cho 派遣先管理台帳 — SAKI Manual Ledger Fields) [M10 NEW]

> 派遣先管理台帳 (Article 42 派遣法) — 11 trường bắt buộc, trong đó các trường "nhập tay" KHÔNG có trong tables khác (苦情処理 SAKI nhận, biện pháp xử lý, ghi chú đào tạo từ SAKI, an toàn vệ sinh, bảo vệ thông tin cá nhân) được lưu ở đây. Khi export 台帳 (CMP-012) → app layer JOIN với contract/staff (cross-schema logical) + treatment notification để compile bản đầy đủ. Retention 3 năm theo BR-1538 (派遣法 Article 42); bản PDF export tĩnh trên S3 giữ 12年4ヶ月 theo 電子帳簿保存法 (Q1 user decision).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (SAKI tenant_id) |
| `company_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_saki_company(id)` |
| `contract_no` | VARCHAR(50) | No |  | IDX | Hợp đồng liên quan (logical reference đến `t_contract` trong MOTO schema — cross-schema) |
| `moto_tenant_id` | BIGINT | No |  | IDX | Tenant MOTO chứa contract gốc — logical FK |
| `staff_id` | BIGINT | No |  | IDX | Staff (logical reference đến `mst_staff` trong MOTO schema) |
| `complaint_handling_payload` | JSONB | Yes | NULL |  | 苦情処理状況 SAKI-side: list [{received_date, received_from, content, resolution_action, resolved_date, escalated_to_moto_flg}]. Display-only audit. ≥5 fields per item |
| `training_records_payload` | JSONB | Yes | NULL |  | 教育訓練実施状況 (SAKI-side training provided): list [{date, type, hours, content, instructor, attended_staff_count}]. Display-only |
| `safety_health_info_payload` | JSONB | Yes | NULL |  | 安全衛生情報: list [{date, type (HEALTH_CHECK/SAFETY_TRAINING/...), content, applicable_staff_count}]. Display-only |
| `personal_info_protection_measures` | TEXT | Yes | NULL |  | 個人情報保護措置 — Mô tả biện pháp bảo vệ thông tin cá nhân của Staff khi tiếp nhận tại SAKI (text, không list) |
| `dispatch_period_start_actual` | DATE | Yes | NULL |  | Ngày SAKI thực sự bắt đầu tiếp nhận Staff (có thể khác `t_contract.start_date` nếu có delay) |
| `dispatch_period_end_actual` | DATE | Yes | NULL |  | Ngày SAKI thực sự kết thúc tiếp nhận Staff |
| `notes` | TEXT | Yes | NULL |  | Ghi chú tự do |
| `last_updated_for_ledger_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm cuối cập nhật phục vụ ledger |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (KHÔNG dùng trước retention 3 năm) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_saki_ledger_manual: (contract_no, staff_id) WHERE deleted_at IS NULL — 1 row per (contract, staff)
- **INDEX** idx_saki_ledger_contract: (contract_no)
- **INDEX** idx_saki_ledger_staff: (staff_id, contract_no)
- **INDEX** idx_saki_ledger_moto_tenant: (moto_tenant_id, contract_no)
- **INDEX** idx_saki_ledger_company_time: (company_id, last_updated_for_ledger_at DESC)
- **CHECK** chk_saki_ledger_period_consistency: dispatch_period_end_actual IS NULL OR dispatch_period_start_actual IS NULL OR dispatch_period_end_actual >= dispatch_period_start_actual
- **Trigger** trg_saki_ledger_updated_at: BEFORE UPDATE → fn_set_updated_at()

---

### `t_saki_company_scheduled_change` (Thay đổi Company Master đã lên lịch — 適用日予約)

> Lưu MỘT thay đổi `mst_saki_company` đã lên lịch hiệu lực tương lai (e-staffing `適用日を予約する`). Batch hằng ngày (họ D16) tới `effective_date` thì apply `payload` vào `mst_saki_company`, set `status='APPLIED'`, ghi before/after vào `t_saki_audit_log` (= 変更履歴参照). Tối đa 1 PENDING/công ty. History dùng audit-log, KHÔNG versioning master row (D14). "Áp ngay" = update in-place thường, không tạo dòng ở đây. `lock_version` theo D18.2.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | **Logical FK**, IDX | Bảo mật Global Scope (SAKI) |
| `company_id` | BIGINT | No |  | FK, IDX | Công ty bị thay đổi → `mst_saki_company(id)` |
| `payload` | JSONB | No |  |  | Bộ giá trị company-master mới sẽ áp dụng (snapshot field đã sửa). Apply-only, không filter từng field — D3.3 JSONB OK |
| `effective_date` | DATE | No |  | IDX | Ngày thay đổi có hiệu lực |
| `status` | VARCHAR(20) | No | 'PENDING' | IDX | PENDING, APPLIED, CANCELLED |
| `applied_at` | TIMESTAMP | Yes | NULL |  | Thời điểm batch áp dụng |
| `requested_by` | BIGINT | Yes | NULL | FK, IDX | User đặt lịch → `mst_saki_user(id)` |
| `lock_version` | INTEGER | No | 0 |  | Optimistic concurrency token (D18.2) — admin có thể sửa/huỷ trước ngày apply |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID người tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID người cập nhật bản ghi |
- **PK**: (id)
- **FK** fk_saki_company_sched_company: (company_id) → mst_saki_company (id)
- **FK** fk_saki_company_sched_user: (requested_by) → mst_saki_user (id)
- **CHECK** chk_saki_company_sched_status: status IN ('PENDING', 'APPLIED', 'CANCELLED')
- **UNIQUE (Partial)** uq_saki_company_scheduled_pending: (company_id) WHERE status = 'PENDING' AND deleted_at IS NULL
- **INDEX (Partial)** idx_saki_company_sched_batch: (status, effective_date) WHERE deleted_at IS NULL
- **RLS policy** — p_saki_company_sched: USING (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint) WITH CHECK (tenant_id = current_setting('app.current_saki_tenant_id', true)::bigint)
- **Trigger** trg_saki_company_sched_updated_at: BEFORE UPDATE → fn_set_updated_at()

---