# テナント利用状況

System: Platform SaaS Admin
Menu: Tenant Management
メニュー: テナント管理
Screen ID: PA-TEN-016
Screen (VI): Tenant Usage Dashboard
Giải thích tính năng: Dashboard thống kê tình hình sử dụng tài nguyên (users, storage, API, logins) theo tenant.
機能説明: テナント利用状況を表示する。
Thông tin hiển thị trên màn hình: User count, storage usage, API usage, monthly login count
画面表示情報: ユーザー数、ストレージ使用量、API利用数、月間ログイン数
URL: /admin/tenants/:id/usage
システム: プラットフォーム管理
API List: PA-TEN-016-API-01-Tenant Usage Dashboard (https://www.notion.so/PA-TEN-016-API-01-Tenant-Usage-Dashboard-368f02c407dd8081a0c2cd89672a64f6?pvs=21)
Combined: No
Screen Specs: No
Status: Not started

# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-TEN-016 |
| Tên màn hình | Dashboard sử dụng Tenant |
| Tên tiếng Nhật | テナント利用状況 |
| Module | Tenant Management (テナント管理) |
| URL | /admin/tenants/{id}/usage |
| Actor | Platform SaaS Admin, Platform SaaS Staff |
| Priority | P2 |

---

# 2. Mục đích

Cho phép quản trị viên hệ thống (Platform SaaS Admin/Staff) theo dõi và giám sát khối lượng tài nguyên mà một Tenant cụ thể đang sử dụng so với hạn mức gói đăng ký của họ. Các số liệu bao gồm số lượng người dùng đang hoạt động, dung lượng lưu trữ tệp tin, tần suất gọi API và số lượt đăng nhập hàng tháng để hỗ trợ việc tính cước hoặc tư vấn nâng cấp gói dịch vụ kịp thời.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống SaaS Platform Admin.
- Có quyền quản trị hoặc xem thông tin tương ứng (`platform.tenant.tenant_usage_dashboard.view`).
- Tenant ID (hoặc `:id` trên URL) tồn tại trong hệ thống.

## Điều kiện sau

- Hiển thị đầy đủ thông tin thống kê tài nguyên sử dụng của Tenant.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình | Action |
| --- | --- | --- |
| PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) | Click nút "利用状況" |
| PA-TEN-008 | 派遣先テナント詳細 (SAKI Tenant Detail) | Click nút "利用状況" |

---

## Màn hình đích

| Action | Screen |
| --- | --- |
| Cancel / Back | Tenant Detail tương ứng (PA-TEN-002 hoặc PA-TEN-008) |

---

# 5. UI/UX Layout

```text
+---------------------------------------------------------------------------------------------------+
|  [Breadcrumb: テナント管理 / テナント詳細 / 利用状況ダッシュボード]                               |
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
|  ■ リソース利用統計 (Thống kê tài nguyên)                                                           |
|  +-----------------------------------+---------------------------------------------------------+  |
|  | ユーザー数 (Users Active)         | 45 / 100 ユーザー (45.0%)                               |  |
|  |                                   | [====================                     ]             |  |
|  +-----------------------------------+---------------------------------------------------------+  |
|  | ストレージ使用量 (Storage)        | 12.4 GB / 50.0 GB (24.8%)                               |  |
|  |                                   | [===========                              ]             |  |
|  +-----------------------------------+---------------------------------------------------------+  |
|  | 当月API呼び出し数 (API Requests)   | 15,240 リクエスト                                       |  |
|  +-----------------------------------+---------------------------------------------------------+  |
|  | 当月ログイン数 (Logins)            | 120 回                                                  |  |
|  +-----------------------------------+---------------------------------------------------------+  |
|                                                                                                   |
|  ■ 月間利用推移 (Biểu đồ xu hướng 6 tháng gần nhất)                                                |
|  +---------------------------------------------------------------------------------------------+  |
|  |  (Biểu đồ cột/đường hiển thị Lượt đăng nhập & Số lượng API gọi theo tháng)                  |  |
|  |  |                                                                                          |  |
|  |  |      [API]                   [Logins]                                                    |  |
|  |  |     +----+                  +----+                                                       |  |
|  |  |     |    |   +----+         |    |   +----+                                              |  |
|  |  |     |    |   |    |         |    |   |    |                                              |  |
|  |  +-----+----+---+----+---------+----+---+----+----------------------------------------------+  |
|  |         12月     1月      2月      3月      4月      5月                                    |  |
|  +---------------------------------------------------------------------------------------------+  |
|                                                                                                   |
|  +---------------------------------------------------------------------------------------------+  |
|  |                                         [ 戻る ]                                            |  |
|  +---------------------------------------------------------------------------------------------+  |
+---------------------------------------------------------------------------------------------------+
```

---

# 6. Quy tắc UI/UX

## Thông tin Tenant (Readonly Area)

- Mã Tenant, Tên công ty và Loại Tenant được hiển thị ở dạng chỉ đọc đầu trang để quản trị viên dễ dàng đối chiếu.

## Thống kê tài nguyên (Metric Area)

- **ユーザー数 (Users)**: Hiển thị tỉ lệ người dùng đang hoạt động so với hạn mức tối đa của Tenant dưới dạng thanh tiến trình (Progress Bar).
- **ストレージ使用量 (Storage)**: Hiển thị dung lượng lưu trữ tệp tin đã dùng (đơn vị GB) trên tổng dung lượng tối đa. Thanh tiến trình hiển thị màu sắc cảnh báo:
  - Dưới 80%: Màu xanh lá cây.
  - Từ 80% đến 90%: Màu cam (Cảnh báo sắp hết dung lượng).
  - Trên 90%: Màu đỏ (Cảnh báo khẩn cấp).

## Biểu đồ xu hướng (Trend Chart)

- Biểu đồ kết hợp cột và đường hiển thị số lượng API và lượt đăng nhập trong 6 tháng gần nhất.
- Hỗ trợ hiển thị Tooltip thông tin chi tiết (Tooltip) khi di chuột vào từng cột/điểm mốc tháng.

---

# 7. Định nghĩa Item

## Thông tin Tenant (Readonly)

| No | Item | Type | Required | Format | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | Label | - | varchar(50) | central_db.mst_tenant.tenant_code |
| 2 | Tên công ty (企業名) | Label | - | varchar(255) | central_db.mst_tenant.company_name |
| 3 | Loại Tenant (テナントタイプ) | Label | - | tinyint (1: MOTO, 2: SAKI) | central_db.mst_tenant.tenant_type |

## Thống kê tài nguyên (Readonly)

| No | Item | Type | Required | Format | Source |
| --- | --- | --- | --- | --- | --- |
| 4 | Số người dùng hoạt động | Progress Bar / Label | - | Đếm số user active / Hạn mức | central_db.tenant_usage_stats.active_users / max_users |
| 5 | Dung lượng lưu trữ đã dùng | Progress Bar / Label | - | Dung lượng (GB) / Hạn mức (GB) | central_db.tenant_usage_stats.storage_used_bytes / max_storage_bytes |
| 6 | API Requests tháng hiện tại | Label | - | Số nguyên dương | central_db.tenant_usage_stats.monthly_api_requests |
| 7 | Lượt đăng nhập tháng hiện tại | Label | - | Số nguyên dương | central_db.tenant_usage_stats.monthly_login_count |
| 8 | Biểu đồ xu hướng | Chart Component | - | Trực quan hóa dữ liệu 6 tháng gần nhất | central_db.tenant_usage_history |

## Nút hành động (Footer Button)

| No | Item | Loại | Mô tả |
| --- | --- | --- | --- |
| 9 | Quay lại (戻る) | Button | Quay lại trang chi tiết Tenant tương ứng |

---

# 8. Validation

Màn hình này hoàn toàn ở chế độ chỉ đọc để hiển thị thông tin thống kê, không áp dụng quy tắc validation dữ liệu đầu vào.

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình dashboard sử dụng.

### Process

1. Lấy `:id` (Tenant ID) từ URL.
2. Gửi yêu cầu gọi API `GET /api/v1/admin/tenants/{id}/usage/`.
3. Server tập hợp và tính toán số liệu tài nguyên đang sử dụng của Tenant.
4. Render số liệu lên 4 khối Metric Card và hiển thị dữ liệu lịch sử lên Biểu đồ xu hướng.

---

## Quay lại

### Trigger

Click nút "戻る" hoặc Cancel.

### Process

1. Điều hướng quay trở lại màn hình chi tiết Tenant tương ứng (`PA-TEN-002` hoặc `PA-TEN-008`).

---

# 10. Mapping Database

## central_db.tenant_usage_stats (Bảng tổng hợp thống kê sử dụng của các Tenant - được cập nhật định kỳ qua Cronjob)

| Column | Type | Value / Mapping | Mô tả |
| --- | --- | --- | --- |
| id | bigint | *System Generated* | PK tự tăng |
| tenant_id | bigint | `mst_tenant.id` | FK liên kết tới Tenant |
| active_users | int | Đếm số tài khoản có `status = 1` | Số lượng tài khoản user đang hoạt động |
| max_users | int | Lấy từ gói đăng ký | Số lượng tài khoản user tối đa cho phép |
| storage_used_bytes | bigint | Tính sum dung lượng tệp tin đã upload | Tổng dung lượng lưu trữ tệp tin đã dùng (bytes) |
| max_storage_bytes | bigint | Lấy từ gói đăng ký | Tổng dung lượng lưu trữ tối đa cho phép (bytes) |
| monthly_api_requests | int | Đếm logs API của tháng hiện tại | Tổng số request gọi API trong tháng hiện tại |
| monthly_login_count | int | Đếm logs login của tháng hiện tại | Tổng số lượt đăng nhập trong tháng hiện tại |
| updated_at | datetime | *System Generated* | Thời điểm cập nhật số liệu gần nhất |

---

# 11. API Mapping

## Get Tenant Usage Details

```
GET /api/v1/admin/tenants/{id}/usage/
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
    "usage": {
      "active_users": 45,
      "max_users": 100,
      "storage_used_bytes": 13314398617,
      "max_storage_bytes": 53687091200,
      "monthly_api_requests": 15240,
      "monthly_login_count": 120,
      "updated_at": "2026-06-11 18:00:00"
    },
    "history": [
      { "month": "2025-12", "login_count": 95, "api_requests": 11200 },
      { "month": "2026-01", "login_count": 100, "api_requests": 12400 },
      { "month": "2026-02", "login_count": 88, "api_requests": 10500 },
      { "month": "2026-03", "login_count": 110, "api_requests": 13800 },
      { "month": "2026-04", "login_count": 105, "api_requests": 14200 },
      { "month": "2026-05", "login_count": 120, "api_requests": 15240 }
    ]
  }
}
```

---

# 12. Notification

- Khi `storage_used_bytes` vượt quá 90% hạn mức `max_storage_bytes`: Hệ thống gửi thông báo cảnh báo đến Tenant Admin và Platform SaaS Admin.
- Khi `active_users` đạt 95% hạn mức `max_users`: Hệ thống gửi thông báo khuyến nghị nâng cấp gói dịch vụ.

---

# 13. Approval Flow

Không áp dụng cho màn hình này.

---

# 14. Permission

| Action | Platform SaaS Admin | Platform SaaS Staff |
| --- | --- | --- |
| View Usage Stats | O | O |

---

# 15. Audit Log

Không áp dụng ghi Audit Log cho hành động xem Dashboard.

---

# 16. Error Handling

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) |
| --- | --- | --- |
| SYS001 | システムエラーが発生しました。 | Lỗi hệ thống. |

---

# 17. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix