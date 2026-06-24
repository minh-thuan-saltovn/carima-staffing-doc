# 派遣元テナント一覧

System: Platform SaaS Admin
Menu: Tenant Management
メニュー: テナント管理
Screen ID: PA-TEN-001
Screen (VI): MOTO Tenant List
Giải thích tính năng: Danh sách tenant MOTO
機能説明: 派遣元テナント一覧を表示し、検索・絞込・出力・停止を行う。
Thông tin hiển thị trên màn hình: Company name, tenant code, contract plan, active staff count, active contract count, billing status, tenant status
画面表示情報: 会社名、テナントコード、契約プラン、稼働スタッフ数、稼働契約数、請求状況、テナントステータス
URL: /admin/moto-tenants
システム: プラットフォーム管理
API List: PA-TEN-001-API-01-Moto tenant list (https://www.notion.so/PA-TEN-001-API-01-Moto-tenant-list-368f02c407dd8093acd3cd3bb1ed416b?pvs=21), PA-TEN-001-API-02-Export moto tenant list (https://www.notion.so/PA-TEN-001-API-02-Export-moto-tenant-list-368f02c407dd8061b76dd671c908064a?pvs=21), ST-HOME-001-API-01-Staff Dashboard (https://www.notion.so/ST-HOME-001-API-01-Staff-Dashboard-36bf02c407dd8085b684c17da6bf2677?pvs=21)
Combined: No
Screen Specs: No
Status: In progress

# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | PA-TEN-001 |
| Tên màn hình | Danh sách Tenant MOTO |
| Tên tiếng Nhật | 派遣元テナント一覧 |
| Module | Tenant Management (テナント管理) |
| Chức năng | Hiển thị danh sách các tenant (doanh nghiệp phái cử MOTO), hỗ trợ tìm kiếm lọc, xuất dữ liệu và tạm ngưng/khôi phục hoạt động của tenant |
| Actor | Platform SaaS Admin |
| URL | /admin/moto-tenants |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# 2. Mục đích màn hình

Cho phép quản trị viên hệ thống (Platform SaaS Admin):

- Lọc tìm kiếm và hiển thị danh sách các doanh nghiệp phái cử MOTO (Tenant) đã đăng ký sử dụng hệ thống.
- Theo dõi nhanh quy mô hoạt động của từng tenant qua: số lượng nhân viên đang hoạt động (Staff), số lượng hợp đồng phái cử đang chạy, gói cước dịch vụ và trạng thái hóa đơn thanh toán.
- Xuất danh sách thông tin tenant ra file CSV.
- Chuyển tiếp nhanh tới các màn hình Xem chi tiết (`PA-TEN-002`), Tạo mới tenant (`PA-TEN-003`), hoặc Chỉnh sửa thông tin tenant (`PA-TEN-004`).
- Tạm ngưng (`PA-TEN-005`) hoặc Khôi phục hoạt động (`PA-TEN-006`) của tenant trực tiếp từ danh sách.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập vào hệ thống quản trị SaaS Platform Admin.
- Tài khoản người dùng có quyền quản trị tương ứng (`tenant.view`).

## Điều kiện sau

- Hiển thị danh sách các MOTO Tenant theo bộ lọc tìm kiếm.

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình |
| --- | --- |
| PA-DB-001 | プラットフォームダッシュボード (Platform Dashboard) |
| PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) - Quay lại |
| PA-TEN-003 | 派遣元テナント作成 (Create MOTO Tenant) - Quay lại |
| PA-TEN-004 | 派遣元テナント編集 (MOTO Tenant Edit) - Quay lại |

---

## Màn hình đích

| Action | Screen ID | Tên màn hình |
| --- | --- | --- |
| Tạo mới tenant | PA-TEN-003 | 派遣元テナント作成 (Create MOTO Tenant) |
| Click Mã Tenant / Chi tiết | PA-TEN-002 | 派遣元テナント詳細 (MOTO Tenant Detail) |
| Chỉnh sửa thông tin | PA-TEN-004 | 派遣元テナント編集 (MOTO Tenant Edit) |

---

# 5. UI/UX Layout

![alt text](image-1.png)

---

## Nguyên tắc UI/UX

> [!NOTE]
> Giao diện màn hình này KHÔNG bao gồm thanh Menu điều hướng bên trái (Sidebar) và Header/Footer dùng chung của Platform SaaS Admin để đảm bảo độc lập thiết kế logic trang.

### **Search Area (Khu vực lọc tìm kiếm)**
- Nằm ở trên cùng khu vực nội dung chính, dạng panel mở rộng/thu gọn.
- Các điều kiện tìm kiếm sắp xếp theo dạng lưới gọn gàng.
- Cung cấp nút "Lọc" (Primary) và link "Reset bộ lọc" (Secondary).

### **Data Table (Bảng dữ liệu)**
- Hỗ trợ sắp xếp (Sort) tại tiêu đề các cột chính.
- Cột trạng thái tenant hiển thị dưới dạng các Badge màu sắc rõ ràng (ví dụ: `有効` (Hoạt động) - Xanh lá, `停止中` (Tạm ngưng) - Màu đỏ/Xám).
- Cột trạng thái thanh toán hiển thị Badge cảnh báo (ví dụ: `未払い` (Chưa thanh toán/Quá hạn) - Màu cam/Đỏ, `支払い済` (Đã thanh toán) - Màu xanh).
- Cột thao tác (Thao tác cuối cùng bên phải) gồm nút Xem chi tiết, Chỉnh sửa, và nút thay đổi trạng thái (Tạm ngưng/Khôi phục) tùy theo trạng thái hiện hành của tenant.

### **Action Button (Nút hành động)**
- **Tạo mới (新規テナント作成):** Màu đen (Primary), nằm ở góc trên bên phải màn hình.
- **Xuất dữ liệu (エクスポート):** Viền đen, nền trắng (Secondary), xuất file CSV dựa theo danh sách lọc.

---

# 6. Danh sách Item màn hình

## Khu vực tìm kiếm

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | Textbox | varchar(50) | No | Tìm kiếm tương đối/chính xác mã Tenant. Placeholder: "例: MOTO-0001" |
| 2 | Tên công ty (企業名) | Textbox | varchar(255) | No | Tìm kiếm tương đối tên doanh nghiệp phái cử MOTO. Placeholder: "企業名で検索" |
| 3 | Trạng thái (ステータス) | Dropdown | tinyint | No | Lọc theo trạng thái hoạt động (0: Tạm ngưng, 1: Hoạt động) |
| 4 | Gói hợp đồng (契約プラン) | Dropdown | tinyint | No | Lọc theo gói dịch vụ (1: Lite, 2: Standard, 3: Pro, 4: Enterprise) |
| 5 | Tìm kiếm (検索) | Button | - | - | Áp dụng bộ lọc tìm kiếm |
| 6 | Làm mới bộ lọc (リセット) | Button | - | - | Reset toàn bộ các bộ lọc đang chọn về mặc định |

## Các nút hành động (Action Buttons)

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 7 | Tạo mới tenant (新規MOTOテナント作成) | Button | - | - | Chuyển tiếp sang màn hình tạo tenant mới (`PA-TEN-003`) |
| 8 | Xuất dữ liệu (CSV出力) | Button | - | - | Kích hoạt xuất dữ liệu tenant ra file CSV (`PA-TEN-001-API-02`) |

---

# 7. Định nghĩa Data Table

## Table hiển thị

### **central_db.mst_tenant**

| STT | Item | DB Column | Type | Width | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Mã Tenant (テナントコード) | tenant_code | varchar | 120px | Hiển thị mã tenant (Link chuyển tiếp tới chi tiết `PA-TEN-002`) |
| 2 | Tên công ty (企業名) | company_name | varchar | 220px | Tên chính thức của công ty phái cử MOTO |
| 3 | Gói hợp đồng (契約プラン) | contract_plan | tinyint | 120px | Gói dịch vụ (Badge: Lite / Standard / Pro / Enterprise) |
| 4 | Số nhân viên hoạt động (稼働スタッフ数) | active_staff_count | int | 120px | Số lượng nhân viên đang hoạt động (Tính toán động từ Database) |
| 5 | Số hợp đồng hoạt động (稼働契約数) | active_contract_count | int | 120px | Số lượng hợp đồng phái cử đang có hiệu lực (Tính toán động) |
| 6 | Trạng thái thanh toán (請求状況) | billing_status | tinyint | 120px | Badge trạng thái thanh toán (未払い: Chưa thanh toán / 支払い済: Đã thanh toán) |
| 7 | Trạng thái (ステータス) | status | tinyint | 100px | Badge trạng thái hoạt động của tenant (有効: Hoạt động / 停止中: Tạm ngưng) |
| 8 | Thao tác (操作) | - | action | 180px | Gồm các nút liên kết nhanh: "詳細" (Xem chi tiết), "編集" (Chỉnh sửa), và nút "停止" (Tạm ngưng - nếu đang hoạt động) hoặc "復旧" (Khôi phục - nếu đang tạm ngưng) |

---

## Sorting

Cho phép sắp xếp (Sort) 2 chiều tăng/giảm dần tại các cột:

- Mã Tenant (`tenant_code`)
- Tên công ty (`company_name`)
- Gói hợp đồng (`contract_plan`)
- Số nhân viên hoạt động (`active_staff_count`)
- Số hợp đồng hoạt động (`active_contract_count`)
- Trạng thái thanh toán (`billing_status`)
- Trạng thái tenant (`status`)

---

## Paging

| Item | Value |
| --- | --- |
| Default | 20 |
| Options | 20, 50, 100 |
| Server Side | Yes |

---

# 8. Mapping Database

## Table sử dụng

### **central_db.mst_tenant**

Bảng cơ sở lưu trữ dữ liệu các Tenant đăng ký trên Platform (loại `tenant_type` = 1 cho MOTO Tenant):

| Column | Type | Description |
| --- | --- | --- |
| id | bigint | PK - ID tự tăng của Tenant |
| tenant_type | tinyint(1) | Loại tenant (1: MOTO, 2: SAKI) |
| tenant_code | varchar(50) | UQ - Mã định danh duy nhất của tenant (Hankaku, ví dụ: subdomain) |
| company_name | varchar(255) | Tên doanh nghiệp phái cử MOTO |
| domain | varchar(100) | UQ - Tên miền hoặc subdomain định danh (vật lý) |
| phone_number | varchar(20) | Số điện thoại liên lạc chính của tenant |
| contract_plan | tinyint(1) | Gói hợp đồng dịch vụ (1: Lite, 2: Standard, 3: Pro, 4: Enterprise) |
| status | tinyint(1) | Trạng thái hoạt động của tenant (0: Tạm ngưng, 1: Hoạt động) |
| created_at | datetime | Thời điểm đăng ký tenant |
| updated_at | datetime | Thời điểm cập nhật thông tin cuối cùng |

---

# 9. Validation

| Item | Rule | Message Code | Mô tả |
| --- | --- | --- | --- |
| Tenant Code | Chiều dài <= 50 ký tự | CMS-VAL-6 | Mã tenant tối đa 50 ký tự |

---

# 10. Event Definition

## **Initial Load (Tải danh sách)**

### **Trigger**
Người dùng truy cập vào URL `/admin/moto-tenants`.

### **Flow**
1. Gửi Request gọi API `GET /api/v1/admin/moto-tenants/` với các tham số phân trang mặc định (`page = 1`, `limit = 20`).
2. Nhận Response trả về từ Server:
    - Thành công: Render danh sách tenant lên Data Table và cập nhật tổng số trang.
    - Thất bại: Hiển thị Toast báo lỗi hệ thống (`CMS-VAL-99`).

---

## **Search (Tìm kiếm)**

### **Trigger**
Người dùng thay đổi bộ lọc tìm kiếm (nhấn nút 検索 hoặc đổi các bộ lọc dropdown).

### **Flow**
1. Gửi Request gọi API `GET /api/v1/admin/moto-tenants/` kèm theo các tham số bộ lọc.
2. Nhận Response trả về từ Server:
    - Thành công: Cập nhật lại Grid dữ liệu bảng tenant và thông tin phân trang.
    - Thất bại: Hiển thị Toast báo lỗi hệ thống (`CMS-VAL-99`).

---

## **Reset (Làm mới bộ lọc)**

### **Trigger**
Người dùng click nút "リセット".

### **Flow**
1. Xóa các điều kiện lọc đang nhập trên panel tìm kiếm về mặc định.
2. Tự động gọi lại API lấy danh sách mặc định ở trang 1.

---

## **Export CSV (Xuất dữ liệu)**

### **Trigger**
Người dùng click nút "CSV出力".

### **Flow**
1. Hệ thống mặc định xuất toàn bộ dữ liệu theo bộ lọc lọc hiện hành.
2. Kích hoạt gọi API `GET /api/v1/admin/moto-tenants/export/` kèm theo tham số lọc.
3. Server xử lý tạo file CSV và gửi trả về file stream.
4. Trình duyệt thực hiện download file tự động. Hiển thị Toast thông báo thành công `CMS-VAL-80` ("CSV file exported successfully.").

---

## **Suspend Tenant (Tạm ngưng hoạt động)**

### **Trigger**
Quản trị viên click chọn nút "停止" (Tạm ngưng) tại cột thao tác của một tenant đang hoạt động (`status = 1`).

### **Flow**
1. Hiển thị Popup/Dialog xác nhận hành động: `CMS-VAL-85` (với Target là "MOTOテナントを停止").
   - Lời thoại hiển thị: "MOTOテナントを停止を更新します。よろしいですか。" (Sẽ tiến hành tạm ngưng hoạt động của MOTO Tenant này. Bạn có chắc chắn không?).
2. Quản trị viên click Xác nhận:
   - Gửi yêu cầu gọi API `PATCH /api/v1/admin/moto-tenants/{id}/suspend`.
   - Server thực hiện cập nhật `status = 0` (Tạm ngưng) cho tenant trong cơ sở dữ liệu.
   - Nhận Response phản hồi:
     - Thành công: Hiển thị Toast thông báo thành công `CMS-VAL-79` ("MOTOテナントを更新しました。" / "Đã cập nhật MOTO Tenant thành công."). Tải lại danh sách để cập nhật trạng thái mới trên Grid.
     - Thất bại: Hiển thị Toast báo lỗi hệ thống (`CMS-VAL-99`) hoặc lỗi không có quyền (`CMS-VAL-95`).

---

## **Restore Tenant (Khôi phục hoạt động)**

### **Trigger**
Quản trị viên click chọn nút "復旧" (Khôi phục) tại cột thao tác của một tenant đang bị tạm ngưng (`status = 0`).

### **Flow**
1. Hiển thị Popup/Dialog xác nhận hành động: `CMS-VAL-85` (với Target là "MOTOテナントを復旧").
   - Lời thoại hiển thị: "MOTOテナントを復旧を更新します。よろしいですか。" (Sẽ tiến hành khôi phục hoạt động của MOTO Tenant này. Bạn có chắc chắn không?).
2. Quản trị viên click Xác nhận:
   - Gửi yêu cầu gọi API `PATCH /api/v1/admin/moto-tenants/{id}/restore`.
   - Server cập nhật `status = 1` (Hoạt động) cho tenant.
   - Nhận Response phản hồi:
     - Thành công: Hiển thị Toast thông báo thành công `CMS-VAL-79` ("MOTOテナントを更新しました。" / "Đã cập nhật MOTO Tenant thành công."). Tải lại danh sách trên Grid.
     - Thất bại: Hiển thị Toast báo lỗi hệ thống (`CMS-VAL-99`).

---

# 11. API Mapping

## **Get MOTO Tenant List**

### Endpoint
```
GET /api/v1/admin/moto-tenants/
```

### Request Parameters (Query String)
| Parameter | Type | Required | Mô tả |
| --- | --- | --- | --- |
| company_name | string | No | Tên doanh nghiệp MOTO |
| tenant_code | string | No | Mã Tenant |
| contract_plan | tinyint | No | Gói dịch vụ (1: Lite, 2: Standard, 3: Pro, 4: Enterprise) |
| status | tinyint | No | Trạng thái hoạt động của tenant (0: Tạm ngưng, 1: Hoạt động) |
| page | int | No | Trang hiện tại (Mặc định: 1) |
| limit | int | No | Số bản ghi trên trang (Mặc định: 20) |
| sort_column | string | No | Cột sắp xếp (Mặc định: created_at) |
| sort_direction | string | No | Hướng sắp xếp (asc/desc, Mặc định: desc) |

### Response (Success 200 OK)
```json
{
  "status": "success",
  "message": "get_moto_tenants_success",
  "data": {
    "items": [
      {
        "id": 1,
        "tenant_code": "MOTO-0001",
        "company_name": "MOTO Staffing Solutions",
        "domain": "moto-staffing.jf-estaffing.jp",
        "contract_plan": 4,
        "active_staff_count": 35,
        "active_contract_count": 10,
        "billing_status": 1,
        "status": 1,
        "created_at": "2026-04-02 00:00:00",
        "updated_at": "2026-05-10 00:00:00"
      }
    ],
    "total": 58,
    "current_page": 1,
    "last_page": 3,
    "per_page": 20
  }
}
```

---

## **Export MOTO Tenant List**

### Endpoint
```
GET /api/v1/admin/moto-tenants/export/
```

### Request Parameters (Query String)
*(Các tham số lọc giống như API lấy danh sách để hỗ trợ export dữ liệu theo điều kiện lọc)*

### Response (Success 200 OK)
- Trả về tệp tin dưới dạng File Stream (`text/csv`).

---

## **Suspend MOTO Tenant**

### Endpoint
```
PATCH /api/v1/admin/moto-tenants/{id}/suspend
```

### Response (Success 200 OK)
```json
{
  "status": "success",
  "message": "suspend_tenant_success"
}
```

---

## **Restore MOTO Tenant**

### Endpoint
```
PATCH /api/v1/admin/moto-tenants/{id}/restore
```

### Response (Success 200 OK)
```json
{
  "status": "success",
  "message": "restore_tenant_success"
}
```

---

# 12. Permission

| Action | Platform SaaS Admin | Platform SaaS Staff |
| --- | --- | --- |
| View (Xem danh sách) | O | O |
| Export (Xuất dữ liệu) | O | O |
| Suspend / Restore (Tạm ngưng / Khôi phục) | O | X |

---

# 13. Message Definition

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) | Loại hiển thị |
| --- | --- | --- | --- |
| **CMS-VAL-6** | {0}は{1}文字以内で入力してください。 | Vui lòng nhập {0} trong vòng {1} ký tự trở xuống. | Inline Validation |
| **CMS-VAL-79** | {Screen name}を更新しました。 | Đã cập nhật {Screen name}. | Toast Success |
| **CMS-VAL-80** | CSVファイルを出力しました。 | Đã xuất file CSV thành công. | Toast Success |
| **CMS-VAL-85** | {Target}を更新します。よろしいですか。 | Sẽ tiến hành cập nhật {Target}. Bạn có chắc chắn không? | Dialog Confirm |
| **CMS-VAL-95** | この機能・リソースへのアクセス権限がありません。 | Bạn không có quyền truy cập vào chức năng/tài nguyên này. | Toast Error |
| **CMS-VAL-99** | システムエラーが発生しました。管理者へお問い合わせください。 | Đã xảy ra lỗi hệ thống. Vui lòng liên hệ với người quản trị. | Toast Error / Popup |

---

# 14. Error Handling

| HTTP Code | Action | Message ID hiển thị |
| --- | --- | --- |
| 400 | Hiển thị thông báo dữ liệu không hợp lệ tại popup/toast. | CMS-VAL-93 |
| 401 | Xóa token tại LocalStorage và tự động chuyển hướng về trang Đăng nhập Platform (/admin/login). | CMS-VAL-94 |
| 403 | Chặn hành động và hiển thị Toast báo lỗi không có quyền truy cập. | CMS-VAL-95 |
| 404 | Hiển thị Toast thông báo dữ liệu không tồn tại. | CMS-VAL-96 |
| 422 | Hiển thị chi tiết lỗi validate nghiệp vụ từ Server tại từng ô nhập liệu tương ứng. | CMS-VAL-98 |
| 500 | Hiển thị Popup thông báo lỗi hệ thống nghiêm trọng. | CMS-VAL-99 |

---

# 15. Audit Log

| Action | Log | Nội dung lưu |
| --- | --- | --- |
| Search | No | - |
| Export CSV | Yes | [User] đã xuất file CSV danh sách MOTO Tenants. |
| Suspend Tenant | Yes | [User] đã tạm ngưng hoạt động của MOTO Tenant [tenant_code]. |
| Restore Tenant | Yes | [User] đã khôi phục hoạt động của MOTO Tenant [tenant_code]. |

---

# 16. Related Documents

- Business Flow Diagram
- ERD
- API Specification
- Role Matrix
- Wireframe
- NFR