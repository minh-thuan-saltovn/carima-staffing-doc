# PA-TEN-008-API-01-Saki tenant detail

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Saki tenant detail |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.1 |
| Ngày tạo | 2026/06/18 |
| Ngày cập nhật | 2026/06/22 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/18 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Saki tenant detail (GET) phục vụ hiển thị đầy đủ 5 tab của màn hình PA-TEN-008 dựa trên UI Figma và ERD PostgreSQL |
| v1.1 | 2026/06/22 | Thuận | Cập nhật lại tài liệu: Loại bỏ tất cả các trường ảo/trường tự định nghĩa (như role_name, office_name, status trong business partner, v.v.), chỉ giữ lại đúng các trường vật lý có trong ERD cơ sở dữ liệu SAKI. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-008-API-01 |
| Tên API | Saki tenant detail |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-008 |
| Chức năng liên quan | Tenant Management (SAKI Tenant Detail) |
| Mục đích API | Lấy thông tin đăng ký chi tiết của một Tenant SAKI cùng với thông tin doanh nghiệp, danh sách chi nhánh, phòng ban, người dùng, nhóm phê duyệt và đối tác liên kết từ Database. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/admin/saki-tenants/{id} |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Dynamic Tenant Schema |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này được sử dụng khi Platform SaaS Admin truy cập vào trang Chi tiết Tenant SAKI (`PA-TEN-008`). API sẽ thực hiện:
1. Lấy thông tin registry của Tenant SAKI từ bảng dùng chung `platform.tenant_registry` theo `id` (ULID).
2. Dựa vào `schema_name` của Tenant, thiết lập kết nối động (Dynamic Database Connection) sang tenant schema tương ứng (`tenant_saki_<id>`).
3. Truy vấn các thông tin doanh nghiệp (`mst_saki_company`), danh sách sự nghiệp sở (`mst_saki_office`), phòng ban (`mst_saki_department`), người dùng (`mst_saki_user`), nhóm phê duyệt (`mst_saki_approval_group`, `mst_saki_approval_group_user`) và đối tác (`mst_saki_business_partner`).
4. Truy vấn các mối quan hệ liên kết với MOTO từ bảng `platform.mst_moto_saki_relation`.
5. Tổng hợp dữ liệu và trả về cho Frontend để hiển thị trên 5 tab tương ứng (概要, 事業所・部署, ユーザー, 承認グループ, 取引先).

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Quản trị viên truy cập trang chi tiết Tenant SAKI. |
| Actor | Platform SaaS Admin / Platform SaaS Staff |
| Trước khi gọi API | Tenant SAKI đã tồn tại trên hệ thống. |
| Sau khi gọi API | Trả về toàn bộ dữ liệu chi tiết của Tenant SAKI để hiển thị lên UI. |
| Thay đổi trạng thái liên quan | Không (API chỉ đọc). |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/admin/saki-tenants/{id}
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | ID của Tenant SAKI (ULID) | 01H2PJX7G3P681729X4R09Q872 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

Không áp dụng (Phương thức GET không sử dụng Request Body).

---

# 6. Response Definition

## 6.1 Success Response

### HTTP Status

```
200 OK
```

### Response Body

```json
{
  "data": {
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "tenant_type": "SAKI",
    "company_id": "SAKI-0001",
    "company_name": "株式会社サキケア",
    "schema_name": "tenant_saki_0001",
    "db_user": "db_saki_user_0001",
    "status": 1,
    "plan_code": "ENTERPRISE",
    "contracted_at": "2026-04-01",
    "created_at": "2026-04-01T10:00:00+09:00",
    "updated_at": "2026-05-10T15:30:00+09:00",
    "company_profile": {
      "company_id": "SAKI-0001",
      "official_name_ja": "株式会社サキケア",
      "official_name_kana": "カブシキガイシャサキケア",
      "display_name_ja": "サキケア",
      "postal_code": "1000001",
      "address_ja": "東京都千代田区千代田1-1",
      "address2_ja": "サキケアビル5F",
      "official_name_en": "SakiCare Co., Ltd.",
      "display_name_en": "SakiCare",
      "treatment_priority_master": 0,
      "equal_pay_method": "労使協定方式"
    },
    "offices": [
      {
        "company_id": "SAKI-0001",
        "office_id": "OFF-0001",
        "official_name_ja": "本社",
        "display_name_ja": "本社",
        "address_ja": "東京都千代田区千代田1-1",
        "official_name_en": "Head Office",
        "display_name_en": "Head Office",
        "address_en": "1-1 Chiyoda, Chiyoda-ku, Tokyo",
        "status": 1,
        "conflict_date_type": 1,
        "conflict_date": "2029-04-01",
        "conflict_date_office_id": null,
        "treatment_setting_method": 0
      },
      {
        "company_id": "SAKI-0001",
        "office_id": "OFF-0002",
        "official_name_ja": "大阪支社",
        "display_name_ja": "大阪支社",
        "address_ja": "大阪府大阪市北区梅田1-1",
        "official_name_en": "Osaka Branch",
        "display_name_en": "Osaka Branch",
        "address_en": "1-1 Umeda, Kita-ku, Osaka-shi, Osaka",
        "status": 1,
        "conflict_date_type": 1,
        "conflict_date": "2029-04-01",
        "conflict_date_office_id": null,
        "treatment_setting_method": 0
      },
      {
        "company_id": "SAKI-0001",
        "office_id": "OFF-0003",
        "official_name_ja": "名古屋支社",
        "display_name_ja": "名古屋支社",
        "address_ja": "愛知県名古屋市中村区名駅1-1",
        "official_name_en": "Nagoya Branch",
        "display_name_en": "Nagoya Branch",
        "address_en": "1-1 Meieki, Nakamura-ku, Nagoya-shi, Aichi",
        "status": 1,
        "conflict_date_type": 2,
        "conflict_date_office_id": "OFF-0001",
        "treatment_setting_method": 0
      }
    ],
    "departments": [
      {
        "company_id": "SAKI-0001",
        "department_id": "DEP-0001",
        "official_name_ja": "人事部",
        "display_name_ja": "人事部",
        "official_name_en": "HR Department",
        "display_name_en": "HR",
        "tel": "03-1111-2222",
        "sub_code": null,
        "status": 1,
        "approval_group_id": "承認G-001",
        "reply_group_id": "返答G-001",
        "org_unit_name": null,
        "org_head_title": null,
        "default_contract_person_id": null,
        "default_dispatch_supervisor_id": null,
        "default_commander_id": null,
        "default_invoice_recipient_id": null,
        "default_complaint_person_id": null
      },
      {
        "company_id": "SAKI-0001",
        "department_id": "DEP-0002",
        "official_name_ja": "経理部",
        "display_name_ja": "経理部",
        "official_name_en": "Accounting Department",
        "display_name_en": "Accounting",
        "tel": "03-2222-3333",
        "sub_code": null,
        "status": 1,
        "approval_group_id": "承認G-002",
        "reply_group_id": "返答G-002",
        "org_unit_name": null,
        "org_head_title": null,
        "default_contract_person_id": null,
        "default_dispatch_supervisor_id": null,
        "default_commander_id": null,
        "default_invoice_recipient_id": null,
        "default_complaint_person_id": null
      },
      {
        "company_id": "SAKI-0001",
        "department_id": "DEP-0003",
        "official_name_ja": "営業部",
        "display_name_ja": "営業部",
        "official_name_en": "Sales Department",
        "display_name_en": "Sales",
        "tel": "03-3333-4444",
        "sub_code": null,
        "status": 1,
        "approval_group_id": "承認G-001",
        "reply_group_id": "返答G-001",
        "org_unit_name": null,
        "org_head_title": null,
        "default_contract_person_id": null,
        "default_dispatch_supervisor_id": null,
        "default_commander_id": null,
        "default_invoice_recipient_id": null,
        "default_complaint_person_id": null
      },
      {
        "company_id": "SAKI-0001",
        "department_id": "DEP-0004",
        "official_name_ja": "製造部",
        "display_name_ja": "製造部",
        "official_name_en": "Production Department",
        "display_name_en": "Production",
        "tel": "03-4444-5555",
        "sub_code": null,
        "status": 0,
        "approval_group_id": "承認G-003",
        "reply_group_id": "返答G-003",
        "org_unit_name": null,
        "org_head_title": null,
        "default_contract_person_id": null,
        "default_dispatch_supervisor_id": null,
        "default_commander_id": null,
        "default_invoice_recipient_id": null,
        "default_complaint_person_id": null
      }
    ],
    "users": [
      {
        "user_id": "U-00001",
        "company_id": "SAKI-0001",
        "office_id": "OFF-0001",
        "department_id": "DEP-0001",
        "last_name_ja": "田中",
        "first_name_ja": "太郎",
        "last_name_kana": "タナカ",
        "first_name_kana": "タロウ",
        "last_name_en": "Tanaka",
        "middle_name_en": null,
        "first_name_en": "Taro",
        "position_ja": null,
        "position_en": null,
        "tel": "03-1234-5678",
        "fax": null,
        "email": "tanaka@sakicare.jp",
        "mail_language": "ja",
        "reference_scope": 5,
        "execution_role_id": "role_admin",
        "edit_on_approval_flg": 0,
        "view_group_company_flg": 0,
        "edit_contract_dept_flg": 0,
        "approval_group_id": "承認G-001",
        "show_as_approver_flg": 0,
        "final_approver_flg": 0,
        "select_dispatch_company_flg": 0,
        "attendance_approver1_id": null,
        "attendance_approver2_id": null,
        "attendance_approver3_id": null,
        "cost_center_code": null,
        "cost_center_comment": null,
        "status": 1
      }
    ],
    "approval_groups": [
      {
        "company_id": "SAKI-0001",
        "approval_group_id": "承認G-001",
        "group_name_ja": "承認グループ1",
        "group_name_en": "Approval Group 1",
        "status": 1
      }
    ],
    "approval_group_users": [
      {
        "company_id": "SAKI-0001",
        "approval_group_id": "承認G-001",
        "user_id": "U-00001"
      }
    ],
    "moto_relations": [
      {
        "relation_id": "REL-000001",
        "moto_tenant_id": "01H2PJX7G3P681729X4R09Q871",
        "saki_tenant_id": "01H2PJX7G3P681729X4R09Q872",
        "moto_company_id": "MOTO-0001",
        "saki_company_id": "SAKI-0001",
        "relation_status": 1,
        "established_date": "2026-04-05",
        "terminated_date": null
      }
    ],
    "business_partners": [
      {
        "company_id": "SAKI-0001",
        "partner_auto_id": 1,
        "partner_company_name": "株式会社ビジネスパートナーA",
        "partner_id": "PTN-0001"
      }
    ]
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả | Bảng nguồn |
| --- | --- | --- | --- |
| data.tenant_id | string | ID định danh duy nhất của Tenant (ULID) | platform.tenant_registry |
| data.tenant_type | string | Loại Tenant (`SAKI`) | platform.tenant_registry |
| data.company_id | string | Mã công ty (business key) của Tenant | platform.tenant_registry |
| data.company_name | string | Tên công ty đăng ký | platform.tenant_registry |
| data.schema_name | string | Tên schema PostgreSQL riêng của Tenant | platform.tenant_registry |
| data.db_user | string | PostgreSQL role riêng của Tenant | platform.tenant_registry |
| data.status | integer | Trạng thái hoạt động (0: Vô hiệu, 1: Hoạt động) | platform.tenant_registry |
| data.plan_code | string | Gói dịch vụ đăng ký | platform.tenant_registry |
| data.contracted_at | string | Ngày ký hợp đồng dịch vụ (YYYY-MM-DD) | platform.tenant_registry |
| data.created_at | string | Thời điểm tạo Tenant | platform.tenant_registry |
| data.updated_at | string | Thời điểm cập nhật Tenant gần nhất | platform.tenant_registry |
| **company_profile** | object | Thông tin hồ sơ doanh nghiệp SAKI | tenant_saki.mst_saki_company |
| company_profile.company_id | string | Mã công ty | tenant_saki.mst_saki_company |
| company_profile.official_name_ja | string | Tên chính thức công ty (tiếng Nhật) | tenant_saki.mst_saki_company |
| company_profile.official_name_kana | string | Tên chính thức công ty (Katakana) | tenant_saki.mst_saki_company |
| company_profile.display_name_ja | string | Tên hiển thị công ty (tiếng Nhật) | tenant_saki.mst_saki_company |
| company_profile.postal_code | string | Mã bưu chính (7 chữ số) | tenant_saki.mst_saki_company |
| company_profile.address_ja | string | Địa chỉ 1 | tenant_saki.mst_saki_company |
| company_profile.address2_ja | string | Địa chỉ 2 | tenant_saki.mst_saki_company |
| company_profile.official_name_en | string | Tên chính thức công ty (tiếng Anh) | tenant_saki.mst_saki_company |
| company_profile.display_name_en | string | Tên hiển thị công ty (tiếng Anh) | tenant_saki.mst_saki_company |
| company_profile.treatment_priority_master | integer | Master ưu tiên thông tin đãi ngộ (0: Công ty, 1: Sự nghiệp sở) | tenant_saki.mst_saki_company |
| company_profile.equal_pay_method | string | Phương thức cân bằng đãi ngộ phía SAKI | tenant_saki.mst_saki_company |
| **offices** | array | Danh sách chi nhánh/sự nghiệp sở trực thuộc | tenant_saki.mst_saki_office |
| offices[].company_id | string | Mã công ty | tenant_saki.mst_saki_office |
| offices[].office_id | string | Mã sự nghiệp sở (duy nhất trong công ty) | tenant_saki.mst_saki_office |
| offices[].official_name_ja | string | Tên sự nghiệp sở chính thức (tiếng Nhật) | tenant_saki.mst_saki_office |
| offices[].display_name_ja | string | Tên sự nghiệp sở hiển thị (tiếng Nhật) | tenant_saki.mst_saki_office |
| offices[].address_ja | string | Địa chỉ của sự nghiệp sở | tenant_saki.mst_saki_office |
| offices[].official_name_en | string | Tên sự nghiệp sở chính thức (tiếng Anh) | tenant_saki.mst_saki_office |
| offices[].display_name_en | string | Tên sự nghiệp sở hiển thị (tiếng Anh) | tenant_saki.mst_saki_office |
| offices[].address_en | string | Địa chỉ tiếng Anh | tenant_saki.mst_saki_office |
| offices[].status | integer | Trạng thái hoạt động (0: Vô hiệu, 1: Hiệu lực) | tenant_saki.mst_saki_office |
| offices[].conflict_date_type | integer | Phân loại cấu hình ngày xung đột | tenant_saki.mst_saki_office |
| offices[].conflict_date | string | Ngày xung đột sự nghiệp sở (nếu conflict_date_type = 1) | tenant_saki.mst_saki_office |
| offices[].conflict_date_office_id | string | Mã sự nghiệp sở áp dụng ngày xung đột (nếu conflict_date_type = 2) | tenant_saki.mst_saki_office |
| offices[].treatment_setting_method | integer | Cách cấu hình thông tin đãi ngộ | tenant_saki.mst_saki_office |
| **departments** | array | Danh sách phòng ban trực thuộc | tenant_saki.mst_saki_department |
| departments[].company_id | string | Mã công ty | tenant_saki.mst_saki_department |
| departments[].department_id | string | Mã phòng ban | tenant_saki.mst_saki_department |
| departments[].official_name_ja | string | Tên phòng ban chính thức | tenant_saki.mst_saki_department |
| departments[].display_name_ja | string | Tên phòng ban hiển thị | tenant_saki.mst_saki_department |
| departments[].official_name_en | string | Tên phòng ban chính thức (tiếng Anh) | tenant_saki.mst_saki_department |
| departments[].display_name_en | string | Tên phòng ban hiển thị (tiếng Anh) | tenant_saki.mst_saki_department |
| departments[].tel | string | Số điện thoại phòng ban | tenant_saki.mst_saki_department |
| departments[].sub_code | string | Mã phụ | tenant_saki.mst_saki_department |
| departments[].status | integer | Trạng thái (0: Vô hiệu, 1: Hiệu lực) | tenant_saki.mst_saki_department |
| departments[].approval_group_id | string | Mã nhóm nhận yêu cầu phê duyệt liên kết | tenant_saki.mst_saki_department |
| departments[].reply_group_id | string | Mã nhóm nhận trả lời | tenant_saki.mst_saki_department |
| departments[].org_unit_name | string | Tên đơn vị tổ chức | tenant_saki.mst_saki_department |
| departments[].org_head_title | string | Chức danh người đứng đầu tổ chức | tenant_saki.mst_saki_department |
| departments[].default_contract_person_id | string | Người phụ trách hợp đồng (mặc định) | tenant_saki.mst_saki_department |
| departments[].default_dispatch_supervisor_id | string | Người phụ trách SAKI (mặc định) | tenant_saki.mst_saki_department |
| departments[].default_commander_id | string | Người chỉ huy (mặc định) | tenant_saki.mst_saki_department |
| departments[].default_invoice_recipient_id | string | Nơi nhận hóa đơn (mặc định) | tenant_saki.mst_saki_department |
| departments[].default_complaint_person_id | string | Nơi tiếp nhận khiếu nại (mặc định) | tenant_saki.mst_saki_department |
| **users** | array | Danh sách người dùng của SAKI | tenant_saki.mst_saki_user |
| users[].user_id | string | ID/Mã đăng nhập người dùng | tenant_saki.mst_saki_user |
| users[].company_id | string | Mã công ty | tenant_saki.mst_saki_user |
| users[].office_id | string | Mã sự nghiệp sở trực thuộc | tenant_saki.mst_saki_user |
| users[].department_id | string | Mã phòng ban trực thuộc | tenant_saki.mst_saki_user |
| users[].last_name_ja | string | Họ (tiếng Nhật) | tenant_saki.mst_saki_user |
| users[].first_name_ja | string | Tên (tiếng Nhật) | tenant_saki.mst_saki_user |
| users[].last_name_kana | string | Họ (Katakana) | tenant_saki.mst_saki_user |
| users[].first_name_kana | string | Tên (Katakana) | tenant_saki.mst_saki_user |
| users[].last_name_en | string | Họ (tiếng Anh) | tenant_saki.mst_saki_user |
| users[].middle_name_en | string | Tên đệm (tiếng Anh) | tenant_saki.mst_saki_user |
| users[].first_name_en | string | Tên (tiếng Anh) | tenant_saki.mst_saki_user |
| users[].position_ja | string | Chức vụ (tiếng Nhật) | tenant_saki.mst_saki_user |
| users[].position_en | string | Chức vụ (tiếng Anh) | tenant_saki.mst_saki_user |
| users[].tel | string | Số điện thoại | tenant_saki.mst_saki_user |
| users[].fax | string | Số FAX | tenant_saki.mst_saki_user |
| users[].email | string | Địa chỉ email người dùng | tenant_saki.mst_saki_user |
| users[].mail_language | string | Ngôn ngữ email thông báo | tenant_saki.mst_saki_user |
| users[].reference_scope | integer | Phạm vi tham chiếu dữ liệu | tenant_saki.mst_saki_user |
| users[].execution_role_id | string | Mã quyền thực thi | tenant_saki.mst_saki_user |
| users[].edit_on_approval_flg | integer | Cờ cho sửa thông tin khi duyệt hợp đồng | tenant_saki.mst_saki_user |
| users[].view_group_company_flg | integer | Cờ tham chiếu thông tin nhóm công ty | tenant_saki.mst_saki_user |
| users[].edit_contract_dept_flg | integer | Cờ cho sửa phòng ban hiển thị trên hợp đồng | tenant_saki.mst_saki_user |
| users[].approval_group_id | string | Mã nhóm nhận yêu cầu phê duyệt | tenant_saki.mst_saki_user |
| users[].show_as_approver_flg | integer | Cờ hiển thị ứng viên nơi gửi yêu cầu duyệt | tenant_saki.mst_saki_user |
| users[].final_approver_flg | integer | Cờ duyệt cuối | tenant_saki.mst_saki_user |
| users[].select_dispatch_company_flg | integer | Cờ chọn công ty MOTO để nộp tới | tenant_saki.mst_saki_user |
| users[].attendance_approver1_id | string | Người duyệt chấm công 1 | tenant_saki.mst_saki_user |
| users[].attendance_approver2_id | string | Người duyệt chấm công 2 | tenant_saki.mst_saki_user |
| users[].attendance_approver3_id | string | Người duyệt chấm công 3 | tenant_saki.mst_saki_user |
| users[].cost_center_code | string | Mã cost center | tenant_saki.mst_saki_user |
| users[].cost_center_comment | string | Ghi chú mã cost center | tenant_saki.mst_saki_user |
| users[].status | integer | Trạng thái hoạt động (0: Vô hiệu, 1: Hiệu lực) | tenant_saki.mst_saki_user |
| **approval_groups** | array | Danh sách các nhóm phê duyệt | tenant_saki.mst_saki_approval_group |
| approval_groups[].company_id | string | Mã công ty | tenant_saki.mst_saki_approval_group |
| approval_groups[].approval_group_id | string | Mã nhóm phê duyệt | tenant_saki.mst_saki_approval_group |
| approval_groups[].group_name_ja | string | Tên nhóm phê duyệt (tiếng Nhật) | tenant_saki.mst_saki_approval_group |
| approval_groups[].group_name_en | string | Tên nhóm phê duyệt (tiếng Anh) | tenant_saki.mst_saki_approval_group |
| approval_groups[].status | integer | Trạng thái hoạt động (0: Vô hiệu, 1: Hiệu lực) | tenant_saki.mst_saki_approval_group |
| **approval_group_users** | array | Danh sách liên kết thành viên nhóm phê duyệt | tenant_saki.mst_saki_approval_group_user |
| approval_group_users[].company_id | string | Mã công ty | tenant_saki.mst_saki_approval_group_user |
| approval_group_users[].approval_group_id | string | Mã nhóm phê duyệt | tenant_saki.mst_saki_approval_group_user |
| approval_group_users[].user_id | string | Mã người dùng thành viên | tenant_saki.mst_saki_approval_group_user |
| **moto_relations** | array | Danh sách các đối tác MOTO liên kết | platform.mst_moto_saki_relation |
| moto_relations[].relation_id | string | Mã quan hệ liên kết (ULID) | platform.mst_moto_saki_relation |
| moto_relations[].moto_tenant_id | string | ID Tenant MOTO | platform.mst_moto_saki_relation |
| moto_relations[].saki_tenant_id | string | ID Tenant SAKI | platform.mst_moto_saki_relation |
| moto_relations[].moto_company_id | string | Mã công ty MOTO (business key) | platform.mst_moto_saki_relation |
| moto_relations[].saki_company_id | string | Mã công ty SAKI (business key) | platform.mst_moto_saki_relation |
| moto_relations[].relation_status | integer | Trạng thái quan hệ (0: Đã chấm dứt, 1: Đang hợp tác) | platform.mst_moto_saki_relation |
| moto_relations[].established_date | string | Ngày thiết lập quan hệ (YYYY-MM-DD) | platform.mst_moto_saki_relation |
| moto_relations[].terminated_date | string | Ngày chấm dứt quan hệ (YYYY-MM-DD) | platform.mst_moto_saki_relation |
| **business_partners** | array | Danh sách đối tác giao dịch (phục vụ nhập nguồn) | tenant_saki.mst_saki_business_partner |
| business_partners[].company_id | string | Mã công ty | tenant_saki.mst_saki_business_partner |
| business_partners[].partner_auto_id | integer | Mã tự tăng đối tác | tenant_saki.mst_saki_business_partner |
| business_partners[].partner_company_name | string | Tên công ty đối tác | tenant_saki.mst_saki_business_partner |
| business_partners[].partner_id | string | Mã định danh đối tác do nội bộ đặt | tenant_saki.mst_saki_business_partner |

---
### 6.3 Error Response - Validation Error

```
422 Unprocessable Entity
```

### 6.4 Error Response - Invalid Credentials

```
401 Unauthorized
```

### 6.5 Error Response - Account Locked

```
403 Forbidden
```

### 6.6 Error Response - Account Inactive

```
403 Forbidden
```

### 6.7 Error Response - Tenant Not Found

```
404 Not Found
```

### 6.8 Error Response - System Error

```
500 Internal Server Error
```

### 6.9 Error Response - Bad Request

```
400 Bad Request
```

### 6.10 Error Response - Method Not Allowed

```
405 Method Not Allowed
```

### 6.11 Error Response - Too Many Requests

```
429 Too Many Requests
```

### 6.12 Error Response - Service Unavailable

```
503 Service Unavailable
```

### 6.13 Error Response - Gateway Timeout

```
504 Gateway Timeout
```

Untitled


# 7. Validation Rules

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | id | required | - | required |
| 2 | id | string | - | string |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Role authorization | Cả Platform SaaS Admin và Platform SaaS Staff có quyền `platform.tenant.saki_tenant_detail.view` đều được gọi API này. |
| BR-002 | Tenant Registry filter | API chỉ lọc và trả về dữ liệu của đúng `tenant_id` được truyền vào từ bảng `platform.tenant_registry`, với điều kiện `tenant_type = 'SAKI'` (hoặc 1). |
| BR-003 | Dynamic Connection | Hệ thống chuyển kết nối DB động sang schema `tenant_saki_<company_id>` (lấy từ trường `schema_name` của registry) để query tiếp các bảng cấu trúc nội bộ của tenant SAKI. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View |
| --- | --- |
| Platform SaaS Admin | Có |
| Platform SaaS Staff | Có |
| Tenant Admin (MOTO/SAKI) | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Xác thực token JWT gửi kèm trong header Authorization.
2. Xác minh tài khoản thuộc nhóm Platform SaaS Admin hoặc Platform SaaS Staff.
3. Kiểm tra quyền "platform.tenant.saki_tenant_detail.view".
4. Nếu hợp lệ, cho phép tiếp tục xử lý.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

Không áp dụng (API chỉ đọc).

---

## 10.2 Mapping từ DB ra Response

Chi tiết cột dữ liệu ánh xạ tương ứng đã được mô tả chi tiết tại **Section 6.2**.

---

# 11. Database Transaction

Không áp dụng.

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: GET /api/v1/admin/saki-tenants/{id}.
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization) của Platform SaaS Admin/Staff (quyền "platform.tenant.saki_tenant_detail.view").
4. Controller validate Path Parameter `id` (ULID format).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity.
5. Service truy vấn thông tin Tenant Registry trong bảng `platform.tenant_registry` theo `id` và loại `tenant_type = 'SAKI'` (hoặc 1).
   - Nếu không tìm thấy: Trả về HTTP 404 Not Found.
6. Service sử dụng giá trị `schema_name` nhận được để thiết lập kết nối động tới schema `tenant_saki_<company_id>` tương ứng.
7. Service thực hiện đồng thời hoặc tuần tự các câu lệnh SELECT từ schema tenant:
   - SELECT * FROM mst_saki_company LIMIT 1;
   - SELECT * FROM mst_saki_office;
   - SELECT * FROM mst_saki_department;
   - SELECT * FROM mst_saki_user;
   - SELECT * FROM mst_saki_approval_group;
   - SELECT * FROM mst_saki_approval_group_user;
   - SELECT * FROM mst_saki_business_partner;
8. Service query platform schema để lấy danh sách liên kết MOTO:
   - SELECT relation_id, moto_tenant_id, moto_company_id, relation_status, established_date 
     FROM platform.mst_moto_saki_relation WHERE saki_tenant_id = :tenant_id;
9. Service định dạng dữ liệu qua API Resource (Resource Collection) để đóng gói response JSON.
10. Controller trả response JSON kết quả kèm HTTP 200 OK.
```

---

# 14. Notification

Không áp dụng.

---

# 15. Audit Log

Không áp dụng.

---

# 16. Xử lý file

Không áp dụng.

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token JWT hợp lệ và phân quyền Platform SaaS Admin/Staff (`platform.tenant.saki_tenant_detail.view`). |
| 2 | Dynamic Connection Security | Ràng buộc kết nối DB sang schema chỉ định phải sử dụng DB Role có phạm vi phân quyền chính xác cho tenant đó. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.3 giây |
| Lượng truy cập dự kiến | Thấp (chỉ sử dụng bởi Platform Admin) |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính `tenant_id` trên platform.tenant_registry, các khóa ngoại và khóa chính trên các bảng thuộc tenant schema. |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Giao diện liên quan | PA-TEN-008 派遣先テナント詳細 (Hiển thị đầy đủ thông tin 5 tab) |
| ERD | Carima-Staffing Data Dictionary / ERD |
| API List | API List 341f02c407dd80c99339fa65d175b7e1.csv |

---

# 20. Open Issues

Không có.

---

# 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Done |
| 2 | Request body đã được định nghĩa | Done (Không áp dụng) |
| 3 | Response body đã được định nghĩa | Done |
| 4 | Error response đã được định nghĩa | Done |
| 5 | Validation rule đã được định nghĩa | Done |
| 6 | Business rule đã được định nghĩa | Done |
| 7 | Permission check đã được định nghĩa | Done |
| 8 | Tenant isolation rule đã được định nghĩa | Done |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done (Không áp dụng) |
| 11 | Status transition đã được định nghĩa | Done (Không áp dụng) |
| 12 | Audit log đã được định nghĩa | Done (Không áp dụng) |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
