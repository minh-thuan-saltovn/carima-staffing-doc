# PA-TEN-016-API-01-Tenant Usage Dashboard

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Tenant Usage Dashboard |
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
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Tenant Usage Dashboard dựa trên thiết kế màn hình PA-TEN-016 và ERD thực tế (platform.t_tenant_usage_metric, platform.t_usage_billing) |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-016-API-01 |
| Tên API | Tenant Usage Dashboard |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-016 |
| Chức năng liên quan | Tenant Management (Usage Dashboard) |
| Mục đích API | Lấy tổng quan số liệu tài nguyên sử dụng và lịch sử 6 tháng gần nhất của một Tenant để hiển thị trên Dashboard. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/admin/tenants/{id}/usage |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này tổng hợp số liệu sử dụng tài nguyên (số lượng người dùng, dung lượng lưu trữ, số lượt gọi API, số lượt đăng nhập) của một Tenant cụ thể từ bảng `platform.t_tenant_usage_metric`, đồng thời trả về lịch sử 6 tháng gần nhất để hiển thị Biểu đồ xu hướng trên màn hình Dashboard.

### Ngữ cảnh nghiệp vụ

API được gọi tự động khi Platform SaaS Admin hoặc Staff mở màn hình Dashboard sử dụng (PA-TEN-016) của một Tenant.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Mở màn hình PA-TEN-016 (Initial Load) |
| Actor | Platform SaaS Admin, Platform SaaS Staff |
| Trước khi gọi API | Tenant đã tồn tại trong hệ thống và có dữ liệu metric được ghi nhận bởi Batch Job định kỳ |
| Sau khi gọi API | Hiển thị số liệu tài nguyên tháng hiện tại và biểu đồ xu hướng 6 tháng trên màn hình |
| Thay đổi trạng thái liên quan | Không (API chỉ đọc) |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/admin/tenants/{id}/usage
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | ID của Tenant cần xem dashboard (ULID) | 01H2PJX7G3P681729X4R09Q872 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

Không áp dụng (API GET không có Request Body).

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
  "data": {
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "company_id": "SAKI-0001",
    "company_name": "株式会社サキケア",
    "tenant_type": "saki",
    "current_month": "2026/06",
    "metrics": [
      {
        "metric_type": "active_users",
        "metric_value": 45
      },
      {
        "metric_type": "storage_used_bytes",
        "metric_value": 13314398617
      },
      {
        "metric_type": "monthly_api_requests",
        "metric_value": 15240
      },
      {
        "metric_type": "monthly_login_count",
        "metric_value": 120
      }
    ],
    "history": [
      {
        "target_year_month": "2025/12",
        "metrics": [
          { "metric_type": "monthly_login_count", "metric_value": 95 },
          { "metric_type": "monthly_api_requests", "metric_value": 11200 }
        ]
      },
      {
        "target_year_month": "2026/01",
        "metrics": [
          { "metric_type": "monthly_login_count", "metric_value": 100 },
          { "metric_type": "monthly_api_requests", "metric_value": 12400 }
        ]
      },
      {
        "target_year_month": "2026/02",
        "metrics": [
          { "metric_type": "monthly_login_count", "metric_value": 88 },
          { "metric_type": "monthly_api_requests", "metric_value": 10500 }
        ]
      },
      {
        "target_year_month": "2026/03",
        "metrics": [
          { "metric_type": "monthly_login_count", "metric_value": 110 },
          { "metric_type": "monthly_api_requests", "metric_value": 13800 }
        ]
      },
      {
        "target_year_month": "2026/04",
        "metrics": [
          { "metric_type": "monthly_login_count", "metric_value": 105 },
          { "metric_type": "monthly_api_requests", "metric_value": 14200 }
        ]
      },
      {
        "target_year_month": "2026/05",
        "metrics": [
          { "metric_type": "monthly_login_count", "metric_value": 120 },
          { "metric_type": "monthly_api_requests", "metric_value": 15240 }
        ]
      }
    ]
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant_id | string | ID định danh duy nhất của Tenant (ULID) — từ `platform.tenant_registry.tenant_id` |
| data.company_id | string | Mã công ty (business key) — từ `platform.tenant_registry.company_id` |
| data.company_name | string | Tên công ty — từ `platform.tenant_registry.company_name` |
| data.tenant_type | string | Loại Tenant (`saki` hoặc `moto`) — từ `platform.tenant_registry.tenant_type` |
| data.current_month | string | Tháng hiện tại đang hiển thị dữ liệu (YYYY/MM) |
| data.metrics | array | Mảng số liệu sử dụng tháng hiện tại |
| data.metrics[].metric_type | string | Loại chỉ số — từ `platform.t_tenant_usage_metric.metric_type` |
| data.metrics[].metric_value | integer | Giá trị chỉ số — từ `platform.t_tenant_usage_metric.metric_value` |
| data.history | array | Mảng dữ liệu lịch sử 6 tháng gần nhất (không bao gồm tháng hiện tại) |
| data.history[].target_year_month | string | Tháng lịch sử (YYYY/MM) — từ `platform.t_tenant_usage_metric.target_year_month` |
| data.history[].metrics | array | Mảng số liệu của tháng lịch sử tương ứng |

### Các giá trị `metric_type` chuẩn

| metric_type | Đơn vị | Mô tả |
| --- | --- | --- |
| `active_users` | người | Số tài khoản người dùng đang hoạt động trong tenant |
| `storage_used_bytes` | bytes | Tổng dung lượng lưu trữ tệp tin đã sử dụng |
| `monthly_api_requests` | lượt | Tổng số lượt gọi API trong tháng |
| `monthly_login_count` | lượt | Tổng số lượt đăng nhập trong tháng |

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
| 1 | id | required | - | required |
| 2 | id | size | 26 | size |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Role authorization | Cả Platform SaaS Admin và Platform SaaS Staff đều có quyền xem Dashboard (quyền `tenant.view`). |
| BR-002 | Metric data source | Dữ liệu số liệu được đọc từ bảng `platform.t_tenant_usage_metric`. Bảng này được Batch Job cập nhật định kỳ (không phải real-time). |
| BR-003 | Current month metrics | Số liệu tháng hiện tại (`current_month`) là bản ghi mới nhất trong `t_tenant_usage_metric` có `target_year_month` = tháng hiện tại của hệ thống. |
| BR-004 | History range | Trả về dữ liệu của **6 tháng liên tiếp gần nhất trước tháng hiện tại** theo chiều tăng dần (`ORDER BY target_year_month ASC`). Nếu tháng nào chưa có dữ liệu thì không trả về tháng đó. |
| BR-005 | History metric types | Lịch sử chỉ trả về 2 loại metric phục vụ biểu đồ xu hướng: `monthly_login_count` và `monthly_api_requests`. |
| BR-006 | No audit log | API này chỉ đọc (GET), không cần ghi Audit Log. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View |
| --- | --- |
| Platform SaaS Admin | Có |
| Platform SaaS Staff | Có |
| Tenant Admin (MOTO/SAKI) | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Xác thực token JWT gửi kèm trong header Authorization.
2. Trích xuất thông tin tài khoản người dùng.
3. Xác minh tài khoản thuộc nhóm Platform SaaS Admin hoặc Platform SaaS Staff.
4. Kiểm tra quyền hạn "tenant.view" của tài khoản.
5. Nếu hợp lệ, cho phép tiếp tục xử lý.
6. Nếu không hợp lệ, từ chối yêu cầu và trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ DB ra Response

### Thông tin Tenant

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | data.tenant_id | ID định danh Tenant (ULID) |
| platform.tenant_registry | company_id | data.company_id | Mã công ty (business key) |
| platform.tenant_registry | company_name | data.company_name | Tên công ty |
| platform.tenant_registry | tenant_type | data.tenant_type | Loại Tenant |

### Số liệu sử dụng tháng hiện tại

| Table | Column | Response Field | Điều kiện lọc |
| --- | --- | --- | --- |
| platform.t_tenant_usage_metric | metric_type | data.metrics[].metric_type | WHERE tenant_id = :id AND target_year_month = :current_month |
| platform.t_tenant_usage_metric | metric_value | data.metrics[].metric_value | WHERE tenant_id = :id AND target_year_month = :current_month |

### Lịch sử 6 tháng gần nhất

| Table | Column | Response Field | Điều kiện lọc |
| --- | --- | --- | --- |
| platform.t_tenant_usage_metric | target_year_month | data.history[].target_year_month | WHERE tenant_id = :id AND target_year_month IN (6 tháng trước tháng hiện tại) AND metric_type IN ('monthly_login_count', 'monthly_api_requests') ORDER BY target_year_month ASC |
| platform.t_tenant_usage_metric | metric_type | data.history[].metrics[].metric_type | (như trên) |
| platform.t_tenant_usage_metric | metric_value | data.history[].metrics[].metric_value | (như trên) |

---

# 11. Database Transaction

Không áp dụng — API chỉ đọc (SELECT), không có ghi DB.

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: GET /api/v1/admin/tenants/{id}/usage.
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization): Platform SaaS Admin hoặc Staff (quyền "tenant.view").
4. Controller validate Path Parameter `id` (ULID format).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity.
5. Service tra cứu thông tin Tenant trong `platform.tenant_registry` theo `id`.
   - Nếu không tìm thấy: Trả về HTTP 404 Not Found.
6. Service xác định tháng hiện tại của hệ thống (format YYYY/MM).
7. Service truy vấn số liệu tháng hiện tại từ `platform.t_tenant_usage_metric`:
   - WHERE tenant_id = :id AND target_year_month = :current_month
   - Lấy tất cả metric_type có sẵn.
8. Service truy vấn lịch sử 6 tháng gần nhất từ `platform.t_tenant_usage_metric`:
   - WHERE tenant_id = :id
     AND target_year_month IN (6 tháng trước tháng hiện tại)
     AND metric_type IN ('monthly_login_count', 'monthly_api_requests')
   - ORDER BY target_year_month ASC
9. Service tổng hợp dữ liệu, nhóm theo target_year_month cho phần history.
10. Controller định dạng response qua API Resource và trả về HTTP 200 OK.
```

---

# 14. Notification

Không áp dụng cho API này.

---

# 15. Audit Log

Không áp dụng — API chỉ đọc (GET), không cần ghi Audit Log.

---

# 16. Xử lý file

Không áp dụng.

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc xác thực JWT Token và kiểm tra quyền `tenant.view`. |
| 2 | Tenant Isolation | API chỉ trả về dữ liệu của đúng `tenant_id` được truyền vào. Không cho phép truy xuất dữ liệu tenant khác. |
| 3 | Parameter Validation | Path parameter `id` phải được validate đúng định dạng ULID trước khi truy vấn DB. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 1 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Không áp dụng |
| Index cần thiết | Index composite `(tenant_id, target_year_month)` và `(metric_type)` trên bảng `platform.t_tenant_usage_metric` — đã có trong ERD |
| Cache cần thiết | Có thể cache response trong 5–15 phút vì dữ liệu metric được Batch Job cập nhật định kỳ, không real-time |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Giao diện liên quan | PA-TEN-016 テナント利用状況 |
| Thiết kế màn hình | spec-screen/PA-TEN-016.md |
| ERD | platform.tenant_registry, platform.t_tenant_usage_metric, platform.t_usage_billing |
| API List | API List 341f02c407dd80c99339fa65d175b7e1.csv |

---

# 20. Open Issues

Không có.

---

# 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Done |
| 2 | Request body đã được định nghĩa | Done |
| 3 | Response body đã được định nghĩa | Done |
| 4 | Error response đã được định nghĩa | Done |
| 5 | Validation rule đã được định nghĩa | Done |
| 6 | Business rule đã được định nghĩa | Done |
| 7 | Permission check đã được định nghĩa | Done |
| 8 | Tenant isolation rule đã được định nghĩa | Done |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done |
| 14 | File handling rule đã được định nghĩa nếu cần | Done |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
