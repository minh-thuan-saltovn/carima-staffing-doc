# PA-TEN-004-API-01-Update moto tenant

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Update moto tenant |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.2 |
| Ngày tạo | 2026/06/22 |
| Ngày cập nhật | 2026/06/22 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/22 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Update moto tenant (PATCH) theo đúng ERD cơ sở dữ liệu vật lý, không có trường ảo. |
| v1.1 | 2026/06/22 | Thuận | Cập nhật Response thành công trả về đầy đủ thông tin chi tiết qua TenantResource. |
| v1.2 | 2026/06/22 | Thuận | Cấu trúc lại Response thành công: Chỉ trả về chính xác các trường thông tin vừa được cập nhật (plan_code, company_profile, admin_user) theo yêu cầu tối giản dữ liệu trả về của user. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-004-API-01 |
| Tên API | Update moto tenant |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-004 |
| Chức năng liên quan | Tenant Management (Edit MOTO Tenant) |
| Mục đích API | Cập nhật thông tin đăng ký Tenant MOTO (như gói dịch vụ, thông tin doanh nghiệp phái cử, thông tin người dùng quản trị) trong Database và trả lại các trường thông tin vừa được cập nhật. |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/admin/moto-tenants/{id} |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Dynamic Tenant Schema |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này được gọi khi Platform SaaS Admin bấm nút "更新する" (Cập nhật) trên màn hình Chỉnh sửa MOTO Tenant (`PA-TEN-004`). API sẽ thực hiện:
1. Cập nhật gói dịch vụ (`plan_code`) trong bảng `platform.tenant_registry` của platform schema.
2. Dựa vào `schema_name` của Tenant, thiết lập kết nối động tới tenant schema tương ứng (`tenant_moto_<company_id>`).
3. Cập nhật thông tin doanh nghiệp trong bảng `mst_moto_company`.
4. Cập nhật thông tin người dùng quản trị (Admin User) trong bảng `mst_moto_user` của schema Tenant.
5. Toàn bộ tiến trình cập nhật dữ liệu ở schema Tenant phải được thực hiện trong Database Transaction.
6. Sau khi cập nhật thành công, API trả về chính xác các thông tin vừa cập nhật.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Quản trị viên click nút "Cập nhật" (更新する) trên giao diện. |
| Actor | Platform SaaS Admin / Platform SaaS Staff |
| Trước khi gọi API | Tenant MOTO đã tồn tại, dữ liệu nhập hợp lệ. |
| Sau khi gọi API | Dữ liệu được lưu trữ, hệ thống trả về thông báo thành công cùng các trường thông tin vừa được cập nhật. |
| Thay đổi trạng thái liên quan | Cập nhật các trường thông tin tương ứng. |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/admin/moto-tenants/{id}
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Định dạng dữ liệu |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | ID của Tenant MOTO (ULID) | 01H2PJX7G3P681729X4R09Q871 |

---

## 4.4 Query Parameters

| Parameter | Type | Bắt buộc | Default | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| - | - | - | - | Không áp dụng | - |

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "plan_code": "ENTERPRISE",
  "company_profile": {
    "official_name_ja": "MOTO Staffing Solutions 株式会社",
    "display_name_ja": "MOTO Staffing",
    "postal_code": "1000005",
    "tel": "03-1234-5678",
    "address_ja": "東京都千代田区丸の内1-1 MOTOビル3F"
  },
  "admin_user": {
    "last_name_ja": "山田",
    "first_name_ja": "太郎",
    "email": "yamada@moto-staffing.jp",
    "tel": "090-1234-5678"
  }
}
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| plan_code | string | Có | 20 | Tồn tại trong danh mục plan | Gói dịch vụ đăng ký (`platform.tenant_registry`) |
| company_profile | object | Có | - | - | Thông tin hồ sơ doanh nghiệp |
| company_profile.official_name_ja | string | Có | 100 | Không rỗng | Tên chính thức công ty (tiếng Nhật) |
| company_profile.display_name_ja | string | Có | 24 | Không rỗng | Tên hiển thị công ty (tiếng Nhật) |
| company_profile.postal_code | string | Có | 7 | 7 chữ số Half-width | Mã bưu chính |
| company_profile.tel | string | Có | 15 | Số điện thoại Half-width | Số điện thoại của công ty |
| company_profile.address_ja | string | Có | 100 | Không rỗng | Địa chỉ công ty |
| admin_user | object | Có | - | - | Thông tin tài khoản quản trị viên của Tenant |
| admin_user.last_name_ja | string | Có | 24 | Không rỗng | Họ của quản trị viên (tiếng Nhật) |
| admin_user.first_name_ja | string | Có | 24 | Không rỗng | Tên của quản trị viên (tiếng Nhật) |
| admin_user.email | string | Có | 128 | Định dạng Email | Email người dùng |
| admin_user.tel | string | Có | 15 | Số điện thoại Half-width | Số điện thoại liên hệ của quản trị viên |

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
    "plan_code": "ENTERPRISE",
    "company_profile": {
      "official_name_ja": "MOTO Staffing Solutions 株式会社",
      "display_name_ja": "MOTO Staffing",
      "postal_code": "1000005",
      "tel": "03-1234-5678",
      "address_ja": "東京都千代田区丸の内1-1 MOTOビル3F"
    },
    "admin_user": {
      "last_name_ja": "山田",
      "first_name_ja": "太郎",
      "email": "yamada@moto-staffing.jp",
      "tel": "090-1234-5678"
    }
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.plan_code | string | Gói dịch vụ đăng ký mới |
| **data.company_profile** | object | Thông tin doanh nghiệp vừa cập nhật |
| data.company_profile.official_name_ja | string | Tên chính thức công ty (tiếng Nhật) |
| data.company_profile.display_name_ja | string | Tên hiển thị công ty (tiếng Nhật) |
| data.company_profile.postal_code | string | Mã bưu chính (7 chữ số) |
| data.company_profile.tel | string | Số điện thoại của công ty |
| data.company_profile.address_ja | string | Địa chỉ công ty |
| **data.admin_user** | object | Thông tin tài khoản quản trị vừa cập nhật |
| data.admin_user.last_name_ja | string | Họ của quản trị viên (tiếng Nhật) |
| data.admin_user.first_name_ja | string | Tên của quản trị viên (tiếng Nhật) |
| data.admin_user.email | string | Email người dùng |
| data.admin_user.tel | string | Số điện thoại liên hệ của quản trị viên |

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
| 3 | plan_code | required | - | required |
| 4 | plan_code | max | 20 | max |
| 5 | company_profile.official_name_ja | required | - | required |
| 6 | company_profile.official_name_ja | max | 100 | max |
| 7 | company_profile.display_name_ja | required | - | required |
| 8 | company_profile.display_name_ja | max | 24 | max |
| 9 | company_profile.postal_code | required | - | required |
| 10 | company_profile.postal_code | digits | 7 | digits |
| 11 | company_profile.tel | required | - | required |
| 12 | company_profile.tel | regex | /^[0-9-]+$/ | regex |
| 13 | company_profile.tel | max | 15 | max |
| 14 | company_profile.address_ja | required | - | required |
| 15 | company_profile.address_ja | max | 100 | max |
| 16 | admin_user.last_name_ja | required | - | required |
| 17 | admin_user.last_name_ja | max | 24 | max |
| 18 | admin_user.first_name_ja | required | - | required |
| 19 | admin_user.first_name_ja | max | 24 | max |
| 20 | admin_user.email | required | - | required |
| 21 | admin_user.email | email | - | email |
| 22 | admin_user.email | max | 128 | max |
| 23 | admin_user.tel | required | - | required |
| 24 | admin_user.tel | regex | /^[0-9-]+$/ | regex |
| 25 | admin_user.tel | max | 15 | max |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Role authorization | Chỉ Platform SaaS Admin hoặc Platform SaaS Staff có quyền `tenant.edit` mới được phép gọi API này. |
| BR-002 | Target validation | Xác minh xem `id` (ULID) có tồn tại trong bảng `platform.tenant_registry` với `tenant_type = 'MOTO'` (hoặc 2) hay không. |
| BR-003 | Dynamic Connection | Sử dụng `schema_name` lấy từ bảng registry để thiết lập kết nối động tới tenant schema để thực hiện update thông tin. |
| BR-004 | Update Target User / Contacts | Cập nhật thông tin admin user trong schema Tenant (xác định qua `execution_role_id = 'role_admin'` hoặc user đầu tiên). Đồng thời, cập nhật số điện thoại của công ty (`company_profile.tel`) vào bảng `mst_moto_company_contact` cho bản ghi liên hệ chính đầu tiên (`seq_no = 1`). |
| BR-005 | DB Transaction | Tiến trình cập nhật trong Tenant DB bắt buộc phải bọc trong transaction. Nếu xảy ra lỗi ở bất kỳ bước nào, rollback toàn bộ thay đổi. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | Edit |
| --- | --- |
| Platform SaaS Admin | Có |
| Platform SaaS Staff | Có |
| Tenant Admin (MOTO/SAKI) | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Xác thực token JWT gửi kèm trong header Authorization.
2. Xác minh tài khoản thuộc nhóm Platform SaaS Admin hoặc Platform SaaS Staff.
3. Kiểm tra quyền "tenant.edit".
4. Nếu hợp lệ, cho phép tiếp tục xử lý.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Mô tả |
| --- | --- | --- | --- |
| plan_code | platform.tenant_registry | plan_code | Gói dịch vụ đăng ký |
| company_profile.official_name_ja | tenant_moto.mst_moto_company | official_name_ja | Tên chính thức công ty (tiếng Nhật) |
| company_profile.display_name_ja | tenant_moto.mst_moto_company | display_name_ja | Tên hiển thị công ty (tiếng Nhật) |
| company_profile.postal_code | tenant_moto.mst_moto_company | postal_code | Mã bưu chính |
| company_profile.tel | tenant_moto.mst_moto_company_contact | tel | Số điện thoại của công ty (với `seq_no = 1`) |
| company_profile.address_ja | tenant_moto.mst_moto_company | address_ja | Địa chỉ công ty |
| admin_user.last_name_ja | tenant_moto.mst_moto_user | last_name_ja | Họ của quản trị viên (tiếng Nhật) |
| admin_user.first_name_ja | tenant_moto.mst_moto_user | first_name_ja | Tên của quản trị viên (tiếng Nhật) |
| admin_user.email | tenant_moto.mst_moto_user | email | Email người dùng |
| admin_user.tel | tenant_moto.mst_moto_user | tel | Số điện thoại liên hệ của quản trị viên |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | plan_code | data.plan_code | Gói dịch vụ đăng ký mới |
| tenant_moto.mst_moto_company | official_name_ja | data.company_profile.official_name_ja | Tên chính thức công ty (tiếng Nhật) |
| tenant_moto.mst_moto_company | display_name_ja | data.company_profile.display_name_ja | Tên hiển thị công ty (tiếng Nhật) |
| tenant_moto.mst_moto_company | postal_code | data.company_profile.postal_code | Mã bưu chính (7 chữ số) |
| tenant_moto.mst_moto_company_contact | tel | data.company_profile.tel | Số điện thoại của công ty |
| tenant_moto.mst_moto_company | address_ja | data.company_profile.address_ja | Địa chỉ công ty |
| tenant_moto.mst_moto_user | last_name_ja | data.admin_user.last_name_ja | Họ của quản trị viên (tiếng Nhật) |
| tenant_moto.mst_moto_user | first_name_ja | data.admin_user.first_name_ja | Tên của quản trị viên (tiếng Nhật) |
| tenant_moto.mst_moto_user | email | data.admin_user.email | Email người dùng |
| tenant_moto.mst_moto_user | tel | data.admin_user.tel | Số điện thoại liên hệ của quản trị viên |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request body đầu vào | Ngoài transaction |
| 2 | Kiểm tra sự tồn tại của Tenant MOTO theo `id` | Ngoài transaction |
| 3 | Khởi tạo transaction trên Platform DB | Trong transaction |
| 4 | UPDATE trường `plan_code` trong bảng `platform.tenant_registry` | Trong transaction |
| 5 | Thiết lập kết nối động (Dynamic Connection) và khởi tạo transaction trên Tenant DB | Trong transaction |
| 6 | UPDATE bảng `mst_moto_company` trong schema tenant | Trong transaction |
| 7 | UPDATE bảng `mst_moto_company_contact` trong schema tenant (với `seq_no = 1`) | Trong transaction |
| 8 | UPDATE bảng `mst_moto_user` trong schema tenant | Trong transaction |
| 9 | Commit transaction trên cả Platform DB và Tenant DB | Trong transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Gặp lỗi ngoại lệ (Exception) hoặc thất bại ở bất kỳ câu lệnh UPDATE nào | Rollback toàn bộ thay đổi trên cả Platform DB và Tenant DB |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| - | - | - | Không áp dụng (Không thay đổi trạng thái hoạt động của Tenant ở API này) |

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: PATCH /api/v1/admin/moto-tenants/{id} kèm Request Body.
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token.
3. Middleware kiểm tra quyền truy cập (Authorization) của Platform SaaS Admin/Staff (quyền "tenant.edit").
4. Controller thực hiện validation dữ liệu Request Body theo Validation Rules (Section 7).
   - Nếu dữ liệu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm thông tin lỗi chi tiết.
5. Service kiểm tra sự tồn tại của Tenant MOTO trong bảng `platform.tenant_registry` theo `id`.
   - Nếu không tìm thấy: Trả về HTTP 404 Not Found.
6. Service mở Database Transaction.
7. Service thực hiện cập nhật trường `plan_code` trong bảng `platform.tenant_registry`.
8. Service thiết lập kết nối động (Dynamic Connection) tới schema `tenant_moto_<company_id>` tương ứng.
9. Service cập nhật thông tin doanh nghiệp trong bảng `mst_moto_company` và số điện thoại công ty trong bảng `mst_moto_company_contact` (với `seq_no = 1`).
10. Service xác định bản ghi quản trị viên Tenant trong bảng `mst_moto_user` và cập nhật thông tin liên quan.
11. Nếu không có lỗi, Service commit transaction. Ngược lại, rollback toàn bộ và trả về HTTP 500 Internal Server Error.
12. Service trả về đúng các trường vừa được cập nhật.
13. Controller đóng gói kết quả thành công và trả về response JSON kèm HTTP 200 OK.
```

---

# 14. Notification

| Hạng mục | Nội dung |
| --- | --- |
| Có cần notification không | Không |
| Thời điểm gửi | - |
| Đối tượng nhận | - |
| Kênh gửi | - |
| Template ID | - |

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | UPDATE |
| Bảng target | platform.tenant_registry, tenant_moto.mst_moto_company, tenant_moto.mst_moto_user |
| ID record target | tenant_id |
| Dữ liệu ghi log | Before / After / Diff |
| Chính sách lưu trữ | Theo NFR |

---

# 16. Xử lý file

## 16.1 Upload

| Hạng mục | Nội dung |
| --- | --- |
| Có upload file không | Không |
| Max file size | - |
| Định dạng cho phép | - |
| Nơi lưu trữ | - |
| Virus Scan | - |
| Quy tắc đặt tên file | - |
| Bảng lưu metadata | - |

## 16.2 Download

| Hạng mục | Nội dung |
| --- | --- |
| Có download file không | Không |
| Permission Check | - |
| Phương thức download | - |
| Thời gian hết hạn | - |
| Audit Log | - |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc kiểm tra token JWT hợp lệ và phân quyền Platform SaaS Admin/Staff (`tenant.edit`). |
| 2 | SQL Injection & Data Sanitization | Đảm bảo dữ liệu đầu vào được validate và sanitize đầy đủ trước khi thực hiện câu lệnh UPDATE trong DB. |
| 3 | Schema Isolation | Ràng buộc kết nối DB sang schema chỉ định phải sử dụng DB Role có phạm vi phân quyền chính xác cho tenant đó. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.5 giây (bao gồm thiết lập connection động và cập nhật nhiều bảng) |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính `tenant_id` trên platform.tenant_registry, các khóa ngoại và khóa chính trên các bảng thuộc tenant schema. |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Wireframe | PA-TEN-004 派遣元テナント編集 |
| Screen Detail Design | Screen Design / UI Figma |
| ERD | Carima-Staffing Data Dictionary / ERD |
| Permission Matrix | Quy định Permission / Authorization (Section 9) |
| Validation Rule | Validation Rules (Section 7) |
| NFR | Không áp dụng |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| - | Không có | - | - | - | - |

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
