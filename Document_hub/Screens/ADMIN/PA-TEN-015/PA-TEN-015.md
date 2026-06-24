# テナントブランド設定

System: Platform SaaS Admin
Menu: Tenant Management
メニュー: テナント管理
Screen ID: PA-TEN-015
Screen (VI): Tenant Branding Setting
Giải thích tính năng: Thiết lập logo, theme color và login background cho từng tenant.
機能説明: テナントブランド設定を管理する。
Thông tin hiển thị trên màn hình: Logo, theme color, login background
画面表示情報: ロゴ、テーマカラー、ログイン背景
URL: /admin/tenants/:id/branding
システム: プラットフォーム管理
API List: PA-TEN-015-API-01-Tenant Branding Setting (https://www.notion.so/PA-TEN-015-API-01-Tenant-Branding-Setting-368f02c407dd80a481bcd8591e80483a?pvs=21)
Combined: No
Screen Specs: No
Status: Not started

# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-TEN-015 |
| Tên màn hình | Cấu hình thương hiệu Tenant |
| Tên tiếng Nhật | テナントブランド設定 |
| Module | Tenant Management (テナント管理) |
| URL | /admin/tenants/{id}/branding |
| Actor | Platform SaaS Admin |
| Priority | P1 |

---

# 2. Mục đích

Cho phép quản trị viên hệ thống (Platform SaaS Admin) cá nhân hóa thương hiệu (Branding) cho một Tenant cụ thể bằng cách tải lên logo, định nghĩa màu sắc chủ đạo (Theme color) và hình nền trang đăng nhập.

Sau khi lưu thành công:
- Tải tệp tin ảnh lên máy chủ lưu trữ (S3/Cloud Storage) và cập nhật đường dẫn URL vào cơ sở dữ liệu `central_db.tenant_branding`.
- Giao diện đăng nhập và làm việc của người dùng thuộc Tenant tương ứng sẽ tự động cập nhật theo cấu hình mới.
- Ghi Audit Log hệ thống.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống SaaS Platform Admin.
- Có quyền quản trị tương ứng (`tenant.update`).
- Tenant ID (hoặc `:id` trên URL) tồn tại trong hệ thống.

## Điều kiện sau

- Cập nhật thông tin thương hiệu Tenant thành công.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình | Action |
| --- | --- | --- |
| PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) | Click nút "ブランド設定" |
| PA-TEN-008 | 派遣先テナント詳細 (SAKI Tenant Detail) | Click nút "ブランド設定" |

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
|  [Breadcrumb: テナント管理 / テナント詳細 / ブランド設定]                                         |
|                                                                                                   |
|  ■ テナント情報 (Thông tin Tenant)                                                                 |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | テナントコード     | MOTO-0001                          |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | 企業名             | 株式会社モトキャリア                |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | テナントタイプ     | 派遣元                             |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|                                                                                                   |
|  ■ ブランド設定 (Cấu hình Branding)                                                               |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | ロゴ画像           | [ Vùng kéo thả hoặc click để chọn file ]                                |  |
|  | (Logo Image)       | * Hỗ trợ PNG, JPEG. Tối đa 2MB.                                         |  |
|  |                    | +----------------------------------+                                    |  |
|  |                    | | (Ảnh Preview Logo hiện tại)      |  [ ロゴ画像を削除 ] (Xóa Logo)      |  |
|  |                    | +----------------------------------+                                    |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | テーマカラー *     | [#1E3A8A  ] [ Color Picker Button ]                                     |  |
|  | (Theme Color)      | * Định dạng mã HEX (#ffffff)                                             |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | ログイン背景画像   | [ Vùng kéo thả hoặc click để chọn file ]                                |  |
|  | (Login Background) | * Hỗ trợ PNG, JPEG. Tối đa 5MB.                                         |  |
|  |                    | +----------------------------------+                                    |  |
|  |                    | | (Ảnh Preview Background hiện tại)|  [ 背景画像を削除 ] (Xóa BG)        |  |
|  |                    | +----------------------------------+                                    |  |
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

## Thiết lập Brand

- **ロゴ画像 (Logo Image)**:
  - Hiển thị vùng kéo thả tệp tin (Drag and Drop Area).
  - Nếu đã có logo, hiển thị ảnh xem trước (Preview) kèm nút "ロゴ画像を削除" để xóa ảnh.
  - Phải hiển thị văn bản cảnh báo giới hạn dung lượng tối đa 2MB và các định dạng hỗ trợ (PNG, JPEG).
- **テーマカラー (Theme Color)**:
  - Gồm một ô nhập văn bản (Mã màu HEX dạng `#1E3A8A`) và nút mở bảng chọn màu (Color Picker). Thay đổi màu trên bảng chọn sẽ tự động điền mã HEX vào ô nhập và ngược lại.
- **ログイン背景画像 (Login Background)**:
  - Hiển thị vùng kéo thả tệp tin. Dung lượng tối đa 5MB.
  - Nếu đã cấu hình background, hiển thị preview tỷ lệ thu nhỏ kèm nút "背景画像を削除" để xóa ảnh.

## Nút 設定保存 (Lưu cấu hình)

- Disable khi đang thực hiện tải tệp lên server.
- Hiển thị trạng thái loading khi đang lưu.

---

# 7. Định nghĩa Item

## Thông tin Tenant (Readonly)

| No | Item | Type | Required | Format | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | Label | - | varchar(50) | central_db.mst_tenant.tenant_code |
| 2 | Tên công ty (企業名) | Label | - | varchar(255) | central_db.mst_tenant.company_name |
| 3 | Loại Tenant (テナントタイプ) | Label | - | tinyint (1: MOTO, 2: SAKI) | central_db.mst_tenant.tenant_type |

## Cấu hình Branding (ブランド設定)

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 4 | ロゴ画像 (Logo Image) | File Input | No | Tệp tin ảnh (.png, .jpeg, .jpg), Dung lượng <= 2MB | logo_url |
| 5 | Xóa logo (ロゴ画像を削除) | Button | - | Đánh dấu xóa logo hiện tại về null | - |
| 6 | テーマカラー (Theme Color) | Color Input / Text | Yes | Mã HEX (Ví dụ: #1E3A8A). Mặc định: #1E3A8A | theme_color |
| 7 | ログイン背景 (Login Background) | File Input | No | Tệp tin ảnh (.png, .jpeg, .jpg), Dung lượng <= 5MB | login_bg_url |
| 8 | Xóa ảnh nền (背景画像を削除) | Button | - | Đánh dấu xóa ảnh nền hiện tại về null | - |

## Các nút hành động (Footer Buttons)

| No | Item | Loại | Mô tả |
| --- | --- | --- | --- |
| 9 | Lưu cấu hình (設定保存) | Button | Kích hoạt gửi form cấu hình cập nhật lên hệ thống sau khi validate thành công |
| 10 | Hủy bỏ (キャンセル) | Button | Quay lại trang chi tiết Tenant tương ứng |

---

# 8. Validation

## Validate Logo

| Rule | Message Code | Message |
| --- | --- | --- |
| Định dạng ảnh hợp lệ | CMS-VAL-121 | ロゴ画像はPNGまたはJPEG形式でアップロードしてください。 (Vui lòng tải lên ảnh Logo định dạng PNG hoặc JPEG) |
| Giới hạn dung lượng 2MB | CMS-VAL-122 | ロゴ画像のサイズは2MB以下にしてください。 (Dung lượng ảnh Logo phải dưới 2MB) |

## Validate Theme Color

| Rule | Message Code | Message |
| --- | --- | --- |
| Định dạng mã HEX | CMS-VAL-125 | テーマカラーは正しいHEX形式（例: #1E3A8A）で入力してください。 (Vui lòng nhập màu chủ đạo theo đúng định dạng mã màu HEX) |

## Validate Login Background

| Rule | Message Code | Message |
| --- | --- | --- |
| Định dạng ảnh hợp lệ | CMS-VAL-123 | 背景画像はPNGまたはJPEG形式でアップロードしてください。 (Vui lòng tải lên ảnh nền định dạng PNG hoặc JPEG) |
| Giới hạn dung lượng 5MB | CMS-VAL-124 | 背景画像のサイズは5MB以下にしてください。 (Dung lượng ảnh nền phải dưới 5MB) |

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình cấu hình.

### Process

1. Lấy `:id` (Tenant ID) từ URL.
2. Gửi yêu cầu gọi API `GET /api/v1/admin/tenants/{id}/branding/`.
3. Server trả về trạng thái cấu hình thương hiệu hiện tại của Tenant.
4. Render thông tin Tenant lên đầu trang và thiết lập ảnh preview của Logo, mã màu Theme và ảnh preview Background tương ứng (nếu đã được cấu hình trước đó).

---

## Logo/Background Selection

### Trigger

Người dùng chọn hoặc kéo thả tệp tin ảnh vào vùng upload.

### Process

1. Validate kích thước và định dạng tệp tin phía Client.
2. Nếu không hợp lệ, hiển thị thông báo lỗi (ví dụ: `CMS-VAL-122` nếu Logo vượt quá 2MB) và xóa tệp tin khỏi bộ chọn.
3. Nếu hợp lệ, sinh mã Object URL tạm thời và hiển thị lên vùng Preview.

---

## Xóa Logo/Background

### Trigger

Người dùng click nút "ロゴ画像を削除" hoặc "背景画像を削除".

### Process

1. Xóa ảnh hiển thị tại vùng Preview.
2. Thiết lập cờ xóa tương ứng (`delete_logo_flg` hoặc `delete_login_bg_flg` thành 1) để chuẩn bị submit lên Server.

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

## Save Branding (Lưu cấu hình)

### Trigger

Click 設定保存 (Lưu cấu hình).

### Process

1. Thực hiện validate logic form (Kiểm tra định dạng mã HEX, kiểm tra tệp tin ảnh).
2. Nếu không hợp lệ, dừng xử lý và hiển thị thông báo lỗi.
3. Hiển thị popup xác nhận hành động: `CMS-VAL-85` (Target: "テナントブランド設定").
4. Nếu người dùng chọn OK:
   - Kích hoạt gọi API `PATCH /api/v1/admin/tenants/{id}/branding/` dưới dạng `multipart/form-data` chứa các file ảnh và cờ xóa.
   - Server tải tệp tin lên CDN/Cloud Storage, cập nhật đường dẫn vào bảng `central_db.tenant_branding`.
   - Ghi nhận Audit Log.
   - Hiển thị Toast thông báo thành công `CMS-VAL-79` ("テナントブランド設定を更新しました。").
   - Điều hướng quay trở lại màn hình chi tiết Tenant tương ứng.

---

# 10. Mapping Database

## central_db.tenant_branding (Bảng lưu thông tin cấu hình thương hiệu Tenant)

| Column | Type | Value / Mapping | Mô tả |
| --- | --- | --- | --- |
| id | bigint | *System Generated* | PK tự tăng |
| tenant_id | bigint | `mst_tenant.id` | FK liên kết tới Tenant |
| logo_url | varchar(255) | `logo_url` | Đường dẫn CDN/Storage của Logo |
| theme_color | varchar(7) | `theme_color` | Mã màu HEX chính của thương hiệu (Ví dụ: #1E3A8A) |
| login_bg_url | varchar(255) | `login_bg_url` | Đường dẫn CDN/Storage của ảnh nền login |

---

# 11. API Mapping

## Get Tenant Branding Setting

```
GET /api/v1/admin/tenants/{id}/branding/
```

### Response

```json
{
  "status": "success",
  "data": {
    "tenant_id": 12,
    "tenant_code": "MOTO-0001",
    "company_name": "株式会社モトキャリア",
    "tenant_type": 1,
    "branding": {
      "logo_url": "https://storage.carima.link/branding/tenant-12/logo.png",
      "theme_color": "#1E3A8A",
      "login_bg_url": "https://storage.carima.link/branding/tenant-12/login-bg.jpg"
    }
  }
}
```

---

## Update Tenant Branding Setting

```
PATCH /api/v1/admin/tenants/{id}/branding/
```

Request (Headers: `Content-Type: multipart/form-data`)

```
theme_color: "#1E3A8A"
logo_file: [Binary File] (Optional)
login_bg_file: [Binary File] (Optional)
delete_logo_flg: 0
delete_login_bg_flg: 0
```

Response

```json
{
  "status": "success",
  "message": "update_tenant_branding_success",
  "data": {
    "logo_url": "https://storage.carima.link/branding/tenant-12/logo.png",
    "theme_color": "#1E3A8A",
    "login_bg_url": "https://storage.carima.link/branding/tenant-12/login-bg.jpg"
  }
}
```

---

# 12. Notification

Không áp dụng cho màn hình này.

---

# 13. Approval Flow

Không áp dụng cho màn hình này. Thiết lập có hiệu lực trực tiếp sau khi Platform SaaS Admin cập nhật và lưu thành công.

---

# 14. Permission

| Action | Platform SaaS Admin | Platform SaaS Staff |
| --- | --- | --- |
| View Branding Settings | O | O |
| Change Branding Settings | O | X |

---

# 15. Audit Log

| Action | Log | Nội dung lưu |
| --- | --- | --- |
| Update Branding | Yes | [User] đã cập nhật thiết lập thương hiệu cho Tenant [tenant_code]. Các thay đổi: Logo (cũ: [old_logo], mới: [new_logo]), Theme (cũ: [old_theme], mới: [new_theme]). |

---

# 16. Error Handling

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) |
| --- | --- | --- |
| VAL001 | 入力データが不正です。 | Dữ liệu đầu vào không hợp lệ. |
| VAL121 | ロゴ画像はPNGまたはJPEG形式でアップロードしてください。 | Logo phải ở định dạng PNG hoặc JPEG. |
| VAL122 | ロゴ画像のサイズは2MB以下にしてください。 | Dung lượng ảnh Logo phải dưới 2MB. |
| VAL123 | 背景画像はPNGまたはJPEG形式でアップロードしてください。 | Ảnh nền phải ở định dạng PNG hoặc JPEG. |
| VAL124 | 背景画像のサイズは5MB以下にしてください。 | Dung lượng ảnh nền phải dưới 5MB. |
| VAL125 | テーマカラーは正しいHEX形式（例: #1E3A8A）で入力してください。 | Vui lòng nhập đúng định dạng mã HEX màu sắc. |
| SYS001 | システムエラーが発生しました。 | Lỗi hệ thống. |

---

# 17. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix
- Wireframe