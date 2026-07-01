# PLATFORM SCHEMA

## NHÓM 1: Quản trị Hệ thống Platform (IAM & Auth)

### `mst_platform_permission` (Danh mục Quyền quản trị Platform)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `permission_key` | VARCHAR(100) | No |  | **UK** | Mã quyền (VD: `platform.tenant.create`, `platform.backup`) |
| `parent_permission_id` | BIGINT | Yes | NULL | FK, IDX | Quyền cha để dựng cây menu/chức năng |
| `permission_type` | VARCHAR(20) | No | 'ACTION' | IDX | MENU, ACTION, DATA_SCOPE, LOGIN_ACCESS |
| `display_order` | INTEGER | No | 0 |  | Thứ tự hiển thị khi import/dựng màn hình phân quyền |
| `description` | VARCHAR(255) | Yes | NULL |  | Mô tả quyền |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `mst_platform_role` (Vai trò quản trị viên Platform)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `role_name` | VARCHAR(100) | No |  | Partial UK | Tên vai trò (VD: Super Admin, Operator) |
| `is_system` | SMALLINT | No | 0 |  | 1: Mặc định không được xóa |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_platform_role_name: (role_name) WHERE deleted_at IS NULL

### `mst_platform_permission_role` (Pivot Phân quyền Platform)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `role_id` | BIGINT | No |  | **PK**, FK | Tham chiếu `mst_platform_role` |
| `permission_id` | BIGINT | No |  | **PK**, FK | Tham chiếu `mst_platform_permission` |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `mst_platform_user` (Tài khoản Quản trị viên Carima/JustFine)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `login_id` | VARCHAR(100) | No |  | **UK** | Tên đăng nhập Admin |
| `password_hash` | VARCHAR(255) | Yes | NULL |  | Mật khẩu (Có thể Null nếu dùng SSO) |
| `role_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_platform_role` |
| `full_name` | VARCHAR(100) | No |  |  | Tên quản trị viên |
| `email` | VARCHAR(255) | No |  | **UK** | Email liên hệ |
| `auth_provider` | VARCHAR(50) | No | 'LOCAL' |  | LOCAL, AZURE_AD, OKTA (Chuẩn bị cho SSO) |
| `sso_id` | VARCHAR(255) | Yes | NULL | IDX | ID định danh từ hệ thống SSO (Chuẩn bị cho SSO) |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Blocked |
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

### `tmp_platform_pwd_reset_token` (Token quên mật khẩu Platform Admin)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_platform_user` |
| `token_hash` | VARCHAR(255) | No |  | **UK** | Token mã hóa |
| `expires_at` | TIMESTAMP | No |  | IDX | Hạn sử dụng token |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token đã được sử dụng |
| `revoked_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm token bị thu hồi/hủy |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, USED, EXPIRED, REVOKED |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối audit/log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |

### `t_platform_login_history` (Lịch sử đăng nhập Platform Admin)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `user_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu `mst_platform_user`; NULL nếu login_id không tồn tại |
| `login_id` | VARCHAR(100) | No |  | IDX | Login ID được nhập ở màn hình đăng nhập |
| `auth_provider` | VARCHAR(50) | No | 'LOCAL' | IDX | LOCAL, AZURE_AD, OKTA |
| `result` | VARCHAR(20) | No |  | IDX | SUCCESS, FAILED, LOCKED, MFA_REQUIRED, MFA_FAILED |
| `failure_reason` | VARCHAR(100) | Yes | NULL |  | PASSWORD_MISMATCH, USER_NOT_FOUND, ACCOUNT_LOCKED, TOKEN_EXPIRED... |
| `ip_address` | VARCHAR(45) | Yes | NULL | IDX | IPv4/IPv6 của client hoặc gateway |
| `user_agent` | TEXT | Yes | NULL |  | User-Agent tại thời điểm đăng nhập |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối log/SIEM |
| `created_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm ghi nhận login attempt |

### `t_platform_auth_session` (Phiên đăng nhập / Refresh Token Platform Admin)

> BF-M01 enterprise short session policy: DB là source of truth, Redis chỉ dùng cache/denylist/rate-limit. Access token JWT TTL mặc định 15 phút; refresh token absolute lifetime mặc định 8 giờ; idle timeout mặc định 60 phút; không có remember-me.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `session_id` | UUID | No |  | **UK** | ID phiên đưa vào JWT claim để revoke/cache |
| `token_family_id` | UUID | No |  | IDX | Nhóm refresh token rotation; reuse token cũ sẽ revoke cả family |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_platform_user` |
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

**Rule:** Refresh token rotation là bắt buộc. Nếu refresh token đã `ROTATED`/`REVOKED` bị dùng lại, revoke toàn bộ `token_family_id` và ghi audit/security event.

---

### `t_platform_user_2fa` (Trạng thái xác thực 2 bước của Platform Admin)

> 2FA email-code + backup code (tham khảo 2段階認証マニュアル). Cấp Platform không có company_id. Ngưỡng cấu hình ở `cfg_platform_system_setting`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `user_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `mst_platform_user(id)` (1:1) |
| `two_factor_enabled_flg` | SMALLINT | No | 0 | IDX | 1: Đã bật 2FA |
| `email_for_2fa` | VARCHAR(255) | Yes | NULL |  | Email nhận mã xác thực |
| `auth_code_hash` | VARCHAR(255) | Yes | NULL |  | Hash mã xác thực email hiện hành |
| `auth_code_expires_at` | TIMESTAMP | Yes | NULL |  | Hết hạn mã email |
| `auth_code_attempt_count` | SMALLINT | No | 0 |  | Số lần nhập sai mã email |
| `auth_code_resend_count` | SMALLINT | No | 0 |  | Số lần gửi lại mã email |
| `auth_code_locked_until` | TIMESTAMP | Yes | NULL | IDX | Khóa luồng email-code theo thời gian (tự mở) |
| `backup_code_locked_flg` | SMALLINT | No | 0 |  | 1: Khóa luồng backup code tới khi update |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_platform_2fa_backup_code` (Backup code 2FA của Platform Admin)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `user_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_platform_user(id)` |
| `code_hash` | VARCHAR(255) | No |  | IDX | Hash của backup code |
| `used_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm code đã dùng (NULL = còn hiệu lực) |
| `generation` | INTEGER | No | 1 |  | Phiên bản bộ backup code |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

## NHÓM 2: Tenant Registry & Configuration (Multi-tenant)

### `mst_tenant_registry` (Sổ Đăng Ký Tenant - Bảng quan trọng nhất) [M09 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính (Chính là Global tenant_id) |
| `tenant_code` | VARCHAR(50) | No |  | **UK** | Mã định danh tenant (Dùng trên URL) |
| `tenant_type` | VARCHAR(20) | No |  | IDX | MOTO hoặc SAKI |
| `company_code` | VARCHAR(50) | No |  | IDX | Mã công ty nghiệp vụ (Business Code, tránh nhầm với Local ID) |
| `company_name` | VARCHAR(255) | No |  |  | Tên công ty mua bản quyền SaaS |
| `db_schema_name` | VARCHAR(100) | No |  | **UK** | Tên schema vật lý trên PostgreSQL |
| `db_connection_secret_ref_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu `cfg_platform_secret_reference(id)` chứa path/ARN AWS Secrets Manager cho kết nối DB tenant; không lưu DB user/password/connection string trong DB |
| `contract_start_date` | DATE | No |  |  | Ngày bắt đầu dịch vụ |
| `contract_end_date` | DATE | Yes | NULL |  | Ngày hết hạn |
| `status` | SMALLINT | No | 1 | IDX | 0: INACTIVE, 1: ACTIVE, 2: PROVISIONING, 3: PROVISION_FAILED, 4: SUSPENDED, 5: RESTORING, 6: TERMINATED |
| `health_status` | SMALLINT | No | 1 | IDX | 1=GREEN / 2=YELLOW / 3=RED — cached từ monitoring metric, không real-time (refresh batch); F-1402 dashboard hiển thị **[M09 NEW]** |
| `last_activity_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm cuối có hoạt động API/login từ tenant này — BR-143 cảnh báo nếu >30 ngày **[M09 NEW]** |
| `last_health_check_at` | TIMESTAMP | Yes | NULL |  | Thời điểm batch health check chạy gần nhất cho tenant này **[M09 NEW]** |
| `last_suspension_reason` | TEXT | Yes | NULL |  | Lý do lần suspend gần nhất (snapshot tại moment) — F-1405 hiển thị cho Platform Admin **[M09 NEW]** |
| `reactivation_count` | INTEGER | No | 0 |  | Số lần đã reactivate (audit insight về tenant không ổn định) **[M09 NEW]** |
| `sla_tier` | SMALLINT | No | 1 | IDX | 1=BRONZE / 2=SILVER / 3=GOLD / 4=PLATINUM — BR-170 SLA per tenant tier **[M09 NEW]** |
| `oem_provider_id` | BIGINT | Yes | NULL | IDX | Trỏ về tenant Provider (OEM) nếu tenant này thuộc gói White-label. NULL = tenant trực tiếp Carima. Future-proof cho M09 P1 OEM cluster **[M09 NEW]** |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE** uq_tenant_registry: (tenant_type, company_code)
- **CHECK** chk_tenant_health_status: health_status IN (1, 2, 3)
- **CHECK** chk_tenant_sla_tier: sla_tier IN (1, 2, 3, 4)
- **CHECK** chk_tenant_reactivation_count: reactivation_count >= 0
- **CHECK** chk_tenant_oem_self_ref: oem_provider_id IS NULL OR oem_provider_id <> id (Provider không thể tự trỏ chính mình)

### `cfg_platform_tenant_setting` (Cấu hình hệ thống riêng của từng Tenant)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, **UK** | Tham chiếu `mst_tenant_registry` (1:1) |
| `custom_domain` | VARCHAR(255) | Yes | NULL | **UK** | Tên miền riêng (nếu có) |
| `logo_url` | VARCHAR(255) | Yes | NULL |  | Logo công ty |
| `theme_color` | VARCHAR(20) | Yes | NULL |  | Mã màu giao diện |
| `enable_sso_flg` | SMALLINT | No | 0 |  | 1: Cho phép Tenant này dùng SSO |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `cfg_platform_tenant_feature` (Bật/tắt module tính năng theo Tenant)

> Module enablement, KHÔNG phải paid plan/subscription (theo BF-M01 action list). Mỗi tenant bật/tắt từng feature key.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_tenant_registry(id)` |
| `feature_key` | VARCHAR(100) | No |  | Composite Partial UK | Mã module/tính năng (VD: ATTENDANCE, BILLING, SHIFT, INTRODUCTION_DISPATCH) |
| `enabled_flg` | SMALLINT | No | 0 | IDX | 1: Bật module cho tenant |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_platform_tenant_feature: (tenant_id, feature_key) WHERE deleted_at IS NULL

### `cfg_platform_secret_reference` (Tham chiếu Secret - Không lưu raw password)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu `mst_tenant_registry`; NULL nếu là secret cấp Platform/global |
| `secret_type` | VARCHAR(50) | No |  | Composite UK | Loại secret: TENANT_DB_CONNECTION, TENANT_TECHNICAL_ACCOUNT, SSO_IDP, EXTERNAL_API, MAIL, STORAGE |
| `secret_path` | VARCHAR(500) | No |  | **UK** | Path/name logic trong AWS Secrets Manager |
| `aws_secret_arn` | VARCHAR(500) | Yes | NULL | **UK** | ARN trỏ tới AWS Secrets Manager nếu đã resolve được ARN |
| `rotation_status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, ROTATING, ROTATION_FAILED, REVOKED |
| `last_rotated_at` | TIMESTAMP | Yes | NULL |  | Thời điểm rotate secret gần nhất |
| `status` | SMALLINT | No | 1 |  | Trạng thái sử dụng |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **UNIQUE (Partial)** uq_platform_secret_reference_tenant_type: (tenant_id, secret_type) WHERE tenant_id IS NOT NULL AND deleted_at IS NULL
- **UNIQUE (Partial)** uq_platform_secret_reference_global_type: (secret_type) WHERE tenant_id IS NULL AND deleted_at IS NULL
- **Rule**: Bảng này chỉ lưu metadata/path/ARN. Secret value như DB host, database, user, password, connection string, access key, private key, SSO client secret không được lưu trong PostgreSQL.

### `cfg_platform_api_key` (Quản lý API Key tích hợp cho Tenant)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_tenant_registry` |
| `key_prefix` | VARCHAR(20) | No |  | **UK** | 8 ký tự đầu để nhận diện Key |
| `key_hash` | VARCHAR(255) | No |  |  | Hash của API Key thực (Bcrypt/SHA256) |
| `ip_allowlist` | TEXT | Yes | NULL |  | Danh sách IP được phép gọi API |
| `expires_at` | TIMESTAMP | Yes | NULL |  | Ngày hết hạn |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Revoked |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_tenant_provision_log` (Nhật ký cấp phát/tạo mới Schema Tenant) [M09 Modified]

> **M09 enhancement:** Thêm tracking đa-bước (step_sequence) cho workflow multi-step (CREATE_SCHEMA → RUN_MIGRATION → ACTIVATE), thời lượng dự kiến/thực tế, và retry state cho FAILED action có thể tự retry.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_tenant_registry` |
| `platform_user_id` | BIGINT | Yes | NULL | FK | Admin thực hiện (Null nếu System tự tạo) |
| `action` | VARCHAR(50) | No |  | IDX | CREATE_SCHEMA, RUN_MIGRATION, SUSPEND, RESTORE, ACTIVATE, TERMINATE |
| `step_sequence` | SMALLINT | No | 1 |  | Bước thứ N trong workflow (1, 2, 3...). Workflow có thể có nhiều bước/bản ghi. 1 = đơn-bước **[M09 NEW]** |
| `status` | VARCHAR(20) | No |  | IDX | SUCCESS, FAILED, RUNNING |
| `old_status` | SMALLINT | Yes | NULL | IDX | Trạng thái tenant trước khi thao tác |
| `new_status` | SMALLINT | Yes | NULL | IDX | Trạng thái tenant sau khi thao tác |
| `expected_duration_ms` | INTEGER | Yes | NULL |  | Thời lượng dự kiến (từ baseline historical) — phục vụ SLA check **[M09 NEW]** |
| `actual_duration_ms` | INTEGER | Yes | NULL |  | Thời lượng thực tế khi RUNNING → SUCCESS/FAILED **[M09 NEW]** |
| `rollback_status` | SMALLINT | Yes | NULL | IDX | NULL=không cần / 1=REQUESTED / 2=IN_PROGRESS / 3=COMPLETED / 4=FAILED — phục vụ rollback step thất bại **[M09 NEW]** |
| `retry_count` | INTEGER | No | 0 |  | Số lần auto-retry đã thực hiện (max 3 — BR-187 áp dụng tương tự) **[M09 NEW]** |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Lịch retry tiếp theo (exponential backoff). NULL=không retry **[M09 NEW]** |
| `reason` | TEXT | Yes | NULL |  | Lý do suspend/restore/terminate hoặc ghi chú vận hành |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID request từ API Gateway |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID tương quan để nối log/SIEM |
| `actor_type` | VARCHAR(20) | No | 'PLATFORM_USER' | IDX | PLATFORM_USER, SYSTEM |
| `actor_id` | BIGINT | Yes | NULL | FK, IDX | Logical FK → `mst_platform_user(id)` khi actor_type='PLATFORM_USER'; NULL khi actor_type='SYSTEM'. CHECK chk_provision_log_actor_consistency enforce |
| `error_log` | TEXT | Yes | NULL |  | Chi tiết lỗi nếu hỏng |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **INDEX** idx_provision_log_retry: (status, next_retry_at) WHERE status = 'FAILED' AND next_retry_at IS NOT NULL — batch worker scan
- **FK** fk_provision_log_actor: (actor_id) → mst_platform_user(id) — Logical FK chỉ áp dụng khi actor_type='PLATFORM_USER'
- **CHECK** chk_provision_log_status: status IN ('SUCCESS', 'FAILED', 'RUNNING')
- **CHECK** chk_provision_log_actor_type: actor_type IN ('PLATFORM_USER', 'SYSTEM')
- **CHECK** chk_provision_log_actor_consistency: (actor_type = 'SYSTEM' AND actor_id IS NULL) OR (actor_type = 'PLATFORM_USER' AND actor_id IS NOT NULL)
- **CHECK** chk_provision_log_step_seq: step_sequence >= 1
- **CHECK** chk_provision_log_rollback_status: rollback_status IS NULL OR rollback_status IN (1, 2, 3, 4)
- **CHECK** chk_provision_log_retry_count: retry_count >= 0 AND retry_count <= 3
- **CHECK** chk_provision_log_durations: (expected_duration_ms IS NULL OR expected_duration_ms > 0) AND (actual_duration_ms IS NULL OR actual_duration_ms >= 0)

### `t_tenant_lifecycle_history` (Lịch sử Thay đổi Trạng thái Tenant — Tenant Status Audit) [M09 NEW]

> Tách riêng khỏi `t_tenant_provision_log` vì mục đích khác: `provision_log` ghi nhật ký **thao tác** (action-centric); bảng này ghi nhật ký **chuyển trạng thái** (state-centric). 1 thao tác provision_log có thể tạo 0 hoặc 1 dòng lifecycle_history (chỉ khi status thực sự đổi). Phục vụ F-1405/F-1408 timeline view và phân tích `duration_in_previous_state` (vd: tenant ở SUSPENDED bao lâu trước khi RESTORE).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, IDX | Tham chiếu `mst_tenant_registry(id)` |
| `old_status` | SMALLINT | Yes | NULL |  | Trạng thái trước (NULL nếu là dòng đầu tiên — khi tenant mới tạo) |
| `new_status` | SMALLINT | No |  | IDX | Trạng thái sau |
| `transition_reason` | TEXT | Yes | NULL |  | Lý do chuyển (snapshot từ user input hoặc system rule) |
| `transition_type` | SMALLINT | No |  | IDX | 1=AUTO_PROVISION (system) / 2=MANUAL_ADMIN (Platform Admin click) / 3=AUTO_SUSPEND (vd: contract expired) / 4=AUTO_FAIL (provision failed) |
| `actor_type` | VARCHAR(20) | No | 'SYSTEM' |  | PLATFORM_USER, SYSTEM |
| `actor_id` | BIGINT | Yes | NULL | IDX | ID Platform Admin nếu MANUAL; NULL nếu SYSTEM |
| `duration_in_previous_state_seconds` | BIGINT | Yes | NULL |  | Tenant ở `old_status` bao lâu (giây) trước khi chuyển sang `new_status`. NULL với dòng đầu tiên. Để phân tích tenant churn/instability |
| `transitioned_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm chuyển trạng thái (event time) |
| `provision_log_id` | BIGINT | Yes | NULL | FK | Trỏ về dòng `t_tenant_provision_log` triggered transition này (nếu có) — link để đào sâu thao tác |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | Trace HTTP |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | Trace workflow |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật (sẽ không cập nhật — immutable) |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (chỉ Platform Admin sau retention) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_lifecycle_tenant_time: (tenant_id, transitioned_at DESC)
- **INDEX** idx_lifecycle_new_status: (new_status, transitioned_at DESC)
- **FK** fk_lifecycle_tenant: (tenant_id) → mst_tenant_registry(id)
- **FK** fk_lifecycle_provision_log: (provision_log_id) → t_tenant_provision_log(id)
- **CHECK** chk_lifecycle_transition_type: transition_type IN (1, 2, 3, 4)
- **CHECK** chk_lifecycle_old_new_different: old_status IS NULL OR old_status <> new_status (chỉ ghi nhận khi thực sự đổi)
- **CHECK** chk_lifecycle_duration: duration_in_previous_state_seconds IS NULL OR duration_in_previous_state_seconds >= 0
- **CHECK** chk_lifecycle_actor_consistency: (actor_type = 'SYSTEM' AND actor_id IS NULL) OR (actor_type = 'PLATFORM_USER' AND actor_id IS NOT NULL)
- **Trigger** trg_lifecycle_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `mst_system_config` (Tham số Hệ thống — システム設定マスタ)

> Platform Admin quản lý ~30 tham số toàn hệ thống (BR-1141→BR-1144). Một số là tham số pháp luật (3年抵触日, cooling_period) — `is_tenant_overridable=0` enforce tại DB; còn lại tenant có thể ghi đè trong `cfg_*` tables. Cache invalidate ngay sau update (BR-1144).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `config_group` | VARCHAR(50) | No |  | IDX | Nhóm cấu hình: ATTENDANCE, CONTRACT, APPROVAL, SYSTEM, INVOICE, ... |
| `config_key` | VARCHAR(100) | No |  | **UK** | Khóa cấu hình (VD: `overtime_monthly_limit_hours`, `cooling_period_months`) — duy nhất toàn hệ thống |
| `config_value` | TEXT | No |  |  | Giá trị hiện tại (string-encoded — application parse theo `config_type`) |
| `default_value` | TEXT | Yes | NULL |  | Giá trị mặc định (dùng khi tenant không ghi đè — BR-1142) |
| `config_type` | VARCHAR(20) | No |  |  | Kiểu dữ liệu: STRING / INTEGER / BOOLEAN / JSON / DATE / TIME — BR-1143 validate |
| `is_tenant_overridable` | SMALLINT | No | 1 | IDX | 1=tenant có thể ghi đè / 0=tham số pháp luật bắt buộc (BR-1141) |
| `description` | TEXT | Yes | NULL |  | Mô tả mục đích tham số |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Inactive |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_system_config_type: config_type IN ('STRING', 'INTEGER', 'BOOLEAN', 'JSON', 'DATE', 'TIME')
- **CHECK** chk_system_config_overridable: is_tenant_overridable IN (0, 1)
- **Trigger** trg_system_config_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_system_config_history` (Lịch sử Phiên bản Tham số Hệ thống — System Config Versioning) [M09 NEW]

> BR-152: System config có history version để xem lịch sử và rollback. Mỗi lần Platform Admin update `mst_system_config.config_value`, INSERT 1 dòng vào đây (snapshot old_value→new_value). Có thể rollback bằng cách copy `old_value` → `mst_system_config.config_value`. Pure-immutable insert-only.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `config_id` | BIGINT | No |  | FK, IDX | Trỏ về `mst_system_config(id)` |
| `version_number` | INTEGER | No |  |  | Phiên bản tăng dần per config_id (1, 2, 3...) |
| `old_value` | TEXT | Yes | NULL |  | Giá trị cũ trước thay đổi (NULL với version đầu tiên — khi config được tạo) |
| `new_value` | TEXT | No |  |  | Giá trị mới sau thay đổi |
| `change_reason` | TEXT | Yes | NULL |  | Lý do thay đổi (Platform Admin nhập) — bắt buộc khi `is_tenant_overridable=0` (config pháp lý) — enforce ở application layer |
| `is_rollback` | SMALLINT | No | 0 |  | 1=lần thay đổi này là rollback về version cũ; 0=thay đổi forward bình thường |
| `rollback_target_version` | INTEGER | Yes | NULL |  | Nếu is_rollback=1, version số mà ta rollback về (vd: hiện tại đang ở v5, rollback về v3 → rollback_target_version=3) |
| `changed_by_platform_user_id` | BIGINT | No |  | FK, IDX | Platform Admin thực hiện thay đổi |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | Trace HTTP |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | Trace workflow |
| `created_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm thay đổi (= version creation time) |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật (sẽ không cập nhật — immutable) |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_config_history_version: (config_id, version_number)
- **INDEX** idx_config_history_config_time: (config_id, created_at DESC)
- **FK** fk_config_history_config: (config_id) → mst_system_config(id)
- **FK** fk_config_history_user: (changed_by_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_config_history_version_pos: version_number >= 1
- **CHECK** chk_config_history_is_rollback: is_rollback IN (0, 1)
- **CHECK** chk_config_history_rollback_target: (is_rollback = 0 AND rollback_target_version IS NULL) OR (is_rollback = 1 AND rollback_target_version IS NOT NULL AND rollback_target_version < version_number)
- **Trigger** trg_config_history_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_master_import_history` (Lịch sử Import Hàng loạt Master Data — アップロード結果)

> Mỗi lần Platform Admin upload file CSV/XLSX để import master data (BR-1121: ≤5000 rows). All-or-nothing (BR-1122). File gốc lưu S3 ≥1 năm (BR-1125). Status PROCESSING → DONE/FAILED.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `master_type` | VARCHAR(20) | No |  | IDX | Loại master data: MST-IND / MST-JOB / MST-SKL / MST-QUA / MST-CFG |
| `platform_user_id` | BIGINT | No |  | FK, IDX | Platform Admin thực hiện upload |
| `file_name` | VARCHAR(255) | No |  |  | Tên file gốc upload |
| `s3_storage_key` | VARCHAR(500) | No |  |  | Đường dẫn S3 lưu file gốc (BR-1125: ≥1 năm) |
| `file_hash` | VARCHAR(64) | Yes | NULL |  | SHA-256 file gốc — chống giả mạo |
| `total_rows` | INTEGER | No | 0 |  | Tổng số dòng data trong file (không tính header) |
| `success_rows` | INTEGER | No | 0 |  | Số dòng thành công (chỉ >0 khi status=DONE — BR-1122 all-or-nothing) |
| `failed_rows` | INTEGER | No | 0 |  | Số dòng lỗi (>0 thì status=FAILED, toàn bộ rollback) |
| `status` | VARCHAR(20) | No | 'PROCESSING' | IDX | PROCESSING / VALIDATING / DONE / FAILED |
| `error_detail_payload` | JSONB | Yes | NULL |  | Chi tiết lỗi từng dòng (display-only, không filter — D3.3): `[{row_number, column, error_code, error_message, raw_value}]` |
| `started_at` | TIMESTAMP | No | NOW() |  | Thời điểm bắt đầu xử lý |
| `completed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm hoàn thành (DONE/FAILED) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **FK** fk_master_import_user: (platform_user_id) → mst_platform_user(id)
- **CHECK** chk_master_import_status: status IN ('PROCESSING', 'VALIDATING', 'DONE', 'FAILED')
- **CHECK** chk_master_import_type: master_type IN ('MST-IND', 'MST-JOB', 'MST-SKL', 'MST-QUA', 'MST-CFG')
- **CHECK** chk_master_import_rows: total_rows >= 0 AND success_rows >= 0 AND failed_rows >= 0 AND success_rows + failed_rows <= total_rows
- **Trigger** trg_master_import_history_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `mst_notification_template` (Template Thông báo — 通知テンプレートマスタ) [M08 NEW]

> Định nghĩa nội dung mẫu (subject + body Mustache) cho 30+ loại Notification (NTF-C01...NTF-N02). Platform quản lý tập trung theo BR-131/133 — Tenant KHÔNG được sửa nội dung template, chỉ cấu hình kênh (email/in-app on/off) thông qua bảng `cfg_*_alarm_setting`. Là bảng dictionary thuần (no tenant scope), retention permanent.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `notification_type_code` | VARCHAR(20) | No |  | **UK**, IDX | Mã loại thông báo (NTF-C01, NTF-A04, NTF-I06, NTF-S03, NTF-N02 ...) — BR-103 |
| `category` | SMALLINT | No |  | IDX | 1=CONTRACT 契約 / 2=ATTENDANCE 勤怠 / 3=INVOICE 請求 / 4=SYSTEM システム / 5=ANNOUNCEMENT お知らせ |
| `priority` | SMALLINT | No | 1 | IDX | 1=NORMAL 通常 / 2=IMPORTANT 重要 / 3=URGENT 緊急 (BR-102) |
| `target_audience` | SMALLINT | No |  |  | 1=SAKI_USER / 2=MOTO_USER / 3=STAFF / 4=PLATFORM_ADMIN (ai sẽ là người nhận) |
| `subject_template_ja` | VARCHAR(500) | No |  |  | Subject Mustache JP — VD: "【契約承認依頼】{{contract_no}} の承認をお願いします" — BR-131 |
| `body_template_ja` | TEXT | No |  |  | Body Mustache JP — placeholder {{}} cho variables |
| `available_variables_payload` | JSONB | Yes | NULL |  | Schema variables hợp lệ cho render (display/doc only — không filter): list {key, type, required, example}. ≥5 fields/items, không JOIN |
| `default_action_url_template` | VARCHAR(500) | Yes | NULL |  | URL Mustache template cho deep-link (vd: "/contract/{{contract_id}}") |
| `is_email_enabled_default` | SMALLINT | No | 1 |  | Mặc định gửi email khi tenant chưa cấu hình |
| `is_inapp_enabled_default` | SMALLINT | No | 1 |  | Mặc định hiện in-app |
| `is_legal_alert` | SMALLINT | No | 0 | IDX | 1=Cảnh báo pháp lý (36協定/抵触日/treatment) — BR-101/BR-126: Tenant KHÔNG được tắt in-app kênh, app layer enforce |
| `is_recipient_user_configurable` | SMALLINT | No | 1 |  | 1=Tenant có thể cấu hình recipient qua `cfg_*_alarm_recipient` / 0=System định nghĩa cố định (NTF-S01...S05) |
| `template_version` | INT | No | 1 |  | Phiên bản template (tăng khi Platform Admin cập nhật) — không phải versioning theo D4, chỉ tracking changelog |
| `status` | SMALLINT | No | 1 | IDX | 1=Active / 0=Deprecated (giữ lại cho audit lookup nhưng không dùng cho noti mới) |
| `description` | TEXT | Yes | NULL |  | Mô tả mục đích — admin documentation |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_notif_template_code: (notification_type_code)
- **INDEX** idx_notif_template_category: (category)
- **INDEX** idx_notif_template_legal: (is_legal_alert) WHERE is_legal_alert = 1
- **INDEX** idx_notif_template_status: (status) WHERE status = 1
- **CHECK** chk_notif_template_code_format: notification_type_code ~ '^NTF-[A-Z]{1}[0-9]{2}$'
- **CHECK** chk_notif_template_category: category IN (1, 2, 3, 4, 5)
- **CHECK** chk_notif_template_priority: priority IN (1, 2, 3)
- **CHECK** chk_notif_template_target_audience: target_audience IN (1, 2, 3, 4)
- **CHECK** chk_notif_template_flags: is_email_enabled_default IN (0, 1) AND is_inapp_enabled_default IN (0, 1) AND is_legal_alert IN (0, 1) AND is_recipient_user_configurable IN (0, 1)
- **CHECK** chk_notif_template_status_val: status IN (0, 1)
- **CHECK** chk_notif_template_legal_inapp: is_legal_alert = 0 OR is_inapp_enabled_default = 1 (Cảnh báo pháp lý bắt buộc default in-app = 1)
- **Trigger** trg_notif_template_updated_at: BEFORE UPDATE → fn_set_updated_at()

---

## NHÓM 3: Bản đồ Quan hệ MOTO - SAKI (Cross-Tenant Access Control)

### `mst_moto_saki_relation` (Sổ Đăng Ký Quan Hệ Kinh Doanh MOTO-SAKI)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `moto_tenant_id` | BIGINT | No |  | FK, IDX | Tham chiếu MOTO tenant |
| `saki_tenant_id` | BIGINT | No |  | FK, IDX | Tham chiếu SAKI tenant |
| `status` | VARCHAR(20) | No | 'ACTIVE' | IDX | ACTIVE, TERMINATED |
| `established_date` | DATE | No |  |  | Ngày thiết lập quan hệ |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE (Partial)** uq_moto_saki_pair: (moto_tenant_id, saki_tenant_id) WHERE deleted_at IS NULL (Note: Ưu tiên Restore bản ghi cũ ở tầng Application)
- **INDEX**: idx_relation_moto: (moto_tenant_id)
- **INDEX**: idx_relation_saki: (saki_tenant_id)
- **FK** fk_moto_saki_moto_tenant: (moto_tenant_id) → mst_tenant_registry(id)
- **FK** fk_moto_saki_saki_tenant: (saki_tenant_id) → mst_tenant_registry(id)
- **CHECK** chk_moto_saki_status: status IN ('ACTIVE', 'TERMINATED')
- **CHECK** chk_moto_saki_different_tenants: moto_tenant_id <> saki_tenant_id (MOTO không thể là SAKI của chính mình)
- **Trigger** trg_moto_saki_relation_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_moto_saki_relation_history` (Lịch sử Đóng/Mở quan hệ)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `relation_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_moto_saki_relation |
| `platform_user_id` | BIGINT | Yes | NULL | FK | Admin thực hiện (nếu có) |
| `action` | VARCHAR(50) | No |  |  | LINKED, SUSPENDED, TERMINATED, RESTORED (Đường chuẩn), RELINKED_NEW_RECORD (Ngoại lệ) |
| `reason` | TEXT | Yes | NULL |  | Lý do thay đổi (Bắt buộc phải nhập Audit Reason nếu action là RELINKED_NEW_RECORD. Dùng TEXT để lưu giải trình dài) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **FK** fk_relation_history_rel: (relation_id) → mst_moto_saki_relation (id)
- **INDEX** idx_relation_history_rel: (relation_id)
- **CHECK Constraint:** chk_relink_reason: CHECK (action <> 'RELINKED_NEW_RECORD' OR (reason IS NOT NULL AND trim(reason) <> ''))

---

## NHÓM 4: Platform Operations, Event Bus & Log (Lõi Kỹ thuật)

### `cfg_platform_system_setting` (Cấu hình lõi Platform)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `setting_key` | VARCHAR(100) | No |  | **UK** | Khóa cấu hình (VD: `MAINTENANCE_MODE`) |
| `setting_value` | TEXT | No |  |  | Giá trị (VD: `TRUE`, `FALSE`) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_platform_notification_history` (Trung tâm お知らせ Broadcast — Announcement Hub) [M08 Modified]

> Bảng này CHỈ phục vụ お知らせ broadcast (Announcement) Platform → Tenants/Users (BR-117/121/138). Notification driven theo event nghiệp vụ (NTF-C01 hợp đồng, NTF-A04 chấm công, ...) dùng `t_*_notification_log` + `t_*_inbox_notification` ở tenant schema. Pattern Read-receipt qua `t_*_notification_read` được giữ cho broadcast (1 record, N user xem) — không fan-out để tiết kiệm disk.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `notification_kind` | SMALLINT | No | 2 | IDX | 1=USER_NTF (legacy, không dùng cho noti mới — đã chuyển sang t_*_notification_log) / 2=ANNOUNCEMENT お知らせ — phân biệt rõ data path **[M08 NEW]** |
| `category` | SMALLINT | No | 5 | IDX | 5=ANNOUNCEMENT mặc định; có thể 1-4 nếu announcement gắn category cụ thể **[M08 NEW]** |
| `priority` | SMALLINT | No | 1 | IDX | 1=NORMAL / 2=IMPORTANT (banner) / 3=URGENT (modal popup) — BR-102 **[M08 NEW]** |
| `status` | SMALLINT | No | 1 | IDX | 1=DRAFT / 2=SCHEDULED / 3=PUBLISHED / 4=EXPIRED / 5=WITHDRAWN (BR-115/116/121) **[M08 NEW]** |
| `target_scope` | SMALLINT | No | 1 | IDX | 1=ALL_PLATFORM / 2=SAKI_ONLY / 3=MOTO_ONLY / 4=SPECIFIC_TENANTS **[M08 NEW]** |
| `sender_tenant_id` | BIGINT | Yes | NULL | IDX | Null nếu System/Platform Admin gửi |
| `target_tenant_id` | BIGINT | Yes | NULL | IDX | Null khi target_scope ∈ (1,2,3) hoặc multi-target (dùng payload bên dưới) |
| `target_tenant_ids_payload` | JSONB | Yes | NULL |  | List target tenant IDs khi target_scope=4 — display-only, không filter individual, ≥5 IDs có thể → JSONB OK **[M08 NEW]** |
| `target_type` | VARCHAR(20) | No |  |  | SAKI_USER, MOTO_USER, STAFF, ALL (kế thừa) |
| `title` | VARCHAR(255) | No |  |  | Tiêu đề お知らせ |
| `content` | TEXT | No |  |  | Nội dung (rich text/markdown — BR-114) |
| `link_url` | VARCHAR(500) | Yes | NULL |  | Link chuyển hướng khi click (single primary action) |
| `attachments_payload` | JSONB | Yes | NULL |  | Đính kèm tối đa 5 file (BR-114): list {s3_key, filename, size, mime_type} — display-only **[M08 NEW]** |
| `send_email_flg` | SMALLINT | No | 0 |  | 1=Gửi kèm email broadcast tới recipients **[M08 NEW]** |
| `publish_at` | TIMESTAMP | Yes | NULL | IDX | Scheduled publish — batch job tự transition DRAFT→PUBLISHED khi tới giờ (BR-120) **[M08 NEW]** |
| `expires_at` | TIMESTAMP | Yes | NULL | IDX | Tự động transition PUBLISHED→EXPIRED (BR-121) **[M08 NEW]** |
| `published_at` | TIMESTAMP | Yes | NULL |  | Thời điểm thực sự published (audit) **[M08 NEW]** |
| `withdrawn_at` | TIMESTAMP | Yes | NULL |  | Thời điểm thu hồi nếu có (BR-116) **[M08 NEW]** |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_notif_hist_status_pub: (status, publish_at) — batch tìm bản DRAFT/SCHEDULED đến giờ publish
- **INDEX** idx_notif_hist_status_exp: (status, expires_at) — batch tìm bản PUBLISHED đã expired
- **INDEX** idx_notif_hist_scope_status: (target_scope, status) — list view filter
- **CHECK** chk_notif_hist_kind: notification_kind IN (1, 2)
- **CHECK** chk_notif_hist_category: category IN (1, 2, 3, 4, 5)
- **CHECK** chk_notif_hist_priority: priority IN (1, 2, 3)
- **CHECK** chk_notif_hist_status: status IN (1, 2, 3, 4, 5)
- **CHECK** chk_notif_hist_target_scope: target_scope IN (1, 2, 3, 4)
- **CHECK** chk_notif_hist_send_email: send_email_flg IN (0, 1)
- **CHECK** chk_notif_hist_target_consistency: (target_scope IN (1, 2, 3) AND target_tenant_id IS NULL) OR (target_scope = 4 AND (target_tenant_id IS NOT NULL OR target_tenant_ids_payload IS NOT NULL))
- **CHECK** chk_notif_hist_expires_after_publish: expires_at IS NULL OR publish_at IS NULL OR expires_at > publish_at
- **CHECK** chk_notif_hist_withdrawn_status: withdrawn_at IS NULL OR status = 5
- **Trigger** trg_notif_hist_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_platform_backup_job` (Giám sát Lịch sử sao lưu DB/S3 theo NFR) [M09 Modified]

> Thêm metadata để giám sát SLA backup (size, RTO/RPO), verification (BR-14J: backup phải verify được trước khi restore), encryption (compliance). Liên kết với `cfg_platform_backup_policy` qua `backup_policy_id` để biết job này thuộc policy nào.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `backup_policy_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ về `cfg_platform_backup_policy(id)` nếu chạy theo policy; NULL nếu manual ad-hoc backup **[M09 NEW]** |
| `backup_type` | VARCHAR(50) | No |  | IDX | FULL_DB, TENANT_SCHEMA, S3_FILES |
| `target_tenant_id` | BIGINT | Yes | NULL | IDX | Null nếu backup toàn hệ thống |
| `status` | VARCHAR(20) | No | 'RUNNING' | IDX | RUNNING, SUCCESS, FAILED |
| `incremental_flg` | SMALLINT | No | 0 |  | 1=Incremental (chỉ thay đổi từ backup trước) / 0=Full **[M09 NEW]** |
| `backup_size_mb` | BIGINT | Yes | NULL |  | Dung lượng backup (MB) — hiển thị dashboard F-1404, đo trends **[M09 NEW]** |
| `retention_days` | INTEGER | Yes | NULL |  | Số ngày giữ backup này (snapshot từ policy tại moment chạy) **[M09 NEW]** |
| `encryption_algorithm` | VARCHAR(50) | Yes | NULL |  | Thuật toán encrypt (vd: AES-256-GCM via KMS) — compliance audit **[M09 NEW]** |
| `verification_status` | SMALLINT | No | 1 | IDX | 1=PENDING / 2=PASSED (restore-test đã chạy thành công) / 3=FAILED / 4=SKIPPED (BR-14J) **[M09 NEW]** |
| `verified_at` | TIMESTAMP | Yes | NULL |  | Thời điểm verify lần cuối **[M09 NEW]** |
| `rto_minutes` | INTEGER | Yes | NULL |  | Recovery Time Objective dự kiến (từ policy) — BR-14H **[M09 NEW]** |
| `rpo_minutes` | INTEGER | Yes | NULL |  | Recovery Point Objective dự kiến — BR-14H **[M09 NEW]** |
| `file_reference` | VARCHAR(500) | Yes | NULL |  | Link tham chiếu tới file backup (VD: S3 URI). BR-14G: S3 prefix theo tenant_id |
| `error_log` | TEXT | Yes | NULL |  | Lỗi nếu có |
| `platform_user_id` | BIGINT | Yes | NULL | FK | Null nếu do System tự động chạy |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_backup_job_status: status IN ('RUNNING', 'SUCCESS', 'FAILED')
- **CHECK** chk_backup_job_incremental: incremental_flg IN (0, 1)
- **CHECK** chk_backup_job_verification: verification_status IN (1, 2, 3, 4)
- **CHECK** chk_backup_job_size: backup_size_mb IS NULL OR backup_size_mb >= 0
- **CHECK** chk_backup_job_retention: retention_days IS NULL OR retention_days > 0
- **CHECK** chk_backup_job_rto_rpo: (rto_minutes IS NULL OR rto_minutes > 0) AND (rpo_minutes IS NULL OR rpo_minutes >= 0)

### `t_platform_audit_log` (Nhật ký hành động của Platform Admin) [M09 Modified]

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `actor_type` | VARCHAR(20) | No | 'ADMIN' |  | SYSTEM, ADMIN |
| `actor_id` | BIGINT | Yes | NULL | FK, IDX | Admin thao tác (Tham chiếu mst_platform_user) |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | ID của HTTP Request (từ API Gateway) |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết Cross-tenant Event / Workflow |
| `action` | VARCHAR(100) | No |  | IDX | API Endpoint (VD: SUSPEND_TENANT) |
| `entity_table` | VARCHAR(50) | No |  | IDX | Bảng bị tác động |
| `entity_id` | VARCHAR(100) | No |  | IDX | ID bị tác động |
| `severity_level` | SMALLINT | No | 1 | IDX | 1=INFO / 2=WARNING / 3=CRITICAL — filter dashboard F-1418 (BR-165) **[M09 NEW]** |
| `change_category` | SMALLINT | Yes | NULL | IDX | 1=IAM (quyền/role) / 2=BILLING / 3=OPERATIONS (suspend/restore/backup) / 4=SECURITY (config bảo mật) / 5=CONFIG (system parameter) — grouping cho audit review **[M09 NEW]** |
| `affected_tenant_count` | INTEGER | Yes | NULL |  | Số tenant chịu ảnh hưởng từ hành động này (vd: maintenance mode = N tenant). NULL nếu không liên quan tenant **[M09 NEW]** |
| `approval_status` | SMALLINT | Yes | NULL | IDX | NULL=không yêu cầu duyệt / 1=PENDING / 2=APPROVED / 3=REJECTED — cho thao tác cần dual approval (vd: restore overwrite BR-14E) **[M09 NEW]** |
| `before_data` | JSONB | Yes | NULL |  | Dữ liệu cũ |
| `after_data` | JSONB | Yes | NULL |  | Dữ liệu mới |
| `ip_address` | VARCHAR(50) | Yes | NULL |  | IP Address của người dùng |
| `user_agent` | VARCHAR(500) | Yes | NULL |  | Trình duyệt/Thiết bị |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
- **CHECK** chk_audit_severity: severity_level IN (1, 2, 3)
- **CHECK** chk_audit_change_category: change_category IS NULL OR change_category IN (1, 2, 3, 4, 5)
- **CHECK** chk_audit_affected_tenant_count: affected_tenant_count IS NULL OR affected_tenant_count >= 0
- **CHECK** chk_audit_approval_status: approval_status IS NULL OR approval_status IN (1, 2, 3)

### `t_platform_inbox_event` (Hàng đợi Nhận Event tại Trạm trung chuyển)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `source_tenant_id` | BIGINT | No | 0 | IDX | Nguồn phát (0 = SYSTEM hoặc các nguồn ngoài EXT_*) |
| `source_tenant_type` | VARCHAR(20) | No |  |  | SAKI, MOTO, SYSTEM, EXT_STRIPE, EXT_BANK, EXT_LEGACY |
| `event_type` | VARCHAR(100) | No |  | IDX | VD: SAKI_TENANT_REGISTERED |
| `aggregate_type` | VARCHAR(50) | No |  | IDX | Loại thực thể (VD: TENANT) |
| `aggregate_id` | VARCHAR(100) | No |  | IDX | ID thực thể (Nâng lên 100 char) |
| `payload` | JSONB | No |  |  | Data chi tiết |
| `idempotency_key` | VARCHAR(100) | No |  | Composite UK | ID do nguồn sinh ra |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết Workflow |
| `status` | VARCHAR(20) | No | 'PENDING' | IDX | PENDING, PROCESSING, COMPLETED, FAILED |
| `retry_count` | SMALLINT | No | 0 |  | Số lần thử lại |
| `last_error` | TEXT | Yes | NULL |  | Lỗi cuối cùng |
| `processed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm xử lý xong |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm chạy lại nếu lỗi |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
- **UNIQUE** uq_platform_inbox_idem: (source_tenant_type, source_tenant_id, idempotency_key)
- **INDEX (composite)** idx_plat_inbox_aggregate: (aggregate_type, aggregate_id, created_at)
- **INDEX (composite)** idx_plat_inbox_worker: (status, next_retry_at)

### `t_platform_outbox_event` (Hàng đợi Phân phối Event đi các Tenant)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `target_tenant_id` | BIGINT | No | 0 | IDX | Đích nhận (0 = BROADCAST/SYSTEM) |
| `target_tenant_type` | VARCHAR(20) | No |  |  | SAKI, MOTO, BROADCAST, SYSTEM |
| `event_type` | VARCHAR(100) | No |  | IDX | VD: PLATFORM_TENANT_SUSPENDED |
| `aggregate_type` | VARCHAR(50) | No |  | IDX | Loại thực thể |
| `aggregate_id` | VARCHAR(100) | No |  | IDX | ID thực thể |
| `payload` | JSONB | No |  |  | Data gửi đi |
| `idempotency_key` | VARCHAR(100) | No |  | Composite UK | ID do Platform sinh ra |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | ID truy vết Workflow |
| `status` | VARCHAR(20) | No | 'PENDING' | IDX | PENDING, SENDING, SENT, FAILED |
| `retry_count` | SMALLINT | No | 0 |  | Số lần thử lại |
| `last_error` | TEXT | Yes | NULL |  | Lỗi cuối cùng |
| `processed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm gửi xong |
| `next_retry_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm chạy lại nếu lỗi |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
- **UNIQUE** uq_platform_outbox_idem: (target_tenant_type, target_tenant_id, idempotency_key)
- **INDEX (composite)** idx_plat_outbox_aggregate: (aggregate_type, aggregate_id, created_at)
- **INDEX (composite)** idx_plat_outbox_worker: (status, next_retry_at)

### `cfg_platform_backup_policy` (Chính sách Sao lưu theo Tenant) [M09 NEW]

> Mỗi tenant có lịch sao lưu độc lập với retention period tùy chỉnh. Bảng cấu hình master cho job backup tự động — `t_platform_backup_job` tham chiếu về policy này tại moment chạy. Tách riêng config khỏi execution log.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ về `mst_tenant_registry(id)`. NULL = policy mặc định cho mọi tenant chưa cấu hình riêng |
| `policy_name` | VARCHAR(100) | No |  |  | Tên định danh (vd: "Daily Full Backup MOTO Tier-1") |
| `backup_type` | VARCHAR(50) | No |  | IDX | FULL_DB, TENANT_SCHEMA, S3_FILES |
| `frequency` | SMALLINT | No |  | IDX | 1=DAILY / 2=WEEKLY / 3=MONTHLY (BR-14D) |
| `cron_expression` | VARCHAR(100) | Yes | NULL |  | Cron schedule cụ thể (vd: "0 2 * * *" — 02:00 JST hàng ngày). NULL nếu dùng frequency mặc định |
| `retention_days` | INTEGER | No | 30 |  | Số ngày giữ backup. Default: 30 cho DAILY, 90 cho WEEKLY, 365 cho MONTHLY |
| `rto_minutes` | INTEGER | No | 60 |  | Recovery Time Objective target — BR-14H |
| `rpo_minutes` | INTEGER | No | 1440 |  | Recovery Point Objective (1 ngày = 1440 phút) — BR-14H |
| `encryption_required_flg` | SMALLINT | No | 1 |  | 1=Bắt buộc encrypt backup (compliance) |
| `s3_destination_prefix` | VARCHAR(500) | Yes | NULL |  | S3 prefix tùy chỉnh; mặc định theo BR-14G `s3://bucket/backups/{tenant_id}/` |
| `is_active` | SMALLINT | No | 1 | IDX | 1=Active, 0=Disabled (tạm tắt policy) |
| `last_executed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm policy chạy gần nhất |
| `next_scheduled_at` | TIMESTAMP | Yes | NULL | IDX | Lịch chạy tiếp theo (calculated từ cron + frequency) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE (Partial)** uq_backup_policy_tenant_type: (tenant_id, backup_type) WHERE deleted_at IS NULL — 1 policy per (tenant, backup_type)
- **INDEX** idx_backup_policy_active_next: (next_scheduled_at) WHERE is_active = 1 AND deleted_at IS NULL — batch worker scan
- **FK** fk_backup_policy_tenant: (tenant_id) → mst_tenant_registry(id)
- **CHECK** chk_backup_policy_frequency: frequency IN (1, 2, 3)
- **CHECK** chk_backup_policy_type: backup_type IN ('FULL_DB', 'TENANT_SCHEMA', 'S3_FILES')
- **CHECK** chk_backup_policy_retention: retention_days > 0
- **CHECK** chk_backup_policy_rto_rpo: rto_minutes > 0 AND rpo_minutes >= 0
- **CHECK** chk_backup_policy_encryption: encryption_required_flg IN (0, 1)
- **CHECK** chk_backup_policy_active: is_active IN (0, 1)
- **Trigger** trg_backup_policy_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_tenant_snapshot` (Snapshot An toàn trước Thao tác Nguy hiểm) [M09 NEW]

> BR-14A (snapshot trước SUSPEND), BR-14F (snapshot trước RESTORE OVERWRITE), BR-14J (snapshot trước MIGRATION). Tách hoàn toàn khỏi `t_platform_backup_job` vì mục đích khác: backup_job là lịch sao lưu định kỳ; snapshot là ad-hoc safety net trước operation. Có `retention_expires_at` — sau ngày này, batch cleanup xóa. Snapshot dùng cho rollback chỉ trong window đó.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, IDX | Trỏ về `mst_tenant_registry(id)` — snapshot scope per tenant |
| `snapshot_type` | SMALLINT | No |  | IDX | 1=PRE_SUSPEND (BR-14A) / 2=PRE_RESTORE (BR-14F) / 3=PRE_MIGRATION (BR-14J) / 4=MANUAL (Platform Admin tạo ad-hoc) |
| `rds_snapshot_id` | VARCHAR(255) | Yes | NULL | IDX | AWS RDS snapshot ID nếu dùng RDS-native snapshot |
| `s3_backup_path` | VARCHAR(500) | Yes | NULL |  | S3 path nếu là logical backup (pg_dump → S3) |
| `snapshot_size_mb` | BIGINT | Yes | NULL |  | Dung lượng snapshot (MB) |
| `triggered_by_operation_type` | VARCHAR(50) | Yes | NULL |  | Mã thao tác kích hoạt (vd: "SUSPEND_TENANT", "RESTORE_OVERWRITE", "DB_MIGRATION") |
| `triggered_by_operation_id` | BIGINT | Yes | NULL | IDX | ID của thao tác trigger (vd: provision_log_id hoặc restore_execution_id) |
| `status` | SMALLINT | No | 1 | IDX | 1=CREATING / 2=AVAILABLE / 3=USED_FOR_RESTORE (đã consume rollback) / 4=FAILED / 5=EXPIRED (quá retention) |
| `created_by_platform_user_id` | BIGINT | Yes | NULL | FK | Platform Admin trigger (NULL nếu auto-system) |
| `retention_expires_at` | TIMESTAMP | No | NOW() + INTERVAL '7 days' | IDX | Hạn tự động xóa snapshot. Default 7 ngày sau khi tạo (vận hành rollback window). App có thể override khi cần giữ lâu hơn (vd PRE_MIGRATION = 30 ngày) |
| `error_message` | TEXT | Yes | NULL |  | Lỗi nếu status=FAILED |
| `created_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm bắt đầu tạo snapshot |
| `available_at` | TIMESTAMP | Yes | NULL |  | Thời điểm snapshot sẵn sàng sử dụng |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_snapshot_tenant_time: (tenant_id, created_at DESC)
- **INDEX** idx_snapshot_expiry: (retention_expires_at, status) WHERE deleted_at IS NULL — batch cleanup
- **INDEX** idx_snapshot_status: (status) WHERE deleted_at IS NULL
- **FK** fk_snapshot_tenant: (tenant_id) → mst_tenant_registry(id)
- **FK** fk_snapshot_user: (created_by_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_snapshot_type: snapshot_type IN (1, 2, 3, 4)
- **CHECK** chk_snapshot_status: status IN (1, 2, 3, 4, 5)
- **CHECK** chk_snapshot_size: snapshot_size_mb IS NULL OR snapshot_size_mb >= 0
- **CHECK** chk_snapshot_location_required: snapshot_type IS NULL OR rds_snapshot_id IS NOT NULL OR s3_backup_path IS NOT NULL (phải có 1 trong 2 location)
- **Trigger** trg_snapshot_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_platform_restore_execution` (Lịch sử Phục hồi Dữ liệu — Critical Safety Log) [M09 NEW]

> BR-14E (Restore PHẢI dual-confirm — operation không hoàn tác được), BR-14F (Trước RESTORE OVERWRITE PHẢI tạo `t_tenant_snapshot` cho rollback). Tách riêng khỏi backup_job/provision_log vì restore là **thao tác cao rủi ro** — cần audit chuyên dụng với dual-confirm token, integrity check, rollback snapshot link.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, IDX | Trỏ về `mst_tenant_registry(id)` |
| `source_backup_job_id` | BIGINT | Yes | NULL | FK, IDX | Backup gốc từ đâu (nếu restore từ backup định kỳ) |
| `source_snapshot_id` | BIGINT | Yes | NULL | FK, IDX | Snapshot gốc nếu restore từ snapshot ad-hoc |
| `restore_mode` | SMALLINT | No |  | IDX | 1=OVERWRITE (ghi đè DB hiện tại — nguy hiểm, cần BR-14F snapshot) / 2=CLONE (restore sang DB mới) |
| `dual_confirm_token` | VARCHAR(100) | No |  |  | Token UUID sinh ở bước confirm thứ 1, validate ở bước confirm thứ 2 (BR-14E dual-confirm) |
| `dual_confirm_status` | SMALLINT | No | 1 | IDX | 1=AWAITING_2ND_CONFIRM / 2=CONFIRMED / 3=EXPIRED_TOKEN / 4=REJECTED |
| `rollback_snapshot_id` | BIGINT | Yes | NULL | FK | Snapshot tạo trước restore (BR-14F) cho rollback. Bắt buộc khi restore_mode=1 OVERWRITE |
| `status` | SMALLINT | No | 1 | IDX | 1=PENDING / 2=IN_PROGRESS / 3=COMPLETED / 4=FAILED / 5=ROLLED_BACK |
| `integrity_check_status` | SMALLINT | Yes | NULL |  | NULL=chưa check / 1=PASSED / 2=FAILED. Chạy sau RESTORE COMPLETED |
| `integrity_check_at` | TIMESTAMP | Yes | NULL |  | Thời điểm integrity check |
| `restored_by_platform_user_id` | BIGINT | No |  | FK, IDX | Platform Admin thực hiện restore |
| `requested_at` | TIMESTAMP | No | NOW() |  | Thời điểm submit yêu cầu restore |
| `started_at` | TIMESTAMP | Yes | NULL |  | Thời điểm bắt đầu chạy restore (sau dual-confirm) |
| `completed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm restore xong |
| `error_message` | TEXT | Yes | NULL |  | Lỗi nếu FAILED |
| `notes` | TEXT | Yes | NULL |  | Ghi chú vận hành — lý do restore |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | Trace HTTP |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | Trace workflow |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_restore_token: (dual_confirm_token)
- **INDEX** idx_restore_tenant_time: (tenant_id, requested_at DESC)
- **FK** fk_restore_tenant: (tenant_id) → mst_tenant_registry(id)
- **FK** fk_restore_source_backup: (source_backup_job_id) → t_platform_backup_job(id)
- **FK** fk_restore_source_snapshot: (source_snapshot_id) → t_tenant_snapshot(id)
- **FK** fk_restore_rollback_snapshot: (rollback_snapshot_id) → t_tenant_snapshot(id)
- **FK** fk_restore_user: (restored_by_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_restore_mode: restore_mode IN (1, 2)
- **CHECK** chk_restore_dual_confirm: dual_confirm_status IN (1, 2, 3, 4)
- **CHECK** chk_restore_status: status IN (1, 2, 3, 4, 5)
- **CHECK** chk_restore_integrity: integrity_check_status IS NULL OR integrity_check_status IN (1, 2)
- **CHECK** chk_restore_source_required: source_backup_job_id IS NOT NULL OR source_snapshot_id IS NOT NULL (phải có 1 nguồn)
- **CHECK** chk_restore_overwrite_rollback_required: restore_mode <> 1 OR rollback_snapshot_id IS NOT NULL (OVERWRITE bắt buộc có rollback snapshot — BR-14F)
- **Trigger** trg_restore_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_platform_maintenance_window` (Khung Giờ Bảo trì Platform) [M09 NEW]

> F-1409, BR-14P (SCHEDULED phải thông báo trước ≥48 giờ), BR-14Q (EMERGENCY có thể ngay nhưng phải thông báo đồng thời), BR-14R (default 02:00-06:00 JST). Quản lý lịch bảo trì có kế hoạch và emergency window. Liên kết với hệ thống notification để gửi tới các tenant bị ảnh hưởng.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `title` | VARCHAR(200) | No |  |  | Tiêu đề ngắn |
| `description` | TEXT | No |  |  | Mô tả chi tiết phục vụ thông báo tới tenant |
| `maintenance_type` | SMALLINT | No |  | IDX | 1=SCHEDULED (kế hoạch, ≥48h thông báo trước) / 2=EMERGENCY (khẩn cấp) |
| `scope` | SMALLINT | No |  | IDX | 1=ALL_TENANTS / 2=SPECIFIC_TENANTS / 3=PLATFORM_ONLY (Platform Admin internal, không ảnh hưởng tenant) |
| `affected_tenant_ids_payload` | JSONB | Yes | NULL |  | Danh sách tenant IDs khi scope=2: list IDs. Display-only, không filter individual → JSONB OK |
| `impact_level` | SMALLINT | No |  | IDX | 1=NO_IMPACT / 2=DEGRADED (slow) / 3=FULL_DOWNTIME |
| `planned_start_at` | TIMESTAMP | No |  | IDX | Thời điểm bắt đầu dự kiến |
| `planned_end_at` | TIMESTAMP | No |  |  | Thời điểm kết thúc dự kiến |
| `actual_start_at` | TIMESTAMP | Yes | NULL |  | Thời điểm thực sự bắt đầu |
| `actual_end_at` | TIMESTAMP | Yes | NULL |  | Thời điểm thực sự kết thúc |
| `status` | SMALLINT | No | 1 | IDX | 1=PLANNED / 2=IN_PROGRESS / 3=COMPLETED / 4=CANCELLED |
| `notification_hours_before` | SMALLINT | No | 48 |  | Gửi thông báo trước N giờ. SCHEDULED tối thiểu 48 (BR-14P) |
| `notification_sent_at_payload` | JSONB | Yes | NULL |  | Timestamps các lần đã gửi thông báo: list ISO datetime — audit "đã thông báo đủ chưa" |
| `cancelled_reason` | TEXT | Yes | NULL |  | Lý do nếu CANCELLED |
| `created_by_platform_user_id` | BIGINT | No |  | FK, IDX | Platform Admin tạo schedule |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_maintenance_status_start: (status, planned_start_at) — batch scan bản sắp đến
- **INDEX** idx_maintenance_window: (planned_start_at, planned_end_at)
- **FK** fk_maintenance_user: (created_by_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_maintenance_type: maintenance_type IN (1, 2)
- **CHECK** chk_maintenance_scope: scope IN (1, 2, 3)
- **CHECK** chk_maintenance_impact: impact_level IN (1, 2, 3)
- **CHECK** chk_maintenance_status: status IN (1, 2, 3, 4)
- **CHECK** chk_maintenance_period: planned_end_at > planned_start_at
- **CHECK** chk_maintenance_actual_period: actual_end_at IS NULL OR actual_start_at IS NULL OR actual_end_at > actual_start_at
- **CHECK** chk_maintenance_scheduled_notice: maintenance_type <> 1 OR notification_hours_before >= 48 (BR-14P)
- **CHECK** chk_maintenance_cancelled_reason: status <> 4 OR cancelled_reason IS NOT NULL (CANCELLED bắt buộc có lý do)
- **Trigger** trg_maintenance_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `mst_feature_flag` (Catalog Feature Flag — Progressive Rollout & A/B Test) [M09 NEW]

> F-1412, BR-14Y (cache Redis ≤30s), BR-14Z (KILL_SWITCH bypass cache <5s), BR-150 (mọi thay đổi audit log), BR-151 (flag tắt 30 ngày → nhắc cleanup). Bảng catalog định nghĩa flag; assignment per-tenant ở bảng riêng (`cfg_feature_flag_assignment`). Phân biệt với `cfg_platform_tenant_feature` (module on/off coarse-grained, semi-permanent).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `flag_key` | VARCHAR(100) | No |  | **UK** | Mã định danh (vd: "NEW_INVOICE_UI_V2", "EXPERIMENTAL_MATCHING_ALGO"). Immutable sau create |
| `flag_name` | VARCHAR(200) | No |  |  | Tên hiển thị |
| `description` | TEXT | Yes | NULL |  | Giải thích mục đích flag |
| `flag_type` | SMALLINT | No |  | IDX | 1=RELEASE (gradual rollout) / 2=EXPERIMENT (A/B test) / 3=OPERATIONAL (turn on/off feature theo vận hành) / 4=KILL_SWITCH (emergency disable — bypass cache BR-14Z) |
| `default_state` | SMALLINT | No | 0 |  | 0=OFF / 1=ON. State mặc định khi tenant chưa có assignment cụ thể |
| `is_active` | SMALLINT | No | 1 | IDX | 1=Flag đang được dùng / 0=Deprecated (sắp cleanup BR-151) |
| `cache_ttl_seconds` | INTEGER | No | 1800 |  | TTL Redis cache (mặc định 30 phút). Với KILL_SWITCH = 0 (bypass) — enforce ở app layer |
| `analytics_enabled_flg` | SMALLINT | No | 0 |  | 1=Track usage metrics (cho EXPERIMENT type) |
| `rollout_percentage` | SMALLINT | Yes | NULL |  | 0-100. Tỷ lệ tenant được bật (gradual rollout). NULL = dùng assignment cụ thể |
| `scheduled_enable_at` | TIMESTAMP | Yes | NULL | IDX | Tự động enable tại moment này |
| `scheduled_disable_at` | TIMESTAMP | Yes | NULL | IDX | Tự động disable tại moment này (cho time-boxed experiment) |
| `last_disabled_for_all_at` | TIMESTAMP | Yes | NULL |  | Thời điểm flag tắt cho tất cả tenant gần nhất — BR-151 30 ngày cleanup reminder |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_feature_flag_key: (flag_key)
- **INDEX** idx_feature_flag_active: (is_active) WHERE is_active = 1
- **INDEX** idx_feature_flag_scheduled_enable: (scheduled_enable_at) WHERE scheduled_enable_at IS NOT NULL AND is_active = 1
- **INDEX** idx_feature_flag_cleanup: (last_disabled_for_all_at) WHERE last_disabled_for_all_at IS NOT NULL AND is_active = 1 — BR-151 30d reminder scan
- **CHECK** chk_feature_flag_type: flag_type IN (1, 2, 3, 4)
- **CHECK** chk_feature_flag_default_state: default_state IN (0, 1)
- **CHECK** chk_feature_flag_active: is_active IN (0, 1)
- **CHECK** chk_feature_flag_cache_ttl: cache_ttl_seconds >= 0
- **CHECK** chk_feature_flag_analytics: analytics_enabled_flg IN (0, 1)
- **CHECK** chk_feature_flag_rollout_pct: rollout_percentage IS NULL OR (rollout_percentage >= 0 AND rollout_percentage <= 100)
- **CHECK** chk_feature_flag_kill_switch_cache: flag_type <> 4 OR cache_ttl_seconds = 0 (KILL_SWITCH phải bypass cache BR-14Z)
- **CHECK** chk_feature_flag_schedule_order: scheduled_disable_at IS NULL OR scheduled_enable_at IS NULL OR scheduled_disable_at > scheduled_enable_at
- **Trigger** trg_feature_flag_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `cfg_feature_flag_assignment` (Gán Feature Flag theo Tenant) [M09 NEW]

> Per-tenant override cho `mst_feature_flag`. NULL tenant_id = áp dụng toàn hệ thống. Hỗ trợ assignment_type INDIVIDUAL (per tenant cụ thể) / GROUP (theo MOTO/SAKI) / PERCENTAGE (gradual rollout — dùng tenant_id mod 100 hoặc hash).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `flag_id` | BIGINT | No |  | FK, IDX | Trỏ về `mst_feature_flag(id)` |
| `tenant_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ về `mst_tenant_registry(id)`. NULL khi assignment_type=GROUP/PERCENTAGE (áp dụng nhiều tenant) |
| `assignment_type` | SMALLINT | No |  | IDX | 1=INDIVIDUAL (1 tenant) / 2=GROUP (tenant_type MOTO/SAKI) / 3=PERCENTAGE (rollout %) |
| `group_tenant_type` | VARCHAR(20) | Yes | NULL |  | Khi assignment_type=2: 'MOTO' hoặc 'SAKI' |
| `is_enabled` | SMALLINT | No | 0 | IDX | 0=OFF / 1=ON cho assignment này |
| `enabled_at` | TIMESTAMP | Yes | NULL |  | Thời điểm bật |
| `disabled_at` | TIMESTAMP | Yes | NULL |  | Thời điểm tắt |
| `assignment_reason` | TEXT | Yes | NULL |  | Lý do gán (vd: "Beta tester program") |
| `enabled_by_platform_user_id` | BIGINT | Yes | NULL | FK | Platform Admin thực hiện |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE (Partial)** uq_flag_assignment_individual: (flag_id, tenant_id) WHERE assignment_type = 1 AND deleted_at IS NULL
- **UNIQUE (Partial)** uq_flag_assignment_group: (flag_id, group_tenant_type) WHERE assignment_type = 2 AND deleted_at IS NULL
- **INDEX** idx_flag_assignment_flag: (flag_id, is_enabled)
- **INDEX** idx_flag_assignment_tenant: (tenant_id, flag_id) WHERE tenant_id IS NOT NULL
- **FK** fk_flag_assignment_flag: (flag_id) → mst_feature_flag(id)
- **FK** fk_flag_assignment_tenant: (tenant_id) → mst_tenant_registry(id)
- **FK** fk_flag_assignment_user: (enabled_by_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_flag_assignment_type: assignment_type IN (1, 2, 3)
- **CHECK** chk_flag_assignment_enabled: is_enabled IN (0, 1)
- **CHECK** chk_flag_assignment_group_consistency: (assignment_type = 1 AND tenant_id IS NOT NULL AND group_tenant_type IS NULL) OR (assignment_type = 2 AND tenant_id IS NULL AND group_tenant_type IN ('MOTO', 'SAKI')) OR (assignment_type = 3 AND tenant_id IS NULL AND group_tenant_type IS NULL)
- **Trigger** trg_flag_assignment_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `mst_batch_job` (Catalog Batch Job — EventBridge Schedule) [M09 NEW]

> F-1425/1428: 11 critical jobs (daily_attendance, monthly_closure, invoice_generation, usage_fee, contract_status_update, notifications, audit_archive, data_retention_cleanup, backup, health_metrics, usage_stats). Catalog các job định nghĩa với schedule, timeout, retry policy. Thực thi → `t_batch_job_execution`. BR-187 retry 3 lần exponential backoff, BR-191 không peak hour (8:00-9:30, 17:00-18:00).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `job_name` | VARCHAR(100) | No |  | **UK** | Mã định danh job (vd: "daily_attendance_aggregation"). Immutable |
| `job_description` | TEXT | Yes | NULL |  | Mô tả mục đích job |
| `job_type` | SMALLINT | No |  | IDX | 1=ACCOUNTING (chấm công/hóa đơn) / 2=BILLING / 3=MONITORING / 4=MAINTENANCE / 5=NOTIFICATION / 6=DATA_RETENTION / 7=BACKUP |
| `cron_expression` | VARCHAR(100) | No |  |  | Cron schedule (vd: "0 2 * * *" — 02:00 JST hàng ngày) |
| `timezone` | VARCHAR(50) | No | 'Asia/Tokyo' |  | Timezone cho cron |
| `status` | SMALLINT | No | 1 | IDX | 1=ENABLED / 0=DISABLED (BR-193: tắt job quan trọng cần dual-confirm) |
| `timeout_minutes` | INTEGER | No | 60 |  | Max thời gian chạy. Vượt → mark FAILED |
| `max_concurrent_tenants` | INTEGER | No | 5 |  | Số tenant chạy song song max (BR-191 protect resource) |
| `retry_policy` | SMALLINT | No | 2 |  | 1=NO_RETRY / 2=AUTO_RETRY (BR-187 3 lần exponential) / 3=MANUAL (chỉ Platform Admin retry thủ công) |
| `max_retries` | SMALLINT | No | 3 |  | Max auto-retry (BR-187: 3 lần) |
| `retry_backoff_base_minutes` | INTEGER | No | 1 |  | Backoff base (minutes). BR-187: 1, 5, 15 → backoff_base × 5^(retry_count) |
| `tenant_filter_payload` | JSONB | Yes | NULL |  | Danh sách tenant_ids chạy. NULL = tất cả active tenant. Display-only, ≥5 items |
| `last_run_at` | TIMESTAMP | Yes | NULL |  | Thời điểm chạy gần nhất |
| `last_run_status` | SMALLINT | Yes | NULL |  | Status lần chạy gần nhất (snapshot cho dashboard) |
| `next_run_at` | TIMESTAMP | Yes | NULL | IDX | Lịch chạy tiếp theo (calculated từ cron) |
| `avg_execution_minutes` | DECIMAL(10,2) | Yes | NULL |  | Thời gian chạy trung bình — BR-186 cảnh báo nếu >2x |
| `critical_flg` | SMALLINT | No | 0 |  | 1=Critical job (daily_attendance, invoice_generation) — disable cần dual-confirm BR-193 |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_batch_job_name: (job_name)
- **INDEX** idx_batch_job_next_run: (next_run_at) WHERE status = 1 AND deleted_at IS NULL — scheduler scan
- **INDEX** idx_batch_job_type: (job_type, status)
- **CHECK** chk_batch_job_type: job_type IN (1, 2, 3, 4, 5, 6, 7)
- **CHECK** chk_batch_job_status: status IN (0, 1)
- **CHECK** chk_batch_job_timeout: timeout_minutes > 0
- **CHECK** chk_batch_job_concurrency: max_concurrent_tenants >= 1
- **CHECK** chk_batch_job_retry_policy: retry_policy IN (1, 2, 3)
- **CHECK** chk_batch_job_max_retries: max_retries >= 0 AND max_retries <= 10
- **CHECK** chk_batch_job_backoff: retry_backoff_base_minutes > 0
- **CHECK** chk_batch_job_critical: critical_flg IN (0, 1)
- **Trigger** trg_batch_job_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_batch_job_execution` (Lịch sử Thực thi Batch Job — Platform-level Summary) [M09 NEW]

> F-1426: Dashboard batch tự cập nhật mỗi 30s. Mỗi lần job chạy → 1 dòng ở bảng này (Platform-level aggregate). Chi tiết per-tenant ở `t_batch_job_tenant_execution`. BR-184 ghi đủ thông tin trace, BR-185 dashboard 30s refresh.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `job_id` | BIGINT | No |  | FK, IDX | Trỏ về `mst_batch_job(id)` |
| `execution_number` | INTEGER | No |  |  | Sequential count per job_id (1, 2, 3...) — audit trail |
| `started_at` | TIMESTAMP | No | NOW() | IDX | Bắt đầu chạy |
| `completed_at` | TIMESTAMP | Yes | NULL |  | Kết thúc chạy |
| `status` | SMALLINT | No | 1 | IDX | 1=RUNNING / 2=SUCCESS / 3=FAILED / 4=TIMEOUT / 5=SKIPPED / 6=MANUALLY_RESOLVED |
| `total_tenants` | INTEGER | No | 0 |  | Số tenant cần xử lý |
| `succeeded_tenants` | INTEGER | No | 0 |  | Tenant thành công |
| `failed_tenants` | INTEGER | No | 0 |  | Tenant lỗi |
| `skipped_tenants` | INTEGER | No | 0 |  | Tenant skip (BR-190: lỗi data — phải có skip_reason) |
| `total_records_processed` | BIGINT | No | 0 |  | Tổng record xử lý qua mọi tenant |
| `total_records_failed` | BIGINT | No | 0 |  | Tổng record fail qua mọi tenant |
| `execution_time_seconds` | INTEGER | Yes | NULL |  | Thời gian chạy (giây) |
| `avg_time_per_tenant_seconds` | DECIMAL(10,2) | Yes | NULL |  | Trung bình per tenant — trend analysis |
| `error_summary` | TEXT | Yes | NULL |  | Tổng hợp lỗi nếu FAILED |
| `retry_count` | INTEGER | No | 0 |  | Số lần auto-retry đã thực hiện (BR-187) |
| `retry_log_payload` | JSONB | Yes | NULL |  | Chi tiết từng lần retry: [{attempt, started_at, failed_at, error}]. Display-only audit |
| `triggered_by` | SMALLINT | No |  | IDX | 1=SCHEDULED (cron) / 2=MANUAL (Platform Admin) / 3=AUTO_RETRY (BR-187) / 4=RETRY_AFTER_SUSPEND (BR-14C catch-up) |
| `triggered_by_platform_user_id` | BIGINT | Yes | NULL | FK | Platform Admin nếu MANUAL |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | Trace |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_batch_exec_job_num: (job_id, execution_number)
- **INDEX** idx_batch_exec_job_time: (job_id, started_at DESC)
- **INDEX** idx_batch_exec_status: (status, started_at DESC) WHERE deleted_at IS NULL
- **FK** fk_batch_exec_job: (job_id) → mst_batch_job(id)
- **FK** fk_batch_exec_user: (triggered_by_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_batch_exec_status: status IN (1, 2, 3, 4, 5, 6)
- **CHECK** chk_batch_exec_triggered: triggered_by IN (1, 2, 3, 4)
- **CHECK** chk_batch_exec_counts: total_tenants >= 0 AND succeeded_tenants >= 0 AND failed_tenants >= 0 AND skipped_tenants >= 0 AND (succeeded_tenants + failed_tenants + skipped_tenants) <= total_tenants
- **CHECK** chk_batch_exec_records: total_records_processed >= 0 AND total_records_failed >= 0 AND total_records_failed <= total_records_processed
- **CHECK** chk_batch_exec_retry: retry_count >= 0 AND retry_count <= 10
- **CHECK** chk_batch_exec_manual_user: triggered_by <> 2 OR triggered_by_platform_user_id IS NOT NULL
- **Trigger** trg_batch_exec_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_batch_job_tenant_execution` (Chi tiết Thực thi Batch Job per Tenant) [M09 NEW]

> BR-183 isolation: lỗi của 1 tenant không ảnh hưởng tenant khác. Mỗi execution_id × tenant_id = 1 row ở đây. Phục vụ F-1427 (retry failed tenants), filter dashboard theo tenant. Volume: ~11 jobs × ~100 tenant × ~1x/ngày × 90d retention = ~99k rows/3 tháng — vừa phải.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `execution_id` | BIGINT | No |  | FK, IDX | Trỏ về `t_batch_job_execution(id)` — parent execution |
| `job_id` | BIGINT | No |  | FK, IDX | Trỏ về `mst_batch_job(id)` — denormalize cho filter dễ |
| `tenant_id` | BIGINT | No |  | FK, IDX | Trỏ về `mst_tenant_registry(id)` |
| `status` | SMALLINT | No | 1 | IDX | 1=RUNNING / 2=SUCCESS / 3=FAILED / 4=TIMEOUT / 5=SKIPPED |
| `records_processed` | INTEGER | No | 0 |  | Số record xử lý cho tenant này |
| `records_failed` | INTEGER | No | 0 |  | Số record fail cho tenant này |
| `execution_time_seconds` | INTEGER | Yes | NULL |  | Thời gian xử lý tenant này |
| `error_message` | TEXT | Yes | NULL |  | Chi tiết lỗi cho tenant này |
| `error_type` | SMALLINT | Yes | NULL |  | 1=DB_ERROR / 2=TIMEOUT / 3=DATA_ERROR / 4=UNKNOWN — categorize cho dashboard |
| `skip_reason` | TEXT | Yes | NULL |  | Bắt buộc nếu status=SKIPPED (BR-190) |
| `start_time` | TIMESTAMP | Yes | NULL |  | Bắt đầu xử lý tenant |
| `end_time` | TIMESTAMP | Yes | NULL |  | Kết thúc xử lý tenant |
| `retry_attempt` | SMALLINT | No | 0 |  | Lần retry thứ N cho tenant cụ thể (0=lần đầu, 1=retry 1...) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_batch_tenant_exec: (execution_id, tenant_id, retry_attempt)
- **INDEX** idx_batch_tenant_exec_status: (job_id, status, start_time DESC) WHERE deleted_at IS NULL — dashboard filter
- **INDEX** idx_batch_tenant_exec_tenant: (tenant_id, start_time DESC) WHERE deleted_at IS NULL — per-tenant view
- **INDEX** idx_batch_tenant_exec_failed: (execution_id, status) WHERE status = 3 AND deleted_at IS NULL — F-1427 retry failed
- **FK** fk_batch_tenant_exec_execution: (execution_id) → t_batch_job_execution(id)
- **FK** fk_batch_tenant_exec_job: (job_id) → mst_batch_job(id)
- **FK** fk_batch_tenant_exec_tenant: (tenant_id) → mst_tenant_registry(id)
- **CHECK** chk_batch_tenant_exec_status: status IN (1, 2, 3, 4, 5)
- **CHECK** chk_batch_tenant_exec_records: records_processed >= 0 AND records_failed >= 0 AND records_failed <= records_processed
- **CHECK** chk_batch_tenant_exec_error_type: error_type IS NULL OR error_type IN (1, 2, 3, 4)
- **CHECK** chk_batch_tenant_exec_skip_reason: status <> 5 OR (skip_reason IS NOT NULL AND trim(skip_reason) <> '')
- **CHECK** chk_batch_tenant_exec_retry: retry_attempt >= 0 AND retry_attempt <= 10
- **Trigger** trg_batch_tenant_exec_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_evidence_timestamp` (Sổ Đăng Ký Dấu Thời Gian Bằng Chứng — 電子帳簿保存法 TSA Registry) [M10 NEW]

> Universal registry cho TSA (Time Stamp Authority) signing toàn hệ thống — BR-1548 (7-day window), BR-1553 (TSA phải được 日本データ通信協会 公認), BR-1554 (SHA-256+, cấm MD5/SHA-1), BR-1555 (gap-free audit chain). Áp dụng cho mọi entity cần chứng minh tính bất biến: 36協定 PDF, 派遣先/派遣元管理台帳 export PDF, hợp đồng PDF, hóa đơn PDF, treatment notification PDF. Polymorphic association qua (entity_type, entity_tenant_id, entity_id) — Backend reuse cho mọi loại tài liệu pháp lý. Retention dài (theo thực thể gốc; tối thiểu 7 năm cho hóa đơn theo 電子帳簿保存法 + 12年4ヶ月 cho hợp đồng/台帳).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `entity_type` | VARCHAR(50) | No |  | IDX | Mã loại entity: "36_AGREEMENT_DOC", "SAKI_LEDGER_EXPORT", "MOTO_LEDGER_EXPORT", "CONTRACT_PDF", "INVOICE_PDF", "TREATMENT_NOTIFICATION_PDF", ... |
| `entity_tenant_id` | BIGINT | Yes | NULL | IDX | Tenant sở hữu entity (NULL nếu Platform-level entity). Cho phép Platform Admin query cross-tenant; tenant chỉ thấy của mình qua RLS |
| `entity_tenant_type` | VARCHAR(20) | Yes | NULL |  | 'MOTO' / 'SAKI' / NULL (Platform). Phân biệt khi entity_id duplicate giữa 2 schema |
| `entity_id` | BIGINT | No |  | IDX | ID của entity cụ thể (vd: id của row trong t_36_agreement_document) |
| `file_hash_sha256` | VARCHAR(64) | No |  | IDX | SHA-256 hash hex 64 ký tự — BR-1554 cấm MD5/SHA-1 |
| `hash_algorithm` | VARCHAR(20) | No | 'SHA-256' |  | CHECK chỉ cho phép SHA-256, SHA-384, SHA-512 |
| `tsa_token` | TEXT | No |  |  | Token nhận từ TSA (RFC 3161 timestamp token) — base64 hoặc PEM |
| `tsa_authority_name` | VARCHAR(200) | No |  | IDX | Tên TSA cấp (vd: "セイコーソリューションズ TS Authority", "アマノセキュアジャパン TSA") — BR-1553 phải được 日本データ通信協会 công nhận |
| `tsa_certificate_serial` | VARCHAR(200) | Yes | NULL |  | Serial number cert TSA dùng signing — phục vụ verify ngược |
| `timestamp_issued_at` | TIMESTAMP | No |  | IDX | Thời điểm TSA cấp timestamp (theo TSA response, không phải DB clock) |
| `file_storage_key` | VARCHAR(500) | Yes | NULL |  | S3 key của file gốc đã được signed (nullable nếu chỉ lưu hash, không lưu file) |
| `file_size_bytes` | BIGINT | Yes | NULL |  | Kích thước file signed |
| `verification_status` | SMALLINT | No | 1 | IDX | 1=ISSUED (mới cấp, chưa verify) / 2=VERIFIED (hash khớp) / 3=VERIFICATION_FAILED (hash không khớp — Critical alert) / 4=TSA_EXPIRED (cert TSA hết hạn) |
| `last_verified_at` | TIMESTAMP | Yes | NULL |  | Lần verify cuối cùng (batch chạy định kỳ verify integrity) |
| `verification_error` | TEXT | Yes | NULL |  | Chi tiết lỗi nếu verify fail |
| `previous_timestamp_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ về timestamp trước theo cùng (entity_type, entity_tenant_id, entity_id) — chain for gap-free audit BR-1555 |
| `chain_sequence` | INTEGER | No | 1 |  | Thứ tự trong chain cho 1 entity (1, 2, 3...) |
| `signed_by_platform_user_id` | BIGINT | Yes | NULL | FK | Platform Admin trigger nếu thủ công; NULL nếu auto-system |
| `created_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm tạo bản ghi (DB clock — có thể chênh với timestamp_issued_at) |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật (chỉ verification_status / last_verified_at update — entity vĩnh viễn immutable BR-1551) |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (KHÔNG được dùng — chỉ Platform Admin sau khi hết retention pháp lý) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_evidence_entity: (entity_type, entity_tenant_type, entity_tenant_id, entity_id)
- **INDEX** idx_evidence_chain: (entity_type, entity_id, chain_sequence)
- **INDEX** idx_evidence_hash: (file_hash_sha256) — verify lookup
- **INDEX** idx_evidence_verify_status: (verification_status) WHERE verification_status IN (3, 4) — Critical alert scan
- **UNIQUE** uq_evidence_chain_seq: (entity_type, entity_tenant_id, entity_id, chain_sequence)
- **FK** fk_evidence_previous: (previous_timestamp_id) → t_evidence_timestamp(id) self-reference
- **FK** fk_evidence_user: (signed_by_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_evidence_hash_algo: hash_algorithm IN ('SHA-256', 'SHA-384', 'SHA-512')
- **CHECK** chk_evidence_hash_format: file_hash_sha256 ~ '^[A-Fa-f0-9]+$' AND length(file_hash_sha256) IN (64, 96, 128)
- **CHECK** chk_evidence_verify_status: verification_status IN (1, 2, 3, 4)
- **CHECK** chk_evidence_chain_seq: chain_sequence >= 1
- **CHECK** chk_evidence_chain_consistency: (chain_sequence = 1 AND previous_timestamp_id IS NULL) OR (chain_sequence > 1 AND previous_timestamp_id IS NOT NULL)
- **CHECK** chk_evidence_entity_tenant_type: entity_tenant_type IS NULL OR entity_tenant_type IN ('MOTO', 'SAKI')
- **Trigger** trg_evidence_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_compliance_violation` (Sổ Đăng Ký Vi Phạm Compliance — VIO-01...VIO-14) [M10 NEW]

> CMP-018 BR-1567/1568/1569 deadline theo severity, BR-1570 immutable (chỉ status thay đổi không xóa), BR-1571 bác bỏ cần dual-approval, BR-1573 Level 3 khóa chức năng, BR-1574 Critical bắt buộc RCA, BR-1575 lưu trữ vĩnh viễn. Cross-tenant central: tracking 14 loại vi phạm trải qua SAKI/MOTO/cross-tenant. Platform Admin xem all cross-tenant (BR-1521); tenant query own qua filter saki_tenant_id/moto_tenant_id.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `violation_no` | VARCHAR(50) | No |  | **UK** | Mã định danh dạng "VIO-YYYYMMDD-NNNN" — sequential per ngày, phục vụ tracking & external reference |
| `violation_type` | VARCHAR(20) | No |  | IDX | "VIO-01" ... "VIO-14" — 14 loại vi phạm (xem CMP-018 doc 15) |
| `severity` | SMALLINT | No |  | IDX | 1=Low / 2=Medium / 3=High / 4=Critical |
| `saki_tenant_id` | BIGINT | Yes | NULL | IDX | SAKI tenant liên quan (NULL nếu vi phạm thuần MOTO) |
| `moto_tenant_id` | BIGINT | Yes | NULL | IDX | MOTO tenant liên quan |
| `related_contract_no` | VARCHAR(50) | Yes | NULL | IDX | Hợp đồng liên quan |
| `related_staff_id` | BIGINT | Yes | NULL | IDX | Staff liên quan (logical reference đến mst_staff trong MOTO schema) |
| `detected_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm phát hiện vi phạm |
| `detection_source` | SMALLINT | No |  |  | 1=BATCH (job tự phát hiện) / 2=REALTIME (real-time check khi ghi nhận chấm công) / 3=MANUAL (Admin nhập) |
| `violation_details_payload` | JSONB | No |  |  | Chi tiết vi phạm theo loại VIO. Variable schema per type (vd VIO-04 multi-month avg: {avg_value, window_months, threshold, exceed_hours}; VIO-08 抵触日 overdue: {office_id, conflict_date, days_overdue}). Display-only, không filter individual — D3 JSONB OK |
| `status` | SMALLINT | No | 1 | IDX | 1=未対応 OPEN / 2=対応中 IN_PROGRESS / 3=是正済み RESOLVED / 4=クローズ CLOSED / 5=棄却 DISMISSED (cần dual-approval BR-1571) |
| `escalation_level` | SMALLINT | No | 1 | IDX | 1=Level 1 (Operator) / 2=Level 2 (Manager) / 3=Level 3 (Platform Admin khóa chức năng — BR-1573) |
| `escalation_deadline_at` | TIMESTAMP | Yes | NULL | IDX | Hạn xử lý: Critical=24h, High=3 ngày làm việc, Medium=7 ngày — BR-1567/1568/1569 |
| `assigned_to_platform_user_id` | BIGINT | Yes | NULL | FK, IDX | Người được giao xử lý (NULL khi chưa assign) |
| `corrective_action` | TEXT | Yes | NULL |  | Mô tả hành động khắc phục đã/sẽ thực hiện |
| `root_cause_payload` | TEXT | Yes | NULL |  | RCA (Root Cause Analysis) — BR-1574 BẮT BUỘC khi severity=Critical trước khi CLOSE |
| `evidence_payload` | JSONB | Yes | NULL |  | File chứng minh (S3 key list, max 5 files mỗi violation). Display-only |
| `dismissal_reason` | TEXT | Yes | NULL |  | Lý do bác bỏ — BR-1571 BẮT BUỘC khi status=DISMISSED |
| `dismissal_approved_by_platform_user_id` | BIGINT | Yes | NULL | FK | Admin cấp cao phê duyệt bác bỏ — BR-1571 dual-approval |
| `dismissal_approved_at` | TIMESTAMP | Yes | NULL |  | Thời điểm duyệt bác bỏ |
| `closed_at` | TIMESTAMP | Yes | NULL |  | Thời điểm CLOSED |
| `closed_by_platform_user_id` | BIGINT | Yes | NULL | FK | Người đóng |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (TUYỆT ĐỐI KHÔNG dùng — BR-1570/1575 immutable; chỉ giữ cho audit format consistency) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **UNIQUE** uq_violation_no: (violation_no)
- **INDEX** idx_violation_status_deadline: (status, escalation_deadline_at) WHERE status IN (1, 2) — batch alert overdue
- **INDEX** idx_violation_severity_status: (severity, status, detected_at DESC) — dashboard critical list
- **INDEX** idx_violation_saki: (saki_tenant_id, detected_at DESC) WHERE saki_tenant_id IS NOT NULL
- **INDEX** idx_violation_moto: (moto_tenant_id, detected_at DESC) WHERE moto_tenant_id IS NOT NULL
- **INDEX** idx_violation_type_severity: (violation_type, severity)
- **FK** fk_violation_assigned: (assigned_to_platform_user_id) → mst_platform_user(id)
- **FK** fk_violation_dismissal_approver: (dismissal_approved_by_platform_user_id) → mst_platform_user(id)
- **FK** fk_violation_closer: (closed_by_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_violation_type_format: violation_type ~ '^VIO-[0-9]{2}$'
- **CHECK** chk_violation_severity: severity IN (1, 2, 3, 4)
- **CHECK** chk_violation_status: status IN (1, 2, 3, 4, 5)
- **CHECK** chk_violation_escalation_level: escalation_level IN (1, 2, 3)
- **CHECK** chk_violation_detection_source: detection_source IN (1, 2, 3)
- **CHECK** chk_violation_critical_rca: severity <> 4 OR status NOT IN (3, 4) OR (root_cause_payload IS NOT NULL AND trim(root_cause_payload) <> '') (BR-1574)
- **CHECK** chk_violation_dismissal_requires_approval: status <> 5 OR (dismissal_reason IS NOT NULL AND dismissal_approved_by_platform_user_id IS NOT NULL AND dismissal_approved_at IS NOT NULL) (BR-1571)
- **CHECK** chk_violation_tenant_required: saki_tenant_id IS NOT NULL OR moto_tenant_id IS NOT NULL (phải có ít nhất 1 tenant liên quan)
- **CHECK** chk_violation_closed_consistency: (status <> 4 AND closed_at IS NULL AND closed_by_platform_user_id IS NULL) OR (status = 4 AND closed_at IS NOT NULL AND closed_by_platform_user_id IS NOT NULL)
- **Trigger** trg_violation_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_compliance_violation_timeline` (Lịch sử Trạng thái & Leo thang Vi phạm — Immutable Audit) [M10 NEW]

> BR-1570 vi phạm immutable, BR-1572 mọi thay đổi trạng thái phải audit log. Tách khỏi `t_compliance_violation` để timeline grow độc lập (1 violation có thể có 5-20 status transitions). Pure-immutable insert-only — không UPDATE/DELETE sau khi tạo (chỉ deleted_at NULL forever theo BR-1575 vĩnh viễn).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `violation_id` | BIGINT | No |  | FK, IDX | Trỏ về `t_compliance_violation(id)` |
| `event_type` | SMALLINT | No |  | IDX | 1=CREATED / 2=STATUS_CHANGED / 3=ESCALATION_LEVEL_UP / 4=ESCALATION_LEVEL_DOWN / 5=ASSIGNED / 6=REASSIGNED / 7=NOTE_ADDED / 8=EVIDENCE_ADDED / 9=DISMISSAL_REQUESTED / 10=DISMISSAL_APPROVED / 11=DISMISSAL_REJECTED |
| `old_status` | SMALLINT | Yes | NULL |  | Status trước (NULL với CREATED) |
| `new_status` | SMALLINT | Yes | NULL |  | Status sau |
| `old_escalation_level` | SMALLINT | Yes | NULL |  | Escalation level trước |
| `new_escalation_level` | SMALLINT | Yes | NULL |  | Escalation level sau |
| `old_assigned_to_platform_user_id` | BIGINT | Yes | NULL |  | Assignee trước |
| `new_assigned_to_platform_user_id` | BIGINT | Yes | NULL |  | Assignee sau |
| `event_note` | TEXT | Yes | NULL |  | Ghi chú (bắt buộc với event_type=7 NOTE_ADDED) |
| `actor_platform_user_id` | BIGINT | Yes | NULL | FK, IDX | Platform Admin thực hiện (NULL nếu auto-system escalation BR-1567 deadline pass) |
| `actor_type` | VARCHAR(20) | No | 'PLATFORM_USER' |  | PLATFORM_USER, SYSTEM |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | Trace HTTP |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | Trace workflow |
| `event_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm event (event time) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi (DB clock — có thể chênh với event_at nếu post hoc) |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối (immutable — sẽ không cập nhật) |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | TUYỆT ĐỐI KHÔNG dùng — BR-1575 vĩnh viễn |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_violation_timeline_violation: (violation_id, event_at DESC)
- **INDEX** idx_violation_timeline_actor: (actor_platform_user_id, event_at DESC) WHERE actor_platform_user_id IS NOT NULL
- **FK** fk_violation_timeline_violation: (violation_id) → t_compliance_violation(id)
- **FK** fk_violation_timeline_actor: (actor_platform_user_id) → mst_platform_user(id)
- **CHECK** chk_violation_timeline_event_type: event_type IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11)
- **CHECK** chk_violation_timeline_status_change: event_type <> 2 OR (old_status IS NOT NULL AND new_status IS NOT NULL AND old_status <> new_status)
- **CHECK** chk_violation_timeline_escalation_change: event_type NOT IN (3, 4) OR (old_escalation_level IS NOT NULL AND new_escalation_level IS NOT NULL AND old_escalation_level <> new_escalation_level)
- **CHECK** chk_violation_timeline_note_required: event_type <> 7 OR (event_note IS NOT NULL AND trim(event_note) <> '')
- **CHECK** chk_violation_timeline_actor_consistency: (actor_type = 'SYSTEM' AND actor_platform_user_id IS NULL) OR (actor_type = 'PLATFORM_USER' AND actor_platform_user_id IS NOT NULL)
- **Trigger** trg_violation_timeline_updated_at: BEFORE UPDATE → fn_set_updated_at()

### `t_ledger_export_history` (Lịch sử Xuất Sổ Quản Lý 派遣先/派遣元 — Cross-tenant Audit) [M10 NEW]

> CMP-012 BR-1544 (PDF format chuẩn 厚生労働省), BR-1545 (watermark người xuất + ngày + mục đích), BR-1547 (audit log mỗi lần xuất). Cross-tenant table cho phép Platform Admin audit toàn bộ exports 監督署 inspection. **KEY**: Mỗi lần export sinh PDF tĩnh trên S3 + SHA-256 hash + link sang `t_evidence_timestamp` cho TSA — đảm bảo immutable snapshot theo 電子帳簿保存法 (retention 12年4ヶ月 cho hợp đồng/台帳, 7 năm cho hóa đơn).
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `ledger_type` | SMALLINT | No |  | IDX | 1=SAKI_LEDGER 派遣先管理台帳 (Article 42) / 2=MOTO_LEDGER 派遣元管理台帳 (Article 37) |
| `exporting_tenant_id` | BIGINT | No |  | FK, IDX | Tenant thực hiện export (SAKI nếu ledger_type=1, MOTO nếu =2) |
| `exporting_tenant_type` | VARCHAR(20) | No |  |  | 'SAKI' / 'MOTO' |
| `related_contract_no` | VARCHAR(50) | Yes | NULL | IDX | Hợp đồng liên quan (NULL nếu export batch all contracts) |
| `related_staff_id` | BIGINT | Yes | NULL | IDX | Staff cụ thể (NULL nếu export batch) |
| `coverage_period_start` | DATE | No |  |  | Kỳ báo cáo từ |
| `coverage_period_end` | DATE | No |  |  | Kỳ báo cáo đến |
| `export_format` | SMALLINT | No |  |  | 1=PDF (default, chuẩn 厚労省) / 2=CSV / 3=EXCEL |
| `export_purpose` | VARCHAR(200) | No |  |  | Mục đích export (BR-1545: watermark sẽ in lý do này lên file) |
| `pdf_s3_storage_key` | VARCHAR(500) | Yes | NULL |  | S3 key của PDF immutable snapshot. NULL nếu format != PDF. Đối với compliance, BẮT BUỘC render PDF tĩnh + hash + TSA (Q1 user decision) |
| `pdf_file_hash_sha256` | VARCHAR(64) | Yes | NULL | IDX | SHA-256 hash PDF — BR-1554 |
| `pdf_file_size_bytes` | BIGINT | Yes | NULL |  | Kích thước PDF |
| `evidence_timestamp_id` | BIGINT | Yes | NULL | FK, IDX | Trỏ về `t_evidence_timestamp(id)` cho TSA signing — BR-1548 trong 7 ngày |
| `watermark_payload` | JSONB | No |  |  | Snapshot watermark đã in trên file: {exported_by_user_name, export_datetime, export_purpose, tenant_name} — BR-1545 |
| `record_count` | INTEGER | No | 0 |  | Số record trong file (số dòng dữ liệu) |
| `legal_basis` | VARCHAR(100) | Yes | NULL |  | "派遣法 Article 42" / "派遣法 Article 37" — căn cứ pháp lý |
| `retention_expires_at` | DATE | No |  | IDX | Hạn lưu trữ tối thiểu: 3 năm + buffer theo 派遣法 BR-1538/1542; tuy nhiên file PDF tĩnh trên S3 giữ tới 12年4ヶ月 theo 電子帳簿保存法 |
| `exported_by_platform_user_id` | BIGINT | Yes | NULL | FK | NULL nếu tenant user export (refer tenant user qua exported_by_tenant_user_id) |
| `exported_by_tenant_user_id` | BIGINT | Yes | NULL |  | User của tenant thực hiện export (NULL nếu Platform Admin) — logical FK đến mst_saki_user/mst_moto_user theo exporting_tenant_type |
| `request_id` | VARCHAR(100) | Yes | NULL | IDX | Trace HTTP |
| `correlation_id` | VARCHAR(100) | Yes | NULL | IDX | Trace workflow |
| `created_at` | TIMESTAMP | No | NOW() | IDX | Thời điểm export |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật (immutable — sẽ không cập nhật) |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Soft Delete (chỉ sau retention_expires_at) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **PK**: (id)
- **INDEX** idx_ledger_export_tenant_time: (exporting_tenant_id, created_at DESC)
- **INDEX** idx_ledger_export_type_period: (ledger_type, coverage_period_end DESC)
- **INDEX** idx_ledger_export_retention: (retention_expires_at) WHERE deleted_at IS NULL — cleanup batch
- **FK** fk_ledger_export_user: (exported_by_platform_user_id) → mst_platform_user(id)
- **FK** fk_ledger_export_evidence: (evidence_timestamp_id) → t_evidence_timestamp(id)
- **CHECK** chk_ledger_export_type: ledger_type IN (1, 2)
- **CHECK** chk_ledger_export_tenant_type: exporting_tenant_type IN ('SAKI', 'MOTO')
- **CHECK** chk_ledger_export_tenant_consistency: (ledger_type = 1 AND exporting_tenant_type = 'SAKI') OR (ledger_type = 2 AND exporting_tenant_type = 'MOTO')
- **CHECK** chk_ledger_export_format: export_format IN (1, 2, 3)
- **CHECK** chk_ledger_export_period: coverage_period_end >= coverage_period_start
- **CHECK** chk_ledger_export_record_count: record_count >= 0
- **CHECK** chk_ledger_export_purpose: trim(export_purpose) <> ''
- **CHECK** chk_ledger_export_pdf_consistency: (export_format <> 1) OR (pdf_s3_storage_key IS NOT NULL AND pdf_file_hash_sha256 IS NOT NULL AND evidence_timestamp_id IS NOT NULL) (PDF format BẮT BUỘC có S3 + hash + TSA — Q1 user decision)
- **CHECK** chk_ledger_export_actor: exported_by_platform_user_id IS NOT NULL OR exported_by_tenant_user_id IS NOT NULL
- **Trigger** trg_ledger_export_updated_at: BEFORE UPDATE → fn_set_updated_at()

---

## NHÓM 5: Quản lý cước phí SaaS (Billing)

### `t_tenant_usage_metric` (Đo lường lượng sử dụng hàng tháng của Tenant)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_tenant_registry(id) |
| `target_month` | VARCHAR(7) | No |  | IDX | Tháng đo lường (YYYY-MM) |
| `active_user_count` | INT | No | 0 |  | Số lượng User hoạt động trong tháng |
| `active_contract_count` | INT | No | 0 |  | Số lượng Hợp đồng đang chạy trong tháng |
| `storage_used_mb` | DECIMAL(10,2) | No | 0 |  | Dung lượng lưu trữ file đã dùng (MB) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

### `t_usage_billing` (OUT_OF_SCOPE_FOR_CARIMA_CURRENT_PHASE - Hóa đơn tính cước dịch vụ Platform)

> Carima current phase không triển khai tenant subscription/platform SaaS billing.
Bảng này chỉ giữ làm reference từ e-staffing-like SaaS billing scope, không implement BF-M01.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `tenant_id` | BIGINT | No |  | FK, IDX | Tham chiếu mst_tenant_registry(id) |
| `billing_month` | VARCHAR(7) | No |  | IDX | Tháng tính cước (YYYY-MM) |
| `base_fee` | DECIMAL(12,2) | No | 0 |  | Phí duy trì nền tảng cơ bản |
| `usage_fee` | DECIMAL(12,2) | No | 0 |  | Phí phát sinh theo lượng sử dụng (User/Hợp đồng) |
| `total_amount` | DECIMAL(12,2) | No | 0 |  | Tổng tiền phải thu |
| `status` | VARCHAR(20) | No | 'DRAFT' |  | DRAFT, ISSUED, PAID, OVERDUE |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---