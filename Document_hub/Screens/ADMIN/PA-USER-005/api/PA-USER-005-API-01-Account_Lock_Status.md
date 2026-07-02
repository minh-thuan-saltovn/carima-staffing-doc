## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Account Lock Status |
| Diagram / Flow áp dụng | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Portal áp dụng | Platform Manager |
| Version | v0.1 |
| Ngày tạo | 2026/07/01 |
| Ngày cập nhật | 2026/07/01 |
| Người tạo | Thuận |
| Người review | Sang |
| Trạng thái | VN Review |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v0.1 | 2026/07/01 | Thuận | Tạo bản đầu tiên |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | PA-USER-005-API-01 |
| Tên API | Account Lock Status |
| Business Flow liên quan | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Screen ID liên quan | PA-USER-005 |
| Chức năng liên quan | Khóa/Mở khóa tài khoản (Modal) |
| Mục đích API | Lấy thông tin chi tiết về trạng thái đăng nhập sai và khóa tài khoản của user |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/admin/users/lock-management |
| Authentication | Có |
| Authorization | Có (platform.user.account_lock_unlock.view) |
| Phạm vi Tenant | Platform Common (central_db) |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | VN Review |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để truy vấn chi tiết trạng thái đăng nhập và trạng thái khóa tài khoản hiện tại của một quản trị viên Platform, bao gồm số lần nhập sai mật khẩu, cờ khóa và thời hạn khóa, phục vụ hiển thị dữ liệu Readonly trên modal.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Admin chọn user từ danh sách và mở modal Khóa/Mở khóa |
| Actor | Platform SaaS Admin |
| Trước khi gọi API | Client nhận ID user được chọn |
| Sau khi gọi API | Hiển thị các trường thông tin readonly lên Modal |
| Thay đổi trạng thái liên quan | Không thay đổi trạng thái |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/admin/users/lock-management
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token của Platform Admin |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ thông báo lỗi |

---

## 4.3 Path Parameters

Không áp dụng.

---

## 4.4 Query Parameters

| Parameter | Type | Bắt buộc | Default | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| user_id | number | Có | - | ID của người dùng cần kiểm tra trạng thái khóa | 1001 |

---

# 5. Request Body

Không áp dụng cho phương thức GET.

---

# 6. Response Definition

### 6.1 Response Body Schema

```json
{
  "data": {
    "user_id": 1001,
    "login_id": "yamada.t",
    "full_name": "山田太郎",
    "email": "yamada.t@moto.jp",
    "login_fail_count": 3,
    "account_locked_flg": 1,
    "locked_at": "2026-07-01T10:00:00Z",
    "locked_until": "2026-07-01T11:00:00Z",
    "status": 1
  }
}
```

### 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.user_id | number | ID của tài khoản quản trị viên |
| data.login_id | string | Mã đăng nhập tài khoản |
| data.full_name | string | Họ và tên |
| data.email | string | Địa chỉ email |
| data.login_fail_count | number | Số lần đăng nhập sai liên tiếp hiện tại |
| data.account_locked_flg | number | Cờ tài khoản bị khóa (1: Đang khóa, 0: Không khóa) |
| data.locked_at | string | Thời điểm bắt đầu khóa tài khoản |
| data.locked_until | string | Thời điểm tự động mở khóa tài khoản |
| data.status | number | Trạng thái chung của tài khoản (1: Hiệu lực, 0: Vô hiệu) |

### 6.3 Handle response status


---

## 7. Validation Rules

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | user_id | required | - | user_id_required |

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Query target | Dữ liệu được truy vấn từ bảng `mst_platform_user` của `central_db`. |
| BR-002 | Permission check | Chỉ người dùng có quyền `platform.user.account_lock_unlock.view` mới được phép truy cập API này. |
| BR-003 | Lock state evaluation | Nếu cờ `account_locked_flg` = 1, hệ thống hiển thị nút Mở khóa. Ngược lại, hiển thị nút Khóa trên giao diện. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Super Admin | Có | Không | Không | Không | Không | Không |
| Platform Operator | Có | Không | Không | Không | Không | Không |
| Platform Security Admin | Có | Không | Không | Không | Không | Không |
| Tenant Admin/User | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Giải mã JWT token trong header Authorization.
2. Kiểm tra quyền platform.user.account_lock_unlock.view của người dùng.
3. Từ chối yêu cầu và trả về lỗi 403 Forbidden nếu thiếu quyền.
```

---

# 10. Status Transition

Không áp dụng.

---

# 11. Sequence / Processing Flow

```
1. Frontend gửi yêu cầu lấy trạng thái khóa (GET /api/v1/admin/users/lock-management?user_id=1001).
2. Backend xác thực JWT token của Admin.
3. Backend kiểm tra quyền platform.user.account_lock_unlock.view.
4. Backend kiểm tra sự tồn tại của user_id trong bảng mst_platform_user.
5. Backend đọc các cột login_fail_count, account_locked_flg, locked_at, locked_until từ DB.
6. Backend trả về Response thành công (200 OK) chứa thông tin trạng thái.
```

---

# 12. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | Diagram 1 ─ 認証・権限・テナント基本フロー |
| Wireframe | PA-USER-005 (image.png) |
| Screen Detail Design | [PA-USER-005.md](file:///Users/macbookpro/Thuan/work/carima_staffing/carima-staffing-doc/Document_hub/Screens/ADMIN/PA-USER-005/PA-USER-005.md) |
| ERD | [PLATFORM SCHEMA.md](file:///Users/macbookpro/Thuan/work/carima_staffing/carima-staffing-doc/Document_hub/ERD/PLATFORM%20SCHEMA.md) |
| Permission Matrix | Platform Role Matrix |
| Validation Rule | Validation Rule Document |
| NFR | NFR Document |

---

# 13. Open Issues

Không có.

---

# 14. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Not Started |
| 2 | Request body đã được định nghĩa | Not Started |
| 3 | Response body đã được định nghĩa | Not Started |
| 4 | Error response đã được định nghĩa | Not Started |
| 5 | Validation rule đã được định nghĩa | Not Started |
| 6 | Business rule đã được định nghĩa | Not Started |
| 7 | Permission check đã được định nghĩa | Not Started |
| 8 | DB mapping đã được định nghĩa | Not Started |
| 9 | Transaction scope đã được định nghĩa | Not Started |
| 10 | Audit log đã được định nghĩa | Not Started |
| 11 | Notification rule đã được định nghĩa | Not Started |
