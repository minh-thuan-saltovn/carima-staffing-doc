# API Detail Design

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design |
| API ID | MO-SET-006-API-02-Update Attendance Type Master |
| API Name | Update Attendance Type Master |
| Business Flow áp dụng | BF-032 |
| Portal áp dụng | MOTO Portal |
| Screen ID liên quan | MO-SET-006 |
| Version | v0.2 |
| Ngày tạo | 2026/06/25 |
| Ngày cập nhật | 2026/06/25 |
| Người tạo | Thuận |
| Người review |  |
| Trạng thái | VN Review |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v0.1 | 2026/06/25 | Thuận | Tạo bản đầu tiên |
| v0.2 | 2026/06/25 | Thuận | Cập nhật cấu trúc trường theo UI mới (略称, 計算区分, 割増率, 給与連携コード, 備考) |

---

## 3. Tổng quan API

### 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | MO-SET-006-API-02-Update Attendance Type Master |
| Tên API | Update Attendance Type Master |
| Tên tiếng Nhật | 勤怠種別マスタ更新 |
| Business Flow liên quan | BF-032 |
| Screen ID liên quan | MO-SET-006 |
| Chức năng liên quan | Quản lý chủng loại chấm công |
| Mục đích API | Cập nhật thông tin cấu hình chủng loại chấm công trong master |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/moto/settings/attendance-type-master/{category_code} |
| Authentication | Yêu cầu JWT token |
| Authorization | Yêu cầu quyền attendance_type.update |
| Phạm vi Tenant | Schema của Tenant hiện tại |
| Có Transaction không | Có |
| Xử lý file | Không |
| Response format | JSON |
| Trạng thái | VN Review |

### 3.2 Mô tả API

API này dùng để cập nhật thông tin chủng loại chấm công bao gồm tên viết tắt, tên chủng loại, trạng thái hoạt động, phân loại tính toán, tỷ lệ phụ trội, mã liên kết lương và ghi chú trong bảng mst_moto_attendance_category dựa trên khóa chính category_code (được truyền trên Path parameter) và company_id (lấy từ JWT token context).

### 3.3 Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Người dùng bấm nút 保存する (Lưu) trên form cập nhật |
| Actor | Admin, Sales, Operation |
| Trước khi gọi API | Đã mở form cập nhật và điền đầy đủ dữ liệu |
| Sau khi gọi API thành công | Cập nhật dữ liệu vào DB, ghi nhận Audit Log và hiển thị Toast thành công |
| Sau khi gọi API thất bại | Giữ nguyên trạng thái form và hiển thị thông báo lỗi tương ứng |
| Thay đổi trạng thái liên quan | Cập nhật các trường short_name, category_name, status, calc_category, premium_rate, payroll_link_code, remarks trong master |

---

## 4. Định nghĩa Endpoint

### 4.1 Endpoint

PATCH /api/v1/moto/settings/attendance-type-master/{category_code}

### 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer token_string | Access Token của user |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept | Có | application/json | Loại dữ liệu response mong muốn |
| Accept-Language | Không | ja | Ngôn ngữ thông điệp lỗi |

### 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả |
| --- | --- | --- | --- |
| category_code | string | Có | Mã chủng loại chấm công cần cập nhật |

### 4.4 Query Parameters

Không có

---

## 5. Request Body

### 5.1 Request Body JSON

```json
{
  "short_name": "残業",
  "category_name": "時間外勤務(更新)",
  "status": 1,
  "calc_category": 2,
  "premium_rate": 25,
  "payroll_link_code": "PAY-OT-001",
  "remarks": "Cập nhật ghi chú"
}
```

### 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| short_name | string | Có | 50 | Required, Max 50 | Tên viết tắt chủng loại chấm công |
| category_name | string | Có | 100 | Required, Max 100 | Tên chủng loại chấm công |
| status | integer | Có | - | Required, In (0, 1) | Trạng thái - 0: vô hiệu, 1: đang dùng |
| calc_category | integer | Có | - | Required, In (1, 2, 3, 4, 5) | Phân loại tính toán (1:通常, 2:残業, 3:深夜, 4:休日, 5:特別) |
| premium_rate | integer | Không | - | Integer, Min 0, Max 100 | Tỷ lệ phụ trội (%) |
| payroll_link_code | string | Không | 50 | Max 50, Half-width alphanumeric | Mã liên kết lương |
| remarks | string | Không | 255 | Max 255 | Ghi chú |

### 5.3 Ghi chú request

Không có

---

## 6. Response Definition

### 6.1 Success Response

HTTP Status: 200 OK

```json
{
  "success": true,
  "data": {
    "category_code": "ATT002",
    "short_name": "残業",
    "category_name": "時間外勤務(更新)",
    "status": 1,
    "calc_category": 2,
    "premium_rate": 25,
    "payroll_link_code": "PAY-OT-001",
    "remarks": "Cập nhật ghi chú",
    "created_at": "2026-06-25T08:00:00+09:00",
    "updated_at": "2026-06-25T15:20:00+09:00"
  }
}
```

### 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| success | boolean | Kết quả xử lý |
| data.category_code | string | Mã chủng loại chấm công |
| data.short_name | string | Tên viết tắt đã cập nhật |
| data.category_name | string | Tên chủng loại đã cập nhật |
| data.status | integer | Trạng thái đã cập nhật (0: vô hiệu, 1: đang dùng) |
| data.calc_category | integer | Phân loại tính toán đã cập nhật |
| data.premium_rate | integer | Tỷ lệ phụ trội đã cập nhật (hoặc null) |
| data.payroll_link_code | string | Mã liên kết lương đã cập nhật (hoặc null) |
| data.remarks | string | Ghi chú đã cập nhật (hoặc null) |
| data.created_at | string | Thời điểm tạo bản ghi |
| data.updated_at | string | Thời điểm cập nhật bản ghi |

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

---

## 7. Validation Rules

[Message list](https://app.notion.com/p/sokucom/Message-list-374f02c407dd8037808eea01e93be8aa?source=copy_link)

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | category_code | required | - | required |
| 2 | category_code | max | 16 | max |
| 3 | short_name | required | - | required |
| 4 | short_name | max | 50 | max |
| 5 | category_name | required | - | required |
| 6 | category_name | max | 100 | max |
| 7 | status | required | - | required |
| 8 | status | in | 0, 1 | in |
| 9 | calc_category | required | - | required |
| 10 | calc_category | in | 1, 2, 3, 4, 5 | in |
| 11 | premium_rate | integer | - | integer |
| 12 | premium_rate | min | 0 | min |
| 13 | premium_rate | max | 100 | max |
| 14 | payroll_link_code | max | 50 | max |
| 15 | remarks | max | 255 | max |

### 7.1 Validation xử lý trước Business Rule

| Step | Nội dung |
| --- | --- |
| 1 | Kiểm tra Authorization Header (Token hợp lệ và chưa hết hạn) |
| 2 | Kiểm tra Content-Type là application/json |
| 3 | Parse JSON request body |
| 4 | Validate các trường bắt buộc và định dạng (short_name, category_name, status, calc_category, premium_rate, payroll_link_code, remarks) |
| 5 | Chuyển sang xử lý Business Rule |

---

## 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ thao tác trên schema tenant hiện tại qua JWT token. Nghiêm cấm ghi dữ liệu chéo tenant. |
| BR-002 | Role authorization | Yêu cầu quyền attendance_type.update để thực hiện API. |
| BR-003 | Database Transaction | Bắt buộc bọc xử lý trong Database Transaction, rollback nếu có lỗi xảy ra. |
| BR-004 | Auditing Fields | Tự động ghi nhận thông tin updated_at và updated_by khi lưu thành công. |
| BR-005 | Resource check | Kiểm tra sự tồn tại của category_code và company_id trước khi cập nhật; trả về 404 nếu không tìm thấy. |

---

## 9. Permission / Authorization

### 9.1 Authentication

| Hạng mục | Nội dung |
| --- | --- |
| API yêu cầu Access Token không | Có |
| Loại Token | Bearer JWT Token |

### 9.2 Authorization

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Admin | Có | Có | Có | Có | Không | Có |
| Tenant Admin | Có | Có | Có | Có | Không | Có |
| MOTO Admin | Có | Có | Có | Không | Không | Có |
| MOTO User | Có | Không | Không | Không | Không | Không |
| SAKI Admin | Không | Không | Không | Không | Không | Không |
| Staff | Không | Không | Không | Không | Không | Không |

---

## 10. Data Mapping

### 10.1 Mapping từ Request vào DB

| Request Param | Table | Column | Mô tả |
| --- | --- | --- | --- |
| category_code (Path) | tenant_db.mst_moto_attendance_category | category_code | Điều kiện tìm kiếm khóa chính để cập nhật |
| company_id (Context) | tenant_db.mst_moto_attendance_category | company_id | Lấy từ JWT token, điều kiện khóa chính |
| short_name (Body) | tenant_db.mst_moto_attendance_category | short_name | Cập nhật tên viết tắt chủng loại chấm công |
| category_name (Body) | tenant_db.mst_moto_attendance_category | category_name | Cập nhật tên chủng loại chấm công |
| status (Body) | tenant_db.mst_moto_attendance_category | status | Cập nhật trạng thái (0: vô hiệu, 1: đang dùng) |
| calc_category (Body) | tenant_db.mst_moto_attendance_category | calc_category | Cập nhật phân loại tính toán |
| premium_rate (Body) | tenant_db.mst_moto_attendance_category | premium_rate | Cập nhật tỷ lệ phụ trội (%) |
| payroll_link_code (Body) | tenant_db.mst_moto_attendance_category | payroll_link_code | Cập nhật mã liên kết lương |
| remarks (Body) | tenant_db.mst_moto_attendance_category | remarks | Cập nhật ghi chú |

### 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_db.mst_moto_attendance_category | category_code | data.category_code | Mã chủng loại chấm công |
| tenant_db.mst_moto_attendance_category | short_name | data.short_name | Tên viết tắt |
| tenant_db.mst_moto_attendance_category | category_name | data.category_name | Tên chủng loại |
| tenant_db.mst_moto_attendance_category | status | data.status | Trạng thái (0: vô hiệu, 1: đang dùng) |
| tenant_db.mst_moto_attendance_category | calc_category | data.calc_category | Phân loại tính toán |
| tenant_db.mst_moto_attendance_category | premium_rate | data.premium_rate | Tỷ lệ phụ trội (%) |
| tenant_db.mst_moto_attendance_category | payroll_link_code | data.payroll_link_code | Mã liên kết lương |
| tenant_db.mst_moto_attendance_category | remarks | data.remarks | Ghi chú |
| tenant_db.mst_moto_attendance_category | created_at | data.created_at | Thời điểm tạo |
| tenant_db.mst_moto_attendance_category | updated_at | data.updated_at | Thời điểm cập nhật |

---

## 11. Database Transaction

| Hạng mục | Nội dung |
| --- | --- |
| Mode | Required |
| Start | Trước khi thực hiện UPDATE câu lệnh SQL |
| Commit | Sau khi UPDATE thành công và không xảy ra bất kỳ Exception nào |
| Rollback | Khi xảy ra bất kỳ SQL Exception hoặc Logic Exception nào trong quá trình xử lý |

---

## 12. Status Transition

Không áp dụng.

---

## 13. Sequence / Processing Flow

1. Client gửi yêu cầu cập nhật kèm theo category_code và request body.
2. API Gateway/Middleware xác thực JWT token và trích xuất company_id, check quyền.
3. Validate đầu vào (Header, Path, Body). Nếu lỗi trả về 422.
4. Bắt đầu DB Transaction.
5. Kiểm tra sự tồn tại của bản ghi cần cập nhật (WHERE category_code = {category_code} AND company_id = {company_id}). Nếu không tồn tại, rollback và trả về 404.
6. Thực hiện cập nhật dữ liệu (short_name, category_name, status, calc_category, premium_rate, payroll_link_code, remarks) vào bảng mst_moto_attendance_category.
7. Ghi Audit Log.
8. Commit Transaction.
9. Trả về HTTP 200 kèm dữ liệu đã cập nhật.

---

## 14. Notification

Không áp dụng.

---

## 15. Audit Log

### 15.1 Audit Log Requirement

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | UPDATE |
| Bảng target | mst_moto_attendance_category |
| ID record target | company_id, category_code |

---

## 16. Xử lý file

Không áp dụng.

---

## 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Tenant Isolation | Bắt buộc lọc theo company_id trích xuất từ JWT token, ngăn chặn sửa đổi chéo công ty |
| 2 | Authorization | Chỉ cho phép người dùng có quyền attendance_type.update gọi API |

---

## 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | ≤ 500ms |
| Index cần thiết | index trên khóa chính composite (company_id, category_code) |

---

## 19. Tài liệu liên quan

- Screen Specification MO-SET-006
- ERD (Carima Data Dictionary)

---

## 20. Open Issues

Không có.

---

## 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | API ID đã được định nghĩa | Done |
| 2 | Endpoint đã được định nghĩa | Done |
| 3 | Method đã được định nghĩa | Done |
| 4 | Request header đã được định nghĩa | Done |
| 5 | Request body đã được định nghĩa bằng JSON chuẩn | Done |
| 6 | Success response đã được định nghĩa bằng JSON chuẩn | Done |
| 7 | Error response đã được định nghĩa bằng JSON chuẩn | Done |
| 8 | Validation rule đã được định nghĩa | Done |
| 9 | Business rule đã được định nghĩa | Done |
| 10 | Permission / Authorization đã được định nghĩa | Done |
| 11 | Data mapping đã được định nghĩa | Done |
| 12 | Transaction scope đã được định nghĩa | Done |
| 13 | Audit Log đã được định nghĩa | Done |
