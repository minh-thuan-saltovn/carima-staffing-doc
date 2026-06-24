# テナントドメイン設定

System: Platform SaaS Admin
Menu: Tenant Management
メニュー: テナント管理
Screen ID: PA-TEN-014
Screen (VI): Tenant Domain Setting
Giải thích tính năng: Mapping domain/subdomain cho tenant và kiểm tra trạng thái kết nối SSL/DNS.
機能説明: テナントドメイン設定を管理する。
Thông tin hiển thị trên màn hình: Domain, SSL status, DNS status
画面表示情報: ドメイン、SSL状態、DNS状態
URL: /admin/tenants/:id/domains
システム: プラットフォーム管理
API List: PA-TEN-014-API-01-Tenant Domain Setting (https://www.notion.so/PA-TEN-014-API-01-Tenant-Domain-Setting-368f02c407dd80e394dae2ca0249ccec?pvs=21)
Combined: No
Screen Specs: No
Status: Not started

# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-TEN-014 |
| Tên màn hình | Cấu hình Domain Tenant |
| Tên tiếng Nhật | テナントドメイン設定 |
| Module | Tenant Management (テナント管理) |
| URL | /admin/tenants/{id}/domains |
| Actor | Platform SaaS Admin |
| Priority | P1 |

---

# 2. Mục đích

Cho phép quản trị viên hệ thống (Platform SaaS Admin) cấu hình tên miền (Subdomain hoặc Custom Domain) cho một Tenant cụ thể, đồng thời kiểm tra và giám sát trạng thái cấu hình DNS, trạng thái chứng chỉ SSL để đảm bảo Tenant có thể truy cập hệ thống qua tên miền riêng một cách an toàn.

Sau khi lưu thành công:
- Cập nhật thông tin cấu hình domain trong database trung tâm (`platform.tenant_domain`).
- Thay đổi cấu hình này sẽ cập nhật cấu hình định tuyến (Routing) cho Tenant đó truy cập vào các Cổng thông tin (MOTO/SAKI Portal).
- Ghi Audit Log hệ thống.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống SaaS Platform Admin.
- Có quyền quản trị tương ứng (`tenant.update`).
- Tenant ID (hoặc `:id` trên URL) tồn tại trong hệ thống.

## Điều kiện sau

- Cập nhật thiết lập tên miền Tenant thành công.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình | Action |
| --- | --- | --- |
| PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) | Click nút "ドメイン設定" |
| PA-TEN-008 | 派遣先テナント詳細 (SAKI Tenant Detail) | Click nút "ドメイン設定" |

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
|  [Breadcrumb: テナント管理 / テナント詳細 / ドメイン設定]                                         |
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
|  ■ ドメイン設定 (Thiết lập Domain)                                                                |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | ドメインタイプ *   | (o) サブドメイン (Subdomain)    ( ) 独自ドメイン (Custom Domain)        |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | サブドメイン *     | [ saki-0001            ] .carima.link                                   |  |
|  | (Khi chọn Subdomain)                                                                         |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | 独自ドメイン *     | [ jobs.sakicare.com                                           ]         |  |
|  | (Khi chọn Custom)  |                                                                         |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | DNS設定案内        | 独自ドメインを使用する場合、DNSサーバーに以下のレコードを設定してください:  |  |
|  | (Thông tin DNS)    | 種類: CNAME                                                             |  |
|  |                    | ホスト名: jobs                                                          |  |
|  |                    | 値 / 宛先: ingress.carima.link                                          |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|                                                                                                   |
|  |                                     [ キャンセル ]   [ 設定保存 ]                           |  |
|  +---------------------------------------------------------------------------------------------+  |
+---------------------------------------------------------------------------------------------------+
```

---

# 6. Quy tắc UI/UX

## Thông tin Tenant (Readonly Area)

- Mã Tenant, Tên công ty và Loại Tenant được hiển thị ở dạng chỉ đọc đầu trang để quản trị viên dễ dàng đối chiếu.

## Thiết lập Domain

- **ドメインタイプ (Domain Type)**: Mặc định chọn "サブドメイン" (Subdomain).
- **Trường nhập liệu động**:
  - Khi chọn "サブドメイン", trường "独自ドメイン" sẽ bị ẩn/disable. Quản trị viên nhập phần tiền tố (Subdomain prefix). Phần đuôi `.carima.link` được hiển thị tĩnh cố định bên cạnh ô nhập liệu.
  - Khi chọn "独自ドメイン", trường "サブドメイン" sẽ bị ẩn/disable. Quản trị viên nhập đầy đủ FQDN (Fully Qualified Domain Name) của tên miền riêng (ví dụ: `jobs.sakicare.com`).
- **DNS設定案内 (DNS Guide Area)**: Chỉ hiển thị khi chọn loại tên miền là "独自ドメイン". Chứa thông tin bản ghi DNS cần thiết mà khách hàng (Tenant) phải cấu hình trên hệ thống quản lý tên miền của họ.

## Trạng thái kết nối (Connection Status Area)

- **DNS接続状態 (DNS Status)**: Hiển thị trạng thái phân giải DNS của domain hiện tại:
  - `未検証` (Màu xám): Chưa thực hiện xác minh DNS.
  - `有効` (Màu xanh lá): Tên miền trỏ đúng địa chỉ IP/CNAME hệ thống.
  - `検証失敗` (Màu đỏ): Tên miền không trỏ đúng cấu hình.
- **SSL証明書状態 (SSL Status)**: Hiển thị trạng thái chứng chỉ SSL:
  - `未適用` (Màu xám): Chưa đăng ký/cấp phát SSL.
  - `適用中` (Màu xanh lá): Chứng chỉ SSL hợp lệ và đang hoạt động.
  - `エラー` (Màu đỏ): Lỗi sinh/cấp phát chứng chỉ SSL hoặc chứng chỉ hết hạn.

## Nút 設定保存 (Lưu cấu hình)

- Disable khi đang thực hiện submit lên Server.
- Hiển thị trạng thái loading khi đang lưu.

---

# 7. Định nghĩa Item

## Thông tin Tenant (Readonly)

| No | Item | Type | Required | Format | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | Label | - | varchar(20) | platform.tenant_registry.company_id |
| 2 | Tên công ty (企業名) | Label | - | varchar(100) | platform.tenant_registry.company_name |
| 3 | Loại Tenant (テナントタイプ) | Label | - | varchar(8) ('saki' hoặc 'moto') | platform.tenant_registry.tenant_type |

## Thiết lập Domain (ドメイン設定)

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 4 | Loại tên miền (ドメインタイプ) | Radio Button | Yes | 1: サブドメイン (Subdomain)<br>2: 独自ドメイン (Custom Domain) | domain_type |
| 5 | Tên Subdomain (サブドメイン) | Text Input | Yes (khi chọn Subdomain) | Bắt đầu bằng chữ cái, chỉ chứa chữ thường, số, dấu gạch ngang. Giới hạn 2-63 ký tự. | domain_name (lưu chuỗi đầy đủ bao gồm phần hậu tố) |
| 6 | Tên miền riêng (独自ドメイン) | Text Input | Yes (khi chọn Custom Domain) | Định dạng Domain hợp lệ (FQDN). Ví dụ: jobs.sakicare.com. | domain_name |
| 7 | Hướng dẫn DNS (DNS設定案内) | Block Text | Readonly | Hiển thị thông tin bản ghi CNAME hướng dẫn cấu hình DNS. | - |

## Trạng thái kết nối (接続ステータス)

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 8 | DNS接続状態 | Badge | Readonly | 0: 未検証 (Grey)<br>1: 有効 (Green)<br>2: 検証失敗 (Red) | dns_status |
| 9 | SSL証明書状態 | Badge | Readonly | 0: 未適用 (Grey)<br>1: 適用中 (Green)<br>2: エラー (Red) | ssl_status |

## Các nút hành động (Footer Buttons)

| No | Item | Loại | Mô tả |
| --- | --- | --- | --- |
| 10 | Lưu cấu hình (設定保存) | Button | Kích hoạt gửi form cấu hình cập nhật lên hệ thống sau khi validate thành công |
| 11 | Hủy bỏ (キャンセル) | Button | Quay lại trang chi tiết Tenant tương ứng |

---

# 8. Validation

## Định dạng Subdomain

| Rule | Message Code | Message |
| --- | --- | --- |
| Định dạng ký tự hợp lệ | CMS-VAL-111 | サブドメインは半角英数字とハイフン（-）のみ使用できます。 (Subdomain chỉ sử dụng chữ cái, chữ số nửa chiều và dấu gạch ngang) |
| Độ dài tối thiểu 2 ký tự | CMS-VAL-112 | サブドメインは2文字以上で入力してください。 (Vui lòng nhập subdomain từ 2 ký tự trở lên) |

## Định dạng Custom Domain

| Rule | Message Code | Message |
| --- | --- | --- |
| Định dạng Domain hợp lệ | CMS-VAL-113 | 正しいドメイン形式で入力してください。 (Vui lòng nhập đúng định dạng tên miền) |

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình cấu hình.

### Process

1. Lấy `:id` (Tenant ID) và loại Tenant (MOTO hoặc SAKI) từ URL / ngữ cảnh điều hướng.
2. Gửi yêu cầu gọi API tương ứng:
   - Nếu là MOTO Tenant (派遣元): Gọi API `GET /api/v1/admin/moto-tenants/{id}/` (PA-TEN-002-API-01).
   - Nếu là SAKI Tenant (派遣先): Gọi API `GET /api/v1/admin/saki-tenants/{id}/` (PA-TEN-008-API-01).
3. Server trả về thông tin Tenant, bao gồm các cấu hình tên miền lồng trong object `domain_settings`.
4. Render thông tin Tenant lên đầu trang và thiết lập loại tên miền, giá trị tên miền và các Badge trạng thái kết nối tương ứng.

---

## Change Domain Type

### Trigger

Thay đổi Radio button "ドメインタイプ".

### Process

1. Nếu chuyển sang "サブドメイン":
   - Hiển thị trường nhập liệu Subdomain, ẩn trường nhập liệu Custom Domain.
   - Ẩn phần hướng dẫn cấu hình DNS.
2. Nếu chuyển sang "独自ドメイン":
   - Hiển thị trường nhập liệu Custom Domain, ẩn trường nhập liệu Subdomain.
   - Hiển thị phần hướng dẫn cấu hình DNS.

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

---

## Save Domain (Lưu cấu hình)

### Trigger

Click 設定保存 (Lưu cấu hình).

### Process

1. Thực hiện validate logic form (Kiểm tra định dạng domain nhập vào).
2. Nếu không hợp lệ, dừng xử lý và hiển thị thông báo lỗi tương ứng.
3. Hiển thị popup xác nhận hành động: `CMS-VAL-85` (Target: "テナントドメイン設定").
4. Nếu người dùng chọn OK:
   - Kích hoạt gọi API `PATCH /api/v1/admin/tenants/{id}/domains/`.
   - Server cập nhật thông tin cấu hình tên miền trong bảng `platform.tenant_domain` và đồng bộ hóa với hệ thống Router/Reverse Proxy.
   - Ghi nhận Audit Log.
   - Hiển thị Toast thông báo thành công `CMS-VAL-79` ("テナントドメインを設定しました。").
   - Điều hướng quay trở lại màn hình chi tiết Tenant tương ứng.

---

# 10. Mapping Database

## platform.tenant_domain (Bảng cấu hình tên miền của Tenant)

| Column | Type | Value / Mapping | Mô tả |
| --- | --- | --- | --- |
| id | bigint | *System Generated* | PK tự tăng |
| tenant_id | char(26) | `tenant_registry.tenant_id` | FK liên kết tới Tenant được cấu hình (ULID) |
| domain_type | smallint | `domain_type` | Loại tên miền (1: Subdomain, 2: Custom Domain) |
| domain_name | varchar(255) | `domain_name` | Tên miền đầy đủ (FQDN) |
| dns_status | smallint | `dns_status` | Trạng thái DNS (0: 未検証, 1: 有効, 2: 検証失敗) |
| ssl_status | smallint | `ssl_status` | Trạng thái SSL (0: 未適用, 1: 適用中, 2: エラー) |
| last_checked_at | timestamptz | `last_checked_at` | Thời gian kiểm tra DNS/SSL gần nhất |

---

# 11. API Mapping

Màn hình này không sử dụng API GET riêng biệt mà tích hợp/lồng ghép dữ liệu cấu hình tên miền vào các API quản lý chi tiết Tenant (MOTO/SAKI) sẵn có trong tài liệu dự án.

## 1. Lấy dữ liệu cấu hình tên miền

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
  "data": {
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "company_id": "SAKI-0001",
    "company_name": "株式会社サキケア",
    "tenant_type": "saki",
    "status": 1,
    "domain_settings": {
      "domain_type": 1,
      "domain_name": "saki-0001.carima.link",
      "dns_status": 1,
      "ssl_status": 1,
      "last_checked_at": "2026-06-11T18:00:00+09:00"
    }
  }
}
```

---

## 2. Cập nhật dữ liệu cấu hình tên miền

Sử dụng API PA-TEN-014-API-01-Tenant Domain Setting:
```
PATCH /api/v1/admin/tenants/{id}/domains/
```

### Request
```json
{
  "domain_type": 2,
  "domain_name": "jobs.sakicare.com"
}
```

### Response
```json
{
  "data": {
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "domain_settings": {
      "domain_type": 2,
      "domain_name": "jobs.sakicare.com",
      "dns_status": 0,
      "ssl_status": 0
    }
  }
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
| View Domain Settings | O | O |
| Change Domain Settings | O | X |

---

# 15. Audit Log

| Action | Log | Nội dung lưu |
| --- | --- | --- |
| Update Domain | Yes | [User] đã cập nhật thiết lập tên miền cho Tenant [company_id]. Domain mới: [domain_name] (Loại: [domain_type]). |

---

# 16. Error Handling

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) |
| --- | --- | --- |
| VAL001 | 入力データが不正です。 | Dữ liệu đầu vào không hợp lệ. |
| VAL111 | サブドメインは半角英数字とハイフン（-）のみ sử dụngできます。 | Subdomain chỉ sử dụng chữ cái, chữ số nửa chiều và dấu gạch ngang. |
| VAL112 | サブドメインは2文字以上で入力してください。 | Vui lòng nhập subdomain từ 2 ký tự trở lên. |
| VAL113 | 正しいドメイン形式で入力してください。 | Vui lòng nhập đúng định dạng tên miền. |
| SYS001 | システムエラーが発生しました。 | Lỗi hệ thống. |

---

# 17. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix
- Wireframe