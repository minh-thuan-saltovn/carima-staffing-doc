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
| Người tạo | Phúc |
| Người review | Sang |
| Trạng thái | Draft / VN Review / JP Review / Final |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v0.1 | YYYY/MM/DD | Phúc | Tạo bản đầu tiên |

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

| Field | Column (DB) | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- | --- |
| client_company_id |  | string | Có | 36 | Tồn tại trong client_company | ID công ty phái cử |
| work_location_id |  | string | Có | 36 | Tồn tại trong work_location | ID nơi làm việc |
| job_title |  | string | Có | 100 | Không được rỗng | Tên công việc |
| required_number |  | number | Có | - | Min: 1 | Số lượng cần tuyển |
| start_date |  | date | Có | - | YYYY-MM-DD | Ngày bắt đầu hợp đồng |
| end_date |  | date | Có | - | YYYY-MM-DD / >= start_date | Ngày kết thúc hợp đồng |
| work_start_time |  | time | Có | - | HH:mm | Giờ bắt đầu làm việc |
| work_end_time |  | time | Có | - | HH:mm / > start time | Giờ kết thúc làm việc |
| description |  | string | Không | 2000 | - | Nội dung công việc |

---

# 6. Response Definition

### 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| success | boolean | Kết quả xử lý |
| data.access_token | string | Access Token dùng cho API sau login |
| data.refresh_token | string | Refresh Token dùng để cấp lại Access Token |
| data.token_type | string | Loại token, mặc định Bearer |
| data.expires_in | integer | Thời gian hết hạn Access Token, đơn vị giây |
| data.expires_at | string | Thời điểm hết hạn Access Token |
| data.user.user_id | string | User ID |
| data.user.company_id | string | Company ID của user |
| data.user.last_name_ja | string | Họ tiếng Nhật |
| data.user.first_name_ja | string | Tên tiếng Nhật |
| data.user.email | string | Email user |
| data.user.role_id | string | Role ID |
| data.user.role_name | string | Tên role |
| data.tenant.tenant_id | string | Tenant ID |
| data.tenant.tenant_code | string | Tenant Code |
| data.tenant.tenant_name | string | Tên Tenant |
| data.permissions | array[string] | Danh sách quyền của user |
| message | string | Message trả về |

### 6.3 Handle response status

Error Handling 

---

## 7. Validation Rules

Message list 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | user_id | required | - | required |
| 2 | user_id | max | 100 | max |
| 3 | password | required | - | required |
| 4 | password | max | 255 | max |
| 5 | remember_me | boolean | - | boolean |

# 8. Business Rules

Chi tiết <Tham khảo tài liệu>

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

Chi tiết <Tham khảo tài liệu>

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

# 10. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Draft | Submit | Submitted | Required fields hợp lệ |
| Submitted | Approve | Approved | Người duyệt có quyền |
| Submitted | Reject | Rejected | Bắt buộc nhập lý do reject |
| Submitted | Return | Returned | Bắt buộc nhập lý do trả về |
| Returned | Resubmit | Submitted | User cập nhật và submit lại |

---

# 11. Sequence / Processing Flow

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

# 12. Tài liệu liên quan

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

# 13. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| 1 |  | High / Medium / Low |  |  | Open |
| 2 |  | High / Medium / Low |  |  | Open |

---

# 14. Checklist Review

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
| 16 | Performance considerations đã được review | Not Started / Done |s