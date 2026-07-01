# Global Master Schema

> Schema vật lý bắt buộc: `global_master`.
Không lưu các bảng này trong `public`.
Dữ liệu trong schema này là dictionary/master dùng chung toàn hệ thống, read-mostly, không chứa dữ liệu vận hành Platform hoặc dữ liệu tenant.
Tenant chỉ tham chiếu bằng Logical FK. Không tạo FK vật lý từ tenant schema sang `global_master`.
Khi export/backup tenant, dữ liệu `global_master` có thể được export/replicate như dictionary tham chiếu, nhưng không kéo theo dữ liệu bảo mật của Platform.
> 

## Global Master Rules

- `global_master` chỉ chứa dữ liệu từ điển dùng chung, an toàn để replicate/export.
- Master code như `job_code`, `skill_code`, `qualification_code`, `expense_code` là khóa ổn định cross-environment.
- `id` chỉ là surrogate key nội bộ trong một database instance.
- Tenant transaction/master nên lưu global master reference bằng `code`, hoặc lưu cả `id + code` nếu cần join nhanh.
- Không thay đổi semantic của một code đã publish. Nếu semantic thay đổi, tạo code mới hoặc version mới.
- Dữ liệu pháp lý đã final như contract/invoice/estimate vẫn phải snapshot label/name tại thời điểm phát hành.

## mst_industry (Từ điển Ngành nghề - 業種)

> **[M07 NEW]** ~83 ngành nghề, phân loại lĩnh vực kinh doanh của SAKI. Đồng bộ từ CARIMA Matching (BR-1161). Hỗ trợ phân cấp `parent_industry_id` cho 大分類→中分類.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `industry_code` | VARCHAR(10) | No |  | **UK** | Mã ngành nghề (immutable sau tạo — BR-1102; pattern `^[A-Z0-9_]{1,10}$` — BR-1115) |
| `industry_name_ja` | VARCHAR(100) | No |  |  | Tên tiếng Nhật (BR-1111 bắt buộc) |
| `industry_name_en` | VARCHAR(100) | Yes | NULL |  | Tên tiếng Anh (tùy chọn) |
| `parent_industry_id` | BIGINT | Yes | NULL | FK, IDX | Tham chiếu mst_industry(id) — phân cấp 大分類. BR-1114: parent phải active, không vòng lặp |
| `display_order` | INTEGER | No | 0 | IDX | Thứ tự hiển thị (BR-1113: positive int; auto = max+10 nếu null) |
| `description` | TEXT | Yes | NULL |  | Mô tả chi tiết |
| `matching_industry_id` | VARCHAR(50) | Yes | NULL | IDX | Mã tham chiếu sang CARIMA Matching — BR-1161/BR-1162; NULL = bản ghi tạo trên Staffing chờ sync ngược |
| `valid_from` | DATE | Yes | NULL |  | Ngày bắt đầu hiệu lực |
| `valid_to` | DATE | Yes | NULL |  | Ngày hết hiệu lực |
| `status` | SMALLINT | No | 1 | IDX | 1: Active, 0: Inactive (BR-1101 soft disable, không xóa vật lý) |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete — sử dụng status=0 là chính, deleted_at chỉ dùng cho hard-delete tracking) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **FK** fk_industry_parent: (parent_industry_id) → mst_industry(id)
- **CHECK** chk_industry_code_format: industry_code ~ '^[A-Z0-9_]{1,10}$'
- **Trigger** trg_industry_updated_at: BEFORE UPDATE → fn_set_updated_at()

## mst_job_type (Từ điển Loại công việc tiêu chuẩn - 職種)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `job_code` | VARCHAR(50) | No |  | **UK** | Mã công việc tiêu chuẩn (immutable sau tạo — BR-1102) |
| `job_name` | VARCHAR(100) | No |  |  | Tên công việc (VD: Data Entry, IT Support) — BR-1111 bắt buộc |
| `job_name_en` | VARCHAR(100) | Yes | NULL |  | **[M07]** Tên tiếng Anh (tùy chọn; cần cho CARIMA Matching multilingual) |
| `parent_job_type_id` | BIGINT | Yes | NULL | FK, IDX | **[M07]** Phân cấp 大分類→小分類; BR-1114 enforce: parent active + no cycle |
| `industry_id` | BIGINT | Yes | NULL | FK, IDX | **[M07]** Liên kết ngành nghề (mst_industry.id) — filter job theo industry trong Module 03 |
| `description` | TEXT | Yes | NULL |  | Mô tả |
| `display_order` | INTEGER | No | 0 |  | Thứ tự hiển thị |
| `matching_job_category_id` | VARCHAR(50) | Yes | NULL | IDX | **[M07]** Mã tham chiếu CARIMA Matching — BR-1161 |
| `valid_from` | DATE | Yes | NULL |  | Ngày bắt đầu hiệu lực nếu cần |
| `valid_to` | DATE | Yes | NULL |  | Ngày hết hiệu lực nếu cần |
| `status` | SMALLINT | No | 1 |  | 1: Active, 0: Inactive |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **FK** fk_job_type_parent: (parent_job_type_id) → mst_job_type(id)
- **FK** fk_job_type_industry: (industry_id) → mst_industry(id)

## mst_skill (Từ điển Kỹ năng tiêu chuẩn - スキル - Phục vụ CARIMA Matching)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `parent_skill_id` | BIGINT | Yes | NULL | FK, IDX | **[Reviewer Fix 9]** Phân cấp tree: row level N reference row level N-1. NULL chỉ với row level=1 (大項目 root). Same-table self-ref pattern (giống mst_industry.parent_industry_id, mst_job_type.parent_job_type_id). Enforce: parent active + no cycle (BR-1114). |
| `skill_level` | SMALLINT | No |  | IDX | **[Reviewer Fix 9]** Level discriminator: 1=大項目 (vd "OAスキル"), 2=中項目 (vd "Microsoft Office"), 3=小項目 (vd "Excel"), 4=スキル名 leaf (vd "表の作成・編集"). Chỉ row level=4 mới là kỹ năng thực sự gán cho staff. |
| `skill_code` | VARCHAR(50) | No |  | **UK** | Mã kỹ năng (immutable sau tạo — BR-1102). Áp dụng cho mọi level (vd "CAT_OA_001" cho 大項目, "SKL_EXCEL_TABLE_EDIT" cho leaf). |
| `skill_name` | VARCHAR(100) | No |  |  | Tên (theo skill_level: tên 大項目/中項目/小項目/スキル名). BR-1111 bắt buộc. |
| `skill_name_en` | VARCHAR(100) | Yes | NULL |  | **[M07]** Tên tiếng Anh (tùy chọn) |
| `description` | TEXT | Yes | NULL |  | Mô tả |
| `display_order` | INTEGER | No | 0 |  | Thứ tự hiển thị trong cùng parent |
| `matching_skill_id` | VARCHAR(50) | Yes | NULL | IDX | **[M07]** Mã tham chiếu CARIMA Matching — BR-1161. Thường chỉ leaf (skill_level=4) có. |
| `valid_from` | DATE | Yes | NULL |  | Ngày bắt đầu hiệu lực nếu cần |
| `valid_to` | DATE | Yes | NULL |  | Ngày hết hiệu lực nếu cần |
| `status` | SMALLINT | No | 1 |  | 1: Active, 0: Inactive |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **FK** fk_skill_parent: (parent_skill_id) → mst_skill(id)
- **CHECK** chk_skill_level: skill_level IN (1, 2, 3, 4) — **[Reviewer Fix 9]**
- **CHECK** chk_skill_level1_no_parent: (skill_level = 1 AND parent_skill_id IS NULL) OR (skill_level > 1 AND parent_skill_id IS NOT NULL) — **[Reviewer Fix 9]** level=1 (大項目) là root, không có parent; level 2-4 bắt buộc có parent
- **INDEX** idx_skill_parent_level: (parent_skill_id, skill_level) WHERE deleted_at IS NULL — **[Reviewer Fix 9]** lookup children
- **INDEX** idx_skill_level: (skill_level) WHERE deleted_at IS NULL AND status = 1 — **[Reviewer Fix 9]** filter "show all leaf skills" common query

## mst_qualification (Từ điển Chứng chỉ / Bằng cấp tiêu chuẩn)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `qualification_code` | VARCHAR(50) | No |  | **UK** | Mã chứng chỉ (VD: IT_PASS, TOEIC_800) — immutable sau tạo (BR-1102) |
| `qualification_name` | VARCHAR(100) | No |  |  | Tên chứng chỉ/bằng cấp |
| `qualification_name_en` | VARCHAR(100) | Yes | NULL |  | **[M07]** Tên tiếng Anh (tùy chọn) |
| `category_name` | VARCHAR(100) | Yes | NULL |  | Nhóm bằng cấp (VD: IT, Ngoại ngữ, Y tế) |
| `issuing_body` | VARCHAR(200) | Yes | NULL |  | **[M07]** Cơ quan cấp chứng chỉ (display-only) |
| `is_national` | SMALLINT | No | 0 | IDX | **[M07]** 1=chứng chỉ quốc gia / 0=khác — filter trong yêu cầu phái cử có ràng buộc pháp lý |
| `has_expiry` | SMALLINT | No | 0 | IDX | **[M07]** 1=có thời hạn / 0=vĩnh viễn — alert gia hạn bằng cấp staff |
| `description` | TEXT | Yes | NULL |  | Mô tả |
| `display_order` | INTEGER | No | 0 |  | Thứ tự hiển thị |
| `matching_qualification_id` | VARCHAR(50) | Yes | NULL | IDX | **[M07]** Mã tham chiếu CARIMA Matching — BR-1161 |
| `valid_from` | DATE | Yes | NULL |  | Ngày bắt đầu hiệu lực nếu cần |
| `valid_to` | DATE | Yes | NULL |  | Ngày hết hiệu lực nếu cần |
| `status` | SMALLINT | No | 1 |  | 1: Active, 0: Inactive |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |
- **CHECK** chk_qualification_is_national: is_national IN (0, 1)
- **CHECK** chk_qualification_has_expiry: has_expiry IN (0, 1)

## mst_expense_category (Từ điển Hạng mục chi phí thanh toán hộ)

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `expense_code` | VARCHAR(50) | No |  | **UK** | Mã chi phí |
| `expense_name` | VARCHAR(100) | No |  |  | Tên chi phí (VD: Phí đi lại, Tiền nhà) |
| `is_taxable_default` | SMALLINT | No | 1 |  | Mặc định có chịu thuế không |
| `description` | TEXT | Yes | NULL |  | Mô tả |
| `display_order` | INTEGER | No | 0 |  | Thứ tự hiển thị |
| `valid_from` | DATE | Yes | NULL |  | Ngày bắt đầu hiệu lực nếu cần |
| `valid_to` | DATE | Yes | NULL |  | Ngày hết hiệu lực nếu cần |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

## mst_ledger_job_category (Từ điển Phân loại nghiệp vụ pháp lý - 台帳職種 / 業務区分)

> Mã pháp lý dùng cho 派遣元/派遣先管理台帳 và 日雇派遣例外業務 (401-418 = 令和5年第1項各号 26業務; 501-510 = các号 khác; 901-903 = 有期PJ/日数限定/代替要員; 999 = 自由化業務). KHÁC `mst_job_type` (loại công việc tự do) — ngữ nghĩa pháp lý riêng. Tenant tham chiếu logic bằng `category_code`.
> 

| Trường | Kiểu | Null | Mặc định | Khóa/Index | Mô tả |
| --- | --- | --- | --- | --- | --- |
| `id` | BIGINT | No | IDENTITY | **PK** | Khóa chính |
| `category_code` | VARCHAR(3) | No |  | **UK** | Mã 業務区分 (401-418/501-510/901-903/999) |
| `category_name` | VARCHAR(160) | No |  |  | Tên nghiệp vụ (VD: 令和5年第1項第1号 情報処理システム開発関係) |
| `category_group` | VARCHAR(20) | Yes | NULL |  | STANDARD_26 / FREE / EXCEPTION |
| `display_order` | INTEGER | No | 0 |  | Thứ tự hiển thị |
| `valid_from` | DATE | Yes | NULL |  | Ngày bắt đầu hiệu lực |
| `valid_to` | DATE | Yes | NULL |  | Ngày hết hiệu lực |
| `status` | SMALLINT | No | 1 |  | 1: Active, 0: Inactive |
| `created_at` | TIMESTAMP | No | NOW() |  | Thời điểm tạo bản ghi |
| `updated_at` | TIMESTAMP | No | NOW() |  | Thời điểm cập nhật bản ghi lần cuối |
| `deleted_at` | TIMESTAMP | Yes | NULL | IDX | Thời điểm xóa bản ghi (Soft Delete) |
| `created_by` | BIGINT | Yes | NULL |  | ID của người dùng đã tạo bản ghi |
| `updated_by` | BIGINT | Yes | NULL |  | ID của người dùng cập nhật bản ghi |

---