# MO-SET-007-API-02-Update Workplace Type Master

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Update Workplace Type Master |
| Description | Cập nhật thông tin master loại nơi làm việc của MOTO. |
| Endpoint | /api/v1/moto/settings/workplace-type-master/{workplace_type_code} |
| Menu | Company Settings |
| Method | PATCH |
| Related Tables | tenant_db.mst_moto_workplace_type (được đề xuất thay thế cho workplace_type_master đang để TBD) |
| Screen list | 就業場所区分マスタ (MO-SET-007) - Workplace Type Master |
| System | MOTO Portal |
| 説明 | 勤務場所区分マスタを更新する (Cập nhật master loại nơi làm việc). |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Update Workplace Type Master |
| Diagram / Flow áp dụng | BF-001 User Management Flow / Tenant Master Management |
| Portal áp dụng | MOTO Portal |
| Version | v1.0 |
| Ngày tạo | 2026/06/24 |
| Ngày cập nhật | 2026/06/24 |
| Người tạo | Antigravity |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/24 | Antigravity | Khởi tạo tài liệu đặc tả chi tiết API Update Workplace Type Master (PATCH) bám sát UI giao diện thực tế và cấu trúc DB MOTO. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | MO-SET-007-API-02 |
| Tên API | Update Workplace Type Master |
| Business Flow liên quan | Tenant Master Management Flow |
| Screen ID liên quan | MO-SET-007 |
| Chức năng liên quan | Workplace Type Master Management |
| Mục đích API | Cập nhật thông tin chi tiết của một loại nơi làm việc đã tồn tại (tên loại, tên tiếng Anh, hình thức làm việc, chế độ đi lại và trạng thái hoạt động). |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/moto/settings/workplace-type-master/{workplace_type_code} |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích
Cập nhật thông tin chi tiết (tên loại, tên tiếng Anh, hình thức làm việc, chế độ đi lại và trạng thái) của một loại nơi làm việc cụ thể trong bảng `mst_moto_workplace_type` theo tenant.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Mô tả |
| --- | --- |
| Kích hoạt | Người dùng nhấn nút "Lưu" trên form chỉnh sửa |
| Actor | MOTO Admin, MOTO Staff |
| Trước khi gọi | Đã mở form chỉnh sửa của bản ghi cụ thể và nhập các thông tin hợp lệ |
| Sau khi gọi | Hiển thị Toast thành công, đóng form và reload danh sách |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/moto/settings/workplace-type-master/{workplace_type_code}
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Định dạng dữ liệu |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| workplace_type_code | string | Có | Mã loại nơi làm việc cần cập nhật | WPT001 |

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Ví dụ Request Body

```json
{
  "workplace_type_name": "クライアント常駐(更新)",
  "workplace_type_name_en": "Client On-site (Updated)",
  "work_style_type": 1,
  "is_transportation_eligible": 1,
  "status": 1
}
```

## 5.2 Định nghĩa các field Request Body

| Field | Type | Bắt buộc | Mô tả |
| --- | --- | --- | --- |
| workplace_type_name | string | Có | Tên loại nơi làm việc |
| workplace_type_name_en | string | Không | Tên loại nơi làm việc (tiếng Anh) |
| work_style_type | number | Có | Hình thức làm việc (1: 常駐 - On-site, 2: 在宅 - Remote, 3: ハイブリッド - Hybrid, 4: 出張 - Business Trip) |
| is_transportation_eligible | number | Có | Đối tượng chi trả phí đi lại (0: Không, 1: Có) |
| status | number | Có | Trạng thái (0: Vô hiệu, 1: Hoạt động) |

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
    "workplace_type_code": "WPT001",
    "workplace_type_name": "クライアント常駐(更新)",
    "workplace_type_name_en": "Client On-site (Updated)",
    "work_style_type": 1,
    "is_transportation_eligible": 1,
    "status": 1,
    "created_at": "2026-06-23T08:00:00+09:00",
    "updated_at": "2026-06-24T17:08:00+09:00"
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data | object | Thông tin chi tiết loại nơi làm việc vừa cập nhật |
| data.workplace_type_code | string | Mã loại nơi làm việc |
| data.workplace_type_name | string | Tên loại nơi làm việc |
| data.workplace_type_name_en | string | Tên loại nơi làm việc (tiếng Anh) |
| data.work_style_type | number | Hình thức làm việc (1: 常駐 - On-site, 2: 在宅 - Remote, 3: ハイブリッド - Hybrid, 4: 出張 - Business Trip) |
| data.is_transportation_eligible | number | Đối tượng chi trả phí đi lại (0: Không, 1: Có) |
| data.status | number | Trạng thái (0: Vô hiệu, 1: Hoạt động) |
| data.created_at | datetime | Thời điểm tạo |
| data.updated_at | datetime | Thời điểm cập nhật |

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
        "field": "workplace_type_name",
        "message": "The workplace_type_name field is required."
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
    "code": "NOT_FOUND",
    "message": "The requested workplace type does not exist."
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
| 1 | workplace_type_code | Bắt buộc (Path Parameter), chuỗi ký tự | CMS-VAL-23 | 種別コードを入力してください。 |
| 2 | workplace_type_code | Tối đa 20 ký tự | CMS-VAL-6 | 種別コードは20文字以内で入力してください。 |
| 3 | workplace_type_name | Bắt buộc, chuỗi ký tự | CMS-VAL-23 | 種別名を入力してください。 |
| 4 | workplace_type_name | Tối đa 100 ký tự | CMS-VAL-6 | 種別名は100文字以内で入力してください。 |
| 5 | workplace_type_name_en | Không bắt buộc, chuỗi ký tự, tối đa 100 ký tự | CMS-VAL-6 | 種別名（英字）は100文字以内で入力してください。 |
| 6 | work_style_type | Bắt buộc, số nguyên | CMS-VAL-23 | 勤務形態を入力してください。 |
| 7 | work_style_type | Phải là số nguyên | CMS-VAL-40 | 勤務形態は整数で指定してください。 |
| 8 | work_style_type | Phải thuộc tập hợp: 1, 2, 3, 4 | CMS-VAL-41 | 選択された勤務形態は正しくありません。 |
| 9 | is_transportation_eligible | Bắt buộc, số nguyên | CMS-VAL-23 | 交通費対象を入力してください。 |
| 10 | is_transportation_eligible | Phải là số nguyên | CMS-VAL-40 | 交通費対象は整数で指定してください。 |
| 11 | is_transportation_eligible | Phải thuộc tập hợp: 0, 1 | CMS-VAL-41 | 選択された交通費対象は正しくありません。 |
| 12 | status | Bắt buộc, số nguyên | CMS-VAL-23 | ステータスを入力してください。 |
| 13 | status | Phải là số nguyên | CMS-VAL-40 | ステータスは整数で指定してください。 |
| 14 | status | Phải thuộc tập hợp: 0, 1 | CMS-VAL-41 | 選択されたステータスは正しくありません。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ thao tác trên schema tenant hiện tại qua JWT token. Nghiêm cấm ghi dữ liệu chéo tenant. |
| BR-002 | Role authorization | Yêu cầu quyền workplace_type.update để thực hiện API. |
| BR-003 | Database Transaction | Bắt buộc bọc xử lý trong Database Transaction, rollback nếu có lỗi xảy ra. |
| BR-004 | Auditing Fields | Tự động ghi nhận thông tin updated_at và updated_by khi lưu thành công. |
| BR-005 | Resource check | Kiểm tra sự tồn tại của workplace_type_code trước khi cập nhật; trả về 404 nếu không tìm thấy. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Admin | Có | Có | Có | Có | Không | Có |
| Tenant Admin | Có | Có | Có | Có | Không | Có |
| MOTO Admin | Có | Có | Có | Không | Không | Có |
| MOTO User | Có | Không | Không | Không | Không | Không |
| SAKI Admin | Không | Không | Không | Không | Không | Không |
| Staff | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Kiểm tra JWT token trong Header.
2. Trích xuất quyền hạn của user.
3. Nếu không có quyền workplace_type.update, trả về 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Param | Table | Column | Mô tả |
| --- | --- | --- | --- |
| workplace_type_code (Path) | tenant_moto.mst_moto_workplace_type | workplace_type_code | Điều kiện tìm kiếm khóa chính để cập nhật |
| workplace_type_name (Body) | tenant_moto.mst_moto_workplace_type | workplace_type_name | Cập nhật tên loại nơi làm việc |
| workplace_type_name_en (Body) | tenant_moto.mst_moto_workplace_type | workplace_type_name_en | Cập nhật tên loại nơi làm việc (tiếng Anh) |
| work_style_type (Body) | tenant_moto.mst_moto_workplace_type | work_style_type | Cập nhật hình thức làm việc |
| is_transportation_eligible (Body) | tenant_moto.mst_moto_workplace_type | is_transportation_eligible | Cập nhật đối tượng chi trả phí đi lại |
| status (Body) | tenant_moto.mst_moto_workplace_type | status | Cập nhật trạng thái hoạt động |

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_moto.mst_moto_workplace_type | workplace_type_code | data.workplace_type_code | Mã loại nơi làm việc |
| tenant_moto.mst_moto_workplace_type | workplace_type_name | data.workplace_type_name | Tên loại nơi làm việc |
| tenant_moto.mst_moto_workplace_type | workplace_type_name_en | data.workplace_type_name_en | Tên loại nơi làm việc (tiếng Anh) |
| tenant_moto.mst_moto_workplace_type | work_style_type | data.work_style_type | Hình thức làm việc |
| tenant_moto.mst_moto_workplace_type | is_transportation_eligible | data.is_transportation_eligible | Đối tượng chi trả phí đi lại |
| tenant_moto.mst_moto_workplace_type | status | data.status | Trạng thái |
| tenant_moto.mst_moto_workplace_type | created_at | data.created_at | Thời điểm tạo |
| tenant_moto.mst_moto_workplace_type | updated_at | data.updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

Áp dụng cho phương thức ghi dữ liệu (PATCH).
Toàn bộ quá trình kiểm tra sự tồn tại và thực hiện lệnh UPDATE phải được đặt trong DB::transaction để đảm bảo tính nhất quán dữ liệu. Nếu xảy ra bất kỳ lỗi runtime nào, toàn bộ các thay đổi cơ sở dữ liệu sẽ bị rollback.

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Nhận request PATCH kèm request body và path parameter.
2. Xác định JWT và kiểm tra quyền workplace_type.update.
3. Validate dữ liệu đầu vào. Trả về 422 nếu không hợp lệ.
4. Kết nối tới schema tenant dựa trên thông tin JWT.
5. Khởi tạo DB Transaction:
   - Tìm kiếm loại nơi làm việc theo workplace_type_code. Nếu không tồn tại, rollback và trả về 404.
   - Cập nhật các trường dữ liệu và ghi nhận audit log (updated_at, updated_by).
6. Commit Transaction.
7. Trả về response JSON của bản ghi sau cập nhật kèm HTTP 200.
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
| Lớp đối tượng | WorkplaceTypeMaster |
| Hành động | Update |
| ID bản ghi | workplace_type_code |
| Mô tả chi tiết | Cập nhật thông tin loại nơi làm việc: {workplace_type_code}. |
| Người thực hiện | user_id (Lấy từ JWT token) |
| Thời gian | Thời điểm hệ thống xử lý ghi thành công |

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
| Quy trình download | Không áp dụng |
| Thời gian hết hạn | Không áp dụng |
| Audit Log | Không áp dụng |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Phải xác thực JWT token và kiểm tra quyền workplace_type.update của người dùng trước khi xử lý. |
| 2 | Tenant Isolation | Thiết lập kết nối động tới đúng tenant schema dựa trên thông tin công ty của người dùng đăng nhập để cách ly dữ liệu. Không cho phép truyền tham số company_id từ client để thay đổi tenant truy vấn dữ liệu. |
| 3 | Input Sanitization & SQL Injection | Sử dụng Eloquent ORM / Parameterized Query để chèn các tham số cập nhật và điều kiện tìm kiếm vào câu lệnh SQL để ngăn chặn tấn công SQL injection. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.3 giây |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính trên bảng mst_moto_workplace_type. |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | Tenant Master Management Flow |
| Wireframe | MO-SET-007 就業場所区分マスタ |
| Screen Detail Design | MO-SET-007 |
| ERD | Carima-Staffing Data Dictionary / ERD |
| Permission Matrix | Portal Permission Matrix |
| Validation Rule | Validation Rules |
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
| 8 | Tenant isolation rule đã được định nghĩa | Done |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done (Không áp dụng) |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
