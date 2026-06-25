# SA-SET-001-API-01-Saki user list

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Saki User List |
| Description | Lấy danh sách người dùng SAKI, hỗ trợ tìm kiếm/lọc và thống kê KPI. |
| Endpoint | /api/v1/saki/settings/users |
| Menu | Settings |
| Method | GET |
| Related Tables | tenant_db.mst_saki_user, tenant_db.mst_saki_department, tenant_db.mst_saki_office, tenant_db.mst_saki_user_credential (giả định lưu thông tin login/lock) |
| Screen list | ユーザー管理 (SA-SET-001) / User Management |
| System | SAKI Portal |
| 説明 | SAKIユーザー一覧を取得し、検索・絞り込み và phân trang, hiển thị KPI thống kê số lượng tài khoản. |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Saki User List |
| Diagram / Flow áp dụng | BF-001 User Management Flow |
| Portal áp dụng | SAKI Portal |
| Version | v1.1 |
| Ngày tạo | 2026/06/22 |
| Ngày cập nhật | 2026/06/22 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | VN Review |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/22 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Saki User List. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-SET-001-API-01-Saki user list |
| Tên API | Saki User List |
| Business Flow liên quan | BF-001 User Management Flow |
| Screen ID liên quan | SA-SET-001 |
| Chức năng liên quan | User Management (SAKI User List) |
| Mục đích API | Lấy danh sách người dùng SAKI, hỗ trợ tìm kiếm, lọc, phân trang và cung cấp số liệu thống kê KPI trong cùng Tenant. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/saki/settings/users |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | VN Review |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để phục vụ màn hình **ユーザー管理 (User Management)** trên SAKI Portal. API thực hiện:
1. Lấy danh sách tài khoản người dùng SAKI thuộc tenant hiện tại của người dùng đăng nhập.
2. Hỗ trợ tìm kiếm theo Họ tên (`user_name`), ID đăng nhập (`user_id`), và các bộ lọc phòng ban (`department_id`), vai trò (`execution_role_id`), trạng thái tài khoản (`status`).
3. Trả về thông tin phân trang và thống kê số lượng tài khoản (Tổng số user, User hoạt động, User bị khóa, User có quyền phê duyệt, User vô hiệu) hiển thị trên các KPI Cards ở đầu trang.

### Ngữ cảnh nghiệp vụ

API được gọi khi người dùng truy cập màn hình Quản lý người dùng (`SA-SET-001`) trên SAKI Portal hoặc thực hiện thao tác lọc, tìm kiếm, sắp xếp và chuyển trang trên màn hình này.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User truy cập màn hình hoặc thay đổi bộ lọc, chuyển trang |
| Actor | SAKI Admin / SAKI User (những người có quyền xem danh sách user) |
| Trước khi gọi API | Đã đăng nhập vào SAKI Portal và có quyền user.view trong phạm vi tenant |
| Sau khi gọi API | Hiển thị danh sách tài khoản người dùng SAKI lên bảng dữ liệu cùng thông tin phân trang và KPI thống kê |
| Thay đổi trạng thái liên quan | Không |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/saki/settings/users
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Định dạng dữ liệu |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

Không áp dụng.

---

## 4.4 Query Parameters

| Parameter | Type | Bắt buộc | Default | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| page | number | Không | 1 | Số trang cần lấy | 1 |
| limit | number | Không | 20 | Số lượng bản ghi mỗi trang | 20 |
| user_name | string | Không | null | Tìm kiếm theo họ tên (last_name_ja, first_name_ja, last_name_kana, first_name_kana) | 山田 |
| user_id | string | Không | null | Tìm kiếm theo ID đăng nhập (user_id) | yamada.taro |
| execution_role_id | string | Không | null | Lọc theo mã vai trò phân quyền | role_admin |
| department_id | string | Không | null | Lọc theo phòng ban trực thuộc | DEP-001 |
| status | number | Không | null | Lọc theo trạng thái tài khoản (0: vô hiệu, 1: hoạt động/hiệu lực, 2: bị khóa) | 1 |
| is_approver_only | number | Không | 0 | Chỉ hiển thị người có quyền phê duyệt (0: Không, 1: Có) | 1 |
| is_active_only | number | Không | 0 | Chỉ hiển thị user hoạt động (0: Không, 1: Có) | 1 |
| sort_column | string | Không | created_at | Cột sắp xếp | created_at |
| sort_direction | string | Không | desc | Hướng sắp xếp (asc / desc) | desc |

---

# 5. Request Body

## 5.1 Request Body Schema

Không áp dụng (Phương thức GET không sử dụng Request Body).

---

## 5.2 Định nghĩa field request

Không áp dụng.

---

# 6. Response Definition

## 6.1 Success Response

### HTTP Status

```
200 OK
```

### Response Body

```json
{
  "data": [
    {
      "user_id": "yamada.taro",
      "user_name": "山田 太郎",
      "last_name_ja": "山田",
      "first_name_ja": "太郎",
      "last_name_kana": "ヤマダ",
      "first_name_kana": "タロウ",
      "email": "yamada.taro@sakicompany.jp",
      "tel": "09012345678",
      "department_id": "DEP-001",
      "department_name": "人事部",
      "office_id": "OFF-001",
      "office_name": "本社",
      "execution_role_id": "role_admin",
      "execution_role_name": "管理者",
      "has_approval_authority": true,
      "show_as_approver_flg": 1,
      "final_approver_flg": 0,
      "status": 1,
      "last_login_at": "2026-04-22T10:30:00+09:00",
      "created_at": "2026-04-01T08:00:00+09:00",
      "updated_at": "2026-04-22T10:30:00+09:00"
    },
    {
      "user_id": "ito.takashi",
      "user_name": "伊藤 隆",
      "last_name_ja": "伊藤",
      "first_name_ja": "隆",
      "last_name_kana": "イトウ",
      "first_name_kana": "タカシ",
      "email": "ito.takashi@sakicompany.jp",
      "tel": "09087654321",
      "department_id": "DEP-001",
      "department_name": "人事部",
      "office_id": "OFF-001",
      "office_name": "本社",
      "execution_role_id": "role_viewer",
      "execution_role_name": "閲覧者",
      "has_approval_authority": false,
      "show_as_approver_flg": 0,
      "final_approver_flg": 0,
      "status": 2,
      "last_login_at": "2026-04-15T14:20:00+09:00",
      "created_at": "2026-04-01T08:00:00+09:00",
      "updated_at": "2026-04-15T14:20:00+09:00"
    }
  ],
  "links": {
    "first": "http://example.com/api/v1/saki/settings/users?page=1",
    "last": "http://example.com/api/v1/saki/settings/users?page=5",
    "prev": null,
    "next": "http://example.com/api/v1/saki/settings/users?page=2"
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 5,
    "path": "http://example.com/api/v1/saki/settings/users",
    "per_page": 10,
    "to": 10,
    "total": 45,
    "kpi": {
      "total_users": 45,
      "active_users": 42,
      "locked_users": 2,
      "approver_users": 12,
      "inactive_users": 3
    }
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data[] | array | Danh sách người dùng SAKI |
| data[].user_id | string | Mã người dùng (ID đăng nhập) |
| data[].user_name | string | Họ tên đầy đủ tiếng Nhật (ghép từ Họ và Tên) |
| data[].last_name_ja | string | Họ (tiếng Nhật) |
| data[].first_name_ja | string | Tên (tiếng Nhật) |
| data[].last_name_kana | string | Họ (katakana) |
| data[].first_name_kana | string | Tên (katakana) |
| data[].email | string | Địa chỉ email |
| data[].tel | string | Số điện thoại liên hệ |
| data[].department_id | string | Mã phòng ban trực thuộc |
| data[].department_name | string | Tên phòng ban hiển thị |
| data[].office_id | string | Mã sự nghiệp sở trực thuộc |
| data[].office_name | string | Tên sự nghiệp sở hiển thị |
| data[].execution_role_id | string | Mã vai trò phân quyền thực thi |
| data[].execution_role_name | string | Tên vai trò hiển thị (Quản lý, Phê duyệt, Nhân sự...) |
| data[].has_approval_authority | boolean | Có quyền phê duyệt không (true nếu show_as_approver_flg = 1) |
| data[].show_as_approver_flg | number | Cờ hiển thị ứng viên phê duyệt |
| data[].final_approver_flg | number | Cờ duyệt cuối |
| data[].status | number | Trạng thái hiển thị (0: Vô hiệu, 1: Hoạt động, 2: Bị khóa) |
| data[].last_login_at | datetime \| null | Thời điểm đăng nhập cuối cùng |
| data[].created_at | datetime | Thời điểm tạo bản ghi |
| data[].updated_at | datetime | Thời điểm cập nhật gần nhất |
| links | object | Các link điều hướng phân trang |
| links.first | string | Link đến trang đầu tiên |
| links.last | string | Link đến trang cuối cùng |
| links.prev | string \| null | Link đến trang trước đó (null nếu đang ở trang đầu) |
| links.next | string \| null | Link đến trang tiếp theo (null nếu đang ở trang cuối) |
| meta | object | Metadata phân trang hệ thống |
| meta.current_page | number | Trang hiện tại |
| meta.from | number | Vị trí bản ghi bắt đầu của trang |
| meta.last_page | number | Trang cuối cùng |
| meta.path | string | Đường dẫn API |
| meta.per_page | number | Số lượng bản ghi trên mỗi trang |
| meta.to | number | Vị trí bản ghi kết thúc của trang |
| meta.total | number | Tổng số bản ghi thỏa mãn bộ lọc |
| **meta.kpi** | object | Số liệu thống kê phục vụ hiển thị KPI Cards |
| meta.kpi.total_users | number | Tổng số người dùng thuộc tenant |
| meta.kpi.active_users | number | Số lượng người dùng đang hoạt động (status = 1 và không bị khóa) |
| meta.kpi.locked_users | number | Số lượng người dùng đang bị khóa tài khoản |
| meta.kpi.approver_users | number | Số lượng người dùng có quyền phê duyệt (show_as_approver_flg = 1) |
| meta.kpi.inactive_users | number | Số lượng người dùng đã bị vô hiệu hóa (status = 0) |

---

## 6.3 Error Response

### Validation Error

```
422 Unprocessable Entity
```

### Unauthorized

```
401 Unauthorized
```

### Forbidden

```
403 Forbidden
```

### Not Found

```
404 Not Found
```

### System Error

```
500 Internal Server Error
```

---


# 7. Validation Rules

Message list 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | page | integer | - | integer |
| 2 | limit | integer | - | integer |
| 3 | user_name | max | 100 | max |
| 4 | user_id | max | 100 | max |
| 5 | department_id | max | 20 | max |
| 6 | execution_role_id | max | 64 | max |
| 7 | status | in | 0, 1, 2 | in |
| 8 | is_approver_only | integer | - | integer |
| 9 | is_active_only | integer | - | integer |
| 10 | sort_column | email | - | email |
| 11 | sort_direction | string | - | string |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ người dùng thuộc tenant hiện tại mới có thể truy cập dữ liệu của tenant đó. Xác định kết nối động tới schema tenant_saki_<company_id> dựa trên JWT context. |
| BR-002 | Role authorization | Chỉ cho phép người dùng thuộc SAKI Portal có vai trò phù hợp và sở hữu quyền user.view truy cập API này. |
| BR-003 | Default pagination & sorting | Nếu page không được gửi, mặc định là 1. Nếu limit không được gửi, mặc định là 20. Nếu sort_column và sort_direction không được gửi, mặc định sắp xếp theo created_at DESC. |
| BR-004 | Search / Filter logic | - Lọc theo user_name: tìm kiếm tương đối (LIKE %keyword%) trên trường ghép last_name_ja \|\| first_name_ja hoặc last_name_kana \|\| first_name_kana. <br> - Lọc theo user_id: tìm kiếm tương đối trên user_id. <br> - Lọc theo status (Trạng thái tài khoản): <br>  &nbsp;&nbsp;+ status = 0 (Vô hiệu): map với mst_saki_user.status = 0. <br>  &nbsp;&nbsp;+ status = 1 (Hoạt động): map với mst_saki_user.status = 1 AND account_locked_flg = 0. <br>  &nbsp;&nbsp;+ status = 2 (Bị khóa): map với mst_saki_user.status = 1 AND account_locked_flg = 1. |
| BR-005 | Join relationship | JOIN bảng mst_saki_user với mst_saki_office qua company_id và office_id để lấy tên văn phòng hiển thị. JOIN với mst_saki_department qua company_id và department_id để lấy tên phòng ban hiển thị. LEFT JOIN với bảng xác thực mst_saki_user_credential qua user_id để lấy last_login_at và account_locked_flg. |
| BR-006 | KPI Calculation | Số liệu thống kê KPI trong meta.kpi được tính bằng cách thực hiện các câu lệnh COUNT gom nhóm (aggregation) trên toàn bộ bảng người dùng của tenant hiện tại mà không bị giới hạn bởi các bộ lọc tìm kiếm (ngoại trừ điều kiện tenant_id) để đảm bảo hiển thị đúng tổng thể số liệu. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Admin | Có | Không | Không | Không | Không | Không |
| Tenant Admin | Có | Không | Không | Không | Không | Không |
| SAKI Admin | Có | Không | Không | Không | Không | Không |
| SAKI User | Có | Không | Không | Không | Không | Không |
| MOTO Admin | Không | Không | Không | Không | Không | Không |
| Staff | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Xác thực token JWT của Client gửi trong header Authorization.
2. Trích xuất thông tin người dùng từ token (User ID, Company ID, Role, Permissions).
3. Xác minh người dùng thuộc SAKI Portal (hoặc Platform Manager với quyền truy cập tương đương).
4. Kiểm tra người dùng có sở hữu quyền "user.view" hay không.
5. Nếu hợp lệ, cho phép xử lý tiếp. Ngược lại, trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Logic truy vấn SQL |
| --- | --- | --- | --- |
| user_name | tenant_saki.mst_saki_user | last_name_ja, first_name_ja, last_name_kana, first_name_kana | WHERE (CONCAT(last_name_ja, first_name_ja) LIKE %user_name% OR CONCAT(last_name_kana, first_name_kana) LIKE %user_name%) |
| user_id | tenant_saki.mst_saki_user | user_id | WHERE user_id LIKE %user_id% |
| department_id | tenant_saki.mst_saki_user | department_id | WHERE department_id = :department_id |
| execution_role_id | tenant_saki.mst_saki_user | execution_role_id | WHERE execution_role_id = :execution_role_id |
| status | tenant_saki.mst_saki_user, tenant_saki.mst_saki_user_credential | status, account_locked_flg | IF status=0: status=0; IF status=1: status=1 AND account_locked_flg=0; IF status=2: status=1 AND account_locked_flg=1 |
| is_approver_only | tenant_saki.mst_saki_user | show_as_approver_flg | WHERE show_as_approver_flg = 1 |
| is_active_only | tenant_saki.mst_saki_user | status | WHERE status = 1 |
| limit | - | - | LIMIT :limit |
| page | - | - | OFFSET (:page - 1) * :limit |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_saki.mst_saki_user | user_id | data[].user_id | Mã người dùng (ID đăng nhập) |
| tenant_saki.mst_saki_user | last_name_ja, first_name_ja | data[].user_name | Ghép Họ và Tên (ví dụ "山田 太郎") |
| tenant_saki.mst_saki_user | last_name_ja | data[].last_name_ja | Họ (tiếng Nhật) |
| tenant_saki.mst_saki_user | first_name_ja | data[].first_name_ja | Tên (tiếng Nhật) |
| tenant_saki.mst_saki_user | last_name_kana | data[].last_name_kana | Họ (katakana) |
| tenant_saki.mst_saki_user | first_name_kana | data[].first_name_kana | Tên (katakana) |
| tenant_saki.mst_saki_user | email | data[].email | Email |
| tenant_saki.mst_saki_user | tel | data[].tel | Số điện thoại |
| tenant_saki.mst_saki_user | department_id | data[].department_id | Mã phòng ban |
| tenant_saki.mst_saki_department | display_name_ja | data[].department_name | Tên phòng ban hiển thị |
| tenant_saki.mst_saki_user | office_id | data[].office_id | Mã sự nghiệp sở |
| tenant_saki.mst_saki_office | display_name_ja | data[].office_name | Tên sự nghiệp sở hiển thị |
| tenant_saki.mst_saki_user | execution_role_id | data[].execution_role_id | Mã vai trò phân quyền |
| - | - | data[].execution_role_name | Tên vai trò hiển thị (ví dụ "管理者") |
| tenant_saki.mst_saki_user | show_as_approver_flg | data[].has_approval_authority | true nếu show_as_approver_flg = 1 |
| tenant_saki.mst_saki_user | show_as_approver_flg | data[].show_as_approver_flg | Cờ hiển thị ứng viên duyệt |
| tenant_saki.mst_saki_user | final_approver_flg | data[].final_approver_flg | Cờ duyệt cuối |
| - | status, account_locked_flg | data[].status | Trạng thái (0: vô hiệu, 1: hoạt động, 2: bị khóa) |
| tenant_saki.mst_saki_user_credential | last_login_at | data[].last_login_at | Thời điểm đăng nhập cuối cùng |
| tenant_saki.mst_saki_user | created_at | data[].created_at | Thời điểm tạo |
| tenant_saki.mst_saki_user | updated_at | data[].updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate query parameters | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | Query list data, count and statistics KPI | Ngoài transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Không áp dụng | Không áp dụng (API đọc dữ liệu GET không có transaction) |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Không áp dụng | Không áp dụng | Không áp dụng | Không áp dụng |

---

# 13. Sequence / Processing Flow

```
1. Client gửi request: GET /api/v1/saki/settings/users kèm theo query parameters.
2. Middleware thực hiện kiểm tra xác thực (Authentication) và giải mã JWT token.
3. Middleware thực hiện kiểm tra phân quyền (Authorization): Xác minh user thuộc SAKI Portal và có quyền "user.view".
4. Controller nhận request, thực hiện Validate Query Parameters theo Validation Rules (Section 7).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm thông tin chi tiết lỗi.
5. Controller xác định tenant của user hiện tại (ví dụ: tenant_id/company_id).
6. Controller thiết lập kết nối động (Dynamic Connection) tới schema tương ứng của tenant (`tenant_saki_<company_id>`).
7. Controller gọi Service thực hiện xây dựng câu lệnh SQL truy vấn:
   - SELECT thông tin từ bảng `mst_saki_user` kết hợp LEFT JOIN với:
     + Bảng `mst_saki_office` qua `company_id` và `office_id` để lấy `display_name_ja` (văn phòng).
     + Bảng `mst_saki_department` qua `company_id` và `department_id` để lấy `display_name_ja` (phòng ban).
     + Bảng `mst_saki_user_credential` qua `user_id` để lấy `last_login_at` và `account_locked_flg`.
   - Áp dụng các bộ lọc tương ứng với Query Parameters (keyword, user_name, user_id, status, department_id, execution_role_id, v.v.).
   - Thực hiện đếm tổng số bản ghi thỏa mãn điều kiện lọc để phục vụ phân trang.
   - Sắp xếp và phân trang theo tham số yêu cầu.
   - Chạy các câu truy vấn đếm KPI tổng quát của Tenant hiện tại (không phụ thuộc vào bộ lọc tìm kiếm) để lấy số liệu: Tổng số user, User hoạt động, Bị khóa, Có quyền phê duyệt, Vô hiệu.
8. Format dữ liệu kết quả thông qua API Resource (định dạng trường, ghép họ tên, tính toán trạng thái 0/1/2...).
9. Controller đóng gói kết quả theo định dạng Paginated Collection của Engineering Handbook, đưa số liệu KPI vào `meta.kpi` và phản hồi về phía Client kèm HTTP 200 OK.
```

---

# 14. Notification

| Hạng mục | Nội dung |
| --- | --- |
| Có cần notification không | Không |
| Thời điểm gửi | Không áp dụng |
| Đối tượng nhận | Không áp dụng |
| Kênh gửi | Không áp dụng |
| Template ID | Không áp dụng |

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Không |
| Loại thao tác | Không áp dụng |
| Bảng target | Không áp dụng |
| ID record target | Không áp dụng |
| Dữ liệu ghi log | Không áp dụng |
| Chính sách lưu trữ | Không áp dụng |

---

# 16. Xử lý file

## 16.1 Upload

| Hạng mục | Nội dung |
| --- | --- |
| Có upload file không | Không |
| Max file size | Không áp dụng |
| Định dạng cho phép | Không áp dụng |
| Nơi lưu trữ | Không áp dụng |
| Virus Scan | Không áp dụng |
| Quy tắc đặt tên file | Không áp dụng |
| Bảng lưu metadata | Không áp dụng |

---

## 16.2 Download

| Hạng mục | Nội dung |
| --- | --- |
| Có download file không | Không |
| Permission Check | Không áp dụng |
| Phương thức download | Không áp dụng |
| Thời gian hết hạn | Không áp dụng |
| Audit Log | Không áp dụng |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Phải xác thực JWT token và kiểm tra quyền user.view của người dùng. |
| 2 | Tenant Isolation | Thiết lập kết nối động tới đúng tenant schema dựa trên thông tin công ty của người dùng đăng nhập để cách ly hoàn toàn dữ liệu giữa các tenant. Không cho phép truyền tham số company_id từ client để thay đổi tenant. |
| 3 | SQL Injection Prevention | Sử dụng Eloquent ORM hoặc Parameterized Query cho tất cả các điều kiện lọc, tìm kiếm. |
| 4 | Parameter Tampering Prevention | Validate chặt chẽ các trường sort_column và sort_direction theo một danh sách cho phép (whitelist) để tránh chèn các câu lệnh SQL độc hại vào mệnh đề ORDER BY. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.8 giây |
| Lượng truy cập dự kiến | Trung bình |
| Pagination | Bắt buộc (Server-side Pagination) |
| Index cần thiết | Index composite (company_id, office_id) và (company_id, department_id) trên mst_saki_user. Index trên created_at để phục vụ sắp xếp mặc định. |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 User Management Flow |
| Wireframe | SA-SET-001 ユーザー管理 (User Management) |
| Screen Detail Design | SA-SET-001 |
| ERD | [Carima-Staffing Data Dictionary / ERD](file:///Users/macbookpro/Thuan/work/carima_staffing/carima-staffing-api/docs/erd/carima-data-dictionary-full.md) |
| Permission Matrix | Portal Permission Matrix |
| Validation Rule | Section 7. Validation Rules |
| NFR | Performance Requirement |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| 1 | Master phân quyền (Execution Roles) | Cần thống nhất danh sách các vai trò quyền thực thi (execution_role_id) để phục vụ lọc trên giao diện. | PM | TBD | Open |

---

# 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Done |
| 2 | Request body đã được định nghĩa | Done (Không áp dụng) |
| 3 | Response body đã được định nghĩa | Done |
| 4 | Error response đã được định nghĩa | Done |
| 5 | Validation rule đã được định nghĩa | Done |
| 6 | Business rule đã được định nghĩa | Done |
| 7 | Permission check đã được định nghĩa | Done |
| 8 | Tenant isolation rule đã được định nghĩa | Done |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done (Không áp dụng) |
| 11 | Status transition đã được định nghĩa | Done (Không áp dụng) |
| 12 | Audit log đã được định nghĩa | Done (Không áp dụng) |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
