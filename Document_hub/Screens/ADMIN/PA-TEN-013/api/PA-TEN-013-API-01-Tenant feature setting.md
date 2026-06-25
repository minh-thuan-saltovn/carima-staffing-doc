# PA-TEN-013-API-01-Tenant feature setting

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Tenant feature setting (Plan setting) |
| Description | Cập nhật Gói dịch vụ Tenant (plan_code). |
| Endpoint | `/api/v1/admin/tenants/{id}/features` |
| Menu | Tenant Management |
| Method | PATCH |
| Related Tables | `platform.tenant_registry` |
| Screen list | テナント機能設定 (PA-TEN-013) — Tenant Feature Setting |
| System | Platform SaaS Admin |
| 説明 | Tenantの利用プラン（plan_code）を更新する। |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Tenant feature setting |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.3 |  
| Ngày tạo | 2026/06/17 |
| Ngày cập nhật | 2026/06/17 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Tenant feature setting theo template cũ |
| v1.1 | 2026/06/17 | Thuận | Loại bỏ bảng tenant_feature ảo, chuyển sang cấu hình plan_code trong platform.tenant_registry |
| v1.2 | 2026/06/17 | Thuận | Loại bỏ các cờ tính năng giả định, giữ thiết kế plan_code |
| v1.3 | 2026/06/17 | Thuận | Tách biệt API: chỉ đặc tả duy nhất phương thức PATCH theo quy chuẩn dự án (1 file = 1 API) |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-013-API-01 |
| Tên API | Tenant feature setting |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-013 |
| Chức năng liên quan | Tenant Management (Tenant Feature Setting) |
| Mục đích API | Cập nhật gói dịch vụ (plan_code) của một Tenant cụ thể trên hệ thống. |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/admin/tenants/{id}/features |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để cập nhật gói dịch vụ (`plan_code`) của Tenant do quản trị viên hệ thống (Platform SaaS Admin) thay đổi. Sự thay đổi có hiệu lực ngay lập tức.

### Ngữ cảnh nghiệp vụ

API được gọi khi quản trị viên hệ thống nhấn nút "設定保存" (Lưu cấu hình) trên form cấu hình tính năng của màn hình `PA-TEN-013`.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Click nút Lưu cấu hình trên màn hình PA-TEN-013, xác nhận qua dialog |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Quản trị viên thay đổi gói dịch vụ trên giao diện màn hình |
| Sau khi gọi API | Cập nhật `plan_code` trong DB, ghi Audit log, thay đổi có hiệu lực ngay lập tức đối với người dùng Tenant |
| Thay đổi trạng thái liên quan | Cập nhật trường `plan_code` trong bảng `platform.tenant_registry` |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/admin/tenants/{id}/features
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
| id | string | Có | ID của tenant cần thiết lập tính năng (khóa chính `tenant_id` - ULID) | 01H2PJX7G3P681729X4R09Q872 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "plan_code": "PRO"
}
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| plan_code | string | Có | 20 | Bắt buộc. Giá trị thuộc: `LITE`, `STANDARD`, `PRO`, `ENTERPRISE` | Mã gói dịch vụ đăng ký |

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
    "plan_code": "PRO"
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant_id | string | ID định danh của Tenant |
| data.plan_code | string | Mã gói dịch vụ sau khi cập nhật thành công |

---

## 6.3 Error Response

### Validation Error (422 Unprocessable Entity)

### Unauthorized (401 Unauthorized)

### Forbidden (403 Forbidden)

### Not Found (404 Not Found)

---


# 7. Validation Rules

Message list 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | id | size | 26 | size |
| 2 | plan_code | required | - | required |
| 3 | plan_code | string | - | string |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Immediate effect | Các thay đổi về `plan_code` có hiệu lực ngay lập tức đối với hệ thống. |
| BR-002 | Authorization check | Chỉ Platform SaaS Admin mới được phép thay đổi gói dịch vụ. Platform SaaS Staff bị từ chối quyền chỉnh sửa (HTTP 403). |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform SaaS Admin | Không | Không | Có | Không | Không | Không |
| Platform SaaS Staff | Không | Không | Không | Không | Không | Không |
| Tenant Admin (MOTO/SAKI) | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Validate JWT token
2. Xác định admin user từ token context
3. Kiểm tra user thuộc nhóm quản trị Platform
4. Kiểm tra quyền ghi tenant.update (phải thuộc Platform SaaS Admin)
5. Từ chối request nếu user không đủ quyền (HTTP 403)
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Parameter | Table | Column | Logic truy vấn & cập nhật |
| --- | --- | --- | --- |
| id | platform.tenant_registry | tenant_id | WHERE tenant_id = :id |

**Cập nhật dữ liệu:**

| Table | Column | Giá trị mới | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | plan_code | `plan_code` | Mã gói dịch vụ mới được cập nhật |
| platform.tenant_registry | updated_at | `now()` | Thời điểm cập nhật trạng thái |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | data.tenant_id | ID định danh Tenant |
| platform.tenant_registry | plan_code | data.plan_code | Mã gói dịch vụ sau cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request body đầu vào (plan_code hợp lệ) | Ngoài transaction |
| 2 | Check permission (Phải có quyền tenant.update) | Ngoài transaction |
| 3 | SELECT FOR UPDATE bản ghi tenant trong platform.tenant_registry | Trong transaction |
| 4 | UPDATE plan_code, updated_at = now() | Trong transaction |
| 5 | INSERT ghi nhận audit log vào platform.tenant_provision_log | Trong transaction |
| 6 | Commit transaction | Trong transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | SELECT FOR UPDATE tenant thất bại hoặc không tồn tại | Rollback toàn bộ DB changes |
| 2 | UPDATE plan_code thất bại | Rollback toàn bộ DB changes |
| 3 | INSERT ghi nhận audit log thất bại | Rollback toàn bộ DB changes |
| 4 | Bất kỳ ngoại lệ hệ thống (Exception) nào phát sinh | Rollback toàn bộ DB changes |

---

# 12. Status Transition

Không áp dụng cho API này.

---

# 13. Sequence / Processing Flow

### 13.1 Quy trình cập nhật cấu hình tính năng (PATCH)

```
1. Client gửi request: PATCH /api/v1/admin/tenants/{id}/features với Request Body chứa `plan_code`.
2. Middleware thực hiện kiểm tra Authentication và xác định JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization): Platform SaaS Admin có quyền "tenant.update".
4. Controller nhận request và thực hiện validate tham số:
   - Validate Path Parameter `id` (ULID format).
   - Validate Request Body (`plan_code` bắt buộc và phải nằm trong list LITE, STANDARD, PRO, ENTERPRISE).
   - Nếu có bất kỳ lỗi validate nào: Trả về HTTP 422 Unprocessable Entity.
5. Controller gọi Service thực hiện nghiệp vụ cập nhật cấu hình trong một Database Transaction:
   - SELECT khóa dòng bản ghi tenant trong bảng `platform.tenant_registry` theo `tenant_id = id` (`SELECT FOR UPDATE`).
     - Nếu không tìm thấy: Throw Exception -> Trả về HTTP 404 Not Found.
   - Thực hiện cập nhật `plan_code` và `updated_at = now()`.
   - Ghi nhật ký thao tác (Audit Log) chi tiết việc thay đổi gói dịch vụ vào bảng `platform.tenant_provision_log` với action = 'update_plan'.
6. Transaction được Commit thành công.
7. Controller trả về trạng thái plan_code sau khi cập nhật kèm HTTP 200 OK.
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
| Dữ liệu ghi log | Giá trị `plan_code` trước và sau khi thay đổi (Before & After) |
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
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token hợp lệ và phân quyền Platform SaaS Admin (`tenant.update`) trước khi thực hiện lưu. |
| 2 | SQL Injection Prevention | Sử dụng Eloquent ORM hoặc Query Builder với bindings tham số đầy đủ khi thực hiện các câu lệnh kiểm tra và cập nhật dữ liệu. |
| 3 | Lock Concurrency | Sử dụng cơ chế khóa dòng `SELECT ... FOR UPDATE` trên bảng `platform.tenant_registry` để tránh tranh chấp trạng thái (Race Condition) khi có nhiều Admin cùng thực hiện thay đổi cấu hình một tenant tại cùng một thời điểm. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.3 giây |
| Lượng truy cập dự kiến | Thấp (chỉ sử dụng bởi Platform Admin) |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính `tenant_id` (ULID) trên bảng `platform.tenant_registry` |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Wireframe | PA-TEN-013 Tenant Feature Setting |
| Screen Detail Design | PA-TEN-013 |
| API liên quan | PA-TEN-002 Moto tenant detail, PA-TEN-008 Saki tenant detail |
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
| 2 | Request body đã được định nghĩa | Done |
| 3 | Response body đã được định nghĩa | Done |
| 4 | Error response đã được định nghĩa | Done |
| 5 | Validation rule đã được định nghĩa | Done |
| 6 | Business rule đã được định nghĩa | Done |
| 7 | Permission check đã được định nghĩa | Done |
| 8 | Tenant isolation rule đã được định nghĩa | Done (Không áp dụng) |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done (Không áp dụng) |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
