# SCREEN SPECIFICATION

# Màn hình Chi tiết hợp đồng

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | SCR-CONTRACT-003 |
| Tên màn hình | Chi tiết hợp đồng |
| Tên tiếng Nhật | 契約詳細 |
| Module | Contract Management |
| Chức năng | Xem chi tiết hợp đồng |
| Actor | Admin, Sales, Operation |
| URL | /contracts/{contract_id} |
| Phiên bản | v1.0 |
| Priority | P1 |

---

# 2. Mục đích màn hình

Cho phép người dùng xem thông tin chi tiết của một hợp đồng đã được tạo.

Người dùng có thể:

- Xem thông tin cơ bản của hợp đồng
- Xem thông tin khách hàng
- Xem thông tin nhân viên phụ trách
- Xem thời gian hợp đồng
- Xem điều kiện làm việc
- Xem trạng thái hiện tại của hợp đồng
- Xem lịch sử cập nhật
- Chỉnh sửa hợp đồng nếu có quyền
- Xóa hợp đồng nếu có quyền
- Download hợp đồng nếu có quyền

Sau khi truy cập màn hình thành công:

- Hiển thị đầy đủ thông tin hợp đồng
- Hiển thị trạng thái mới nhất của hợp đồng
- Hiển thị lịch sử cập nhật gần nhất

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập
- Có quyền CONTRACT_VIEW
- Contract ID tồn tại trong hệ thống
- Hợp đồng chưa bị xóa vật lý khỏi database

## Điều kiện sau

- Hiển thị thông tin chi tiết hợp đồng
- Hiển thị lịch sử cập nhật của hợp đồng
- Nếu không tìm thấy dữ liệu, hiển thị message lỗi
- Nếu không có quyền truy cập, hiển thị màn hình Access Denied hoặc message lỗi

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình |
| --- | --- |
| SCR-CONTRACT-001 | Danh sách hợp đồng |
| SCR-CONTRACT-004 | Chỉnh sửa hợp đồng |
| SCR-DASHBOARD | Dashboard |

## Màn hình đích

| Action | Screen ID | Tên màn hình |
| --- | --- | --- |
| Back | SCR-CONTRACT-001 | Danh sách hợp đồng |
| Edit | SCR-CONTRACT-004 | Chỉnh sửa hợp đồng |
| Delete Success | SCR-CONTRACT-001 | Danh sách hợp đồng |
| Session Timeout | SCR-LOGIN | Login |

---

# 5. UI/UX Layout

{image}

---

## Nguyên tắc UI/UX

{Tham khảo UI/UX Rules}

## Header Area

- Hiển thị tiêu đề màn hình
- Hiển thị Contract No
- Hiển thị trạng thái hợp đồng bằng Badge
- Các action button đặt ở góc phải màn hình
- Header có thể sticky khi scroll nếu nội dung màn hình dài

## Action Button

| Button | Type | Điều kiện hiển thị |
| --- | --- | --- |
| Back | Secondary Button | Luôn hiển thị |
| Edit | Primary Button | Có quyền CONTRACT_EDIT |
| Delete | Danger Button | Có quyền CONTRACT_DELETE |
| Download | Secondary Button | Có quyền CONTRACT_DOWNLOAD |

## Detail Area

- Các thông tin được hiển thị dạng readonly
- Không cho phép nhập trực tiếp trên màn hình Detail
- Chia thông tin thành từng section rõ ràng
- Label nằm bên trái, value nằm bên phải
- Nếu dữ liệu null hoặc rỗng, hiển thị dấu
- Các section có thể collapse/expand nếu nội dung dài

## Status Badge

| Status | Hiển thị |
| --- | --- |
| Draft | Gray Badge |
| Pending Approval | Yellow Badge |
| Approved | Blue Badge |
| Active | Green Badge |
| Expired | Dark Badge |
| Cancelled | Red Badge |

## History Area

- Hiển thị lịch sử cập nhật theo thứ tự mới nhất ở trên
- Nếu không có lịch sử, hiển thị message: `Không có lịch sử cập nhật`
- Có paging nếu số lượng lịch sử vượt quá limit mặc định
- Có thể reload lại lịch sử bằng nút Refresh

---

# 6. Định nghĩa Item màn hình

## Thông tin cơ bản

| No | Item | Type | Format | DB | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Contract No | Label | varchar(50) | contract_no | Mã hợp đồng |
| 2 | Contract Name | Label | varchar(200) | contract_name | Tên hợp đồng |
| 3 | Contract Type | Label | Enum | contract_type | Loại hợp đồng |
| 4 | Status | Badge | Enum | status | Trạng thái hợp đồng |
| 5 | Created At | Label | yyyy/MM/dd HH:mm | created_at | Ngày tạo |
| 6 | Updated At | Label | yyyy/MM/dd HH:mm | updated_at | Ngày cập nhật cuối |

## Thông tin khách hàng

| No | Item | Type | Format | DB | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Client Name | Label | varchar(255) | clients.client_name | Tên khách hàng |
| 2 | Client Code | Label | varchar(50) | clients.client_code | Mã khách hàng |
| 3 | Department | Label | varchar(255) | clients.department_name | Phòng ban |
| 4 | Contact Person | Label | varchar(255) | clients.contact_person | Người phụ trách |
| 5 | Phone Number | Label | varchar(20) | clients.phone_number | Số điện thoại |
| 6 | Email | Label | varchar(255) | clients.email | Email |

## Thông tin nhân viên

| No | Item | Type | Format | DB | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Staff Name | Label | varchar(255) | staffs.staff_name | Tên nhân viên |
| 2 | Staff Code | Label | varchar(50) | staffs.staff_code | Mã nhân viên |
| 3 | Position | Label | varchar(100) | staffs.position | Chức vụ |
| 4 | Skill | Label | text | staffs.skill | Kỹ năng |
| 5 | Phone Number | Label | varchar(20) | staffs.phone_number | Số điện thoại |
| 6 | Email | Label | varchar(255) | staffs.email | Email |

## Thời gian hợp đồng

| No | Item | Type | Format | DB | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Start Date | Label | yyyy/MM/dd | start_date | Ngày bắt đầu |
| 2 | End Date | Label | yyyy/MM/dd | end_date | Ngày kết thúc |
| 3 | Contract Period | Label | Number | - | Số tháng hợp đồng |
| 4 | Renewal Flag | Label | Yes/No | renewal_flag | Có tự động gia hạn hay không |

## Điều kiện làm việc

| No | Item | Type | Format | DB | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Work Location | Label | varchar(255) | work_location | Địa điểm làm việc |
| 2 | Work Time | Label | varchar(255) | work_time | Thời gian làm việc |
| 3 | Overtime | Label | Yes/No | overtime_flag | Có làm thêm giờ hay không |
| 4 | Holiday Work | Label | Yes/No | holiday_work_flag | Có làm ngày nghỉ hay không |
| 5 | Monthly Unit Price | Label | Number/Currency | monthly_unit_price | Đơn giá theo tháng |
| 6 | Payment Terms | Label | varchar(255) | payment_terms | Điều kiện thanh toán |

## Ghi chú

| No | Item | Type | Format | DB | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Memo | Text Display | text | memo | Ghi chú nội bộ |

## Lịch sử cập nhật

| No | Item | Type | Format | DB | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Action | Label | varchar(50) | contract_histories.action | Hành động |
| 2 | Updated By | Label | varchar(255) | contract_histories.updated_by | Người thực hiện |
| 3 | Updated At | Label | yyyy/MM/dd HH:mm | contract_histories.updated_at | Thời gian thực hiện |
| 4 | Comment | Label | text | contract_histories.comment | Nội dung thay đổi |

---

# 7. Định nghĩa Data Table

## Table lịch sử cập nhật

### tbl_contract_histories

| STT | Item | DB Column | Type | Width |
| --- | --- | --- | --- | --- |
| 1 | Action | action | varchar(50) | 150px |
| 2 | Updated By | updated_by | varchar(255) | 200px |
| 3 | Updated At | updated_at | datetime | 180px |
| 4 | Comment | comment | text | auto |

## Sorting

Cho phép sort:

- Updated At
- Updated By
- Action

## Paging

| Item | Value |
| --- | --- |
| Default | 10 |
| Options | 10,20,50 |
| Server Side | Yes |

---

# 8. Mapping Database

## Table sử dụng

{Liên kết qua ERD}

## contracts

{Liên kết qua ERD}

| Column | Type | Mô tả |
| --- | --- | --- |
| id | bigint | ID hợp đồng |
| contract_no | varchar(50) | Mã hợp đồng |
| contract_name | varchar(200) | Tên hợp đồng |
| client_id | bigint | ID khách hàng |
| staff_id | bigint | ID nhân viên |
| contract_type | varchar(50) | Loại hợp đồng |
| status | varchar(50) | Trạng thái |
| start_date | date | Ngày bắt đầu |
| end_date | date | Ngày kết thúc |
| work_location | varchar(255) | Địa điểm làm việc |
| work_time | varchar(255) | Thời gian làm việc |
| overtime_flag | boolean | Làm thêm giờ |
| holiday_work_flag | boolean | Làm ngày nghỉ |
| monthly_unit_price | decimal | Đơn giá tháng |
| payment_terms | varchar(255) | Điều kiện thanh toán |
| renewal_flag | boolean | Tự động gia hạn |
| memo | text | Ghi chú |
| created_at | datetime | Ngày tạo |
| updated_at | datetime | Ngày cập nhật |

## clients

{Liên kết qua ERD}

| Column | Type | Mô tả |
| --- | --- | --- |
| id | bigint | ID khách hàng |
| client_code | varchar(50) | Mã khách hàng |
| client_name | varchar(255) | Tên khách hàng |
| department_name | varchar(255) | Phòng ban |
| contact_person | varchar(255) | Người phụ trách |
| phone_number | varchar(20) | Số điện thoại |
| email | varchar(255) | Email |

## staffs

{Liên kết qua ERD}

| Column | Type | Mô tả |
| --- | --- | --- |
| id | bigint | ID nhân viên |
| staff_code | varchar(50) | Mã nhân viên |
| staff_name | varchar(255) | Tên nhân viên |
| position | varchar(100) | Chức vụ |
| skill | text | Kỹ năng |
| phone_number | varchar(20) | Số điện thoại |
| email | varchar(255) | Email |

## contract_histories

{Liên kết qua ERD}

| Column | Type | Mô tả |
| --- | --- | --- |
| id | bigint | ID lịch sử |
| contract_id | bigint | ID hợp đồng |
| action | varchar(50) | Hành động |
| updated_by | varchar(255) | Người thực hiện |
| comment | text | Nội dung |
| created_at | datetime | Ngày tạo |

---

# 9. Validation

Màn hình Detail là màn hình readonly nên không có input validation.

Tuy nhiên cần validate các điều kiện xử lý sau:

| Item | Rule | Message |
| --- | --- | --- |
| Contract ID | Required | Contract ID không được để trống |
| Contract ID | Exists | Hợp đồng không tồn tại |
| Permission | CONTRACT_VIEW | Không có quyền truy cập |
| Delete | Status không phải Active | Không thể xóa hợp đồng đang hoạt động |
| Download | File tồn tại | File hợp đồng không tồn tại |

---

# 10. Event Definition

## Initial Load

### Trigger

Mở màn hình Detail

### Flow

1. Lấy Contract ID từ URL
2. Validate Contract ID
3. Call API Get Contract Detail
4. Call API Get Contract Histories
5. Render thông tin chi tiết
6. Render lịch sử cập nhật
7. Kiểm tra permission để hiển thị action button tương ứng

## Back

### Trigger

Click nút Back

### Flow

1. Quay lại màn hình danh sách hợp đồng
2. Giữ lại điều kiện search trước đó nếu có

## Edit

### Trigger

Click nút Edit

### Flow

1. Kiểm tra quyền CONTRACT_EDIT
2. Kiểm tra trạng thái hợp đồng
3. Chuyển sang màn hình Edit

## Delete

### Trigger

Click nút Delete

### Flow

1. Kiểm tra quyền CONTRACT_DELETE
2. Kiểm tra trạng thái hợp đồng
3. Hiển thị popup xác nhận
4. Nếu người dùng confirm, call API Delete Contract
5. Hiển thị message thành công
6. Quay lại màn hình danh sách hợp đồng

### Confirm Message

```
Hợp đồng này sẽ bị xóa.
Bạn có chắc chắn muốn tiếp tục không?
```

## Download

### Trigger

Click nút Download

### Flow

1. Kiểm tra quyền CONTRACT_DOWNLOAD
2. Call API Download Contract
3. Download file PDF
4. Nếu lỗi, hiển thị message lỗi

## Reload History

### Trigger

Click nút Refresh ở khu vực lịch sử

### Flow

1. Call API Get Contract Histories
2. Render lại danh sách lịch sử

---

# 11. API Mapping

## Get Contract Detail

### Endpoint

```
GET /api/contracts/{id}
```

### Request Path Parameter

| Parameter | Type | Required | Mô tả |
| --- | --- | --- | --- |
| id | bigint | Yes | Contract ID |

### Response

```json
{
  "id": 1001,
  "contract_no": "CTR-202606-0001",
  "contract_name": "ABC Contract",
  "contract_type": "DISPATCH",
  "status": "ACTIVE",
  "client": {
    "id": 1,
    "client_code": "CLI-0001",
    "client_name": "ABC Company",
    "department_name": "System Department",
    "contact_person": "Tanaka Taro",
    "phone_number": "090-0000-0000",
    "email": "tanaka@example.com"
  },
  "staff": {
    "id": 10,
    "staff_code": "STF-0010",
    "staff_name": "Suzuki Ichiro",
    "position": "Frontend Engineer",
    "skill": "Vue, Nuxt, TypeScript",
    "phone_number": "080-0000-0000",
    "email": "suzuki@example.com"
  },
  "start_date": "2026-06-01",
  "end_date": "2026-12-31",
  "work_location": "Tokyo",
  "work_time": "09:00 - 18:00",
  "overtime_flag": true,
  "holiday_work_flag": false,
  "monthly_unit_price": 600000,
  "payment_terms": "月末締め翌月末払い",
  "renewal_flag": true,
  "memo": "Internal memo",
  "created_at": "2026-06-01T10:00:00",
  "updated_at": "2026-06-02T15:30:00"
}
```

## Get Contract Histories

### Endpoint

```
GET /api/contracts/{id}/histories
```

### Request Query

```json
{
  "page": 1,
  "limit": 10,
  "sort": "created_at",
  "order": "desc"
}
```

### Response

```json
{
  "items": [
    {
      "id": 1,
      "action": "CREATE",
      "updated_by": "Admin User",
      "comment": "Create contract",
      "created_at": "2026-06-01T10:00:00"
    },
    {
      "id": 2,
      "action": "UPDATE_STATUS",
      "updated_by": "Sales User",
      "comment": "Status changed from Draft to Active",
      "created_at": "2026-06-02T15:30:00"
    }
  ],
  "total": 2
}
```

## Delete Contract

### Endpoint

```
DELETE /api/contracts/{id}
```

### Response

```json
{
  "success": true
}
```

## Download Contract

### Endpoint

```
GET /api/contracts/{id}/download
```

### Response

```json
{
  "file_name": "CTR-202606-0001.pdf",
  "download_url": "https://example.com/download/contracts/CTR-202606-0001.pdf"
}
```

---

# 12. Permission

| Action | Admin | Sales | Operation |
| --- | --- | --- | --- |
| View | O | O | O |
| Edit | O | O | X |
| Delete | O | X | X |
| Download | O | O | O |
| View History | O | O | O |

---

# 13. Message Definition

{Refer message list}

| Code | Message |
| --- | --- |
| MSG001 | Dữ liệu không tồn tại |
| MSG002 | Xóa thành công |
| MSG003 | Không có quyền truy cập |
| MSG004 | Hệ thống đang bận |
| MSG005 | Không thể xóa hợp đồng đang hoạt động |
| MSG006 | File hợp đồng không tồn tại |
| MSG007 | Download thành công |
| MSG008 | Không có lịch sử cập nhật |

---

# 14. Error Handling

{Refer error handling}

| HTTP Code | Action |
| --- | --- |
| 400 | Hiển thị lỗi validate |
| 401 | Chuyển Login |
| 403 | Hiển thị Access Denied |
| 404 | Hiển thị Not Found hoặc message dữ liệu không tồn tại |
| 409 | Hiển thị lỗi conflict trạng thái |
| 500 | Hiển thị System Error |

---

# 15. Audit Log

{Refer Audit log}

| Action | Log |
| --- | --- |
| View Detail | No |
| Edit Click | No |
| Delete | Yes |
| Download | Yes |
| View History | No |

---

# 16. Related Documents

- Business Flow Diagram
- ERD
- API Specification
- Role Matrix
- Wireframe
- Message List
- Error Handling
- Audit Log
- Notification Flow