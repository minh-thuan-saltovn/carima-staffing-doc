# SCREEN SPECIFICATION

# Màn hình Tạo hợp đồng

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | SCR-CONTRACT-002 |
| Tên màn hình | Tạo hợp đồng |
| Tên tiếng Nhật | 契約登録 |
| Module | Contract Management |
| URL | /contracts/create |
| Actor | Admin, Sales |
| Priority | P1 |

---

# 2. Mục đích

Cho phép người dùng tạo mới hợp đồng giữa Khách hàng và Nhân viên.

Sau khi lưu thành công:

- Sinh mã hợp đồng
- Ghi log hệ thống
- Gửi workflow phê duyệt (nếu bật)

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập
- Có quyền CONTRACT_CREATE

## Điều kiện sau

- Tạo mới hợp đồng thành công

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên |
| --- | --- |
| SCR-CONTRACT-001 | Danh sách hợp đồng |

---

## Màn hình đích

| Action | Screen |
| --- | --- |
| Save Success | Contract Detail |
| Cancel | Contract List |
| Save Draft | Stay Current Screen |

---

# 5. UI/UX Layout

{image}

---

# 6. Quy tắc UI/UX

## Contract No

- Readonly
- Chỉ hiển thị sau khi lưu

---

## Save Button

- Disable khi còn lỗi validation
- Loading khi submit

---

## Required Field

Hiển thị:

```
*
```

màu đỏ phía sau label.

---

## Date

Format:

```
yyyy/MM/dd
```

Ví dụ:

```
2026/06/01
```

---

# 7. Định nghĩa Item

## Thông tin cơ bản

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 1 | Contract No | Label | No | Auto | contract_no |
| 2 | Contract Name | Textbox | Yes | 200 ký tự | contract_name |
| 3 | Client | Dropdown | Yes | FK | client_id |
| 4 | Staff | Dropdown | Yes | FK | staff_id |
| 5 | Contract Type | Dropdown | Yes | Enum | contract_type |

---

## Thời gian hợp đồng

| Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- |
| Start Date | Datepicker | Yes | yyyy/MM/dd | start_date |
| End Date | Datepicker | Yes | yyyy/MM/dd | end_date |

---

## Điều kiện làm việc

| Item | Type | Required | DB |
| --- | --- | --- | --- |
| Work Location | Textbox | Yes | work_location |
| Work Time | Textbox | Yes | work_time |
| Overtime | Checkbox | No | overtime_flag |
| Holiday Work | Checkbox | No | holiday_work_flag |

---

## Ghi chú

| Item | Type | Required |
| --- | --- | --- |
| Memo | Textarea | No |

---

# 8. Validation

## Contract Name

| Rule | Message |
| --- | --- |
| Required | Tên hợp đồng không được để trống |
| Max 200 | Tối đa 200 ký tự |

---

## Client

| Rule | Message |
| --- | --- |
| Required | Vui lòng chọn khách hàng |

---

## Staff

| Rule | Message |
| --- | --- |
| Required | Vui lòng chọn nhân viên |

---

## Date

| Rule | Message |
| --- | --- |
| Start <= End | Ngày bắt đầu phải nhỏ hơn ngày kết thúc |

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình

### Process

1. Load Master Client
2. Load Master Staff
3. Load Contract Type

---

## Save Draft

### Trigger

Click Save Draft

### Process

1. Validate cơ bản
2. Lưu trạng thái Draft
3. Hiển thị thông báo

---

## Save

### Trigger

Click Save

### Process

1. Validate toàn bộ
2. Call API Create Contract
3. Sinh Contract No
4. Ghi Audit Log
5. Gửi Notification
6. Chuyển sang Detail Screen

---

## Cancel

### Trigger

Click Cancel

### Process

Hiển thị popup

```
Dữ liệu chưa lưu sẽ bị mất.
Bạn có muốn thoát không?
```

---

# 10. Mapping Database

## contracts

| Column | Type |
| --- | --- |
| id | bigint |
| contract_no | varchar(50) |
| contract_name | varchar(200) |
| client_id | bigint |
| staff_id | bigint |
| contract_type | varchar(50) |
| start_date | date |
| end_date | date |
| work_location | varchar(255) |
| work_time | varchar(255) |
| overtime_flag | boolean |
| holiday_work_flag | boolean |
| memo | text |

---

# 11. API Mapping

## Create Contract

```
POST /api/contracts
```

Request

```json
{
  "contract_name": "ABC Contract",
  "client_id": 1,
  "staff_id": 10,
  "contract_type": "DISPATCH",
  "start_date": "2026-06-01",
  "end_date": "2026-12-31"
}
```

Response

```json
{
  "id": 1001,
  "contract_no": "CTR-202606-0001"
}
```

---

# 12. Notification

## Trigger

Create Contract Success

### Receiver

- Contract Manager
- Operation Manager

### Channel

- Email
- In-App Notification

---

# 13. Approval Flow

## Nếu Approval ON

```
Draft
 ↓
Pending Approval
 ↓
Approved
 ↓
Active
```

---

# 14. Permission

| Action | Admin | Sales | Operation |
| --- | --- | --- | --- |
| Create | O | O | X |
| Save Draft | O | O | X |
| Submit Approval | O | O | X |

---

# 15. Audit Log

| Action | Log |
| --- | --- |
| Create | Yes |
| Save Draft | Yes |
| Submit Approval | Yes |

---

# 16. Error Handling

| Code | Message |
| --- | --- |
| VAL001 | Dữ liệu không hợp lệ |
| VAL002 | Contract Name đã tồn tại |
| SYS001 | Lỗi hệ thống |

---

# 17. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix
- Approval Flow
- Notification Flow