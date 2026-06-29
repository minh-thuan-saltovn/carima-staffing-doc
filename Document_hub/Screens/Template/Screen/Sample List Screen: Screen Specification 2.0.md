# SCREEN SPECIFICATION

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | SCR-CONTRACT-001 |
| Tên màn hình | Danh sách hợp đồng |
| Tên tiếng Nhật | 契約一覧 |
| Module | Contract Management |
| Chức năng | Quản lý hợp đồng |
| Actor | Admin, Sales, Operation |
| URL | /contracts |
| Priority | P1 |
| Phiên bản | v1.0 |

---

# 2. Mục đích màn hình

Cho phép người dùng:

- Tìm kiếm hợp đồng
- Xem chi tiết hợp đồng
- Tạo mới hợp đồng
- Chỉnh sửa hợp đồng
- Xóa hợp đồng
- Export dữ liệu

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập
- Có quyền CONTRACT_VIEW

## Điều kiện sau

- Hiển thị danh sách hợp đồng theo điều kiện tìm kiếm

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên màn hình |
| --- | --- |
| SCR-DASHBOARD | Dashboard |
| SCR-MENU-CONTRACT | Menu Contract |

---

## Màn hình đích

| Action | Screen ID | Tên màn hình |
| --- | --- | --- |
| Tạo mới | SCR-CONTRACT-002 | Tạo hợp đồng |
| Xem chi tiết | SCR-CONTRACT-003 | Chi tiết hợp đồng |
| Chỉnh sửa | SCR-CONTRACT-004 | Chỉnh sửa hợp đồng |

---

# 5. UI/UX Layout

{image}

---

# 6. Định nghĩa Item màn hình

## Khu vực tìm kiếm

| No | Item | Loại | Format | Bắt buộc | Mô tả |
| --- | --- | --- | --- | --- | --- |
| 1 | Contract No | Textbox | varchar(50) | No | Mã hợp đồng |
| 2 | Client Name | Textbox | varchar(255) | No | Tên khách hàng |
| 3 | Staff Name | Textbox | varchar(255) | No | Tên nhân viên |
| 4 | Status | Dropdown | Enum | No | Trạng thái |
| 5 | Start Date From | Datepicker | yyyy/MM/dd | No | Ngày bắt đầu |
| 6 | Start Date To | Datepicker | yyyy/MM/dd | No | Ngày kết thúc |

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

## Search Contract

### Endpoint

```
GET /api/contracts
```

Request Param

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| contract_no | string | No | SAKI company ID |
| client_name | string | No | Search keyword / キーワード |
| status | string | No | Status |
| page | number | Yes | Page number |
| limit | number | Yes | Page size |

Response

```json
{
  "items": [],
  "total": 100
}
```

---

## Delete Contract

```
DELETE /api/contracts/{id}
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
- NFR