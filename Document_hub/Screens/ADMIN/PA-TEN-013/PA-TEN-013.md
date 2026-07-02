# テナント機能設定

System: Platform SaaS Admin
Menu: Tenant Management
メニュー: テナント管理
Screen ID: PA-TEN-013
Screen (VI): Tenant Plan Setting
Giải thích tính năng: Cấu hình gói dịch vụ (plan_code) cho tenant
機能説明: テナントの利用プラン（plan_code）を設定する。
Thông tin hiển thị trên màn hình: Gói dịch vụ (plan_code), danh sách tính năng tương ứng của gói (Readonly)
画面表示情報: 利用プラン、プランに紐づく機能一覧（参照のみ）
URL: /admin/tenants/:id/features
システム: プラットフォーム管理
API List: PA-TEN-013-API-01-Tenant feature setting (https://www.notion.so/PA-TEN-013-API-01-Tenant-feature-setting-368f02c407dd80e9ab21f689af44d1af?pvs=21)
Combined: No
Screen Specs: No
Status: Draft

# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-TEN-013 |
| Tên màn hình | Cấu hình tính năng Tenant |
| Tên tiếng Nhật | テナント機能設定 |
| Module | Tenant Management (テナント管理) |
| URL | /admin/tenants/{id}/features |
| Actor | Platform SaaS Admin |
| Priority | P1 |

---

# 2. Mục đích

Cho phép quản trị viên hệ thống (Platform SaaS Admin) thay đổi gói dịch vụ (`plan_code`) cho một Tenant cụ thể. Quyền hạn sử dụng module và tính năng sẽ tự động thay đổi tương ứng theo gói dịch vụ đã chọn.

Sau khi lưu thành công:
- Cập nhật gói dịch vụ (`plan_code`) của Tenant trong bảng đăng ký tenant trung tâm (`platform.tenant_registry`).
- Thay đổi này sẽ có hiệu lực ngay lập tức đối với người dùng thuộc Tenant khi truy cập hệ thống.
- Ghi Audit Log hệ thống.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống SaaS Platform Admin.
- Có quyền quản trị tương ứng (`platform.tenant.tenant_feature_setting.edit`).
- Tenant ID (hoặc `:id` trên URL) tồn tại trong hệ thống.

## Điều kiện sau

- Cập nhật thiết lập gói dịch vụ Tenant thành công.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình |
| --- | --- |
| PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) - Click "機能設定" |
| PA-TEN-008 | 派遣先テナント詳細 (SAKI Tenant Detail) - Click "機能設定" |

---

## Màn hình đích

| Action | Screen |
| --- | --- |
| Save Success | Tenant Detail tương ứng (PA-TEN-002 hoặc PA-TEN-008) |
| Cancel | Tenant Detail tương ứng (PA-TEN-002 hoặc PA-TEN-008) |

---

# 5. UI/UX Layout

![Giao diện cấu hình tính năng](image.png)

---

# 6. Quy tắc UI/UX

## Thông tin Tenant (Readonly Area)

- Mã Tenant, Tên công ty và Loại Tenant được hiển thị ở dạng chỉ đọc đầu trang để quản trị viên dễ dàng đối chiếu.

## Thiết lập Gói dịch vụ (Plan Selection)

- Sử dụng Dropdown hoặc Radio Group để chọn gói dịch vụ sử dụng cho Tenant:
  - **LITE**
  - **STANDARD**
  - **PRO**
  - **ENTERPRISE**
- Khi chọn gói dịch vụ, phần **Danh sách tính năng đi kèm (Readonly)** bên dưới sẽ tự động thay đổi trạng thái Checkbox/Toggle (đang bật/tắt) để quản trị viên biết gói đó bao gồm những tính năng gì. Admin không thể click thay đổi trực tiếp các Checkbox/Toggle này.

## Nút 設定保存 (Lưu cấu hình)

- Disable khi đang thực hiện submit lên Server.
- Hiển thị trạng thái loading khi đang lưu.

---

# 7. Định nghĩa Item

## Thông tin Tenant (Readonly)

| No | Item | Type | Required | Format | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | Label | - | varchar(50) | platform.tenant_registry.company_id |
| 2 | Tên công ty (企業名) | Label | - | varchar(255) | platform.tenant_registry.company_name |
| 3 | Loại Tenant (テナントタイプ) | Label | - | varchar(8) ('saki' hoặc 'moto') | platform.tenant_registry.tenant_type |

## Cấu hình Gói dịch vụ (利用プラン設定)

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 4 | Gói dịch vụ (利用プラン) | Dropdown / Radio | Yes | `LITE`, `STANDARD`, `PRO`, `ENTERPRISE` | platform.tenant_registry.plan_code |

## Danh sách tính năng đi kèm (Danh sách hiển thị Readonly trên UI)

*Trạng thái Bật (ON)/Tắt (OFF) hiển thị tự động dựa trên Gói dịch vụ được chọn:*

| No | Tính năng hiển thị | Gói LITE | Gói STANDARD | Gói PRO | Gói ENTERPRISE |
| --- | --- | --- | --- | --- | --- |
| 5 | Quản lý nhân viên (スタッフ管理) | ON | ON | ON | ON |
| 6 | Quản lý hợp đồng (契約管理) | OFF | ON | ON | ON |
| 7 | Quản lý chấm công (勤怠管理) | OFF | OFF | ON | ON |
| 8 | Cảnh báo Hiệp định 36 (36協定アラーム) | OFF | OFF | ON | ON |
| 9 | Quản lý & Xuất hóa đơn (請求管理) | OFF | ON | ON | ON |
| 10 | Tích hợp API bên ngoài (外部API連携) | OFF | OFF | OFF | ON |

## Các nút hành động (Footer Buttons)

| No | Item | Loại | Mô tả |
| --- | --- | --- | --- |
| 11 | Lưu cấu hình (設定保存) | Button | Kích hoạt gửi gói dịch vụ cập nhật lên hệ thống sau khi xác nhận |
| 12 | Hủy bỏ (キャンセル) | Button | Quay lại trang chi tiết Tenant tương ứng |

---

# 8. Validation

Không có validation nghiệp vụ phức tạp ngoài việc kiểm tra tính hợp lệ của gói dịch vụ đầu vào.

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình cấu hình.

### Process

1. Lấy `:id` (Tenant ID) từ URL.
2. Gửi yêu cầu gọi API `GET /api/v1/admin/tenants/{id}/features/`.
3. Server trả về thông tin tenant và gói dịch vụ `plan_code` hiện tại.
4. Render thông tin Tenant lên đầu trang, thiết lập giá trị gói dịch vụ hiện tại cho Dropdown/Radio.
5. Cập nhật hiển thị danh sách tính năng đi kèm tương ứng ở dạng Readonly.

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

## Save Features (Lưu cấu hình)

### Trigger

Click 設定保存 (Lưu cấu hình).

### Process

1. Hiển thị popup xác nhận hành động: `CMS-VAL-85` (Target: "テナント機能設定").
2. Nếu người dùng chọn OK:
   - Kích hoạt gọi API `PATCH /api/v1/admin/tenants/{id}/features/` kèm `plan_code` đã chọn.
   - Server cập nhật trường `plan_code` trong bảng `platform.tenant_registry`.
   - Ghi nhận Audit Log.
   - Hiển thị Toast thông báo thành công `CMS-VAL-79` ("テナント機能を更新しました。").
   - Điều hướng quay trở lại màn hình chi tiết Tenant tương ứng.

---

# 10. Mapping Database

## platform.tenant_registry (Bảng đăng ký và cấu hình Tenant)

| Column | Type | Value / Mapping | Mô tả |
| --- | --- | --- | --- |
| tenant_id | char(26) | *System Generated* | ULID khóa chính |
| plan_code | varchar(20) | `plan_code` | Mã gói dịch vụ của Tenant |
| updated_at | timestamptz | `now()` | Thời điểm cập nhật |

---

# 11. API Mapping

## Get Tenant Feature Setting

```
GET /api/v1/admin/tenants/{id}/features
```

### Response

```json
{
  "success": true,
  "data": {
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "company_id": "SAKI-0001",
    "company_name": "SAKI Partner Factory",
    "tenant_type": "saki",
    "plan_code": "STANDARD"
  }
}
```

---

## Update Tenant Feature Setting

```
PATCH /api/v1/admin/tenants/{id}/features
```

Request

```json
{
  "plan_code": "PRO"
}
```

Response

```json
{
  "success": true,
  "data": {
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "plan_code": "PRO"
  },
  "message": "Tenant plan updated successfully."
}
```

---

# 12. Notification

Không áp dụng cho màn hình này.

---

# 13. Approval Flow

Không áp dụng cho màn hình này. Thiết lập có hiệu lực trực tiếp sau khi Platform SaaS Admin nhấn thiết lập và lưu thành công.

---

# 14. Permission

| Action | Platform SaaS Admin | Platform SaaS Staff |
| --- | --- | --- |
| View Features | O | O |
| Change Features | O | X |

---

# 15. Audit Log

| Action | Log | Nội dung lưu |
| --- | --- | --- |
| Update Features | Yes | [User] đã cập nhật thiết lập gói dịch vụ cho Tenant [company_id]. Gói cũ: [plan_code_before] -> Gói mới: [plan_code_after]. |

---

# 16. Error Handling

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) |
| --- | --- | --- |
| VAL001 | 入力データが不正です。 | Dữ liệu đầu vào không hợp lệ. |
| SYS001 | システムエラーが発生しました。 | Lỗi hệ thống. |

---

# 17. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix
- Wireframe