# Carima-Staffing — Data Dictionary / ERD

> Kiến trúc **multi-tenant, schema-per-tenant** trên PostgreSQL 16. Cô lập 2 lớp: schema riêng + Postgres role riêng mỗi tenant, và **Row-Level Security (RLS)** trên 7 bảng đọc/ghi chéo.

> - **Platform**: schema cố định `platform`, dùng chung.
> - **SAKI** (先 — công ty phái cử đối tác): `tenant_saki_<id>`.
> - **MOTO** (元 — công ty cung ứng nhân lực): `tenant_moto_<id>`.

> **Logical FK** = không ràng buộc vật lý xuyên schema, chỉ đánh index. **FK vật lý** chỉ trong cùng schema.


**Quy ước đọc:**

- Cột `Null` = `NO` nghĩa là **NOT NULL**; để trống = cho phép NULL.

- Kiểu `(IDENTITY)` = PostgreSQL `GENERATED ALWAYS AS IDENTITY` (tự tăng).

- Thời gian ca làm lưu bằng **SMALLINT = số phút từ 00:00** (ca đêm vắt ngày: 29:30 = 1770).

- Tiền tệ JPY dùng `DECIMAL(12,0)` (không phần thập phân).

- 4 cột audit chung: `created_at`, `updated_at` (TIMESTAMPTZ, default `now()`), `created_by`, `updated_by` (VARCHAR(100)).

- Kiểu dữ liệu hiển thị là **kiểu PostgreSQL cụ thể**. Một số cột ID dùng `CHAR(26)` (ULID), mã công ty `VARCHAR(20)`, tên schema/role `VARCHAR(63)` (giới hạn identifier của Postgres).


---

## Mục lục

- **Platform Schema**
  - Platform (Registry · Master · Notification · Billing)
- **SAKI Schema**
  - SAKI Master & Config
  - Request Job — 派遣照会・見積 (Yêu cầu phái cử & Báo giá)
- **MOTO Schema**
  - MOTO Master
  - Contract — 契約管理 (Quản lý hợp đồng)
  - Attendance — 勤怠管理 (Quản lý chấm công)
  - Billing — 請求管理 (Quản lý hóa đơn)
  - Shift & Batch (Carima nội bộ)

---


# Platform Schema

Vùng dùng chung cho mọi tenant (định tuyến, master tra cứu, thông báo, billing SaaS). Schema cố định `platform`.

## Platform (Registry · Master · Notification · Billing)

Toàn bộ bảng platform: đăng ký tenant, admin, master dùng chung, lịch sử thông báo, và billing/usage của chính SaaS Carima.


### `tenant_registry`

Quản lý vòng đời tenant trên toàn hệ thống


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `tenant_id` | CHAR(26) | NO |  | PK | Mã định danh tenant (ULID) — khóa chính, dùng định tuyến toàn hệ thống |
| 2 | `tenant_type` | VARCHAR(8) | NO |  |  | Loại tenant — 1:SAKI (派遣先) 2:MOTO (派遣元) |
| 3 | `company_id` | VARCHAR(20) | NO |  |  | Mã công ty (business key) tương ứng tenant này |
| 4 | `company_name` | VARCHAR(100) | NO |  |  | Tên công ty của tenant |
| 5 | `schema_name` | VARCHAR(63) | NO |  |  | Tên schema PostgreSQL riêng của tenant (vd tenant_saki_001) |
| 6 | `db_user` | VARCHAR(63) | NO |  |  | PostgreSQL role riêng của tenant (least-privilege, RLS áp dụng) |
| 7 | `status` | SMALLINT | NO | `1` |  | Trạng thái tenant — 0:vô hiệu 1:đang hoạt động |
| 8 | `plan_code` | VARCHAR(20) |  |  |  | Mã gói dịch vụ SaaS tenant đăng ký |
| 9 | `contracted_at` | DATE |  |  |  | Thời điểm ký hợp đồng dịch vụ |
| 10 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 11 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`tenant_id`)
- **UNIQUE** `uq_tenant_registry_schema`: (`schema_name`)
- **UNIQUE** `uq_tenant_registry_dbuser`: (`db_user`)
- **UNIQUE** `uq_tenant_registry_company`: (`tenant_type`, `company_id`)
- **INDEX** `idx_registry_type`: (`tenant_type`)
- **INDEX** `idx_registry_status`: (`status`)
- **INDEX** `idx_registry_company`: (`company_id`)


### `admin_user`

Tài khoản quản trị viên Platform (Carima)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `admin_user_id` | CHAR(26) | NO |  | PK | Mã quản trị viên Platform (ULID) — khóa chính |
| 2 | `login_id` | VARCHAR(100) | NO |  |  | Mã đăng nhập của quản trị viên |
| 3 | `password_hash` | VARCHAR(255) | NO |  |  | Hash mật khẩu (bcrypt) — cấm lưu dạng thô |
| 4 | `admin_name` | VARCHAR(100) | NO |  |  | Tên quản trị viên |
| 5 | `email` | VARCHAR(255) | NO |  |  | Email quản trị viên |
| 6 | `role` | SMALLINT | NO | `1` |  | Vai trò quản trị — phân quyền cấp Platform |
| 7 | `status` | SMALLINT | NO | `1` |  | Trạng thái tài khoản — 0:vô hiệu 1:hoạt động |
| 8 | `last_login_at` | TIMESTAMPTZ |  |  |  | Thời điểm đăng nhập gần nhất |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`admin_user_id`)
- **UNIQUE** `uq_admin_user_login`: (`login_id`)
- **UNIQUE** `uq_admin_user_email`: (`email`)


### `tenant_provision_log`

Nhật ký cấp phát/thay đổi tenant (audit Admin)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `log_id` | CHAR(26) | NO |  | PK | Mã bản ghi log (ULID) — khóa chính |
| 2 | `tenant_id` | CHAR(26) |  |  |  | Tenant liên quan đến thao tác cấp phát |
| 3 | `admin_user_id` | CHAR(26) |  |  |  | Quản trị viên thực hiện thao tác |
| 4 | `action` | VARCHAR(40) | NO |  |  | Loại thao tác — provision/teardown/migrate... |
| 5 | `detail` | JSONB |  |  |  | Chi tiết thao tác (JSON/text) |
| 6 | `occurred_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm xảy ra thao tác |

**Ràng buộc & Index:**

- **PK**: (`log_id`)
- **INDEX** `idx_provlog_tenant`: (`tenant_id`)
- **INDEX** `idx_provlog_time`: (`occurred_at`)


### `mst_moto_saki_relation`

Bản đồ quan hệ N–N giữa MOTO và SAKI


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `relation_id` | CHAR(26) | NO |  | PK | Mã quan hệ MOTO–SAKI (ULID) — khóa chính |
| 2 | `moto_tenant_id` | CHAR(26) | NO |  | FK | Tenant MOTO trong quan hệ (FK tenant_registry) |
| 3 | `saki_tenant_id` | CHAR(26) | NO |  | FK | Tenant SAKI trong quan hệ (FK tenant_registry) |
| 4 | `moto_company_id` | VARCHAR(20) | NO |  |  | Mã công ty MOTO (business key) |
| 5 | `saki_company_id` | VARCHAR(20) | NO |  |  | Mã công ty SAKI (business key) |
| 6 | `relation_status` | SMALLINT | NO | `1` |  | Trạng thái quan hệ — 1:đang hợp tác 0:đã chấm dứt |
| 7 | `established_date` | DATE |  |  |  | Ngày thiết lập quan hệ hợp tác |
| 8 | `terminated_date` | DATE |  |  |  | Ngày chấm dứt quan hệ (NULL nếu còn hiệu lực) |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`relation_id`)
- **UNIQUE** `uq_moto_saki_pair`: (`moto_tenant_id`, `saki_tenant_id`) — composite
- **FK** `fk_relation_moto`: (`moto_tenant_id`) → `platform.tenant_registry` (tenant_id)
- **FK** `fk_relation_saki`: (`saki_tenant_id`) → `platform.tenant_registry` (tenant_id)
- **INDEX** `idx_relation_moto`: (`moto_tenant_id`)
- **INDEX** `idx_relation_saki`: (`saki_tenant_id`)


### `mst_job_type`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `job_type_code` | VARCHAR(10) | NO |  | PK | Mã loại công việc (職種) — khóa chính |
| 2 | `name_ja` | VARCHAR(100) | NO |  |  | Tên loại công việc (tiếng Nhật) |
| 3 | `name_en` | VARCHAR(100) |  |  |  | Tên loại công việc (tiếng Anh) |
| 4 | `sort_order` | SMALLINT | NO | `0` |  | Thứ tự hiển thị |
| 5 | `is_active` | SMALLINT | NO | `1` |  | Cờ hiệu lực — 0:ẩn 1:đang dùng |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`job_type_code`)


### `mst_skill`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `skill_code` | VARCHAR(20) | NO |  | PK | Mã kỹ năng — khóa chính |
| 2 | `category` | VARCHAR(40) |  |  |  | Nhóm kỹ năng (ngoại ngữ/PC/nghiệp vụ...) |
| 3 | `name_ja` | VARCHAR(100) | NO |  |  | Tên kỹ năng (tiếng Nhật) |
| 4 | `name_en` | VARCHAR(100) |  |  |  | Tên kỹ năng (tiếng Anh) |
| 5 | `sort_order` | SMALLINT | NO | `0` |  | Thứ tự hiển thị |
| 6 | `is_active` | SMALLINT | NO | `1` |  | Cờ hiệu lực — 0:ẩn 1:đang dùng |
| 7 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 8 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`skill_code`)


### `mst_expense_category`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `expense_category` | VARCHAR(10) | NO |  | PK | Mã hạng mục chi phí — khóa chính |
| 2 | `name_ja` | VARCHAR(100) | NO |  |  | Tên hạng mục (tiếng Nhật) |
| 3 | `name_en` | VARCHAR(100) |  |  |  | Tên hạng mục (tiếng Anh) |
| 4 | `sort_order` | SMALLINT | NO | `0` |  | Thứ tự hiển thị |
| 5 | `is_active` | SMALLINT | NO | `1` |  | Cờ hiệu lực — 0:ẩn 1:đang dùng |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`expense_category`)


### `notification_history`

Lịch sử gửi thông báo đa kênh


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `notification_id` | BIGINT (IDENTITY) |  |  | PK | Mã thông báo (số tự tăng) — khóa chính |
| 2 | `relation_id` | CHAR(26) |  |  |  | Quan hệ MOTO–SAKI liên quan (nếu có) |
| 3 | `target_type` | VARCHAR(20) | NO |  |  | Loại đối tượng nhận — 1:SAKI 2:MOTO 3:Staff |
| 4 | `target_company_id` | VARCHAR(20) |  |  |  | Mã công ty nhận thông báo |
| 5 | `notification_type` | VARCHAR(50) | NO |  |  | Loại thông báo (nghiệp vụ phát sinh) |
| 6 | `send_status` | SMALLINT | NO | `0` |  | Trạng thái gửi — 0:chờ 1:đã gửi 2:thất bại |
| 7 | `title` | VARCHAR(255) | NO |  |  | Tiêu đề thông báo |
| 8 | `body_content` | TEXT |  |  |  | Nội dung thông báo |
| 9 | `sent_at` | TIMESTAMPTZ |  |  |  | Thời điểm gửi thực tế |
| 10 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 11 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`notification_id`)
- **INDEX (composite)** `idx_notif_target`: (`target_company_id`, `target_type`)


### `t_usage_billing`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `billing_id` | CHAR(26) | NO |  | PK | Mã hóa đơn sử dụng dịch vụ (ULID) — khóa chính |
| 2 | `tenant_id` | CHAR(26) | NO |  | FK | Tenant bị tính phí |
| 3 | `target_year_month` | CHAR(7) | NO |  |  | Tháng tính phí (YYYY/MM) |
| 4 | `base_fee` | DECIMAL(12,0) | NO | `0` |  | Phí cố định theo gói |
| 5 | `pay_per_use_fee` | DECIMAL(12,0) | NO | `0` |  | Phí theo mức sử dụng |
| 6 | `total_amount` | DECIMAL(12,0) | NO | `0` |  | Tổng tiền phải trả |
| 7 | `billing_status` | SMALLINT | NO | `0` |  | Trạng thái hóa đơn — 0:nháp 1:chốt 2:đã thu |
| 8 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 9 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 10 | `created_by` | VARCHAR(100) | NO |  |  | Tài khoản tạo bản ghi (phục vụ audit) |
| 11 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`billing_id`)
- **UNIQUE** `uq_usage_billing`: (`tenant_id`, `target_year_month`) — composite
- **FK** `fk_billing_tenant`: (`tenant_id`) → `platform.tenant_registry` (tenant_id)
- **INDEX** `idx_usage_billing_tenant`: (`tenant_id`)
- **INDEX** `idx_usage_billing_ym`: (`target_year_month`)
- **INDEX** `idx_usage_billing_status`: (`billing_status`)


### `t_tenant_usage_metric`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `metric_id` | BIGINT (IDENTITY) |  |  | PK | Mã chỉ số (số tự tăng) — khóa chính |
| 2 | `tenant_id` | CHAR(26) | NO |  | FK | Tenant được đo |
| 3 | `target_year_month` | CHAR(7) | NO |  |  | Tháng đo (YYYY/MM) |
| 4 | `metric_type` | VARCHAR(50) | NO |  |  | Loại chỉ số (số user, số hợp đồng, dung lượng...) |
| 5 | `metric_value` | INTEGER | NO | `0` |  | Giá trị chỉ số đo được |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |

**Ràng buộc & Index:**

- **PK**: (`metric_id`)
- **FK** `fk_metric_tenant`: (`tenant_id`) → `platform.tenant_registry` (tenant_id)
- **INDEX (composite)** `idx_usage_metric_tenant`: (`tenant_id`, `target_year_month`)
- **INDEX** `idx_usage_metric_type`: (`metric_type`)


# SAKI Schema

Schema riêng từng công ty phái cử đối tác (SAKI / 先). Khởi tạo động khi onboard: `tenant_saki_<id>`.

## SAKI Master & Config

Master và cấu hình nội bộ của một SAKI: công ty, phòng ban, văn phòng, nhân sự duyệt, cấu hình hiển thị/quyền.


### `tmp_saki_setup_credential`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK | Mã công ty — mã công ty phát hành đặt trước; đăng ký vào mst_saki_company sau khi hoàn tất khởi tạo |
| 2 | `temp_user_id` | VARCHAR(100) | NO |  |  | ID người dùng tạm — ID đăng nhập; ≥4 byte; chữ-số Half-width |
| 3 | `password_hash` | VARCHAR(255) | NO |  |  | Hash mật khẩu — giá trị hash (bcrypt...) |
| 4 | `setup_completed_flg` | SMALLINT | NO | `0` |  | Cờ hoàn tất khởi tạo — 0:chưa xong (chuyển bắt buộc tới màn hình khởi tạo) 1:xong (vô hiệu hóa) |
| 5 | `expires_at` | TIMESTAMPTZ | NO |  |  | Thời hạn hiệu lực — sau khi hết hạn không đăng nhập được |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 8 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 9 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)


### `mst_saki_company`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK | Mã công ty |
| 2 | `official_name_ja` | VARCHAR(100) | NO |  |  | Tên công ty chính thức (tiếng Nhật) |
| 3 | `official_name_kana` | VARCHAR(200) | NO |  |  | Tên công ty chính thức (katakana) |
| 4 | `display_name_ja` | VARCHAR(24) | NO |  |  | Tên công ty hiển thị (tiếng Nhật) |
| 5 | `postal_code` | CHAR(7) | NO |  |  | Mã bưu chính |
| 6 | `address_ja` | VARCHAR(50) | NO |  |  | Địa chỉ 1 |
| 7 | `address2_ja` | VARCHAR(50) |  |  |  | Địa chỉ 2 — nhập khi địa chỉ 1 không đủ ký tự, hoặc muốn xuống dòng |
| 8 | `official_name_en` | VARCHAR(100) |  |  |  | Tên công ty chính thức (tiếng Anh) — chữ-số-ký tự Half-width |
| 9 | `display_name_en` | VARCHAR(24) |  |  |  | Tên công ty hiển thị (tiếng Anh) |
| 10 | `treatment_priority_master` | SMALLINT |  |  |  | Master ưu tiên cấu hình thông tin đãi ngộ — 0:ưu tiên master công ty 1:ưu tiên master sự nghiệp sở |
| 11 | `equal_pay_method` | TEXT |  |  |  | Phương thức cân bằng đãi ngộ phía SAKI — 100 ký tự ngang (50 toàn giác) × 70 dòng; riêng theo công ty nên không di chuyển |
| 12 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 13 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 14 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 15 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)


### `mst_saki_office`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `office_id` | VARCHAR(20) | NO |  | PK | Mã sự nghiệp sở — duy nhất trong công ty; cho phép gạch nối; ≥4 byte |
| 3 | `official_name_ja` | VARCHAR(100) | NO |  |  | Tên sự nghiệp sở chính thức (tiếng Nhật) — tương đương 50 ký tự toàn giác |
| 4 | `display_name_ja` | VARCHAR(24) | NO |  |  | Tên sự nghiệp sở hiển thị (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 5 | `address_ja` | VARCHAR(100) | NO |  |  | Địa chỉ (tiếng Nhật) — tương đương 50 ký tự toàn giác |
| 6 | `official_name_en` | VARCHAR(100) |  |  |  | Tên sự nghiệp sở chính thức (tiếng Anh) — chữ-số-ký tự Half-width |
| 7 | `display_name_en` | VARCHAR(24) |  |  |  | Tên sự nghiệp sở hiển thị (tiếng Anh) |
| 8 | `address_en` | VARCHAR(100) |  |  |  | Địa chỉ (tiếng Anh) |
| 9 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 10 | `conflict_date_type` | SMALLINT | NO | `0` |  | Phân loại cấu hình ngày xung đột — 0:không cấu hình 1:đặt theo sự nghiệp sở 2:dùng sự nghiệp sở áp dụng ngày xung đột |
| 11 | `conflict_date` | DATE |  |  |  | Ngày xung đột sự nghiệp sở — đặt khi conflict_date_type=1 |
| 12 | `conflict_date_office_id` | VARCHAR(20) |  |  |  | Mã sự nghiệp sở áp dụng ngày xung đột — đặt khi conflict_date_type=2 |
| 13 | `treatment_setting_method` | SMALLINT |  |  |  | Cách cấu hình thông tin đãi ngộ — 0:cấu hình theo master 1:dùng thông tin đãi ngộ của sự nghiệp sở áp dụng ngày xung đột |
| 14 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 15 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 16 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 17 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `office_id`) — composite
- **FK** `fk_saki_office_company`: (`company_id`) → `mst_saki_company` (company_id)


### `mst_saki_department`

approval_group_id / reply_group_id FK con (không đầu PK)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `department_id` | VARCHAR(20) | NO |  | PK | Mã phòng ban — duy nhất trong công ty; cho phép gạch nối; ≥3 byte |
| 3 | `official_name_ja` | VARCHAR(400) | NO |  |  | Tên phòng ban chính thức (tiếng Nhật) — tương đương 200 ký tự toàn giác |
| 4 | `display_name_ja` | VARCHAR(24) | NO |  |  | Tên phòng ban hiển thị (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 5 | `official_name_en` | VARCHAR(100) |  |  |  | Tên phòng ban chính thức (tiếng Anh) — chữ-số-ký tự Half-width |
| 6 | `display_name_en` | VARCHAR(24) |  |  |  | Tên phòng ban hiển thị (tiếng Anh) |
| 7 | `tel` | VARCHAR(15) |  |  |  | Số điện thoại phòng ban — chữ-số-ký tự Half-width |
| 8 | `sub_code` | VARCHAR(16) |  |  |  | Mã phụ — chữ-số Half-width; dùng kiểm soát thứ tự khi tải hợp đồng |
| 9 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 10 | `approval_group_id` | VARCHAR(16) | NO |  |  | Mã nhóm nhận yêu cầu phê duyệt |
| 11 | `reply_group_id` | VARCHAR(16) | NO |  |  | Mã nhóm nhận trả lời |
| 12 | `org_unit_name` | VARCHAR(400) |  |  |  | Tên đơn vị tổ chức — tương đương 200 ký tự toàn giác |
| 13 | `org_head_title` | VARCHAR(200) |  |  |  | Chức danh người đứng đầu tổ chức — tương đương 100 ký tự toàn giác |
| 14 | `default_contract_person_id` | VARCHAR(100) |  |  |  | Người phụ trách hợp đồng (mặc định) |
| 15 | `default_dispatch_supervisor_id` | VARCHAR(100) |  |  |  | Người phụ trách SAKI (mặc định) |
| 16 | `default_commander_id` | VARCHAR(100) |  |  |  | Người chỉ huy (mặc định) |
| 17 | `default_invoice_recipient_id` | VARCHAR(100) |  |  |  | Nơi nhận hóa đơn (mặc định) |
| 18 | `default_complaint_person_id` | VARCHAR(100) |  |  |  | Nơi tiếp nhận khiếu nại (mặc định) |
| 19 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 20 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 21 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 22 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `department_id`) — composite
- **FK** `fk_saki_dept_company`: (`company_id`) → `mst_saki_company` (company_id)
- **INDEX (composite)** `idx_saki_dept_apprgrp`: (`company_id`, `approval_group_id`)
- **INDEX (composite)** `idx_saki_dept_replygrp`: (`company_id`, `reply_group_id`)


### `mst_saki_user`

company_id/office_id/department_id là FK con, hay filter theo tổ chức


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `user_id` | VARCHAR(100) | NO |  | PK | Mã người dùng — ID đăng nhập; ≥4 byte; chữ-số-ký tự Half-width (trừ ' " ,) |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Mã công ty |
| 3 | `office_id` | VARCHAR(20) | NO |  |  | Mã sự nghiệp sở — sự nghiệp sở trực thuộc |
| 4 | `department_id` | VARCHAR(20) | NO |  |  | Mã phòng ban — phòng ban trực thuộc |
| 5 | `last_name_ja` | VARCHAR(24) | NO |  |  | Họ (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 6 | `first_name_ja` | VARCHAR(24) | NO |  |  | Tên (tiếng Nhật) |
| 7 | `last_name_kana` | VARCHAR(24) | NO |  |  | Họ (katakana) |
| 8 | `first_name_kana` | VARCHAR(24) | NO |  |  | Tên (katakana) |
| 9 | `last_name_en` | VARCHAR(24) |  |  |  | Họ (tiếng Anh) — chữ-số-ký tự Half-width |
| 10 | `middle_name_en` | VARCHAR(12) |  |  |  | Tên đệm (tiếng Anh) |
| 11 | `first_name_en` | VARCHAR(24) |  |  |  | Tên (tiếng Anh) |
| 12 | `position_ja` | VARCHAR(50) |  |  |  | Chức vụ (tiếng Nhật) — tương đương 25 ký tự toàn giác |
| 13 | `position_en` | VARCHAR(50) |  |  |  | Chức vụ (tiếng Anh) — chữ-số-ký tự Half-width |
| 14 | `tel` | VARCHAR(15) | NO |  |  | Số điện thoại — chữ-số-ký tự Half-width |
| 15 | `fax` | VARCHAR(15) |  |  |  | Số FAX |
| 16 | `email` | VARCHAR(128) | NO |  |  | Email — cho phép gạch dưới |
| 17 | `mail_language` | CHAR(2) | NO | `'ja'` |  | Ngôn ngữ email thông báo — ja / en |
| 18 | `reference_scope` | SMALLINT | NO |  |  | Phạm vi tham chiếu — 1:mình phụ trách 2:phòng ban trực thuộc 3:sự nghiệp sở trực thuộc 4:sự nghiệp sở hoặc phòng ban 5:tất cả 6:tùy chọn (→mst_saki_user_scope) |
| 19 | `execution_role_id` | VARCHAR(64) | NO |  |  | Mã quyền thực thi — mã vai trò quyền thực thi chức năng (master phân quyền TBD) |
| 20 | `edit_on_approval_flg` | SMALLINT | NO | `0` |  | Cờ cho sửa thông tin khi duyệt hợp đồng — 0:không 1:được |
| 21 | `view_group_company_flg` | SMALLINT | NO | `0` |  | Cờ tham chiếu thông tin nhóm công ty |
| 22 | `edit_contract_dept_flg` | SMALLINT | NO | `0` |  | Cờ cho sửa phòng ban hiển thị trên hợp đồng |
| 23 | `approval_group_id` | VARCHAR(16) |  |  |  | Mã nhóm nhận yêu cầu phê duyệt |
| 24 | `show_as_approver_flg` | SMALLINT | NO | `0` |  | Cờ hiển thị ứng viên nơi gửi yêu cầu duyệt — 0:không 1:có |
| 25 | `final_approver_flg` | SMALLINT | NO | `0` |  | Cờ duyệt cuối |
| 26 | `select_dispatch_company_flg` | SMALLINT | NO | `0` |  | Cờ chọn công ty MOTO để nộp tới |
| 27 | `attendance_approver1_id` | VARCHAR(100) |  |  |  | Người duyệt chấm công 1 — tự đặt từ người chỉ huy (logic nghiệp vụ) |
| 28 | `attendance_approver2_id` | VARCHAR(100) |  |  |  | Người duyệt chấm công 2 |
| 29 | `attendance_approver3_id` | VARCHAR(100) |  |  |  | Người duyệt chấm công 3 |
| 30 | `cost_center_code` | VARCHAR(20) |  |  |  | Mã cost center — không dùng kana Half-width |
| 31 | `cost_center_comment` | VARCHAR(500) |  |  |  | Ghi chú mã cost center — tương đương 250 ký tự toàn giác |
| 32 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 33 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 34 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 35 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 36 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`user_id`)
- **FK** `fk_saki_user_company`: (`company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_saki_user_company`: (`company_id`)
- **INDEX** `idx_saki_user_office`: (`office_id`)
- **INDEX** `idx_saki_user_department`: (`department_id`)


### `mst_saki_user_scope`

user_id FK con


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `scope_id` | BIGINT (IDENTITY) |  |  | PK | Mã phạm vi (scope) |
| 2 | `user_id` | VARCHAR(100) | NO |  | FK | Mã người dùng |
| 3 | `scope_type` | SMALLINT | NO |  |  | Loại phạm vi — 1:sự nghiệp sở 2:phòng ban |
| 4 | `target_id` | VARCHAR(20) | NO |  |  | Mã đối tượng — scope_type=1→office_id, scope_type=2→department_id |
| 5 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 6 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 7 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 8 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`scope_id`)
- **FK** `fk_saki_scope_user`: (`user_id`) → `mst_saki_user` (user_id)
- **INDEX** `idx_saki_user_scope_user`: (`user_id`)


### `mst_saki_approval_group`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `approval_group_id` | VARCHAR(16) | NO |  | PK | Mã nhóm phê duyệt — duy nhất trong công ty, cho phép gạch nối |
| 3 | `group_name_ja` | VARCHAR(24) | NO |  |  | Tên nhóm phê duyệt (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 4 | `group_name_en` | VARCHAR(24) |  |  |  | Tên nhóm phê duyệt (tiếng Anh) |
| 5 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 8 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 9 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `approval_group_id`) — composite
- **FK** `fk_saki_apprgrp_company`: (`company_id`) → `mst_saki_company` (company_id)


### `mst_saki_approval_group_user`

user_id FK con (approval_group_id+user_id trong PK, company_id đầu PK)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `approval_group_id` | VARCHAR(16) | NO |  | PK FK | Mã nhóm phê duyệt |
| 3 | `user_id` | VARCHAR(100) | NO |  | PK FK | Mã người dùng |
| 4 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 5 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 6 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 7 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `approval_group_id`, `user_id`) — composite
- **FK** `fk_saki_apprgrpuser_group`: (`company_id`, `approval_group_id`) → `mst_saki_approval_group` (company_id, approval_group_id)
- **FK** `fk_saki_apprgrpuser_user`: (`user_id`) → `mst_saki_user` (user_id)
- **INDEX** `idx_saki_apprgrpuser_user`: (`user_id`)


### `mst_saki_custom_job_type`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `custom_job_type_id` | VARCHAR(16) | NO |  | PK | Mã loại công việc tùy biến — duy nhất trong công ty; chữ-số-ký tự Half-width (cho phép gạch nối); 1~16 byte |
| 3 | `job_type_name_ja` | VARCHAR(50) | NO |  |  | Tên loại công việc tùy biến (tiếng Nhật) — chưa phản ánh vào chứng từ; đặt tự do |
| 4 | `job_type_name_en` | VARCHAR(50) |  |  |  | Tên loại công việc tùy biến (tiếng Anh) — chữ-số-ký tự Half-width |
| 5 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 8 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 9 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `custom_job_type_id`) — composite
- **FK** `fk_saki_cjt_company`: (`company_id`) → `mst_saki_company` (company_id)


### `mst_saki_custom_job_type_skill`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `custom_job_type_id` | VARCHAR(16) | NO |  | PK FK | Mã loại công việc tùy biến |
| 3 | `skill_code` | VARCHAR(20) | NO |  | PK | Mã kỹ năng — xem phụ lục mã kỹ năng; chữ-số-ký tự Half-width |
| 4 | `skill_level` | SMALLINT | NO |  |  | Cấp độ kỹ năng — 1:không cần 2:chấp nhận được 3:cần 4:bắt buộc |
| 5 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 6 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 7 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 8 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `custom_job_type_id`, `skill_code`) — composite
- **FK** `fk_saki_cjtskill_cjt`: (`company_id`, `custom_job_type_id`) → `mst_saki_custom_job_type` (company_id, custom_job_type_id)


### `mst_saki_conflict_date_office`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `conflict_date_office_id` | VARCHAR(20) | NO |  | PK | Mã sự nghiệp sở áp dụng ngày xung đột — duy nhất trong công ty, cho phép gạch nối, ≥4 byte |
| 3 | `display_name_ja` | VARCHAR(24) | NO |  |  | Tên sự nghiệp sở hiển thị — tương đương 12 ký tự toàn giác |
| 4 | `address_ja` | VARCHAR(100) | NO |  |  | Địa chỉ (tiếng Nhật) — tương đương 50 ký tự toàn giác |
| 5 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 6 | `edit_note` | VARCHAR(200) |  |  |  | Ghi chú nội dung sửa — ghi chú loại thao tác sửa thông tin ngày xung đột |
| 7 | `conflict_date` | DATE | NO |  |  | Ngày xung đột sự nghiệp sở |
| 8 | `managed_office_id` | VARCHAR(16) |  |  |  | Mã sự nghiệp sở được quản lý — nơi quản lý ngày xung đột |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `conflict_date_office_id`) — composite
- **FK** `fk_saki_cdo_company`: (`company_id`) → `mst_saki_company` (company_id)


### `cfg_saki_work_schedule_default`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `schedule_default_id` | BIGINT (IDENTITY) |  |  | PK | Mã cấu hình hình thức làm việc mặc định |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Mã công ty |
| 3 | `parent_type` | SMALLINT | NO |  |  | Phân loại bảng cha — 1:master công ty 2:master phòng ban |
| 4 | `parent_id` | VARCHAR(20) | NO |  |  | Mã cha — parent_type=1→company_id, parent_type=2→department_id |
| 5 | `work_day_mon_flg` | SMALLINT | NO | `0` |  | Cờ thứ Hai là ngày làm — 0:nghỉ 1:đi làm |
| 6 | `work_day_tue_flg` | SMALLINT | NO | `0` |  | Cờ thứ Ba là ngày làm |
| 7 | `work_day_wed_flg` | SMALLINT | NO | `0` |  | Cờ thứ Tư là ngày làm |
| 8 | `work_day_thu_flg` | SMALLINT | NO | `0` |  | Cờ thứ Năm là ngày làm |
| 9 | `work_day_fri_flg` | SMALLINT | NO | `0` |  | Cờ thứ Sáu là ngày làm |
| 10 | `work_day_sat_flg` | SMALLINT | NO | `0` |  | Cờ thứ Bảy là ngày làm |
| 11 | `work_day_sun_flg` | SMALLINT | NO | `0` |  | Cờ Chủ nhật là ngày làm |
| 12 | `work_day_holiday_flg` | SMALLINT | NO | `0` |  | Cờ ngày lễ là ngày làm |
| 13 | `work_shift_flg` | SMALLINT |  |  |  | Cờ ca làm — 0:không 1:có |
| 14 | `work_start_time` | SMALLINT |  |  |  | Giờ bắt đầu làm — định dạng HH:MM (00:00~47:59) |
| 15 | `work_end_time` | SMALLINT |  |  |  | Giờ kết thúc làm |
| 16 | `break_start_time` | SMALLINT |  |  |  | Giờ bắt đầu nghỉ |
| 17 | `break_duration` | SMALLINT |  |  |  | Thời gian nghỉ (phút) — 0~180 phút |
| 18 | `smoking_type` | SMALLINT |  |  |  | Phân loại hút thuốc — 0:cấm hút 1:khu vực riêng 2:có giờ hút 3:được hút |
| 19 | `uniform_type` | SMALLINT |  |  |  | Phân loại đồng phục — 0:không 1:có |
| 20 | `cafeteria_type` | SMALLINT |  |  |  | Căng tin nhân viên — 0:không 1:có |
| 21 | `car_commute_type` | SMALLINT |  |  |  | Phân loại đi làm bằng xe — 0:không được 1:được 2:bắt buộc |
| 22 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 23 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 24 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 25 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`schedule_default_id`)
- **FK** `fk_saki_wsd_company`: (`company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_saki_wsd_company`: (`company_id`)


### `cfg_saki_treatment_info`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `treatment_info_id` | BIGINT (IDENTITY) |  |  | PK | Mã thông tin đãi ngộ |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Mã công ty |
| 3 | `parent_type` | SMALLINT | NO |  |  | Phân loại bảng cha — 1:master công ty 2:master sự nghiệp sở 3:master sự nghiệp sở áp dụng ngày xung đột |
| 4 | `parent_id` | VARCHAR(20) | NO |  |  | Mã cha — PK của bảng tương ứng parent_type |
| 5 | `training_flg` | SMALLINT |  |  |  | Cờ đào tạo — 0:không 1:có 9:chưa cấu hình |
| 6 | `training_ojt_procedure_flg` | SMALLINT |  |  |  | Đào tạo: OJT quy trình nghiệp vụ — 0:không 1:có |
| 7 | `training_ojt_industry_flg` | SMALLINT |  |  |  | Đào tạo: OJT kiến thức ngành |
| 8 | `training_ojt_system_flg` | SMALLINT |  |  |  | Đào tạo: OJT hệ thống nội bộ |
| 9 | `training_other` | VARCHAR(500) |  |  |  | Đào tạo: khác |
| 10 | `cafeteria_facility_flg` | SMALLINT |  |  |  | Nhà ăn — 0:không 1:có 9:chưa cấu hình |
| 11 | `rest_room_flg` | SMALLINT |  |  |  | Phòng nghỉ — 0:không 1:có 9:chưa cấu hình |
| 12 | `changing_room_flg` | SMALLINT |  |  |  | Phòng thay đồ — 0:không 1:có 9:chưa cấu hình |
| 13 | `benefit_clinic_flg` | SMALLINT |  |  |  | Tiện ích: cơ sở y tế — 0:không 1:có |
| 14 | `benefit_uniform_rental_flg` | SMALLINT |  |  |  | Tiện ích: cho mượn đồng phục |
| 15 | `benefit_resort_flg` | SMALLINT |  |  |  | Tiện ích: cơ sở nghỉ dưỡng |
| 16 | `benefit_shop_flg` | SMALLINT |  |  |  | Tiện ích: điểm bán hàng |
| 17 | `benefit_other1` | VARCHAR(100) |  |  |  | Tiện ích: khác 1 — tương đương 50 ký tự toàn giác |
| 18 | `benefit_other2` | VARCHAR(100) |  |  |  | Tiện ích: khác 2 |
| 19 | `benefit_other3` | VARCHAR(100) |  |  |  | Tiện ích: khác 3 |
| 20 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 21 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 22 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 23 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`treatment_info_id`)
- **FK** `fk_saki_ti_company`: (`company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_saki_treatment_company`: (`company_id`)


### `t_saki_opinion_hearing`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `opinion_hearing_id` | BIGINT (IDENTITY) |  |  | PK | Mã phiếu lấy ý kiến (意見聴取) — khóa chính |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Công ty SAKI sở hữu |
| 3 | `target_type` | SMALLINT | NO |  |  | Loại đối tượng lấy ý kiến |
| 4 | `target_id` | VARCHAR(20) | NO |  |  | Mã đối tượng được lấy ý kiến |
| 5 | `hearing_date` | DATE |  |  |  | Ngày tổ chức lấy ý kiến |
| 6 | `opinion_content` | TEXT |  |  |  | Nội dung ý kiến thu thập |
| 7 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 8 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 9 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 10 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`opinion_hearing_id`)
- **FK** `fk_saki_oh_company`: (`company_id`) → `mst_saki_company` (company_id)


### `cfg_saki_contract_item_control`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `cost_center_required_flg` | SMALLINT | NO | `0` |  | Cờ bắt buộc nhập cost center — 0:tùy chọn 1:bắt buộc |
| 3 | `cost_center_visible_to_vendor_flg` | SMALLINT | NO | `0` |  | Cờ hiển thị cost center cho công ty MOTO — 0:ẩn 1:hiện |
| 4 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 5 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 6 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 7 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_saki_cic_company`: (`company_id`) → `mst_saki_company` (company_id)


### `cfg_saki_contract_remarks_control`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `remarks_ctrl_id` | BIGINT (IDENTITY) |  |  | PK | Mã cấu hình ô ghi chú |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Mã công ty |
| 3 | `remarks_type` | SMALLINT | NO |  |  | Loại ô ghi chú — 1:ô ghi chú 2:ô ghi chú (mã) 3:ô ghi chú dạng chọn |
| 4 | `seq_no` | SMALLINT | NO |  |  | Số thứ tự — thứ tự hiển thị trong cùng remarks_type (bắt đầu từ 1) |
| 5 | `label` | VARCHAR(28) |  |  |  | Tên hiển thị — tương đương 14 ký tự toàn giác; nếu NULL hiển thị tên mặc định |
| 6 | `required_flg` | SMALLINT | NO | `0` |  | Cờ bắt buộc nhập — 0:tùy chọn 1:bắt buộc |
| 7 | `visible_to_vendor_flg` | SMALLINT | NO | `0` |  | Cờ hiển thị cho MOTO — 0:ẩn 1:hiện |
| 8 | `editable_by_vendor_flg` | SMALLINT | NO | `0` |  | Cờ cho phép MOTO sửa — 0:không 1:được |
| 9 | `code_char_type` | SMALLINT |  |  |  | Loại ký tự (chỉ ô mã) — chỉ dùng khi remarks_type=2; 0:không chỉ định 1:chữ số Half-width 2:chữ-số Half-width 3:chữ-số-ký hiệu Half-width |
| 10 | `code_max_length` | SMALLINT |  |  |  | Số chữ số tối đa (chỉ ô mã) — chỉ dùng khi remarks_type=2; 1~999 |
| 11 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 12 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 13 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 14 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`remarks_ctrl_id`)
- **FK** `fk_saki_crc_company`: (`company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_saki_remarks_company`: (`company_id`)


### `mst_saki_business_partner`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `partner_auto_id` | BIGINT (IDENTITY) |  |  | PK | Mã đối tác tự đánh số |
| 3 | `partner_company_name` | VARCHAR(255) | NO |  |  | Tên công ty đối tác — bảng nguồn tham chiếu TBD |
| 4 | `partner_id` | VARCHAR(20) |  |  |  | Mã đối tác — mã định danh do nội bộ quyết định; cho phép gạch nối |
| 5 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 6 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 7 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 8 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `partner_auto_id`) — composite
- **FK** `fk_saki_bp_company`: (`company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_saki_bp_company`: (`company_id`)


### `cfg_saki_permission_setting`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `role_type` | SMALLINT | NO |  | PK | Loại vai trò — 1:người phụ trách hợp đồng 2:người phụ trách SAKI 3:người chỉ huy 4:nơi nhận hóa đơn 5:nơi tiếp nhận khiếu nại 6:người duyệt chấm công 1 7:người duyệt chấm công 2 8:người duyệt chấm công 3 |
| 3 | `feature_type` | SMALLINT | NO |  | PK | Loại chức năng — 1:hợp đồng 2:chấm công liên kết hóa đơn 3:hóa đơn 4:chấm công (WebTimeCard) |
| 4 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 5 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 6 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 7 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `role_type`, `feature_type`) — composite
- **FK** `fk_saki_ps_company`: (`company_id`) → `mst_saki_company` (company_id)


### `cfg_saki_alarm_setting`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `alarm_setting_id` | BIGINT (IDENTITY) |  |  | PK | Mã cấu hình cảnh báo |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Mã công ty |
| 3 | `alarm_type` | SMALLINT | NO |  |  | Loại cảnh báo — 1:cảnh báo ngày xung đột cá nhân 2:cảnh báo ngày xung đột sự nghiệp sở |
| 4 | `send_flg` | SMALLINT | NO | `0` |  | Cờ gửi — 0:không gửi 1:gửi |
| 5 | `dest_estaffing1_flg` | SMALLINT | NO | `0` |  | Người nhận: phụ trách carima-staffing 1 (cờ) — 0:không áp dụng 1:áp dụng |
| 6 | `dest_estaffing2_flg` | SMALLINT | NO | `0` |  | Người nhận: phụ trách carima-staffing 2 (cờ) |
| 7 | `dest_contract_confirmer_flg` | SMALLINT | NO | `0` |  | Người nhận: người xác nhận hợp đồng (cờ) — chỉ hiệu lực khi alarm_type=1 |
| 8 | `dest_final_approver_flg` | SMALLINT | NO | `0` |  | Người nhận: người duyệt cuối (cờ) — chỉ hiệu lực khi alarm_type=1 |
| 9 | `dest_contract_person_flg` | SMALLINT | NO | `0` |  | Người nhận: người phụ trách hợp đồng (cờ) — chỉ hiệu lực khi alarm_type=1 |
| 10 | `dest_dispatch_supervisor_flg` | SMALLINT | NO | `0` |  | Người nhận: người phụ trách SAKI (cờ) — chỉ hiệu lực khi alarm_type=1 |
| 11 | `dest_commander_flg` | SMALLINT | NO | `0` |  | Người nhận: người chỉ huy (cờ) — chỉ hiệu lực khi alarm_type=1 |
| 12 | `proxy1_user_id` | VARCHAR(100) |  |  |  | User đại diện 1 — nơi gửi thay thế khi người nhận bị vô hiệu |
| 13 | `proxy2_user_id` | VARCHAR(100) |  |  |  | User đại diện 2 |
| 14 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 15 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 16 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 17 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`alarm_setting_id`)
- **FK** `fk_saki_as_company`: (`company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_saki_alarm_company`: (`company_id`)


### `cfg_saki_alarm_send_condition`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `condition_id` | BIGINT (IDENTITY) |  |  | PK | Mã điều kiện gửi |
| 2 | `alarm_setting_id` | BIGINT | NO |  | FK | Mã cấu hình cảnh báo |
| 3 | `send_condition_type` | SMALLINT | NO |  |  | Loại điều kiện gửi — 1:A (chỉ định khoảng cách/số lần) 2:B (chỉ định ngày) |
| 4 | `cond_a_days_before` | SMALLINT |  |  |  | [Điều kiện A] Số ngày tới ngày xung đột — 1~999; bắt buộc khi send_condition_type=1 |
| 5 | `cond_a_repeat_type` | SMALLINT |  |  |  | [Điều kiện A] Loại số lần cảnh báo — 0:chỉ một lần 1:lặp lại |
| 6 | `cond_a_interval_days` | SMALLINT |  |  |  | [Điều kiện A] Số ngày giữa các lần lặp — 1~99; khi chọn lặp lại |
| 7 | `cond_a_repeat_count` | SMALLINT |  |  |  | [Điều kiện A] Số lần lặp — 1~9; khi chọn lặp lại |
| 8 | `cond_b_months_ahead` | SMALLINT |  |  |  | [Điều kiện B] Số tháng trước khi gửi — 1~37; bắt buộc khi send_condition_type=2 |
| 9 | `cond_b_send_day_type` | SMALLINT |  |  |  | [Điều kiện B] Loại ngày gửi — 0:lặp mỗi tháng 1:chỉ lần đầu tiên |
| 10 | `cond_b_send_days_multi` | VARCHAR(100) |  |  |  | [Điều kiện B] Ngày gửi (cho phép nhiều) — khi lặp mỗi tháng; số ngày phân tách bằng dấu phẩy (0=cuối tháng), vd: 5,10,15,0 |
| 11 | `cond_b_send_day_single` | SMALLINT |  |  |  | [Điều kiện B] Ngày gửi (đơn) — cho lần đầu tiên; 1~31 hoặc 0 (cuối tháng) |
| 12 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 13 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 14 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 15 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`condition_id`)
- **FK** `fk_saki_asc_setting`: (`alarm_setting_id`) → `cfg_saki_alarm_setting` (alarm_setting_id)
- **INDEX** `idx_saki_alarm_cond_setting`: (`alarm_setting_id`)


### `cfg_saki_alarm_specified_user`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `specified_user_id` | BIGINT (IDENTITY) |  |  | PK | User được chỉ định |
| 2 | `alarm_setting_id` | BIGINT | NO |  | FK | Mã cấu hình cảnh báo |
| 3 | `user_id` | VARCHAR(100) | NO |  | FK | Mã người dùng |
| 4 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 5 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 6 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 7 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`specified_user_id`)
- **FK** `fk_saki_asu_setting`: (`alarm_setting_id`) → `cfg_saki_alarm_setting` (alarm_setting_id)
- **FK** `fk_saki_asu_user`: (`user_id`) → `mst_saki_user` (user_id)
- **INDEX** `idx_saki_alarm_spec_setting`: (`alarm_setting_id`)


### `cfg_saki_36_agreement_alarm`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `overtime_alarm_flg` | SMALLINT | NO | `0` |  | Cờ cảnh báo giờ làm thêm — 0:không gửi 1:gửi |
| 3 | `avg_month_alarm_flg` | SMALLINT | NO | `0` |  | Cờ cảnh báo giờ trung bình nhiều tháng |
| 4 | `annual_overtime_alarm_flg` | SMALLINT | NO | `0` |  | Cờ cảnh báo giờ làm thêm theo năm |
| 5 | `threshold_70_flg` | SMALLINT | NO | `0` |  | Cờ thông báo khi vượt 70% giờ giới hạn |
| 6 | `threshold_90_flg` | SMALLINT | NO | `0` |  | Cờ thông báo khi vượt 90% |
| 7 | `threshold_100_flg` | SMALLINT | NO | `0` |  | Cờ thông báo khi đạt 100% |
| 8 | `special_clause_notify_flg` | SMALLINT | NO | `0` |  | Cờ thông báo khi áp dụng điều khoản đặc biệt |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_saki_36aa_company`: (`company_id`) → `mst_saki_company` (company_id)


### `cfg_saki_holiday`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `holiday_setting_id` | BIGINT (IDENTITY) |  |  | PK | Mã cấu hình ngày nghỉ |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Mã công ty |
| 3 | `holiday_mon_flg` | SMALLINT |  |  |  | Cờ thứ Hai là ngày nghỉ — 0:đi làm 1:nghỉ |
| 4 | `holiday_tue_flg` | SMALLINT |  |  |  | Cờ thứ Ba là ngày nghỉ |
| 5 | `holiday_wed_flg` | SMALLINT |  |  |  | Cờ thứ Tư là ngày nghỉ |
| 6 | `holiday_thu_flg` | SMALLINT |  |  |  | Cờ thứ Năm là ngày nghỉ |
| 7 | `holiday_fri_flg` | SMALLINT |  |  |  | Cờ thứ Sáu là ngày nghỉ |
| 8 | `holiday_sat_flg` | SMALLINT |  |  |  | Cờ thứ Bảy là ngày nghỉ |
| 9 | `holiday_sun_flg` | SMALLINT |  |  |  | Cờ Chủ nhật là ngày nghỉ |
| 10 | `holiday_national_flg` | SMALLINT |  |  |  | Cờ ngày lễ là ngày nghỉ |
| 11 | `apply_start_date` | DATE | NO |  |  | Ngày bắt đầu áp dụng |
| 12 | `apply_end_date` | DATE | NO |  |  | Ngày kết thúc áp dụng |
| 13 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 14 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 15 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 16 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`holiday_setting_id`)
- **FK** `fk_saki_holiday_company`: (`company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_saki_holiday_company`: (`company_id`)


### `cfg_saki_specified_date`

holiday_setting_id FK con


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `specified_date_id` | BIGINT (IDENTITY) |  |  | PK | Mã ngày chỉ định |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Mã công ty |
| 3 | `holiday_setting_id` | BIGINT | NO |  | FK | Mã cấu hình ngày nghỉ |
| 4 | `date_type` | SMALLINT | NO |  |  | Phân loại ngày — 0:ngày nghỉ chỉ định 1:ngày làm chỉ định |
| 5 | `target_date` | DATE | NO |  |  | Ngày áp dụng |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 8 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 9 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`specified_date_id`)
- **FK** `fk_saki_sd_company`: (`company_id`) → `mst_saki_company` (company_id)
- **FK** `fk_saki_sd_holiday`: (`holiday_setting_id`) → `cfg_saki_holiday` (holiday_setting_id)
- **INDEX** `idx_saki_specdate_holiday`: (`holiday_setting_id`)

## Request Job — 派遣照会・見積 (Yêu cầu phái cử & Báo giá)

Nghiệp vụ SAKI khởi tạo yêu cầu phái cử, broadcast tới nhiều MOTO, thu thập & so sánh báo giá. SAKI sở hữu vì SAKI khởi tạo và tổng hợp. t_dispatch_estimate được MOTO ghi chéo (RLS theo moto_company_id).


### `t_dispatch_inquiry_draft`

bản nháp (JSON, 8 ngày)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `draft_id` | BIGINT (IDENTITY) |  |  | PK | Mã bản nháp |
| 2 | `saki_company_id` | VARCHAR(20) | NO |  | FK | Mã công ty SAKI |
| 3 | `requester_user_id` | VARCHAR(100) | NO |  |  | User người khởi tạo |
| 4 | `inquiry_type` | SMALLINT | NO |  |  | Loại yêu cầu — 0:yêu cầu báo giá 1:yêu cầu phái cử |
| 5 | `current_step` | SMALLINT | NO | `1` |  | Bước hiện tại — bước hiển thị khi tiếp tục (1~5) |
| 6 | `form_data` | JSONB | NO |  |  | Dữ liệu form — tuần tự hóa toàn bộ trạng thái nhập từ bước 1~4 |
| 7 | `saved_at` | TIMESTAMPTZ | NO |  |  | Thời điểm lưu — thời điểm lưu tạm |
| 8 | `expires_at` | TIMESTAMPTZ | NO |  |  | Thời hạn hiệu lực — saved_at + 8 ngày; bản ghi quá hạn bị batch xóa |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`draft_id`)
- **FK** `fk_draft_company`: (`saki_company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_draft_company`: (`saki_company_id`)
- **INDEX** `idx_draft_requester`: (`requester_user_id`)
- **INDEX** `idx_draft_expires`: (`expires_at`)


### `t_dispatch_inquiry`

bản ghi chính (sau submit)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `inquiry_id` | BIGINT (IDENTITY) |  |  | PK | Mã yêu cầu phái cử |
| 2 | `inquiry_no` | VARCHAR(20) | NO |  |  | Số yêu cầu — để hiển thị màn hình; hệ thống tự đánh số; UQ |
| 3 | `inquiry_type` | SMALLINT | NO |  |  | Loại yêu cầu — 0:yêu cầu báo giá 1:yêu cầu phái cử |
| 4 | `saki_company_id` | VARCHAR(20) | NO |  | FK | Mã công ty SAKI |
| 5 | `requester_user_id` | VARCHAR(100) | NO |  |  | User người khởi tạo (đồng thời là người yêu cầu) |
| 6 | `requester_department_id` | VARCHAR(20) | NO |  |  | Phòng ban của người khởi tạo |
| 7 | `requester_tel` | VARCHAR(15) |  |  |  | Điện thoại người khởi tạo (chỉ lần này) — đổi được ở bước 1; chỉ hiệu lực cho yêu cầu này; nếu NULL dùng điện thoại từ master người dùng |
| 8 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 1:đã nộp 2:đang duyệt 3:đang báo giá 4:đã báo giá 5:đang yêu cầu phái cử 6:từ chối 7:thu hồi 8:đóng |
| 9 | `submitted_at` | TIMESTAMPTZ | NO |  |  | Thời điểm nộp — thời điểm gửi yêu cầu duyệt cho người duyệt |
| 10 | `final_approved_at` | TIMESTAMPTZ |  |  |  | Thời điểm duyệt cuối — khi người duyệt cuối duyệt (cũng là thời điểm gửi cho MOTO) |
| 11 | `closed_at` | TIMESTAMPTZ |  |  |  | Thời điểm đóng — đóng thủ công hoặc tự động khi xác lập hợp đồng |
| 12 | `display_hidden_flg` | SMALLINT | NO | `0` |  | Cờ ẩn mục đã xử lý — 0:hiện trên màn hình 1:ẩn (đã xử lý); quay lại từ tìm kiếm sẽ về 0 |
| 13 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 14 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 15 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 16 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`inquiry_id`)
- **UNIQUE** `uq_dispatch_inquiry_no`: (`inquiry_no`)
- **FK** `fk_inquiry_company`: (`saki_company_id`) → `mst_saki_company` (company_id)
- **INDEX** `idx_inquiry_company`: (`saki_company_id`)
- **INDEX** `idx_inquiry_requester`: (`requester_user_id`)
- **INDEX** `idx_inquiry_dept`: (`requester_department_id`)
- **INDEX** `idx_inquiry_status`: (`status`)


### `t_dispatch_inquiry_condition`

điều kiện (1:1)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `inquiry_id` | BIGINT | NO |  | PK FK | Mã yêu cầu phái cử |
| 2 | `required_count` | SMALLINT | NO |  |  | Số người cần — bắt buộc |
| 3 | `work_description` | TEXT | NO |  |  | Nội dung nghiệp vụ — bắt buộc |
| 4 | `contract_start_date` | DATE | NO |  |  | Ngày bắt đầu hợp đồng — bắt buộc |
| 5 | `contract_end_date` | DATE | NO |  |  | Ngày kết thúc hợp đồng — bắt buộc |
| 6 | `work_location` | VARCHAR(200) | NO |  |  | Nơi làm việc — bắt buộc; tương đương 100 ký tự toàn giác |
| 7 | `work_time_start` | SMALLINT | NO |  |  | Giờ bắt đầu làm — bắt buộc |
| 8 | `work_time_end` | SMALLINT | NO |  |  | Giờ kết thúc làm — bắt buộc |
| 9 | `break_duration` | SMALLINT | NO |  |  | Thời gian nghỉ (phút) — bắt buộc |
| 10 | `request_background` | TEXT |  |  |  | Bối cảnh yêu cầu (tùy chọn) |
| 11 | `overtime_flg` | SMALLINT |  |  |  | Cờ có làm thêm hay không (tùy chọn) — 0:không 1:có |
| 12 | `overtime_hours_monthly` | SMALLINT |  |  |  | Ước tính giờ làm thêm trong tháng — khi overtime_flg=1 |
| 13 | `skill_requirements` | TEXT |  |  |  | Yêu cầu kỹ năng — tùy chọn; văn bản tự do |
| 14 | `remarks` | TEXT |  |  |  | Ghi chú khác (tùy chọn) |
| 15 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 16 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 17 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 18 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`inquiry_id`)
- **FK** `fk_cond_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)


### `t_dispatch_inquiry_job_type`

Loại công việc (職種) — 1:N


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `inquiry_id` | BIGINT | NO |  | PK FK | Mã yêu cầu phái cử |
| 2 | `job_type_code` | VARCHAR(20) | NO |  | PK | Mã loại công việc — master loại công việc TBD |
| 3 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 4 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 5 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 6 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`inquiry_id`, `job_type_code`) — composite
- **FK** `fk_jobtype_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)
- **INDEX** `idx_inq_jobtype_code`: (`job_type_code`)


### `t_dispatch_inquiry_work_skill`

Kỹ năng nghiệp vụ (業務スキル) — 1:N


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `inquiry_id` | BIGINT | NO |  | PK FK | Mã yêu cầu phái cử |
| 2 | `skill_code` | VARCHAR(20) | NO |  | PK | Mã kỹ năng nghiệp vụ — master kỹ năng nghiệp vụ TBD |
| 3 | `skill_level` | SMALLINT |  |  |  | Cấp độ kỹ năng — 1:bắt buộc 2:cần 3:chấp nhận được 4:không cần |
| 4 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 5 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 6 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 7 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`inquiry_id`, `skill_code`) — composite
- **FK** `fk_workskill_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)
- **INDEX** `idx_inq_workskill_code`: (`skill_code`)


### `t_dispatch_inquiry_lang_skill`

Kỹ năng ngoại ngữ / PC (語学・PCスキル) — 1:N


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `inquiry_id` | BIGINT | NO |  | PK FK | Mã yêu cầu phái cử |
| 2 | `skill_type` | SMALLINT | NO |  | PK | Loại kỹ năng — 1:ngoại ngữ 2:kỹ năng PC |
| 3 | `skill_code` | VARCHAR(20) | NO |  | PK | Mã kỹ năng — mã ngoại ngữ (TOEIC...) hoặc mã kỹ năng PC; master TBD |
| 4 | `level_code` | VARCHAR(10) |  |  |  | Mã cấp độ — hệ thống cấp độ TBD |
| 5 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 6 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 7 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 8 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`inquiry_id`, `skill_type`, `skill_code`) — composite
- **FK** `fk_langskill_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)


### `t_dispatch_inquiry_office_env`

môi trường văn phòng (1:1)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `inquiry_id` | BIGINT | NO |  | PK FK | Mã yêu cầu phái cử |
| 2 | `smoking_type` | SMALLINT |  |  |  | Phân loại hút thuốc — 0:cấm hút 1:khu vực riêng 2:có giờ hút 3:được hút; master TBD |
| 3 | `nearest_station` | VARCHAR(100) |  |  |  | Ga/tuyến gần nhất — tương đương 50 ký tự toàn giác |
| 4 | `commute_minutes` | SMALLINT |  |  |  | Thời gian từ ga (phút) — đi bộ/xe buýt... |
| 5 | `parking_flg` | SMALLINT |  |  |  | Cờ bãi đỗ xe — 0:không 1:có |
| 6 | `dress_code` | VARCHAR(200) |  |  |  | Quy định trang phục — tương đương 100 ký tự toàn giác |
| 7 | `other_environment` | TEXT |  |  |  | Ghi chú môi trường khác |
| 8 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 9 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 10 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 11 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`inquiry_id`)
- **FK** `fk_officeenv_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)


### `t_dispatch_inquiry_target_company`

Công ty MOTO được hỏi (照会先派遣会社) — 1:N


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `inquiry_id` | BIGINT | NO |  | PK FK | Mã yêu cầu phái cử |
| 2 | `moto_company_id` | VARCHAR(20) | NO |  | PK | Mã công ty MOTO — chỉ chọn được công ty MOTO có quan hệ giao dịch với SAKI |
| 3 | `company_status` | SMALLINT | NO | `0` |  | Trạng thái theo từng công ty — 0:chưa gửi 1:chưa trả lời (đã gửi, MOTO chưa trả lời) 2:đang trả lời (ứng viên dưới định mức) 3:đã trả lời (đủ định mức) 4:đóng 5:từ chối |
| 4 | `inquiry_revision_flg` | SMALLINT | NO | `0` |  | Cờ phân loại — 0:tạo mới 1:đổi nội dung (SAKI đã đổi nội dung và gửi lại); phản ánh ở cột "phân loại" trong danh sách trả lời của MOTO |
| 5 | `sent_at` | TIMESTAMPTZ |  |  |  | Thời điểm gửi cho MOTO — đặt khi hoàn tất duyệt cuối |
| 6 | `closed_at` | TIMESTAMPTZ |  |  |  | Thời điểm đóng — đóng thủ công hoặc tự động khi xác lập hợp đồng |
| 7 | `close_comment` | VARCHAR(500) |  |  |  | Ghi chú khi đóng — SAKI nhập khi đóng; hiển thị ở màn hình tiến trình/kết quả trả lời phía MOTO |
| 8 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 9 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 10 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 11 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`inquiry_id`, `moto_company_id`) — composite
- **FK** `fk_target_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)
- **INDEX** `idx_target_moto`: (`moto_company_id`)
- **INDEX** `idx_target_status`: (`company_status`)


### `t_dispatch_inquiry_approval`

luồng duyệt nội bộ SAKI (1:N)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `approval_id` | BIGINT (IDENTITY) |  |  | PK | Mã phê duyệt |
| 2 | `inquiry_id` | BIGINT | NO |  | FK | Mã yêu cầu phái cử |
| 3 | `seq_no` | SMALLINT | NO |  |  | Thứ tự phê duyệt — số thứ tự bắt đầu từ 1 |
| 4 | `approver_type` | SMALLINT | NO |  |  | Loại người duyệt — 1:chỉ định user 2:chỉ định nhóm |
| 5 | `approver_user_id` | VARCHAR(100) |  |  |  | User người duyệt — khi approver_type=1 |
| 6 | `approver_group_id` | VARCHAR(16) |  |  |  | Mã nhóm phê duyệt — khi approver_type=2 |
| 7 | `is_final_flg` | SMALLINT | NO | `0` |  | Cờ người duyệt cuối — 1:là người duyệt cuối; khi duyệt sẽ gửi cho công ty MOTO |
| 8 | `approval_status` | SMALLINT | NO | `0` |  | Trạng thái phê duyệt — 0:chưa xử lý 1:đã duyệt 2:từ chối |
| 9 | `reject_comment` | TEXT |  |  |  | Ghi chú từ chối — ghi chú gửi cho người khởi tạo khi từ chối; không gửi cho công ty MOTO |
| 10 | `processed_at` | TIMESTAMPTZ |  |  |  | Thời điểm xử lý — thời điểm duyệt hoặc từ chối |
| 11 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 12 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 13 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 14 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`approval_id`)
- **FK** `fk_approval_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)
- **INDEX** `idx_inqappr_inquiry`: (`inquiry_id`)
- **INDEX** `idx_inqappr_approver`: (`approver_user_id`)


### `t_dispatch_estimate`

Trả lời báo giá (見積回答) — MOTO ghi chéo vào schema SAKI


> 🔒 **RLS bật** (FORCE). Backend phải set `app.current_saki`/`app.current_moto`/`app.current_role` trước khi query.


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `estimate_id` | BIGINT (IDENTITY) |  |  | PK | Mã trả lời báo giá — tương ứng "số trả lời" trên màn hình |
| 2 | `inquiry_id` | BIGINT | NO |  | FK | Mã yêu cầu phái cử |
| 3 | `moto_company_id` | VARCHAR(20) | NO |  |  | Mã công ty MOTO |
| 4 | `unit_price` | DECIMAL(12,0) |  |  |  | Đơn giá — yêu cầu báo giá: số tiền báo giá; yêu cầu phái cử: đơn giá hóa đơn |
| 5 | `unit_type` | SMALLINT |  |  |  | Đơn vị đơn giá — 0:giờ 1:ngày 2:tháng; dùng khi unit_price đã được đặt |
| 6 | `estimate_valid_until` | DATE |  |  |  | Thời hạn hiệu lực báo giá — thời hạn hiệu lực của trả lời báo giá; nhập tùy chọn |
| 7 | `candidate_count` | SMALLINT |  |  |  | Số ứng viên — đặt khi trả lời yêu cầu phái cử; số người đề xuất trong lần trả lời này |
| 8 | `response_comment` | TEXT |  |  |  | Ghi chú — ghi chú/điều kiện... từ công ty MOTO |
| 9 | `response_status` | SMALLINT | NO | `1` |  | Trạng thái trả lời — 1:đã trả lời 2:từ chối 3:đã hủy |
| 10 | `responded_at` | TIMESTAMPTZ | NO |  |  | Thời điểm trả lời |
| 11 | `cancel_count` | SMALLINT |  |  |  | Số người hủy — nhập khi hủy trả lời; số ứng viên rút lại |
| 12 | `cancel_comment` | TEXT |  |  |  | Ghi chú khi hủy — ghi chú tùy chọn khi hủy trả lời |
| 13 | `canceled_at` | TIMESTAMPTZ |  |  |  | Thời điểm hủy — đặt khi response_status=3 |
| 14 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 15 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 16 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 17 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`estimate_id`)
- **FK** `fk_estimate_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)
- **INDEX** `idx_estimate_inquiry`: (`inquiry_id`)
- **INDEX** `idx_estimate_moto`: (`moto_company_id`)
- **RLS policy** — p_estimate_moto_isolation: USING ( moto_company_id = current_setting('app.current_moto', true) ) WITH CHECK ( moto_company_id = current_setting('app.current_moto', true) )
- **RLS policy** — p_estimate_saki_owner: USING ( current_setting('app.current_role', true) = 'saki_owner' ) WITH CHECK ( current_setting('app.current_role', true) = 'saki_owner' )


### `t_dispatch_inquiry_revision_history`

lịch sử chỉnh sửa (1:N)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `revision_history_id` | BIGINT (IDENTITY) |  |  | PK | Mã bản ghi lịch sử chỉnh sửa — khóa chính |
| 2 | `inquiry_id` | BIGINT | NO |  | FK | Yêu cầu phái cử được chỉnh sửa |
| 3 | `revision_no` | SMALLINT | NO |  |  | Số lần chỉnh sửa (tăng dần) |
| 4 | `trigger_type` | SMALLINT | NO |  |  | Nguyên nhân chỉnh sửa (sửa nội dung/từ chối...) |
| 5 | `triggered_by_approval_id` | BIGINT |  |  | FK | Bản ghi duyệt gây ra chỉnh sửa (nếu có) |
| 6 | `form_snapshot` | JSONB |  |  |  | Ảnh chụp nội dung form tại thời điểm sửa (JSON) |
| 7 | `resubmitted_at` | TIMESTAMPTZ |  |  |  | Thời điểm gửi lại sau chỉnh sửa |
| 8 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 9 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 10 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 11 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`revision_history_id`)
- **FK** `fk_revhist_inquiry`: (`inquiry_id`) → `t_dispatch_inquiry` (inquiry_id)
- **FK** `fk_revhist_approval`: (`triggered_by_approval_id`) → `t_dispatch_inquiry_approval` (approval_id)
- **INDEX** `idx_revhist_inquiry`: (`inquiry_id`)


### `t_saki_contract_upload_history`

Lịch sử SAKI upload TSV (xác nhận hàng loạt 一括確認)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `upload_id` | BIGINT (IDENTITY) |  |  | PK | Mã upload — hệ thống tự đánh số |
| 2 | `uploader_user_id` | VARCHAR(100) | NO |  |  | User upload — có thể là user MOTO hoặc SAKI |
| 3 | `upload_contract_type` | SMALLINT | NO |  |  | Phân loại hợp đồng — MOTO: 1:tạo mới 2:gia hạn 3:sửa 4:từ trả lời / SAKI: 1:tạo mới 2:gia hạn |
| 4 | `file_name` | VARCHAR(255) | NO |  |  | Tên file — tên file TSV đã upload |
| 5 | `total_count` | INTEGER | NO | `0` |  | Tổng số dòng — số dòng dữ liệu của file TSV |
| 6 | `success_count` | INTEGER | NO | `0` |  | Số thành công |
| 7 | `error_count` | INTEGER | NO | `0` |  | Số lỗi |
| 8 | `process_status` | SMALLINT | NO | `1` |  | Trạng thái xử lý — 1:đang xử lý 2:hoàn tất 3:lỗi |
| 9 | `error_detail` | TEXT |  |  |  | Chi tiết lỗi — nội dung lỗi kiểm tra hợp lệ (số dòng, tên trường, thông điệp) |
| 10 | `uploaded_at` | TIMESTAMPTZ | NO |  |  | Thời điểm upload |
| 11 | `completed_at` | TIMESTAMPTZ |  |  |  | Thời điểm xử lý xong |
| 12 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 13 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 14 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 15 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`upload_id`)
- **INDEX** `idx_saki_upload_uploader`: (`uploader_user_id`)


### `t_saki_batch_execution_history`

lịch sử job nền/export của SAKI


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `job_id` | BIGINT (IDENTITY) |  |  | PK | Mã job batch (số tự tăng) — khóa chính |
| 2 | `job_type` | VARCHAR(50) | NO |  |  | Loại job (export/tổng hợp...) |
| 3 | `execute_user_id` | VARCHAR(100) | NO |  |  | Người chạy job |
| 4 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 1:đang chạy 2:hoàn tất 3:lỗi |
| 5 | `file_url` | VARCHAR(255) |  |  |  | Đường dẫn file kết quả (nếu có) |
| 6 | `started_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm bắt đầu |
| 7 | `finished_at` | TIMESTAMPTZ |  |  |  | Thời điểm kết thúc |
| 8 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |

**Ràng buộc & Index:**

- **PK**: (`job_id`)
- **INDEX** `idx_saki_batch_type`: (`job_type`)
- **INDEX** `idx_saki_batch_user`: (`execute_user_id`)


# MOTO Schema

Schema riêng từng công ty cung ứng nhân lực (MOTO / 元). Khởi tạo động: `tenant_moto_<id>`. Chứa toàn bộ dữ liệu vận hành nặng nhất.

## MOTO Master & Config

Master nội bộ MOTO: công ty, văn phòng, nhân viên (staff), chứng chỉ, ngày xung đột (conflict date), cấu hình.


### `tmp_moto_setup_credential`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK | Mã công ty — mã công ty phát hành đặt trước; đăng ký vào mst_moto_company sau khi hoàn tất khởi tạo |
| 2 | `tmp_user_id` | VARCHAR(16) | NO |  |  | ID người dùng tạm — ID đăng nhập; ≥4 byte; chữ-số Half-width |
| 3 | `password_hash` | VARCHAR(255) | NO |  |  | Hash mật khẩu — giá trị hash (bcrypt...); cùng giá trị với ID người dùng tạm |
| 4 | `setup_completed_flg` | SMALLINT | NO | `0` |  | Cờ hoàn tất khởi tạo — 0:chưa xong (chuyển bắt buộc tới màn hình khởi tạo) 1:xong (vô hiệu hóa) |
| 5 | `expires_at` | TIMESTAMPTZ | NO |  |  | Thời hạn hiệu lực — sau khi hết hạn không đăng nhập được |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 8 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 9 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)


### `mst_moto_company`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK | Mã công ty — hệ thống tự đánh số |
| 2 | `official_name_ja` | VARCHAR(100) | NO |  |  | Tên công ty chính thức (tiếng Nhật) — tương đương 50 ký tự toàn giác |
| 3 | `official_name_kana` | VARCHAR(200) | NO |  |  | Tên công ty chính thức (katakana) — tương đương 100 ký tự toàn giác |
| 4 | `display_name_ja` | VARCHAR(24) | NO |  |  | Tên công ty hiển thị (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 5 | `postal_code` | CHAR(7) | NO |  |  | Mã bưu chính — 7 chữ số, không gạch nối |
| 6 | `address_ja` | VARCHAR(50) | NO |  |  | Địa chỉ (tiếng Nhật) 1 — tương đương 25 ký tự toàn giác |
| 7 | `address2_ja` | VARCHAR(50) |  |  |  | Địa chỉ (tiếng Nhật) 2 — nhập khi địa chỉ 1 không đủ ký tự, hoặc muốn xuống dòng |
| 8 | `official_name_en` | VARCHAR(100) | NO |  |  | Tên công ty chính thức (tiếng Anh) — chữ-số-ký tự Half-width |
| 9 | `display_name_en` | VARCHAR(24) | NO |  |  | Tên công ty hiển thị (tiếng Anh) |
| 10 | `dispatch_permit_first` | CHAR(2) |  |  |  | Số giấy phép phái cử (phần đầu) — chỉ chữ số; khi quản lý theo đơn vị công ty |
| 11 | `dispatch_permit_last` | CHAR(6) |  |  |  | Số giấy phép phái cử (phần sau) — chỉ chữ số |
| 12 | `manage_permit_by_office_flg` | SMALLINT |  | `0` |  | Cờ quản lý số giấy phép theo sự nghiệp sở — 0:quản lý ở master công ty 1:quản lý ở master sự nghiệp sở |
| 13 | `qualified_invoice_status` | SMALLINT | NO | `0` |  | Tình trạng đăng ký hóa đơn hợp lệ — 0:không có số đăng ký 1:có số đăng ký |
| 14 | `qualified_invoice_no` | CHAR(13) |  |  |  | Số đăng ký doanh nghiệp phát hành hóa đơn hợp lệ — chỉ chữ số; bắt buộc khi qualified_invoice_status=1 |
| 15 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 16 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 17 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 18 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)


### `mst_moto_company_contact`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `seq_no` | SMALLINT | NO |  | PK | Số thứ tự người phụ trách — 1 hoặc 2 |
| 3 | `department` | VARCHAR(100) |  |  |  | Đơn vị trực thuộc — tương đương 50 ký tự toàn giác |
| 4 | `name_ja` | VARCHAR(48) | NO |  |  | Họ tên (tiếng Nhật) — tương đương 24 ký tự toàn giác |
| 5 | `name_kana` | VARCHAR(48) |  |  |  | Họ tên (katakana) |
| 6 | `tel` | VARCHAR(15) | NO |  |  | Số điện thoại — chữ-số-ký tự Half-width |
| 7 | `email` | VARCHAR(128) | NO |  |  | Email |
| 8 | `remarks` | VARCHAR(500) |  |  |  | Ghi chú — tương đương 250 ký tự toàn giác |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `seq_no`) — composite
- **FK** `fk_moto_contact_company`: (`company_id`) → `mst_moto_company` (company_id)


### `mst_moto_office`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `office_id` | VARCHAR(20) | NO |  | PK | Mã sự nghiệp sở — duy nhất trong công ty; ≥4 byte; cho phép gạch nối |
| 3 | `official_name_ja` | VARCHAR(100) | NO |  |  | Tên sự nghiệp sở chính thức (tiếng Nhật) — tương đương 50 ký tự toàn giác |
| 4 | `display_name_ja` | VARCHAR(24) | NO |  |  | Tên sự nghiệp sở hiển thị (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 5 | `address_ja` | VARCHAR(100) | NO |  |  | Địa chỉ (tiếng Nhật) — tương đương 50 ký tự toàn giác |
| 6 | `official_name_en` | VARCHAR(100) |  |  |  | Tên sự nghiệp sở chính thức (tiếng Anh) — chữ-số-ký tự Half-width |
| 7 | `display_name_en` | VARCHAR(24) |  |  |  | Tên sự nghiệp sở hiển thị (tiếng Anh) |
| 8 | `address_en` | VARCHAR(100) |  |  |  | Địa chỉ (tiếng Anh) |
| 9 | `dispatch_permit_first` | CHAR(2) |  |  |  | Số giấy phép phái cử (phần đầu) — dùng khi mst_moto_company.manage_permit_by_office_flg=1 |
| 10 | `dispatch_permit_last` | CHAR(6) |  |  |  | Số giấy phép phái cử (phần sau) |
| 11 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 12 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 13 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 14 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 15 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `office_id`) — composite
- **FK** `fk_moto_office_company`: (`company_id`) → `mst_moto_company` (company_id)


### `mst_moto_department`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `department_id` | VARCHAR(20) | NO |  | PK | Mã phòng ban — duy nhất trong công ty; ≥3 byte; cho phép gạch nối |
| 3 | `official_name_ja` | VARCHAR(100) | NO |  |  | Tên phòng ban chính thức (tiếng Nhật) — tương đương 50 ký tự toàn giác |
| 4 | `display_name_ja` | VARCHAR(24) | NO |  |  | Tên phòng ban hiển thị (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 5 | `official_name_en` | VARCHAR(100) |  |  |  | Tên phòng ban chính thức (tiếng Anh) — chữ-số-ký tự Half-width |
| 6 | `display_name_en` | VARCHAR(24) |  |  |  | Tên phòng ban hiển thị (tiếng Anh) |
| 7 | `tel` | VARCHAR(15) |  |  |  | Số điện thoại phòng ban — chữ-số-ký tự Half-width |
| 8 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `department_id`) — composite
- **FK** `fk_moto_dept_company`: (`company_id`) → `mst_moto_company` (company_id)


### `mst_moto_user`

company_id/office_id/department_id FK con (PK là user_id đơn)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `user_id` | VARCHAR(16) | NO |  | PK | Mã người dùng — ID đăng nhập; ≥4 byte; chữ-số Half-width |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Mã công ty |
| 3 | `last_name_ja` | VARCHAR(24) | NO |  |  | Họ (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 4 | `first_name_ja` | VARCHAR(24) | NO |  |  | Tên (tiếng Nhật) |
| 5 | `last_name_kana` | VARCHAR(24) | NO |  |  | Họ (katakana) |
| 6 | `first_name_kana` | VARCHAR(24) | NO |  |  | Tên (katakana) |
| 7 | `last_name_en` | VARCHAR(24) | NO |  |  | Họ (tiếng Anh) — chữ-số-ký tự Half-width |
| 8 | `middle_name_en` | VARCHAR(12) |  |  |  | Tên đệm (tiếng Anh) |
| 9 | `first_name_en` | VARCHAR(24) | NO |  |  | Tên (tiếng Anh) |
| 10 | `office_id` | VARCHAR(20) | NO |  |  | Mã sự nghiệp sở — sự nghiệp sở trực thuộc |
| 11 | `department_id` | VARCHAR(20) | NO |  |  | Mã phòng ban — phòng ban trực thuộc |
| 12 | `position_ja` | VARCHAR(50) |  |  |  | Chức vụ (tiếng Nhật) — tương đương 25 ký tự toàn giác |
| 13 | `position_en` | VARCHAR(50) |  |  |  | Chức vụ (tiếng Anh) — chữ-số-ký tự Half-width |
| 14 | `tel` | VARCHAR(15) | NO |  |  | Số điện thoại — chữ-số-ký tự Half-width |
| 15 | `fax` | VARCHAR(15) |  |  |  | Số FAX |
| 16 | `email` | VARCHAR(128) | NO |  |  | Email |
| 17 | `client_code` | VARCHAR(20) |  |  |  | Mã SAKI giao dịch — khi giới hạn SAKI phụ trách |
| 18 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 19 | `reference_scope` | SMALLINT | NO |  |  | Phạm vi tham chiếu — 0:chỉ SAKI giao dịch 5:tham chiếu tất cả |
| 20 | `execution_role` | SMALLINT | NO |  |  | Quyền thực thi — mã vai trò phân quyền (chi tiết TBD) |
| 21 | `special_contract_edit_flg` | SMALLINT | NO | `0` |  | Cờ cho sửa hợp đồng đặc biệt — 0:không 1:có |
| 22 | `invoice_download_flg` | SMALLINT | NO | `0` |  | Cờ cho tải hóa đơn — 0:không 1:được |
| 23 | `default_sales_user_id` | VARCHAR(16) |  |  |  | Nhân viên kinh doanh MOTO (mặc định) — giá trị mặc định khi yêu cầu hợp đồng |
| 24 | `default_manager_user_id` | VARCHAR(16) |  |  |  | Người phụ trách MOTO (mặc định) |
| 25 | `default_complaint_user_id` | VARCHAR(16) |  |  |  | Nơi tiếp nhận khiếu nại MOTO (mặc định) |
| 26 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 27 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 28 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 29 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`user_id`)
- **FK** `fk_moto_user_company`: (`company_id`) → `mst_moto_company` (company_id)
- **INDEX** `idx_moto_user_company`: (`company_id`)
- **INDEX** `idx_moto_user_office`: (`office_id`)
- **INDEX** `idx_moto_user_department`: (`department_id`)
- **INDEX** `idx_moto_user_client`: (`client_code`)


### `mst_moto_attendance_category`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Công ty MOTO sở hữu |
| 2 | `category_code` | VARCHAR(16) | NO |  | PK | Mã phân loại chấm công |
| 3 | `category_name` | VARCHAR(100) | NO |  |  | Tên phân loại (đi làm, nghỉ, ca bù...) |
| 4 | `sort_order` | SMALLINT | NO | `0` |  | Thứ tự hiển thị |
| 5 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:đang dùng |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 8 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 9 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `category_code`) — composite
- **FK** `fk_moto_atc_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_client_company`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `client_code` | VARCHAR(20) | NO |  | PK | Mã client — chữ-số Half-width |
| 3 | `display_name_ja` | VARCHAR(100) | NO |  |  | Tên hiển thị công ty đối tác — bảng nguồn tham chiếu TBD (để hiển thị) |
| 4 | `receive_email` | VARCHAR(128) | NO |  |  | Email nhận — chữ-số-ký tự Half-width |
| 5 | `test_mail_send_flg` | SMALLINT | NO | `0` |  | Cờ gửi email thử — 0:không gửi 1:gửi |
| 6 | `remarks` | VARCHAR(24) |  |  |  | Ghi chú — tương đương 12 ký tự toàn giác |
| 7 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 8 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 9 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 10 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `client_code`) — composite
- **FK** `fk_moto_cc_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_client_company_user`

client_code, user_id FK con


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `client_code` | VARCHAR(20) | NO |  | PK FK | Mã client |
| 3 | `user_id` | VARCHAR(16) | NO |  | PK FK | Mã người dùng |
| 4 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 5 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 6 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 7 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `client_code`, `user_id`) — composite
- **FK** `fk_moto_ccu_client`: (`company_id`, `client_code`) → `cfg_moto_client_company` (company_id, client_code)
- **FK** `fk_moto_ccu_user`: (`user_id`) → `mst_moto_user` (user_id)
- **INDEX (composite)** `idx_moto_ccu_client`: (`company_id`, `client_code`)
- **INDEX** `idx_moto_ccu_user`: (`user_id`)


### `mst_moto_contract_company`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `contract_company_code` | VARCHAR(16) | NO |  | PK | Mã đối tác hợp đồng — chữ-số-ký tự Half-width |
| 3 | `official_name_ja` | VARCHAR(100) | NO |  |  | Tên công ty ký hợp đồng chính thức (tiếng Nhật) |
| 4 | `official_name_en` | VARCHAR(100) | NO |  |  | Tên công ty ký hợp đồng chính thức (tiếng Anh) — chữ-số-ký tự Half-width |
| 5 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:hiệu lực |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 8 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 9 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `contract_company_code`) — composite
- **FK** `fk_moto_cont_comp_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_contract_default`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `contract_no_display_pattern` | SMALLINT | NO | `0` |  | Mẫu hiển thị số hợp đồng — cấu hình hiển thị chứng từ; 0:chỉ Job code 1:Job code + mã nhân viên |
| 3 | `fee_display_flg` | SMALLINT | NO | `0` |  | Cờ hiển thị phí phái cử — cấu hình hiển thị chứng từ; 0:ẩn 1:hiện |
| 4 | `social_insurance_days` | SMALLINT | NO |  |  | Số ngày thủ tục tham gia bảo hiểm xã hội — giá trị nhập mặc định; chỉ chữ số |
| 5 | `work_address_ja` | VARCHAR(100) |  |  |  | Địa chỉ nơi làm việc (tiếng Nhật) — giá trị nhập mặc định; tương đương 50 ký tự toàn giác |
| 6 | `treatment_method` | VARCHAR(100) |  |  |  | Phương thức quyết định đãi ngộ — giá trị nhập mặc định |
| 7 | `training_flg` | SMALLINT |  |  |  | Cờ đào tạo — giá trị nhập mặc định; 0:không 1:có |
| 8 | `training_ojt_procedure_flg` | SMALLINT |  |  |  | Đào tạo: OJT quy trình nghiệp vụ (cờ) — giá trị nhập mặc định |
| 9 | `training_ojt_knowledge_flg` | SMALLINT |  |  |  | Đào tạo: OJT kiến thức ngành (cờ) — giá trị nhập mặc định |
| 10 | `training_ojt_system_flg` | SMALLINT |  |  |  | Đào tạo: OJT hệ thống nội bộ (cờ) — giá trị nhập mặc định |
| 11 | `training_other` | VARCHAR(500) |  |  |  | Đào tạo: khác — giá trị nhập mặc định; tương đương 250 ký tự toàn giác |
| 12 | `cafeteria_flg` | SMALLINT |  |  |  | Cờ nhà ăn — giá trị nhập mặc định; 0:không 1:có |
| 13 | `rest_room_flg` | SMALLINT |  |  |  | Cờ phòng nghỉ — giá trị nhập mặc định |
| 14 | `changing_room_flg` | SMALLINT |  |  |  | Cờ phòng thay đồ — giá trị nhập mặc định |
| 15 | `benefit_clinic_flg` | SMALLINT |  |  |  | Tiện ích: cơ sở y tế (cờ) — giá trị nhập mặc định |
| 16 | `benefit_uniform_flg` | SMALLINT |  |  |  | Tiện ích: cho mượn đồng phục (cờ) — giá trị nhập mặc định |
| 17 | `benefit_resort_flg` | SMALLINT |  |  |  | Tiện ích: cơ sở nghỉ dưỡng (cờ) — giá trị nhập mặc định |
| 18 | `benefit_shop_flg` | SMALLINT |  |  |  | Tiện ích: điểm bán hàng (cờ) — giá trị nhập mặc định |
| 19 | `benefit_other1` | VARCHAR(100) |  |  |  | Tiện ích: khác 1 — giá trị nhập mặc định; tương đương 50 ký tự toàn giác |
| 20 | `benefit_other2` | VARCHAR(100) |  |  |  | Tiện ích: khác 2 — giá trị nhập mặc định |
| 21 | `benefit_other3` | VARCHAR(100) |  |  |  | Tiện ích: khác 3 — giá trị nhập mặc định |
| 22 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 23 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 24 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 25 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_cd_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_contract_additional_text`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `complaint_text` | TEXT |  |  |  | Văn bản bổ sung xử lý khiếu nại — tương đương 1000 ký tự toàn giác |
| 3 | `mid_cancel_text` | TEXT |  |  |  | Văn bản bổ sung khi chấm dứt hợp đồng phái cử giữa chừng |
| 4 | `dispute_general_text` | TEXT |  |  |  | Văn bản bổ sung biện pháp phòng tranh chấp (phái cử thông thường) |
| 5 | `dispute_referral_use_flg` | SMALLINT | NO | `0` |  | Cờ dùng câu chữ hợp đồng phái cử dự kiến giới thiệu — 0:không dùng 1:dùng |
| 6 | `dispute_referral_text` | TEXT |  |  |  | Văn bản bổ sung biện pháp phòng tranh chấp (dự kiến giới thiệu) |
| 7 | `safety_text` | VARCHAR(500) |  |  |  | Văn bản bổ sung an toàn và vệ sinh — tương đương 250 ký tự toàn giác |
| 8 | `special_terms_text` | TEXT |  |  |  | Văn bản bổ sung điều khoản đặc ước khác — 15000 ký tự toàn giác (150 ngang × 200 dòng) |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_cat_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_contract_text_per_client`

contract_company_code FK con (không đầu PK)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `contract_company_code` | VARCHAR(16) | NO |  | PK FK | Mã đối tác hợp đồng |
| 3 | `complaint_override_flg` | SMALLINT | NO | `0` |  | Cờ ưu tiên xử lý khiếu nại — 0:dùng cấu hình chung 1:ưu tiên cấu hình theo đối tác |
| 4 | `complaint_text` | TEXT |  |  |  | Văn bản bổ sung xử lý khiếu nại |
| 5 | `mid_cancel_override_flg` | SMALLINT |  |  |  | Cờ ưu tiên văn bản chấm dứt giữa chừng |
| 6 | `mid_cancel_text` | TEXT |  |  |  | Văn bản bổ sung chấm dứt giữa chừng |
| 7 | `dispute_general_override_flg` | SMALLINT |  |  |  | Cờ ưu tiên văn bản phòng tranh chấp (phái cử thông thường) |
| 8 | `dispute_general_text` | TEXT |  |  |  | Văn bản bổ sung phòng tranh chấp (phái cử thông thường) |
| 9 | `dispute_referral_override_flg` | SMALLINT |  |  |  | Cờ ưu tiên văn bản phòng tranh chấp (dự kiến giới thiệu) |
| 10 | `dispute_referral_use_flg` | SMALLINT |  |  |  | Cờ dùng câu chữ hợp đồng phái cử dự kiến giới thiệu — 0:giống phái cử thường 1:dùng câu chữ khác |
| 11 | `dispute_referral_text` | TEXT |  |  |  | Văn bản bổ sung phòng tranh chấp (dự kiến giới thiệu) |
| 12 | `special_terms_override_flg` | SMALLINT |  |  |  | Cờ ưu tiên điều khoản đặc ước |
| 13 | `special_terms_text` | TEXT |  |  |  | Văn bản bổ sung điều khoản đặc ước khác |
| 14 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 15 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 16 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 17 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `contract_company_code`) — composite
- **FK** `fk_moto_ctpc_company`: (`company_id`) → `mst_moto_company` (company_id)
- **FK** `fk_moto_ctpc_contract_company`: (`company_id`, `contract_company_code`) → `mst_moto_contract_company` (company_id, contract_company_code)
- **INDEX (composite)** `idx_moto_ctpc_contcomp`: (`company_id`, `contract_company_code`)


### `cfg_moto_invoice_default`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `postal_code` | CHAR(7) | NO |  |  | Mã bưu chính — chỉ chữ số |
| 3 | `address_ja` | VARCHAR(50) | NO |  |  | Địa chỉ (tiếng Nhật) 1 |
| 4 | `address2_ja` | VARCHAR(50) |  |  |  | Địa chỉ (tiếng Nhật) 2 |
| 5 | `contact_display_name` | VARCHAR(40) |  |  |  | Tên hiển thị nơi liên hệ — tương đương 20 ký tự toàn giác |
| 6 | `contact_tel` | VARCHAR(15) |  |  |  | Số điện thoại nơi liên hệ — chữ-số-ký tự Half-width |
| 7 | `invoice_date_default` | VARCHAR(50) |  |  |  | Cấu hình giá trị mặc định ngày hóa đơn — giá trị hiển thị mặc định (TBD) |
| 8 | `invoice_target_month_default` | VARCHAR(50) |  |  |  | Cấu hình giá trị mặc định năm-tháng tính hóa đơn |
| 9 | `invoice_target_period_default` | VARCHAR(50) |  |  |  | Cấu hình giá trị mặc định kỳ tính hóa đơn |
| 10 | `payment_due_default` | VARCHAR(50) |  |  |  | Cấu hình giá trị mặc định hạn thanh toán |
| 11 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 12 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 13 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 14 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_id_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_invoice_calculation`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `invoice_format` | SMALLINT | NO | `0` |  | Định dạng hóa đơn — 0:định dạng truyền thống 1:định dạng hóa đơn hợp lệ |
| 3 | `provisional_calc_flg` | SMALLINT | NO | `0` |  | Cờ tính tạm — 0:không 1:có |
| 4 | `premium_rate_overtime` | SMALLINT |  |  |  | Tỷ lệ phụ trội làm thêm (%) — chỉ chữ số |
| 5 | `premium_rate_holiday` | SMALLINT |  |  |  | Tỷ lệ phụ trội làm ngày nghỉ (%) |
| 6 | `premium_rate_legal_holiday` | SMALLINT |  |  |  | Tỷ lệ phụ trội làm ngày nghỉ pháp định (%) |
| 7 | `premium_rate_night` | SMALLINT |  |  |  | Tỷ lệ phụ trội ca đêm (%) |
| 8 | `premium_rate_display_flg` | SMALLINT | NO | `0` |  | Cờ hiển thị tỷ lệ phụ trội — 0:ẩn 1:hiện |
| 9 | `work_time_calc_method` | SMALLINT | NO |  |  | Cách tính giờ làm việc — cách tính khi nhập chấm công từ WebTimeCard (các lựa chọn TBD) |
| 10 | `tax_calc_flg` | SMALLINT | NO |  |  | Cờ tính thuế tiêu thụ |
| 11 | `tax_fraction` | SMALLINT |  |  |  | Làm tròn thuế tiêu thụ (※1) — dùng khi invoice_format=1; 0:làm tròn lên 1:làm tròn xuống 2:làm tròn |
| 12 | `transport_tax_convert_flg` | SMALLINT |  |  |  | Cờ chuyển đổi thuế tiêu thụ phí đi lại (※1) — dùng khi invoice_format=1; 0:không chuyển 1:chuyển thuế ngoài |
| 13 | `transport_tax_fraction` | SMALLINT |  |  |  | Làm tròn phần lẻ phí đi lại (※1) — dùng khi invoice_format=1 |
| 14 | `transport_tax_same_flg` | SMALLINT |  |  |  | Cờ tính cùng cách cho phí đi lại (※2) — dùng khi invoice_format=0 |
| 15 | `transport_tax_fraction2` | SMALLINT |  |  |  | Làm tròn phần lẻ phí đi lại (※2) — dùng khi invoice_format=0 |
| 16 | `tax_calc_method` | SMALLINT |  |  |  | Cách áp dụng thuế suất tiêu thụ (※2) — dùng khi invoice_format=0 |
| 17 | `tax_target_subtotal_flg` | SMALLINT |  |  |  | Cờ đối tượng tính: tạm tính hóa đơn (※2) — dùng khi invoice_format=0 |
| 18 | `tax_target_adjustment1_flg` | SMALLINT |  |  |  | Cờ đối tượng tính: khoản điều chỉnh đặc biệt 1 (※2) — dùng khi invoice_format=0 |
| 19 | `tax_target_adjustment2_flg` | SMALLINT |  |  |  | Cờ đối tượng tính: khoản điều chỉnh đặc biệt 2 (※2) — dùng khi invoice_format=0 |
| 20 | `tax_item_subtotal_method` | VARCHAR(50) |  |  |  | Tạm tính: cách áp dụng thuế suất tiêu thụ (※2) — dùng khi invoice_format=0 |
| 21 | `tax_fraction_total` | SMALLINT |  |  |  | Làm tròn phần lẻ: tổng cộng (※2) — dùng khi invoice_format=0 |
| 22 | `tax_fraction_subtotal` | SMALLINT |  |  |  | Làm tròn phần lẻ: tạm tính (※2) — dùng khi invoice_format=0 |
| 23 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 24 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 25 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 26 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_ic_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_invoice_fraction`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `fraction_type` | SMALLINT | NO |  | PK | Loại làm tròn phần lẻ — 1:trong hợp đồng 2:làm thêm 3:làm ngày nghỉ 4:làm ngày nghỉ pháp định 5:ca đêm 6:khấu trừ |
| 3 | `method` | SMALLINT | NO |  |  | Cách làm tròn phần lẻ — 0:làm tròn lên 1:làm tròn xuống 2:làm tròn |
| 4 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 5 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 6 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 7 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `fraction_type`) — composite
- **FK** `fk_moto_if_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_bank_account`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `account_seq` | SMALLINT | NO |  | PK | Số thứ tự tài khoản — 1~6 |
| 3 | `bank_name` | VARCHAR(100) |  |  |  | Tên ngân hàng — tương đương 50 ký tự toàn giác |
| 4 | `bank_branch_name` | VARCHAR(100) |  |  |  | Tên chi nhánh |
| 5 | `bank_account_type` | SMALLINT |  |  |  | Loại tài khoản — 0:thường 1:vãng lai |
| 6 | `bank_account_no` | VARCHAR(7) |  |  |  | Số tài khoản — chữ-số Half-width |
| 7 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 8 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 9 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 10 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `account_seq`) — composite
- **FK** `fk_moto_ba_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_seal`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `vertical_position` | DECIMAL(4,1) |  |  |  | Vị trí dọc (mm) — -10.0~+10.0 |
| 3 | `horizontal_position` | DECIMAL(4,1) |  |  |  | Vị trí ngang (mm) — -10.0~+10.0 |
| 4 | `size_correction_flg` | SMALLINT | NO | `0` |  | Cờ hiệu chỉnh kích cỡ — 0:không 1:tự động hiệu chỉnh |
| 5 | `seal_image` | VARCHAR(255) |  |  |  | Ảnh con dấu — định dạng JPEG; có thể quản lý theo đường dẫn file (TBD) |
| 6 | `invoice_seal_output_type` | SMALLINT | NO | `0` |  | Phân loại in con dấu hóa đơn — 0:không in 1:mọi hóa đơn 2:chỉ SAKI đang giao dịch |
| 7 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 8 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 9 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 10 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_seal_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_seal_per_client`

client_code FK con (composite PK company+client; company đầu PK)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `client_code` | VARCHAR(20) | NO |  | PK FK | Mã client |
| 3 | `seal_override_flg` | SMALLINT | NO | `0` |  | Cờ áp dụng cấu hình con dấu — 0:dùng cấu hình chung 1:ưu tiên cấu hình theo từng đối tác |
| 4 | `seal_display_flg` | SMALLINT | NO | `0` |  | Cờ hiển thị con dấu — 0:không hiện 1:hiện |
| 5 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 6 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 7 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 8 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `client_code`) — composite
- **FK** `fk_moto_spc_client`: (`company_id`, `client_code`) → `cfg_moto_client_company` (company_id, client_code)
- **INDEX (composite)** `idx_moto_spc_client`: (`company_id`, `client_code`)


### `cfg_moto_advance_payment_screen`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `apply_seq` | SMALLINT | NO |  | PK | Số thứ tự áp dụng |
| 3 | `screen_type` | SMALLINT | NO | `0` |  | Loại màn hình — 0:màn hình thường 1:màn hình áp dụng chế độ hóa đơn (invoice) |
| 4 | `apply_start_date` | DATE | NO |  |  | Ngày bắt đầu áp dụng |
| 5 | `apply_end_date` | DATE | NO |  |  | Ngày kết thúc áp dụng |
| 6 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 7 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 8 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 9 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `apply_seq`) — composite
- **FK** `fk_moto_aps_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_36_agreement`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `agreement_seq` | SMALLINT | NO |  | PK | Số thứ tự hiệp định — 1~20; chỉ số 1 là bắt buộc |
| 3 | `agreement_use_flg` | SMALLINT | NO | `0` |  | Cờ sử dụng — 0:không dùng 1:dùng |
| 4 | `business_type` | VARCHAR(140) | NO |  |  | Loại sự nghiệp sở và nghiệp vụ |
| 5 | `valid_period_start_year` | CHAR(4) | NO |  |  | Năm bắt đầu kỳ hiệu lực — chỉ chữ số |
| 6 | `valid_period_end_year` | CHAR(4) | NO |  |  | Năm kết thúc kỳ hiệu lực |
| 7 | `overtime_limit_flg` | SMALLINT | NO | `1` |  | Cờ áp dụng quy định giới hạn — 0:miễn áp dụng 1:áp dụng |
| 8 | `overtime_daily_hour` | SMALLINT | NO |  |  | Giờ giới hạn 1 ngày (đơn vị giờ) — chỉ chữ số |
| 9 | `overtime_daily_min` | SMALLINT | NO |  |  | Phút giới hạn 1 ngày |
| 10 | `overtime_monthly_hour` | SMALLINT | NO |  |  | Giờ giới hạn 1 tháng (đơn vị giờ) — dưới 100 giờ |
| 11 | `overtime_monthly_min` | SMALLINT | NO |  |  | Phút giới hạn 1 tháng |
| 12 | `overtime_yearly_hour` | SMALLINT | NO |  |  | Giờ giới hạn 1 năm (đơn vị giờ) — trong vòng 720 giờ |
| 13 | `overtime_yearly_min` | SMALLINT | NO |  |  | Phút giới hạn 1 năm |
| 14 | `calc_start_year` | CHAR(4) | NO |  |  | Năm tính mốc theo năm — chỉ chữ số |
| 15 | `calc_start_day` | CHAR(2) | NO |  |  | Ngày tính mốc — chỉ chữ số |
| 16 | `calc_start_weekday` | SMALLINT |  |  |  | Thứ tính mốc — khi tính theo tuần; 0:Chủ nhật ~ 6:thứ Bảy |
| 17 | `holiday_work_period` | SMALLINT | NO |  |  | Kỳ áp dụng lao động ngày nghỉ — chỉ chữ số |
| 18 | `holiday_work_limit` | SMALLINT | NO |  |  | Lao động ngày nghỉ — giờ hoặc số ngày giới hạn |
| 19 | `special_clause_flg` | SMALLINT | NO | `0` |  | Cờ cấu hình điều khoản đặc biệt — 0:không 1:có |
| 20 | `special_clause_count` | SMALLINT |  |  |  | Điều khoản đặc biệt: số lần được gia hạn — tối đa 6 lần |
| 21 | `special_clause_monthly_hour` | SMALLINT |  |  |  | Điều khoản đặc biệt: giờ giới hạn 1 tháng (đơn vị giờ) — dưới 100 giờ |
| 22 | `special_clause_monthly_min` | SMALLINT |  |  |  | Điều khoản đặc biệt: phút giới hạn 1 tháng |
| 23 | `special_clause_yearly_hour` | SMALLINT |  |  |  | Điều khoản đặc biệt: giờ giới hạn 1 năm (đơn vị giờ) — trong vòng 720 giờ |
| 24 | `special_clause_yearly_min` | SMALLINT |  |  |  | Điều khoản đặc biệt: phút giới hạn 1 năm |
| 25 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 26 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 27 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 28 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `agreement_seq`) — composite
- **FK** `fk_moto_36a_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_36_agreement_free`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `free_business_type` | VARCHAR(140) |  |  |  | Loại sự nghiệp sở và nghiệp vụ (nhập tự do) |
| 3 | `free_agreement_content` | TEXT |  |  |  | Nội dung nhập tự do Hiệp định 36 — 150 ký tự ngang × 3 dòng (225 ký tự toàn giác) |
| 4 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 5 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 6 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 7 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_36af_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_36_overtime_start`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `overtime_yearly_start_month` | SMALLINT | NO |  |  | Tháng bắt đầu tổng hợp — 1~12 |
| 3 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 4 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 5 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 6 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_36os_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_legal_holiday`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `legal_holiday_type` | SMALLINT | NO |  |  | Cách xác định ngày nghỉ pháp định — 1:thứ cố định 2:mỗi tuần 1 lần 3:mỗi tuần 1 lần hoặc 4 lần/4 tuần |
| 3 | `legal_holiday_weekday` | SMALLINT |  |  |  | Ngày nghỉ pháp định: thứ cụ thể — 0:Chủ nhật ~ 6:thứ Bảy; khi legal_holiday_type=1 |
| 4 | `week_start_weekday` | SMALLINT |  |  |  | Thứ bắt đầu tuần (khi đặt thứ cố định) — chỉ khi legal_holiday_type=1 |
| 5 | `weekly_start_weekday` | SMALLINT |  |  |  | Thứ bắt đầu tuần (mỗi tuần 1 lần) — khi legal_holiday_type=2 |
| 6 | `four_week_start_date` | DATE |  |  |  | Ngày tính mốc 4 tuần — khi legal_holiday_type=2 hoặc 3 |
| 7 | `legal_holiday_judge_method` | SMALLINT |  |  |  | Cách tự xác định ngày nghỉ pháp định — khi legal_holiday_type=2 hoặc 3 (các lựa chọn TBD) |
| 8 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 9 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 10 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 11 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_lh_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_compensatory_holiday`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `compensatory_holiday_period` | VARCHAR(100) | NO |  |  | Kỳ cấu hình nghỉ bù — chỉ định kỳ hiệu lực được nghỉ bù (chi tiết TBD) |
| 3 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 4 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 5 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 6 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_ch_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_alarm_conflict`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `send_flg` | SMALLINT | NO | `0` |  | Cờ gửi — 0:không gửi 1:gửi |
| 3 | `send_condition_type` | SMALLINT |  |  |  | Loại điều kiện gửi — 1:A (chỉ định khoảng cách/số lần) 2:B (chỉ định ngày) |
| 4 | `cond_a_days_before` | SMALLINT |  |  |  | [Điều kiện A] Số ngày tới ngày xung đột — 1~999 |
| 5 | `cond_a_repeat_type` | SMALLINT |  |  |  | [Điều kiện A] Loại số lần cảnh báo — 0:chỉ một lần 1:lặp lại |
| 6 | `cond_a_interval_days` | SMALLINT |  |  |  | [Điều kiện A] Số ngày giữa các lần lặp — 1~99 |
| 7 | `cond_a_repeat_count` | SMALLINT |  |  |  | [Điều kiện A] Số lần lặp — 1~9 |
| 8 | `cond_b_months_ahead` | SMALLINT |  |  |  | [Điều kiện B] Số tháng trước khi gửi — 1~37 |
| 9 | `cond_b_send_day_type` | SMALLINT |  |  |  | [Điều kiện B] Loại ngày gửi — 0:chỉ lần đầu tiên 1:lặp mỗi tháng |
| 10 | `cond_b_send_days_multi` | VARCHAR(100) |  |  |  | [Điều kiện B] Ngày gửi (cho phép nhiều) — khi lặp mỗi tháng; số ngày phân tách bằng dấu phẩy (0=cuối tháng) |
| 11 | `cond_b_send_day_single` | SMALLINT |  |  |  | [Điều kiện B] Ngày gửi (đơn) — cho lần đầu tiên; 1~31 hoặc 0 (cuối tháng) |
| 12 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 13 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 14 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 15 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_ac_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_alarm_36`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `send_flg` | SMALLINT | NO | `0` |  | Cờ gửi — 0:không gửi 1:gửi |
| 3 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 4 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 5 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 6 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`)
- **FK** `fk_moto_a36_company`: (`company_id`) → `mst_moto_company` (company_id)


### `cfg_moto_alarm_36_recipient`

user_id FK con (type=4)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Mã công ty |
| 2 | `recipient_type` | SMALLINT | NO |  | PK | Loại người nhận — 1:nhân viên kinh doanh 2:người phụ trách MOTO 3:nơi tiếp nhận khiếu nại 4:user cụ thể |
| 3 | `send_flg` | SMALLINT | NO | `0` |  | Cờ gửi — 0:không gửi 1:gửi |
| 4 | `threshold_flags` | SMALLINT |  |  |  | Cờ ngưỡng điều kiện gửi — bitmask: bit0=50% bit1=60% bit2=70% bit3=80% bit4=90% bit5=100% |
| 5 | `frequency_type` | SMALLINT |  |  |  | Tần suất gửi — 0:mỗi ngày 1:chỉ một lần |
| 6 | `user_id` | VARCHAR(16) |  |  |  | User được chỉ định — chỉ dùng khi recipient_type=4 |
| 7 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 8 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 9 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 10 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `recipient_type`) — composite
- **FK** `fk_moto_a36r_company`: (`company_id`) → `mst_moto_company` (company_id)
- **INDEX** `idx_moto_a36r_user`: (`user_id`)


### `mst_staff`

moto_company_id (FK con), moto_office_id — hay filter staff theo công ty/sự nghiệp sở


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `staff_code` | VARCHAR(16) | NO |  | PK | Mã nhân viên (staff) — duy nhất trong hệ thống; MOTO tự đặt hoặc hệ thống đánh số (TBD); chữ-số Half-width, cho phép gạch nối |
| 2 | `moto_company_id` | VARCHAR(20) | NO |  | FK | Mã công ty MOTO trực thuộc — công ty tuyển dụng nhân viên |
| 3 | `moto_office_id` | VARCHAR(20) |  |  |  | Mã sự nghiệp sở trực thuộc — đặt khi có sự nghiệp sở phụ trách |
| 4 | `last_name_ja` | VARCHAR(24) | NO |  |  | Họ (tiếng Nhật) — tương đương 12 ký tự toàn giác |
| 5 | `first_name_ja` | VARCHAR(24) | NO |  |  | Tên (tiếng Nhật) |
| 6 | `last_name_kana` | VARCHAR(48) | NO |  |  | Họ (katakana) — tương đương 24 ký tự toàn giác; katakana |
| 7 | `first_name_kana` | VARCHAR(48) | NO |  |  | Tên (katakana) |
| 8 | `last_name_en` | VARCHAR(24) |  |  |  | Họ (tiếng Anh) — chữ cái Half-width |
| 9 | `middle_name_en` | VARCHAR(12) |  |  |  | Tên đệm (tiếng Anh) |
| 10 | `first_name_en` | VARCHAR(24) |  |  |  | Tên (tiếng Anh) |
| 11 | `gender` | SMALLINT |  |  |  | Giới tính — 1:nam 2:nữ 3:khác/không trả lời |
| 12 | `birth_date` | DATE |  |  |  | Ngày sinh — dùng để tính tuổi, thủ tục bảo hiểm... |
| 13 | `employment_type` | SMALLINT |  |  |  | Hình thức tuyển dụng — 1:có thời hạn 2:vô thời hạn; dùng để xác định áp dụng ngày xung đột cá nhân khi tạo hợp đồng |
| 14 | `registered_date` | DATE |  |  |  | Ngày đăng ký — ngày đăng ký với công ty MOTO |
| 15 | `notification_lang` | CHAR(2) | NO | `'ja'` |  | Ngôn ngữ email thông báo — ja:tiếng Nhật en:tiếng Anh |
| 16 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu (nghỉ việc/xóa đăng ký) 1:hiệu lực |
| 17 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 18 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 19 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 20 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`staff_code`)
- **FK** `fk_staff_company`: (`moto_company_id`) → `mst_moto_company` (company_id)
- **INDEX** `idx_staff_company`: (`moto_company_id`)
- **INDEX** `idx_staff_office`: (`moto_office_id`)


### `mst_staff_credential`


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `staff_code` | VARCHAR(16) | NO |  | PK FK | Mã nhân viên (staff) |
| 2 | `login_id` | VARCHAR(100) | NO |  |  | ID đăng nhập — có ràng buộc UQ; là email...; chữ-số-ký tự Half-width |
| 3 | `password_hash` | VARCHAR(255) | NO |  |  | Hash mật khẩu — giá trị hash (bcrypt...); cấm lưu dạng thô |
| 4 | `last_login_at` | TIMESTAMPTZ |  |  |  | Thời điểm đăng nhập gần nhất |
| 5 | `pwd_change_required_flg` | SMALLINT | NO | `1` |  | Cờ yêu cầu đổi mật khẩu — 0:không cần 1:yêu cầu đổi ở lần đăng nhập tới (lần phát hành đầu = 1) |
| 6 | `pwd_expires_at` | TIMESTAMPTZ |  |  |  | Thời hạn mật khẩu — NULL:không giới hạn |
| 7 | `login_fail_count` | SMALLINT | NO | `0` |  | Số lần đăng nhập thất bại — dùng để xác định khóa tài khoản |
| 8 | `account_locked_flg` | SMALLINT | NO | `0` |  | Cờ khóa tài khoản — 0:bình thường 1:đang khóa |
| 9 | `locked_until` | TIMESTAMPTZ |  |  |  | Thời điểm mở khóa — NULL:đã mở khóa hoặc chưa từng khóa |
| 10 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 11 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 12 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 13 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`staff_code`)
- **UNIQUE** `uq_staff_credential_login`: (`login_id`)
- **FK** `fk_staff_cred_staff`: (`staff_code`) → `mst_staff` (staff_code)


### `mst_staff_conflict_date`

LOGICAL FK sang SAKI — BẮT BUỘC index, không có constraint nào tự index.


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `staff_code` | VARCHAR(16) | NO |  | PK FK | Mã nhân viên (staff) |
| 2 | `saki_company_id` | VARCHAR(20) | NO |  | PK | Mã công ty SAKI — logical FK sang DB SAKI; không ràng buộc vật lý |
| 3 | `saki_department_id` | VARCHAR(20) | NO |  | PK | Mã phòng ban nơi làm việc (đơn vị tổ chức) — ngày xung đột cá nhân quản lý theo đơn vị tổ chức; logical FK sang DB SAKI |
| 4 | `first_dispatch_date` | DATE | NO |  |  | Ngày bắt đầu làm việc lần đầu — ngày bắt đầu làm đầu tiên tại đơn vị tổ chức; điểm gốc tính ngày xung đột |
| 5 | `conflict_date` | DATE | NO |  |  | Ngày xung đột cá nhân — first_dispatch_date + 3 năm + 1 ngày; ngày giới hạn theo Luật phái cử |
| 6 | `cumulative_months` | SMALLINT | NO | `0` |  | Tổng số tháng làm việc — tổng tháng làm tại đơn vị tổ chức (xung đột ở 36 tháng); ứng dụng cập nhật khi gia hạn hợp đồng |
| 7 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:đã reset (gián đoạn trên 3 tháng...) 1:hiệu lực |
| 8 | `reset_reason` | VARCHAR(200) |  |  |  | Lý do reset — ghi khi status=0; vd "gián đoạn trên 3 tháng (YYYY-MM-DD~)" |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`staff_code`, `saki_company_id`, `saki_department_id`) — composite
- **FK** `fk_staff_cd_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **INDEX** `idx_staff_cd_saki_company`: (`saki_company_id`)
- **INDEX (composite)** `idx_staff_cd_saki_dept`: (`saki_company_id`, `saki_department_id`)
- **INDEX (partial)** `idx_staff_cd_conflict_date`: (`conflict_date`) WHERE status = 1

## Contract — 契約管理 (Quản lý hợp đồng)

Hợp đồng phái cử (core của hệ thống). MOTO sở hữu vòng đời. t_contract_saki_detail được SAKI ghi chéo. RLS theo saki_company_id.


### `t_contract`

core hợp đồng (PK = contract_no, không IDENTITY)


> 🔒 **RLS bật** (FORCE). Backend phải set `app.current_saki`/`app.current_moto`/`app.current_role` trước khi query.


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `contract_no` | VARCHAR(14) | NO |  | PK | Số hợp đồng — số hợp đồng carima-staffing; vd C00000000-000 |
| 2 | `original_contract_no` | VARCHAR(14) |  |  | FK | Số hợp đồng gốc — hợp đồng nguồn khi gia hạn/sửa; NULL nếu tạo mới |
| 3 | `saki_company_id` | VARCHAR(20) | NO |  |  | Mã công ty SAKI |
| 4 | `saki_office_id` | VARCHAR(20) | NO |  |  | Mã sự nghiệp sở nơi làm việc |
| 5 | `saki_department_id` | VARCHAR(20) |  |  |  | Mã phòng ban nơi làm việc |
| 6 | `moto_company_id` | VARCHAR(20) | NO |  | FK | Mã công ty MOTO |
| 7 | `moto_office_id` | VARCHAR(20) | NO |  |  | Mã sự nghiệp sở MOTO |
| 8 | `staff_code` | VARCHAR(16) | NO |  | FK | Mã nhân viên (staff) |
| 9 | `contract_type` | SMALLINT | NO | `1` |  | Phân loại hợp đồng — 1:tạo mới 2:gia hạn 3:sửa 4:kết thúc |
| 10 | `contract_category` | SMALLINT | NO | `1` |  | Loại hợp đồng — 1:thường 2:ngoài giới hạn thời hạn 3:nghiệp vụ sản xuất 4:phái cử dự kiến giới thiệu |
| 11 | `initial_start_date` | DATE | NO |  |  | Ngày bắt đầu hợp đồng lần đầu — ngày bắt đầu làm việc đầu tiên, không đổi dù gia hạn nhiều lần |
| 12 | `period_start` | DATE | NO |  |  | Ngày bắt đầu làm việc của hợp đồng này |
| 13 | `period_end` | DATE | NO |  |  | Ngày kết thúc làm việc của hợp đồng này |
| 14 | `office_conflict_date` | DATE |  |  |  | Ngày xung đột sự nghiệp sở — NULL nếu ngoài giới hạn thời hạn |
| 15 | `individual_conflict_date` | DATE |  |  |  | Ngày xung đột cá nhân — đặt khi tuyển dụng có thời hạn |
| 16 | `webtc_enabled_flg` | SMALLINT | NO | `1` |  | Cờ sử dụng WebTimeCard — 0:không 1:có |
| 17 | `timecard_close_unit` | SMALLINT |  |  |  | Đơn vị chốt timecard — 1:chốt cuối tháng 2:chốt ngày 15 3:khác (TBD) |
| 18 | `is_manufacturing_flg` | SMALLINT | NO | `0` |  | Cờ nghiệp vụ sản xuất — 0:không phải 1:nghiệp vụ sản xuất |
| 19 | `is_referral_planned_flg` | SMALLINT | NO | `0` |  | Cờ phái cử dự kiến giới thiệu — 0:thường 1:phái cử dự kiến giới thiệu |
| 20 | `has_job_posting_flg` | SMALLINT | NO | `0` |  | Cờ đính kèm phiếu tuyển dụng — 0:không 1:có; chỉ hiệu lực với phái cử dự kiến giới thiệu |
| 21 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 1:SAKI đang nhập 2:SAKI đang xác nhận 3:trả lại 4:đã rút 5:xác lập 6:hủy |
| 22 | `requestor_user_id` | VARCHAR(100) | NO |  |  | User yêu cầu hợp đồng — người tạo hợp đồng phía MOTO |
| 23 | `confirmor_user_id` | VARCHAR(100) | NO |  |  | User xác nhận hợp đồng — nơi nhận yêu cầu xác nhận phía SAKI |
| 24 | `submitted_at` | TIMESTAMPTZ |  |  |  | Thời điểm nộp — khi MOTO thực hiện "nộp hợp đồng" |
| 25 | `confirmed_at` | TIMESTAMPTZ |  |  |  | Thời điểm xác lập — khi SAKI duyệt cuối; kích hoạt tạo chứng từ |
| 26 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 27 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 28 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 29 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`contract_no`)
- **FK** `fk_contract_original`: (`original_contract_no`) → `t_contract` (contract_no)
- **FK** `fk_contract_moto_company`: (`moto_company_id`) → `mst_moto_company` (company_id)
- **FK** `fk_contract_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **INDEX** `idx_contract_saki_company`: (`saki_company_id`)
- **INDEX** `idx_contract_saki_office`: (`saki_office_id`)
- **INDEX** `idx_contract_moto_company`: (`moto_company_id`)
- **INDEX** `idx_contract_staff`: (`staff_code`)
- **INDEX** `idx_contract_status`: (`status`)
- **INDEX** `idx_contract_original`: (`original_contract_no`)
- **INDEX (composite)** `idx_contract_period`: (`period_start`, `period_end`)
- **RLS policy** — p_contract_saki: USING (saki_company_id = current_setting('app.current_saki', true)) WITH CHECK (saki_company_id = current_setting('app.current_saki', true))
- **RLS policy** — p_contract_moto_owner: USING (current_setting('app.current_role', true) = 'moto_owner') WITH CHECK (current_setting('app.current_role', true) = 'moto_owner')


### `t_contract_moto_detail`

chi tiết MOTO ghi (1:1)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `detail_moto_id` | BIGINT (IDENTITY) |  |  | PK | Mã chi tiết phía MOTO — hệ thống tự đánh số |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 3 | `job_code` | VARCHAR(20) |  |  |  | Job code — mã chữ-số Half-width do MOTO tự đặt |
| 4 | `client_code` | VARCHAR(20) |  |  |  | Mã đối tác hợp đồng — mã chữ-số Half-width do MOTO tự đặt |
| 5 | `treatment_decision_method` | SMALLINT | NO | `1` |  | Phương thức quyết định đãi ngộ — 1:phương thức cân bằng phía SAKI 2:phương thức thỏa thuận lao động |
| 6 | `employment_type` | SMALLINT | NO | `1` |  | Hình thức tuyển dụng — 1:có thời hạn 2:vô thời hạn |
| 7 | `billing_unit_price` | DECIMAL(12,0) |  |  |  | Đơn giá hóa đơn |
| 8 | `billing_unit_type` | SMALLINT |  |  |  | Đơn vị đơn giá — 1:giờ 2:ngày 3:tháng |
| 9 | `overtime_unit_price` | DECIMAL(12,0) |  |  |  | Đơn giá phụ trội |
| 10 | `transportation_fee` | DECIMAL(12,0) |  |  |  | Số tiền tương đương phí đi lại |
| 11 | `show_billing_on_contract_flg` | SMALLINT | NO | `1` |  | Cờ hiển thị trên hóa đơn — 0:ẩn 1:hiện đơn giá... trên hợp đồng riêng |
| 12 | `show_transport_on_contract_flg` | SMALLINT | NO | `0` |  | Cờ hiển thị phí đi lại trên hợp đồng riêng — 0:ẩn 1:hiện |
| 13 | `health_insurance_status` | SMALLINT |  |  |  | Tình trạng tham gia bảo hiểm y tế — 1:đã tham gia 2:chưa tham gia 3:đang xin 4:đang làm thủ tục (TBD) |
| 14 | `welfare_pension_status` | SMALLINT |  |  |  | Tình trạng tham gia lương hưu phúc lợi — 1:đã tham gia 2:chưa tham gia 3:đang xin 4:đang làm thủ tục (TBD) |
| 15 | `employment_insurance_status` | SMALLINT |  |  |  | Tình trạng tham gia bảo hiểm việc làm — 1:đã tham gia 2:chưa tham gia 3:đang xin 4:đang làm thủ tục (TBD) |
| 16 | `business_content` | TEXT |  |  |  | Nội dung nghiệp vụ |
| 17 | `responsibility_level` | VARCHAR(500) |  |  |  | Mức độ trách nhiệm |
| 18 | `period_exempt_reason` | VARCHAR(500) |  |  |  | Lý do ngoài giới hạn thời hạn — chỉ nhập khi contract_category=2 |
| 19 | `comparison_worker_treatment` | TEXT |  |  |  | Thông tin đãi ngộ người lao động đối chiếu — chỉ khi treatment_decision_method=1 (phương thức cân bằng phía SAKI) |
| 20 | `equal_treatment_measures` | TEXT |  |  |  | Biện pháp đảm bảo đãi ngộ cân bằng phía SAKI — TBD |
| 21 | `extra_work_address` | VARCHAR(500) |  |  |  | Nội dung ghi thêm địa chỉ nơi làm việc — phần ghi thêm khi làm tại nhà... |
| 22 | `show_extra_address_flg` | SMALLINT | NO | `0` |  | Cờ hiển thị phần ghi thêm địa chỉ nơi làm việc trên chứng từ — 0:ẩn 1:hiện |
| 23 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 24 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 25 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 26 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`detail_moto_id`)
- **UNIQUE** `uq_contract_moto_detail`: (`contract_no`)
- **FK** `fk_moto_detail_contract`: (`contract_no`) → `t_contract` (contract_no)
- **INDEX** `idx_moto_detail_contract`: (`contract_no`)


### `t_contract_saki_detail`

chi tiết SAKI ghi chéo (1:1)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `detail_saki_id` | BIGINT (IDENTITY) |  |  | PK | Mã chi tiết phía SAKI — hệ thống tự đánh số |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 3 | `contract_manager_user_id` | VARCHAR(100) |  |  |  | User phụ trách hợp đồng (tùy chọn) |
| 4 | `responsible_person_user_id` | VARCHAR(100) |  |  |  | User người phụ trách SAKI — đặt người cùng sự nghiệp sở với nhân viên phái cử |
| 5 | `supervisor_user_id` | VARCHAR(100) |  |  |  | User người chỉ huy — người trực tiếp chỉ huy nhân viên |
| 6 | `invoice_recipient_user_id` | VARCHAR(100) |  |  |  | User nơi nhận hóa đơn — người nhận email thông báo đăng ký hóa đơn |
| 7 | `complaint_contact_user_id` | VARCHAR(100) |  |  |  | User tiếp nhận khiếu nại — có thể kiêm nhiệm bởi người phụ trách SAKI |
| 8 | `timecard_approver_1_user_id` | VARCHAR(100) |  |  |  | User người duyệt timecard 1 — không có ưu tiên giữa người duyệt 1~3 |
| 9 | `timecard_approver_2_user_id` | VARCHAR(100) |  |  |  | User người duyệt timecard 2 |
| 10 | `timecard_approver_3_user_id` | VARCHAR(100) |  |  |  | User người duyệt timecard 3 |
| 11 | `organization_unit` | VARCHAR(100) |  |  |  | Đơn vị tổ chức — có thể sao chép từ master phòng ban |
| 12 | `organization_head_title` | VARCHAR(100) |  |  |  | Chức danh người đứng đầu tổ chức — có thể sao chép từ master phòng ban |
| 13 | `cost_center_code` | VARCHAR(20) |  |  |  | Mã cost center — có thể đặt bắt buộc trong cấu hình master công ty |
| 14 | `cost_center_comment` | VARCHAR(200) |  |  |  | Ghi chú mã cost center — có thể đặt bắt buộc trong cấu hình master công ty |
| 15 | `remark_1` | VARCHAR(500) |  |  |  | Ghi chú 1 — tên trường có thể đổi trong cấu hình công ty |
| 16 | `remark_2` | VARCHAR(500) |  |  |  | Ghi chú 2 |
| 17 | `remark_3` | VARCHAR(500) |  |  |  | Ghi chú 3 |
| 18 | `remark_4` | VARCHAR(500) |  |  |  | Ghi chú 4 |
| 19 | `remark_5` | VARCHAR(500) |  |  |  | Ghi chú 5 |
| 20 | `remark_6` | VARCHAR(500) |  |  |  | Ghi chú 6 |
| 21 | `remark_7` | VARCHAR(500) |  |  |  | Ghi chú 7 |
| 22 | `remark_8` | VARCHAR(500) |  |  |  | Ghi chú 8 |
| 23 | `remark_9` | VARCHAR(500) |  |  |  | Ghi chú 9 |
| 24 | `remark_10` | VARCHAR(500) |  |  |  | Ghi chú 10 |
| 25 | `remark_code_1` | VARCHAR(20) |  |  |  | Mã ghi chú 1 |
| 26 | `remark_code_2` | VARCHAR(20) |  |  |  | Mã ghi chú 2 |
| 27 | `remark_code_3` | VARCHAR(20) |  |  |  | Mã ghi chú 3 |
| 28 | `remark_code_4` | VARCHAR(20) |  |  |  | Mã ghi chú 4 |
| 29 | `remark_code_5` | VARCHAR(20) |  |  |  | Mã ghi chú 5 |
| 30 | `select_remark_1` | VARCHAR(100) |  |  |  | Ghi chú dạng chọn 1 — chọn theo master (TBD) |
| 31 | `select_remark_2` | VARCHAR(100) |  |  |  | Ghi chú dạng chọn 2 |
| 32 | `select_remark_3` | VARCHAR(100) |  |  |  | Ghi chú dạng chọn 3 |
| 33 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 34 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 35 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 36 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`detail_saki_id`)
- **UNIQUE** `uq_contract_saki_detail`: (`contract_no`)
- **FK** `fk_saki_detail_contract`: (`contract_no`) → `t_contract` (contract_no)
- **INDEX** `idx_saki_detail_contract`: (`contract_no`)


### `t_contract_approval`

lịch sử duyệt hợp đồng phía SAKI (1:N)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `approval_id` | BIGINT (IDENTITY) |  |  | PK | Mã lịch sử phê duyệt — hệ thống tự đánh số |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 3 | `operator_user_id` | VARCHAR(100) | NO |  |  | User thao tác — user SAKI đã thao tác |
| 4 | `approval_action` | SMALLINT | NO |  |  | Hành động phê duyệt — 1:xác nhận xong (chuyển người duyệt tiếp theo) 2:duyệt cuối 3:trả lại |
| 5 | `next_approver_user_id` | VARCHAR(100) |  |  |  | User nhận yêu cầu duyệt tiếp theo — chỉ đặt khi approval_action=1 |
| 6 | `next_approver_group_id` | VARCHAR(16) |  |  |  | Mã nhóm nhận yêu cầu duyệt tiếp theo — khi gửi yêu cầu cho nhóm (TBD) |
| 7 | `rejection_comment` | VARCHAR(500) |  |  |  | Ghi chú trả lại — chỉ đặt khi approval_action=3 |
| 8 | `action_at` | TIMESTAMPTZ | NO |  |  | Thời điểm thao tác |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`approval_id`)
- **FK** `fk_contract_approval_contract`: (`contract_no`) → `t_contract` (contract_no)
- **INDEX** `idx_contract_approval_contract`: (`contract_no`)
- **INDEX** `idx_contract_approval_operator`: (`operator_user_id`)


### `t_contract_status_history`

audit log (CÙNG-GHI, system viết)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `status_history_id` | BIGINT (IDENTITY) |  |  | PK | Mã lịch sử đổi trạng thái — hệ thống tự đánh số |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 3 | `from_status` | SMALLINT |  |  |  | Trạng thái trước khi đổi — NULL khi đăng ký lần đầu; tương ứng giá trị t_contract.status |
| 4 | `to_status` | SMALLINT | NO |  |  | Trạng thái sau khi đổi — tương ứng giá trị t_contract.status |
| 5 | `changed_by_user_id` | VARCHAR(100) | NO |  |  | User thao tác — có thể là user MOTO hoặc SAKI |
| 6 | `changed_by_type` | SMALLINT | NO |  |  | Loại người thao tác — 1:MOTO 2:SAKI 3:hệ thống |
| 7 | `change_reason` | VARCHAR(500) |  |  |  | Lý do thay đổi / ghi chú — ghi chú khi trả lại... |
| 8 | `changed_at` | TIMESTAMPTZ | NO |  |  | Thời điểm thay đổi |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`status_history_id`)
- **FK** `fk_status_history_contract`: (`contract_no`) → `t_contract` (contract_no)
- **INDEX** `idx_status_history_contract`: (`contract_no`)
- **INDEX** `idx_status_history_type`: (`changed_by_type`)


### `t_treatment_notification`

Thông báo đãi ngộ / ngày xung đột (待遇 / 抵触日) — SAKI ghi chéo


> 🔒 **RLS bật** (FORCE). Backend phải set `app.current_saki`/`app.current_moto`/`app.current_role` trước khi query.


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `notification_id` | BIGINT (IDENTITY) |  |  | PK | Mã thông báo — hệ thống tự đánh số |
| 2 | `saki_company_id` | VARCHAR(20) | NO |  |  | Mã công ty SAKI |
| 3 | `moto_company_id` | VARCHAR(20) | NO |  | FK | Mã công ty MOTO |
| 4 | `contract_no` | VARCHAR(14) |  |  |  | Số hợp đồng tham chiếu — chỉ đặt với phương thức chọn theo thông tin hợp đồng; NULL với phương thức chọn công ty MOTO |
| 5 | `notification_type` | SMALLINT | NO |  |  | Loại thông báo — 1:chỉ thông tin đãi ngộ 2:chỉ ngày xung đột sự nghiệp sở 3:cả hai |
| 6 | `notification_source` | SMALLINT | NO |  |  | Phương thức cung cấp/thông báo — 1:chọn công ty MOTO (cho hợp đồng mới) 2:chọn theo thông tin hợp đồng (cho gia hạn) |
| 7 | `treatment_info` | TEXT |  |  |  | Nội dung thông tin đãi ngộ — nội dung đãi ngộ đã cung cấp; khi notification_type=1 hoặc 3 |
| 8 | `custom_company_name` | VARCHAR(100) |  |  |  | Ghi đè tên công ty trên chứng từ — khi dùng tên khác tên công ty chính thức |
| 9 | `notification_status` | SMALLINT | NO | `1` |  | Trạng thái thông báo — 1:đã cung cấp 2:hủy cung cấp |
| 10 | `notified_by_user_id` | VARCHAR(100) | NO |  |  | User gửi thông báo |
| 11 | `notified_at` | TIMESTAMPTZ | NO |  |  | Thời điểm thông báo |
| 12 | `cancelled_by_user_id` | VARCHAR(100) |  |  |  | User thực hiện hủy |
| 13 | `cancelled_at` | TIMESTAMPTZ |  |  |  | Thời điểm hủy |
| 14 | `is_auto_notified_flg` | SMALLINT | NO | `0` |  | Cờ thông báo tự động — 0:thủ công 1:tự động theo cấu hình email của công ty |
| 15 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 16 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 17 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 18 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`notification_id`)
- **FK** `fk_treatment_moto_company`: (`moto_company_id`) → `mst_moto_company` (company_id)
- **INDEX** `idx_treatment_saki`: (`saki_company_id`)
- **INDEX** `idx_treatment_moto`: (`moto_company_id`)
- **INDEX** `idx_treatment_contract`: (`contract_no`)
- **RLS policy** — p_treatment_saki: USING (saki_company_id = current_setting('app.current_saki', true)) WITH CHECK (saki_company_id = current_setting('app.current_saki', true))
- **RLS policy** — p_treatment_moto_owner: USING (current_setting('app.current_role', true) = 'moto_owner') WITH CHECK (current_setting('app.current_role', true) = 'moto_owner')


### `t_notification_office_conflict`

Chi tiết ngày xung đột (抵触日) theo sự nghiệp sở — 1:N


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `conflict_id` | BIGINT (IDENTITY) |  |  | PK | Mã chi tiết ngày xung đột — hệ thống tự đánh số |
| 2 | `notification_id` | BIGINT | NO |  | FK | Mã thông báo |
| 3 | `office_id` | VARCHAR(20) | NO |  |  | Mã sự nghiệp sở |
| 4 | `conflict_date` | DATE | NO |  |  | Ngày xung đột — ngày xung đột giới hạn thời hạn của sự nghiệp sở |
| 5 | `effective_start` | DATE |  |  |  | Ngày bắt đầu hiệu lực — ngày bắt đầu áp dụng ngày xung đột (TBD) |
| 6 | `effective_end` | DATE |  |  |  | Ngày kết thúc hiệu lực — ngày kết thúc áp dụng ngày xung đột (TBD) |
| 7 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 8 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 9 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 10 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`conflict_id`)
- **FK** `fk_office_conflict_notification`: (`notification_id`) → `t_treatment_notification` (notification_id)
- **INDEX** `idx_office_conflict_notification`: (`notification_id`)
- **INDEX** `idx_office_conflict_office`: (`office_id`)


### `t_moto_contract_upload_history`

lịch sử upload TSV của MOTO (system viết)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `upload_id` | BIGINT (IDENTITY) |  |  | PK | Mã upload — hệ thống tự đánh số |
| 2 | `uploader_user_id` | VARCHAR(16) | NO |  |  | User upload — có thể là user MOTO hoặc SAKI |
| 3 | `upload_contract_type` | SMALLINT | NO |  |  | Phân loại hợp đồng — MOTO: 1:tạo mới 2:gia hạn 3:sửa 4:từ trả lời / SAKI: 1:tạo mới 2:gia hạn |
| 4 | `file_name` | VARCHAR(255) | NO |  |  | Tên file — tên file TSV đã upload |
| 5 | `total_count` | INTEGER | NO | `0` |  | Tổng số dòng — số dòng dữ liệu của file TSV |
| 6 | `success_count` | INTEGER | NO | `0` |  | Số thành công |
| 7 | `error_count` | INTEGER | NO | `0` |  | Số lỗi |
| 8 | `process_status` | SMALLINT | NO | `1` |  | Trạng thái xử lý — 1:đang xử lý 2:hoàn tất 3:lỗi |
| 9 | `error_detail` | TEXT |  |  |  | Chi tiết lỗi — nội dung lỗi kiểm tra hợp lệ (số dòng, tên trường, thông điệp) |
| 10 | `uploaded_at` | TIMESTAMPTZ | NO |  |  | Thời điểm upload |
| 11 | `completed_at` | TIMESTAMPTZ |  |  |  | Thời điểm xử lý xong |
| 12 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 13 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 14 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 15 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`upload_id`)

## Attendance — 勤怠管理 (Quản lý chấm công)

Chấm công ngày/tháng, duyệt bởi SAKI, chốt công, Hiệp định 36 (36協定). Vùng I/O nặng nhất. RLS theo saki_company_id (denormalized).


### `t_attendance_daily`

Chấm công ngày (staff khởi tạo)


> 🔒 **RLS bật** (FORCE). Backend phải set `app.current_saki`/`app.current_moto`/`app.current_role` trước khi query.


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `daily_id` | BIGINT (IDENTITY) |  |  | PK | Mã chấm công ngày — hệ thống tự đánh số |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng — số hợp đồng carima-staffing (vd C00000000-000) |
| 3 | `saki_company_id` | VARCHAR(20) | NO |  |  | DENORMALIZE từ t_contract. RLS key + tránh JOIN. |
| 4 | `staff_code` | VARCHAR(16) | NO |  | FK | Mã nhân viên (staff) |
| 5 | `work_date` | DATE | NO |  |  | Ngày làm việc |
| 6 | `start_time` | SMALLINT |  |  |  | Giờ bắt đầu làm |
| 7 | `end_time` | SMALLINT |  |  |  | Giờ kết thúc làm — ca qua ngày hôm sau vượt 24:00 (vd 29:30); lưu bằng SMALLINT (TBD) |
| 8 | `break_minutes` | SMALLINT |  |  |  | Thời gian nghỉ (đơn vị phút) |
| 9 | `night_break_minutes` | SMALLINT |  |  |  | Giờ nghỉ ca đêm (đơn vị phút) — nghỉ trong khung 22:00~5:00 hôm sau |
| 10 | `legal_work_minutes` | SMALLINT |  |  |  | Giờ làm trong pháp định (đơn vị phút) — giờ làm trong vòng 8 giờ/ngày |
| 11 | `overtime_minutes` | SMALLINT |  |  |  | Giờ làm thêm (đơn vị phút) |
| 12 | `total_work_minutes` | SMALLINT |  |  |  | Tổng giờ làm việc (đơn vị phút) |
| 13 | `overtime_excess_minutes` | SMALLINT |  |  |  | Giờ làm thêm vượt mức cho phép mỗi ngày theo Hiệp định 36 (đơn vị phút) |
| 14 | `attendance_category` | SMALLINT | NO | `1` |  | Phân loại chấm công — 1:thường 2:nghỉ phép năm 3:làm ngày nghỉ 4:vắng 5:nghỉ nửa ngày 6:làm bù |
| 15 | `apply_status` | SMALLINT | NO | `1` |  | Tình trạng đơn — 1:đang xin 2:đã duyệt 3:thu hồi 4:từ chối 5:hủy duyệt 6:xin hộ |
| 16 | `compensatory_date` | DATE |  |  |  | Ngày nghỉ bù — ngày nghỉ bù tương ứng ngày làm bù |
| 17 | `legal_holiday_work_flg` | SMALLINT | NO | `0` |  | Cờ đi làm ngày nghỉ pháp định — 0:không 1:có |
| 18 | `night_work_minutes` | SMALLINT |  |  |  | Giờ làm ca đêm (đơn vị phút) — giờ làm trong khung 22:00~5:00 hôm sau |
| 19 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 20 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 21 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 22 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`daily_id`)
- **FK** `fk_daily_contract`: (`contract_no`) → `t_contract` (contract_no)
- **FK** `fk_daily_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **INDEX** `idx_daily_contract`: (`contract_no`)
- **INDEX** `idx_daily_staff`: (`staff_code`)
- **INDEX** `idx_daily_date`: (`work_date`)
- **INDEX (composite)** `idx_daily_contract_date`: (`contract_no`, `work_date`)
- **INDEX** `idx_daily_saki`: (`saki_company_id`)
- **RLS policy** — p_att_daily_saki: USING (saki_company_id = current_setting('app.current_saki', true)) WITH CHECK (saki_company_id = current_setting('app.current_saki', true))
- **RLS policy** — p_att_daily_moto_owner: USING (current_setting('app.current_role', true) = 'moto_owner') WITH CHECK (current_setting('app.current_role', true) = 'moto_owner')


### `t_attendance_daily_approval`

SAKI duyệt ngày (SAKI ghi chéo)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `daily_approval_id` | BIGINT (IDENTITY) |  |  | PK | Mã phê duyệt ngày — hệ thống tự đánh số |
| 2 | `daily_id` | BIGINT | NO |  | FK | Mã chấm công ngày |
| 3 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 4 | `staff_code` | VARCHAR(16) | NO |  |  | Mã nhân viên (staff) |
| 5 | `work_date` | DATE | NO |  |  | Ngày làm việc |
| 6 | `approver_user_id` | VARCHAR(100) | NO |  |  | User người duyệt — người phụ trách duyệt phía SAKI |
| 7 | `approval_type` | SMALLINT | NO |  |  | Loại phê duyệt — 1:duyệt 2:từ chối 3:hủy duyệt |
| 8 | `approval_at` | TIMESTAMPTZ | NO |  |  | Thời điểm phê duyệt |
| 9 | `rejection_comment` | VARCHAR(500) |  |  |  | Ghi chú từ chối — ghi chú gửi cho nhân viên khi từ chối |
| 10 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 11 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 12 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 13 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`daily_approval_id`)
- **FK** `fk_daily_appr_daily`: (`daily_id`) → `t_attendance_daily` (daily_id)
- **FK** `fk_daily_appr_contract`: (`contract_no`) → `t_contract` (contract_no)
- **INDEX** `idx_daily_appr_daily`: (`daily_id`)
- **INDEX** `idx_daily_appr_contract`: (`contract_no`)
- **INDEX** `idx_daily_appr_approver`: (`approver_user_id`)


### `t_attendance_close`

Chốt chấm công cuối kỳ (staff khởi tạo)


> 🔒 **RLS bật** (FORCE). Backend phải set `app.current_saki`/`app.current_moto`/`app.current_role` trước khi query.


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `close_id` | BIGINT (IDENTITY) |  |  | PK | Mã đơn chốt — hệ thống tự đánh số |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 3 | `saki_company_id` | VARCHAR(20) | NO |  |  | DENORMALIZE từ t_contract. RLS key. |
| 4 | `staff_code` | VARCHAR(16) | NO |  | FK | Mã nhân viên (staff) |
| 5 | `target_year_month` | CHAR(7) | NO |  |  | Năm-tháng áp dụng — định dạng YYYY/MM |
| 6 | `close_start_date` | DATE | NO |  |  | Ngày bắt đầu chốt — ngày bắt đầu kỳ chốt |
| 7 | `close_end_date` | DATE | NO |  |  | Ngày kết thúc chốt — ngày kết thúc kỳ chốt |
| 8 | `apply_status` | SMALLINT | NO | `1` |  | Tình trạng đơn — 1:đang xin 2:đã duyệt 3:từ chối 4:hủy duyệt |
| 9 | `apply_at` | TIMESTAMPTZ | NO |  |  | Thời điểm xin |
| 10 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 11 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 12 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 13 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`close_id`)
- **FK** `fk_close_contract`: (`contract_no`) → `t_contract` (contract_no)
- **FK** `fk_close_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **INDEX** `idx_close_contract`: (`contract_no`)
- **INDEX** `idx_close_staff`: (`staff_code`)
- **INDEX (composite)** `idx_close_yearmonth`: (`contract_no`, `target_year_month`)
- **INDEX** `idx_close_saki`: (`saki_company_id`)
- **RLS policy** — p_att_close_saki: USING (saki_company_id = current_setting('app.current_saki', true)) WITH CHECK (saki_company_id = current_setting('app.current_saki', true))
- **RLS policy** — p_att_close_moto_owner: USING (current_setting('app.current_role', true) = 'moto_owner') WITH CHECK (current_setting('app.current_role', true) = 'moto_owner')


### `t_attendance_close_approval`

SAKI duyệt chốt (SAKI ghi chéo)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `close_approval_id` | BIGINT (IDENTITY) |  |  | PK | Mã phê duyệt chốt — hệ thống tự đánh số |
| 2 | `close_id` | BIGINT | NO |  | FK | Mã đơn chốt |
| 3 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 4 | `staff_code` | VARCHAR(16) | NO |  |  | Mã nhân viên (staff) |
| 5 | `target_year_month` | CHAR(7) | NO |  |  | Năm-tháng áp dụng — định dạng YYYY/MM |
| 6 | `approver_user_id` | VARCHAR(100) | NO |  |  | User người duyệt — người phụ trách duyệt phía SAKI |
| 7 | `approval_type` | SMALLINT | NO |  |  | Loại phê duyệt — 1:duyệt 2:từ chối 3:hủy duyệt |
| 8 | `approval_at` | TIMESTAMPTZ | NO |  |  | Thời điểm phê duyệt |
| 9 | `rejection_comment` | VARCHAR(500) |  |  |  | Ghi chú từ chối — ghi chú gửi cho nhân viên khi từ chối |
| 10 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 11 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 12 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 13 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`close_approval_id`)
- **FK** `fk_close_appr_close`: (`close_id`) → `t_attendance_close` (close_id)
- **FK** `fk_close_appr_contract`: (`contract_no`) → `t_contract` (contract_no)
- **INDEX** `idx_close_appr_close`: (`close_id`)
- **INDEX** `idx_close_appr_contract`: (`contract_no`)
- **INDEX** `idx_close_appr_approver`: (`approver_user_id`)


### `t_working_time_monthly`

tổng hợp tháng (batch)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `monthly_id` | BIGINT (IDENTITY) |  |  | PK | Mã bản ghi tổng hợp — hệ thống tự đánh số |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 3 | `staff_code` | VARCHAR(16) | NO |  | FK | Mã nhân viên (staff) |
| 4 | `target_year_month` | CHAR(7) | NO |  |  | Năm-tháng áp dụng — định dạng YYYY/MM |
| 5 | `work_site_office_name` | VARCHAR(100) |  |  |  | Tên sự nghiệp sở nơi làm việc |
| 6 | `work_site_department_name` | VARCHAR(100) |  |  |  | Tên phòng ban nơi làm việc |
| 7 | `partner_id` | VARCHAR(20) |  |  |  | Mã đối tác — mã đặt riêng theo từng công ty MOTO |
| 8 | `agency_name` | VARCHAR(24) |  |  |  | Tên công ty MOTO |
| 9 | `total_work_minutes` | INTEGER | NO | `0` |  | Tổng giờ làm việc (đơn vị phút) |
| 10 | `legal_work_minutes` | INTEGER | NO | `0` |  | Giờ làm trong pháp định (đơn vị phút) — tổng giờ làm trong vòng 8 giờ/ngày |
| 11 | `overtime_daily_8h_minutes` | INTEGER | NO | `0` |  | Thời gian làm vượt 8 giờ/ngày (đơn vị phút) |
| 12 | `overtime_weekly_40h_minutes` | INTEGER | NO | `0` |  | Thời gian làm vượt 40 giờ/tuần (đơn vị phút) |
| 13 | `overtime_minutes` | INTEGER | NO | `0` |  | Giờ làm thêm (đơn vị phút) — tổng phần vượt 8 giờ/ngày và 40 giờ/tuần |
| 14 | `night_work_minutes` | INTEGER | NO | `0` |  | Giờ làm ca đêm (đơn vị phút) — khung 22:00~5:00 hôm sau |
| 15 | `legal_holiday_work_minutes` | INTEGER | NO | `0` |  | Giờ làm ngày nghỉ pháp định (đơn vị phút) |
| 16 | `monthly_60h_excess_minutes` | INTEGER | NO | `0` |  | Thời gian làm vượt 60 giờ/tháng (đơn vị phút) |
| 17 | `safety_overtime_minutes` | INTEGER | NO | `0` |  | Giờ làm thêm + làm ngày nghỉ (đơn vị phút) — tính theo công thức Luật An toàn Vệ sinh Lao động |
| 18 | `work_days` | SMALLINT | NO | `0` |  | Số ngày đi làm |
| 19 | `absence_days` | SMALLINT | NO | `0` |  | Số ngày vắng |
| 20 | `annual_leave_days` | SMALLINT | NO | `0` |  | Số ngày nghỉ phép năm |
| 21 | `calendar_days` | SMALLINT | NO | `0` |  | Số ngày lịch — số ngày của năm-tháng áp dụng |
| 22 | `non_work_days` | SMALLINT | NO | `0` |  | Số ngày không làm việc = số ngày trong tháng − số ngày đi làm |
| 23 | `job_type_snapshot` | VARCHAR(160) |  |  |  | Loại công việc tại thời điểm tải — loại công việc yêu cầu tại thời điểm tải |
| 24 | `approver_snapshot` | VARCHAR(149) |  |  |  | Người duyệt tại thời điểm tải — tên người đã duyệt chốt |
| 25 | `aggregated_at` | TIMESTAMPTZ | NO |  |  | Thời điểm tổng hợp — thời điểm chạy batch tổng hợp |
| 26 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 27 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 28 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 29 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`monthly_id`)
- **FK** `fk_monthly_contract`: (`contract_no`) → `t_contract` (contract_no)
- **FK** `fk_monthly_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **INDEX** `idx_monthly_contract`: (`contract_no`)
- **INDEX** `idx_monthly_staff`: (`staff_code`)
- **INDEX (composite)** `idx_monthly_yearmonth`: (`contract_no`, `target_year_month`)


### `t_36_agreement_monthly`

Hiệp định 36 (36協定) — tổng hợp tháng (batch)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `agreement_monthly_id` | BIGINT (IDENTITY) |  |  | PK | Mã Hiệp định 36 theo tháng — hệ thống tự đánh số |
| 2 | `monthly_id` | BIGINT | NO |  | FK | Mã tổng hợp giờ lao động theo tháng |
| 3 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 4 | `staff_code` | VARCHAR(16) | NO |  |  | Mã nhân viên (staff) |
| 5 | `target_year_month` | CHAR(7) | NO |  |  | Năm-tháng áp dụng — định dạng YYYY/MM |
| 6 | `calc_start_date_monthly` | DATE | NO |  |  | Ngày bắt đầu kỳ tổng hợp (tháng) — ngày tính mốc 1 tháng của Hiệp định 36 |
| 7 | `calc_end_date_monthly` | DATE | NO |  |  | Ngày kết thúc kỳ tổng hợp (tháng) — ngày kết thúc kỳ tổng hợp 1 tháng của Hiệp định 36 |
| 8 | `limit_minutes_monthly` | SMALLINT |  |  |  | Giờ giới hạn tháng (đơn vị phút) — giờ làm thêm cho phép/tháng theo Hiệp định 36 |
| 9 | `overtime_minutes_monthly` | INTEGER | NO | `0` |  | Giờ làm thêm (tháng, đơn vị phút) |
| 10 | `overtime_minutes_monthly_prev` | INTEGER |  |  |  | Giờ làm thêm trong tháng - phần từ nơi làm trước (đơn vị phút) |
| 11 | `legal_holiday_minutes_monthly` | INTEGER | NO | `0` |  | Giờ làm ngày nghỉ pháp định trong tháng (đơn vị phút) |
| 12 | `legal_holiday_minutes_monthly_prev` | INTEGER |  |  |  | Giờ làm ngày nghỉ pháp định trong tháng - phần từ nơi làm trước (đơn vị phút) |
| 13 | `special_clause_limit_monthly` | SMALLINT |  |  |  | Giờ giới hạn điều khoản đặc biệt (tháng, đơn vị phút) — giờ cho phép/tháng khi áp dụng điều khoản đặc biệt |
| 14 | `overtime_plus_holiday_monthly` | INTEGER | NO | `0` |  | Làm thêm + ngày nghỉ pháp định (tháng, đơn vị phút) |
| 15 | `remaining_minutes_monthly` | INTEGER |  |  |  | Thời gian còn lại tới giới hạn (tháng, đơn vị phút) |
| 16 | `remaining_minutes_current` | INTEGER |  |  |  | Thời gian còn lại trong tháng (đơn vị phút) — lấy mốc gần nhất trong các giới hạn tháng/năm/nhiều tháng |
| 17 | `special_clause_limit_count` | SMALLINT |  |  |  | Số lần giới hạn điều khoản đặc biệt — tối đa 6 lần/năm |
| 18 | `special_clause_applied_count` | SMALLINT | NO | `0` |  | Số lần áp dụng điều khoản đặc biệt |
| 19 | `special_clause_applied_prev` | SMALLINT |  |  |  | Số lần áp dụng điều khoản đặc biệt - phần từ nơi làm trước |
| 20 | `limit_exceeded_count` | SMALLINT | NO | `0` |  | Số lần vượt giờ giới hạn |
| 21 | `special_clause_status_monthly` | SMALLINT | NO | `0` |  | Tình trạng điều khoản đặc biệt (tháng) — 0:chưa xin 1:chưa áp dụng 2:áp dụng 3:hoãn |
| 22 | `agreement_format` | SMALLINT |  |  |  | Mẫu Hiệp định 36 — 0:mẫu cũ 1:mẫu mới |
| 23 | `agreement_valid_start` | DATE |  |  |  | Ngày bắt đầu kỳ hiệu lực Hiệp định 36 |
| 24 | `agreement_valid_end` | DATE |  |  |  | Ngày kết thúc kỳ hiệu lực Hiệp định 36 |
| 25 | `agreement_calc_start_monthly` | SMALLINT |  |  |  | Ngày tính mốc Hiệp định 36 (tháng) — 1~31 (ngày) |
| 26 | `remaining_minutes_multi_month` | INTEGER |  |  |  | Thời gian còn lại tới giới hạn trung bình nhiều tháng theo Hiệp định 36 (đơn vị phút) |
| 27 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 28 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 29 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 30 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`agreement_monthly_id`)
- **FK** `fk_36m_monthly`: (`monthly_id`) → `t_working_time_monthly` (monthly_id)
- **FK** `fk_36m_contract`: (`contract_no`) → `t_contract` (contract_no)
- **INDEX** `idx_36m_monthly`: (`monthly_id`)
- **INDEX** `idx_36m_contract`: (`contract_no`)
- **INDEX** `idx_36m_staff`: (`staff_code`)


### `t_36_agreement_yearly`

Hiệp định 36 (36協定) — tổng hợp năm (batch)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `agreement_yearly_id` | BIGINT (IDENTITY) |  |  | PK | Mã Hiệp định 36 theo năm — hệ thống tự đánh số |
| 2 | `agreement_monthly_id` | BIGINT | NO |  | FK | Mã Hiệp định 36 theo tháng |
| 3 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 4 | `staff_code` | VARCHAR(16) | NO |  |  | Mã nhân viên (staff) |
| 5 | `calc_start_date_yearly` | DATE | NO |  |  | Ngày bắt đầu kỳ tổng hợp (năm) — mẫu mới: ngày cấu hình Hiệp định 36; mẫu cũ: ngày bắt đầu hiệu lực sớm nhất |
| 6 | `calc_end_date_yearly` | DATE | NO |  |  | Ngày kết thúc kỳ tổng hợp (năm) |
| 7 | `limit_minutes_yearly` | INTEGER |  |  |  | Giờ giới hạn năm (đơn vị phút) — giờ làm thêm cho phép/năm theo Hiệp định 36 |
| 8 | `overtime_minutes_yearly` | INTEGER | NO | `0` |  | Giờ làm thêm trong năm (đơn vị phút) — tổng từ ngày bắt đầu hiệu lực Hiệp định 36 đến hiện tại |
| 9 | `remaining_minutes_yearly` | INTEGER |  |  |  | Thời gian còn lại tới giới hạn (năm, đơn vị phút) |
| 10 | `special_clause_status_yearly` | SMALLINT | NO | `0` |  | Tình trạng điều khoản đặc biệt (năm) — 0:chưa xin 1:chưa áp dụng 2:áp dụng 3:hoãn 4:không áp dụng |
| 11 | `special_clause_limit_yearly` | INTEGER |  |  |  | Giờ giới hạn năm của điều khoản đặc biệt (đơn vị phút) — giờ làm thêm cho phép/năm khi áp dụng điều khoản đặc biệt |
| 12 | `yearly_overtime_exceeded_count` | SMALLINT | NO | `0` |  | Số lần vượt giới hạn tháng của Hiệp định 36 trong năm |
| 13 | `agreement_calc_start_yearly` | DATE |  |  |  | Ngày tính mốc Hiệp định 36 (năm) |
| 14 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 15 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 16 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 17 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`agreement_yearly_id`)
- **FK** `fk_36y_monthly`: (`agreement_monthly_id`) → `t_36_agreement_monthly` (agreement_monthly_id)
- **FK** `fk_36y_contract`: (`contract_no`) → `t_contract` (contract_no)
- **INDEX** `idx_36y_monthly`: (`agreement_monthly_id`)
- **INDEX** `idx_36y_contract`: (`contract_no`)
- **INDEX** `idx_36y_staff`: (`staff_code`)


### `t_special_clause_application`

Đơn điều khoản đặc biệt (SAKI khởi tạo)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `special_clause_id` | BIGINT (IDENTITY) |  |  | PK | Mã đơn điều khoản đặc biệt — hệ thống tự đánh số |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng |
| 3 | `staff_code` | VARCHAR(16) | NO |  | FK | Mã nhân viên (staff) |
| 4 | `target_year_month` | CHAR(7) | NO |  |  | Năm-tháng áp dụng — định dạng YYYY/MM |
| 5 | `application_type` | SMALLINT | NO |  |  | Loại đơn — 1:tháng 2:năm |
| 6 | `apply_count` | SMALLINT | NO |  |  | Số lần áp dụng — số lần áp dụng lần này (lũy kế); tối đa 6 lần |
| 7 | `applied_overtime_minutes` | SMALLINT | NO |  |  | Giờ làm thêm xin duyệt (đơn vị phút) — giờ làm thêm dự kiến |
| 8 | `apply_reason` | VARCHAR(500) | NO |  |  | Lý do xin — lý do tình huống đặc biệt |
| 9 | `applicant_user_id` | VARCHAR(100) | NO |  |  | User người xin — user SAKI đã gửi đơn |
| 10 | `apply_at` | TIMESTAMPTZ | NO |  |  | Thời điểm xin |
| 11 | `apply_status` | SMALLINT | NO | `0` |  | Tình trạng đơn — 0:đang xin 1:áp dụng 2:hoãn |
| 12 | `agency_processed_at` | TIMESTAMPTZ |  |  |  | Thời điểm MOTO xử lý — khi MOTO duyệt/từ chối |
| 13 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 14 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 15 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 16 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`special_clause_id`)
- **FK** `fk_special_clause_contract`: (`contract_no`) → `t_contract` (contract_no)
- **FK** `fk_special_clause_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **INDEX** `idx_special_clause_contract`: (`contract_no`)
- **INDEX** `idx_special_clause_staff`: (`staff_code`)
- **INDEX** `idx_special_clause_applicant`: (`applicant_user_id`)


### `t_attendance_email_history`

-- contract_no: FK vật lý t_contract (cùng schema MOTO).


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `email_history_id` | BIGINT (IDENTITY) |  |  | PK | Mã lịch sử gửi email — hệ thống tự đánh số |
| 2 | `send_type` | SMALLINT | NO |  |  | Loại gửi — 1:nhắc chưa nộp đơn (gửi nhân viên) 2:nhắc chưa duyệt (gửi người duyệt) |
| 3 | `close_year_month` | CHAR(7) | NO |  |  | Tháng chốt — định dạng YYYY/MM |
| 4 | `close_start_date` | DATE |  |  |  | Ngày bắt đầu chốt — ngày bắt đầu kỳ áp dụng |
| 5 | `close_end_date` | DATE |  |  |  | Ngày kết thúc chốt — ngày kết thúc kỳ áp dụng |
| 6 | `contract_no` | VARCHAR(14) | NO |  |  | Số hợp đồng — không ràng buộc FK (ảnh chụp tại thời điểm gửi) |
| 7 | `staff_code` | VARCHAR(16) | NO |  |  | Mã nhân viên (staff) |
| 8 | `staff_name` | VARCHAR(64) | NO |  |  | Tên nhân viên — ảnh chụp tại thời điểm gửi |
| 9 | `to_email` | VARCHAR(513) | NO |  |  | Địa chỉ email nhận |
| 10 | `sender_type` | SMALLINT | NO |  |  | Loại nơi gửi — 1:SAKI 2:MOTO |
| 11 | `sender_user_id` | VARCHAR(100) | NO |  |  | User người gửi — user thực hiện thao tác gửi |
| 12 | `sender_user_name` | VARCHAR(64) | NO |  |  | Tên người gửi |
| 13 | `comment` | VARCHAR(500) |  |  |  | Ghi chú — ghi chú tùy chọn thêm khi gửi |
| 14 | `scheduled_at` | TIMESTAMPTZ |  |  |  | Thời điểm hẹn gửi — chỉ với trường hợp gửi theo lịch hẹn |
| 15 | `sent_at` | TIMESTAMPTZ |  |  |  | Thời điểm gửi xong — thời điểm thực tế đã gửi |
| 16 | `send_status` | SMALLINT | NO | `0` |  | Tình trạng gửi — 0:chờ gửi 1:gửi xong 2:gửi thất bại |
| 17 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 18 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 19 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 20 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`email_history_id`)
- **INDEX** `idx_email_history_contract`: (`contract_no`)
- **INDEX** `idx_email_history_staff`: (`staff_code`)

## Billing — 請求管理 (Quản lý hóa đơn)

Tính & xuất hóa đơn (JOIN sâu 5-6 bảng). SAKI có quyền khóa hóa đơn qua System Connection. RLS theo client_code (= saki_company_id).


### `t_invoice`

header hóa đơn


> 🔒 **RLS bật** (FORCE). Backend phải set `app.current_saki`/`app.current_moto`/`app.current_role` trước khi query.


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `invoice_id` | BIGINT (IDENTITY) |  |  | PK | Mã hóa đơn — hệ thống tự đánh số |
| 2 | `invoice_code` | VARCHAR(16) | NO |  |  | Mã hóa đơn — khóa nghiệp vụ trên carima-staffing; UQ |
| 3 | `target_year_month` | CHAR(7) | NO |  |  | Năm-tháng tính hóa đơn — định dạng YYYY/MM |
| 4 | `client_code` | VARCHAR(20) | NO |  |  | Mã client — mã công ty SAKI |
| 5 | `invoice_destination_code` | VARCHAR(16) | NO |  |  | Mã nơi nhận hóa đơn |
| 6 | `invoice_destination_name` | VARCHAR(100) | NO |  |  | Tên nơi nhận hóa đơn |
| 7 | `billing_start_date` | DATE | NO |  |  | Ngày bắt đầu kỳ tính hóa đơn |
| 8 | `billing_end_date` | DATE | NO |  |  | Ngày kết thúc kỳ tính hóa đơn |
| 9 | `invoice_date` | DATE | NO |  |  | Ngày hóa đơn |
| 10 | `payment_due_date` | DATE | NO |  |  | Hạn thanh toán |
| 11 | `subtotal` | DECIMAL(12,0) | NO | `0` |  | Tạm tính hóa đơn — số tiền cơ bản chưa gồm thuế tiêu thụ và tiền tạm ứng |
| 12 | `tax_amount` | DECIMAL(12,0) | NO | `0` |  | Số tiền thuế tiêu thụ |
| 13 | `invoice_total` | DECIMAL(12,0) | NO | `0` |  | Tổng tiền hóa đơn — tạm tính + thuế tiêu thụ |
| 14 | `grand_total` | DECIMAL(12,0) | NO | `0` |  | Tổng cộng cuối hóa đơn — số tiền cuối gồm tổng + tiền tạm ứng... |
| 15 | `revision_flag` | SMALLINT | NO | `0` |  | Cờ chỉnh sửa — 0:nộp trực tiếp 1:lưu tạm (cùng nghĩa với giá trị của phương thức upload file) |
| 16 | `invoice_status` | SMALLINT | NO | `1` |  | Trạng thái hóa đơn — 1:lưu tạm 2:đã nộp 3:đã sửa 4:đã khóa 5:không sửa được 6:đã chốt tháng |
| 17 | `agency_company_id` | VARCHAR(20) | NO |  | FK | Mã công ty MOTO — công ty MOTO đã tạo hóa đơn |
| 18 | `invoice_format_type` | CHAR(2) |  |  |  | Phân loại định dạng hóa đơn — chỉ đặt với hóa đơn hợp lệ (TBD) |
| 19 | `layout_version` | VARCHAR(10) |  |  |  | Phiên bản layout — chỉ đặt với hóa đơn hợp lệ |
| 20 | `standard_tax_subtotal` | DECIMAL(12,0) |  |  |  | Tạm tính chịu thuế suất chuẩn — chỉ đặt với hóa đơn hợp lệ |
| 21 | `standard_tax_amount` | DECIMAL(12,0) |  |  |  | Thuế tiêu thụ suất chuẩn — chỉ đặt với hóa đơn hợp lệ |
| 22 | `reduced_tax_subtotal` | DECIMAL(12,0) |  |  |  | Tạm tính chịu thuế suất giảm — chỉ đặt với hóa đơn hợp lệ |
| 23 | `reduced_tax_amount` | DECIMAL(12,0) |  |  |  | Thuế tiêu thụ suất giảm — chỉ đặt với hóa đơn hợp lệ |
| 24 | `other_addition_total` | DECIMAL(12,0) |  |  |  | Tổng cộng thêm khác — chỉ đặt với hóa đơn hợp lệ |
| 25 | `other_deduction_total` | DECIMAL(12,0) |  |  |  | Tổng trừ khác — chỉ đặt với hóa đơn hợp lệ |
| 26 | `inquiry_contact` | VARCHAR(200) |  |  |  | Nơi liên hệ |
| 27 | `comment` | VARCHAR(500) |  |  |  | Ghi chú |
| 28 | `invoice_issue_unit` | SMALLINT |  |  |  | Đơn vị phát hành hóa đơn — 1:mỗi công ty 1 hóa đơn 2:mỗi sự nghiệp sở 1 hóa đơn 3:mỗi phòng ban 1 hóa đơn |
| 29 | `lock_flag` | SMALLINT | NO | `0` |  | Cờ khóa — 0:chưa khóa 1:đã khóa (SAKI đặt khi tải xuống) |
| 30 | `no_modification_flag` | SMALLINT | NO | `0` |  | Cờ không cho sửa — 0:sửa được 1:không sửa được (SAKI gán) |
| 31 | `monthly_close_flag` | SMALLINT | NO | `0` |  | Cờ chốt tháng — 0:chưa chốt 1:đã chốt tháng (SAKI thực hiện) |
| 32 | `upload_history_id` | BIGINT |  |  |  | Mã lịch sử upload — chỉ đặt khi tạo bằng phương thức upload file |
| 33 | `submitted_at` | TIMESTAMPTZ |  |  |  | Thời điểm nộp — thời điểm nộp lần đầu |
| 34 | `locked_at` | TIMESTAMPTZ |  |  |  | Thời điểm khóa — thời điểm đặt khóa gần nhất |
| 35 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 36 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 37 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 38 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`invoice_id`)
- **UNIQUE** `uq_invoice_code`: (`invoice_code`)
- **FK** `fk_invoice_agency`: (`agency_company_id`) → `mst_moto_company` (company_id)
- **INDEX** `idx_invoice_client`: (`client_code`)
- **INDEX** `idx_invoice_agency`: (`agency_company_id`)
- **INDEX** `idx_invoice_yearmonth`: (`target_year_month`)
- **INDEX** `idx_invoice_status`: (`invoice_status`)
- **INDEX** `idx_invoice_upload`: (`upload_history_id`)
- **RLS policy** — p_invoice_saki: USING (client_code = current_setting('app.current_saki', true)) WITH CHECK (client_code = current_setting('app.current_saki', true))
- **RLS policy** — p_invoice_moto_owner: USING (current_setting('app.current_role', true) = 'moto_owner') WITH CHECK (current_setting('app.current_role', true) = 'moto_owner')


### `t_invoice_bank_account`

tài khoản nhận tiền (max 6)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `bank_account_id` | BIGINT (IDENTITY) |  |  | PK | Mã tài khoản nhận tiền — hệ thống tự đánh số |
| 2 | `invoice_id` | BIGINT | NO |  | FK | Mã hóa đơn |
| 3 | `bank_seq` | SMALLINT | NO |  |  | Số thứ tự tài khoản — 1~6; số thứ tự UQ trong cùng invoice_id |
| 4 | `bank_name` | VARCHAR(50) | NO |  |  | Tên ngân hàng nhận tiền |
| 5 | `branch_name` | VARCHAR(50) | NO |  |  | Tên chi nhánh |
| 6 | `account_type` | SMALLINT | NO |  |  | Loại tài khoản — 1:thường 2:vãng lai |
| 7 | `account_number` | VARCHAR(20) | NO |  |  | Số tài khoản |
| 8 | `account_holder` | VARCHAR(100) | NO |  |  | Tên chủ tài khoản |
| 9 | `bank_code` | CHAR(4) |  |  |  | Mã ngân hàng — mã tổ chức tài chính 4 chữ số |
| 10 | `branch_code` | CHAR(3) |  |  |  | Mã chi nhánh |
| 11 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 12 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 13 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 14 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`bank_account_id`)
- **UNIQUE** `uq_invoice_bank_seq`: (`invoice_id`, `bank_seq`) — composite
- **FK** `fk_bank_invoice`: (`invoice_id`) → `t_invoice` (invoice_id)
- **INDEX** `idx_invoice_bank_invoice`: (`invoice_id`)


### `t_invoice_detail`

chi tiết hóa đơn theo staff/kỳ


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `invoice_detail_id` | BIGINT (IDENTITY) |  |  | PK | Mã chi tiết hóa đơn — hệ thống tự đánh số |
| 2 | `invoice_id` | BIGINT | NO |  | FK | Mã hóa đơn |
| 3 | `invoice_detail_code` | VARCHAR(16) | NO |  |  | Mã chi tiết hóa đơn — khóa nghiệp vụ chi tiết trên carima-staffing; UQ trong invoice_id |
| 4 | `contract_no` | VARCHAR(14) | NO |  | FK | Số hợp đồng carima-staffing — vd C00000000-000 |
| 5 | `invoice_code` | VARCHAR(16) | NO |  |  | Mã hóa đơn — cùng giá trị với invoice_code của t_invoice; giữ để đối chiếu trong phương thức upload file |
| 6 | `agency_company_id` | VARCHAR(20) | NO |  |  | Mã công ty MOTO |
| 7 | `staff_code` | VARCHAR(16) | NO |  | FK | Mã nhân viên (staff) |
| 8 | `staff_name` | VARCHAR(64) | NO |  |  | Tên nhân viên — ảnh chụp tại thời điểm đăng ký |
| 9 | `job_code` | VARCHAR(20) | NO |  |  | Job code |
| 10 | `target_year_month` | CHAR(7) | NO |  |  | Năm-tháng tính hóa đơn — định dạng YYYY/MM |
| 11 | `billing_start_date` | DATE | NO |  |  | Ngày bắt đầu hóa đơn |
| 12 | `billing_end_date` | DATE | NO |  |  | Ngày kết thúc hóa đơn |
| 13 | `detail_unit` | SMALLINT | NO |  |  | Đơn vị tạo chi tiết — 1:theo từng ngày chốt 2:theo kỳ tính hóa đơn 3:theo kỳ tùy chọn |
| 14 | `attendance_source` | SMALLINT | NO |  |  | Nguồn lấy chấm công — 1:WebTimeCard 2:hợp đồng 3:không |
| 15 | `timecard_reimbursement_import_flag` | SMALLINT | NO | `0` |  | Cờ nhập tiền tạm ứng từ timecard — 1=tự nhập thông tin tiền tạm ứng từ WebTimeCard; bắt buộc là hóa đơn hợp lệ và có số hợp đồng carima-staffing |
| 16 | `in_contract_hours` | SMALLINT |  |  |  | Giờ trong hợp đồng (đơn vị giờ) |
| 17 | `in_contract_minutes` | SMALLINT |  |  |  | Phút trong hợp đồng (0~59) |
| 18 | `in_contract_amount` | DECIMAL(12,0) |  |  |  | Số tiền trong hợp đồng |
| 19 | `overtime_hours` | SMALLINT |  |  |  | Giờ làm thêm (đơn vị giờ) |
| 20 | `overtime_minutes` | SMALLINT |  |  |  | Phút làm thêm (0~59) |
| 21 | `overtime_amount` | DECIMAL(12,0) |  |  |  | Số tiền làm thêm |
| 22 | `holiday_work_hours` | SMALLINT |  |  |  | Giờ làm ngày nghỉ (đơn vị giờ) |
| 23 | `holiday_work_minutes` | SMALLINT |  |  |  | Phút làm ngày nghỉ (0~59) |
| 24 | `holiday_work_amount` | DECIMAL(12,0) |  |  |  | Số tiền làm ngày nghỉ |
| 25 | `out_of_contract_subtotal_hours` | SMALLINT |  |  |  | Giờ tạm tính ngoài hợp đồng (đơn vị giờ) — tổng giờ làm thêm và làm ngày nghỉ |
| 26 | `out_of_contract_subtotal_minutes` | SMALLINT |  |  |  | Phút tạm tính ngoài hợp đồng (0~59) — tổng giờ làm thêm và làm ngày nghỉ |
| 27 | `out_of_contract_subtotal_amount` | DECIMAL(12,0) |  |  |  | Số tiền tạm tính ngoài hợp đồng |
| 28 | `late_night_hours` | SMALLINT |  |  |  | Giờ ca đêm (đơn vị giờ) |
| 29 | `late_night_minutes` | SMALLINT |  |  |  | Phút ca đêm (0~59) |
| 30 | `late_night_amount` | DECIMAL(12,0) |  |  |  | Số tiền ca đêm |
| 31 | `deduction_hours` | SMALLINT |  |  |  | Giờ khấu trừ (đơn vị giờ) |
| 32 | `deduction_days` | SMALLINT |  |  |  | Số ngày khấu trừ |
| 33 | `subtotal` | DECIMAL(12,0) | NO | `0` |  | Tạm tính — tạm tính hóa đơn của chi tiết này |
| 34 | `adjustment_amount` | DECIMAL(12,0) | NO | `0` |  | Khoản điều chỉnh — tổng của chi tiết khoản điều chỉnh |
| 35 | `reimbursement_amount` | DECIMAL(12,0) | NO | `0` |  | Số tiền tạm ứng — tổng của bảng quyết toán tiền tạm ứng |
| 36 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 37 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 38 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 39 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`invoice_detail_id`)
- **UNIQUE** `uq_invoice_detail_code`: (`invoice_id`, `invoice_detail_code`) — composite
- **FK** `fk_detail_invoice`: (`invoice_id`) → `t_invoice` (invoice_id)
- **FK** `fk_detail_contract`: (`contract_no`) → `t_contract` (contract_no)
- **FK** `fk_detail_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **INDEX** `idx_invoice_detail_invoice`: (`invoice_id`)
- **INDEX** `idx_invoice_detail_contract`: (`contract_no`)
- **INDEX** `idx_invoice_detail_staff`: (`staff_code`)
- **INDEX** `idx_invoice_detail_agency`: (`agency_company_id`)


### `t_invoice_detail_attendance_daily`

chấm công ngày trong hóa đơn (1:N)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `attendance_daily_id` | BIGINT (IDENTITY) |  |  | PK | Mã chi tiết chấm công ngày trên hóa đơn — hệ thống tự đánh số |
| 2 | `invoice_detail_id` | BIGINT | NO |  | FK | Mã chi tiết hóa đơn |
| 3 | `timecard_period_type` | VARCHAR(9) |  |  |  | Loại kỳ timecard — chỉ đặt với phương thức upload file |
| 4 | `work_date` | DATE | NO |  |  | Ngày làm việc |
| 5 | `attendance_category` | SMALLINT | NO | `1` |  | Phân loại chấm công — 0:nghỉ bù 1:thường 2:nghỉ phép năm 3:làm ngày nghỉ 4:vắng 5:nghỉ nửa ngày 6:làm bù |
| 6 | `start_time_hour` | SMALLINT |  |  |  | Giờ bắt đầu (0~23) |
| 7 | `start_time_minute` | SMALLINT |  |  |  | Phút bắt đầu (0~59) |
| 8 | `end_time_hour` | SMALLINT |  |  |  | Giờ kết thúc — ca qua ngày hôm sau cho phép ≥24 (TBD; TINYINT hỗ trợ 0~255) |
| 9 | `end_time_minute` | SMALLINT |  |  |  | Phút kết thúc (0~59) |
| 10 | `break_minutes` | SMALLINT |  |  |  | Thời gian nghỉ (đơn vị phút) |
| 11 | `late_night_break_minutes` | SMALLINT |  |  |  | Giờ nghỉ ca đêm (đơn vị phút) — nghỉ trong khung 22:00~5:00 hôm sau |
| 12 | `in_contract_minutes` | SMALLINT |  |  |  | Giờ trong hợp đồng (đơn vị phút) — giờ làm trong hợp đồng theo ngày |
| 13 | `out_of_contract_minutes` | SMALLINT |  |  |  | Giờ ngoài hợp đồng (đơn vị phút) — giờ làm ngoài hợp đồng theo ngày |
| 14 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 15 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 16 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 17 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`attendance_daily_id`)
- **FK** `fk_inv_att_daily_detail`: (`invoice_detail_id`) → `t_invoice_detail` (invoice_detail_id)
- **INDEX** `idx_inv_att_daily_detail`: (`invoice_detail_id`)


### `t_invoice_adjustment_detail`

điều chỉnh (1:N)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `adjustment_detail_id` | BIGINT (IDENTITY) |  |  | PK | Mã chi tiết khoản điều chỉnh — hệ thống tự đánh số |
| 2 | `invoice_detail_id` | BIGINT | NO |  | FK | Mã chi tiết hóa đơn |
| 3 | `adjustment_category` | SMALLINT | NO |  |  | Phân loại khoản điều chỉnh — 0:khoản điều chỉnh 1 1:khoản điều chỉnh khác 2:khoản điều chỉnh đặc biệt 1 3:khoản điều chỉnh đặc biệt 2 |
| 4 | `tax_category` | SMALLINT | NO |  |  | Phân loại thuế tiêu thụ — 0:thuế suất chuẩn 1:thuế suất giảm |
| 5 | `amount` | DECIMAL(12,0) | NO | `0` |  | Số tiền — cho phép giá trị âm (khấu trừ) |
| 6 | `description` | VARCHAR(200) |  |  |  | Nội dung / lý do |
| 7 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 8 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 9 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 10 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`adjustment_detail_id`)
- **FK** `fk_adjustment_detail`: (`invoice_detail_id`) → `t_invoice_detail` (invoice_detail_id)
- **INDEX** `idx_adjustment_detail`: (`invoice_detail_id`)


### `t_invoice_reimbursement_header`

Header bảng quyết toán tiền tạm ứng (立替金) — 1:1 với detail


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `reimbursement_header_id` | BIGINT (IDENTITY) |  |  | PK | Mã header bảng quyết toán tiền tạm ứng — hệ thống tự đánh số |
| 2 | `invoice_detail_id` | BIGINT | NO |  | FK | Mã chi tiết hóa đơn — UQ; quan hệ 1:1 với t_invoice_detail |
| 3 | `timecard_import_flag` | SMALLINT | NO | `0` |  | Cờ nhập tiền tạm ứng từ timecard — 1=tự nhập từ WebTimeCard; chỉ hiệu lực khi là hóa đơn hợp lệ và có số hợp đồng carima-staffing |
| 4 | `transport_total_amount` | DECIMAL(12,0) | NO | `0` |  | Tổng phí đi lại công việc — tổng loại tiền tạm ứng 1 |
| 5 | `other_reimbursement_total` | DECIMAL(12,0) | NO | `0` |  | Tổng tiền tạm ứng khác — tổng loại tiền tạm ứng 2 |
| 6 | `tax_exempt_reimbursement_total` | DECIMAL(12,0) | NO | `0` |  | Tổng tiền tạm ứng không chịu thuế — tổng loại tiền tạm ứng 3 (không chịu thuế) |
| 7 | `non_taxable_reimbursement_total` | DECIMAL(12,0) | NO | `0` |  | Tổng tiền tạm ứng ngoài đối tượng thuế — tổng loại tiền tạm ứng 3 (ngoài đối tượng thuế) |
| 8 | `grand_total` | DECIMAL(12,0) | NO | `0` |  | Tổng tiền tạm ứng — tổng tất cả các loại |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`reimbursement_header_id`)
- **UNIQUE** `uq_reimbursement_detail`: (`invoice_detail_id`)
- **FK** `fk_reimb_header_detail`: (`invoice_detail_id`) → `t_invoice_detail` (invoice_detail_id)
- **INDEX** `idx_reimb_header_detail`: (`invoice_detail_id`)


### `t_invoice_reimbursement_detail`

Chi tiết tiền tạm ứng (立替金) theo phân loại chi phí (費用区分) — 1:N


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `reimbursement_detail_id` | BIGINT (IDENTITY) |  |  | PK | Mã chi tiết bảng quyết toán tiền tạm ứng — hệ thống tự đánh số |
| 2 | `reimbursement_header_id` | BIGINT | NO |  | FK | Mã header bảng quyết toán tiền tạm ứng |
| 3 | `expense_category` | CHAR(2) | NO |  |  | Phân loại chi phí — mã phân loại 01~94 (theo tài liệu chuẩn cấu hình) |
| 4 | `expense_date` | DATE |  |  |  | Ngày phát sinh chi phí |
| 5 | `description` | VARCHAR(200) |  |  |  | Nội dung |
| 6 | `amount` | DECIMAL(12,0) | NO | `0` |  | Số tiền |
| 7 | `tax_category` | SMALLINT |  |  |  | Phân loại thuế tiêu thụ — 0:thuế suất chuẩn 1:thuế suất giảm 2:không chịu thuế 3:ngoài đối tượng thuế |
| 8 | `reimbursement_type` | SMALLINT | NO |  |  | Loại tiền tạm ứng — 1:phí đi lại công việc 2:tiền tạm ứng khác 3:tiền tạm ứng không chịu thuế / ngoài đối tượng thuế |
| 9 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 10 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 11 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 12 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`reimbursement_detail_id`)
- **FK** `fk_reimb_detail_header`: (`reimbursement_header_id`) → `t_invoice_reimbursement_header` (reimbursement_header_id)
- **INDEX** `idx_reimb_detail_header`: (`reimbursement_header_id`)
- **INDEX** `idx_reimb_detail_expense`: (`expense_category`)


### `t_invoice_upload_history`

lịch sử upload ZIP (MOTO)


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `upload_history_id` | BIGINT (IDENTITY) |  |  | PK | Mã lịch sử upload — hệ thống tự đánh số |
| 2 | `agency_company_id` | VARCHAR(20) | NO |  | FK | Mã công ty MOTO — công ty MOTO đã thực hiện upload |
| 3 | `upload_user_id` | VARCHAR(100) | NO |  |  | User thực hiện upload |
| 4 | `zip_file_name` | VARCHAR(255) | NO |  |  | Tên file ZIP — quy tắc đặt tên: company_id_mmddhhmm |
| 5 | `upload_at` | TIMESTAMPTZ | NO |  |  | Thời điểm upload |
| 6 | `upload_status` | SMALLINT | NO | `0` |  | Kết quả xử lý — 0:đang xử lý 1:thành công 2:lỗi |
| 7 | `invoice_count` | INTEGER |  |  |  | Số hóa đơn đã nhập — đặt khi kiểm tra hợp lệ thành công |
| 8 | `detail_count` | INTEGER |  |  |  | Số chi tiết đã nhập — đặt khi kiểm tra hợp lệ thành công |
| 9 | `error_message` | TEXT |  |  |  | Nội dung lỗi — đặt khi kiểm tra hợp lệ thất bại; nhiều lỗi lưu phân tách bằng xuống dòng |
| 10 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 11 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 12 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 13 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`upload_history_id`)
- **FK** `fk_invoice_upload_agency`: (`agency_company_id`) → `mst_moto_company` (company_id)
- **INDEX** `idx_invoice_upload_agency`: (`agency_company_id`)

## Shift & Batch

Ca làm việc, lịch ca, trạng thái đọc thông báo, lịch sử batch.


### `mst_moto_shift`

master ca làm việc do MOTO định nghĩa


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `company_id` | VARCHAR(20) | NO |  | PK FK | Công ty MOTO sở hữu |
| 2 | `shift_code` | VARCHAR(20) | NO |  | PK | Mã ca làm việc |
| 3 | `shift_name` | VARCHAR(100) | NO |  |  | Tên ca làm việc |
| 4 | `start_time` | SMALLINT | NO |  |  | Giờ bắt đầu ca (phút từ 00:00) |
| 5 | `end_time` | SMALLINT | NO |  |  | Giờ kết thúc ca (phút từ 00:00; ca đêm >1440) |
| 6 | `break_minutes` | SMALLINT | NO | `0` |  | Thời gian nghỉ trong ca (phút) |
| 7 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 0:vô hiệu 1:đang dùng |
| 8 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 9 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 10 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 11 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`company_id`, `shift_code`) — composite
- **FK** `fk_shift_company`: (`company_id`) → `mst_moto_company` (company_id)


### `t_shift_schedule`

lịch làm việc dự kiến của Staff (シフト表)


> 🔒 **RLS bật** (FORCE). Backend phải set `app.current_saki`/`app.current_moto`/`app.current_role` trước khi query.


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `schedule_id` | BIGINT (IDENTITY) |  |  | PK | Mã lịch xếp ca (số tự tăng) — khóa chính |
| 2 | `contract_no` | VARCHAR(14) | NO |  | FK | Hợp đồng liên quan |
| 3 | `staff_code` | VARCHAR(16) | NO |  | FK | Nhân viên được xếp ca |
| 4 | `saki_company_id` | VARCHAR(20) | NO |  |  | Công ty SAKI (dùng cho RLS) |
| 5 | `work_date` | DATE | NO |  |  | Ngày làm việc |
| 6 | `company_id` | VARCHAR(20) |  |  | FK | Công ty MOTO sở hữu |
| 7 | `shift_code` | VARCHAR(20) |  |  | FK | Mã ca áp dụng (FK mst_moto_shift) |
| 8 | `start_time` | SMALLINT |  |  |  | Giờ bắt đầu thực tế (phút từ 00:00) |
| 9 | `end_time` | SMALLINT |  |  |  | Giờ kết thúc thực tế (phút từ 00:00) |
| 10 | `is_day_off_flg` | SMALLINT | NO | `0` |  | Cờ ngày nghỉ — 0:làm việc 1:nghỉ |
| 11 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |
| 12 | `updated_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm cập nhật gần nhất |
| 13 | `created_by` | VARCHAR(100) | NO |  |  | User/tiến trình tạo bản ghi |
| 14 | `updated_by` | VARCHAR(100) | NO |  |  | User/tiến trình cập nhật gần nhất |

**Ràng buộc & Index:**

- **PK**: (`schedule_id`)
- **FK** `fk_schedule_contract`: (`contract_no`) → `t_contract` (contract_no)
- **FK** `fk_schedule_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **FK** `fk_schedule_shift`: (`company_id`, `shift_code`) → `mst_moto_shift` (company_id, shift_code)
- **INDEX** `idx_shift_saki`: (`saki_company_id`)
- **INDEX (composite)** `idx_shift_staff_date`: (`staff_code`, `work_date`)
- **INDEX** `idx_shift_contract`: (`contract_no`)
- **RLS policy** — p_shift_schedule_saki: USING (saki_company_id = current_setting('app.current_saki', true)) WITH CHECK (saki_company_id = current_setting('app.current_saki', true))
- **RLS policy** — p_shift_schedule_moto_owner: USING (current_setting('app.current_role', true) = 'moto_owner') WITH CHECK (current_setting('app.current_role', true) = 'moto_owner')


### `t_staff_notification_read`

trạng thái đọc thông báo của Staff App


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `staff_code` | VARCHAR(16) | NO |  | PK FK | Nhân viên đọc thông báo |
| 2 | `notification_id` | BIGINT | NO |  | PK | Thông báo được đọc (logical FK platform.notification_history) |
| 3 | `is_read_flg` | SMALLINT | NO | `1` |  | Cờ đã đọc — 0:chưa đọc 1:đã đọc |
| 4 | `read_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm đọc |

**Ràng buộc & Index:**

- **PK**: (`staff_code`, `notification_id`) — composite
- **FK** `fk_staff_notif_staff`: (`staff_code`) → `mst_staff` (staff_code)
- **INDEX** `idx_staff_notif_notif`: (`notification_id`)


### `t_moto_batch_execution_history`

lịch sử async job của MOTO


| # | Trường | Kiểu | Null | Mặc định | Khóa | Mô tả |
|---|--------|------|------|----------|------|-------|
| 1 | `job_id` | BIGINT (IDENTITY) |  |  | PK | Mã job batch (số tự tăng) — khóa chính |
| 2 | `company_id` | VARCHAR(20) | NO |  | FK | Công ty MOTO sở hữu job |
| 3 | `job_type` | VARCHAR(50) | NO |  |  | Loại job (tổng hợp tháng/36協定/tạo hóa đơn...) |
| 4 | `execute_user_id` | VARCHAR(100) | NO |  |  | Người chạy job |
| 5 | `status` | SMALLINT | NO | `1` |  | Trạng thái — 1:đang chạy 2:hoàn tất 3:lỗi |
| 6 | `file_url` | VARCHAR(255) |  |  |  | Đường dẫn file kết quả (nếu có) |
| 7 | `error_log` | TEXT |  |  |  | Log lỗi khi job thất bại |
| 8 | `started_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm bắt đầu |
| 9 | `finished_at` | TIMESTAMPTZ |  |  |  | Thời điểm kết thúc |
| 10 | `created_at` | TIMESTAMPTZ | NO | `now()` |  | Thời điểm tạo bản ghi |

**Ràng buộc & Index:**

- **PK**: (`job_id`)
- **FK** `fk_moto_batch_company`: (`company_id`) → `mst_moto_company` (company_id)
- **INDEX** `idx_moto_batch_company`: (`company_id`)
- **INDEX** `idx_moto_batch_type`: (`job_type`)
