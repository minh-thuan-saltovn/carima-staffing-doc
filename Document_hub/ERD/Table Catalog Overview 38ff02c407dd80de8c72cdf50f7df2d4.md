# Table Catalog Overview

> Tổng quan toàn bộ bảng, nhóm theo schema. Mỗi bảng gồm: mô tả chức năng (tiếng Việt) và các bảng có **quan hệ khóa ngoại trong cùng schema**.
> 

> • **→ Tham chiếu**: bảng này có FK trỏ tới (phụ thuộc bảng kia).
• **← Được tham chiếu**: bảng kia có FK trỏ về bảng này.
Quan hệ xuyên schema (logical FK) không có ràng buộc vật lý; chỉ liệt kê trong catalog khi cần làm rõ nghiệp vụ và phải đánh dấu rõ là (logical FK).
Quy tắc Index: Toàn bộ các cột đóng vai trò là Logical FK (như saki_tenant_id, moto_tenant_id, staff_code, contract_no) bắt buộc phải được đánh Index (Chỉ mục) để đảm bảo hiệu năng khi Backend thực hiện Application-Level JOIN.
> 

---

## Quy ước đặt tên

Tên bảng sử dụng `snake_case`, dạng số ít (singular), và tuân theo cấu trúc chung:

```
<purpose_prefix>_<domain_or_business_prefix>_<entity_name>
```

Ví dụ:

```
mst_saki_user
cfg_moto_invoice_default
t_dispatch_inquiry
t_contract_status_history
tmp_platform_pwd_reset_token
```

### Purpose Prefix

| Prefix | Ý nghĩa | Cách dùng |
| --- | --- | --- |
| `mst_` | Master / reference data | Dữ liệu gốc, danh mục, user/role/permission, master nghiệp vụ |
| `cfg_` | Configuration / setting | Cấu hình hệ thống, tenant, công ty, nghiệp vụ |
| `t_` | Transaction / runtime business data | Dữ liệu phát sinh trong vận hành nghiệp vụ |
| `tmp_` | Temporary / short-lived data | Token, setup credential, draft hoặc dữ liệu tạm có vòng đời ngắn |

### Domain / Business Prefix

Phần sau `purpose_prefix` thể hiện domain sở hữu hoặc nghiệp vụ chính của bảng.

| Prefix | Ý nghĩa |
| --- | --- |
| `global_master` | Dữ liệu master dùng chung cho các tenant |
| `platform` | Dữ liệu vận hành Platform / JustFine |
| `tenant` | Dữ liệu quản lý tenant ở Platform schema |
| `saki` | Dữ liệu thuộc SAKI tenant |
| `moto` | Dữ liệu thuộc MOTO tenant |
| `staff` | Dữ liệu liên quan Staff |
| `dispatch` | Nghiệp vụ yêu cầu phái cử / báo giá |
| `contract` | Nghiệp vụ hợp đồng |
| `attendance` | Nghiệp vụ chấm công |
| `invoice` | Nghiệp vụ hóa đơn |
| `notification` | Nghiệp vụ thông báo |

### Quy ước hậu tố

| Suffix | Ý nghĩa |
| --- | --- |
| `_detail` | Dòng chi tiết / child rows |
| `_history` | Lịch sử nghiệp vụ hoặc lịch sử thay đổi trạng thái |
| `_setting` | Cấu hình |
| `_token` | Token dùng một lần hoặc có thời hạn |
| `_credential` | Thông tin xác thực hoặc setup credential |
| `_relation` | Quan hệ giữa hai business entities |
| `_scope` | Phạm vi truy cập dữ liệu |
| `_approval` | Dữ liệu phê duyệt |
| `_log` | Log kỹ thuật hoặc audit/provision log |

### Rules

- Tên bảng dùng số ít (singular), không dùng plural.
- Không dùng động từ/API action trong tên bảng, ví dụ không đặt `t_submit_contract` hoặc `t_get_invoice_result`.
- Không lặp lại schema name nếu business prefix đã đủ rõ, nhưng với SAKI/MOTO tenant vẫn giữ `saki`/`moto` trong tên bảng để Laravel model, migration và dynamic schema dễ đọc.
- Các bảng core business có thể dùng business prefix trực tiếp, ví dụ `t_contract`, `t_invoice`, `t_attendance_daily`, không bắt buộc thêm `moto`.
- Bảng chỉ lưu metadata tham chiếu secret bên ngoài DB; secret value không được lưu trong database.
- Tất cả thông tin private như tenant DB connection, DB user/password, technical account, access key, private key, SSO client secret phải lưu trong AWS Secrets Manager. PostgreSQL chỉ lưu `secret_path`, ARN hoặc reference để service layer resolve khi cần.

---

## Architectural Notes

- Physical FK (Khóa ngoại vật lý) chỉ được dùng trong cùng một schema.
- Cross-schema references (Tham chiếu liên schema) là các tham chiếu logic và bắt buộc phải được validate bởi service layer (tầng dịch vụ).
- Mọi logical FK (Khóa ngoại logic) được dùng để lọc (filtering) hoặc kết hợp (joining) đều phải có một index rõ ràng trong schema sở hữu (owning schema).
- Cross-tenant writes (Các thao tác ghi liên tenant) phải được thực thi thông qua một system role được kiểm soát và phải được ghi nhận vào bảng outbox/job/audit.
- Các chính sách RLS (Row-Level Security) phải phân biệt ít nhất bốn ngữ cảnh (contexts): platform_admin, saki_user, moto_user, và staff_user.
- Quyền truy cập Staff Web luôn luôn phải được giới hạn phạm vi (scoped) bởi staff_code đã được định danh (authenticated), không dựa vào tham số truyền từ request (request parameter).
- Các logical company identifier dùng cho cross-schema reference như saki_tenant_id, moto_tenant_id phải là immutable; nếu cần đổi mã hiển thị thì dùng display code riêng hoặc lưu snapshot.

---

### Rủi ro kiến trúc & Lưu ý khi triển khai (Architectural Risks & Mitigations)

Hệ thống được thiết kế theo dạng Multi-tenant, Event-Driven và có lượng dữ liệu lớn (có thể lên tới 1.000.000+ record/tháng - Phase 3).
Đội ngũ Backend cần đặc biệt tuân thủ các lưu ý sau khi thao tác với Database:

#### 1. Rủi ro Phình to và Nghẽn Event Queue (Outbox/Inbox Pattern)

- **Vấn đề:** Để xử lý Ghi chéo (Cross-write) giữa SAKI và MOTO (ví dụ: SAKI duyệt chấm công bắn sang MOTO), hệ thống sử dụng bảng `t_outbox_event` và `t_inbox_event`. Với lượng dữ liệu lớn, bảng này sẽ phình to rất nhanh, làm chậm tốc độ query của Job Worker.
- **Biện pháp xử lý (Mitigation):**
    - Bắt buộc phải viết một Cronjob (Cleanup Job) chạy ngầm để xóa hoặc Archive (lưu trữ ra cold storage) các event có status là `COMPLETED` sau một khoảng thời gian (ví dụ: 7 ngày).
    - Phải thiết lập hệ thống Monitor độ trễ (Delay) của Queue.

#### 2. Rủi ro Rác dữ liệu từ Logical Foreign Key (Dangling FKs)

- **Vấn đề:** Để xử lý Ghi chéo, hệ thống sử dụng các Khóa ngoại logic (Logical FK) xuyên Schema, ví dụ: bảng Hợp đồng ở schema MOTO lưu `saki_tenant_id`. Nếu Platform Admin xóa hoặc vô hiệu hóa một công ty SAKI ở bảng `mst_tenant_registry`, Database sẽ không báo lỗi vì không có ràng buộc vật lý, dẫn đến MOTO truy vấn ra dữ liệu SAKI bị lỗi.
- **Biện pháp xử lý (Mitigation):** Bắt buộc áp dụng cơ chế **Cascading Event** ở tầng Application. Khi một SAKI bị Suspend/Delete, Platform phải bắn Broadcast Event xuống các tenant MOTO liên quan để khóa (Freeze) các giao dịch đang trỏ tới SAKI đó.

#### 3. Rủi ro "Khóa cứng" người duyệt khi nhân sự nghỉ việc

- **Vấn đề:** Theo logic, ID của người duyệt (ví dụ: Người duyệt chấm công) được đóng cứng (Hardcode) trực tiếp vào từng bản ghi hợp đồng (`timecard_approver1_id`). Nếu vị Sếp đó nghỉ việc hoặc chuyển phòng ban, hàng ngàn Staff sẽ không có người duyệt chấm công.
- **Biện pháp xử lý (Mitigation):**
    - Backend bắt buộc phải thiết kế một API hỗ trợ tính năng **"Sửa đổi hàng loạt người duyệt" (承認者一括修正)**. API này sẽ quét qua toàn bộ bảng `t_contract` đang active, tìm các ID người duyệt cũ và thay thế bằng ID người duyệt mới chỉ trong 1 thao tác, giống hệt tính năng upload CSV thay đổi của e-staffing gốc.

#### 4. Quy tắc chuẩn hóa định dạng Thời gian (Date/Month Boundary Conversion)

- **Vấn đề:** Các hệ thống của Nhật Bản (đặc biệt là e-staffing) bắt buộc định dạng tháng nhập/xuất file CSV phải là `YYYY/MM` (VD: `2023/12`). Nếu sai sẽ bị trả về lỗi "フォーマットエラー". Tuy nhiên, việc lưu trữ dấu slash (`/`) vào Database vi phạm chuẩn ISO 8601, gây khó khăn cho việc query và tính toán.
- **Quyết định Kiến trúc (Canonical DB & Boundary Convert):**
    - **Tại Database:** Tất cả các cột lưu tháng (như `target_month`, `billing_month`, `closing_month`) được thiết kế là `VARCHAR(7)` và **bắt buộc lưu theo chuẩn ISO 8601: `YYYY-MM`** (VD: `2023-12`).
    - **Tại Application/Boundary Layer:** Mọi thao tác Import/Export CSV, hoặc gọi API ra hệ thống ngoài đều phải đi qua các class DTO/Serializer để **thực hiện Replace/Convert tự động** giữa `YYYY-MM` (DB) và `YYYY/MM` (External). Dev tuyệt đối không thao tác chuỗi (`str_replace`) trực tiếp trong logic Core.

#### 5. Quy ước Soft Delete & Phân loại Unique Constraint

- **Vấn đề:** Việc áp dụng Unique Constraint cùng với `deleted_at` cần sự rạch ròi về mặt ngữ nghĩa (Semantics). Nếu dùng sai, hệ thống sẽ chặn tái sử dụng mã hợp lệ hoặc cho phép trùng lặp dữ liệu nhạy cảm.
- **Quyết định Kiến trúc:**
    - **Đối với Business Master Code (Có khả năng tái sử dụng):** Các mã nghiệp vụ do người dùng quản lý (như `office_code`, `department_code`, `staff_code`) BẮT BUỘC dùng **Partial Unique Index** (`UNIQUE ... WHERE deleted_at IS NULL`). Điều này cho phép cấp lại mã sau khi bản ghi cũ đã bị xóa mềm.
    - **Đối với Technical / Immutable / Security Data (Không được phép tái sử dụng):** Các định danh như `idempotency_key`, `token`, bảng Event, phiên bản Document Artifact, hoặc Audit Log BẮT BUỘC dùng **Full Unique Index** (Không có mệnh đề WHERE). Dữ liệu này phải là duy nhất trên toàn bộ lịch sử database.
    - **Đối với Platform/Tenant Identifiers:** Các mã như `tenant_code`, `db_schema_name`, `api_key_prefix` dùng Full Unique Index để tránh các sự cố nghiêm trọng về DNS, khôi phục schema, hoặc nhầm lẫn Audit Log.

#### 6. Quy ước chống đụng độ Mã Giao Dịch (Invariant cho Phase 3)

- **Vấn đề:** Hệ thống giữ `contract_no`, `inquiry_no`, `invoice_no` làm Khóa chính (PK) thay vì Surrogate ID để tránh đứt gãy kiến trúc Khóa ngoại (15+ bảng liên quan). Tuy nhiên, nếu sau này áp dụng mô hình Multi-company per Tenant, việc dựa dẫm vào "niềm tin" rằng các mã này sẽ không trùng là cực kỳ rủi ro.
- **Quyết định Kiến trúc (Enforcement Rules):**
    - Database **vẫn giữ nguyên PK/Unique** trên các cột mã giao dịch này.
    - Bắt buộc phải có một **Sequence Generator Service duy nhất** chịu trách nhiệm sinh mã. Tuyệt đối **cấm Client/User tự nhập** các mã này qua API.
    - **Định dạng Mã (Document Format):** Phải chứa Tiền tố bất biến của Tenant và Company (để chuỗi sequence là tenant/schema-wide unique).
        - *Chuẩn định dạng:* `{tenant_prefix}-{company_prefix}-{type}{YYMM}-{sequence}` (Ví dụ: `T123-C456-C2606-0001` cho Contract).
    - Quy tắc này đảm bảo các mã giao dịch lõi là **Globally Unique** trong toàn bộ Tenant/Schema một cách tuyệt đối an toàn.

#### 7. Logical identifier

- Các logical identifier dùng cho cross-schema reference như `saki_tenant_id`, `moto_tenant_id`, `saki_company_code`, `moto_company_code` phải là immutable (không thay đổi); trong đó `tenant_id` (Kiểu BIGINT) được dùng làm khóa chính để định tuyến và áp dụng RLS (Row-Level Security), còn `company_code` (Kiểu VARCHAR) dùng để hiển thị và lưu snapshot.

#### 8. Quy tắc Quản lý DRAFT & Versioning (Estimate & Inquiry)

- **Estimate DRAFT:** Trong cùng một chuỗi `estimate_group_id`, hệ thống BẮT BUỘC chỉ cho phép tồn tại tối đa 1 bản ghi có trạng thái DRAFT. Việc lưu/sửa nháp sẽ cập nhật đè lên bản DRAFT này. Khi Submit, bản DRAFT sẽ chốt thành Version N, đồng thời các bản ghi Version cũ tự động chuyển thành trạng thái `SUPERSEDED`.
- **Inquiry Revisions:** Khi MOTO nhận yêu cầu sửa đổi (Revision) từ SAKI, hệ thống BẮT BUỘC ghi insert một bản ghi mới (Immutable) thay vì update bản cũ, để đảm bảo Báo giá (Estimate) mà MOTO đã làm trước đó không bị mất Context (nguồn gốc dữ liệu). Bản cũ chuyển sang `SUPERSEDED`.

#### 9. Quy tắc Tái kết nối Quan hệ MOTO-SAKI (Relation Reconnection Rule)

- **Vấn đề:** Trong nghiệp vụ phái cử, MOTO và SAKI có thể hủy hợp tác và ký kết lại sau một thời gian. Nếu Database tạo ID (Record) mới cho cùng một cặp MOTO-SAKI cũ, toàn bộ lịch sử (Hợp đồng, Hóa đơn, Báo giá, Audit) sẽ bị phân mảnh (Fragmented), gây đứt gãy Business Identity.
- **Quyết định Kiến trúc (2-Tier Rule):**
    - **Tầng Database (An toàn vật lý):** Áp dụng Partial Unique (`WHERE deleted_at IS NULL`) cho các bảng quan hệ (`mst_moto_saki_relation` tại Platform và `cfg_moto_client_company` tại MOTO) để Database không bị khóa chết nếu buộc phải tạo mới.
    - **Tầng Application (Toàn vẹn nghiệp vụ):** Khi tái kết nối, Backend **BẮT BUỘC ưu tiên RESTORE** bản ghi soft-deleted cũ. Việc tạo Row mới (`RELINKED_NEW_RECORD`) là đường ngoại lệ, chỉ được dùng khi dữ liệu cũ không thể restore (do pháp lý/migration) và **BẮT BUỘC phải ghi rõ lý do vào log**.

#### 10. Triết lý sử dụng "NULL" vs "Sentinel 0" cho Global Data

- **Vấn đề:** Khi một dữ liệu áp dụng cho toàn bộ hệ thống (Broadcast) hoặc xuất phát từ hệ thống ngoài, giá trị `tenant_id` sẽ không trỏ cụ thể về một Tenant nào. Việc dùng `NULL` hay `0` cần được rạch ròi.
- **Quyết định Kiến trúc:**
    - **Đối với hệ thống Event Queue (Inbox/Outbox):** BẮT BUỘC dùng giá trị Sentinel `0`. (Lý do: Trong PostgreSQL, `NULL` được xem là distinct trong `UNIQUE constraint`, nên nhiều dòng có `NULL` vẫn có thể cùng tồn tại, làm vô hiệu hóa khả năng chống trùng lặp Idempotency).
        - *Trong Platform Schema:* `0` có thể mang nghĩa là `SYSTEM` hoặc các nguồn ngoài `EXT_*` (Inbox) và `SYSTEM` hoặc `BROADCAST` (Outbox).
        - *Trong Tenant Schema (SAKI/MOTO):* `0` có thể mang nghĩa là `PLATFORM` hoặc `SYSTEM` (Inbox) và `PLATFORM` hoặc `BROADCAST` (Outbox).
    - **Đối với các bảng Nghiệp vụ thông thường (Notification, Backup...):** Dùng giá trị `NULL` để biểu thị "Toàn hệ thống". Dev không được áp dụng máy móc số `0` ra ngoài hệ thống Event.

## Tổng quan số lượng

| Schema | Số bảng |
| --- | --- |
| Global Master Schema | 6 |
| Platform Schema | 42 |
| SAKI Schema | 58 |
| MOTO Schema | 118 |
| **Tổng** | **224** |

---