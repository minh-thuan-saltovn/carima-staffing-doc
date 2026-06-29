# SCREEN SPECIFICATION

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

# 6. Định nghĩa Item

## 1. Thông tin cơ bản

| No | Item | Type | Required | Format | DB | Default | Điều kiện hiển thị |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Contract No | Label | No | Auto | contract_no |  | Permission Key: 

 |
| 2 | Contract Name | Textbox | Yes | 200 ký tự | contract_name |  |  |
| 3 | Client | Dropdown | Yes | FK | client_id |  |  |
| 4 | Staff | Dropdown | Yes | FK | staff_id |  |  |
| 5 | Contract Type | Dropdown | Yes | Enum | contract_type |  |  |
| 6 | Save | Button |  |  |  |  | Permission Key: moto.contract.create_contract.create |

---

## 2. Thời gian hợp đồng

| Item | Type | Required | Format | DB | Default | Điều kiện hiển thị |
| --- | --- | --- | --- | --- | --- | --- |
| Start Date | Datepicker | Yes | yyyy/MM/dd | start_date |  |  |
| End Date | Datepicker | Yes | yyyy/MM/dd | end_date |  |  |

---

## Điều kiện làm việc

| Item | Type | Required | DB | Default | Điều kiện hiển thị |
| --- | --- | --- | --- | --- | --- |
| Work Location | Textbox | Yes | work_location |  |  |
| Work Time | Textbox | Yes | work_time |  |  |
| Overtime | Checkbox | No | overtime_flag |  |  |
| Holiday Work | Checkbox | No | holiday_work_flag |  |  |

---

## Ghi chú

| Item | Type | Required |
| --- | --- | --- |
| Memo | Textarea | No |

---

# 7. Validation

Reference Link

---

# 8. Event Definition

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| screen | Cancel | Click Cancel Button | system.module.screen.action | Hiển thị popup
Dữ liệu chưa lưu sẽ bị mất.
Bạn có muốn thoát không? |
| api | Initial Load | Mở màn hình | system.module.screen.action |   1. Call API <API ID> |

# 9. API Mapping

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

# 9. Notification

## Trigger

Create Contract Success

### Receiver

- Contract Manager
- Operation Manager

### Channel

- Email
- In-App Notification

---

# 10. Approval Flow

{Reference Link}

# 11. Error Handling

Reference Link

---

# 12. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix
- Approval Flow
- Notification Flow
- NFR