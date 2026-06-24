# PA-TEN-001-API-02-Export moto tenant list

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Export moto tenant list |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.0 |
| Ngày tạo | 2026/06/17 |
| Ngày cập nhật | 2026/06/17 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Export moto tenant list dựa trên thiết kế và cấu trúc DB thực tế |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-001-API-02 |
| Tên API | Export moto tenant list |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-001 |
| Chức năng liên quan | Tenant Management (MOTO Tenant List - Export) |
| Mục đích API | Xuất danh sách tenant MOTO ra file CSV. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/admin/moto-tenants/export |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Có (Download CSV) |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để xuất danh sách tất cả hoặc danh sách đã lọc các MOTO Tenant (派遣元テナント) ra tệp tin định dạng CSV phục vụ cho việc lưu trữ, báo cáo của Platform SaaS Admin.

### Ngữ cảnh nghiệp vụ

API được gọi khi Platform SaaS Admin nhấn nút "Xuất file" (CSV) trên màn hình quản lý Tenant MOTO. Bộ lọc tìm kiếm hiện tại trên màn hình sẽ được gửi kèm để xuất đúng dữ liệu đang được lọc.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User nhấn nút Export CSV trên giao diện màn hình PA-TEN-001 |
| Actor | Platform SaaS Admin / Platform SaaS Staff |
| Trước khi gọi API | Đã đăng nhập vào hệ thống Platform Manager và có quyền `tenant.view` / `tenant.export` |
| Sau khi gọi API | Tải tệp tin CSV chứa danh sách Tenant MOTO về máy của Client |
| Thay đổi trạng thái liên quan | Không |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/admin/moto-tenants/export
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về khi có lỗi |

---

## 4.3 Path Parameters

Không áp dụng.

---

## 4.4 Query Parameters

*Các tham số lọc tương tự như API lấy danh sách để hỗ trợ xuất dữ liệu theo đúng bộ lọc trên màn hình.*

| Parameter | Type | Bắt buộc | Default | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| tenant_code | string | Không | null | Lọc theo mã Tenant (map vào `company_id` tìm kiếm tương đối) | MOTO-0001 |
| company_name | string | Không | null | Lọc theo tên công ty (tìm kiếm tương đối) | MOTO Staffing |
| status | number | Không | null | Lọc theo trạng thái hoạt động (0: Tạm ngưng, 1: Hoạt động) | 1 |
| plan_code | string | Không | null | Mã gói dịch vụ (LITE, STANDARD, PRO, ENTERPRISE) | STANDARD |
| sort_column | string | Không | created_at | Cột sắp xếp | created_at |
| sort_direction | string | Không | desc | Hướng sắp xếp (asc / desc) | desc |

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

### Response Headers

| Header Name | Giá trị | Mô tả |
| --- | --- | --- |
| Content-Type | text/csv; charset=UTF-8 | Định dạng file xuất ra là CSV |
| Content-Disposition | attachment; filename="moto_tenants_export_YYYYMMDD.csv" | Tên tệp tin tải về mặc định |

### Response Body

*Dữ liệu dạng CSV stream tải về trực tiếp.*

```csv
"ID Tenant","Mã Tenant","Tên Doanh Nghiệp","Schema Name","DB User","Gói Dịch Vụ","Ngày Ký Hợp Đồng","Số Nhân Viên","Số Hợp Đồng","Trạng Thái Thanh Toán","Trạng Thái","Ngày Tạo"
"01H2PJX7G3P681729X4R09Q871","MOTO-0001","MOTO Staffing Solutions","tenant_moto_0001","db_moto_user_0001","STANDARD","2026-04-02",35,10,"Đã thanh toán","Hoạt động","2026-04-02 00:00:00"
```

---

## 6.2 Định nghĩa các cột trong file CSV xuất ra

| Số thứ tự | Tên cột (Header CSV) | Cột tương ứng trong DB | Mô tả |
| --- | --- | --- | --- |
| 1 | ID Tenant | `platform.tenant_registry.tenant_id` | Khóa chính (ULID) |
| 2 | Mã Tenant | `platform.tenant_registry.company_id` | Mã công ty định danh |
| 3 | Tên Doanh Nghiệp | `platform.tenant_registry.company_name` | Tên doanh nghiệp MOTO |
| 4 | Schema Name | `platform.tenant_registry.schema_name` | Tên schema riêng của tenant |
| 5 | DB User | `platform.tenant_registry.db_user` | Postgres role riêng của tenant |
| 6 | Gói Dịch Vụ | `platform.tenant_registry.plan_code` | Tên gói cước đăng ký |
| 7 | Ngày Ký Hợp Đồng | `platform.tenant_registry.contracted_at` | Ngày bắt đầu ký dịch vụ |
| 8 | Số Nhân Viên | `platform.t_tenant_usage_metric.metric_value` | Số lượng nhân viên đang hoạt động (Lọc theo `target_year_month = :current_ym` và `metric_type = 'active_staff_count'`) |
| 9 | Số Hợp Đồng | `platform.t_tenant_usage_metric.metric_value` | Số hợp đồng phái cử có hiệu lực (Lọc theo `target_year_month = :current_ym` và `metric_type = 'active_contract_count'`) |
| 10 | Trạng Thái Thanh Toán | `platform.t_usage_billing.billing_status` | Trạng thái thanh toán phí dịch vụ (Lọc theo `target_year_month = :current_ym` và ánh xạ giá trị) |
| 11 | Trạng Thái | `platform.tenant_registry.status` | 0: Tạm ngưng, 1: Hoạt động |
| 12 | Ngày Tạo | `platform.tenant_registry.created_at` | Ngày tạo bản ghi hệ thống |

---

## 6.3 Error Response

*Trả về JSON dạng lỗi nếu validate bộ lọc thất bại hoặc không có quyền gọi API.*

### Validation Error

```
422 Unprocessable Entity
```

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed.",
    "details": [
      {
        "field": "status",
        "message": "Status must be 0 or 1."
      }
    ]
  }
}
```

### Unauthorized

```
401 Unauthorized
```

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication is required."
  }
}
```

---

# 7. Validation Rules

| No. | Field | Rule | Error Code | Error Message |
| --- | --- | --- | --- | --- |
| 1 | status | Không bắt buộc, giá trị phải là 0 hoặc 1 | CMS-VAL-41 | 選択されたステータスは正しくありません。 |
| 2 | plan_code | Không bắt buộc, phải thuộc danh sách: LITE, STANDARD, PRO, ENTERPRISE | CMS-VAL-41 | 選択されたプランコードは正しくありません。 |
| 3 | tenant_code | Không bắt buộc, tối đa 50 ký tự | CMS-VAL-6 | テナントコードは50文字以内で入力してください。 |
| 4 | company_name | Không bắt buộc, tối đa 255 ký tự | CMS-VAL-6 | 企業名は255文字以内で入力してください。 |
| 5 | sort_column | Phải thuộc: tenant_code, company_name, plan_code, status, created_at | CMS-VAL-41 | 選択されたソートカラムは正しくありません。 |
| 6 | sort_direction | Phải là asc hoặc desc | CMS-VAL-41 | 選択されたソート方向は正しくありません。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant type restriction | Chỉ xuất các tenant có `tenant_type = 'moto'` (MOTO Tenant). |
| BR-002 | Role authorization | Chỉ người dùng thuộc nhóm quản trị Platform (SaaS Admin / SaaS Staff) có quyền `tenant.view` và `tenant.export` mới được gọi API này. |
| BR-003 | Filter and Order application | Dữ liệu xuất ra file CSV bắt buộc phải được lọc và sắp xếp chính xác theo các tham số Query Parameters được truyền lên. |
| BR-004 | File formatting | Xuất file dưới dạng mã hóa UTF-8 kèm BOM (Byte Order Mark) để đảm bảo hiển thị đúng font tiếng Nhật/tiếng Việt khi mở bằng Microsoft Excel. |
| BR-005 | Billing Status mapping | Trạng thái thanh toán hiển thị trên CSV được ánh xạ từ `platform.t_usage_billing.billing_status`: Nếu là `2` (đã thu) thì hiển thị "Đã thanh toán" (支払い済), còn lại (`0`: nháp, `1`: chốt) hoặc không có bản ghi cho tháng hiện tại thì hiển thị "Chưa thanh toán" (未払い). |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Export |
| --- | --- | --- |
| Platform SaaS Admin | Có | Có |
| Platform SaaS Staff | Có | Có |
| Tenant Admin (MOTO/SAKI) | Không | Không |

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Parameter | Table | Column | Logic truy vấn SQL |
| --- | --- | --- | --- |
| tenant_code | platform.tenant_registry | company_id | WHERE company_id LIKE CONCAT('%', :tenant_code, '%') |
| company_name | platform.tenant_registry | company_name | WHERE company_name LIKE CONCAT('%', :company_name, '%') |
| status | platform.tenant_registry | status | WHERE status = :status |
| plan_code | platform.tenant_registry | plan_code | WHERE plan_code = :plan_code |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field (CSV Column) | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | ID Tenant | ID khóa chính (ULID) |
| platform.tenant_registry | company_id | Mã Tenant | Mã Tenant |
| platform.tenant_registry | company_name | Tên Doanh Nghiệp | Tên công ty phái cử |
| platform.tenant_registry | schema_name | Schema Name | Tên schema PostgreSQL của tenant |
| platform.tenant_registry | db_user | DB User | PostgreSQL role của tenant |
| platform.tenant_registry | plan_code | Gói Dịch Vụ | Gói dịch vụ hợp đồng |
| platform.tenant_registry | contracted_at | Ngày Ký Hợp Đồng | Ngày ký hợp đồng dịch vụ |
| platform.t_tenant_usage_metric | metric_value | Số Nhân Viên | Số lượng staff đang hoạt động. Lọc theo `target_year_month = :current_ym` và `metric_type = 'active_staff_count'` |
| platform.t_tenant_usage_metric | metric_value | Số Hợp Đồng | Số hợp đồng đang hoạt động. Lọc theo `target_year_month = :current_ym` và `metric_type = 'active_contract_count'` |
| platform.t_usage_billing | billing_status | Trạng Thái Thanh Toán | Ánh xạ từ `billing_status` (2 -> Đã thanh toán, ngược lại -> Chưa thanh toán). Lọc theo `target_year_month = :current_ym` |
| platform.tenant_registry | status | Trạng Thái | Trạng thái hoạt động (0: Tạm ngưng, 1: Hoạt động) |
| platform.tenant_registry | created_at | Ngày Tạo | Ngày đăng ký |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | Query & Export data | Ngoài transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Không áp dụng | Không áp dụng |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Không áp dụng | Không áp dụng | Không áp dụng | Không áp dụng |

---

# 13. Sequence / Processing Flow

```
1. Client gửi request: GET /api/v1/admin/moto-tenants/export kèm query parameters bộ lọc.
2. Middleware thực hiện kiểm tra Authentication và xác định JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập: Platform SaaS Admin/Staff có quyền "tenant.view" và "tenant.export".
4. Controller nhận request và thực hiện Validate các tham số Query Parameters.
   - Nếu lỗi: Trả về HTTP 422 Unprocessable Entity kèm chi tiết lỗi JSON.
5. Controller gọi Service để lấy dữ liệu xuất bản:
   - Truy vấn bảng `platform.tenant_registry` với điều kiện `tenant_type = 'moto'`.
   - Thực hiện LEFT JOIN với bảng `platform.t_tenant_usage_metric` trên trường `tenant_id` và `target_year_month = :current_ym` để lấy `active_staff_count` (khi `metric_type = 'active_staff_count'`) và `active_contract_count` (khi `metric_type = 'active_contract_count'`) thông qua kỹ thuật xoay dòng (pivot/conditional aggregation).
   - Thực hiện LEFT JOIN với bảng `platform.t_usage_billing` trên trường `tenant_id` và `target_year_month = :current_ym` để lấy `billing_status`.
   - Áp dụng các điều kiện lọc và sắp xếp.
6. Service tạo CSV Stream:
   - Ghi dòng Header của CSV.
   - Lặp qua từng bản ghi dữ liệu và ghi vào Stream (sử dụng chunking/cursor nếu dữ liệu lớn để tránh tràn bộ nhớ). Ánh xạ các giá trị hiển thị (VD: billing_status).
7. Controller thiết lập HTTP Headers (`Content-Type`, `Content-Disposition`) và trả về CSV Stream 200 OK.
```

---

# 14. Notification

| Hạng mục | Nội dung |
| --- | --- |
| Có cần notification không | Không |
| Thời điểm gửi | Không áp dụng |
| Đối tượng nhận | Không áp dụng |
| Kênh gửi | Không áp dụng |
| Template ID | Không áp dụng |

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | EXPORT |
| Bảng target | platform.tenant_registry |
| ID record target | Không áp dụng |
| Dữ liệu ghi log | Ghi nhận nhật ký xuất file danh sách Tenant MOTO vào hệ thống audit |
| Chính sách lưu trữ | Theo NFR |

---

# 16. Xử lý file

## 16.1 Upload

| Hạng mục | Nội dung |
| --- | --- |
| Có upload file không | Không |
| Max file size | Không áp dụng |
| Định dạng cho phép | Không áp dụng |
| Nơi lưu trữ | Không áp dụng |
| Virus Scan | Không áp dụng |
| Quy tắc đặt tên file | Không áp dụng |
| Bảng lưu metadata | Không áp dụng |

---

## 16.2 Download

| Hạng mục | Nội dung |
| --- | --- |
| Có download file không | Có |
| Permission Check | Bắt buộc |
| Phương thức download | Stream |
| Thời gian hết hạn | Không áp dụng |
| Audit Log | Bắt buộc |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token hợp lệ và phân quyền Platform SaaS Admin/Staff (`tenant.view` và `tenant.export`). |
| 2 | SQL Injection Prevention | Sử dụng Eloquent ORM / Query Builder với bindings tham số để ngăn chặn tấn công SQL injection qua bộ lọc. |
| 3 | Path Traversal / Parameter Tampering | Giới hạn danh sách các cột trong `sort_column` để tránh việc truyền tên cột bất hợp pháp phá hỏng câu query SQL. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 1.5 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Không áp dụng |
| Index cần thiết | Index các cột `tenant_type`, `status`, `plan_code` và `created_at` trong bảng `tenant_registry` |
| Cache cần thiết | Không |
| Bulk Processing | Có (Sử dụng Laravel lazy loading / cursor / chunking để streaming trực tiếp) |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Wireframe | PA-TEN-001 MOTO Tenant List |
| Screen Detail Design | PA-TEN-001 |
| API liên quan | PA-TEN-001-API-01 Moto tenant list |
| ERD | Carima-Staffing Data Dictionary / ERD |
| Permission Matrix | Platform Portal Permission Matrix |
| Validation Rule | Section 7. Validation Rules |
| NFR | Performance Requirement |

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
| 8 | Tenant isolation rule đã được định nghĩa | Done (Không áp dụng) |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done (Không áp dụng) |
| 11 | Status transition đã được định nghĩa | Done (Không áp dụng) |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
