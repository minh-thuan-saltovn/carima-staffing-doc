# Carima-Staffing — Danh mục bảng (Table Catalog)

> Tổng quan toàn bộ **107 bảng**, nhóm theo schema. Mỗi bảng gồm: mô tả chức năng (tiếng Việt) và các bảng có **quan hệ khóa ngoại trong cùng schema**.

> - **→ Tham chiếu**: bảng này có FK trỏ tới (phụ thuộc bảng kia).
> - **← Được tham chiếu**: bảng kia có FK trỏ về bảng này.
> Quan hệ xuyên schema (logical FK) không liệt kê ở đây vì không có ràng buộc vật lý.
> Quy tắc Index: Toàn bộ các cột đóng vai trò là Logical FK (như saki_company_id, moto_company_id, staff_code, contract_no) bắt buộc phải được đánh Index (Chỉ mục) để đảm bảo hiệu năng khi Backend thực hiện Application-Level JOIN.


---

## Tổng quan số lượng

| Schema | Số bảng |
|--------|---------|
| Platform Schema | 10 |
| SAKI Schema | 37 |
| MOTO Schema | 60 |
| **Tổng** | **107** |

---


# Platform Schema

Schema vật lý: `platform` — 10 bảng.


| Bảng | Chức năng | → Tham chiếu (FK ra) | ← Được tham chiếu (FK vào) |
|------|-----------|----------------------|----------------------------|
| `tenant_registry` | Sổ đăng ký tenant: ánh xạ mỗi công ty SAKI/MOTO tới schema và role PostgreSQL riêng. Trung tâm định tuyến toàn hệ thống. | — | `mst_moto_saki_relation`, `t_tenant_usage_metric`, `t_usage_billing` |
| `admin_user` | Tài khoản quản trị viên cấp Platform (vận hành Carima), tách biệt với người dùng tenant. | — | — |
| `tenant_provision_log` | Nhật ký cấp phát/thay đổi tenant (tạo schema/role/migrate), phục vụ audit và gỡ lỗi onboarding. | — | — |
| `mst_moto_saki_relation` | Bản đồ quan hệ N–N giữa MOTO và SAKI ở cấp Platform: định tuyến yêu cầu phái cử giữa các tenant. | `tenant_registry` | — |
| `mst_job_type` | Master loại công việc (職種) dùng chung toàn hệ thống. | — | — |
| `mst_skill` | Master kỹ năng dùng chung (danh mục skill chuẩn). | — | — |
| `mst_expense_category` | Master hạng mục chi phí dùng chung (phục vụ thanh toán hộ/đi lại). | — | — |
| `notification_history` | Lịch sử gửi thông báo đa kênh tới tenant (định tuyến thông báo toàn hệ thống). | — | — |
| `t_usage_billing` | Hóa đơn sử dụng dịch vụ SaaS Carima theo tháng cho từng tenant (billing của chính nền tảng). | `tenant_registry` | — |
| `t_tenant_usage_metric` | Số liệu sử dụng tài nguyên theo tenant (log hệ thống, phục vụ tính phí và giám sát). | `tenant_registry` | — |


# SAKI Schema

Schema vật lý: `tenant_saki_<id>` — 37 bảng.


| Bảng | Chức năng | → Tham chiếu (FK ra) | ← Được tham chiếu (FK vào) |
|------|-----------|----------------------|----------------------------|
| `tmp_saki_setup_credential` | Bảng tạm lưu thông tin khởi tạo tài khoản SAKI khi onboarding (dùng một lần). | — | — |
| `mst_saki_company` | Thông tin công ty SAKI (đối tác phái cử): tên, địa chỉ, mã số. | — | `cfg_saki_36_agreement_alarm`, `cfg_saki_alarm_setting`, `cfg_saki_contract_item_control`, `cfg_saki_contract_remarks_control`, `cfg_saki_holiday`, `cfg_saki_permission_setting`, `cfg_saki_specified_date`, `cfg_saki_treatment_info`, `cfg_saki_work_schedule_default`, `mst_saki_approval_group`, `mst_saki_business_partner`, `mst_saki_conflict_date_office`, `mst_saki_custom_job_type`, `mst_saki_department`, `mst_saki_office`, `mst_saki_user`, `t_dispatch_inquiry`, `t_dispatch_inquiry_draft`, `t_saki_opinion_hearing` |
| `mst_saki_office` | Văn phòng/chi nhánh của SAKI. | `mst_saki_company`, `mst_saki_conflict_date_office` | — |
| `mst_saki_department` | Phòng ban của SAKI, gắn nhóm duyệt (approval_group) và nhóm trả lời (reply_group). | `mst_saki_approval_group`, `mst_saki_company` | — |
| `mst_saki_user` | Người dùng thuộc SAKI; được gắn với công ty, văn phòng và phòng ban để phân quyền và lọc dữ liệu theo cơ cấu tổ chức. | `mst_saki_company` | `cfg_saki_alarm_specified_user`, `mst_saki_approval_group_user`, `mst_saki_user_scope` |
| `mst_saki_user_scope` | Phạm vi dữ liệu (scope) mà một người dùng SAKI được phép truy cập. | `mst_saki_user` | — |
| `mst_saki_approval_group` | Nhóm phê duyệt của SAKI (cấu hình luồng duyệt). | `mst_saki_company` | `mst_saki_approval_group_user`, `mst_saki_department` |
| `mst_saki_approval_group_user` | Thành viên trong nhóm phê duyệt của SAKI. | `mst_saki_approval_group`, `mst_saki_user` | — |
| `mst_saki_custom_job_type` | Loại công việc tùy biến riêng của SAKI (mở rộng từ master chung). | `mst_saki_company` | `mst_saki_custom_job_type_skill` |
| `mst_saki_custom_job_type_skill` | Liên kết kỹ năng cho loại công việc tùy biến của SAKI. | `mst_saki_custom_job_type` | — |
| `mst_saki_conflict_date_office` | Cấu hình ngày xung đột (抵触日) theo văn phòng SAKI. | `mst_saki_company` | `mst_saki_office` |
| `cfg_saki_work_schedule_default` | Cấu hình lịch làm việc mặc định của SAKI. | `mst_saki_company` | — |
| `cfg_saki_treatment_info` | Cấu hình thông tin đãi ngộ (待遇) của SAKI. | `mst_saki_company` | — |
| `t_saki_opinion_hearing` | Phiếu lấy ý kiến/điều trần (意見聴取) của SAKI. | `mst_saki_company` | — |
| `cfg_saki_contract_item_control` | Cấu hình kiểm soát các mục hiển thị trên hợp đồng phía SAKI. | `mst_saki_company` | — |
| `cfg_saki_contract_remarks_control` | Cấu hình kiểm soát phần ghi chú hợp đồng phía SAKI. | `mst_saki_company` | — |
| `mst_saki_business_partner` | Danh mục đối tác kinh doanh của SAKI. | `mst_saki_company` | — |
| `cfg_saki_permission_setting` | Cấu hình phân quyền chi tiết trong nội bộ SAKI. | `mst_saki_company` | — |
| `cfg_saki_alarm_setting` | Cấu hình cảnh báo (alarm) của SAKI. | `mst_saki_company` | `cfg_saki_alarm_send_condition`, `cfg_saki_alarm_specified_user` |
| `cfg_saki_alarm_send_condition` | Điều kiện gửi cảnh báo của SAKI. | `cfg_saki_alarm_setting` | — |
| `cfg_saki_alarm_specified_user` | Danh sách người nhận cảnh báo chỉ định của SAKI. | `cfg_saki_alarm_setting`, `mst_saki_user` | — |
| `cfg_saki_36_agreement_alarm` | Cấu hình cảnh báo vượt giới hạn Hiệp định 36 (36協定) phía SAKI. | `mst_saki_company` | — |
| `cfg_saki_holiday` | Lịch ngày nghỉ/lễ cấu hình riêng của SAKI. | `mst_saki_company` | `cfg_saki_specified_date` |
| `cfg_saki_specified_date` | Ngày chỉ định (đặc biệt) trong cấu hình lịch nghỉ của SAKI. | `cfg_saki_holiday`, `mst_saki_company` | — |
| `t_dispatch_inquiry_draft` | Bản nháp yêu cầu phái cử (lưu JSON, giữ ~8 ngày trước khi submit). | `mst_saki_company` | — |
| `t_dispatch_inquiry` | Yêu cầu phái cử (派遣照会) chính thức sau khi submit — broadcast tới nhiều MOTO. | `mst_saki_company` | `t_dispatch_estimate`, `t_dispatch_inquiry_approval`, `t_dispatch_inquiry_condition`, `t_dispatch_inquiry_job_type`, `t_dispatch_inquiry_lang_skill`, `t_dispatch_inquiry_office_env`, `t_dispatch_inquiry_revision_history`, `t_dispatch_inquiry_target_company`, `t_dispatch_inquiry_work_skill` |
| `t_dispatch_inquiry_condition` | Điều kiện của yêu cầu phái cử (quan hệ 1:1). | `t_dispatch_inquiry` | — |
| `t_dispatch_inquiry_job_type` | Loại công việc (職種) yêu cầu trong một yêu cầu phái cử (1:N). | `t_dispatch_inquiry` | — |
| `t_dispatch_inquiry_work_skill` | Kỹ năng nghiệp vụ (業務スキル) yêu cầu (1:N). | `t_dispatch_inquiry` | — |
| `t_dispatch_inquiry_lang_skill` | Kỹ năng ngoại ngữ/PC (語学・PC) yêu cầu (1:N). | `t_dispatch_inquiry` | — |
| `t_dispatch_inquiry_office_env` | Môi trường văn phòng của yêu cầu phái cử (1:1). | `t_dispatch_inquiry` | — |
| `t_dispatch_inquiry_target_company` | Danh sách công ty MOTO nhận yêu cầu phái cử (照会先, 1:N). | `t_dispatch_inquiry` | — |
| `t_dispatch_inquiry_approval` | Luồng phê duyệt nội bộ SAKI cho yêu cầu phái cử (1:N). | `t_dispatch_inquiry` | `t_dispatch_inquiry_revision_history` |
| `t_dispatch_estimate` | Báo giá (見積回答) do MOTO GHI CHÉO vào schema SAKI. RLS theo moto_company_id. (MOTO ghi chéo vào) | `t_dispatch_inquiry` | — |
| `t_dispatch_inquiry_revision_history` | Lịch sử chỉnh sửa yêu cầu phái cử (1:N). | `t_dispatch_inquiry`, `t_dispatch_inquiry_approval` | — |
| `t_saki_contract_upload_history` | Lịch sử upload file (TSV) xác nhận/đăng ký hợp đồng hàng loạt. | — | — |
| `t_saki_batch_execution_history` | Lịch sử job nền/export của SAKI. | — | — |


# MOTO Schema

Schema vật lý: `tenant_moto_<id>` — 60 bảng.


| Bảng | Chức năng | → Tham chiếu (FK ra) | ← Được tham chiếu (FK vào) |
|------|-----------|----------------------|----------------------------|
| `tmp_moto_setup_credential` | Bảng tạm lưu thông tin khởi tạo tài khoản MOTO khi onboarding (dùng một lần). | — | — |
| `mst_moto_company` | Thông tin công ty MOTO (công ty cung ứng nhân lực). | — | `cfg_moto_36_agreement`, `cfg_moto_36_agreement_free`, `cfg_moto_36_overtime_start`, `cfg_moto_advance_payment_screen`, `cfg_moto_alarm_36`, `cfg_moto_alarm_36_recipient`, `cfg_moto_alarm_conflict`, `cfg_moto_bank_account`, `cfg_moto_client_company`, `cfg_moto_compensatory_holiday`, `cfg_moto_contract_additional_text`, `cfg_moto_contract_default`, `cfg_moto_contract_text_per_client`, `cfg_moto_invoice_calculation`, `cfg_moto_invoice_default`, `cfg_moto_invoice_fraction`, `cfg_moto_legal_holiday`, `cfg_moto_seal`, `mst_moto_attendance_category`, `mst_moto_company_contact`, `mst_moto_contract_company`, `mst_moto_department`, `mst_moto_office`, `mst_moto_shift`, `mst_moto_user`, `mst_staff`, `t_contract`, `t_invoice`, `t_invoice_upload_history`, `t_moto_batch_execution_history`, `t_treatment_notification` |
| `mst_moto_company_contact` | Thông tin liên hệ của công ty MOTO. | `mst_moto_company` | — |
| `mst_moto_office` | Văn phòng/chi nhánh của MOTO. | `mst_moto_company` | — |
| `mst_moto_department` | Phòng ban của MOTO. | `mst_moto_company` | — |
| `mst_moto_user` | Người dùng thuộc MOTO; gắn công ty/văn phòng/phòng ban. | `mst_moto_company` | `cfg_moto_client_company_user` |
| `mst_moto_attendance_category` | Master phân loại chấm công của MOTO (đi làm, nghỉ, ca bù...). | `mst_moto_company` | — |
| `cfg_moto_client_company` | Cấu hình công ty khách hàng (SAKI) mà MOTO phục vụ. | `mst_moto_company` | `cfg_moto_client_company_user`, `cfg_moto_seal_per_client` |
| `cfg_moto_client_company_user` | Người dùng gắn với công ty khách hàng phía MOTO. | `cfg_moto_client_company`, `mst_moto_user` | — |
| `mst_moto_contract_company` | Danh mục công ty ký hợp đồng phía MOTO. | `mst_moto_company` | `cfg_moto_contract_text_per_client` |
| `cfg_moto_contract_default` | Cấu hình mặc định cho hợp đồng phía MOTO. | `mst_moto_company` | — |
| `cfg_moto_contract_additional_text` | Văn bản bổ sung mặc định trên hợp đồng MOTO. | `mst_moto_company` | — |
| `cfg_moto_contract_text_per_client` | Văn bản hợp đồng tùy biến theo từng khách hàng (SAKI). | `mst_moto_company`, `mst_moto_contract_company` | — |
| `cfg_moto_invoice_default` | Cấu hình mặc định cho hóa đơn phía MOTO. | `mst_moto_company` | — |
| `cfg_moto_invoice_calculation` | Cấu hình quy tắc tính toán hóa đơn của MOTO. | `mst_moto_company` | — |
| `cfg_moto_invoice_fraction` | Cấu hình xử lý phần lẻ (làm tròn) khi tính hóa đơn. | `mst_moto_company` | — |
| `cfg_moto_bank_account` | Tài khoản ngân hàng nhận thanh toán của MOTO. | `mst_moto_company` | — |
| `cfg_moto_seal` | Cấu hình con dấu (印鑑) dùng trên chứng từ MOTO. | `mst_moto_company` | — |
| `cfg_moto_seal_per_client` | Cấu hình con dấu tùy biến theo từng khách hàng (SAKI). | `cfg_moto_client_company` | — |
| `cfg_moto_advance_payment_screen` | Cấu hình màn hình thanh toán tạm ứng (立替) của MOTO. | `mst_moto_company` | — |
| `cfg_moto_36_agreement` | Cấu hình Hiệp định 36 (36協定) của MOTO. | `mst_moto_company` | — |
| `cfg_moto_36_agreement_free` | Cấu hình Hiệp định 36 dạng tự do (đặc biệt) của MOTO. | `mst_moto_company` | — |
| `cfg_moto_36_overtime_start` | Cấu hình mốc bắt đầu tính giờ làm thêm theo Hiệp định 36. | `mst_moto_company` | — |
| `cfg_moto_legal_holiday` | Cấu hình ngày nghỉ pháp định (法定休日) của MOTO. | `mst_moto_company` | — |
| `cfg_moto_compensatory_holiday` | Cấu hình ngày nghỉ bù (振替休日) của MOTO. | `mst_moto_company` | — |
| `cfg_moto_alarm_conflict` | Cấu hình cảnh báo xung đột ngày (抵触日) của MOTO. | `mst_moto_company` | — |
| `cfg_moto_alarm_36` | Cấu hình cảnh báo Hiệp định 36 của MOTO. | `mst_moto_company` | — |
| `cfg_moto_alarm_36_recipient` | Danh sách người nhận cảnh báo Hiệp định 36 của MOTO. | `mst_moto_company` | — |
| `mst_staff` | Hồ sơ nhân viên phái cử (staff) của MOTO — đối tượng trung tâm của chấm công/hợp đồng. | `mst_moto_company` | `mst_staff_conflict_date`, `mst_staff_credential`, `t_attendance_close`, `t_attendance_daily`, `t_contract`, `t_invoice_detail`, `t_shift_schedule`, `t_special_clause_application`, `t_staff_notification_read`, `t_working_time_monthly` |
| `mst_staff_credential` | Chứng chỉ/bằng cấp của nhân viên phái cử. | `mst_staff` | — |
| `mst_staff_conflict_date` | Ngày xung đột (không thể làm) của nhân viên; có logical FK sang SAKI (bắt buộc index). | `mst_staff` | — |
| `t_contract` | Hợp đồng phái cử (core hệ thống). PK = contract_no (không IDENTITY). MOTO sở hữu vòng đời. RLS theo saki_company_id. | `mst_moto_company`, `mst_staff`, `t_contract` (self) | `t_36_agreement_monthly`, `t_36_agreement_yearly`, `t_attendance_close`, `t_attendance_close_approval`, `t_attendance_daily`, `t_attendance_daily_approval`, `t_contract_approval`, `t_contract_moto_detail`, `t_contract_saki_detail`, `t_contract_status_history`, `t_invoice_detail`, `t_shift_schedule`, `t_special_clause_application`, `t_working_time_monthly` |
| `t_contract_moto_detail` | Chi tiết hợp đồng phía MOTO ghi: đơn giá chuẩn/làm thêm phục vụ tính hóa đơn (1:1). | `t_contract` | — |
| `t_contract_saki_detail` | Chi tiết hợp đồng do SAKI ghi chéo vào schema MOTO (1:1). | `t_contract` | — |
| `t_contract_approval` | Lịch sử phê duyệt hợp đồng phía SAKI (1:N). (SAKI ghi chéo vào) | `t_contract` | — |
| `t_contract_status_history` | Nhật ký thay đổi trạng thái hợp đồng (cùng-ghi, system viết). | `t_contract` | — |
| `t_treatment_notification` | Thông báo điều kiện đãi ngộ/ngày xung đột (待遇・抵触日) — SAKI ghi chéo. RLS theo saki_company_id. | `mst_moto_company` | `t_notification_office_conflict` |
| `t_notification_office_conflict` | Chi tiết thông báo xung đột theo văn phòng/đơn vị kinh doanh (事業所抵触日, 1:N). | `t_treatment_notification` | — |
| `t_moto_contract_upload_history` | Lịch sử upload file (TSV) xác nhận/đăng ký hợp đồng hàng loạt. | — | — |
| `t_attendance_daily` | Chấm công ngày của staff (vùng I/O nặng nhất). Thời gian lưu bằng phút từ 00:00. RLS theo saki_company_id. (Đã phi chuẩn hóa: copy sẵn cột saki_company_id từ bảng Hợp đồng sang để phục vụ RLS và tối ưu tốc độ truy vấn) | `mst_staff`, `t_contract` | `t_attendance_daily_approval` |
| `t_attendance_daily_approval` | Lịch sử SAKI duyệt chấm công ngày (SAKI ghi chéo). | `t_attendance_daily`, `t_contract` | — |
| `t_attendance_close` | Chốt chấm công cuối kỳ. RLS theo saki_company_id. | `mst_staff`, `t_contract` | `t_attendance_close_approval` |
| `t_attendance_close_approval` | Lịch sử SAKI duyệt chốt chấm công (SAKI ghi chéo). | `t_attendance_close`, `t_contract` | — |
| `t_working_time_monthly` | Tổng hợp thời gian làm việc theo tháng (batch tổng hợp từ chấm công ngày). | `mst_staff`, `t_contract` | `t_36_agreement_monthly` |
| `t_36_agreement_monthly` | Theo dõi giới hạn làm thêm theo tháng (36協定, batch). | `t_contract`, `t_working_time_monthly` | `t_36_agreement_yearly` |
| `t_36_agreement_yearly` | Theo dõi giới hạn làm thêm theo năm (36協定, batch). | `t_36_agreement_monthly`, `t_contract` | — |
| `t_special_clause_application` | Đơn áp dụng điều khoản đặc biệt vượt giới hạn 36 (SAKI申請). | `mst_staff`, `t_contract` | — |
| `t_attendance_email_history` | Lịch sử email liên quan chấm công (nhắc duyệt, cảnh báo). | — | — |
| `t_invoice` | Header hóa đơn gửi SAKI. SAKI có quyền khóa qua System Connection. RLS theo client_code (=SAKI). | `mst_moto_company`, `t_invoice_upload_history` | `t_invoice_bank_account`, `t_invoice_detail` |
| `t_invoice_bank_account` | Tài khoản nhận thanh toán in trên hóa đơn (tối đa 6). | `t_invoice` | — |
| `t_invoice_detail` | Chi tiết hóa đơn theo từng nhân viên/kỳ. | `mst_staff`, `t_contract`, `t_invoice` | `t_invoice_adjustment_detail`, `t_invoice_detail_attendance_daily`, `t_invoice_reimbursement_header` |
| `t_invoice_detail_attendance_daily` | Chi tiết từng ngày chấm công ánh xạ vào dòng hóa đơn (1:N, phục vụ in chứng từ). | `t_invoice_detail` | — |
| `t_invoice_adjustment_detail` | Các khoản điều chỉnh cộng/trừ trên hóa đơn (1:N). | `t_invoice_detail` | — |
| `t_invoice_reimbursement_header` | Header khoản thanh toán hộ/tạm ứng (立替金) trên hóa đơn (1:1 với detail). | `t_invoice_detail` | `t_invoice_reimbursement_detail` |
| `t_invoice_reimbursement_detail` | Chi tiết khoản thanh toán hộ theo hạng mục chi phí (1:N). | `t_invoice_reimbursement_header` | — |
| `t_invoice_upload_history` | Lịch sử upload file (ZIP) liên quan hóa đơn phía MOTO. | `mst_moto_company` | `t_invoice` |
| `mst_moto_shift` | Master ca làm việc do MOTO định nghĩa (giờ bắt đầu/kết thúc, nghỉ — lưu bằng phút). | `mst_moto_company` | `t_shift_schedule` |
| `t_shift_schedule` | Lịch làm việc dự kiến (シフト表) của staff. RLS theo saki_company_id. | `mst_moto_shift`, `mst_staff`, `t_contract` | — |
| `t_staff_notification_read` | Trạng thái đã đọc thông báo của Staff App (logical FK tới notification_history ở platform). | `mst_staff` | — |
| `t_moto_batch_execution_history` | Lịch sử async job của MOTO (tổng hợp tháng, Hiệp định 36, tạo hóa đơn). | `mst_moto_company` | — |
