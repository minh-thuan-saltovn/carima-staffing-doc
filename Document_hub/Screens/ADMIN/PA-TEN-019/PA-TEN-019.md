# テナントセキュリティ設定

System: Platform SaaS Admin
Menu: Tenant Management
メニュー: テナント管理
Sub menu: Empty
サブメニュー: Empty
Screen ID: PA-TEN-019
Screen (VI):
- Mỗi tenant setting được thời hạn password
- PWD độ mạnh
- IP Address
Giải thích tính năng: Empty
機能説明:
- パスワードの有効期限設定
- 複雑PW強制
- IP制限が必要
- テナント毎に設定可否/内容変更を行える必要がある
Thông tin hiển thị trên màn hình: Empty
画面表示情報: Empty
URL: Empty
Business Flow: Empty
システム: プラットフォーム管理
API List: PA-TEN-002-API-01 (MOTO Detail), PA-TEN-004-API-01 (MOTO Update), PA-TEN-008-API-01 (SAKI Detail), PA-TEN-010-API-01 (SAKI Update)
Combined: No
Screen Specs: No
Status: Not started

# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-TEN-019 |
| Tên màn hình | - Mỗi tenant setting được thời hạn password<br>- PWD độ mạnh<br>- IP Address |
| Tên tiếng Nhật | テナントセキュリティ設定 |
| Module | Tenant Management (テナント管理) |
| URL | /admin/tenants/{id}/security |
| Actor | Platform SaaS Admin |
| Priority | P1 |

---

# 2. Mục đích

Cho phép quản trị viên hệ thống (Platform SaaS Admin) thiết lập và tùy chỉnh các quy định bảo mật (Chính sách mật khẩu và Giới hạn IP truy cập) độc lập cho từng Tenant nhằm đảm bảo mức độ an toàn dữ liệu phù hợp với nhu cầu của từng khách hàng.

Sau khi lưu thành công:
- Cập nhật cấu hình bảo mật trực tiếp vào các trường tương ứng của Tenant trong database trung tâm (`central_db.mst_tenant`).
- Cấu hình bảo mật mới sẽ có hiệu lực ngay lập tức cho toàn bộ người dùng thuộc Tenant đó khi họ đăng nhập và tương tác với hệ thống.
- Ghi Audit Log hệ thống.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống SaaS Platform Admin.
- Có quyền quản trị tương ứng (`platform.tenant.tenant_security_setting.edit`).
- Tenant ID (hoặc `:id` trên URL) tồn tại trong hệ thống.

## Điều kiện sau

- Cập nhật thiết lập bảo mật Tenant thành công.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình | Action |
| --- | --- | --- |
| PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) | Click nút "セキュリティ設定" (Cấu hình bảo mật) |
| PA-TEN-008 | 派遣先テナント詳細 (SAKI Tenant Detail) | Click nút "セキュリティ設定" (Cấu hình bảo mật) |

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
|  [Breadcrumb: テナント管理 / テナント詳細 / セキュリティ設定]                                       |
|                                                                                                   |
|  ■ テナント情報 (Thông tin Tenant)                                                                 |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | テナントコード     | SAKI-0001                          |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | 企業名             | 株式会社サキケア                    |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|  | テナントタイプ     | 派遣先 (SAKI)                      |                                   |  |
|  +--------------------+------------------------------------+-----------------------------------+  |
|                                                                                                   |
|  ■ パスワードポリシー設定 (Chính sách mật khẩu)                                                     |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | パスワード有効期限 | ( ) 無期限    ( ) 30日    ( ) 60日    ( ) 90日    (*) カスタム              |  |
|  | (Password Expiry)  | [ 180     ] 日 (Nhập từ 1-365 ngày)                                     |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | 複雑性強制         | [ ON  ] (Bật/Tắt bắt buộc độ phức tạp)                                  |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  |  └ 最小文字数      | [ 8       ] 文字 (Độ dài từ 8-32 ký tự)                                 |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  |  └ 必須キャラクター| [x] 大文字必須 (Chữ hoa)            [x] 小文字必須 (Chữ thường)                 |  |
|  |    (Quy tắc ký tự) | [x] 数字必須 (Chữ số)               [ ] 記号必須 (Kí tự đặc biệt)               |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|                                                                                                   |
|  ■ IPアドレス制限設定 (Giới hạn địa chỉ IP)                                                        |
|  +--------------------+-------------------------------------------------------------------------+  |
|  | IPアドレス制限     | [ ON  ] (Bật/Tắt chặn IP truy cập)                                      |  |
|  +--------------------+-------------------------------------------------------------------------+  |
|  |  └ 許可IPリスト    | +-----------------------------------------+------------------+--------+ |  |
|  |    (Cho phép IP)   | | 許可IPアドレス / CIDR                   | メモ (Ghi chú)   | 操作   | |  |
|  |                    | +-----------------------------------------+------------------+--------+ |  |
|  |                    | | [ 192.168.1.0/24                      ] | [ Văn phòng chính] [ Xóa ] | |  |
|  |                    | +-----------------------------------------+------------------+--------+ |  |
|  |                    | | [ 203.0.113.5                         ] | [ Chi nhánh HCM  ] [ Xóa ] | |  |
|  |                    | +-----------------------------------------+------------------+--------+ |  |
|  |                    |                                                           [ IPを追加 ]  |  |
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

## Thiết lập chính sách mật khẩu (Password Policy Settings)

- **パスワード有効期限 (Password Expiry)**:
  - Cho phép chọn thời hạn hết hạn mật khẩu.
  - Khi chọn "カスタム" (Custom), ô nhập số ngày sẽ được enable và bắt buộc nhập số ngày. Ngược lại, nếu chọn các giá trị tĩnh thì ô nhập custom sẽ bị disabled và clear giá trị.
- **複雑性強制 (Force Complexity)**:
  - Sử dụng nút Toggle/Switch.
  - Khi bật `ON`, các trường "最小文字数" (Độ dài tối thiểu) và "必須キャラクター" (Quy tắc bắt buộc ký tự) sẽ hiển thị và bắt buộc nhập/chọn.
  - Khi tắt `OFF`, các trường cấu hình chi tiết trên sẽ ẩn/disabled.

## Thiết lập giới hạn IP (IP Address Restrictions)

- **IPアドレス制限 (IP Restriction Toggle)**:
  - Sử dụng nút Toggle/Switch.
  - Khi bật `ON`, bảng "許可IPリスト" (Danh sách IP cho phép) sẽ hiển thị để thao tác.
  - Khi tắt `OFF`, bảng danh sách IP sẽ bị ẩn.
- **許可IPリスト (Allowed IP List Table)**:
  - Cho phép thêm tối đa 50 dòng địa chỉ IP/CIDR.
  - Nút "IPを追加" (Thêm IP) dùng để append thêm 1 dòng trống mới vào cuối danh sách.
  - Mỗi dòng có nút "削除" (Xóa) để loại bỏ dòng IP tương ứng khỏi danh sách tạm.

## Nút 設定保存 (Lưu cấu hình)

- Disable khi đang thực hiện submit lên Server.
- Hiển thị trạng thái loading khi đang lưu.

---

# 7. Định nghĩa Item

## Thông tin Tenant (Readonly)

| No | Item | Type | Required | Format | Source |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | Label | - | VARCHAR(16) | mst_moto_company.company_id / mst_saki_company.company_id |
| 2 | Tên công ty (企業名) | Label | - | VARCHAR(100) | mst_moto_company.official_name_ja / mst_saki_company.official_name_ja |
| 3 | Loại Tenant (テナントタイプ) | Label | - | TINYINT(1) | Dựa vào tham số gọi màn hình (1: MOTO, 2: SAKI) |

## Thiết lập chính sách mật khẩu (パスワードポリシー設定)

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 4 | Thời hạn mật khẩu (パスワード有効期限) | Radio Button / Select | Yes | 0: 無期限 (Không giới hạn)<br>30: 30日<br>60: 60日<br>90: 90日<br>-1: カスタム (Custom) | password_expiry_days (Lưu số ngày tương ứng, nếu chọn Custom thì lấy từ ô nhập số ngày) |
| 5 | Số ngày tự chọn (カスタム日数) | Number Input | Yes (Khi chọn Custom) | Số nguyên dương (1 - 365) | password_expiry_days |
| 6 | Bắt buộc độ phức tạp (複雑性強制) | Toggle / Switch | Yes | TINYINT(1) (0: OFF, 1: ON). Mặc định: ON. | force_complex_password |
| 7 | Độ dài tối thiểu (最小文字数) | Number Input | Yes (Khi bật phức tạp) | Số nguyên dương (8 - 32). Mặc định: 8. | password_policy->min_length |
| 8 | Bắt buộc chữ hoa (大文字必須) | Checkbox | - | TINYINT(1) (0: Không, 1: Có). Mặc định: ON. | password_policy->require_uppercase |
| 9 | Bắt buộc chữ thường (小文字必須) | Checkbox | - | TINYINT(1) (0: Không, 1: Có). Mặc định: ON. | password_policy->require_lowercase |
| 10 | Bắt buộc chữ số (数字必須) | Checkbox | - | TINYINT(1) (0: Không, 1: Có). Mặc định: ON. | password_policy->require_numeric |
| 11 | Bắt buộc ký tự đặc biệt (記号必須) | Checkbox | - | TINYINT(1) (0: Không, 1: Có). Mặc định: OFF. | password_policy->require_special |

## Thiết lập giới hạn địa chỉ IP (IPアドレス制限設定)

| No | Item | Type | Required | Format / Options | DB Column |
| --- | --- | --- | --- | --- | --- |
| 12 | Giới hạn IP (IPアドレス制限) | Toggle / Switch | Yes | TINYINT(1) (0: OFF, 1: ON). Mặc định: OFF. | ip_restriction_enabled |
| 13 | Địa chỉ IP / CIDR | Text Input | Yes (Khi bật chặn IP) | Định dạng IPv4/IPv6 hoặc CIDR hợp lệ (VARCHAR(50)) | central_db.mst_tenant.allowed_ips |
| 14 | Ghi chú (メモ) | Text Input | - | VARCHAR(100) | central_db.mst_tenant.allowed_ips |
| 15 | Thêm IP (IPを追加) | Button | - | Click để append thêm hàng nhập mới | - |
| 16 | Xóa (削除) | Button | - | Loại bỏ hàng IP tương ứng | - |

## Các nút hành động (Footer Buttons)

| No | Item | Loại | Mô tả |
| --- | --- | --- | --- |
| 17 | Lưu cấu hình (設定保存) | Button | Gửi form cấu hình cập nhật lên hệ thống sau khi validate thành công |
| 18 | Hủy bỏ (キャンセル) | Button | Quay lại trang chi tiết Tenant tương ứng |

---

# 8. Validation

## Số ngày tự chọn (Custom Password Expiry Days)

| Rule | Message Code | Message |
| --- | --- | --- |
| Phải nhập giá trị | CMS-VAL-151 | 有効期限の日数を入力してください。 (Vui lòng nhập số ngày có hiệu lực) |
| Giá trị từ 1 đến 365 | CMS-VAL-152 | 有効期限は1日以上365日以内で入力してください。 (Thời hạn hiệu lực phải nhập trong khoảng từ 1 đến 365 ngày) |

## Độ dài tối thiểu mật khẩu (Password Min Length)

| Rule | Message Code | Message |
| --- | --- | --- |
| Phải nhập giá trị | CMS-VAL-153 | パスワードの最小文字数を入力してください。 (Vui lòng nhập độ dài tối thiểu của mật khẩu) |
| Giá trị từ 8 đến 32 | CMS-VAL-154 | 最小文字数は8文字以上32文字以内で入力してください。 (Độ dài tối thiểu phải từ 8 đến 32 ký tự) |

## Quy tắc ký tự bắt buộc (Character Complexity Rule)

| Rule | Message Code | Message |
| --- | --- | --- |
| Chọn ít nhất 1 loại ký tự | CMS-VAL-155 | 少なくとも1つの文字種別ルールを選択してください。 (Vui lòng chọn ít nhất một quy tắc loại ký tự) |

## Địa chỉ IP / CIDR

| Rule | Message Code | Message |
| --- | --- | --- |
| Phải nhập giá trị IP | CMS-VAL-156 | IPアドレスを入力してください。 (Vui lòng nhập địa chỉ IP) |
| Định dạng IP/CIDR hợp lệ | CMS-VAL-157 | 正しいIPアドレスまたはCIDR形式で入力してください。 (Vui lòng nhập đúng định dạng địa chỉ IP hoặc CIDR) |
| IP trùng lặp trong danh sách | CMS-VAL-158 | 重複したIPアドレスが入力されています。 (Địa chỉ IP nhập vào đã bị trùng lặp) |

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình cấu hình bảo mật.

### Process

1. Lấy `:id` (Tenant ID) và loại Tenant (MOTO hoặc SAKI) từ URL / ngữ cảnh điều hướng.
2. Gửi yêu cầu gọi API tương ứng:
   - Nếu là MOTO Tenant (派遣元): Gọi API `GET /api/v1/admin/moto-tenants/{id}/` (PA-TEN-002-API-01).
   - Nếu là SAKI Tenant (派遣先): Gọi API `GET /api/v1/admin/saki-tenants/{id}/` (PA-TEN-008-API-01).
3. Server trả về thông tin Tenant, bao gồm các cấu hình bảo mật lồng trong object `security_settings`.
4. Render thông tin Tenant lên đầu trang và gán các giá trị chính sách mật khẩu, cờ toggle, độ phức tạp và danh sách IP được phép tương ứng vào form.

---

## Change Expiry Type

### Trigger

Thay đổi Radio button "パスワード有効期限".

### Process

1. Nếu chọn "カスタム" (Custom):
   - Enable trường nhập liệu số ngày "カスタム日数".
2. Nếu chọn các giá trị tĩnh (無期限, 30日...):
   - Disable trường nhập liệu "カスタム日数".
   - Reset/Clear dữ liệu của trường "カスタム日数".

---

## Toggle Password Complexity

### Trigger

Thay đổi trạng thái Toggle "複雑性強制".

### Process

1. Nếu bật `ON`:
   - Hiển thị các trường cấu hình chi tiết: "最小文字数", các Checkbox bắt buộc ký tự.
2. Nếu tắt `OFF`:
   - Ẩn/Disabled toàn bộ các trường cấu hình chi tiết trên.

---

## Toggle IP Restriction

### Trigger

Thay đổi trạng thái Toggle "IPアドレス制限".

### Process

1. Nếu bật `ON`:
   - Hiển thị bảng "許可IPリスト".
   - Nếu danh sách IP trống, tự động thêm một hàng nhập đầu tiên.
2. Nếu tắt `OFF`:
   - Ẩn bảng "許可IPリスト".

---

## Add IP Row

### Trigger

Click nút "IPを追加".

### Process

1. Tạo một đối tượng hàng trống mới với cấu trúc: `{ ip_address: "", description: "" }`.
2. Append vào cuối mảng danh sách IP hiển thị trên giao diện.
3. Tối đa cho phép thêm 50 dòng. Khi đạt tới 50, disable nút "IPを追加".

---

## Remove IP Row

### Trigger

Click nút "削除" tại một dòng IP.

### Process

1. Loại bỏ hàng IP tương ứng khỏi mảng danh sách IP.
2. Nếu mảng còn dưới 50 dòng, enable lại nút "IPを追加".

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

## Save Security Settings (Lưu cấu hình)

### Trigger

Click 設定保存 (Lưu cấu hình).

### Process

1. Thực hiện validate logic form:
   - Nếu bật "カスタム" hạn mật khẩu: Kiểm tra số ngày có trống hay nằm ngoài dải 1-365.
   - Nếu bật "複雑性強制":
     - Kiểm tra độ dài tối thiểu có trống hay ngoài dải 8-32.
     - Kiểm tra xem đã chọn ít nhất một checkbox quy tắc ký tự bắt buộc hay chưa.
   - Nếu bật "IPアドレス制限":
     - Kiểm tra các hàng IP có bị để trống hay không hợp lệ (regex IPv4/IPv6/CIDR) không.
     - Kiểm tra xem có bị trùng lặp IP/CIDR giữa các hàng hay không.
2. Nếu phát hiện lỗi validate, dừng xử lý và hiển thị thông báo lỗi tương ứng trên giao diện tại từng trường lỗi.
3. Hiển thị popup xác nhận hành động: `CMS-VAL-85` (Target: "テナントセキュリティ設定").
4. Nếu người dùng chọn OK:
   - Gửi yêu cầu cập nhật lên Server:
     - Nếu là MOTO Tenant (派遣元): Gọi API `PUT /api/v1/admin/moto-tenants/{id}/` (PA-TEN-004-API-01).
     - Nếu là SAKI Tenant (派遣先): Gọi API `PUT /api/v1/admin/saki-tenants/{id}/` (PA-TEN-010-API-01).
     Request body sẽ truyền kèm dữ liệu cấu hình bảo mật trong trường `security_settings`.
   - Server cập nhật thông tin cài đặt bảo mật và danh sách IP được phép.
   - Ghi nhận Audit Log.
   - Hiển thị Toast thông báo thành công `CMS-VAL-79` ("テナントセキュリティ設定を更新しました。").
   - Điều hướng quay trở lại màn hình chi tiết Tenant tương ứng.

---

# 10. Mapping Database

## central_db.mst_tenant (Bảng lưu thông tin Tenant - bao gồm cấu hình bảo mật)

| Column | Type | Value / Mapping | Mô tả |
| --- | --- | --- | --- |
| id | BIGINT | *System Generated* | PK tự tăng của Tenant |
| tenant_code | VARCHAR(16) | `company_id` | Mã Tenant (Khớp với mst_moto_company / mst_saki_company) |
| company_name | VARCHAR(100) | `official_name_ja` | Tên công ty |
| tenant_type | TINYINT(1) | `tenant_type` | Loại Tenant (1: MOTO, 2: SAKI) |
| password_expiry_days | INT | `password_expiry_days` | Số ngày hết hạn mật khẩu (0: 無期限, Custom: số ngày nhập) |
| force_complex_password | TINYINT(1) | `force_complex_password` | Cờ bật bắt buộc mật khẩu phức tạp (0: OFF, 1: ON) |
| password_policy | JSON | `password_policy` | Cấu hình chi tiết độ phức tạp mật khẩu (min_length, require_uppercase, require_lowercase, require_numeric, require_special) |
| ip_restriction_enabled | TINYINT(1) | `ip_restriction_enabled` | Cờ bật/tắt chặn IP (0: OFF, 1: ON) |
| allowed_ips | JSON | `allowed_ips` | Danh sách IP/CIDR được phép truy cập (Ví dụ: `[{"ip_address": "192.168.1.0/24", "description": "Văn phòng"}]`) |

---

# 11. API Mapping

Màn hình này không sử dụng API riêng biệt mà tích hợp/lồng ghép dữ liệu cấu hình bảo mật vào các API quản lý chi tiết Tenant (MOTO/SAKI) sẵn có trong tài liệu dự án.

## 1. Lấy dữ liệu cấu hình bảo mật

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
    "security_settings": {
      "password_expiry_days": 30,
      "force_complex_password": 1,
      "password_policy": {
        "min_length": 8,
        "require_uppercase": 1,
        "require_lowercase": 1,
        "require_numeric": 1,
        "require_special": 0
      },
      "ip_restriction_enabled": 1,
      "allowed_ips": [
        {
          "id": 1,
          "ip_address": "192.168.1.0/24",
          "description": "Văn phòng chính"
        },
        {
          "id": 2,
          "ip_address": "203.0.113.5",
          "description": "Máy chủ chi nhánh"
        }
      ]
    }
  }
}
```

---

## 2. Cập nhật dữ liệu cấu hình bảo mật

### 2.1 Đối với Tenant MOTO (派遣元)
Sử dụng API PA-TEN-004-API-01-Update moto tenant:
```
PUT /api/v1/admin/moto-tenants/{id}/
```

### 2.2 Đối với Tenant SAKI (派遣先)
Sử dụng API PA-TEN-010-API-01-Update saki tenant:
```
PUT /api/v1/admin/saki-tenants/{id}/
```

### Request lồng ghép
```json
{
  "security_settings": {
    "password_expiry_days": 120,
    "force_complex_password": 1,
    "password_policy": {
      "min_length": 10,
      "require_uppercase": 1,
      "require_lowercase": 1,
      "require_numeric": 1,
      "require_special": 1
    },
    "ip_restriction_enabled": 1,
    "allowed_ips": [
      {
        "ip_address": "192.168.1.0/24",
        "description": "Văn phòng chính"
      },
      {
        "ip_address": "203.0.113.5",
        "description": "Máy chủ chi nhánh"
      }
    ]
  }
}
```

### Response
```json
{
  "status": "success",
  "message": "update_tenant_success"
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
| View Security Settings | O | O |
| Change Security Settings | O | X |

---

# 15. Audit Log

| Action | Log | Nội dung lưu |
| --- | --- | --- |
| Update Security Settings | Yes | [User] đã cập nhật thiết lập bảo mật cho Tenant [tenant_code]. Thời hạn mật khẩu: [password_expiry_days] ngày, Bắt buộc phức tạp: [force_complex_password], Chặn IP: [ip_restriction_enabled]. Số lượng IP cho phép: [ip_count]. |

---

# 16. Error Handling

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) |
| --- | --- | --- |
| VAL001 | 入力データが不正です。 | Dữ liệu đầu vào không hợp lệ. |
| VAL151 | 有効期限の日数を入力してください。 | Vui lòng nhập số ngày có hiệu lực. |
| VAL152 | 有効期限は1日以上365日以内で入力してください。 | Thời hạn hiệu lực phải nhập trong khoảng từ 1 đến 365 ngày. |
| VAL153 | パスワードの最小文字数を入力してください。 | Vui lòng nhập độ dài tối thiểu của mật khẩu. |
| VAL154 | 最小文字数は8文字以上32文字以内で入力してください。 | Độ dài tối thiểu phải từ 8 đến 32 ký tự. |
| VAL155 | 少なくとも1つの文字種別ルールを選択してください。 | Vui lòng chọn ít nhất một quy tắc loại ký tự. |
| VAL156 | IPアドレスを入力してください。 | Vui lòng nhập địa chỉ IP. |
| VAL157 | 正しいIPアドレスまたはCIDR形式で入力してください。 | Vui lòng nhập đúng định dạng địa chỉ IP hoặc CIDR. |
| VAL158 | 重複したIPアドレスが入力されています。 | Địa chỉ IP nhập vào đã bị trùng lặp. |
| SYS001 | システムエラーが発生しました。 | Lỗi hệ thống. |

---

# 17. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix
- Wireframe
