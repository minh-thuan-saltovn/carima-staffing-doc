# PA-TEN-014-API-01-Tenant Domain Setting

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Tenant Domain Setting |
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
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Tenant Domain Setting dựa trên thiết kế màn hình, cấu trúc DB thực tế và danh sách API List mới nhất |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-014-API-01 |
| Tên API | Tenant Domain Setting |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-014 |
| Chức năng liên quan | Tenant Management (Domain Setting) |
| Mục đích API | Cấu hình tên miền (Subdomain hoặc Custom Domain) và khởi tạo trạng thái xác minh kết nối DNS/SSL cho một Tenant. |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/admin/tenants/{id}/domains |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để thiết lập tên miền truy cập riêng (Subdomain hệ thống hoặc Custom Domain) cho một Tenant, đồng thời đưa các cờ trạng thái xác minh DNS và chứng chỉ SSL về giá trị chờ kiểm tra (`0: 未検証` và `0: 未適用`) để các tác vụ nền tự động kiểm tra kết nối sau đó.

### Ngữ cảnh nghiệp vụ

API được gọi khi Platform SaaS Admin nhấn nút "Lưu cấu hình" (Lưu) tại màn hình cấu hình tên miền Tenant (PA-TEN-014).

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User nhấn nút "Lưu cấu hình" (Lưu) trên màn hình PA-TEN-014 sau khi xác nhận dialog |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Tenant đã tồn tại trong hệ thống. |
| Sau khi gọi API | Cấu hình tên miền được ghi nhận vào bảng cấu hình trung tâm, ghi audit log, reset trạng thái kiểm tra SSL/DNS. |
| Thay đổi trạng thái liên quan | Cập nhật cấu hình routing tên miền tương ứng cho tenant. |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/admin/tenants/{id}/domains
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | ID của Tenant cần cấu hình tên miền (ULID) | 01H2PJX7G3P681729X4R09Q872 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "domain_type": 2,
  "domain_name": "jobs.sakicare.com"
}
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| domain_type | number | Có | - | Phải là 1 hoặc 2 | Loại tên miền (1: Subdomain, 2: Custom Domain) |
| domain_name | string | Có | 255 | Không được rỗng. Đúng định dạng tương ứng loại tên miền | Tên miền thiết lập (FQDN hoặc phần tiền tố) |

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
    "domain_settings": {
      "domain_type": 2,
      "domain_name": "jobs.sakicare.com",
      "dns_status": 0,
      "ssl_status": 0,
      "last_checked_at": null
    }
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant_id | string | ID định danh duy nhất của Tenant (ULID) |
| data.domain_settings | object | Cấu hình tên miền sau khi thiết lập |
| data.domain_settings.domain_type | number | Loại tên miền (1: Subdomain, 2: Custom Domain) |
| data.domain_settings.domain_name | string | Tên miền đầy đủ thiết lập (FQDN) |
| data.domain_settings.dns_status | number | Trạng thái DNS (0: 未検証 - Chưa xác minh) |
| data.domain_settings.ssl_status | number | Trạng thái chứng chỉ SSL (0: 未適用 - Chưa cấp phát) |
| data.domain_settings.last_checked_at | string | Thời gian kiểm tra gần nhất (mặc định null sau khi cập nhật) |

---

## 6.3 Error Response

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
        "field": "domain_name",
        "message": "The custom domain format is invalid."
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

### Forbidden

```
403 Forbidden
```

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to perform this operation."
  }
}
```

### Not Found

```
404 Not Found
```

```json
{
  "error": {
    "code": "TENANT_NOT_FOUND",
    "message": "Tenant not found."
  }
}
```

### State Conflict / Duplicate Value

```
409 Conflict
```

```json
{
  "error": {
    "code": "DOMAIN_ALREADY_EXISTS",
    "message": "The domain name has already been registered by another tenant."
  }
}
```

### System Error

```
500 Internal Server Error
```

```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred."
  }
}
```

---

# 7. Validation Rules

| No. | Field | Rule | Error Code | Error Message |
| --- | --- | --- | --- | --- |
| 1 | id | Bắt buộc, phải đúng định dạng ULID (26 ký tự) | CMS-VAL-23 | idを入力してください。 |
| 2 | domain_type | Bắt buộc, giá trị phải là 1 (Subdomain) hoặc 2 (Custom Domain) | CMS-VAL-23 | domain_typeを入力してください。 |
| 3 | domain_name | Bắt buộc nhập, tối đa 255 ký tự | CMS-VAL-23 | domain_nameを入力してください。 |
| 4 | domain_name | Nếu domain_type = 1: Chỉ cho phép chữ thường a-z, số 0-9 và dấu gạch ngang -, giới hạn 2-63 ký tự | CMS-VAL-27 | domain_nameは数値で入力してください。 |
| 5 | domain_name | Nếu domain_type = 2: Đúng định dạng tên miền FQDN hợp lệ | INVALID_CUSTOM_DOMAIN_FORMAT | Custom domain format is invalid. |
| 6 | domain_name | Tên miền đầy đủ (bao gồm cả phần sinh tự động suffix .carima.link đối với loại Subdomain) phải là duy nhất và chưa được đăng ký bởi bất kỳ Tenant nào khác trong hệ thống | CMS-VAL-11 | domain_nameの値は既に存在しています。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Role authorization | Chỉ tài khoản người dùng thuộc nhóm quản trị Platform (SaaS Admin) có quyền `tenant.update` mới được phép thực hiện gọi API này. Platform SaaS Staff bị từ chối truy cập (HTTP 403). |
| BR-002 | Subdomain domain suffix auto-generation | Trường hợp `domain_type = 1` (Subdomain), hệ thống sẽ tự động ghép phần tiền tố nhập vào với phần hậu tố cố định của nền tảng (ví dụ: `.carima.link`) để lưu trữ giá trị tên miền FQDN hoàn chỉnh vào cột `domain_name`. |
| BR-003 | Connection status initialization | Khi cấu hình tên miền của một Tenant thay đổi hoặc được thiết lập mới, hệ thống bắt buộc phải khởi tạo lại trạng thái DNS và SSL về `0` (`dns_status = 0` - Chưa xác minh và `ssl_status = 0` - Chưa cấp phát) để đưa vào hàng đợi kiểm tra tự động của tiến trình nền hệ thống. |
| BR-004 | Audit log recording | Mọi thao tác cập nhật cấu hình tên miền của Tenant đều phải ghi nhận chi tiết nhật ký thao tác (Audit Log) vào bảng `platform.tenant_provision_log` để phục vụ công tác giám sát hệ thống. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Update |
| --- | --- | --- |
| Platform SaaS Admin | Có | Có |
| Platform SaaS Staff | Có | Không |
| Tenant Admin (MOTO/SAKI) | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Xác thực token JWT gửi kèm trong header Authorization.
2. Trích xuất thông tin tài khoản người dùng và xác minh thuộc nhóm Platform SaaS Admin.
3. Kiểm tra quyền hạn "tenant.update" của tài khoản.
4. Nếu hợp lệ, cho phép đi tiếp để xử lý logic API.
5. Nếu không hợp lệ, từ chối yêu cầu và trả về lỗi HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Logic xử lý / Mapping |
| --- | --- | --- | --- |
| id | platform.tenant_registry | tenant_id | Xác định Tenant đích cần cập nhật |
| domain_type | platform.tenant_domain | domain_type | Lưu loại tên miền (1 hoặc 2) |
| domain_name | platform.tenant_domain | domain_name | - Nếu domain_type = 1: Ghép chuỗi `:domain_name` + '.carima.link'<br>- Nếu domain_type = 2: Lưu trực tiếp `:domain_name` |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_domain | tenant_id | data.tenant_id | ID định danh của Tenant |
| platform.tenant_domain | domain_type | data.domain_settings.domain_type | Loại tên miền |
| platform.tenant_domain | domain_name | data.domain_settings.domain_name | Tên miền đầy đủ thiết lập |
| platform.tenant_domain | dns_status | data.domain_settings.dns_status | Trạng thái xác minh DNS |
| platform.tenant_domain | ssl_status | data.domain_settings.ssl_status | Trạng thái chứng chỉ SSL |
| platform.tenant_domain | last_checked_at | data.domain_settings.last_checked_at | Thời gian kiểm tra gần nhất |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request data & headers | Ngoài transaction |
| 2 | Check permission platform user | Ngoài transaction |
| 3 | SELECT FOR UPDATE dòng tenant trong `platform.tenant_registry` | Trong transaction |
| 4 | SELECT kiểm tra trùng lặp `domain_name` trong `platform.tenant_domain` | Trong transaction |
| 5 | UPSERT (INSERT hoặc UPDATE) thông tin cấu hình vào `platform.tenant_domain` | Trong transaction |
| 6 | INSERT bản ghi hành động vào `platform.tenant_provision_log` | Trong transaction |
| 7 | Commit transaction | Trong transaction |
| 8 | Đẩy sự kiện kiểm tra SSL/DNS vào hàng đợi xử lý ngầm (Queue Job) | Ngoài transaction (sau commit) |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Không tìm thấy Tenant tương ứng trong `tenant_registry` | Không khởi động transaction, trả về lỗi 404 Not Found |
| 2 | Trùng tên miền đầy đủ thiết lập với Tenant khác | Không khởi động/rollback transaction, trả về lỗi 409 Conflict |
| 3 | Thao tác ghi/cập nhật bảng `platform.tenant_domain` thất bại | Rollback toàn bộ DB changes |
| 4 | Thao tác ghi log bảng `platform.tenant_provision_log` thất bại | Rollback toàn bộ DB changes |
| 5 | Bất kỳ ngoại lệ Exception hệ thống nào phát sinh | Rollback toàn bộ DB changes |

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: PATCH /api/v1/admin/tenants/{id}/domains kèm theo Request Body.
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization) của Platform SaaS Admin (quyền "tenant.update").
4. Controller validate các tham số đầu vào:
   - Tham số Path Parameter `id` (ULID format).
   - Tham số Request Body `domain_type` (1 hoặc 2) và `domain_name` (đúng định dạng tương ứng).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm mô tả chi tiết lỗi.
5. Controller gọi Service xử lý nghiệp vụ cập nhật thông tin tên miền trong Database Transaction:
   - SELECT bản ghi của Tenant trong bảng `platform.tenant_registry` kèm khóa Lock (`SELECT FOR UPDATE`).
     - Nếu không tìm thấy: Throw Exception -> Trả về HTTP 404 Not Found.
   - Xây dựng chuỗi tên miền đầy đủ FQDN (ghép suffix nếu type = 1).
   - Truy vấn bảng `platform.tenant_domain` để kiểm tra trùng lặp tên miền với tenant_id khác:
     - Nếu đã tồn tại bản ghi có domain_name trùng và tenant_id khác: Throw Exception -> Trả về HTTP 409 Conflict.
   - Thực hiện cập nhật hoặc tạo mới (UPSERT) dữ liệu cấu hình tên miền vào bảng `platform.tenant_domain`:
     - Thiết lập domain_type, domain_name.
     - Đặt giá trị mặc định dns_status = 0 (Chưa xác minh), ssl_status = 0 (Chưa cấp phát).
     - Cập nhật thời gian last_checked_at = NULL.
   - Ghi nhận Audit Log hành động vào bảng `platform.tenant_provision_log` với hành động `update_domain`.
6. Transaction được Commit thành công.
7. Đẩy job xác minh kết nối DNS và cấp phát SSL tự động cho tên miền mới vào hàng đợi hệ thống xử lý ngầm (Queue/Event).
8. Controller định dạng response qua API Resource và trả về kết quả thành công HTTP 200 OK.
```

---

# 14. Notification

Không áp dụng cho API này.

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | UPDATE |
| Bảng target | platform.tenant_domain |
| ID record target | tenant_id |
| Dữ liệu ghi log | user_id, action = 'update_domain', detail = JSON chứa cấu hình domain mới |
| Chính sách lưu trữ | Ghi vào bảng `platform.tenant_provision_log` |

---

# 16. Xử lý file

Không áp dụng.

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc xác thực JWT Token hợp lệ và kiểm tra quyền admin quản trị Platform (`tenant.update`). |
| 2 | Input Sanitization | Thực hiện làm sạch và validate ký tự đầu vào của tên miền (ngăn chặn ký tự đặc biệt, SQL injection, Command injection qua tên miền). |
| 3 | Domain hijacking prevention | Chỉ cho phép cập nhật tên miền duy nhất trên toàn hệ thống, ngăn chặn một tenant cấu hình đè hoặc cướp tên miền của tenant khác. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 1.5 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Không áp dụng |
| Index cần thiết | Index cột `tenant_id` và `domain_name` trên bảng `platform.tenant_domain` |
| Cache cần thiết | Cache thông tin routing tên miền (sau khi DNS/SSL được xác minh thành công) để Router/Reverse Proxy tra cứu nhanh chóng |
| Bulk Processing | Không |
| Async Processing | Có (Tác vụ kiểm tra DNS và xin cấp phát chứng chỉ SSL thực hiện bất đồng bộ) |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Giao diện liên quan | PA-TEN-014 Cấu hình Domain Tenant |
| Thiết kế màn hình | spec-screen/PA-TEN-014.md |
| ERD | Carima-Staffing Data Dictionary / ERD |
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
