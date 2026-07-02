# 導入準備状況

System: Platform SaaS Admin
Menu: Tenant Management
メニュー: テナント管理
Screen ID: PA-TEN-017
Screen (VI): Tenant Onboarding Status
Giải thích tính năng: Theo dõi onboarding/setup tenant trước khi bàn giao hệ thống.
機能説明: テナント導入準備状況を確認する。
Thông tin hiển thị trên màn hình: Setup checklist, completed steps, pending tasks
画面表示情報: 導入チェックリスト、完了項目、未完了項目
URL: /admin/tenants/:id/onboarding
Business Flow: 派遣元テナント管理フロー (https://www.notion.so/362f02c407dd80c5acdbe79deb66f419?pvs=21)
システム: プラットフォーム管理
API List: PA-TEN-017-API-01-Tenant Onboarding Status (https://www.notion.so/PA-TEN-017-API-01-Tenant-Onboarding-Status-368f02c407dd804daf12ed16606fd965?pvs=21)
Combined: No
Screen Specs: No
Status: Not started

# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-TEN-017 |
| Tên màn hình | Tình trạng chuẩn bị triển khai |
| Tên tiếng Nhật | 導入準備状況 |
| Module | Tenant Management (テナント管理) |
| URL | /admin/tenants/{id}/onboarding |
| Actor | Platform SaaS Admin |
| Priority | P2 |

---

# 2. Mục đích

Cho phép quản trị viên hệ thống (Platform SaaS Admin) giám sát tiến trình thiết lập và triển khai hệ thống (Onboarding checklist) cho một Tenant mới tạo. Giúp xác minh các cấu hình bắt buộc trước khi bàn giao hệ thống, đồng thời cập nhật trạng thái hoạt động chính thức (Go-live) của Tenant.

Sau khi lưu thành công:
- Cập nhật tiến độ checklist và trạng thái triển khai trong database trung tâm.
- Kích hoạt gửi email tự động chào mừng và bàn giao cho Tenant Admin nếu Tenant chuyển sang trạng thái hoạt động chính thức (`本番稼働中`).
- Ghi Audit Log hệ thống.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống SaaS Platform Admin.
- Có quyền quản trị tương ứng (`platform.tenant.tenant_onboarding_status.view`).
- Tenant ID (hoặc `:id` trên URL) tồn tại trong hệ thống.

## Điều kiện sau

- Cập nhật tình trạng triển khai và trạng thái hoạt động Tenant thành công.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình | Action |
| --- | --- | --- |
| PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) | Click nút "導入準備状況" |
| PA-TEN-008 | 派遣先テナント詳細 (SAKI Tenant Detail) | Click nút "導入準備状況" |

---

## Màn hình đích

| Action | Screen |
| --- | --- |
| Save Success | Tenant Detail tương ứng (PA-TEN-002 hoặc PA-TEN-008) |
| Cancel | Tenant Detail tương ứng (PA-TEN-002 hoặc PA-TEN-008) |

---

# 5. UI/UX Layout

```text
+---------------------------------------------------------------------------------------------------+
|  [Breadcrumb: テナント管理 / テナント詳細 / 導入準備状況]                                         |
|                                                                                                   |
|  ■ テナント情報 (Thông tin Tenant)                                                                 |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | テナントコード     | SAKI-0001                          |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | 企業名             | 株式会社サキケア                    |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | テナントタイプ     | 派遣先                             |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|                                                                                                   |
|  ■ 導入進捗 (Tiến độ triển khai)                                                                  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | 総合進捗率         | 80% (4 / 5 項目完了)                                                     |  |
|  | (Overall Progress) | [==================================         ]                           |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|                                                                                                   |
|  ■ 導入チェックリスト (Checklist triển khai)                                                         |
|  +----+--------------------------------------------+-----------------------+--------------------+  |
|  | No | 項目 (Hạng mục)                            | 判定 (Cách xác định)  | 状態 (Trạng thái)  |  |
|  +----+--------------------------------------------+-----------------------+--------------------+  |
|  | 1  | テナント・管理者アカウント作成             | システム自動判定       | [ 完了 (Auto) ]    |  |
|  |    | (Tạo Tenant & TK Admin)                    |                       |                    |  |
|  +----+--------------------------------------------+-----------------------+--------------------+  |
|  | 2  | 利用機能設定 (Cấu hình tính năng)          | システム自動判定       | [ 完了 (Auto) ]    |  |
|  +----+--------------------------------------------+-----------------------+--------------------+  |
|  | 3  | ドメイン・SSL設定 (Cấu hình tên miền)      | システム自動判定       | [ 完了 (Auto) ]    |  |
|  +----+--------------------------------------------+-----------------------+--------------------+  |
|  | 4  | ブランド設定 (Branding logo/theme)         | システム自動判定       | [ 完了 (Auto) ]    |  |
|  +----+--------------------------------------------+-----------------------+--------------------+  |
|  | 5  | 初期マスタデータ登録                       | 手動チェック           | [ ] 完了にする     |  |
|  |    | (Khởi tạo dữ liệu Master ban đầu)          |                       | (Đánh dấu hoàn thành)|
|  +----+--------------------------------------------+-----------------------+--------------------+  |
|                                                                                                   |
|  ■ 導入ステータス設定 (Thiết lập trạng thái triển khai)                                             |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | 導入ステータス *   | [ 準備中 (Preparing)                     [v] ]                          |  |
|  |                    | * Options: 準備中 (Preparing), テスト利用中 (Testing), 本番稼働中 (Active) |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|                                                                                                   |
|  +---------------------------------------------------------------------------------------------+  |
|  |                                     [ キャンセル ]   [ 設定保存 ]                           |  |
|  +---------------------------------------------------------------------------------------------+  |
+---------------------------------------------------------------------------------------------------+
```

---

# 6. Quy tắc UI/UX

## Thông tin Tenant (Readonly Area)

- Mã Tenant, Tên công ty và Loại Tenant được hiển thị ở dạng chỉ đọc đầu trang để quản trị viên dễ dàng đối chiếu.

## Checklist triển khai (導入チェックリスト)

- Giao diện hiển thị danh sách 5 bước triển khai bắt buộc. Trạng thái mỗi bước sẽ ảnh hưởng trực tiếp đến thanh tiến độ (Progress Bar) tổng thể:
  - **Bước 1**: Tự động đánh dấu `完了` nếu thông tin Tenant và bản ghi User đầu tiên đã tồn tại.
  - **Bước 2**: Tự động đánh dấu `完了` nếu Tenant đã cấu hình ít nhất 1 Module tại màn hình cấu hình tính năng (`PA-TEN-013`).
  - **Bước 3**: Tự động đánh dấu `完了` nếu trạng thái DNS và SSL tại màn hình cấu hình domain (`PA-TEN-014`) đều ở trạng thái `有効` (Active).
  - **Bước 4**: Tự động đánh dấu `完了` nếu Tenant đã cấu hình tải lên Logo hoặc chọn màu sắc Theme tại màn hình branding (`PA-TEN-015`).
  - **Bước 5**: Quản trị viên sử dụng Checkbox để tự đánh dấu hoàn thành thủ công sau khi đã hỗ trợ import dữ liệu master ban đầu cho Tenant.

## Thiết lập trạng thái triển khai (導入ステータス設定)

- Quản trị viên sử dụng hộp chọn (Select Box) để cập nhật trạng thái tổng thể của Tenant:
  - `準備中` (Preparing): Trạng thái mặc định khi mới khởi tạo.
  - `テスト利用中` (Testing): Cho phép người dùng của Tenant đăng nhập dùng thử/kiểm tra hệ thống.
  - `本番稼働中` (Active): Cho phép go-live chính thức.

---

# 7. Định nghĩa Item

## Thông tin Tenant (Readonly)

| No | Item | Type | Required | Format | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | Label | - | varchar(50) | central_db.mst_tenant.tenant_code |
| 2 | Tên công ty (企業名) | Label | - | varchar(255) | central_db.mst_tenant.company_name |
| 3 | Loại Tenant (テナントタイプ) | Label | - | tinyint (1: MOTO, 2: SAKI) | central_db.mst_tenant.tenant_type |

## Checklist & Trạng thái triển khai

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 4 | Tiến độ tổng thể (総合進捗率) | Progress Bar | - | Tỉ lệ % tự động tính toán (Số bước xong / 5) | - |
| 5 | Đăng ký Master Data (No 5) | Checkbox | No | 0: 未完了, 1: 完了 | master_data_imported |
| 6 | Trạng thái triển khai tổng thể | Select Box | Yes | 1: 準備中 (Preparing)<br>2: テスト利用中 (Testing)<br>3: 本番稼働中 (Active) | central_db.mst_tenant.status |

## Các nút hành động (Footer Buttons)

| No | Item | Loại | Mô tả |
| --- | --- | --- | --- |
| 7 | Lưu cấu hình (設定保存) | Button | Kích hoạt gửi form cấu hình cập nhật lên hệ thống sau khi validate thành công |
| 8 | Hủy bỏ (キャンセル) | Button | Quay lại trang chi tiết Tenant tương ứng |

---

# 8. Validation

## Bắt buộc hoàn tất checklist trước khi Active (Go-live)

| Rule | Message Code | Message |
| --- | --- | --- |
| Toàn bộ 5 bước checklist phải là `完了` khi chọn trạng thái tổng thể là `本番稼働中`. | CMS-VAL-131 | すべての導入チェックリストが完了するまで、本番稼働中に設定することはできません。 (Không thể chuyển trạng thái sang hoạt động chính thức cho đến khi hoàn thành toàn bộ checklist triển khai.) |

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình.

### Process

1. Lấy `:id` (Tenant ID) từ URL.
2. Gửi yêu cầu gọi API `GET /api/v1/admin/tenants/{id}/onboarding/`.
3. Server trả về trạng thái checklist 1-4 do hệ thống tự đánh giá, trạng thái lưu của bước 5 và trạng thái Tenant hiện tại.
4. Render số liệu, cập nhật thanh tiến độ và thiết lập trạng thái hộp chọn Dropdown tương ứng.

---

## Checkbox Master Data Toggle

### Trigger

Click chọn hoặc bỏ chọn checkbox "初期マスタデータ登録".

### Process

1. Đánh dấu trạng thái bước 5 là `完了` hoặc `未完了`.
2. Tính toán lại tỉ lệ hoàn thành tổng thể và cập nhật giao diện thanh tiến độ ngay lập tức (mỗi bước hoàn thành cộng 20%).

---

## Cancel

### Trigger

Click Cancel (キャンセル).

### Process

1. Hiển thị popup xác nhận:
   ```
   Dữ liệu chưa lưu sẽ bị mất.
   Bạn có muốn thoát không?
   ```
2. Nếu chọn Đồng ý (OK), điều hướng quay trở lại màn hình chi tiết Tenant tương ứng (`PA-TEN-002` hoặc `PA-TEN-008`).

---

## Save Onboarding (Lưu cấu hình)

### Trigger

Click 設定保存 (Lưu cấu hình).

### Process

1. Kiểm tra logic validate:
   - Nếu `導入ステータス` được chọn là `3` (本番稼働中 - Active) nhưng tiến độ chưa đạt 100% (còn bước chưa hoàn thành), dừng xử lý và hiển thị thông báo lỗi `CMS-VAL-131`.
2. Hiển thị popup xác nhận hành động: `CMS-VAL-85` (Target: "導入準備状況").
3. Nếu người dùng chọn OK:
   - Kích hoạt gọi API `PATCH /api/v1/admin/tenants/{id}/onboarding/`.
   - Server cập nhật thông tin cột `master_data_imported` trong bảng `central_db.tenant_onboarding` và đồng thời cập nhật cột `status` trong bảng `central_db.mst_tenant`.
   - Nếu trạng thái mới là `本番稼働中`, Server kích hoạt job gửi email bàn giao hệ thống tự động cho Tenant Admin.
   - Ghi nhận Audit Log.
   - Hiển thị Toast thông báo thành công `CMS-VAL-79` ("導入準備状況を更新しました。").
   - Điều hướng quay trở lại màn hình chi tiết Tenant tương ứng.

---

# 10. Mapping Database

## central_db.tenant_onboarding (Bảng theo dõi checklist triển khai Tenant)

| Column | Type | Value / Mapping | Mô tả |
| --- | --- | --- | --- |
| id | bigint | *System Generated* | PK tự tăng |
| tenant_id | bigint | `mst_tenant.id` | FK liên kết tới Tenant |
| master_data_imported | tinyint(1) | `master_data_imported` | Cờ xác nhận đã import dữ liệu master ban đầu (0: Chưa, 1: Xong) |
| updated_at | datetime | *System Generated* | Thời điểm cập nhật cuối cùng |

## central_db.mst_tenant (Bảng thông tin chính Tenant)

| Column | Type | Value / Mapping | Mô tả |
| --- | --- | --- | --- |
| status | tinyint(1) | `onboarding_status` | Trạng thái Tenant (1: 準備中, 2: テスト利用中, 3: 本番稼働中) |

---

# 11. API Mapping

## Get Tenant Onboarding Status

```
GET /api/v1/admin/tenants/{id}/onboarding/
```

### Response

```json
{
  "status": "success",
  "data": {
    "tenant_id": 12,
    "tenant_code": "SAKI-0001",
    "company_name": "株式会社サキケア",
    "tenant_type": 2,
    "checklist": {
      "step_account_created": 1,
      "step_features_configured": 1,
      "step_domain_verified": 1,
      "step_branding_configured": 0,
      "step_master_data_imported": 0
    },
    "onboarding_status": 1
  }
}
```

---

## Update Tenant Onboarding Status

```
PATCH /api/v1/admin/tenants/{id}/onboarding/
```

Request

```json
{
  "master_data_imported": 1,
  "onboarding_status": 2
}
```

Response

```json
{
  "status": "success",
  "message": "update_tenant_onboarding_success"
}
```

---

# 12. Notification

Khi `onboarding_status` được cập nhật thành `3` (本番稼働中):
- Kích hoạt gửi Email thông báo bàn giao hệ thống kèm tài liệu hướng dẫn đăng nhập đến tài khoản Tenant Admin của Tenant.

---

# 13. Approval Flow

Không áp dụng cho màn hình này.

---

# 14. Permission

| Action | Platform SaaS Admin | Platform SaaS Staff |
| --- | --- | --- |
| View Onboarding Status | O | O |
| Change Onboarding Status | O | X |

---

# 15. Audit Log

| Action | Log | Nội dung lưu |
| --- | --- | --- |
| Update Onboarding | Yes | [User] đã cập nhật tiến độ triển khai cho Tenant [tenant_code]. Trạng thái checklist Master: [master_data_imported], Trạng thái hoạt động mới: [onboarding_status]. |

---

# 16. Error Handling

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) |
| --- | --- | --- |
| VAL001 | 入力データが不正です。 | Dữ liệu đầu vào không hợp lệ. |
| VAL131 | すべての導入チェックリストが完了するまで、本番稼働中に設定することはできません。 | Không thể chuyển trạng thái sang hoạt động chính thức cho đến khi hoàn thành toàn bộ checklist triển khai. |
| SYS001 | システムエラーが発生しました。 | Lỗi hệ thống. |

---

# 17. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix