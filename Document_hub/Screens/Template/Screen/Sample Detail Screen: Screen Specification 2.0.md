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

# 6. Định nghĩa Item màn hình

## Thông tin cơ bản

| No | Item | Type | Format | DB | Default | Điều kiện hiển thị |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Contract No | Label | varchar(50) | contract_no |  |  |
| 2 | Contract Name | Label | varchar(200) | contract_name |  |  |
| 3 | Contract Type | Label | Enum | contract_type |  |  |
| 4 | Status | Badge | Enum | status |  |  |
| 5 | Created At | Label | yyyy/MM/dd HH:mm | created_at |  |  |
| 6 | Updated At | Label | yyyy/MM/dd HH:mm | updated_at |  |  |

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

---

# 9. API Mapping

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

## Query Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| page | number | Yes | Page number |
| limit | number | Yes | Page size |
| sort | string | No | Sort target column |
| order | string | No | Sort direction: `asc` or `desc` |

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

# 10. Message Definition

Reference Link

---

# 11. Error Handling

Reference Link

---

# 12. Related Documents

- Business Flow Diagram
- ERD
- API Specification
- Role Matrix
- Wireframe
- Message List
- Error Handling
- Audit Log
- Notification Flow