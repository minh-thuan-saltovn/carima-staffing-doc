# PA-TEN-018-API-01-Additional Onboarding Procedure

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Additional Onboarding Procedure |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.2 |
| Ngày tạo | 2026/06/18 |
| Ngày cập nhật | 2026/06/18 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/18 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Additional Onboarding Procedure theo yêu cầu Notion và ERD PostgreSQL |
| v1.1 | 2026/06/18 | Thuận | Thiết kế dựa trên giả định từ màn hình |
| v1.2 | 2026/06/18 | Thuận | Cập nhật lại toàn bộ tài liệu: Loại bỏ hoàn toàn các trường thông tin giả định từ màn hình thiết kế bị sai lệch (như modules, users, storage). Thiết kế lại API tinh gọn, chỉ thực hiện nhiệm vụ khởi tạo/tạo mới quy trình onboarding bổ sung trong cơ sở dữ liệu bám sát ERD. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-018-API-01 |
| Tên API | Additional Onboarding Procedure |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-018 |
| Chức năng liên quan | Tenant Management (Additional Onboarding) |
| Mục đích API | Tạo mới một quy trình onboarding bổ sung cho Tenant trong cơ sở dữ liệu để theo dõi tiến độ chuẩn bị. |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/admin/tenants/{id}/additional-onboarding |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để khởi tạo (tạo mới) một quy trình onboarding bổ sung (Additional Onboarding Procedure) cho Tenant trong bảng quản lý trạng thái `platform.tenant_onboarding` (TBD). Quy trình này giúp hệ thống ghi nhận và theo dõi một lượt triển khai/cấu hình mới của Tenant sau khi đã đi vào hoạt động chính thức.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Quản trị viên Platform click nút khởi tạo quy trình onboarding bổ sung cho một Tenant cụ thể. |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Tenant đã tồn tại trên hệ thống và đang hoạt động. |
| Sau khi gọi API | Tạo mới một lượt ghi nhận tiến độ onboarding với loại quy trình là bổ sung (Additional), ghi Audit Log. |
| Thay đổi trạng thái liên quan | Thêm mới bản ghi vào bảng `platform.tenant_onboarding` (TBD). |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/admin/tenants/{id}/additional-onboarding
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
| id | string | Có | ID của Tenant (ULID) | 01H2PJX7G3P681729X4R09Q872 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "onboarding_type": 2,
  "notes": "Khởi tạo quy trình cấu hình tính năng bổ sung cho Tenant."
}
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| onboarding_type | integer | Có | - | Phải là giá trị: 2 | Loại onboarding (1: Onboarding ban đầu, 2: Onboarding bổ sung) |
| notes | string | Không | 500 | Tối đa 500 ký tự | Ghi chú lý do khởi tạo quy trình |

---

# 6. Response Definition

## 6.1 Success Response

### HTTP Status

```
201 Created
```

### Response Body

```json
{
  "data": {
    "onboarding_id": "01H2PJX7G3P681729X4R09Q879",
    "tenant_id": "01H2PJX7G3P681729X4R09Q872",
    "onboarding_type": 2,
    "status": 1,
    "notes": "Khởi tạo quy trình cấu hình tính năng bổ sung cho Tenant.",
    "created_at": "2026-06-18T11:52:37+07:00"
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.onboarding_id | string | ID định danh duy nhất của lượt onboarding bổ sung vừa tạo (ULID) |
| data.tenant_id | string | ID định danh của Tenant |
| data.onboarding_type | integer | Loại onboarding (2: Onboarding bổ sung) |
| data.status | integer | Trạng thái quy trình (1: 準備中 - Preparing) |
| data.notes | string | Ghi chú |
| data.created_at | string | Thời gian khởi tạo |

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
    "message": "Validation failed."
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
| 1 | id | Bắt buộc, đúng định dạng ULID (26 ký tự) | CMS-VAL-23 | IDを入力してください。 |
| 2 | onboarding_type | Bắt buộc, phải bằng 2 | CMS-VAL-23 | オンボーディングタイプを入力してください。 |
| 3 | notes | Tối đa 500 ký tự | CMS-VAL-6 | 備考は500文字以内で入力してください。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Role authorization | Chỉ Platform SaaS Admin có quyền `tenant.update` mới được phép khởi tạo quy trình này. |
| BR-002 | Status Initialization | Quy trình onboarding bổ sung mới tạo luôn được đặt trạng thái mặc định ban đầu là `1` (準備中 - Preparing). |
| BR-003 | Audit Log | Thao tác khởi tạo quy trình phải ghi nhận nhật ký vào `platform.tenant_provision_log`. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | Create |
| --- | --- |
| Platform SaaS Admin | Có |
| Platform SaaS Staff | Không |
| Tenant Admin (MOTO/SAKI) | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Xác thực token JWT gửi kèm trong header Authorization.
2. Xác minh tài khoản thuộc nhóm Platform SaaS Admin.
3. Kiểm tra quyền "tenant.update".
4. Nếu hợp lệ, cho phép tiếp tục xử lý.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Logic xử lý / Mapping |
| --- | --- | --- | --- |
| id | platform.tenant_registry | tenant_id | Xác định Tenant |
| onboarding_type | platform.tenant_onboarding (TBD) | onboarding_type | Lưu loại quy trình |
| notes | platform.tenant_onboarding (TBD) | notes | Lưu ghi chú |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_onboarding (TBD) | onboarding_id | data.onboarding_id | ID quy trình vừa tạo |
| platform.tenant_onboarding (TBD) | tenant_id | data.tenant_id | ID Tenant |
| platform.tenant_onboarding (TBD) | onboarding_type | data.onboarding_type | Loại onboarding |
| platform.tenant_onboarding (TBD) | status | data.status | Trạng thái quy trình |
| platform.tenant_onboarding (TBD) | notes | data.notes | Ghi chú |
| platform.tenant_onboarding (TBD) | created_at | data.created_at | Thời gian tạo |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request data & headers | Ngoài transaction |
| 2 | Check permission platform user | Ngoài transaction |
| 3 | SELECT FOR UPDATE dòng tenant trong `platform.tenant_registry` | Trong transaction |
| 4 | INSERT bản ghi yêu cầu mới vào `platform.tenant_onboarding` (TBD) | Trong transaction |
| 5 | INSERT bản ghi hành động vào `platform.tenant_provision_log` | Trong transaction |
| 6 | Commit transaction | Trong transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Không tìm thấy Tenant tương ứng trong `tenant_registry` | Không khởi động transaction, trả về lỗi 404 Not Found |
| 2 | Thao tác ghi dữ liệu vào bảng `platform.tenant_onboarding` (TBD) thất bại | Rollback toàn bộ DB changes |
| 3 | Thao tác ghi log bảng `platform.tenant_provision_log` thất bại | Rollback toàn bộ DB changes |
| 4 | Bất kỳ ngoại lệ Exception hệ thống nào phát sinh | Rollback toàn bộ DB changes |

---

# 12. Status Transition

Không áp dụng (Chỉ khởi tạo bản ghi mới).

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: POST /api/v1/admin/tenants/{id}/additional-onboarding kèm theo Request Body.
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization) của Platform SaaS Admin.
4. Controller validate các tham số đầu vào.
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity.
5. Controller gọi Service xử lý nghiệp vụ trong Database Transaction:
   - SELECT bản ghi của Tenant trong bảng `platform.tenant_registry` kèm khóa Lock (`SELECT FOR UPDATE`).
     - Nếu không tìm thấy: Throw Exception -> Trả về HTTP 404 Not Found.
   - Thực hiện sinh ID mới (ULID) và INSERT bản ghi quy trình onboarding bổ sung mới vào bảng `platform.tenant_onboarding` (TBD).
   - Ghi nhận Audit Log hành động vào bảng `platform.tenant_provision_log` với hành động `create_additional_onboarding`.
6. Transaction được Commit thành công.
7. Controller trả response JSON kết quả bản ghi vừa tạo kèm HTTP 201 Created.
```

---

# 14. Notification

Không áp dụng.

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | CREATE |
| Bảng target | platform.tenant_onboarding (TBD) |
| ID record target | onboarding_id |
| Dữ liệu ghi log | user_id, action = 'create_additional_onboarding', detail = JSON chứa thông tin yêu cầu |
| Chính sách lưu trữ | Ghi vào bảng `platform.tenant_provision_log` |

---

# 16. Xử lý file

Không áp dụng.

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc xác thực token JWT và kiểm tra quyền Platform SaaS Admin (`tenant.update`). |
| 2 | Input Validation | Validate giới hạn độ dài ký tự notes, trị số onboarding_type hợp lệ. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.5 giây |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Không áp dụng |
| Index cần thiết | Index cột `tenant_id` trên bảng `platform.tenant_onboarding` (TBD) |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Giao diện liên quan | PA-TEN-018 Đăng ký bổ sung tính năng Tenant |
| Thiết kế màn hình | spec-screen/PA-TEN-018.md (Chỉ tham chiếu luồng nghiệp vụ) |
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
| 11 | Status transition đã được định nghĩa | Done (Không áp dụng) |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
