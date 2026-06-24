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

## Nguyên tắc UI/UX

### Search Area

- Luôn hiển thị phía trên
- Sticky khi scroll

### Data Table

- Sort được
- Filter được
- Responsive

### Action Button

- New Contract: Primary Button
- Search: Primary Button
- Reset: Secondary Button
- Delete: Danger Button

---

# 6. Danh sách Item màn hình

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

# 7. Định nghĩa Data Table

## Table hiển thị

### tbl_contracts

| STT | Item | DB Column | Type | Width |
| --- | --- | --- | --- | --- |
| 1 | Contract No | contract_no | varchar(50) | 150px |
| 2 | Client Name | client_name | varchar(255) | 250px |
| 3 | Staff Name | staff_name | varchar(255) | 250px |
| 4 | Start Date | start_date | date | 120px |
| 5 | End Date | end_date | date | 120px |
| 6 | Status | status | enum | 100px |
| 7 | Updated At | updated_at | datetime | 180px |
| 8 | Action | - | button | 180px |

---

## Sorting

Cho phép sort:

- Contract No
- Client Name
- Start Date
- End Date
- Updated At

---

## Paging

| Item | Value |
| --- | --- |
| Default | 20 |
| Options | 20,50,100 |
| Server Side | Yes |

---

# 8. Mapping Database

## Table sử dụng

### contracts

| Column | Type | Description |
| --- | --- | --- |
| id | bigint | PK |
| contract_no | varchar(50) | Mã hợp đồng |
| client_id | bigint | FK |
| staff_id | bigint | FK |
| start_date | date | Ngày bắt đầu |
| end_date | date | Ngày kết thúc |
| status | enum | Trạng thái |
| created_at | datetime | Ngày tạo |
| updated_at | datetime | Ngày cập nhật |

---

### clients

| Column | Type |
| --- | --- |
| id | bigint |
| client_name | varchar(255) |

---

### staffs

| Column | Type |
| --- | --- |
| id | bigint |
| staff_name | varchar(255) |

---

# 9. Validation

| Item | Rule | Message |
| --- | --- | --- |
| Contract No | <= 50 ký tự | Mã hợp đồng tối đa 50 ký tự |
| Date Range | From <= To | Ngày bắt đầu phải nhỏ hơn ngày kết thúc |

---

# 10. Event Definition

## Search

### Trigger

Click nút Search

### Flow

1. Validate dữ liệu
2. Call API Search Contract
3. Render Data Table

---

## Reset

### Trigger

Click Reset

### Flow

1. Clear tất cả điều kiện
2. Reload dữ liệu mặc định

---

## Delete

### Trigger

Click Delete

### Flow

1. Hiển thị popup xác nhận
2. Call API Delete
3. Reload danh sách

---

# 11. API Mapping

## Search Contract

### Endpoint

```
GET /api/contracts
```

Request

```json
{
  "contract_no": "",
  "client_name": "",
  "status": "",
  "page": 1,
  "limit": 20
}
```

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

# 12. Permission

| Action | Admin | Sales | Operation |
| --- | --- | --- | --- |
| View | O | O | O |
| Create | O | O | X |
| Edit | O | O | X |
| Delete | O | X | X |
| Export | O | O | O |

---

# 13. Message Definition

| Code | Message |
| --- | --- |
| MSG001 | Dữ liệu không tồn tại |
| MSG002 | Xóa thành công |
| MSG003 | Không có quyền truy cập |
| MSG004 | Hệ thống đang bận |

---

# 14. Error Handling

| HTTP Code | Action |
| --- | --- |
| 400 | Hiển thị lỗi validate |
| 401 | Chuyển Login |
| 403 | Hiển thị Access Denied |
| 500 | Hiển thị System Error |

---

# 15. Audit Log

| Action | Log |
| --- | --- |
| Search | No |
| Create | Yes |
| Update | Yes |
| Delete | Yes |

---

# 16. Related Documents

- Business Flow Diagram
- ERD
- API Specification
- Role Matrix
- Wireframe
- NFR