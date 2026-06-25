# PA-TEN-017-API-01-Tenant Onboarding Status

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Tenant Onboarding Status |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.1 |
| Ngày tạo | 2026/06/18 |
| Ngày cập nhật | 2026/06/18 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/18 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Tenant Onboarding Status theo thiết kế màn hình PA-TEN-017 |
| v1.1 | 2026/06/18 | Thuận | Điều chỉnh bám sát ERD và Notion API List; loại bỏ hoàn toàn logic cập nhật (đặc tả duy nhất hành động truy vấn/tính toán trạng thái) |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-017-API-01 |
| Tên API | Tenant Onboarding Status |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-017 |
| Chức năng liên quan | Tenant Management (Onboarding Status) |
| Mục đích API | Truy vấn, tính toán động và trả về tổng quan trạng thái chuẩn bị triển khai (Onboarding Status) của một Tenant dựa trên dữ liệu hệ thống. |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/admin/tenants/{id}/onboarding |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để lấy tổng quan tiến độ thiết lập hệ thống (Checklist) của một Tenant mới tạo. Khi được gọi dưới dạng `POST`, hệ thống sẽ tự động quét, truy vấn và đánh giá trạng thái hiện tại của các hạng mục thiết lập bắt buộc trong cơ sở dữ liệu để trả về kết quả checklist thực tế.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Quản trị viên truy cập màn hình giám sát tiến độ triển khai Tenant. |
| Actor | Platform SaaS Admin / Platform SaaS Staff |
| Trước khi gọi API | Tenant đã tồn tại trong hệ thống. |
| Sau khi gọi API | Trả về thông tin tiến độ checklist 5 bước và trạng thái tổng quan để hiển thị cho quản trị viên. |
| Thay đổi trạng thái liên quan | Không (Đây là API chỉ đọc/tính toán động, không thay đổi dữ liệu). |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/admin/tenants/{id}/onboarding
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
| id | string | Có | ID của Tenant cần truy vấn trạng thái (ULID) | 01H2PJX7G3P681729X4R09Q872 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

Không áp dụng (Request Body gửi rỗng `{}`).

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
    "tenant_code": "SAKI-0001",
    "company_name": "株式会社サキケア",
    "tenant_type": "saki",
    "checklist": {
      "step_account_created": 1,
      "step_features_configured": 1,
      "step_domain_verified": 1,
      "step_branding_configured": 0,
      "step_master_data_imported": 0
    },
    "onboarding_status": 1
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant_id | string | ID định danh của Tenant (ULID) |
| data.tenant_code | string | Mã công ty (business key) của Tenant — từ `platform.tenant_registry.company_id` |
| data.company_name | string | Tên công ty của Tenant — từ `platform.tenant_registry.company_name` |
| data.tenant_type | string | Loại Tenant (`saki` hoặc `moto`) — từ `platform.tenant_registry.tenant_type` |
| data.checklist | object | Kết quả tự động đánh giá tiến trình thiết lập (0: Chưa hoàn thành, 1: Hoàn thành) |
| data.checklist.step_account_created | integer | Bước 1: Tạo Tenant & Tài khoản Admin đầu tiên của Tenant |
| data.checklist.step_features_configured | integer | Bước 2: Cấu hình gói dịch vụ (Plan) |
| data.checklist.step_domain_verified | integer | Bước 3: Cấu hình và xác minh tên miền DNS & SSL |
| data.checklist.step_branding_configured | integer | Bước 4: Cấu hình Branding thương hiệu |
| data.checklist.step_master_data_imported | integer | Bước 5: Nhập dữ liệu master ban đầu |
| data.onboarding_status | integer | Trạng thái Tenant tổng thể — từ `platform.tenant_registry.status` (0: Vô hiệu, 1: Đang hoạt động) hoặc trạng thái triển khai [TBD] |

---
### 6.3 Error Response - Validation Error

### 6.4 Error Response - Invalid Credentials

HTTP Status: 401 Unauthorized

### 6.5 Error Response - Account Locked

HTTP Status: 403 Forbidden

### 6.6 Error Response - Account Inactive

### 6.7 Error Response - Tenant Not Found

HTTP Status: 404 Not Found

### 6.8 Error Response - System Error

HTTP Status: 500 Internal Server Error

### 6.9 Error Response - Bad Request

HTTP Status: 400 Bad Request

### 6.10 Error Response - Method Not Allowed

HTTP Status: 405 Method Not Allowed

### 6.11 Error Response - Too Many Requests

HTTP Status: 429 Too Many Requests

### 6.12 Error Response - Service Unavailable

HTTP Status: 503 Service Unavailable

### 6.13 Error Response - Gateway Timeout

HTTP Status: 504 Gateway Timeout

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
| BR-001 | Role authorization | Cả Platform SaaS Admin và Platform SaaS Staff có quyền `tenant.view` đều được gọi API này. |
| BR-002 | Checklist evaluation logic | Tiến trình checklist 5 bước được tính toán động như sau:<br>- **Bước 1 (step_account_created)**: Trả về `1` nếu tồn tại ít nhất 1 tài khoản người dùng trong schema của Tenant tương ứng (`mst_saki_user` với SAKI hoặc `mst_moto_user` với MOTO). Ngược lại trả về `0`.<br>- **Bước 2 (step_features_configured)**: Trả về `1` nếu trường `plan_code` trong bảng `platform.tenant_registry` của Tenant khác NULL. Ngược lại trả về `0`.<br>- **Bước 3 (step_domain_verified)**: Trả về `1` nếu tồn tại bản ghi trong bảng cấu hình domain `platform.tenant_domain` (TBD) có trạng thái `dns_status = 1` và `ssl_status = 1`. Ngược lại trả về `0`.<br>- **Bước 4 (step_branding_configured)**: Trả về `1` nếu các cấu hình branding tương ứng đã được thiết lập (TBD).<br>- **Bước 5 (step_master_data_imported)**: Trả về trạng thái nhập master data từ bảng theo dõi `platform.tenant_onboarding` (TBD). Mặc định trả về `0` nếu chưa cấu hình. |
| BR-003 | No DB modification | API này chỉ thực hiện truy vấn và tính toán động, tuyệt đối không thay đổi trạng thái hoặc chỉnh sửa dữ liệu trong cơ sở dữ liệu. |

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
2. Xác minh tài khoản thuộc nhóm Platform SaaS Admin hoặc Platform SaaS Staff.
3. Kiểm tra quyền "tenant.view".
4. Nếu hợp lệ, cho phép tiếp tục xử lý.
5. Nếu không hợp lệ, từ chối yêu cầu và trả về lỗi HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

Không áp dụng (API chỉ đọc, không ghi nhận thông tin đầu vào).

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | data.tenant_id | ID định danh duy nhất của Tenant |
| platform.tenant_registry | company_id | data.tenant_code | Mã Tenant |
| platform.tenant_registry | company_name | data.company_name | Tên công ty của Tenant |
| platform.tenant_registry | tenant_type | data.tenant_type | Loại Tenant |
| platform.tenant_registry | status | data.onboarding_status | Trạng thái Tenant |
| platform.tenant_onboarding (TBD) | master_data_imported | data.checklist.step_master_data_imported | Trạng thái bước 5 |

---

# 11. Database Transaction

Không áp dụng (API chỉ đọc, không có ghi cơ sở dữ liệu).

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: POST /api/v1/admin/tenants/{id}/onboarding (Request Body rỗng {}).
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization) của Platform SaaS Admin/Staff (quyền "tenant.view").
4. Controller validate Path Parameter `id` (ULID format).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity.
5. Service truy vấn thông tin Tenant trong bảng `platform.tenant_registry` theo `id`.
   - Nếu không tìm thấy: Trả về HTTP 404 Not Found.
6. Service thực hiện các câu lệnh SELECT để kiểm tra dữ liệu cấu hình thực tế của Tenant:
   - SELECT COUNT từ `mst_saki_user`/`mst_moto_user` trong schema riêng của Tenant để kiểm tra bước 1.
   - Kiểm tra `plan_code` trong `platform.tenant_registry` để đánh giá bước 2.
   - SELECT kiểm tra trạng thái domain từ `platform.tenant_domain` (TBD) để đánh giá bước 3.
   - Kiểm tra thông tin branding (TBD) để đánh giá bước 4.
   - SELECT trạng thái từ `platform.tenant_onboarding` (TBD) để lấy giá trị bước 5.
7. Service tổng hợp tiến độ và trạng thái, chuyển dữ liệu qua API Resource.
8. Controller trả response JSON kết quả kèm HTTP 200 OK.
```

---

# 14. Notification

Không áp dụng.

---

# 15. Audit Log

Không áp dụng (API chỉ đọc, không ghi log thao tác dữ liệu).

---

# 16. Xử lý file

Không áp dụng.

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc xác thực token JWT và kiểm tra quyền `tenant.view`. |
| 2 | Tenant Isolation | Khi truy vấn dữ liệu chi tiết của user bên trong schema tenant để đánh giá bước 1, hệ thống phải kết nối đúng schema của Tenant tương ứng (`tenant_saki_{id}` hoặc `tenant_moto_{id}`), ngăn chặn việc kết nối chéo schema. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.5 giây |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Không áp dụng |
| Index cần thiết | Index khoá ngoại `tenant_id` trên các bảng cấu hình phụ (`platform.tenant_onboarding`, `platform.tenant_domain`) |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Giao diện liên quan | PA-TEN-017 Tình trạng chuẩn bị triển khai |
| Thiết kế màn hình | spec-screen/PA-TEN-017.md (Tham chiếu loại bỏ phần lưu cấu hình) |
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
| 10 | Transaction scope đã được định nghĩa | Done (Không áp dụng) |
| 11 | Status transition đã được định nghĩa | Done (Không áp dụng) |
| 12 | Audit log đã được định nghĩa | Done (Không áp dụng) |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
