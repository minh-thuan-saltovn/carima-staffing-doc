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

# 5. UI/UX Layout List

 Screen List

---

# 6. Event Definition

| **Type** | **Event** | **Trigger** | **Permission Key** | **Process/Flow** |
| --- | --- | --- | --- | --- |
| screen | Cancel | Click Cancel Button | system.module.screen.action | Hiển thị popup
Dữ liệu chưa lưu sẽ bị mất.
Bạn có muốn thoát không? |
| api | Initial Load | Mở màn hình | system.module.screen.action |   1. Call API <API ID> |

---

# 9. Error Handling

Reference Link

---

# 10. Related Documents

- Business Flow
- ERD
- API Specification
- Role Matrix
- Approval Flow
- Notification Flow