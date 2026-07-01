# Global Master Schema

Schema vật lý: `global_master` — 4 bảng.
Dữ liệu từ điển dùng chung, bất biến, an toàn để export/nhân bản cùng Tenant.
*(Lưu ý Kiến trúc: BẮT BUỘC dùng tên `global_master`. TUYỆT ĐỐI KHÔNG lưu các bảng này vào schema `public` mặc định của PostgreSQL để tránh lẫn lộn với các Database Extensions và ngăn chặn rủi ro bảo mật Search Path Hijacking).*

Tenant schema chỉ tham chiếu `global_master` bằng Logical FK. Không tạo FK vật lý xuyên schema để tránh phá vỡ tenant backup/restore/export/import. Với dữ liệu cần export/import/cross-environment, ưu tiên lưu stable master code hoặc id+code.

| Bảng | Chức năng | → Tham chiếu (FK ra) | ← Được tham chiếu (FK vào) |
| --- | --- | --- | --- |
| `mst_industry` | Master ngành nghề (業種, ~83 items) phân loại lĩnh vực kinh doanh SAKI. Hỗ trợ phân cấp (parent_industry_id). Sync với CARIMA Matching qua matching_industry_id. | `mst_industry` (self) | `mst_job_type` |
| `mst_job_type` | Master loại công việc (職種, ~418 items). +parent_job_type_id (phân cấp), +industry_id (link 業種), +matching_job_category_id (CARIMA sync). | `mst_industry`, `mst_job_type` (self) | `mst_saki_custom_job_type` (logical FK) |
| `mst_skill` | Master kỹ năng (~345 items). +matching_skill_id (CARIMA sync). | — | `mst_staff_skill` (logical FK), `mst_saki_custom_job_type_skill` (logical FK) |
| `mst_qualification` | Master bằng cấp/chứng chỉ (~134 items). +is_national, +has_expiry, +issuing_body, +matching_qualification_id (CARIMA sync). | — | `mst_staff_qualification` (logical FK) |
| `mst_expense_category` | Master hạng mục chi phí dùng chung toàn hệ thống (phục vụ thanh toán hộ/đi lại). | — | — |
| `mst_ledger_job_category` | Master phân loại nghiệp vụ pháp lý (台帳職種/業務区分: 401-418/501-510/901-903/999) cho 派遣元・派遣先管理台帳 và 日雇派遣例外業務; khác `mst_job_type`. | — | `t_contract` (logical FK qua `job_category_payload`) |

---