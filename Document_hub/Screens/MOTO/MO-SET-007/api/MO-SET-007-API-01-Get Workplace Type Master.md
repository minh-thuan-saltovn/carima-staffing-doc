# MO-SET-007-API-01-Get Workplace Type Master

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Get Workplace Type Master |
| Description | Lấy danh sách thông tin master loại nơi làm việc của MOTO. |
| Endpoint | /api/v1/moto/settings/workplace-type-master |
| Menu | Company Settings |
| Method | GET |
| Related Tables | tenant_db.mst_moto_workplace_type (được đề xuất thay thế cho workplace_type_master đang để TBD) |
| Screen list | 就業場所区分マスタ (MO-SET-007) - Workplace Type Master |
| System | MOTO Portal |
| 説明 | 勤務場所区分マスタの情報を取得する (Lấy thông tin master loại nơi làm việc). |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Get Workplace Type Master |
| Diagram / Flow áp dụng | BF-001 User Management Flow / Tenant Master Management |
| Portal áp dụng | MOTO Portal |
| Version | v1.0 |
| Ngày tạo | 2026/06/23 |
| Ngày cập nhật | 2026/06/23 |
| Người tạo | Thuận |
| Người review |  |
| Trạng thái | VN Review |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/23 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Get Workplace Type Master. |
| v1.1 | 2026/06/24 | Antigravity | Cập nhật bám sát UI list_workplace_type.png (bổ sung workplace_type_name_en) và cấu trúc ERD; loại bỏ dấu backtick trong các bảng. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | MO-SET-007-API-01 |
| Tên API | Get Workplace Type Master |
| Business Flow liên quan | Tenant Master Management Flow |
| Screen ID liên quan | MO-SET-007 |
| Chức năng liên quan | Workplace Type Master Management |
| Mục đích API | Lấy thông tin danh sách các loại nơi làm việc. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/moto/settings/workplace-type-master |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | VN Review |

---

## 3.2 Mô tả API

### Mục đích
Lấy danh sách các loại nơi làm việc của MOTO từ bảng `mst_moto_workplace_type`.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Mô tả |
| --- | --- |
| Kích hoạt | Khi mở màn hình, tìm kiếm, lọc trạng thái hoặc phân trang |
| Actor | MOTO Admin, MOTO Staff |
| Trước khi gọi | Đã đăng nhập và có quyền workplace_type.view |
| Sau khi gọi | Hiển thị danh sách lên Grid |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/moto/settings/workplace-type-master
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Định dạng dữ liệu |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

## 4.3 Path Parameters

Không áp dụng.

## 4.4 Query Parameters

| Parameter | Type | Bắt buộc | Default | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| page | number | Không | 1 | Số trang cần lấy | 1 |
| limit | number | Không | 20 | Số lượng bản ghi trên một trang | 20 |
| keyword | string | Không | null | Từ khóa tìm kiếm (lọc theo workplace_code hoặc pc_display_name) | オフィス |
| status | number | Không | null | Lọc theo trạng thái hiệu lực | 1 |

---

# 5. Request Body

Không áp dụng cho phương thức GET.

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
  "data": [
    {
      "id": 1,
      "workplace_code": "OFFICE",
      "pc_display_name": "オフィス",
      "pc_display_name_en": "Office",
      "mobile_display_name": "オ",
      "mobile_display_name_en": "Off",
      "valid_from_month": "2026-01",
      "valid_to_month": null,
      "use_fax_flg": 0,
      "created_at": "2026-06-23T08:00:00+09:00",
      "updated_at": "2026-06-23T08:00:00+09:00"
    },
    {
      "id": 2,
      "workplace_code": "HOME",
      "pc_display_name": "自宅",
      "pc_display_name_en": "Home",
      "mobile_display_name": "自",
      "mobile_display_name_en": "Home",
      "valid_from_month": "2026-01",
      "valid_to_month": "2026-05",
      "use_fax_flg": 0,
      "created_at": "2026-06-23T08:05:00+09:00",
      "updated_at": "2026-06-23T08:05:00+09:00"
    }
  ],
  "links": {
    "first": "http://localhost/api/v1/moto/settings/workplace-type-master?page=1",
    "last": "http://localhost/api/v1/moto/settings/workplace-type-master?page=1",
    "prev": null,
    "next": null
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 1,
    "path": "http://localhost/api/v1/moto/settings/workplace-type-master",
    "per_page": 20,
    "to": 2,
    "total": 2
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data | array | Danh sách các loại nơi làm việc |
| data[].id | number | ID tự tăng khóa chính |
| data[].workplace_code | string | Mã loại nơi làm việc |
| data[].pc_display_name | string | Tên hiển thị trên PC (Tiếng Nhật) |
| data[].pc_display_name_en | string | Tên hiển thị trên PC (Tiếng Anh) |
| data[].mobile_display_name | string | Tên hiển thị trên Mobile (Tiếng Nhật) |
| data[].mobile_display_name_en | string | Tên hiển thị trên Mobile (Tiếng Anh) |
| data[].valid_from_month | string | Tháng bắt đầu áp dụng (YYYY-MM) |
| data[].valid_to_month | string | Tháng kết thúc áp dụng (YYYY-MM hoặc null) |
| data[].use_fax_flg | number | Cờ sử dụng cho FAX (1: Có, 0: Không) |
| data[].created_at | datetime | Thời điểm tạo |
| data[].updated_at | datetime | Thời điểm cập nhật |
| links | object | Các link điều hướng phân trang |
| meta | object | Metadata thông tin phân trang |

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
| 1 | keyword | max | 100 | max |
| 2 | status | integer | - | integer |
| 3 | status | in | 0, 1 | in |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ truy vấn dữ liệu từ schema tenant hiện tại qua JWT token. Nghiêm cấm truy cập chéo tenant. |
| BR-002 | Role authorization | Yêu cầu quyền workplace_type.view để thực hiện API. |
| BR-003 | Default Sorting | Kết quả sắp xếp tăng dần theo workplace_code (ASC). |
| BR-004 | Status Calculation | Trạng thái Hoạt động/Hết hiệu lực được tính toán động tại thời điểm truy vấn dựa trên tháng hiện tại so với khoảng [valid_from_month, valid_to_month]. |

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
3. Nếu không có quyền workplace_type.view, trả về 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Query Param | Table | Column | Mô tả |
| --- | --- | --- | --- |
| keyword | tenant_moto_<id>.mst_moto_workplace_type | workplace_code / pc_display_name | Điều kiện tìm kiếm tương đối (LIKE %keyword%) |
| status | tenant_moto_<id>.mst_moto_workplace_type | valid_from_month / valid_to_month | Điều kiện lọc trạng thái hiệu lực (1: Hoạt động, 2: Hết hiệu lực) |

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_moto_<id>.mst_moto_workplace_type | id | data[].id | ID tự tăng khóa chính |
| tenant_moto_<id>.mst_moto_workplace_type | workplace_code | data[].workplace_code | Mã loại nơi làm việc |
| tenant_moto_<id>.mst_moto_workplace_type | pc_display_name | data[].pc_display_name | Tên hiển thị trên PC (Tiếng Nhật) |
| tenant_moto_<id>.mst_moto_workplace_type | pc_display_name_en | data[].pc_display_name_en | Tên hiển thị trên PC (Tiếng Anh) |
| tenant_moto_<id>.mst_moto_workplace_type | mobile_display_name | data[].mobile_display_name | Tên hiển thị trên Mobile (Tiếng Nhật) |
| tenant_moto_<id>.mst_moto_workplace_type | mobile_display_name_en | data[].mobile_display_name_en | Tên hiển thị trên Mobile (Tiếng Anh) |
| tenant_moto_<id>.mst_moto_workplace_type | valid_from_month | data[].valid_from_month | Tháng bắt đầu áp dụng |
| tenant_moto_<id>.mst_moto_workplace_type | valid_to_month | data[].valid_to_month | Tháng kết thúc áp dụng |
| tenant_moto_<id>.mst_moto_workplace_type | use_fax_flg | data[].use_fax_flg | Cờ sử dụng cho FAX |
| tenant_moto_<id>.mst_moto_workplace_type | created_at | data[].created_at | Thời điểm tạo |
| tenant_moto_<id>.mst_moto_workplace_type | updated_at | data[].updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

| Hạng mục | Nội dung |
| --- | --- |
| Transaction | Không áp dụng (GET) |

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Xác thực quyền workplace_type.view và validate request.
2. Truy vấn bảng mst_moto_workplace_type (deleted_at IS NULL) của tenant:
   - status = 1 (Hoạt động): Lọc bản ghi còn hiệu lực (current_month thuộc [valid_from, valid_to]).
   - status = 2 (Hết hiệu lực): Lọc bản ghi ngoài khoảng hiệu lực.
   - keyword: Tìm tương đối theo workplace_code hoặc pc_display_name.
   - Sắp xếp theo workplace_code ASC và phân trang.
3. Trả về HTTP 200 kèm dữ liệu JSON.
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

Không áp dụng cho phương thức GET.

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
| 1 | Auth | Xác thực JWT và kiểm tra quyền workplace_type.view. |
| 2 | Tenant Isolation | Kết nối động tới schema tenant theo JWT, cấm truyền company_id từ client. |
| 3 | SQL Injection | Dùng Eloquent/Parameterized Query để phòng ngừa SQL injection. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.3 giây |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Có (mặc định 20 bản ghi/trang) |
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
| 2 | Request body đã được định nghĩa | Done (Không áp dụng) |
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
