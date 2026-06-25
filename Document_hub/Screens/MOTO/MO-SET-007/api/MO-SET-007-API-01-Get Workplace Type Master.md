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
| Mục đích API | Lấy thông tin danh sách các loại nơi làm việc (Workplace Types) phục vụ hiển thị lên bảng dữ liệu tại màn hình master. Hỗ trợ tìm kiếm, lọc theo trạng thái và phân trang. |
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
Lấy danh sách các loại nơi làm việc của MOTO từ bảng `mst_moto_workplace_type` (hỗ trợ tìm kiếm tương đối, lọc theo trạng thái và phân trang).

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
| keyword | string | Không | null | Từ khóa tìm kiếm (theo mã loại hoặc tên loại nơi làm việc) | 常駐 |
| status | number | Không | null | Lọc theo trạng thái (0: Vô hiệu, 1: Hoạt động) | 1 |

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
      "workplace_type_code": "WPT001",
      "workplace_type_name": "クライアント常駐",
      "workplace_type_name_en": "Client On-site",
      "work_style_type": 1,
      "is_transportation_eligible": 1,
      "status": 1,
      "created_at": "2026-06-23T08:00:00+09:00",
      "updated_at": "2026-06-23T08:00:00+09:00"
    },
    {
      "workplace_type_code": "WPT002",
      "workplace_type_name": "完全在宅勤務",
      "workplace_type_name_en": "Fully Remote",
      "work_style_type": 2,
      "is_transportation_eligible": 0,
      "status": 1,
      "created_at": "2026-06-23T08:05:00+09:00",
      "updated_at": "2026-06-23T08:05:00+09:00"
    },
    {
      "workplace_type_code": "WPT005",
      "workplace_type_name": "旧・社内常駐",
      "workplace_type_name_en": "Former Internal On-site",
      "work_style_type": 1,
      "is_transportation_eligible": 0,
      "status": 0,
      "created_at": "2026-06-23T08:10:00+09:00",
      "updated_at": "2026-06-23T08:10:00+09:00"
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
    "to": 3,
    "total": 3
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data | array | Danh sách các loại nơi làm việc |
| data[].workplace_type_code | string | Mã loại nơi làm việc |
| data[].workplace_type_name | string | Tên loại nơi làm việc |
| data[].workplace_type_name_en | string | Tên loại nơi làm việc (tiếng Anh) |
| data[].work_style_type | number | Hình thức làm việc (1: 常駐 - On-site, 2: 在宅 - Remote, 3: ハイブリッド - Hybrid, 4: 出張 - Business Trip) |
| data[].is_transportation_eligible | number | Đối tượng chi trả phí đi lại (0: Không, 1: Có) |
| data[].status | number | Trạng thái (0: Vô hiệu, 1: Hoạt động) |
| data[].created_at | datetime | Thời điểm tạo |
| data[].updated_at | datetime | Thời điểm cập nhật |
| links | object | Các link điều hướng phân trang |
| meta | object | Metadata thông tin phân trang |

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
        "field": "status",
        "message": "The status field must be 0 or 1."
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
| 1 | page | Không bắt buộc, số nguyên > 0 | CMS-VAL-40 | ページは整数で指定してください。 |
| 2 | limit | Không bắt buộc, số nguyên từ 1 đến 100 | CMS-VAL-40 | 表示件数は整数で指定してください。 |
| 3 | keyword | Không bắt buộc, chuỗi ký tự, tối đa 100 ký tự | CMS-VAL-6 | 検索キーワードは100文字以内で入力してください。 |
| 4 | status | Không bắt buộc, số nguyên | CMS-VAL-40 | ステータスは整数で指定してください。 |
| 5 | status | Không bắt buộc, phải thuộc tập hợp: 0, 1 | CMS-VAL-41 | 選択されたステータスは正しくありません。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ truy vấn dữ liệu từ schema tenant hiện tại qua JWT token. Nghiêm cấm truy cập chéo tenant. |
| BR-002 | Role authorization | Yêu cầu quyền workplace_type.view để thực hiện API. |
| BR-003 | Default Sorting | Kết quả sắp xếp tăng dần theo workplace_type_code (ASC). |

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
| keyword | tenant_moto.mst_moto_workplace_type | workplace_type_code / workplace_type_name | Điều kiện tìm kiếm tương đối (LIKE %keyword%) |
| status | tenant_moto.mst_moto_workplace_type | status | Điều kiện lọc chính xác (= status) |

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_moto.mst_moto_workplace_type | workplace_type_code | data[].workplace_type_code | Mã loại nơi làm việc |
| tenant_moto.mst_moto_workplace_type | workplace_type_name | data[].workplace_type_name | Tên loại nơi làm việc |
| tenant_moto.mst_moto_workplace_type | workplace_type_name_en | data[].workplace_type_name_en | Tên loại nơi làm việc (tiếng Anh) |
| tenant_moto.mst_moto_workplace_type | work_style_type | data[].work_style_type | Hình thức làm việc (1: On-site, 2: Remote, 3: Hybrid, 4: Business Trip) |
| tenant_moto.mst_moto_workplace_type | is_transportation_eligible | data[].is_transportation_eligible | Đối tượng chi trả phí đi lại (0: Không, 1: Có) |
| tenant_moto.mst_moto_workplace_type | status | data[].status | Trạng thái (0: Vô hiệu, 1: Hoạt động) |
| tenant_moto.mst_moto_workplace_type | created_at | data[].created_at | Thời điểm tạo |
| tenant_moto.mst_moto_workplace_type | updated_at | data[].updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

Không áp dụng cho phương thức đọc dữ liệu (GET).

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Nhận request GET kèm query parameters.
2. Xác thực JWT và kiểm tra quyền workplace_type.view.
3. Validate tham số đầu vào (page, limit, keyword, status). Trả về 422 nếu lỗi.
4. Kết nối tới schema tenant dựa trên thông tin JWT.
5. Truy vấn bảng mst_moto_workplace_type:
   - Lọc theo status nếu truyền lên.
   - Lọc theo keyword (LIKE %keyword%) trên workplace_type_code hoặc workplace_type_name.
   - Sắp xếp tăng dần theo workplace_type_code và phân trang.
6. Trả về response JSON của danh sách kết quả kèm HTTP 200.
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
| 1 | Authentication & Authorization | Phải xác thực JWT token và kiểm tra quyền workplace_type.view của người dùng trước khi xử lý. |
| 2 | Tenant Isolation | Thiết lập kết nối động tới đúng tenant schema dựa trên thông tin công ty của người dùng đăng nhập để cách ly dữ liệu. Không cho phép truyền tham số company_id từ client để thay đổi tenant truy vấn dữ liệu. |
| 3 | Input Sanitization & SQL Injection | Sử dụng Eloquent ORM / Parameterized Query để chèn các tham số tìm kiếm (keyword, status) vào câu lệnh SQL để ngăn chặn tấn công SQL injection. |

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
