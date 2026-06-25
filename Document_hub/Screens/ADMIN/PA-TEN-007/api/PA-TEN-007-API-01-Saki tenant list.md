# PA-TEN-007-API-01-Saki tenant list

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Saki tenant list |
| Description | Lấy danh sách tenant SAKI, hỗ trợ tìm kiếm/lọc. |
| Endpoint | `/api/v1/admin/saki-tenants` |
| Menu | Tenant Management |
| Method | GET |
| Related Tables | `platform.tenant_registry` |
| Screen list | 派遣先テナント一覧 (PA-TEN-007) — SAKI Tenant List |
| System | Platform SaaS Admin |
| 説明 | SAKIテナント一覧を取得し、検索・絞り込みに利用する。 |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Saki tenant list |
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
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Saki tenant list theo template và cấu trúc DB thực tế |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-007-API-01 |
| Tên API | Saki tenant list |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-007 |
| Chức năng liên quan | Tenant Management (SAKI Tenant List) |
| Mục đích API | Lấy danh sách tenant SAKI, hỗ trợ tìm kiếm, lọc và phân trang. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/admin/saki-tenants |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để lấy danh sách tất cả các SAKI Tenant (派遣先テナント) trong hệ thống phục vụ cho việc hiển thị, tìm kiếm, lọc và phân trang tại màn hình quản lý Tenant SAKI của Platform SaaS Admin.

### Ngữ cảnh nghiệp vụ

API được gọi khi Platform SaaS Admin truy cập vào màn hình quản lý Tenant SAKI hoặc thực hiện thao tác lọc/tìm kiếm/chuyển trang trên màn hình này.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User truy cập màn hình hoặc thay đổi bộ lọc, chuyển trang |
| Actor | Platform SaaS Admin / Platform SaaS Staff |
| Trước khi gọi API | Đã đăng nhập vào hệ thống Platform Manager và có quyền `tenant.view` |
| Sau khi gọi API | Hiển thị danh sách tenant SAKI lên bảng dữ liệu cùng thông tin phân trang |
| Thay đổi trạng thái liên quan | Không |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/admin/saki-tenants
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

Không áp dụng.

---

## 4.4 Query Parameters

| Parameter | Type | Bắt buộc | Default | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| page | number | Không | 1 | Số trang | 1 |
| limit | number | Không | 20 | Số record mỗi trang | 20 |
| tenant_code | string | Không | null | Lọc theo mã Tenant (company_id) tìm kiếm tương đối | SAKI-0001 |
| company_name | string | Không | null | Lọc theo tên công ty (tìm kiếm tương đối) | SAKI Partner |
| status | number | Không | null | Lọc theo trạng thái hoạt động (0: 停止中 - Tạm ngưng, 1: 有効 - Hoạt động) | 1 |
| plan_code | string | Không | null | Mã gói dịch vụ (LITE, STANDARD, PRO, ENTERPRISE) | STANDARD |
| sort_column | string | Không | created_at | Cột sắp xếp | created_at |
| sort_direction | string | Không | desc | Hướng sắp xếp (asc / desc) | desc |

---

# 5. Request Body

## 5.1 Request Body Schema

Không áp dụng (Phương thức GET không sử dụng Request Body).

---

## 5.2 Định nghĩa field request

Không áp dụng.

---

# 6. Response Definition

## 6.1 Success Response

### HTTP Status

```
200 OK
```

### Response Body

```json
{
  "data": [
    {
      "tenant_id": "01H2PJX7G3P681729X4R09Q872",
      "company_id": "SAKI-0001",
      "company_name": "SAKI Partner Factory",
      "schema_name": "tenant_saki_0001",
      "db_user": "db_saki_user_0001",
      "plan_code": "ENTERPRISE",
      "contracted_at": "2026-04-01",
      "connected_moto_count": 5,
      "active_contract_count": 12,
      "approval_user_count": 8,
      "billing_status": 1,
      "status": 1,
      "created_at": "2026-04-01T00:00:00+09:00",
      "updated_at": "2026-05-08T00:00:00+09:00"
    }
  ],
  "links": {
    "first": "http://example.com/api/v1/admin/saki-tenants?page=1",
    "last": "http://example.com/api/v1/admin/saki-tenants?page=4",
    "prev": null,
    "next": "http://example.com/api/v1/admin/saki-tenants?page=2"
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 4,
    "path": "http://example.com/api/v1/admin/saki-tenants",
    "per_page": 20,
    "to": 20,
    "total": 62
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data[] | array | Danh sách SAKI Tenants |
| data[].tenant_id | string | ID khóa chính (ULID) của tenant trong `tenant_registry` |
| data[].company_id | string | Mã công ty (business key) tương ứng tenant này |
| data[].company_name | string | Tên doanh nghiệp phái cử SAKI |
| data[].schema_name | string | Tên schema PostgreSQL riêng của tenant |
| data[].db_user | string | PostgreSQL role riêng của tenant |
| data[].plan_code | string | Gói dịch vụ hợp đồng (LITE, STANDARD, PRO, ENTERPRISE) |
| data[].contracted_at | string | Ngày ký hợp đồng dịch vụ (YYYY-MM-DD) |
| data[].connected_moto_count | int | Số lượng doanh nghiệp MOTO đang kết nối |
| data[].active_contract_count | int | Số lượng hợp đồng phái cử đang có hiệu lực |
| data[].approval_user_count | int | Số lượng tài khoản người dùng có quyền phê duyệt |
| data[].billing_status | tinyint | Trạng thái thanh toán (0: Chưa thanh toán, 1: Đã thanh toán) |
| data[].status | tinyint | Trạng thái hoạt động của tenant (0: Tạm ngưng, 1: Hoạt động) |
| data[].created_at | datetime | Thời gian tạo |
| data[].updated_at | datetime | Thời gian cập nhật |
| links | object | Các link điều hướng phân trang |
| links.first | string | Link đến trang đầu tiên |
| links.last | string | Link đến trang cuối cùng |
| links.prev | string | Link đến trang trước đó (null nếu đang ở trang 1) |
| links.next | string | Link đến trang tiếp theo (null nếu đang ở trang cuối) |
| meta | object | Metadata phân trang hệ thống |
| meta.current_page | number | Trang hiện tại |
| meta.from | number | Vị trí bản ghi bắt đầu của trang |
| meta.last_page | number | Trang cuối cùng |
| meta.path | string | Đường dẫn API |
| meta.per_page | number | Số lượng bản ghi trên một trang |
| meta.to | number | Vị trí bản ghi kết thúc của trang |
| meta.total | number | Tổng số bản ghi thỏa mãn bộ lọc |

---
### 6.3 Error Response - Validation Error

```
422 Unprocessable Entity
```

### 6.4 Error Response - Invalid Credentials

```
401 Unauthorized
```

### 6.5 Error Response - Account Locked

```
403 Forbidden
```

### 6.6 Error Response - Account Inactive

```
403 Forbidden
```

### 6.7 Error Response - Tenant Not Found

```
404 Not Found
```

### 6.8 Error Response - System Error

```
500 Internal Server Error
```

### 6.9 Error Response - Bad Request

```
400 Bad Request
```

### 6.10 Error Response - Method Not Allowed

```
405 Method Not Allowed
```

### 6.11 Error Response - Too Many Requests

```
429 Too Many Requests
```

### 6.12 Error Response - Service Unavailable

```
503 Service Unavailable
```

### 6.13 Error Response - Gateway Timeout

```
504 Gateway Timeout
```

Untitled


# 7. Validation Rules

[Message list](https://app.notion.com/p/sokucom/Message-list-374f02c407dd8037808eea01e93be8aa?source=copy_link) 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | page | integer | - | integer |
| 2 | limit | string | - | string |
| 3 | status | in | 0, 1 | in |
| 4 | plan_code | string | - | string |
| 5 | tenant_code | max | 50 | max |
| 6 | company_name | max | 255 | max |
| 7 | sort_column | string | - | string |
| 8 | sort_direction | string | - | string |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant type restriction | API này chỉ được phép trả về các tenant có loại `tenant_type = 'saki'` (SAKI Tenant). |
| BR-002 | Role authorization | Chỉ người dùng thuộc nhóm quản trị Platform (SaaS Admin / SaaS Staff) có quyền `tenant.view` mới được phép thực hiện gọi API này. |
| BR-003 | Filter logic | Lọc theo `tenant_code` (map vào `company_id` like '%keyword%'), `company_name` (like '%keyword%'), `status`, và `plan_code` nếu các giá trị này được gửi lên. |
| BR-004 | Default values | Nếu `page` không được truyền, mặc định là 1. Nếu `limit` không được truyền, mặc định là 20. Nếu `sort_column` và `sort_direction` không được truyền, mặc định sắp xếp theo `created_at` DESC. |
| BR-005 | Dynamically calculated counts | `connected_moto_count` được tính bằng cách đếm số relation trong `platform.mst_moto_saki_relation` mà có `saki_tenant_id = tenant_id`. `active_contract_count`, `approval_user_count` và `billing_status` được lấy từ bảng cache thống kê hoạt động `platform.t_tenant_usage_metric` và `platform.t_usage_billing` của tháng hiện tại để tránh truy vấn trực tiếp vào database riêng của từng tenant, nhằm đảm bảo hiệu năng. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform SaaS Admin | Có | Không | Không | Không | Không | Không |
| Platform SaaS Staff | Có | Không | Không | Không | Không | Không |
| Tenant Admin (MOTO/SAKI) | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Xác thực access token JWT gửi kèm trong Authorization header.
2. Trích xuất thông tin người dùng và quyền hạn từ token hoặc database trung tâm.
3. Kiểm tra xem user có thuộc hệ thống quản trị Platform Manager và có permission "tenant.view" hay không.
4. Nếu hợp lệ, cho phép đi tiếp để xử lý truy vấn.
5. Nếu không hợp lệ, từ chối yêu cầu và trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Parameter | Table | Column | Logic truy vấn SQL |
| --- | --- | --- | --- |
| tenant_code | platform.tenant_registry | company_id | WHERE company_id LIKE CONCAT('%', :tenant_code, '%') |
| company_name | platform.tenant_registry | company_name | WHERE company_name LIKE CONCAT('%', :company_name, '%') |
| status | platform.tenant_registry | status | WHERE status = :status |
| plan_code | platform.tenant_registry | plan_code | WHERE plan_code = :plan_code |
| limit | - | - | LIMIT :limit |
| page | - | - | OFFSET (:page - 1) * :limit |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | data.items[].tenant_id | ID khóa chính (ULID) |
| platform.tenant_registry | company_id | data.items[].company_id | Mã Tenant |
| platform.tenant_registry | company_name | data.items[].company_name | Tên công ty phái cử |
| platform.tenant_registry | schema_name | data.items[].schema_name | Tên schema PostgreSQL của tenant |
| platform.tenant_registry | db_user | data.items[].db_user | PostgreSQL role của tenant |
| platform.tenant_registry | plan_code | data.items[].plan_code | Gói dịch vụ hợp đồng |
| platform.tenant_registry | contracted_at | data.items[].contracted_at | Ngày ký hợp đồng dịch vụ |
| platform.mst_moto_saki_relation | count | data.items[].connected_moto_count | Đếm số lượng liên kết có saki_tenant_id bằng tenant_id |
| platform.t_tenant_usage_metric | metric_value | data.items[].active_contract_count | Số hợp đồng đang hoạt động. Lọc theo `target_year_month = :current_ym` và `metric_type = 'active_contract_count'` |
| platform.t_tenant_usage_metric | metric_value | data.items[].approval_user_count | Số tài khoản phê duyệt SAKI. Lọc theo `target_year_month = :current_ym` và `metric_type = 'approval_user_count'` |
| platform.t_usage_billing | billing_status | data.items[].billing_status | Trạng thái thanh toán dịch vụ. Lọc theo `target_year_month = :current_ym` |
| platform.tenant_registry | status | data.items[].status | Trạng thái hoạt động (0: Tạm ngưng, 1: Hoạt động) |
| platform.tenant_registry | created_at | data.items[].created_at | Ngày đăng ký |
| platform.tenant_registry | updated_at | data.items[].updated_at | Ngày cập nhật gần nhất |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | Query data | Ngoài transaction |

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
1. Client gửi request: GET /api/v1/admin/saki-tenants kèm query parameters.
2. Middleware thực hiện kiểm tra Authentication và xác định JWT Token hợp lệ.
3. Middleware thực hiện kiểm tra Authorization (quyền "tenant.view").
4. Controller nhận request và thực hiện Validate các tham số Query Parameters.
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm chi tiết lỗi validation.
5. Controller gọi Service để lấy dữ liệu:
   - Xây dựng Query Builder truy vấn bảng `platform.tenant_registry` với điều kiện `tenant_type = 'saki'`.
   - Thực hiện LEFT JOIN với bảng `platform.mst_moto_saki_relation` để tính `connected_moto_count` thông qua đếm gom nhóm (count group by).
   - Thực hiện LEFT JOIN với bảng `platform.t_tenant_usage_metric` trên trường `tenant_id` và `target_year_month = :current_ym` để lấy `active_contract_count` (khi `metric_type = 'active_contract_count'`) và `approval_user_count` (khi `metric_type = 'approval_user_count'`) thông qua kỹ thuật xoay dòng (pivot/conditional aggregation).
   - Thực hiện LEFT JOIN với bảng `platform.t_usage_billing` trên trường `tenant_id` và `target_year_month = :current_ym` để lấy `billing_status`.
   - Áp dụng các điều kiện lọc: `tenant_code` (map vào `company_id`), `company_name`, `status`, `plan_code` nếu có.
   - Đếm tổng số bản ghi thỏa mãn điều kiện lọc.
   - Áp dụng Sắp xếp (ORDER BY theo `sort_column` và `sort_direction`).
   - Áp dụng Phân trang (LIMIT và OFFSET theo `page` và `limit`).
   - Thực hiện truy vấn dữ liệu từ DB.
6. Format dữ liệu kết quả thông qua API Resource (định dạng trường, chuyển múi giờ...).
7. Controller phản hồi Response JSON thành công 200 OK về phía Client.
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
| Có cần Audit Log không | Không |
| Loại thao tác | Không áp dụng |
| Bảng target | Không áp dụng |
| ID record target | Không áp dụng |
| Dữ liệu ghi log | Không áp dụng |
| Chính sách lưu trữ | Không áp dụng |

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
| Có download file không | Không |
| Permission Check | Không áp dụng |
| Phương thức download | Không áp dụng |
| Thời gian hết hạn | Không áp dụng |
| Audit Log | Không áp dụng |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token hợp lệ và phân quyền Platform SaaS Admin/Staff (`tenant.view`). |
| 2 | SQL Injection Prevention | Sử dụng Eloquent ORM / Query Builder với bindings tham số để ngăn chặn tấn công SQL injection thông qua các tham số search text (`tenant_code`, `company_name`). |
| 3 | Parameter Tampering | Giới hạn danh sách các cột được phép dùng trong `sort_column` để tránh việc khai thác cấu trúc DB hoặc gây lỗi truy vấn khi truyền tên cột không hợp lệ. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 1.0 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Bắt buộc với API danh sách (Server-side Pagination) |
| Index cần thiết | Index các cột `tenant_type`, `status`, `plan_code` và `created_at` trong bảng `tenant_registry` |
| Cache cần thiết | Có (Tối ưu hóa truy vấn JOIN hoặc cache bảng thống kê `t_tenant_usage_metric`) |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Wireframe | PA-TEN-007 SAKI Tenant List |
| Screen Detail Design | PA-TEN-007 |
| API liên quan | PA-TEN-007-API-02 Export saki tenant list |
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
| 12 | Audit log đã được định nghĩa | Done (Không áp dụng) |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
