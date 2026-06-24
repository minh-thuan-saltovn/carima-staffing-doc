# PA-TEN-015-API-01-Tenant Branding Setting

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Tenant Branding Setting |
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
| v1.0 | 2026/06/17 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Tenant Branding Setting dựa trên thiết kế màn hình PA-TEN-015, ERD platform schema và API List mới nhất |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-TEN-015-API-01 |
| Tên API | Tenant Branding Setting |
| Business Flow liên quan | BF-003 Tenant Management Flow |
| Screen ID liên quan | PA-TEN-015 |
| Chức năng liên quan | Tenant Management (Branding Setting) |
| Mục đích API | Cập nhật cấu hình thương hiệu (Logo, Theme Color, Login Background) cho một Tenant cụ thể. |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/admin/tenants/{id}/branding |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Platform Common / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Upload (Logo, Login Background) |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để cập nhật cấu hình thương hiệu (Branding) của một Tenant, bao gồm tải lên Logo mới, thiết lập mã màu chủ đạo (Theme Color) và tải lên hình nền trang đăng nhập (Login Background). Các tệp tin ảnh được tải lên CDN/Cloud Storage và chỉ cập nhật URL vào cơ sở dữ liệu sau khi lưu trữ thành công.

### Ngữ cảnh nghiệp vụ

API được gọi khi Platform SaaS Admin nhấn nút "設定保存" (Lưu cấu hình) tại màn hình cấu hình thương hiệu Tenant (PA-TEN-015).

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User nhấn nút "設定保存" (Lưu cấu hình) trên màn hình PA-TEN-015 sau khi xác nhận dialog |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Tenant đã tồn tại trong hệ thống. Quản trị viên đã chọn/nhập logo, theme color, login background trên giao diện. |
| Sau khi gọi API | Cập nhật cấu hình branding trong DB, ghi Audit Log, giao diện của Tenant tương ứng áp dụng theme mới ngay lập tức. |
| Thay đổi trạng thái liên quan | Cập nhật bản ghi cấu hình thương hiệu trong bảng `platform.tenant_branding` (TBD). |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/admin/tenants/{id}/branding
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | multipart/form-data | Bắt buộc dùng multipart/form-data do có upload file ảnh |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | ID của Tenant cần cập nhật branding (ULID) | 01H2PJX7G3P681729X4R09Q872 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

> **Lưu ý:** Request phải gửi dưới dạng `multipart/form-data` vì có tệp tin nhị phân đính kèm.

```
Content-Type: multipart/form-data

theme_color:         "#1E3A8A"
logo_file:           [Binary File - tùy chọn]
login_bg_file:       [Binary File - tùy chọn]
delete_logo_flg:     0
delete_login_bg_flg: 0
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| theme_color | string | Có | 7 | Đúng định dạng mã màu HEX (ví dụ: `#1E3A8A`). Pattern: `^#[0-9A-Fa-f]{6}$` | Mã màu chủ đạo (Theme Color) |
| logo_file | file | Không | 2MB | Định dạng PNG hoặc JPEG. Dung lượng tối đa 2MB. | Tệp tin ảnh Logo tải lên |
| login_bg_file | file | Không | 5MB | Định dạng PNG hoặc JPEG. Dung lượng tối đa 5MB. | Tệp tin ảnh nền Login Background tải lên |
| delete_logo_flg | integer | Không | - | Giá trị 0 hoặc 1. Mặc định: 0 | Cờ xóa logo hiện tại (1: xóa, 0: giữ nguyên) |
| delete_login_bg_flg | integer | Không | - | Giá trị 0 hoặc 1. Mặc định: 0 | Cờ xóa ảnh nền hiện tại (1: xóa, 0: giữ nguyên) |

> **Lưu ý nghiệp vụ:** Nếu `delete_logo_flg = 1` và đồng thời có `logo_file`, hệ thống ưu tiên xóa logo hiện tại trước, sau đó tải lên logo mới. Tương tự với `delete_login_bg_flg`.

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
    "branding": {
      "logo_url": "https://storage.carima.link/branding/01H2PJX7G3P681729X4R09Q872/logo.png",
      "theme_color": "#1E3A8A",
      "login_bg_url": "https://storage.carima.link/branding/01H2PJX7G3P681729X4R09Q872/login-bg.jpg"
    }
  }
}
```

> **Lưu ý:** Nếu logo hoặc login_bg bị xóa, `logo_url` hoặc `login_bg_url` trả về `null`.

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.tenant_id | string | ID định danh duy nhất của Tenant (ULID) |
| data.branding | object | Cấu hình thương hiệu sau khi cập nhật |
| data.branding.logo_url | string \| null | URL công khai của ảnh Logo trên CDN/Storage. `null` nếu chưa cấu hình hoặc đã xóa |
| data.branding.theme_color | string | Mã màu HEX chủ đạo của thương hiệu |
| data.branding.login_bg_url | string \| null | URL công khai của ảnh nền Login. `null` nếu chưa cấu hình hoặc đã xóa |

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
        "field": "theme_color",
        "message": "The theme color must be a valid HEX color code (e.g., #1E3A8A)."
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

### Payload Too Large

```
413 Payload Too Large
```

```json
{
  "error": {
    "code": "FILE_TOO_LARGE",
    "message": "The uploaded file exceeds the maximum allowed size."
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
| 1 | id | Bắt buộc, đúng định dạng ULID (26 ký tự) | CMS-VAL-23 | idを入力してください。 |
| 2 | theme_color | Bắt buộc nhập | CMS-VAL-23 | theme_colorを入力してください。 |
| 3 | theme_color | Đúng định dạng mã màu HEX 6 ký tự. Pattern: ^#[0-9A-Fa-f]{6}$ | CMS-VAL-41 | 選択されたthemecolorは正しくありません。 |
| 4 | logo_file | Nếu có upload: Chỉ chấp nhận định dạng PNG hoặc JPEG (MIME type: image/png, image/jpeg) | CMS-VAL-41 | 選択されたlogofileは正しくありません。 |
| 5 | logo_file | Nếu có upload: Dung lượng tối đa 2MB (2,097,152 bytes) | CMS-VAL-6 | logo_fileは2文字以内で入力してください。 |
| 6 | login_bg_file | Nếu có upload: Chỉ chấp nhận định dạng PNG hoặc JPEG | CMS-VAL-41 | 選択されたloginbgfileは正しくありません。 |
| 7 | login_bg_file | Nếu có upload: Dung lượng tối đa 5MB (5,242,880 bytes) | CMS-VAL-6 | login_bg_fileは5文字以内で入力してください。 |
| 8 | delete_logo_flg | Nếu có: Giá trị phải là 0 hoặc 1 | CMS-VAL-41 | 選択されたdelete_logo_flgは正しくありません。 |
| 9 | delete_login_bg_flg | Nếu có: Giá trị phải là 0 hoặc 1 | CMS-VAL-41 | 選択されたdelete_login_bg_flgは正しくありません。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Role authorization | Chỉ tài khoản thuộc nhóm Platform SaaS Admin có quyền `tenant.update` mới được phép thực hiện API này. Platform SaaS Staff bị từ chối (HTTP 403). |
| BR-002 | File storage path convention | Tệp tin ảnh được lưu trên CDN/Cloud Storage theo cấu trúc path: `branding/{tenant_id}/{logo\|login-bg}.{ext}`. Mỗi lần cập nhật sẽ ghi đè (overwrite) file cũ cùng path để tránh tích lũy file rác. |
| BR-003 | Delete flag priority | Nếu `delete_logo_flg = 1` được gửi đồng thời với `logo_file` mới, hệ thống ưu tiên xóa file cũ, sau đó lưu file mới và cập nhật URL mới vào DB. |
| BR-004 | Partial update | Chỉ các trường được gửi lên trong request sẽ được cập nhật. Nếu không có `logo_file` và `delete_logo_flg != 1`, URL Logo hiện tại được giữ nguyên. |
| BR-005 | Async file processing | Tác vụ tải file lên CDN/Storage phải được xử lý đồng bộ trong cùng request (vì cần URL trả về ngay). Tuy nhiên, các tác vụ hậu xử lý như tối ưu hóa hình ảnh (image compression, resize) phải được đẩy vào Queue Job xử lý ngầm sau khi response đã được trả về. |
| BR-006 | Audit log recording | Mọi thao tác cập nhật cấu hình branding phải ghi nhận chi tiết nhật ký (Audit Log) bao gồm giá trị cũ và giá trị mới vào `platform.tenant_provision_log`. |

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
4. Nếu hợp lệ, cho phép tiếp tục xử lý.
5. Nếu không hợp lệ, từ chối yêu cầu và trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Logic xử lý / Mapping |
| --- | --- | --- | --- |
| id | platform.tenant_registry | tenant_id | Xác định Tenant đích cần cập nhật |
| theme_color | platform.tenant_branding | theme_color | Lưu trực tiếp giá trị mã HEX |
| logo_file | platform.tenant_branding | logo_url | Upload lên Storage → Lưu URL công khai vào DB. Nếu `delete_logo_flg = 1` → lưu `NULL` |
| login_bg_file | platform.tenant_branding | login_bg_url | Upload lên Storage → Lưu URL công khai vào DB. Nếu `delete_login_bg_flg = 1` → lưu `NULL` |

> **Lưu ý:** Bảng `platform.tenant_branding` hiện đang ở trạng thái **TBD** trong ERD. Cấu trúc bảng dự kiến:

| Column | Type | Mô tả |
| --- | --- | --- |
| id | BIGINT | PK tự tăng |
| tenant_id | CHAR(26) | FK → `platform.tenant_registry.tenant_id` (ULID) |
| logo_url | VARCHAR(500) | URL công khai của ảnh Logo trên CDN/Storage |
| theme_color | VARCHAR(7) | Mã màu HEX chủ đạo (ví dụ: `#1E3A8A`) |
| login_bg_url | VARCHAR(500) | URL công khai của ảnh nền Login trên CDN/Storage |
| created_at | TIMESTAMPTZ | Thời điểm tạo bản ghi |
| updated_at | TIMESTAMPTZ | Thời điểm cập nhật gần nhất |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| platform.tenant_registry | tenant_id | data.tenant_id | ID định danh của Tenant |
| platform.tenant_branding | logo_url | data.branding.logo_url | URL ảnh Logo sau khi cập nhật |
| platform.tenant_branding | theme_color | data.branding.theme_color | Mã màu HEX sau khi cập nhật |
| platform.tenant_branding | login_bg_url | data.branding.login_bg_url | URL ảnh nền Login sau khi cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request data & headers | Ngoài transaction |
| 2 | Check permission Platform Admin | Ngoài transaction |
| 3 | Validate file MIME type & size | Ngoài transaction |
| 4 | SELECT kiểm tra Tenant tồn tại trong `platform.tenant_registry` | Ngoài transaction |
| 5 | Upload file(s) lên CDN/Cloud Storage, nhận URL kết quả | Ngoài transaction (trước khi vào transaction) |
| 6 | BEGIN TRANSACTION |  |
| 7 | UPSERT thông tin branding vào `platform.tenant_branding` | Trong transaction |
| 8 | INSERT bản ghi hành động vào `platform.tenant_provision_log` | Trong transaction |
| 9 | COMMIT transaction | Trong transaction |
| 10 | Đẩy Queue Job tối ưu hóa hình ảnh (compress/resize) vào hàng đợi | Ngoài transaction (sau commit) |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Không tìm thấy Tenant trong `platform.tenant_registry` | Không khởi động transaction, trả về HTTP 404 |
| 2 | Upload file lên Storage thất bại | Không khởi động transaction, trả về HTTP 500 |
| 3 | UPSERT bảng `platform.tenant_branding` thất bại | Rollback toàn bộ DB changes |
| 4 | INSERT log vào `platform.tenant_provision_log` thất bại | Rollback toàn bộ DB changes |
| 5 | Bất kỳ ngoại lệ Exception hệ thống nào phát sinh | Rollback toàn bộ DB changes |

> **Lưu ý:** Nếu transaction rollback sau khi file đã upload lên Storage thành công, hệ thống phải thực hiện xóa file vừa upload ra khỏi Storage (cleanup orphan files) để tránh rác tích lũy.

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: PATCH /api/v1/admin/tenants/{id}/branding (multipart/form-data).
2. Middleware thực hiện kiểm tra Authentication và xác minh JWT Token hợp lệ.
3. Middleware kiểm tra quyền truy cập (Authorization) của Platform SaaS Admin (quyền "tenant.update").
4. Controller validate các tham số đầu vào:
   - Path Parameter `id` (ULID format).
   - Form field `theme_color` (HEX format).
   - File `logo_file` (nếu có): MIME type PNG/JPEG, size <= 2MB.
   - File `login_bg_file` (nếu có): MIME type PNG/JPEG, size <= 5MB.
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm chi tiết lỗi.
5. Service kiểm tra Tenant tồn tại trong `platform.tenant_registry` theo `id`.
   - Nếu không tìm thấy: Trả về HTTP 404 Not Found.
6. Service xử lý upload file (ngoài transaction):
   - Với mỗi file hợp lệ được gửi lên: Upload lên CDN/Cloud Storage tại path `branding/{tenant_id}/{logo|login-bg}.{ext}`.
   - Nhận về URL công khai sau khi upload thành công.
   - Nếu upload thất bại: Không khởi động transaction, trả về HTTP 500.
7. Service thực hiện cập nhật DB trong Database Transaction:
   - UPSERT bản ghi branding vào bảng `platform.tenant_branding`:
     - Cập nhật `theme_color`.
     - Cập nhật `logo_url` = URL mới (hoặc NULL nếu `delete_logo_flg = 1` và không có file mới).
     - Cập nhật `login_bg_url` = URL mới (hoặc NULL nếu `delete_login_bg_flg = 1` và không có file mới).
   - INSERT bản ghi Audit Log vào `platform.tenant_provision_log` với action `update_branding`.
8. Transaction được Commit thành công.
9. Đẩy Queue Job tối ưu hóa hình ảnh (compress/resize) vào hàng đợi xử lý ngầm.
10. Controller định dạng response qua API Resource và trả về HTTP 200 OK.
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
| Bảng target | platform.tenant_branding |
| ID record target | tenant_id |
| Dữ liệu ghi log | user_id, action = 'update_branding', detail = JSON chứa cấu hình branding cũ và mới (logo_url, theme_color, login_bg_url) |
| Chính sách lưu trữ | Ghi vào bảng `platform.tenant_provision_log` |

---

# 16. Xử lý file

## 16.1 Upload

| Hạng mục | Nội dung |
| --- | --- |
| Có upload file không | Có |
| Các file được upload | Logo (`logo_file`), Login Background (`login_bg_file`) |
| Max file size - Logo | 2MB (2,097,152 bytes) |
| Max file size - Background | 5MB (5,242,880 bytes) |
| Định dạng cho phép | PNG (`image/png`), JPEG (`image/jpeg`) |
| Nơi lưu trữ | CDN / Cloud Storage (S3 hoặc tương đương) |
| Quy tắc đặt tên / path | `branding/{tenant_id}/{logo\|login-bg}.{ext}` (ghi đè file cũ cùng path) |
| Hậu xử lý sau upload | Đẩy Queue Job tối ưu hóa hình ảnh (compress/resize) sau khi commit DB thành công |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Bắt buộc xác thực JWT Token hợp lệ và kiểm tra quyền Platform SaaS Admin (`tenant.update`). |
| 2 | File MIME type validation | Kiểm tra MIME type thực tế của file (không chỉ dựa vào phần mở rộng) để ngăn chặn upload file độc hại. |
| 3 | File content scanning | Khuyến nghị quét virus/malware với tệp tin upload trước khi lưu vào Storage chính thức. |
| 4 | Storage access control | URL CDN của ảnh phải được phân quyền phù hợp (public-read với logo/background là chấp nhận được vì đây là tài nguyên giao diện). |
| 5 | Path traversal prevention | Tên file đầu vào không được dùng trực tiếp làm tên file trên Storage. Hệ thống tự sinh path theo `tenant_id`. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 3 giây (bao gồm upload file lên Storage) |
| Lượng truy cập dự kiến | TBD |
| Pagination | Không áp dụng |
| Index cần thiết | Index cột `tenant_id` trên bảng `platform.tenant_branding` (UNIQUE) |
| Cache cần thiết | Cache thông tin branding (logo_url, theme_color) phía Client/CDN để tải giao diện nhanh |
| Bulk Processing | Không |
| Async Processing | Có (Tác vụ tối ưu hóa hình ảnh thực hiện bất đồng bộ sau khi response trả về) |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-003 Tenant Management Flow |
| Giao diện liên quan | PA-TEN-015 テナントブランド設定 |
| Thiết kế màn hình | spec-screen/PA-TEN-015.md |
| ERD | Carima-Staffing Data Dictionary / ERD (platform.tenant_branding — TBD) |
| API List | API List 341f02c407dd80c99339fa65d175b7e1.csv |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| 1 | Bảng `platform.tenant_branding` chưa được thiết kế trong ERD (ghi nhận là TBD trong API List). Cần thiết kế và bổ sung vào ERD chính thức. | High | Thuận | TBD | Open |

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
