# 追加導入手続き

System: Platform SaaS Admin
Menu: Tenant Management
メニュー: テナント管理
Screen ID: PA-TEN-018
Screen (VI): Additional Onboarding Procedure
Giải thích tính năng: Thực hiện thêm module/setup onboarding và đăng ký nâng cấp tài nguyên.
機能説明: 追加導入手続きを行う。
Thông tin hiển thị trên màn hình: Requested module, onboarding status, approval status
画面表示情報: 追加機能、導入状態、承認状態
URL: /admin/tenants/:id/additional-onboarding
システム: プラットフォーム管理
API List: PA-TEN-018-API-01-Additional Onboarding Procedure (https://www.notion.so/PA-TEN-018-API-01-Additional-Onboarding-Procedure-368f02c407dd80fe9d1fd15babefa643?pvs=21)
Combined: No
Screen Specs: No
Status: Not started

# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-TEN-018 |
| Tên màn hình | Đăng ký bổ sung tính năng Tenant |
| Tên tiếng Nhật | 追加導入手続き |
| Module | Tenant Management (テナント管理) |
| URL | /admin/tenants/{id}/additional-onboarding |
| Actor | Platform SaaS Admin |
| Priority | P2 |

---

# 2. Mục đích

Cho phép quản trị viên hệ thống (Platform SaaS Admin) thực hiện đăng ký kích hoạt thêm các module tính năng hoặc mua thêm tài nguyên (tăng số lượng người dùng tối đa, tăng dung lượng lưu trữ) cho một Tenant cụ thể ngoài gói dịch vụ ban đầu. Đồng thời hiển thị lịch sử đăng ký bổ sung và trạng thái phê duyệt để theo dõi cước phí phát sinh.

Sau khi lưu thành công:
- Tạo mới yêu cầu đăng ký bổ sung trong database ở trạng thái Chờ duyệt (`申請中`).
- Sau khi được phê duyệt, hệ thống tự động cập nhật nâng hạn mức tài nguyên/module tương ứng cho Tenant.
- Ghi Audit Log hệ thống.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống SaaS Platform Admin.
- Có quyền quản trị tương ứng (`platform.tenant.additional_onboarding_procedure.view`).
- Tenant ID (hoặc `:id` trên URL) tồn tại trong hệ thống.

## Điều kiện sau

- Gửi yêu cầu đăng ký bổ sung tính năng/tài nguyên thành công.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình | Action |
| --- | --- | --- |
| PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) | Click nút "追加導入" |
| PA-TEN-008 | 派遣先テナント詳細 (SAKI Tenant Detail) | Click nút "追加導入" |

---

## Màn hình đích

| Action | Screen |
| --- | --- |
| Cancel / Back | Tenant Detail tương ứng (PA-TEN-002 hoặc PA-TEN-008) |

---

# 5. UI/UX Layout

```text
+---------------------------------------------------------------------------------------------------+
|  [Breadcrumb: テナント管理 / テナント詳細 / 追加導入手続き]                                       |
|                                                                                                   |
|  ■ テナント情報 (Thông tin Tenant)                                                                 |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | テナントコード     | SAKI-0001                          |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | 企業名             | 株式会社サキケア                    |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | 契約プラン         | スタンダードプラン (Standard Plan) |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|                                                                                                   |
|  ■ 追加導入申請 (Đăng ký bổ sung)                                                                 |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | 追加機能           | [ ] 勤怠管理 (Chấm công)           [ ] 36協定アラーム (Cảnh báo 36)      |  |
|  | (Module bổ sung)   | [ ] 請求管理 (Hóa đơn)             [ ] 外部API連携 (API ngoài)           |  |
|  |                    | * Chỉ hiển thị các module hiện tại chưa được kích hoạt cho Tenant.       |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | 追加ユーザー数     | [ 20      ] ユーザー                                                    |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | 追加ストレージ量   | [ 50      ] GB                                                          |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | 申請理由 / 備考    | [ Nhập lý do đăng ký bổ sung hoặc ghi chú tại đây...                 ]  |
|  |                    |                                                                         |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  |                    |                                                      [ 申請送信 ]       |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|                                                                                                   |
|  ■ 申請履歴 (Lịch sử đăng ký)                                                                     |
|  +------------+--------------------------------------------+------------------------------------+  |
|  | 申請日     | 申請内容 (Nội dung đăng ký)                | 承認状態 (Status)                  |  |
|  +------------+--------------------------------------------+------------------------------------+  |
|  | 2026-06-11 | ユーザー数: +20, ストレージ: +50GB         | [ 申請中 ]                         |  |
|  +------------+--------------------------------------------+------------------------------------+  |
|  | 2026-05-10 | 機能追加: 請求管理                         | [ 承認済み ]                       |  |
|  +------------+--------------------------------------------+------------------------------------+  |
|                                                                                                   |
|  +---------------------------------------------------------------------------------------------+  |
|  |                                         [ 戻る ]                                            |  |
|  +---------------------------------------------------------------------------------------------+  |
+---------------------------------------------------------------------------------------------------+
```

---

# 6. Quy tắc UI/UX

## Thông tin Tenant & Plan hiện tại (Readonly Area)

- Mã Tenant, Tên công ty và Gói dịch vụ hiện tại được hiển thị ở dạng chỉ đọc đầu trang để quản trị viên dễ dàng đối chiếu.

## Đăng ký bổ sung (Request Form)

- **追加機能 (Additional Modules)**: Chỉ hiển thị các Checkbox dành cho các tính năng/module hiện tại **chưa được kích hoạt** của Tenant này. Các tính năng đã kích hoạt sẽ được ẩn đi.
- **Số lượng User/Storage bổ sung**:
  - Ô nhập số (Number Input) giới hạn chỉ cho phép nhập số nguyên dương lớn hơn hoặc bằng 0.
- **Nút 申請送信 (Submit Request)**:
  - Chỉ sáng lên khi có thay đổi trên form (chọn ít nhất 1 module hoặc nhập số lượng user/storage > 0).

## Bảng lịch sử đăng ký (Request History Table)

- Hiển thị danh sách các yêu cầu đăng ký bổ sung trước đây, được sắp xếp theo ngày gửi giảm dần (mới nhất lên đầu).
- **Trạng thái phê duyệt (Status Badge)**:
  - `申請中` (Pending): Màu vàng.
  - `承認済み` (Approved): Màu xanh lá cây.
  - `却下` (Rejected): Màu đỏ.

---

# 7. Định nghĩa Item

## Thông tin Tenant (Readonly)

| No | Item | Type | Required | Format | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | Label | - | varchar(50) | central_db.mst_tenant.tenant_code |
| 2 | Tên công ty (企業名) | Label | - | varchar(255) | central_db.mst_tenant.company_name |
| 3 | Gói dịch vụ (契約プラン) | Label | - | varchar(100) | central_db.mst_tenant.plan_name |

## Đăng ký bổ sung (追加導入申請)

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 4 | Module bổ sung | Checkbox | No | List các module chưa active | requested_modules |
| 5 | Số User bổ sung | Number Input | No | Số nguyên >= 0 | additional_users |
| 6 | Dung lượng Storage bổ sung | Number Input | No | Số nguyên >= 0 (GB) | additional_storage_gb |
| 7 | Lý do / Ghi chú | Textarea | No | Tối đa 500 ký tự | notes |
| 8 | Gửi yêu cầu | Button | - | Thực hiện gửi form đăng ký lên Server | - |

## Lịch sử đăng ký (申請履歴)

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 9 | Bảng lịch sử | Table | - | Danh sách các yêu cầu cũ | - |
| 10 | Quay lại (戻る) | Button | - | Quay lại trang chi tiết Tenant | - |

---

# 8. Validation

## Bắt buộc điền thông tin đăng ký

| Rule | Message Code | Message |
| --- | --- | --- |
| Phải chọn ít nhất một module hoặc nhập user/storage bổ sung. | CMS-VAL-141 | 追加する機能、ユーザー数、またはストレージ量のいずれかを入力してください。 (Vui lòng chọn tính năng cần thêm hoặc nhập số lượng user/dung lượng storage bổ sung.) |
| Số lượng nhập vào phải lớn hơn hoặc bằng 1. | CMS-VAL-142 | 追加ユーザー数およびストレージ量は1以上の数値を入力してください。 (Số lượng user hoặc dung lượng lưu trữ bổ sung phải lớn hơn hoặc bằng 1.) |

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình.

### Process

1. Lấy `:id` (Tenant ID) và loại Tenant (MOTO hoặc SAKI) từ URL / ngữ cảnh điều hướng.
2. Gửi yêu cầu gọi API tương ứng:
   - Nếu là MOTO Tenant (派遣元): Gọi API `GET /api/v1/admin/moto-tenants/{id}/` (PA-TEN-002-API-01).
   - Nếu là SAKI Tenant (派遣先): Gọi API `GET /api/v1/admin/saki-tenants/{id}/` (PA-TEN-008-API-01).
3. Server trả về thông tin Tenant, bao gồm các cấu hình module đã/chưa kích hoạt và lịch sử đăng ký bổ sung lồng trong object `additional_onboarding_settings`.
4. Render các Checkbox module tương ứng và nạp dữ liệu lịch sử vào bảng.

---

## Submit Request (Gửi yêu cầu đăng ký)

### Trigger

Click nút "申請送信".

### Process

1. Thực hiện validate dữ liệu form (Kiểm tra xem có ít nhất một thông tin đăng ký hợp lệ theo quy tắc `VAL-141` và `VAL-142`).
2. Nếu không hợp lệ, dừng xử lý và hiển thị thông báo lỗi.
3. Hiển thị popup xác nhận hành động: `CMS-VAL-85` (Target: "追加導入申請").
4. Nếu người dùng chọn OK:
   - Kích hoạt gọi API `POST /api/v1/admin/tenants/{id}/additional-onboarding/` (PA-TEN-018-API-01).
   - Server lưu bản ghi mới vào bảng `central_db.tenant_additional_request` với trạng thái `1` (申請中).
   - Ghi nhận Audit Log.
   - Hiển thị Toast thông báo thành công `CMS-VAL-79` ("追加導入申請を送信しました。").
   - Reset các trường nhập liệu trên form và tải lại dữ liệu Tenant để cập nhật danh sách lịch sử đăng ký.

---

## Cancel / Back

### Trigger

Click Back (戻る) hoặc Cancel.

### Process

1. Điều hướng quay trở lại màn hình chi tiết Tenant tương ứng (`PA-TEN-002` hoặc `PA-TEN-008`).

---

# 10. Mapping Database

## central_db.tenant_additional_request (Bảng lưu các yêu cầu bổ sung của Tenant)

| Column | Type | Value / Mapping | Mô tả |
| --- | --- | --- | --- |
| id | bigint | *System Generated* | PK tự tăng |
| tenant_id | bigint | `mst_tenant.id` | FK liên kết tới Tenant |
| requested_modules | json | `requested_modules` | Danh sách cấu hình các module đăng ký thêm |
| additional_users | int | `additional_users` | Số lượng tài khoản đăng ký thêm |
| additional_storage_gb | int | `additional_storage_gb` | Dung lượng lưu trữ đăng ký thêm (GB) |
| status | tinyint(1) | `status` | Trạng thái yêu cầu (1: 申請中, 2: 承認済み, 3: 却下, 4: キャンセル済み) |
| notes | text | `notes` | Ghi chú lý do đăng ký |
| created_by | bigint | `mst_user.id` | ID người tạo yêu cầu |
| created_at | datetime | *System Generated* | Thời điểm gửi yêu cầu |

---

# 11. API Mapping

Màn hình này tích hợp lồng ghép dữ liệu danh sách lịch sử đăng ký bổ sung và các module chưa kích hoạt vào các API chi tiết Tenant (MOTO/SAKI) sẵn có, và chỉ sử dụng API POST chính thức để gửi yêu cầu mới.

## 1. Lấy dữ liệu cấu hình bổ sung và lịch sử

### 1.1 Đối với Tenant MOTO (派遣元)
Sử dụng API PA-TEN-002-API-01-Moto tenant detail:
```
GET /api/v1/admin/moto-tenants/{id}/
```

### 1.2 Đối với Tenant SAKI (派遣先)
Sử dụng API PA-TEN-008-API-01-Saki tenant detail:
```
GET /api/v1/admin/saki-tenants/{id}/
```

### Response lồng ghép (Ví dụ cho SAKI Tenant Detail)
```json
{
  "status": "success",
  "data": {
    "id": 12,
    "tenant_code": "SAKI-0001",
    "company_name": "株式会社サキケア",
    "tenant_type": 2,
    "status": "Active",
    "additional_onboarding_settings": {
      "current_plan": "スタンダードプラン",
      "inactive_modules": [
        { "key": "feature_attendance_flg", "name": "勤怠管理" },
        { "key": "feature_alarm_36_flg", "name": "36協定アラーム" },
        { "key": "api_integration_flg", "name": "外部API連携" }
      ],
      "history": [
        {
          "request_id": 101,
          "created_at": "2026-06-11 18:00:00",
          "description": "ユーザー数: +20, ストレージ: +50GB",
          "status": 1
        },
        {
          "request_id": 85,
          "created_at": "2026-05-10 10:00:00",
          "description": "機能追加: 請求管理",
          "status": 2
        }
      ]
    }
  }
}
```

---

## 2. Gửi yêu cầu đăng ký bổ sung

Sử dụng API PA-TEN-018-API-01-Additional Onboarding Procedure:
```
POST /api/v1/admin/tenants/{id}/additional-onboarding/
```

### Request
```json
{
  "requested_modules": {
    "feature_attendance_flg": 1
  },
  "additional_users": 20,
  "additional_storage_gb": 50,
  "notes": "業務拡大に伴うユーザー枠およびストレージの追加申請"
}
```

### Response
```json
{
  "status": "success",
  "message": "submit_additional_request_success"
}
```

---

# 12. Notification

Khi có yêu cầu đăng ký bổ sung mới được gửi lên, hệ thống tự động gửi email thông báo cho phòng ban Kế toán/Kinh doanh của SaaS Platform để thực hiện liên hệ làm thủ tục thanh toán và phát hành hóa đơn điều chỉnh.

---

# 13. Approval Flow

- Yêu cầu sau khi gửi ở trạng thái `申請中` (Chờ duyệt).
- Sau khi được bộ phận phê duyệt xác nhận thanh toán thành công (thao tác trực tiếp từ trang Admin hoặc qua API), trạng thái chuyển sang `承認済み` (Đã duyệt). Hệ thống tự động nâng hạn mức tài nguyên/module tương ứng cho Tenant trong bảng `central_db.tenant_feature` và `central_db.mst_tenant`.

---

# 14. Permission

| Action | Platform SaaS Admin | Platform SaaS Staff |
| --- | --- | --- |
| View Request History | O | O |
| Submit Request | O | X |

---

# 15. Audit Log

| Action | Log | Nội dung lưu |
| --- | --- | --- |
| Create Request | Yes | [User] đã gửi yêu cầu đăng ký bổ sung cho Tenant [tenant_code]. Chi tiết: User [additional_users], Storage [additional_storage_gb] GB, Modules [requested_modules]. |

---

# 16. Error Handling

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) |
| --- | --- | --- |
| VAL001 | 入力データが不正です。 | Dữ liệu đầu vào không hợp lệ. |
| VAL141 | 追加する機能、ユーザー数、またはストレージ量のいずれかを入力してください。 | Vui lòng chọn tính năng cần thêm hoặc nhập số lượng user/dung lượng storage bổ sung. |
| VAL142 | 追加ユーザー数およびストレージ量は1以上の数値を入力してください。 | Số lượng user hoặc dung lượng lưu trữ bổ sung phải lớn hơn hoặc bằng 1. |
| SYS001 | システムエラーが発生しました。 | Lỗi hệ thống. |

---

# 17. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix