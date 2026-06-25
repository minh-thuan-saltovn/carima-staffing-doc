# PA-TEN-010-API-01-Update saki tenant

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Update saki tenant |
| Diagram / Flow áp dụng | BF-003 Tenant Management Flow |
| Portal áp dụng | Platform Manager |
| Version | v1.0 |
| Ngày tạo | 2026/06/22 |
| Ngày cập nhật | 2026/06/22 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/22 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Update saki tenant (PUT) theo đúng ERD, Template chuẩn, và API Convention của Engineering Handbook. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-010-API-01 |
| Tên API | Update saki tenant |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-010 |
| Chức năng liên quan | Tenant Management (Edit SAKI Tenant) |
| Mục đích API | Cập nhật thông tin đăng ký Tenant SAKI (gói dịch vụ, thông tin doanh nghiệp tiếp nhận, thông tin người dùng quản trị) trong Database và trả lại các trường thông tin vừa được cập nhật. |
| Loại API | Frontend API |
| Method | PUT |
| Endpoint | /api/v1/admin/saki-tenants/{id} |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Dynamic Tenant Schema |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này được gọi khi Platform SaaS Admin bấm nút "更新する" (Cập nhật) trên màn hình Chỉnh sửa SAKI Tenant (`PA-TEN-010`). API sẽ thực hiện:
1. Cập nhật gói dịch vụ (`plan_code`) trong bảng `platform.tenant_registry` của platform schema.
2. Dựa vào `schema_name` của Tenant, thiết lập kết nối động tới tenant schema tương ứng (`tenant_saki_<company_id>`).
3. Cập nhật thông tin doanh nghiệp trong bảng `mst_saki_company`.
4. Cập nhật thông tin người dùng quản trị (Admin User) trong bảng `mst_saki_user` của schema Tenant.
5. Toàn bộ tiến trình cập nhật dữ liệu ở schema Tenant phải được thực hiện trong Database Transaction.
6. Sau khi cập nhật thành công, API trả về chính xác các thông tin vừa cập nhật.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Quản trị viên click nút "Cập nhật" (更新する) trên giao diện. |
| Actor | Platform SaaS Admin / Platform SaaS Staff |
| Trước khi gọi API | Tenant SAKI đã tồn tại, dữ liệu nhập hợp lệ. |
| Sau khi gọi API | Dữ liệu được lưu trữ, hệ thống trả về các trường thông tin vừa được cập nhật. |
| Thay đổi trạng thái liên quan | Cập nhật các trường thông tin tương ứng. |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PUT /api/v1/admin/saki-tenants/{id}
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
| id | string | Có | ID của Tenant SAKI (ULID) | 01H2PJX7G3P681729X4R09Q871 |

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
    "official_name_ja": "株式会社サキケア",
    "official_name_kana": "カブシキガイシャサキケア",
    "display_name_ja": "サキケア",
    "postal_code": "1000001",
    "address_ja": "東京都千代田区千代田1-1",
    "address2_ja": "サキケアビル5F"
  },
  "admin_user": {
    "last_name_ja": "山田",
    "first_name_ja": "太郎",
    "email": "yamada@sakicare.jp",
    "tel": "03-1234-5678"
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
| company_profile.official_name_kana | string | Có | 200 | Không rỗng | Tên chính thức công ty (Katakana) |
| company_profile.display_name_ja | string | Có | 24 | Không rỗng | Tên hiển thị công ty (tiếng Nhật) |
| company_profile.postal_code | string | Có | 7 | 7 chữ số Half-width | Mã bưu chính |
| company_profile.address_ja | string | Có | 50 | Không rỗng | Địa chỉ 1 |
| company_profile.address2_ja | string | Không | 50 | - | Địa chỉ 2 |
| admin_user | object | Có | - | - | Thông tin tài khoản quản trị viên của Tenant |
| admin_user.last_name_ja | string | Có | 24 | Không rỗng | Họ của quản trị viên (tiếng Nhật) |
| admin_user.first_name_ja | string | Có | 24 | Không rỗng | Tên của quản trị viên (tiếng Nhật) |
| admin_user.email | string | Có | 128 | Định dạng Email | Email người dùng |
| admin_user.tel | string | Có | 15 | Số điện thoại Half-width | Số điện thoại liên hệ |

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
      "official_name_ja": "株式会社サキケア",
      "official_name_kana": "カブシキガイシャサキケア",
      "display_name_ja": "サキケア",
      "postal_code": "1000001",
      "address_ja": "東京都千代田区千代田1-1",
      "address2_ja": "サキケアビル5F"
    },
    "admin_user": {
      "last_name_ja": "山田",
      "first_name_ja": "太郎",
      "email": "yamada@sakicare.jp",
      "tel": "03-1234-5678"
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
| data.company_profile.official_name_kana | string | Tên chính thức công ty (Katakana) |
| data.company_profile.display_name_ja | string | Tên hiển thị công ty (tiếng Nhật) |
| data.company_profile.postal_code | string | Mã bưu chính (7 chữ số) |
| data.company_profile.address_ja | string | Địa chỉ 1 |
| data.company_profile.address2_ja | string | Địa chỉ 2 |
| **data.admin_user** | object | Thông tin tài khoản quản trị vừa cập nhật |
| data.admin_user.last_name_ja | string | Họ của quản trị viên (tiếng Nhật) |
| data.admin_user.first_name_ja | string | Tên của quản trị viên (tiếng Nhật) |
| data.admin_user.email | string | Email người dùng |
| data.admin_user.tel | string | Số điện thoại liên hệ |

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
        "field": "company_profile.postal_code",
        "message": "The postal code must be exactly 7 digits."
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
    "message": "Tenant SAKI not found."
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

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | id | required | - | required |
| 2 | id | size | 26 | size |
| 3 | plan_code | required | - | required |
| 4 | plan_code | max | 20 | max |
| 5 | company_profile.official_name_ja | required | - | required |
| 6 | company_profile.official_name_ja | max | 100 | max |
| 7 | company_profile.official_name_kana | required | - | required |
| 8 | company_profile.official_name_kana | regex | /^[ァ-ヶー]+$/u | regex |
| 9 | company_profile.official_name_kana | max | 200 | max |
| 10 | company_profile.display_name_ja | required | - | required |
| 11 | company_profile.display_name_ja | max | 24 | max |
| 12 | company_profile.postal_code | required | - | required |
| 13 | company_profile.postal_code | digits | 7 | digits |
| 14 | company_profile.address_ja | required | - | required |
| 15 | company_profile.address_ja | max | 50 | max |
| 16 | company_profile.address2_ja | max | 50 | max |
| 17 | admin_user.last_name_ja | required | - | required |
| 18 | admin_user.last_name_ja | max | 24 | max |
| 19 | admin_user.first_name_ja | required | - | required |
| 20 | admin_user.first_name_ja | max | 24 | max |
| 21 | admin_user.email | required | - | required |
| 22 | admin_user.email | email | - | email |
| 23 | admin_user.email | max | 128 | max |
| 24 | admin_user.tel | required | - | required |
| 25 | admin_user.tel | regex | /^[0-9-]+$/ | regex |
| 26 | admin_user.tel | max | 15 | max |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Role authorization | Chỉ Platform SaaS Admin hoặc Platform SaaS Staff có quyền `tenant.edit` mới được phép gọi API này. |
| BR-002 | Target validation | Xác minh xem `id` (ULID) có tồn tại trong bảng `platform.tenant_registry` với `tenant_type = 'SAKI'` (hoặc 1) hay không. |
| BR-003 | Dynamic Connection | Sử dụng `schema_name` lấy từ bảng registry để thiết lập kết nối động tới tenant schema để thực hiện update thông tin. |
| BR-004 | Update Target User | Cập nhật thông tin admin user trong schema Tenant. Hệ thống xác định bản ghi người dùng quản trị để cập nhật bằng cách truy vấn trong bảng `mst_saki_user` tìm user đầu tiên được khởi tạo hoặc user có `execution_role_id = 'role_admin'`. |
| BR-005 | DB Transaction | Tiến trình cập nhật trong Tenant DB bắt buộc phải bọc trong transaction. Nếu xảy ra lỗi ở bất kỳ bước nào, rollback toàn bộ thay đổi. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Admin | Có | Có | Có | Có | Có | Có |
| Tenant Admin | Có | Có | Có | Có | Có | Có |
| SAKI Admin | Có | Có | Có | Không | Có | Có |
| SAKI User | Có | Có | Chỉ dữ liệu của mình | Không | Không | Không |
| MOTO Admin | Không | Không | Không | Không | Không | Không |
| Staff | Không | Không | Không | Không | Không | Không |

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
| company_profile.official_name_ja | tenant_saki.mst_saki_company | official_name_ja | Tên chính thức công ty (tiếng Nhật) |
| company_profile.official_name_kana | tenant_saki.mst_saki_company | official_name_kana | Tên chính thức công ty (Katakana) |
| company_profile.display_name_ja | tenant_saki.mst_saki_company | display_name_ja | Tên hiển thị công ty (tiếng Nhật) |
| company_profile.postal_code | tenant_saki.mst_saki_company | postal_code | Mã bưu chính |
| company_profile.address_ja | tenant_saki.mst_saki_company | address_ja | Địa chỉ 1 |
| company_profile.address2_ja | tenant_saki.mst_saki_company | address2_ja | Địa chỉ 2 |
| admin_user.last_name_ja | tenant_saki.mst_saki_user | last_name_ja | Họ của quản trị viên (tiếng Nhật) |
| admin_user.first_name_ja | tenant_saki.mst_saki_user | first_name_ja | Tên của quản trị viên (tiếng Nhật) |
| admin_user.email | tenant_saki.mst_saki_user | email | Email người dùng |
| admin_user.tel | tenant_saki.mst_saki_user | tel | Số điện thoại |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | plan_code | data.plan_code | Gói dịch vụ đăng ký mới |
| tenant_saki.mst_saki_company | official_name_ja | data.company_profile.official_name_ja | Tên chính thức công ty (tiếng Nhật) |
| tenant_saki.mst_saki_company | official_name_kana | data.company_profile.official_name_kana | Tên chính thức công ty (Katakana) |
| tenant_saki.mst_saki_company | display_name_ja | data.company_profile.display_name_ja | Tên hiển thị công ty (tiếng Nhật) |
| tenant_saki.mst_saki_company | postal_code | data.company_profile.postal_code | Mã bưu chính (7 chữ số) |
| tenant_saki.mst_saki_company | address_ja | data.company_profile.address_ja | Địa chỉ 1 |
| tenant_saki.mst_saki_company | address2_ja | data.company_profile.address2_ja | Địa chỉ 2 |
| tenant_saki.mst_saki_user | last_name_ja | data.admin_user.last_name_ja | Họ của quản trị viên (tiếng Nhật) |
| tenant_saki.mst_saki_user | first_name_ja | data.admin_user.first_name_ja | Tên của quản trị viên (tiếng Nhật) |
| tenant_saki.mst_saki_user | email | data.admin_user.email | Email người dùng |
| tenant_saki.mst_saki_user | tel | data.admin_user.tel | Số điện thoại liên hệ |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request body đầu vào | Ngoài transaction |
| 2 | Kiểm tra sự tồn tại của Tenant SAKI theo `id` | Ngoài transaction |
| 3 | Khởi tạo transaction trên Platform DB | Trong transaction |
| 4 | UPDATE trường `plan_code` trong bảng `platform.tenant_registry` | Trong transaction |
| 5 | Thiết lập kết nối động (Dynamic Connection) và khởi tạo transaction trên Tenant DB | Trong transaction |
| 6 | UPDATE bảng `mst_saki_company` trong schema tenant | Trong transaction |
| 7 | UPDATE bảng `mst_saki_user` trong schema tenant | Trong transaction |
| 8 | Commit transaction trên cả Platform DB và Tenant DB | Trong transaction |

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
1. Client gửi yêu cầu: PUT /api/v1/admin/saki-tenants/{id} kèm Request Body.
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token.
3. Middleware kiểm tra quyền truy cập (Authorization) của Platform SaaS Admin/Staff (quyền "tenant.edit").
4. Controller thực hiện validation dữ liệu Request Body theo Validation Rules (Section 7).
   - Nếu dữ liệu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm thông tin lỗi chi tiết.
5. Service kiểm tra sự tồn tại của Tenant SAKI trong bảng `platform.tenant_registry` theo `id`.
   - Nếu không tìm thấy: Trả về HTTP 404 Not Found.
6. Service mở Database Transaction.
7. Service thực hiện cập nhật trường `plan_code` trong bảng `platform.tenant_registry`.
8. Service thiết lập kết nối động (Dynamic Connection) tới schema `tenant_saki_<company_id>` tương ứng.
9. Service cập nhật thông tin doanh nghiệp trong bảng `mst_saki_company`.
10. Service xác định bản ghi quản trị viên Tenant trong bảng `mst_saki_user` và cập nhật thông tin liên quan.
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
| Bảng target | platform.tenant_registry, tenant_saki.mst_saki_company, tenant_saki.mst_saki_user |
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
| Wireframe | PA-TEN-010 派遣先テナント編集 |
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
