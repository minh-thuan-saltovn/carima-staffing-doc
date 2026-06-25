# PA-TEN-002-API-01-Moto tenant detail

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Moto tenant detail |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.1 |
| Ngày tạo | 2026/06/18 |
| Ngày cập nhật | 2026/06/18 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/18 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Moto tenant detail theo yêu cầu Notion và ERD PostgreSQL |
| v1.1 | 2026/06/18 | Thuận | Cập nhật lại toàn bộ tài liệu: Loại bỏ hoàn toàn các trường thông tin thống kê, hồ sơ doanh nghiệp từ schema riêng, cấu hình tên miền và thương hiệu giả định từ màn hình thiết kế bị sai lệch. Đặc tả API chỉ truy vấn và trả về dữ liệu duy nhất từ bảng platform.tenant_registry. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-002-API-01 |
| Tên API | Moto tenant detail |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-002 |
| Chức năng liên quan | Tenant Management (MOTO Tenant Detail) |
| Mục đích API | Lấy thông tin đăng ký chi tiết của một Tenant MOTO cùng với thông tin doanh nghiệp, người phụ trách e-staffing, danh sách chi nhánh, phòng ban, người dùng và đối tác liên kết từ Database. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/admin/moto-tenants/{id} |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Dynamic Tenant Schema |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để lấy chi tiết thông tin của một Tenant MOTO (派遣元テナント) để hiển thị lên màn hình Chi tiết Tenant MOTO (`PA-TEN-002`). API sẽ thực hiện:
1. Lấy thông tin registry của Tenant MOTO từ bảng dùng chung `platform.tenant_registry` theo `id` (ULID).
2. Dựa vào `schema_name` của Tenant, thiết lập kết nối động (Dynamic Database Connection) sang tenant schema tương ứng (`tenant_moto_<id>`).
3. Truy vấn các thông tin doanh nghiệp (`mst_moto_company`), người phụ trách e-staffing (`mst_moto_company_contact`), danh sách sự nghiệp sở (`mst_moto_office`), phòng ban (`mst_moto_department`), người dùng (`mst_moto_user`) và cấu hình công ty khách hàng (`cfg_moto_client_company`).
4. Truy vấn các mối quan hệ liên kết với SAKI từ bảng `platform.mst_moto_saki_relation`.
5. Tổng hợp dữ liệu và trả về cho Frontend để hiển thị trên các tab của màn hình chi tiết.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Quản trị viên truy cập trang chi tiết Tenant MOTO. |
| Actor | Platform SaaS Admin / Platform SaaS Staff |
| Trước khi gọi API | Tenant MOTO đã tồn tại trên hệ thống. |
| Sau khi gọi API | Trả về đầy đủ dữ liệu chi tiết của Tenant MOTO để hiển thị. |
| Thay đổi trạng thái liên quan | Không (API chỉ đọc). |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/admin/moto-tenants/{id}
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
| id | string | Có | ID của Tenant MOTO (ULID) | 01H2PJX7G3P681729X4R09Q871 |

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
    "tenant_id": "01H2PJX7G3P681729X4R09Q871",
    "tenant_type": "MOTO",
    "company_id": "MOTO-0001",
    "company_name": "MOTO Staffing Solutions",
    "schema_name": "tenant_moto_0001",
    "db_user": "db_moto_user_0001",
    "status": 1,
    "plan_code": "ENTERPRISE",
    "contracted_at": "2026-04-02",
    "created_at": "2026-04-02T10:00:00+09:00",
    "updated_at": "2026-05-10T15:30:00+09:00",
    "company_profile": {
      "company_id": "MOTO-0001",
      "official_name_ja": "MOTO Staffing Solutions 株式会社",
      "official_name_kana": "モトスタッフィングソリューションズ",
      "display_name_ja": "MOTO Staffing",
      "postal_code": "1000005",
      "address_ja": "東京都千代田区丸の内1-1",
      "address2_ja": "MOTOビル3F",
      "official_name_en": "MOTO Staffing Solutions Co., Ltd.",
      "display_name_en": "MOTO Staffing",
      "dispatch_permit_first": "派",
      "dispatch_permit_last": "13-123456",
      "manage_permit_by_office_flg": 0,
      "qualified_invoice_status": 1,
      "qualified_invoice_no": "T1234567890123"
    },
    "contacts": [
      {
        "company_id": "MOTO-0001",
        "seq_no": 1,
        "department": "営業部",
        "name_ja": "山田 Taro",
        "name_kana": "ヤマダ タロウ",
        "tel": "03-1234-5678",
        "email": "yamada@moto-staffing.jp",
        "remarks": "Phụ trách chính"
      },
      {
        "company_id": "MOTO-0001",
        "seq_no": 2,
        "department": "管理部",
        "name_ja": "佐藤 花子",
        "name_kana": "サトウ ハナコ",
        "tel": "03-2345-6789",
        "email": "sato@moto-staffing.jp",
        "remarks": ""
      }
    ],
    "offices": [
      {
        "company_id": "MOTO-0001",
        "office_id": "OFF-M001",
        "official_name_ja": "本社",
        "display_name_ja": "本社",
        "address_ja": "東京都千代田区丸の内1-1",
        "official_name_en": "Head Office",
        "display_name_en": "Head Office",
        "address_en": "1-1 Marunouchi, Chiyoda-ku, Tokyo",
        "dispatch_permit_first": null,
        "dispatch_permit_last": null,
        "status": 1
      },
      {
        "company_id": "MOTO-0001",
        "office_id": "OFF-M002",
        "official_name_ja": "大阪支社",
        "display_name_ja": "大阪支社",
        "address_ja": "大阪府大阪市北区梅田2-2",
        "official_name_en": "Osaka Branch",
        "display_name_en": "Osaka Branch",
        "address_en": "2-2 Umeda, Kita-ku, Osaka-shi, Osaka",
        "dispatch_permit_first": null,
        "dispatch_permit_last": null,
        "status": 1
      },
      {
        "company_id": "MOTO-0001",
        "office_id": "OFF-M003",
        "official_name_ja": "福岡支社",
        "display_name_ja": "福岡支社",
        "address_ja": "福岡県福岡市博多区博多駅前3-3",
        "official_name_en": "Fukuoka Branch",
        "display_name_en": "Fukuoka Branch",
        "address_en": "3-3 Hakataekimae, Hakata-ku, Fukuoka-shi, Fukuoka",
        "dispatch_permit_first": null,
        "dispatch_permit_last": null,
        "status": 1
      }
    ],
    "departments": [
      {
        "company_id": "MOTO-0001",
        "department_id": "DEP-M001",
        "official_name_ja": "営業部",
        "display_name_ja": "営業部",
        "official_name_en": "Sales Department",
        "display_name_en": "Sales",
        "tel": "03-1111-2222",
        "status": 1
      },
      {
        "company_id": "MOTO-0001",
        "department_id": "DEP-M002",
        "official_name_ja": "管理部",
        "display_name_ja": "管理部",
        "official_name_en": "Administration Department",
        "display_name_en": "Admin",
        "tel": "03-2222-3333",
        "status": 1
      },
      {
        "company_id": "MOTO-0001",
        "department_id": "DEP-M003",
        "official_name_ja": "人材開発部",
        "display_name_ja": "人材開発部",
        "official_name_en": "Human Resources Department",
        "display_name_en": "HR",
        "tel": "03-3333-4444",
        "status": 1
      }
    ],
    "users": [
      {
        "user_id": "U-M001",
        "company_id": "MOTO-0001",
        "last_name_ja": "田中",
        "first_name_ja": "一郎",
        "last_name_kana": "タナカ",
        "first_name_kana": "イチロウ",
        "last_name_en": "Tanaka",
        "middle_name_en": null,
        "first_name_en": "Ichiro",
        "position_ja": "管理者",
        "position_en": "Admin",
        "tel": "03-1234-5678",
        "fax": null,
        "email": "tanaka@moto-staffing.jp",
        "mail_language": "ja",
        "reference_scope": 5,
        "execution_role_id": "role_admin",
        "edit_on_approval_flg": 0,
        "view_group_company_flg": 0,
        "edit_contract_dept_flg": 0,
        "approval_group_id": null,
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
    "saki_relations": [
      {
        "relation_id": "REL-M001",
        "moto_tenant_id": "01H2PJX7G3P681729X4R09Q871",
        "saki_tenant_id": "01H2PJX7G3P681729X4R09Q872",
        "moto_company_id": "MOTO-0001",
        "saki_company_id": "SAKI-0001",
        "relation_status": 1,
        "established_date": "2026-04-05",
        "terminated_date": null
      }
    ],
    "client_settings": [
      {
        "company_id": "MOTO-0001",
        "client_code": "CLI-001",
        "display_name_ja": "サキケア",
        "receive_email": "receive@sakicare.jp",
        "test_mail_send_flg": 0,
        "remarks": null
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
| data.tenant_type | string | Loại Tenant (`MOTO`) | platform.tenant_registry |
| data.company_id | string | Mã công ty (business key) của Tenant | platform.tenant_registry |
| data.company_name | string | Tên công ty đăng ký | platform.tenant_registry |
| data.schema_name | string | Tên schema PostgreSQL riêng của Tenant | platform.tenant_registry |
| data.db_user | string | PostgreSQL role riêng của Tenant | platform.tenant_registry |
| data.status | integer | Trạng thái hoạt động (0: Vô hiệu, 1: Hoạt động) | platform.tenant_registry |
| data.plan_code | string | Gói dịch vụ đăng ký | platform.tenant_registry |
| data.contracted_at | string | Ngày ký hợp đồng dịch vụ (YYYY-MM-DD) | platform.tenant_registry |
| data.created_at | string | Thời điểm tạo Tenant | platform.tenant_registry |
| data.updated_at | string | Thời điểm cập nhật Tenant gần nhất | platform.tenant_registry |
| **company_profile** | object | Thông tin doanh nghiệp của MOTO | tenant_moto.mst_moto_company |
| company_profile.company_id | string | Mã công ty | tenant_moto.mst_moto_company |
| company_profile.official_name_ja | string | Tên chính thức công ty (tiếng Nhật) | tenant_moto.mst_moto_company |
| company_profile.official_name_kana | string | Tên chính thức công ty (Katakana) | tenant_moto.mst_moto_company |
| company_profile.display_name_ja | string | Tên hiển thị công ty (tiếng Nhật) | tenant_moto.mst_moto_company |
| company_profile.postal_code | string | Mã bưu chính (7 chữ số) | tenant_moto.mst_moto_company |
| company_profile.address_ja | string | Địa chỉ 1 | tenant_moto.mst_moto_company |
| company_profile.address2_ja | string | Địa chỉ 2 | tenant_moto.mst_moto_company |
| company_profile.official_name_en | string | Tên chính thức công ty (tiếng Anh) | tenant_moto.mst_moto_company |
| company_profile.display_name_en | string | Tên hiển thị công ty (tiếng Anh) | tenant_moto.mst_moto_company |
| company_profile.dispatch_permit_first | string | Số giấy phép phái cử (phần đầu) | tenant_moto.mst_moto_company |
| company_profile.dispatch_permit_last | string | Số giấy phép phái cử (phần sau) | tenant_moto.mst_moto_company |
| company_profile.manage_permit_by_office_flg | integer | Cờ quản lý số giấy phép theo sự nghiệp sở | tenant_moto.mst_moto_company |
| company_profile.qualified_invoice_status | integer | Trạng thái đăng ký hóa đơn hợp lệ (0: Không, 1: Có) | tenant_moto.mst_moto_company |
| company_profile.qualified_invoice_no | string | Số đăng ký phát hành hóa đơn hợp lệ | tenant_moto.mst_moto_company |
| **contacts** | array | Danh sách người phụ trách e-staffing | tenant_moto.mst_moto_company_contact |
| contacts[].company_id | string | Mã công ty | tenant_moto.mst_moto_company_contact |
| contacts[].seq_no | integer | Số thứ tự liên hệ (1 hoặc 2) | tenant_moto.mst_moto_company_contact |
| contacts[].department | string | Đơn vị trực thuộc | tenant_moto.mst_moto_company_contact |
| contacts[].name_ja | string | Họ tên (tiếng Nhật) | tenant_moto.mst_moto_company_contact |
| contacts[].name_kana | string | Họ tên (Katakana) | tenant_moto.mst_moto_company_contact |
| contacts[].tel | string | Số điện thoại liên hệ | tenant_moto.mst_moto_company_contact |
| contacts[].email | string | Email người liên hệ | tenant_moto.mst_moto_company_contact |
| contacts[].remarks | string | Ghi chú | tenant_moto.mst_moto_company_contact |
| **offices** | array | Danh sách chi nhánh/sự nghiệp sở trực thuộc | tenant_moto.mst_moto_office |
| offices[].company_id | string | Mã công ty | tenant_moto.mst_moto_office |
| offices[].office_id | string | Mã sự nghiệp sở | tenant_moto.mst_moto_office |
| offices[].official_name_ja | string | Tên sự nghiệp sở chính thức (tiếng Nhật) | tenant_moto.mst_moto_office |
| offices[].display_name_ja | string | Tên sự nghiệp sở hiển thị (tiếng Nhật) | tenant_moto.mst_moto_office |
| offices[].address_ja | string | Địa chỉ tiếng Nhật | tenant_moto.mst_moto_office |
| offices[].official_name_en | string | Tên sự nghiệp sở chính thức (tiếng Anh) | tenant_moto.mst_moto_office |
| offices[].display_name_en | string | Tên sự nghiệp sở hiển thị (tiếng Anh) | tenant_moto.mst_moto_office |
| offices[].address_en | string | Địa chỉ tiếng Anh | tenant_moto.mst_moto_office |
| offices[].dispatch_permit_first | string | Số giấy phép phái cử (phần đầu) của sự nghiệp sở | tenant_moto.mst_moto_office |
| offices[].dispatch_permit_last | string | Số giấy phép phái cử (phần sau) của sự nghiệp sở | tenant_moto.mst_moto_office |
| offices[].status | integer | Trạng thái hoạt động (0: Vô hiệu, 1: Hiệu lực) | tenant_moto.mst_moto_office |
| **departments** | array | Danh sách phòng ban trực thuộc | tenant_moto.mst_moto_department |
| departments[].company_id | string | Mã công ty | tenant_moto.mst_moto_department |
| departments[].department_id | string | Mã phòng ban | tenant_moto.mst_moto_department |
| departments[].official_name_ja | string | Tên phòng ban chính thức (tiếng Nhật) | tenant_moto.mst_moto_department |
| departments[].display_name_ja | string | Tên phòng ban hiển thị (tiếng Nhật) | tenant_moto.mst_moto_department |
| departments[].official_name_en | string | Tên phòng ban chính thức (tiếng Anh) | tenant_moto.mst_moto_department |
| departments[].display_name_en | string | Tên phòng ban hiển thị (tiếng Anh) | tenant_moto.mst_moto_department |
| departments[].tel | string | Số điện thoại phòng ban | tenant_moto.mst_moto_department |
| departments[].status | integer | Trạng thái (0: Vô hiệu, 1: Hiệu lực) | tenant_moto.mst_moto_department |
| **users** | array | Danh sách người dùng của MOTO | tenant_moto.mst_moto_user |
| users[].user_id | string | ID người dùng | tenant_moto.mst_moto_user |
| users[].company_id | string | Mã công ty | tenant_moto.mst_moto_user |
| users[].last_name_ja | string | Họ (tiếng Nhật) | tenant_moto.mst_moto_user |
| users[].first_name_ja | string | Tên (tiếng Nhật) | tenant_moto.mst_moto_user |
| users[].last_name_kana | string | Họ (Katakana) | tenant_moto.mst_moto_user |
| users[].first_name_kana | string | Tên (Katakana) | tenant_moto.mst_moto_user |
| users[].last_name_en | string | Họ (tiếng Anh) | tenant_moto.mst_moto_user |
| users[].middle_name_en | string | Tên đệm (tiếng Anh) | tenant_moto.mst_moto_user |
| users[].first_name_en | string | Tên (tiếng Anh) | tenant_moto.mst_moto_user |
| users[].position_ja | string | Chức vụ (tiếng Nhật) | tenant_moto.mst_moto_user |
| users[].position_en | string | Chức vụ (tiếng Anh) | tenant_moto.mst_moto_user |
| users[].tel | string | Số điện thoại | tenant_moto.mst_moto_user |
| users[].fax | string | Số fax | tenant_moto.mst_moto_user |
| users[].email | string | Email người dùng | tenant_moto.mst_moto_user |
| users[].mail_language | string | Ngôn ngữ nhận email | tenant_moto.mst_moto_user |
| users[].reference_scope | integer | Phạm vi tham chiếu dữ liệu | tenant_moto.mst_moto_user |
| users[].execution_role_id | string | Mã quyền thực thi | tenant_moto.mst_moto_user |
| users[].edit_on_approval_flg | integer | Cờ cho sửa thông tin khi duyệt hợp đồng | tenant_moto.mst_moto_user |
| users[].view_group_company_flg | integer | Cờ tham chiếu thông tin nhóm công ty | tenant_moto.mst_moto_user |
| users[].edit_contract_dept_flg | integer | Cờ cho sửa phòng ban trên hợp đồng | tenant_moto.mst_moto_user |
| users[].approval_group_id | string | Mã nhóm nhận yêu cầu phê duyệt | tenant_moto.mst_moto_user |
| users[].show_as_approver_flg | integer | Cờ hiển thị ứng viên nơi gửi yêu cầu duyệt | tenant_moto.mst_moto_user |
| users[].final_approver_flg | integer | Cờ duyệt cuối | tenant_moto.mst_moto_user |
| users[].select_dispatch_company_flg | integer | Cờ chọn công ty MOTO để nộp tới | tenant_moto.mst_moto_user |
| users[].attendance_approver1_id | string | Người duyệt chấm công 1 | tenant_moto.mst_moto_user |
| users[].attendance_approver2_id | string | Người duyệt chấm công 2 | tenant_moto.mst_moto_user |
| users[].attendance_approver3_id | string | Người duyệt chấm công 3 | tenant_moto.mst_moto_user |
| users[].cost_center_code | string | Mã cost center | tenant_moto.mst_moto_user |
| users[].cost_center_comment | string | Ghi chú mã cost center | tenant_moto.mst_moto_user |
| users[].status | integer | Trạng thái hoạt động (0: Vô hiệu, 1: Hiệu lực) | tenant_moto.mst_moto_user |
| **saki_relations** | array | Danh sách đối tác liên kết SAKI | platform.mst_moto_saki_relation |
| saki_relations[].relation_id | string | Mã quan hệ liên kết (ULID) | platform.mst_moto_saki_relation |
| saki_relations[].moto_tenant_id | string | ID Tenant MOTO | platform.mst_moto_saki_relation |
| saki_relations[].saki_tenant_id | string | ID Tenant SAKI | platform.mst_moto_saki_relation |
| saki_relations[].moto_company_id | string | Mã công ty MOTO | platform.mst_moto_saki_relation |
| saki_relations[].saki_company_id | string | Mã công ty SAKI | platform.mst_moto_saki_relation |
| saki_relations[].relation_status | integer | Trạng thái quan hệ (0: Đã chấm dứt, 1: Đang hợp tác) | platform.mst_moto_saki_relation |
| saki_relations[].established_date | string | Ngày thiết lập quan hệ (YYYY-MM-DD) | platform.mst_moto_saki_relation |
| saki_relations[].terminated_date | string | Ngày chấm dứt quan hệ (YYYY-MM-DD) | platform.mst_moto_saki_relation |
| **client_settings** | array | Danh sách đối tác giao dịch (Client) | tenant_moto.cfg_moto_client_company |
| client_settings[].company_id | string | Mã công ty | tenant_moto.cfg_moto_client_company |
| client_settings[].client_code | string | Mã client đối tác | tenant_moto.cfg_moto_client_company |
| client_settings[].display_name_ja | string | Tên hiển thị công ty đối tác | tenant_moto.cfg_moto_client_company |
| client_settings[].receive_email | string | Email nhận hóa đơn/thông báo | tenant_moto.cfg_moto_client_company |
| client_settings[].test_mail_send_flg | integer | Cờ gửi email thử | tenant_moto.cfg_moto_client_company |
| client_settings[].remarks | string | Ghi chú | tenant_moto.cfg_moto_client_company |

---

## 6.3 Error Response

### Unauthorized

```
401 Unauthorized
```

### Forbidden

```
403 Forbidden
```

### Not Found

```
404 Not Found
```

### System Error

```
500 Internal Server Error
```

---


# 7. Validation Rules

Message list 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | id | required | - | required |
| 2 | id | size | 26 | size |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Role authorization | Cả Platform SaaS Admin và Platform SaaS Staff có quyền `tenant.view` đều được gọi API này. |
| BR-002 | Tenant Registry filter | API chỉ lọc và trả về dữ liệu của đúng `tenant_id` được truyền vào từ bảng `platform.tenant_registry`, với điều kiện `tenant_type = 'MOTO'` (hoặc 2). |
| BR-003 | Dynamic Connection | Hệ thống tự chuyển kết nối DB sang schema tương ứng dựa trên `schema_name` của `tenant_registry` để lấy tiếp dữ liệu của MOTO. |

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
3. Kiểm tra quyền "tenant.view".
4. Nếu hợp lệ, cho phép tiếp tục xử lý.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

Không áp dụng (API chỉ đọc).

---

## 10.2 Mapping từ DB ra Response

Chi tiết cột dữ liệu ánh xạ tương ứng đã được mô tả tại **Section 6.2**.

---

# 11. Database Transaction

Không áp dụng.

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: GET /api/v1/admin/moto-tenants/{id}.
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization) của Platform SaaS Admin/Staff (quyền "tenant.view").
4. Controller validate Path Parameter `id` (ULID format).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity.
5. Service truy vấn thông tin Tenant Registry trong bảng `platform.tenant_registry` theo `id` và loại `tenant_type = 'MOTO'` (hoặc 2).
   - Nếu không tìm thấy: Trả về HTTP 404 Not Found.
6. Service sử dụng giá trị `schema_name` nhận được để thiết lập kết nối động tới schema `tenant_moto_<company_id>` tương ứng.
7. Service thực hiện đồng thời hoặc tuần tự các câu lệnh SELECT từ schema tenant:
   - SELECT * FROM mst_moto_company LIMIT 1;
   - SELECT * FROM mst_moto_company_contact;
   - SELECT * FROM mst_moto_office;
   - SELECT * FROM mst_moto_department;
   - SELECT * FROM mst_moto_user;
   - SELECT * FROM cfg_moto_client_company;
8. Service query platform schema để lấy danh sách liên kết SAKI:
   - SELECT relation_id, saki_tenant_id, saki_company_id, relation_status, established_date 
     FROM platform.mst_moto_saki_relation WHERE moto_tenant_id = :tenant_id;
   - (Join sang platform.tenant_registry theo saki_tenant_id để lấy company_name của SAKI).
9. Service định dạng dữ liệu qua API Resource.
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
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token JWT hợp lệ và phân quyền Platform SaaS Admin/Staff (`tenant.view`). |
| 2 | Dynamic Connection Security | Ràng buộc kết nối DB sang schema chỉ định phải sử dụng DB Role có phạm vi phân quyền chính xác cho tenant đó. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.2 giây |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính `tenant_id` trên bảng `platform.tenant_registry` |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Giao diện liên quan | PA-TEN-002 派遣元テナント詳細 (Hiển thị đầy đủ thông tin 4 tab) |
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
