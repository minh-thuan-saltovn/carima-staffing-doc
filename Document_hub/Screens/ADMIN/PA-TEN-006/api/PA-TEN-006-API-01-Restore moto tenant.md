# PA-TEN-006-API-01-Restore moto tenant

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Restore moto tenant |
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
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Restore MOTO Tenant dựa trên thiết kế và cấu trúc DB thực tế |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-006-API-01 |
| Tên API | Restore moto tenant |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-006 |
| Chức năng liên quan | Tenant Management (Restore MOTO Tenant) |
| Mục đích API | Khôi phục trạng thái hoạt động của một MOTO Tenant (từ Tạm ngưng sang Hoạt động). |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/admin/moto-tenants/{id}/restore |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để khôi phục trạng thái hoạt động cho một MOTO Tenant (派遣元テナント) đã bị tạm ngưng (suspend) trước đó, cho phép người dùng của Tenant đó đăng nhập lại và sử dụng bình thường.

### Ngữ cảnh nghiệp vụ

API được gọi khi Platform SaaS Admin nhấn nút "Khôi phục" (Restore) trên màn hình khôi phục Tenant MOTO (PA-TEN-006) hoặc từ menu hành động nhanh trên màn hình danh sách (PA-TEN-001).

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User nhấn nút Restore Tenant trên giao diện màn hình PA-TEN-006 hoặc PA-TEN-001 |
| Actor | Platform SaaS Admin / Platform SaaS Staff |
| Trước khi gọi API | Tenant đang bị tạm ngưng (`status = 0`) trong database |
| Sau khi gọi API | Tenant chuyển sang trạng thái hoạt động (`status = 1`), ghi nhận audit log hệ thống |
| Thay đổi trạng thái liên quan | Trạng thái hoạt động của Tenant chuyển từ `0` sang `1` |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/admin/moto-tenants/{id}/restore
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
| id | string | Có | ID của tenant cần khôi phục (khóa chính `tenant_id` - ULID) | 01H2PJX7G3P681729X4R09Q871 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

Không áp dụng (Hành động khôi phục thông qua Path Parameter không cần dữ liệu gửi kèm ở body).

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
    "tenant_id": "01H2PJX7G3P681729X4R09Q871",
    "company_id": "MOTO-0001",
    "company_name": "MOTO Staffing Solutions",
    "status": 1,
    "updated_at": "2026-06-17T09:30:00+09:00"
  }
}
```

### Chi tiết các trường dữ liệu trả về

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant_id | string | ID khóa chính (ULID) của tenant vừa khôi phục |
| data.company_id | string | Mã doanh nghiệp (business key) của tenant |
| data.company_name | string | Tên doanh nghiệp MOTO |
| data.status | number | Trạng thái mới của tenant (1: Hoạt động) |
| data.updated_at | string | Thời gian cập nhật trạng thái |

---

## 6.2 Định nghĩa cấu trúc lỗi

### Validation/Invalid Parameter Error

```
422 Unprocessable Entity
```

```json
{
  "error": {
    "code": "INVALID_TENANT_ID",
    "message": "The tenant ID format is invalid."
  }
}
```

### Resource Not Found

```
404 Not Found
```

```json
{
  "error": {
    "code": "TENANT_NOT_FOUND",
    "message": "Tenant not found or not a MOTO tenant."
  }
}
```

### State Conflict Error

```
409 Conflict
```

```json
{
  "error": {
    "code": "TENANT_ALREADY_ACTIVE",
    "message": "The tenant is already in active status."
  }
}
```

---

# 7. Validation Rules

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | id | required | - | required |
| 2 | id | size | 26 | size |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant type restriction | Chỉ cho phép thực hiện khôi phục trạng thái đối với tenant thuộc loại `tenant_type = 'moto'`. Nếu tìm thấy tenant có ID đó nhưng là loại khác, hệ thống sẽ trả về lỗi `404 Not Found`. |
| BR-002 | Initial status requirement | Chỉ cho phép khôi phục khi trạng thái hiện tại trong cơ sở dữ liệu của tenant là tạm ngưng (`status = 0`). Trường hợp tenant đã ở trạng thái hoạt động (`status = 1`), trả về lỗi `409 Conflict`. |
| BR-003 | Authorization check | Chỉ người dùng thuộc nhóm quản trị Platform (SaaS Admin / SaaS Staff) có quyền `tenant.restore` mới được phép gọi API này. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Restore |
| --- | --- | --- |
| Platform SaaS Admin | Có | Có |
| Platform SaaS Staff | Có | Có |
| Tenant Admin (MOTO/SAKI) | Không | Không |

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Parameter | Table | Column | Logic truy vấn SQL |
| --- | --- | --- | --- |
| id | platform.tenant_registry | tenant_id | WHERE tenant_id = :id AND tenant_type = 'moto' |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | data.tenant_id | ID khóa chính (ULID) |
| platform.tenant_registry | company_id | data.company_id | Mã Tenant |
| platform.tenant_registry | company_name | data.company_name | Tên doanh nghiệp MOTO |
| platform.tenant_registry | status | data.status | Trạng thái hoạt động (1: Hoạt động) |
| platform.tenant_registry | updated_at | data.updated_at | Thời gian cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | SELECT FOR UPDATE bản ghi tenant | Trong transaction |
| 4 | UPDATE trạng thái status = 1, updated_at = now() | Trong transaction |
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
| 0 (Tạm ngưng) | PATCH /restore | 1 (Hoạt động) | Thao tác thành công |

---

# 13. Sequence / Processing Flow

```
1. Client gửi request: PATCH /api/v1/admin/moto-tenants/{id}/restore.
2. Middleware thực hiện kiểm tra Authentication và xác định JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization): Platform SaaS Admin/Staff có quyền "tenant.restore".
4. Controller nhận request và thực hiện validate tham số Path Parameter `id` (ULID format).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity.
5. Controller gọi Service thực hiện nghiệp vụ khôi phục trong một Database Transaction:
   - Truy vấn bản ghi trong bảng `platform.tenant_registry` theo `id` kèm theo khóa Lock (`SELECT FOR UPDATE`).
   - Nếu không tìm thấy hoặc `tenant_type != 'moto'`: Throw Exception -> Trả về HTTP 404 Not Found.
   - Kiểm tra trạng thái hiện tại của tenant:
     - Nếu `status == 1`: Throw Exception -> Trả về HTTP 409 Conflict.
   - Thực hiện cập nhật trạng thái bản ghi: `status = 1`, `updated_at = now()`.
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
| Dữ liệu ghi log | Before: status = 0; After: status = 1 |
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
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token hợp lệ và phân quyền Platform SaaS Admin/Staff (`tenant.restore`). |
| 2 | SQL Injection Prevention | Sử dụng Eloquent ORM hoặc Query Builder với bindings tham số đầy đủ khi thực hiện các câu lệnh kiểm tra và cập nhật dữ liệu. |
| 3 | Lock Concurrency | Sử dụng cơ chế khóa dòng `SELECT ... FOR UPDATE` để tránh tranh chấp trạng thái (Race Condition) khi có nhiều Admin cùng thực hiện thao tác khôi phục/tạm ngưng một tenant tại cùng một thời điểm. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.5 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính `tenant_id` (ULID) |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Wireframe | PA-TEN-006 Restore MOTO Tenant |
| Screen Detail Design | PA-TEN-006 |
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
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
