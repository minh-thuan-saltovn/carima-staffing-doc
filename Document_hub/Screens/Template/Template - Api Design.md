## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design |
| Diagram / Flow áp dụng | Diagram No. / Tên Flow |
| Portal áp dụng | Platform Manager / SAKI Portal / MOTO Portal / Staff App |
| Version | v0.1 |
| Ngày tạo | YYYY/MM/DD |
| Ngày cập nhật | YYYY/MM/DD |
| Người tạo |  |
| Người review |  |
| Trạng thái | Draft / VN Review / JP Review / Final |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v0.1 | YYYY/MM/DD |  | Tạo bản đầu tiên |
| v0.2 | YYYY/MM/DD |  | Cập nhật sau review |
| v1.0 | YYYY/MM/DD |  | Final |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | API-XXX-001 |
| Tên API |  |
| Business Flow liên quan | Diagram 1 / Diagram 2 / Diagram 3 / Diagram 4 / Diagram 5 |
| Screen ID liên quan | SCR-XXX-001 |
| Chức năng liên quan |  |
| Mục đích API |  |
| Loại API | Frontend API / Backend Internal API / External API |
| Method | GET / POST / PUT / PATCH / DELETE |
| Endpoint | /api/v1/... |
| Authentication | Có / Không |
| Authorization | Có / Không |
| Phạm vi Tenant | Platform Common / Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Có / Không |
| Xử lý file | Không / Upload / Download |
| Trạng thái | Draft / Review / Final |

---

## 3.2 Mô tả API

### Mục đích

Mô tả API này dùng để làm gì.

Ví dụ:

> API này dùng để tạo mới yêu cầu phái cử từ SAKI Portal.
> 
> 
> API sẽ validate điều kiện yêu cầu, lưu dữ liệu vào tenant schema và trả về mã yêu cầu đã tạo.
> 

### Ngữ cảnh nghiệp vụ

Mô tả API được sử dụng ở bước nào trong business flow.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User bấm nút “Submit” |
| Actor | SAKI User / MOTO User / Staff / Platform Admin |
| Trước khi gọi API |  |
| Sau khi gọi API |  |
| Thay đổi trạng thái liên quan | Draft → Submitted |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/job-requests
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| X-Tenant-ID | Có | tenant_001 | Mã định danh tenant |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | ID của resource cần xử lý | JR-000001 |

---

## 4.4 Query Parameters

| Parameter | Type | Bắt buộc | Default | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| page | number | Không | 1 | Số trang | 1 |
| limit | number | Không | 20 | Số record mỗi trang | 20 |
| keyword | string | Không | null | Từ khóa tìm kiếm | Tokyo |
| status | string | Không | null | Lọc theo trạng thái | submitted |

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "client_company_id": "CC-000001",
  "work_location_id": "WL-000001",
  "job_title": "事務スタッフ",
  "required_number": 3,
  "start_date": "2026-07-01",
  "end_date": "2026-09-30",
  "work_start_time": "09:00",
  "work_end_time": "18:00",
  "description": "業務内容の詳細"
}
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| client_company_id | string | Có | 36 | Tồn tại trong client_company | ID công ty phái cử |
| work_location_id | string | Có | 36 | Tồn tại trong work_location | ID nơi làm việc |
| job_title | string | Có | 100 | Không được rỗng | Tên công việc |
| required_number | number | Có | - | Min: 1 | Số lượng cần tuyển |
| start_date | date | Có | - | YYYY-MM-DD | Ngày bắt đầu hợp đồng |
| end_date | date | Có | - | YYYY-MM-DD / >= start_date | Ngày kết thúc hợp đồng |
| work_start_time | time | Có | - | HH:mm | Giờ bắt đầu làm việc |
| work_end_time | time | Có | - | HH:mm / > start time | Giờ kết thúc làm việc |
| description | string | Không | 2000 | - | Nội dung công việc |

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
  "success": true,
  "data": {
    "job_request_id": "JR-000001",
    "status": "submitted",
    "created_at": "2026-06-15T10:00:00+09:00"
  },
  "message": "Job request has been created successfully."
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| success | boolean | Kết quả xử lý |
| data.job_request_id | string | ID yêu cầu đã tạo |
| data.status | string | Trạng thái hiện tại |
| data.created_at | datetime | Thời gian tạo |
| message | string | Message kết quả |

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
| 1 | client_company_id | required | - | required |
| 2 | client_company_id | exists | - | exists |
| 3 | start_date | required | - | required |
| 4 | end_date | required | - | required |
| 5 | end_date | after_or_equal | after_or_equal | after_or_equal |
| 6 | required_number | after_or_equal | after_or_equal | after_or_equal |
| 7 | work_end_time | string | - | string |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | User chỉ được truy cập dữ liệu trong tenant hiện tại. |
| BR-002 | Permission check | Chỉ user có quyền tạo mới mới được thực hiện API này. |
| BR-003 | Status transition | Trạng thái ban đầu sau khi tạo là `submitted`. |
| BR-004 | Duplicate check | Kiểm tra trùng điều kiện yêu cầu nếu nghiệp vụ cần. |
| BR-005 | Audit log | Thao tác tạo mới phải được ghi vào audit log. |

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
1. Validate JWT token
2. Xác định tenant từ X-Tenant-ID hoặc user context
3. Kiểm tra user có thuộc tenant hay không
4. Kiểm tra quyền cần thiết của function
5. Từ chối request nếu user không đủ quyền
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Mô tả |
| --- | --- | --- | --- |
| client_company_id | job_requests | client_company_id | ID công ty phái cử |
| work_location_id | job_requests | work_location_id | ID nơi làm việc |
| job_title | job_requests | job_title | Tên công việc |
| required_number | job_requests | required_number | Số lượng cần tuyển |
| start_date | job_requests | start_date | Ngày bắt đầu hợp đồng |
| end_date | job_requests | end_date | Ngày kết thúc hợp đồng |
| work_start_time | job_requests | work_start_time | Giờ bắt đầu làm việc |
| work_end_time | job_requests | work_end_time | Giờ kết thúc làm việc |
| description | job_requests | description | Nội dung công việc |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| job_requests | id | data.job_request_id | ID yêu cầu đã tạo |
| job_requests | status | data.status | Trạng thái hiện tại |
| job_requests | created_at | data.created_at | Thời gian tạo |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | Insert main record | Trong transaction |
| 4 | Insert detail records | Trong transaction |
| 5 | Insert audit log | Trong transaction |
| 6 | Commit transaction | Trong transaction |
| 7 | Send notification | Sau commit |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Insert DB thất bại | Rollback toàn bộ DB changes |
| 2 | Insert detail thất bại | Rollback main và detail records |
| 3 | Insert audit log thất bại | Rollback nếu audit log là bắt buộc |
| 4 | Gửi notification thất bại | Không rollback nếu dữ liệu đã tạo thành công |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Draft | Submit | Submitted | Required fields hợp lệ |
| Submitted | Approve | Approved | Người duyệt có quyền |
| Submitted | Reject | Rejected | Bắt buộc nhập lý do reject |
| Submitted | Return | Returned | Bắt buộc nhập lý do trả về |
| Returned | Resubmit | Submitted | User cập nhật và submit lại |

---

# 13. Sequence / Processing Flow

```
1. Frontend gọi API
2. Backend validate authentication token
3. Backend xác định tenant schema
4. Backend check permission
5. Backend validate request body
6. Backend check master data tồn tại
7. Backend start DB transaction
8. Backend insert / update data
9. Backend ghi audit log
10. Backend commit transaction
11. Backend gửi notification nếu cần
12. Backend trả response
```

---

# 14. Notification

| Hạng mục | Nội dung |
| --- | --- |
| Có cần notification không | Có / Không |
| Thời điểm gửi | Sau khi commit thành công |
| Đối tượng nhận | Approver / Requester / Admin |
| Kênh gửi | Email / In-app / Slack / Không |
| Template ID | NTF-XXX-001 |

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | CREATE / UPDATE / DELETE / APPROVE / EXPORT |
| Bảng target | job_requests |
| ID record target | job_request_id |
| Dữ liệu ghi log | Before / After / Diff |
| Chính sách lưu trữ | Theo NFR |

---

# 16. Xử lý file

## 16.1 Upload

| Hạng mục | Nội dung |
| --- | --- |
| Có upload file không | Có / Không |
| Max file size | 10MB |
| Định dạng cho phép | pdf, xlsx, csv, png, jpg |
| Nơi lưu trữ | S3 / Local / Khác |
| Virus Scan | Bắt buộc / Không bắt buộc |
| Quy tắc đặt tên file | tenant_id/resource_id/original_filename |
| Bảng lưu metadata | file_attachments |

---

## 16.2 Download

| Hạng mục | Nội dung |
| --- | --- |
| Có download file không | Có / Không |
| Permission Check | Bắt buộc |
| Phương thức download | Presigned URL / Stream |
| Thời gian hết hạn | 5 phút |
| Audit Log | Bắt buộc |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication | Phải validate JWT token. |
| 2 | Authorization | Phải check permission trước khi xử lý. |
| 3 | Tenant Isolation | Không cho phép truy cập dữ liệu cross-tenant. |
| 4 | Input Validation | Tất cả request parameters phải được validate. |
| 5 | SQL Injection Prevention | Sử dụng ORM hoặc parameterized query. |
| 6 | Sensitive Data Masking | Dữ liệu cá nhân cần được mask khi ghi log. |
| 7 | Rate Limit | Áp dụng nếu API public. |
| 8 | Audit Log | Các thao tác quan trọng phải được ghi audit log. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 2 giây |
| Lượng truy cập dự kiến | TBD |
| Pagination | Bắt buộc với API danh sách |
| Index cần thiết | Có / Không |
| Cache cần thiết | Có / Không |
| Bulk Processing | Có / Không |
| Async Processing | Có / Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | Diagram No. |
| Wireframe | WF No. |
| Screen Detail Design | Screen Detail Design ID |
| ERD | ERD Version |
| Permission Matrix | Permission Matrix Version |
| Validation Rule | Validation Rule Document |
| NFR | NFR Document |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| 1 |  | High / Medium / Low |  |  | Open |
| 2 |  | High / Medium / Low |  |  | Open |

---

# 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Not Started / Done |
| 2 | Request body đã được định nghĩa | Not Started / Done |
| 3 | Response body đã được định nghĩa | Not Started / Done |
| 4 | Error response đã được định nghĩa | Not Started / Done |
| 5 | Validation rule đã được định nghĩa | Not Started / Done |
| 6 | Business rule đã được định nghĩa | Not Started / Done |
| 7 | Permission check đã được định nghĩa | Not Started / Done |
| 8 | Tenant isolation rule đã được định nghĩa | Not Started / Done |
| 9 | DB mapping đã được định nghĩa | Not Started / Done |
| 10 | Transaction scope đã được định nghĩa | Not Started / Done |
| 11 | Status transition đã được định nghĩa | Not Started / Done |
| 12 | Audit log đã được định nghĩa | Not Started / Done |
| 13 | Notification rule đã được định nghĩa | Not Started / Done |
| 14 | File handling rule đã được định nghĩa nếu cần | Not Started / Done |
| 15 | Security considerations đã được review | Not Started / Done |
| 16 | Performance considerations đã được review | Not Started / Done |