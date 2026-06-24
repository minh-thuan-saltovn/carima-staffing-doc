# MO-COM-000-API-01-Get Tenant Init Setup

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design |
| API ID | MO-COM-000-API-01 |
| Tên API | Get Tenant Init Setup |
| Tên tiếng Nhật | テナント初期設定情報取得 |
| Diagram / Flow áp dụng | MOTO Initial Setup Flow |
| Portal áp dụng | MOTO Portal |
| Screen ID liên quan | MO-INIT-001 |
| Màn hình liên quan | 派遣元 初期設定ウィザード |
| Version | v0.1 |
| Ngày tạo | 2026/06/18 |
| Ngày cập nhật | 2026/06/18 |
| Người tạo | Duy Pham |
| Người review | TBD |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v0.1 | 2026/06/18 | Duy Pham | Tạo bản đầu tiên cho API Get Tenant Init Setup |

---

## 3. Tổng quan API

### 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | MO-COM-000-API-01 |
| Tên API | Get Tenant Init Setup |
| Tên tiếng Nhật | テナント初期設定情報取得 |
| Business Flow liên quan | MOTO Initial Setup Flow |
| Screen ID liên quan | MO-INIT-001 |
| Chức năng liên quan | Lấy thông tin trạng thái thiết lập ban đầu của tenant MOTO |
| Mục đích API | Lấy trạng thái setup, current step, danh sách step, dữ liệu đã nhập ở từng step để hiển thị màn hình Initial Setup Wizard |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | `/api/v1/moto/init-setup` |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

### 3.2 Mô tả API

### Mục đích

API này dùng để lấy thông tin thiết lập ban đầu của tenant MOTO sau khi user đăng nhập vào MOTO Portal.

API trả về:

- Trạng thái tenant đã hoàn tất initial setup hay chưa
- Step hiện tại cần hiển thị trên wizard
- Danh sách 7 step của flow setup
- Trạng thái hoàn thành của từng step
- Dữ liệu master/config đã đăng ký trước đó
- Dữ liệu phục vụ màn hình confirm ở step 7

API này chỉ đọc dữ liệu, không cập nhật database.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User đăng nhập vào MOTO Portal hoặc mở màn hình Initial Setup |
| Actor | MOTO Tenant Admin |
| Trước khi gọi API | User đã login thành công và hệ thống xác định được tenant |
| Sau khi gọi API | Frontend hiển thị đúng step setup hiện tại |
| Thay đổi trạng thái liên quan | Không thay đổi trạng thái |
| Ghi DB | Không |
| Đọc DB | Có |

---

## 4. Định nghĩa Endpoint

### 4.1 Endpoint

```
GET /api/v1/moto/init-setup
```

---

### 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| X-Tenant-ID | Có | 01HX0000000000000000000001 | Mã tenant hiện tại |
| Accept | Không | application/json | Response format |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

### 4.3 Path Parameters

Không có.

---

### 4.4 Query Parameters

| Parameter | Type | Bắt buộc | Default | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| include_data | boolean | Không | true | Có lấy dữ liệu đã nhập của từng step hay không | true |
| include_confirmation | boolean | Không | false | Có lấy dữ liệu confirm step 7 hay không | true |

---

## 5. Request Body

### 5.1 Request Body Schema

API này sử dụng method GET nên không có request body.

```json
{}
```

---

### 5.2 Định nghĩa field request

Không có.

---

## 6. Response Definition

### 6.1 Success Response

### HTTP Status

```
200 OK
```

### Response Body

```json
{
  "data": {
    "tenant": {
      "tenant_id": "01HX0000000000000000000001",
      "tenant_type": "MOTO",
      "company_id": "MOTO001",
      "company_name": "株式会社XYZ人材サービス",
      "schema_name": "tenant_moto_001",
      "status": 1
    },
    "setup": {
      "setup_completed": false,
      "setup_status": "in_progress",
      "current_step": 3,
      "completed_steps": [1, 2],
      "next_step": 3,
      "can_start_use": false
    },
    "steps": [
      {
        "step_no": 1,
        "step_key": "company",
        "step_name": "企業情報",
        "status": "completed",
        "required": true,
        "skippable": false
      },
      {
        "step_no": 2,
        "step_key": "usage_setting",
        "step_name": "利用情報設定",
        "status": "completed",
        "required": true,
        "skippable": false
      },
      {
        "step_no": 3,
        "step_key": "office",
        "step_name": "事業所設定",
        "status": "in_progress",
        "required": true,
        "skippable": true
      },
      {
        "step_no": 4,
        "step_key": "department",
        "step_name": "部署情報",
        "status": "not_started",
        "required": true,
        "skippable": true
      },
      {
        "step_no": 5,
        "step_key": "user",
        "step_name": "ユーザー登録",
        "status": "not_started",
        "required": true,
        "skippable": true
      },
      {
        "step_no": 6,
        "step_key": "approval_route",
        "step_name": "承認ルート設定",
        "status": "not_started",
        "required": false,
        "skippable": true
      },
      {
        "step_no": 7,
        "step_key": "start_use",
        "step_name": "利用開始",
        "status": "not_started",
        "required": true,
        "skippable": false
      }
    ],
    "data": {
      "company": {
        "company_id": "MOTO001",
        "official_name_ja": "株式会社XYZ人材サービス",
        "official_name_kana": "カブシキガイシャエックスワイゼットジンザイサービス",
        "display_name_ja": "XYZ人材サービス",
        "postal_code": "1500001",
        "address_ja": "東京都渋谷区渋谷1-1-1",
        "address2_ja": "XYZビル5階",
        "official_name_en": "XYZ Staffing Services Inc.",
        "display_name_en": "XYZ Staffing",
        "dispatch_permit_first": "派13",
        "dispatch_permit_last": "000000",
        "manage_permit_by_office_flg": 0,
        "qualified_invoice_status": 1,
        "qualified_invoice_no": "1234567890123"
      },
      "contacts": [
        {
          "seq_no": 1,
          "department": "人事部",
          "name_ja": "山田 太郎",
          "name_kana": "ヤマダ タロウ",
          "tel": "03-1234-5678",
          "email": "yamada@example.com",
          "remarks": "メイン担当者"
        },
        {
          "seq_no": 2,
          "department": "営業部",
          "name_ja": "佐藤 花子",
          "name_kana": "サトウ ハナコ",
          "tel": "03-1234-5679",
          "email": "sato@example.com",
          "remarks": null
        }
      ],
      "offices": [
        {
          "office_id": "BRANCH001",
          "official_name_ja": "株式会社XYZ人材サービス 大阪支店",
          "display_name_ja": "XYZ大阪支店",
          "address_ja": "大阪府大阪市北区梅田1-1-1",
          "official_name_en": "XYZ Staffing Services Osaka Branch",
          "display_name_en": "XYZ Osaka",
          "address_en": "1-1-1 Umeda, Kita-ku, Osaka-shi, Osaka",
          "dispatch_permit_first": "派27",
          "dispatch_permit_last": "000000",
          "status": 1
        }
      ],
      "departments": [
        {
          "department_id": "DEPT001",
          "official_name_ja": "派遣事業部",
          "display_name_ja": "派遣事業部",
          "official_name_en": "Staffing Services Division",
          "display_name_en": "Staffing Division",
          "tel": "03-1234-5678",
          "status": 1
        }
      ],
      "users": [
        {
          "user_id": "USER001",
          "last_name_ja": "山田",
          "first_name_ja": "太郎",
          "last_name_kana": "ヤマダ",
          "first_name_kana": "タロウ",
          "last_name_en": "Yamada",
          "middle_name_en": null,
          "first_name_en": "Taro",
          "office_id": "BRANCH001",
          "department_id": "DEPT001",
          "position_ja": "部長",
          "position_en": "Manager",
          "tel": "03-1234-5678",
          "email": "yamada@example.com",
          "status": 1,
          "reference_scope": 5,
          "execution_role": 1,
          "special_contract_edit_flg": 0,
          "invoice_download_flg": 0
        }
      ],
      "usage_setting": {
        "contract_type": "individual",
        "billing_type": "moto_billing",
        "document_language": "ja"
      }
    }
  }
}
```

---

### 6.2 Unauthorized Response

### HTTP Status

```
401 Unauthorized
```

```json
{
  "data": null,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication is required."
  }
}
```

---

### 6.3 Forbidden Response

### HTTP Status

```
403 Forbidden
```

```json
{
  "data": null,
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to access tenant init setup."
  }
}
```

---

## 7. Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant | object | Thông tin tenant hiện tại |
| data.tenant.tenant_id | string | Tenant ID |
| data.tenant.tenant_type | string | Loại tenant. API này chỉ trả `MOTO` |
| data.tenant.company_id | string | Company ID của tenant |
| data.tenant.company_name | string | Tên công ty |
| data.tenant.schema_name | string | Schema tenant tương ứng |
| data.tenant.status | number | Trạng thái tenant |
| data.setup | object | Trạng thái setup |
| data.setup.setup_completed | boolean | Đã hoàn tất initial setup hay chưa |
| data.setup.setup_status | string | `not_started` / `in_progress` / `completed` |
| data.setup.current_step | number | Step hiện tại cần hiển thị |
| data.setup.completed_steps | array | Danh sách step đã hoàn thành |
| data.setup.next_step | number | Step tiếp theo |
| data.setup.can_start_use | boolean | Có được bắt đầu sử dụng hệ thống hay chưa |
| data.steps | array | Danh sách 7 step wizard |
| data.steps[].step_no | number | Số thứ tự step |
| data.steps[].step_key | string | Key xử lý frontend |
| data.steps[].step_name | string | Tên step tiếng Nhật |
| data.steps[].status | string | Trạng thái của step |
| data.steps[].required | boolean | Step có bắt buộc hay không |
| data.steps[].skippable | boolean | Step có thể skip hay không |
| data.data.company | object | Thông tin company master |
| data.data.contacts | array | Danh sách CARIMA Staffing担当者 |
| data.data.offices | array | Danh sách事業所 |
| data.data.departments | array | Danh sách部署 |
| data.data.users | array | Danh sách user |
| data.data.usage_setting | object | Thông tin 利用情報設定 |

## 8. Business Rules

| Rule ID | Nội dung |
| --- | --- |
| BR-001 | API chỉ dùng cho MOTO Portal. |
| BR-002 | Tenant type phải là `MOTO`. Nếu tenant là `SAKI` hoặc Platform thì trả lỗi 403. |
| BR-003 | API chỉ đọc dữ liệu theo tenant hiện tại, không cho phép cross tenant. |
| BR-004 | Nếu `tmp_moto_setup_credential.setup_completed_flg = 0` thì tenant chưa hoàn tất initial setup. |
| BR-005 | Nếu `tmp_moto_setup_credential.setup_completed_flg = 1` thì tenant đã hoàn tất initial setup. |
| BR-006 | Current step được xác định dựa trên dữ liệu master/config đã tồn tại. |
| BR-007 | Step 1 hoàn thành khi đã tồn tại `mst_moto_company` theo `company_id`. |
| BR-008 | Step 2 hoàn thành khi usage setting đã được lưu. |
| BR-009 | Step 3 hoàn thành khi tồn tại ít nhất 1 record active trong `mst_moto_office`. |
| BR-010 | Step 4 hoàn thành khi tồn tại ít nhất 1 record active trong `mst_moto_department`. |
| BR-011 | Step 5 hoàn thành khi tồn tại ít nhất 1 record active trong `mst_moto_user`. |
| BR-012 | Step 6 là optional, có thể `not_started`, `skipped` hoặc `completed`. |
| BR-013 | Step 7 chỉ được `available` khi các step bắt buộc đã hợp lệ. |
| BR-014 | Nếu tenant đã completed setup, API vẫn có thể trả dữ liệu để frontend redirect hoặc hiển thị trạng thái đã hoàn tất. |

---

## 9. Validation Rules

### 9.1 Header validation

| Field | Rule | Error |
| --- | --- | --- |
| Authorization | Bắt buộc có Bearer token hợp lệ | 401 Unauthorized |
| X-Tenant-ID | Bắt buộc có hoặc lấy được từ session | 400 Bad Request |
| Accept-Language | Nếu không truyền thì default `ja` | Không lỗi |

---

### 9.2 Query validation

| Parameter | Rule | Error |
| --- | --- | --- |
| include_data | Chỉ nhận boolean `true/false` | 400 Validation Error |
| include_confirmation | Chỉ nhận boolean `true/false` | 400 Validation Error |

---

### 9.3 Tenant validation

| Điều kiện | Kết quả |
| --- | --- |
| Tenant không tồn tại | 404 TENANT_NOT_FOUND |
| Tenant không active | 403 TENANT_INACTIVE |
| Tenant type khác MOTO | 403 INVALID_TENANT_TYPE |
| User không thuộc tenant | 403 FORBIDDEN |
| Schema tenant không tồn tại | 500 TENANT_SCHEMA_NOT_FOUND |

---

## 10. Database Mapping

### 10.1 Platform schema

| Table | Field | Mục đích |
| --- | --- | --- |
| `platform.tenant_registry` | `tenant_id` | Xác định tenant |
| `platform.tenant_registry` | `tenant_type` | Kiểm tra tenant là MOTO |
| `platform.tenant_registry` | `company_id` | Company ID dùng để query tenant schema |
| `platform.tenant_registry` | `company_name` | Tên công ty |
| `platform.tenant_registry` | `schema_name` | Schema vật lý của tenant |
| `platform.tenant_registry` | `status` | Trạng thái tenant |
| `platform.tenant_registry` | `plan_code` | Gói dịch vụ nếu cần hiển thị |
| `platform.tenant_registry` | `contracted_at` | Ngày ký hợp đồng SaaS nếu cần hiển thị |

---

### 10.2 MOTO tenant schema

| Table | Field | Mục đích |
| --- | --- | --- |
| `tmp_moto_setup_credential` | `company_id` | Company ID khởi tạo |
| `tmp_moto_setup_credential` | `setup_completed_flg` | Xác định đã hoàn tất setup hay chưa |
| `tmp_moto_setup_credential` | `expires_at` | Hạn hiệu lực tài khoản setup |
| `mst_moto_company` | `company_id` | Company ID |
| `mst_moto_company` | `official_name_ja` | 正式企業名 |
| `mst_moto_company` | `official_name_kana` | 正式企業名カナ |
| `mst_moto_company` | `display_name_ja` | システム表示企業名 |
| `mst_moto_company` | `postal_code` | 郵便番号 |
| `mst_moto_company` | `address_ja` | 住所1 |
| `mst_moto_company` | `address2_ja` | 住所2 |
| `mst_moto_company` | `official_name_en` | 正式企業名英字 |
| `mst_moto_company` | `display_name_en` | システム表示企業名英字 |
| `mst_moto_company` | `dispatch_permit_first` | 派遣許可番号 前半 |
| `mst_moto_company` | `dispatch_permit_last` | 派遣許可番号 後半 |
| `mst_moto_company` | `manage_permit_by_office_flg` | 事業所ごとに派遣許可番号を管理する |
| `mst_moto_company` | `qualified_invoice_status` | 適格請求書発行事業者登録状態 |
| `mst_moto_company` | `qualified_invoice_no` | 適格請求書発行事業者登録番号 |
| `mst_moto_company_contact` | `seq_no` | 担当者 thứ tự 1/2 |
| `mst_moto_company_contact` | `department` | 所属 |
| `mst_moto_company_contact` | `name_ja` | 氏名 |
| `mst_moto_company_contact` | `name_kana` | 氏名カナ |
| `mst_moto_company_contact` | `tel` | TEL |
| `mst_moto_company_contact` | `email` | E-mail |
| `mst_moto_company_contact` | `remarks` | 備考 |
| `mst_moto_office` | `office_id` | 事業所ID |
| `mst_moto_office` | `official_name_ja` | 正式事業所名 |
| `mst_moto_office` | `display_name_ja` | システム表示事業所名 |
| `mst_moto_office` | `address_ja` | 住所 |
| `mst_moto_office` | `official_name_en` | 正式事業所名（英字） |
| `mst_moto_office` | `display_name_en` | システム表示事業所名（英字） |
| `mst_moto_office` | `address_en` | 住所（英字） |
| `mst_moto_office` | `status` | Trạng thái office |
| `mst_moto_department` | `department_id` | 部署ID |
| `mst_moto_department` | `official_name_ja` | 正式部署名 |
| `mst_moto_department` | `display_name_ja` | システム表示部署名 |
| `mst_moto_department` | `official_name_en` | 正式部署名（英字） |
| `mst_moto_department` | `display_name_en` | システム表示部署名（英字） |
| `mst_moto_department` | `tel` | 部署TEL |
| `mst_moto_department` | `status` | Trạng thái department |
| `mst_moto_user` | `user_id` | ユーザーID / ログインID |
| `mst_moto_user` | `last_name_ja` | 姓 |
| `mst_moto_user` | `first_name_ja` | 名 |
| `mst_moto_user` | `last_name_kana` | セイ |
| `mst_moto_user` | `first_name_kana` | メイ |
| `mst_moto_user` | `last_name_en` | Last name English |
| `mst_moto_user` | `middle_name_en` | Middle name English |
| `mst_moto_user` | `first_name_en` | First name English |
| `mst_moto_user` | `office_id` | 所属事業所 |
| `mst_moto_user` | `department_id` | 所属部署 |
| `mst_moto_user` | `position_ja` | 役職（日本語） |
| `mst_moto_user` | `position_en` | 役職（英字） |
| `mst_moto_user` | `tel` | TEL |
| `mst_moto_user` | `email` | E-mail |
| `mst_moto_user` | `reference_scope` | 参照範囲 |
| `mst_moto_user` | `execution_role` | 実行権限 |
| `mst_moto_user` | `special_contract_edit_flg` | 特別契約修正 |
| `mst_moto_user` | `invoice_download_flg` | 請求書ダウンロード |
| `mst_moto_user` | `default_sales_user_id` | 営業担当者 default |
| `mst_moto_user` | `default_manager_user_id` | 派遣元責任者 default |
| `mst_moto_user` | `default_complaint_user_id` | 苦情申出先 default |

---

## 11. SQL / Data Fetch Logic

### 11.1 Lấy tenant registry

```sql
SELECT
  tenant_id,
  tenant_type,
  company_id,
  company_name,
  schema_name,
  status,
  plan_code,
  contracted_at
FROM platform.tenant_registry
WHERE tenant_id = :tenant_id
  AND status = 1;
```

---

### 11.2 Lấy setup credential

```sql
SELECT
  company_id,
  setup_completed_flg,
  expires_at,
  created_at,
  updated_at
FROM tenant_moto_<id>.tmp_moto_setup_credential
WHERE company_id = :company_id;
```

---

### 11.3 Lấy company master

```sql
SELECT
  company_id,
  official_name_ja,
  official_name_kana,
  display_name_ja,
  postal_code,
  address_ja,
  address2_ja,
  official_name_en,
  display_name_en,
  dispatch_permit_first,
  dispatch_permit_last,
  manage_permit_by_office_flg,
  qualified_invoice_status,
  qualified_invoice_no,
  updated_at
FROM tenant_moto_<id>.mst_moto_company
WHERE company_id = :company_id;
```

---

### 11.4 Lấy contact

```sql
SELECT
  seq_no,
  department,
  name_ja,
  name_kana,
  tel,
  email,
  remarks
FROM tenant_moto_<id>.mst_moto_company_contact
WHERE company_id = :company_id
ORDER BY seq_no ASC;
```

---

### 11.5 Lấy office / department / user

```sql
SELECT *
FROM tenant_moto_<id>.mst_moto_office
WHERE company_id = :company_id
  AND status = 1
ORDER BY office_id ASC;
```

```sql
SELECT *
FROM tenant_moto_<id>.mst_moto_department
WHERE company_id = :company_id
  AND status = 1
ORDER BY department_id ASC;
```

```sql
SELECT *
FROM tenant_moto_<id>.mst_moto_user
WHERE company_id = :company_id
  AND status = 1
ORDER BY user_id ASC;
```

---

## 12. Step Status Decision Logic

| Step | step_key | Điều kiện completed |
| --- | --- | --- |
| 1 | company | Có record trong `mst_moto_company` |
| 2 | usage_setting | Có usage setting đã lưu hoặc default config đã được tạo |
| 3 | office | Có ít nhất 1 record active trong `mst_moto_office` |
| 4 | department | Có ít nhất 1 record active trong `mst_moto_department` |
| 5 | user | Có ít nhất 1 record active trong `mst_moto_user` |
| 6 | approval_route | Có approval route hoặc user đã skip |
| 7 | start_use | Các step bắt buộc đã completed và setup chưa completed |

Logic xác định `current_step`:

```
Nếu setup_completed_flg = 1
  current_step = 7
  setup_status = completed

Ngược lại
  current_step = step nhỏ nhất chưa completed trong danh sách step bắt buộc

Nếu step 6 chưa completed nhưng skippable = true
  Không block step 7
```

---

## 13. Error Response

### 13.1 Validation Error

```
400 Bad Request
```

```json
{
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed.",
    "details": [
      {
        "field": "include_data",
        "message": "include_data must be boolean."
      }
    ]
  }
}
```

---

### 13.2 Tenant Not Found

```
404 Not Found
```

```json
{
  "data": null,
  "error": {
    "code": "TENANT_NOT_FOUND",
    "message": "Tenant was not found."
  }
}
```

---

### 13.3 Invalid Tenant Type

```
403 Forbidden
```

```json
{
  "data": null,
  "error": {
    "code": "INVALID_TENANT_TYPE",
    "message": "This API is only available for MOTO tenant."
  }
}
```

---

### 13.4 Tenant Schema Not Found

```
500 Internal Server Error
```

```json
{
  "data": null,
  "error": {
    "code": "TENANT_SCHEMA_NOT_FOUND",
    "message": "Tenant schema was not found."
  }
}
```

---

## 14. Transaction

API này là API đọc dữ liệu nên không cần transaction ghi dữ liệu.

| Xử lý | Transaction |
| --- | --- |
| Đọc `platform.tenant_registry` | Không |
| Đọc tenant schema | Không |
| Tính current step | Không |
| Trả response | Không |

Ghi chú:

- Không insert/update/delete dữ liệu.
- Không ghi audit log bắt buộc.
- Có thể ghi access log / application log nếu cần tracking.

---

## 15. Authorization

| Điều kiện | Kết quả |
| --- | --- |
| User chưa đăng nhập | 401 Unauthorized |
| Token hết hạn | 401 Unauthorized |
| User không thuộc tenant hiện tại | 403 Forbidden |
| User không có quyền xem initial setup | 403 Forbidden |
| Tenant không phải MOTO | 403 Invalid Tenant Type |

Permission đề xuất:

```
INIT_SETUP_VIEW
```

Rule:

- MOTO Tenant Admin được gọi API.
- MOTO User thường chỉ được gọi nếu được cấp quyền.
- Platform Admin không gọi trực tiếp API này trong tenant portal nếu không có impersonation hợp lệ.

## 16. Security

| Item | Nội dung |
| --- | --- |
| Authentication | Bắt buộc JWT / session token hợp lệ |
| Authorization | Kiểm tra permission `INIT_SETUP_VIEW` |
| Tenant Isolation | Resolve tenant từ token/session và `X-Tenant-ID` |
| Cross Tenant | Không cho phép đọc dữ liệu tenant khác |
| Schema Access | Chỉ query schema tương ứng với `platform.tenant_registry.schema_name` |
| Sensitive Data | Không trả `password_hash` từ `tmp_moto_setup_credential` |
| Logging | Không log token, password, password_hash |
| RLS | Nếu có RLS thì query phải chạy bằng tenant role tương ứng |
| Input | Validate query parameter trước khi xử lý |

---

## 17. Frontend Behavior

### 17.1 Khi mở màn hình Initial Setup

Frontend gọi API:

```
GET /api/v1/moto/init-setup?include_data=true
```

Sau khi nhận response:

| Điều kiện | Frontend xử lý |
| --- | --- |
| `setup.setup_completed = false` | Hiển thị wizard tại `setup.current_step` |
| `setup.setup_completed = true` | Redirect dashboard hoặc hiển thị màn hình đã hoàn tất |
| `steps[].status = completed` | Hiển thị step màu completed |
| `steps[].status = in_progress` | Hiển thị step active |
| `steps[].status = not_started` | Hiển thị step inactive |
| `data.company` có dữ liệu | Pre-fill form Step 1 |
| `data.offices` có dữ liệu | Hiển thị danh sách registered office |
| `data.departments` có dữ liệu | Hiển thị danh sách registered department |
| `data.users` có dữ liệu | Hiển thị danh sách registered user |

---

### 17.2 Mapping theo hình design

| Step trên UI | Dữ liệu từ API |
| --- | --- |
| 1. 企業情報 | `data.company`, `data.contacts` |
| 2. 利用情報設定 | `data.usage_setting` |
| 3. 事業所設定 | `data.offices` |
| 4. 部署情報 | `data.departments` |
| 5. ユーザー登録 | `data.users` |
| 6. 承認ルート設定 | `data.approval_routes` nếu có |
| 7. 利用開始 | Tổng hợp toàn bộ data |

---

## 18. Non-functional Requirements

| Item | Yêu cầu |
| --- | --- |
| Performance | Response dưới 2 giây với dữ liệu setup thông thường |
| Timeout | Theo timeout chuẩn backend |
| Data size | Danh sách office/department/user nên giới hạn nếu quá lớn |
| Availability | API phải hoạt động sau khi tenant schema được provision |
| Compatibility | Response JSON giữ stable field name cho frontend |
| Language | Message error theo `Accept-Language`, default `ja` |
| Idempotent | GET API gọi nhiều lần không làm thay đổi dữ liệu |
| Cache | Không cache dữ liệu setup nhạy cảm ở public cache |

---

## 19. Test Cases

| Test ID | Nội dung | Expected |
| --- | --- | --- |
| TC-001 | Gọi API khi chưa login | Trả 401 Unauthorized |
| TC-002 | Gọi API với token hết hạn | Trả 401 Unauthorized |
| TC-003 | Gọi API không có `X-Tenant-ID` | Trả 400 hoặc lấy từ session |
| TC-004 | Tenant không tồn tại | Trả 404 TENANT_NOT_FOUND |
| TC-005 | Tenant type là SAKI | Trả 403 INVALID_TENANT_TYPE |
| TC-006 | Tenant inactive | Trả 403 TENANT_INACTIVE |
| TC-007 | Tenant MOTO chưa setup | Trả `setup_completed = false` |
| TC-008 | Chưa có company master | `current_step = 1` |
| TC-009 | Có company, chưa có usage setting | `current_step = 2` |
| TC-010 | Có Step 1-2, chưa có office | `current_step = 3` |
| TC-011 | Có office, chưa có department | `current_step = 4` |
| TC-012 | Có department, chưa có user | `current_step = 5` |
| TC-013 | Step 6 optional chưa đăng ký | Không block Step 7 |
| TC-014 | Setup completed | Trả `setup_status = completed` |
| TC-015 | Response không chứa `password_hash` | Pass |
| TC-016 | User tenant A gọi tenant B | Trả 403 Forbidden |

---

## 20. Open Questions

| No | Nội dung cần xác nhận |
| --- | --- |
| Q1 | Usage setting Step 2 sẽ lưu chính thức ở bảng nào? Hiện mapping tạm với `cfg_moto_contract_default` và `cfg_moto_invoice_default`. |
| Q2 | Có bảng riêng để lưu trạng thái từng step không, hay tính theo dữ liệu master đã tồn tại? |
| Q3 | Approval route Step 6 sẽ dùng bảng nào trong MOTO schema? |
| Q4 | Step 3, 4, 5 trong initial setup có bắt buộc ít nhất 1 record hay được skip hoàn toàn? |
| Q5 | Khi `setup_completed_flg = 1`, frontend redirect dashboard hay vẫn hiển thị confirm readonly? |
| Q6 | Có cần phân trang cho `offices`, `departments`, `users` trong API này không? |

---

## 21. Ghi chú

- API này được thiết kế dựa trên màn hình Initial Setup Wizard gồm 7 step trong ảnh design.
- Database mapping dựa theo tài liệu Carima Data Dictionary và Table Catalog.
- API chỉ đọc dữ liệu, không cập nhật trạng thái setup.
- Field `password_hash` trong `tmp_moto_setup_credential` tuyệt đối không được trả về response.
- Các dữ liệu chưa có bảng chính thức trong DB sẽ cần confirm thêm trước khi Final.
- Nếu cần API lưu từng step, sẽ tách thành các API riêng:
    - `MO-COM-000-API-02-Update Company Init Setup`
    - `MO-COM-000-API-03-Update Usage Setting`
    - `MO-COM-000-API-04-Create Init Office`
    - `MO-COM-000-API-05-Create Init Department`
    - `MO-COM-000-API-06-Create Init User`
    - `MO-COM-000-API-07-Create Init Approval Route`
    - `MO-COM-000-API-08-Complete Tenant Init Setup`

---

# Kết luận

API `MO-COM-000-API-01-Get Tenant Init Setup` dùng để lấy toàn bộ trạng thái và dữ liệu ban đầu của MOTO tenant setup.

API phục vụ màn hình `MO-INIT-001 - 派遣元 初期設定ウィザード`, giúp frontend xác định:

- Tenant đang ở step nào
- Step nào đã hoàn thành
- Dữ liệu nào đã được nhập
- Có thể chuyển sang màn hình confirm / 利用開始 hay chưa

API không ghi dữ liệu, không tạo transaction ghi, chỉ đọc từ `platform.tenant_registry` và các bảng master/config trong `tenant_moto_<id>`.