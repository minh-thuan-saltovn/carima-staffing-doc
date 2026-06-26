# PA-TEN-011-API-01-Suspend saki tenant

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Suspend saki tenant |
| Description | Tạm ngưng tenant SAKI. |
| Endpoint | `/api/v1/admin/saki-tenants/{id}/suspend` |
| Menu | Tenant Management |
| Method | PATCH |
| Related Tables | `platform.tenant_registry` |
| Screen list | 派遣先テナント停止 (PA-TEN-011) — Suspend SAKI Tenant |
| System | Platform SaaS Admin |
| 説明 | SAKIテナントを停止する。 |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Suspend saki tenant |
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
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Suspend saki tenant theo template và cấu trúc DB thực tế |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-011-API-01 |
| Tên API | Suspend saki tenant |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-011 / PA-TEN-007 |
| Chức năng liên quan | Tenant Management (Suspend SAKI Tenant) |
| Mục đích API | Tạm ngưng hoạt động của một SAKI Tenant (từ Hoạt động sang Tạm ngưng). |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/admin/saki-tenants/{id}/suspend |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để tạm ngưng hoạt động của một SAKI Tenant (派遣先テナント), ngăn người dùng của Tenant đó đăng nhập và sử dụng hệ thống cho đến khi được khôi phục.

### Ngữ cảnh nghiệp vụ

API được gọi khi Platform SaaS Admin nhấn nút "停止" (Tạm ngưng) trên màn hình danh sách SAKI Tenant (PA-TEN-007) hoặc từ màn hình tạm ngưng Tenant SAKI (PA-TEN-011).

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User nhấn nút Suspend Tenant trên giao diện màn hình PA-TEN-007 hoặc PA-TEN-011, xác nhận qua dialog `CMS-VAL-85` |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Tenant đang ở trạng thái hoạt động (`status = 1`) trong database |
| Sau khi gọi API | Tenant chuyển sang trạng thái tạm ngưng (`status = 0`), ghi nhận audit log hệ thống |
| Thay đổi trạng thái liên quan | Trạng thái hoạt động của Tenant chuyển từ `1` sang `0` |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/admin/saki-tenants/{id}/suspend
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về khi có lỗi |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | ID của tenant cần tạm ngưng (khóa chính `tenant_id` - ULID) | 01H2PJX7G3P681729X4R09Q872 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

Không áp dụng (Hành động tạm ngưng thông qua Path Parameter, không cần dữ liệu gửi kèm ở body).

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
  "data": {
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "company_id": "SAKI-0001",
    "company_name": "SAKI Partner Factory",
    "status": 0,
    "updated_at": "2026-06-17T09:30:00+09:00"
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant_id | string | ID khóa chính (ULID) của tenant vừa tạm ngưng |
| data.company_id | string | Mã doanh nghiệp (business key) của tenant |
| data.company_name | string | Tên doanh nghiệp SAKI |
| data.status | number | Trạng thái mới của tenant (0: Tạm ngưng) |
| data.updated_at | string | Thời gian cập nhật trạng thái |

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

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | id | required | - | required |
| 2 | id | string | - | string |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant type restriction | Chỉ cho phép thực hiện tạm ngưng đối với tenant thuộc loại `tenant_type = 'saki'`. Nếu tìm thấy tenant có ID đó nhưng là loại khác, hệ thống sẽ trả về lỗi `404 Not Found`. |
| BR-002 | Initial status requirement | Chỉ cho phép tạm ngưng khi trạng thái hiện tại trong cơ sở dữ liệu của tenant là hoạt động (`status = 1`). Trường hợp tenant đã ở trạng thái tạm ngưng (`status = 0`), trả về lỗi `409 Conflict`. |
| BR-003 | Authorization check | Chỉ người dùng thuộc nhóm Platform SaaS Admin có quyền `tenant.suspend` mới được phép gọi API này. |
| BR-004 | Access blocking | Sau khi tạm ngưng thành công, toàn bộ người dùng thuộc tenant SAKI bị từ chối đăng nhập và truy cập hệ thống cho đến khi tenant được khôi phục. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Suspend |
| --- | --- | --- |
| Platform SaaS Admin | Có | Có |
| Platform SaaS Staff | Có | Không |
| Tenant Admin (MOTO/SAKI) | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Validate JWT token
2. Xác định admin user từ token context
3. Kiểm tra user thuộc nhóm Platform SaaS Admin
4. Kiểm tra quyền tenant.suspend
5. Từ chối request nếu user không đủ quyền (HTTP 403)
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Parameter | Table | Column | Logic truy vấn SQL |
| --- | --- | --- | --- |
| id | platform.tenant_registry | tenant_id | WHERE tenant_id = :id AND tenant_type = 'saki' |

**Cập nhật dữ liệu:**

| Table | Column | Giá trị mới | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | status | `0` | Chuyển trạng thái sang Tạm ngưng (停止中) |
| platform.tenant_registry | updated_at | `now()` | Thời điểm cập nhật trạng thái |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | data.tenant_id | ID khóa chính (ULID) |
| platform.tenant_registry | company_id | data.company_id | Mã Tenant |
| platform.tenant_registry | company_name | data.company_name | Tên doanh nghiệp SAKI |
| platform.tenant_registry | status | data.status | Trạng thái hoạt động (0: Tạm ngưng) |
| platform.tenant_registry | updated_at | data.updated_at | Thời gian cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | SELECT FOR UPDATE bản ghi tenant | Trong transaction |
| 4 | UPDATE trạng thái status = 0, updated_at = now() | Trong transaction |
| 5 | INSERT ghi nhận audit log vào platform.tenant_provision_log | Trong transaction |
| 6 | Commit transaction | Trong transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | SELECT FOR UPDATE thất bại hoặc có lỗi DB | Rollback toàn bộ DB changes |
| 2 | UPDATE trạng thái tenant thất bại | Rollback toàn bộ DB changes |
| 3 | INSERT ghi nhận audit log thất bại | Rollback toàn bộ DB changes |
| 4 | Bất kỳ ngoại lệ hệ thống (Exception) nào phát sinh | Rollback toàn bộ DB changes |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| 1 (Hoạt động) | PATCH /suspend | 0 (Tạm ngưng) | Thao tác thành công |

---

# 13. Sequence / Processing Flow

```
1. Client gửi request: PATCH /api/v1/admin/saki-tenants/{id}/suspend.
2. Middleware thực hiện kiểm tra Authentication và xác định JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization): Platform SaaS Admin có quyền "tenant.suspend".
4. Controller nhận request và thực hiện validate tham số Path Parameter `id` (ULID format).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity.
5. Controller gọi Service thực hiện nghiệp vụ tạm ngưng trong một Database Transaction:
   - Truy vấn bản ghi trong bảng `platform.tenant_registry` theo `id` kèm theo khóa Lock (`SELECT FOR UPDATE`).
   - Nếu không tìm thấy hoặc `tenant_type != 'saki'`: Throw Exception -> Trả về HTTP 404 Not Found.
   - Kiểm tra trạng thái hiện tại của tenant:
     - Nếu `status == 0`: Throw Exception -> Trả về HTTP 409 Conflict.
   - Thực hiện cập nhật bản ghi: `status = 0`, `updated_at = now()`.
   - Ghi nhật ký thao tác (Audit Log) vào bảng `platform.tenant_provision_log`.
6. Transaction được Commit thành công.
7. Controller trả về dữ liệu Tenant sau cập nhật kèm HTTP 200 OK.
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
| Loại thao tác | UPDATE |
| Bảng target | platform.tenant_registry |
| ID record target | tenant_id |
| Dữ liệu ghi log | Before: status = 1; After: status = 0 |
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
| Có download file không | Không |
| Permission Check | Không áp dụng |
| Phương thức download | Không áp dụng |
| Thời gian hết hạn | Không áp dụng |
| Audit Log | Không áp dụng |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token hợp lệ và phân quyền Platform SaaS Admin (`tenant.suspend`). |
| 2 | SQL Injection Prevention | Sử dụng Eloquent ORM hoặc Query Builder với bindings tham số đầy đủ khi thực hiện các câu lệnh kiểm tra và cập nhật dữ liệu. |
| 3 | Lock Concurrency | Sử dụng cơ chế khóa dòng `SELECT ... FOR UPDATE` để tránh tranh chấp trạng thái (Race Condition) khi có nhiều Admin cùng thực hiện thao tác tạm ngưng/khôi phục một tenant tại cùng một thời điểm. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.5 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính `tenant_id` (ULID), index `idx_registry_status` trên cột `status` |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Wireframe | PA-TEN-011 Suspend SAKI Tenant |
| Screen Detail Design | PA-TEN-011 |
| API liên quan | PA-TEN-007-API-01 Saki tenant list, PA-TEN-012-API-01 Restore saki tenant |
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
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
