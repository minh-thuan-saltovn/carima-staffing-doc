# MO-SET-007-API-02-Update Workplace Type Master

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Update Workplace Type Master |
| Description | Cập nhật thông tin master loại nơi làm việc của MOTO. |
| Endpoint | /api/v1/moto/settings/workplace-type-master/{workplace_code} |
| Menu | Company Settings |
| Method | PATCH |
| Related Tables | tenant_moto_<id>.mst_moto_workplace_type |
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
| Version | v1.2 |
| Ngày tạo | 2026/06/24 |
| Ngày cập nhật | 2026/06/25 |
| Người tạo | Antigravity |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/24 | Antigravity | Khởi tạo tài liệu đặc tả chi tiết API Update Workplace Type Master (PATCH) bám sát UI giao diện thực tế và cấu trúc DB MOTO. |
| v1.2 | 2026/06/25 | Antigravity | Cập nhật cấu trúc trường theo bảng mst_moto_workplace_type của ERD chuẩn và logic tính hiệu lực. |

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
| Mục đích API | Cập nhật thông tin chi tiết của một loại nơi làm việc đã tồn tại (tên hiển thị PC/Mobile bằng tiếng Nhật/Anh, tháng hiệu lực, cờ sử dụng WebTC/FAX). |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/moto/settings/workplace-type-master/{workplace_code} |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích
Cập nhật thông tin chi tiết của một loại nơi làm việc cụ thể trong bảng mst_moto_workplace_type theo tenant.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Mô tả |
| --- | --- |
| Kích hoạt | Người dùng nhấn nút Lưu trên form chỉnh sửa |
| Actor | MOTO Admin, MOTO Staff |
| Trước khi gọi | Đã mở form chỉnh sửa của bản ghi cụ thể và nhập các thông tin hợp lệ |
| Sau khi gọi | Hiển thị Toast thành công, đóng form và reload danh sách |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/moto/settings/workplace-type-master/{workplace_code}
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
| workplace_code | string | Có | Mã loại nơi làm việc cần cập nhật | OFFICE |

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Ví dụ Request Body

```json
{
  "pc_display_name": "オフィス(更新)",
  "pc_display_name_en": "Office (Updated)",
  "mobile_display_name": "オウ",
  "mobile_display_name_en": "Off-U",
  "valid_from_month": "2026-01",
  "valid_to_month": "2027-12",
  "use_fax_flg": 1
}
```

## 5.2 Định nghĩa các field Request Body

| Field | Type | Bắt buộc | Mô tả |
| --- | --- | --- | --- |
| pc_display_name | string | Có | Tên hiển thị trên PC (Tiếng Nhật) |
| pc_display_name_en | string | Không | Tên hiển thị trên PC (Tiếng Anh) |
| mobile_display_name | string | Có | Tên hiển thị trên Mobile (Tiếng Nhật) |
| mobile_display_name_en | string | Không | Tên hiển thị trên Mobile (Tiếng Anh) |
| valid_from_month | string | Có | Tháng bắt đầu áp dụng (Định dạng YYYY-MM) |
| valid_to_month | string | Không | Tháng kết thúc áp dụng (Định dạng YYYY-MM hoặc null) |
| use_fax_flg | number | Có | Cờ sử dụng cho FAX (1: Có, 0: Không) |

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
    "id": 1,
    "workplace_code": "OFFICE",
    "pc_display_name": "オフィス(更新)",
    "pc_display_name_en": "Office (Updated)",
    "mobile_display_name": "オウ",
    "mobile_display_name_en": "Off-U",
    "valid_from_month": "2026-01",
    "valid_to_month": "2027-12",
    "use_fax_flg": 1,
    "created_at": "2026-06-23T08:00:00+09:00",
    "updated_at": "2026-06-24T17:08:00+09:00"
  }
}
```

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data | object | Thông tin chi tiết loại nơi làm việc vừa cập nhật |
| data.id | number | ID tự tăng khóa chính |
| data.workplace_code | string | Mã loại nơi làm việc |
| data.pc_display_name | string | Tên hiển thị trên PC (Tiếng Nhật) |
| data.pc_display_name_en | string | Tên hiển thị trên PC (Tiếng Anh) |
| data.mobile_display_name | string | Tên hiển thị trên Mobile (Tiếng Nhật) |
| data.mobile_display_name_en | string | Tên hiển thị trên Mobile (Tiếng Anh) |
| data.valid_from_month | string | Tháng bắt đầu áp dụng (YYYY-MM) |
| data.valid_to_month | string | Tháng kết thúc áp dụng (YYYY-MM hoặc null) |
| data.use_fax_flg | number | Cờ sử dụng cho FAX (1: Có, 0: Không) |
| data.created_at | datetime | Thời điểm tạo |
| data.updated_at | datetime | Thời điểm cập nhật |

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
| 1 | workplace_code | required | - | required |
| 2 | workplace_code | max | 20 | max |
| 3 | pc_display_name | required | - | required |
| 4 | pc_display_name | max | 50 | max |
| 5 | pc_display_name_en | max | 50 | max |
| 6 | mobile_display_name | required | - | required |
| 7 | mobile_display_name | max | 20 | max |
| 8 | mobile_display_name_en | max | 20 | max |
| 9 | valid_from_month | required | - | required |
| 10 | valid_from_month | date_format | YYYY-MM | date_format |
| 11 | valid_to_month | date_format | YYYY-MM | date_format |
| 12 | valid_to_month | after_or_equal | valid_from_month | after_or_equal |
| 13 | use_fax_flg | required | - | required |
| 14 | use_fax_flg | in | 0, 1 | in |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ thao tác trên schema tenant hiện tại qua JWT token. Nghiêm cấm ghi dữ liệu chéo tenant. |
| BR-002 | Role authorization | Yêu cầu quyền workplace_type.update để thực hiện API. |
| BR-003 | Database Transaction | Bắt buộc bọc xử lý trong Database Transaction, rollback nếu có lỗi xảy ra. |
| BR-004 | Auditing Fields | Tự động ghi nhận thông tin updated_at và updated_by khi lưu thành công. |
| BR-005 | Resource check | Kiểm tra sự tồn tại của workplace_code trước khi cập nhật; trả về 404 nếu không tìm thấy. |

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
| workplace_code (Path) | tenant_moto_<id>.mst_moto_workplace_type | workplace_code | Điều kiện tìm kiếm khóa chính để cập nhật |
| pc_display_name (Body) | tenant_moto_<id>.mst_moto_workplace_type | pc_display_name | Cập nhật tên hiển thị trên PC (Tiếng Nhật) |
| pc_display_name_en (Body) | tenant_moto_<id>.mst_moto_workplace_type | pc_display_name_en | Cập nhật tên hiển thị trên PC (Tiếng Anh) |
| mobile_display_name (Body) | tenant_moto_<id>.mst_moto_workplace_type | mobile_display_name | Cập nhật tên hiển thị trên Mobile (Tiếng Nhật) |
| mobile_display_name_en (Body) | tenant_moto_<id>.mst_moto_workplace_type | mobile_display_name_en | Cập nhật tên hiển thị trên Mobile (Tiếng Anh) |
| valid_from_month (Body) | tenant_moto_<id>.mst_moto_workplace_type | valid_from_month | Cập nhật tháng bắt đầu áp dụng |
| valid_to_month (Body) | tenant_moto_<id>.mst_moto_workplace_type | valid_to_month | Cập nhật tháng kết thúc áp dụng |
| use_fax_flg (Body) | tenant_moto_<id>.mst_moto_workplace_type | use_fax_flg | Cờ sử dụng cho FAX |

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_moto_<id>.mst_moto_workplace_type | id | data.id | ID tự tăng khóa chính |
| tenant_moto_<id>.mst_moto_workplace_type | workplace_code | data.workplace_code | Mã loại nơi làm việc |
| tenant_moto_<id>.mst_moto_workplace_type | pc_display_name | data.pc_display_name | Tên hiển thị trên PC (Tiếng Nhật) |
| tenant_moto_<id>.mst_moto_workplace_type | pc_display_name_en | data.pc_display_name_en | Tên hiển thị trên PC (Tiếng Anh) |
| tenant_moto_<id>.mst_moto_workplace_type | mobile_display_name | data.mobile_display_name | Tên hiển thị trên Mobile (Tiếng Nhật) |
| tenant_moto_<id>.mst_moto_workplace_type | mobile_display_name_en | data.mobile_display_name_en | Tên hiển thị trên Mobile (Tiếng Anh) |
| tenant_moto_<id>.mst_moto_workplace_type | valid_from_month | data.valid_from_month | Tháng bắt đầu áp dụng |
| tenant_moto_<id>.mst_moto_workplace_type | valid_to_month | data.valid_to_month | Tháng kết thúc áp dụng |
| tenant_moto_<id>.mst_moto_workplace_type | use_fax_flg | data.use_fax_flg | Cờ sử dụng cho FAX |
| tenant_moto_<id>.mst_moto_workplace_type | created_at | data.created_at | Thời điểm tạo |
| tenant_moto_<id>.mst_moto_workplace_type | updated_at | data.updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

| Hạng mục | Nội dung |
| --- | --- |
| Transaction | Áp dụng (PATCH) |
| Phạm vi áp dụng | Bao bọc toàn bộ quá trình SELECT kiểm tra tồn tại và UPDATE |
| Xử lý rollback | Tự động rollback nếu xảy ra bất kỳ ngoại lệ nào trong quá trình xử lý |

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Xác thực quyền workplace_type.update và validate request.
2. Tìm bản ghi theo workplace_code (chưa bị xóa, deleted_at IS NULL) của tenant. Nếu không tồn tại, trả về HTTP 404.
3. Trong Transaction: Cập nhật các trường dữ liệu, ghi log audit (updated_at, updated_by) và commit.
4. Trả về HTTP 200 kèm thông tin bản ghi sau cập nhật.
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
| ID bản ghi | workplace_code |
| Mô tả chi tiết | Cập nhật thông tin loại nơi làm việc: {workplace_code}. |
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
| 1 | Auth | Xác thực JWT và kiểm tra quyền workplace_type.update. |
| 2 | Tenant Isolation | Kết nối động tới schema tenant theo JWT, cấm truyền company_id từ client. |
| 3 | SQL Injection | Dùng Eloquent/Parameterized Query để phòng ngừa SQL injection. |

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
