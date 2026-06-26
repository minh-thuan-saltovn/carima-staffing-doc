# SA-SET-003-API-01-Get approval group master

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Get Approval Group Master |
| Description | Lấy danh sách nhóm phê duyệt của SAKI. |
| Endpoint | /api/v1/saki/settings/approval-groups |
| Menu | Settings |
| Method | GET |
| Related Tables | tenant_db.mst_saki_approval_group, tenant_db.mst_saki_approval_group_user, tenant_db.mst_saki_user |
| Screen list | 承認ルート一覧 (SA-SET-003) - Approval Route List / Workplace List |
| System | SAKI Portal |
| 説明 | SAKIの承認グループ（承認ルート）一覧を取得し、検索・絞り込み và phân trang. |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Get Approval Group Master |
| Diagram / Flow áp dụng | BF-001 User Management Flow |
| Portal áp dụng | SAKI Portal |
| Version | v1.0 |
| Ngày tạo | 2026/06/22 |
| Ngày cập nhật | 2026/06/22 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/22 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Get Approval Group Master (GET) bám sát 100% ERD thực tế, tuân thủ Template và API Convention. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-SET-003-API-01-Get approval group master |
| Tên API | Get Approval Group Master |
| Business Flow liên quan | BF-001 User Management Flow |
| Screen ID liên quan | SA-SET-003 |
| Chức năng liên quan | Approval Route Management (SAKI Approval Group) |
| Mục đích API | Lấy danh sách nhóm phê duyệt, hỗ trợ tìm kiếm theo tên, lọc theo trạng thái và phân trang trong cùng Tenant. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/saki/settings/approval-groups |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để lấy danh sách các nhóm phê duyệt (承認ルート) cấu hình trong SAKI Portal của tenant hiện tại phục vụ hiển thị lên bảng dữ liệu tại màn hình **承認ルート一覧 (Approval Route List - SA-SET-003)**. API hỗ trợ lọc theo trạng thái hoạt động, tìm kiếm tương đối theo tên nhóm phê duyệt và phân trang.

### Ngữ cảnh nghiệp vụ

API được gọi khi người dùng truy cập màn hình Danh sách tuyến phê duyệt (`SA-SET-003`) hoặc thay đổi các bộ lọc, thực hiện tìm kiếm, chuyển trang trên màn hình này.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User truy cập màn hình hoặc thay đổi bộ lọc, chuyển trang |
| Actor | SAKI Admin / SAKI User (những người có quyền quản trị cài đặt) |
| Trước khi gọi API | Đã đăng nhập vào SAKI Portal và có quyền approval.view |
| Sau khi gọi API | Hiển thị danh sách tuyến phê duyệt cùng thông tin phân trang lên bảng dữ liệu |
| Thay đổi trạng thái liên quan | Không |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/saki/settings/approval-groups
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
| keyword | string | Không | null | Tìm kiếm tương đối theo tên nhóm phê duyệt (group_name_ja) | 特別条項 |
| status | number | Không | null | Lọc theo trạng thái (0: Vô hiệu, 1: Hiệu lực/Hoạt động) | 1 |
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
      "approval_group_id": "APG-001",
      "group_name_ja": "特別条項申請・2段階承認ルート",
      "group_name_en": null,
      "steps_count": 2,
      "approver_flow": "山田 太郎 ➔ 神田 一郎",
      "approvers": [
        {
          "user_id": "yamada.taro",
          "user_name": "山田 太郎",
          "assigned_at": "2026-06-15T10:00:00+09:00"
        },
        {
          "user_id": "kanda.ichiro",
          "user_name": "神田 一郎",
          "assigned_at": "2026-06-15T10:05:00+09:00"
        }
      ],
      "status": 1,
      "created_at": "2026-06-15T10:00:00+09:00",
      "updated_at": "2026-06-20T12:00:00+09:00"
    },
    {
      "approval_group_id": "APG-002",
      "group_name_ja": "勤怠管理・標準1段階ルート",
      "group_name_en": null,
      "steps_count": 1,
      "approver_flow": "宇都宮 健",
      "approvers": [
        {
          "user_id": "utsunomiya.ken",
          "user_name": "宇都宮 健",
          "assigned_at": "2026-06-16T09:00:00+09:00"
        }
      ],
      "status": 1,
      "created_at": "2026-06-16T09:00:00+09:00",
      "updated_at": "2026-06-16T09:00:00+09:00"
    }
  ],
  "links": {
    "first": "http://example.com/api/v1/saki/settings/approval-groups?page=1",
    "last": "http://example.com/api/v1/saki/settings/approval-groups?page=1",
    "prev": null,
    "next": null
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 1,
    "path": "http://example.com/api/v1/saki/settings/approval-groups",
    "per_page": 20,
    "to": 2,
    "total": 2
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data[] | array | Danh sách các nhóm phê duyệt |
| data[].approval_group_id | string | Mã nhóm phê duyệt (khóa chính) |
| data[].group_name_ja | string | Tên nhóm phê duyệt tiếng Nhật (Tên tuyến phê duyệt) |
| data[].group_name_en | string \| null | Tên nhóm phê duyệt tiếng Anh |
| data[].steps_count | number | Tổng số bước phê duyệt của tuyến (tổng số người duyệt gán trong nhóm) |
| data[].approver_flow | string | Chuỗi biểu diễn luồng người duyệt (ghép theo thứ tự gán tăng dần, ví dụ: "山田 太郎 ➔ 神田 一郎") |
| data[].approvers[] | array | Danh sách chi tiết người duyệt của tuyến |
| data[].approvers[].user_id | string | ID của người duyệt |
| data[].approvers[].user_name | string | Họ tên đầy đủ người duyệt |
| data[].approvers[].assigned_at | datetime | Thời điểm gán người duyệt vào nhóm (mst_saki_approval_group_user.created_at) |
| data[].status | number | Trạng thái tuyến phê duyệt (0: Vô hiệu, 1: Hiệu lực/Hoạt động) |
| data[].created_at | datetime | Thời điểm tạo bản ghi nhóm phê duyệt |
| data[].updated_at | datetime | Thời điểm cập nhật bản ghi nhóm phê duyệt |
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

---
### 6.3 Error Response - Validation Error

```
422 Unprocessable Entity
```

### 6.4 Error Response - Invalid Credentials

```
401 Unauthorized
```

### 6.5 Error Response - Account Locked

```
403 Forbidden
```

### 6.6 Error Response - Account Inactive

```
403 Forbidden
```

### 6.7 Error Response - Tenant Not Found

```
404 Not Found
```

### 6.8 Error Response - System Error

```
500 Internal Server Error
```

### 6.9 Error Response - Bad Request

```
400 Bad Request
```

### 6.10 Error Response - Method Not Allowed

```
405 Method Not Allowed
```

### 6.11 Error Response - Too Many Requests

```
429 Too Many Requests
```

### 6.12 Error Response - Service Unavailable

```
503 Service Unavailable
```

### 6.13 Error Response - Gateway Timeout

```
504 Gateway Timeout
```

Untitled


# 7. Validation Rules

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | page | integer | - | integer |
| 2 | limit | integer | - | integer |
| 3 | keyword | max | - | max |
| 4 | status | integer | - | integer |
| 5 | sort_column | string | - | string |
| 6 | sort_direction | string | - | string |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ người dùng thuộc tenant hiện tại mới có thể truy cập cấu hình của tenant đó. Kết nối động tới schema tenant_saki_<company_id> dựa trên JWT context. |
| BR-002 | Role authorization | Chỉ cho phép người dùng thuộc SAKI Portal có vai trò phù hợp và sở hữu quyền approval.view truy cập API này. |
| BR-003 | Default pagination & sorting | Nếu page không được gửi, mặc định là 1. Nếu limit không được gửi, mặc định là 20. Sắp xếp mặc định theo created_at DESC. |
| BR-004 | Target Module Logic on UI | Do bảng mst_saki_approval_group trong ERD không có cột target_module, thông tin **対象モジュール (Target Module)** trên giao diện sẽ được giải quyết ở phía Frontend dựa trên logic phân loại/quy ước đặt tên hoặc liên kết sử dụng nhóm phê duyệt tại các bảng chức năng (ví dụ: gán cho phòng ban nhận phê duyệt chấm công...). API trả về đúng các thuộc tính vật lý hiện có của nhóm phê duyệt. |
| BR-005 | Approver Flow & Steps Count | Do bảng mst_saki_approval_group_user không có cột thứ tự bước duyệt, luồng người duyệt trên UI được Backend ghép động theo thứ tự thời gian gán người dùng vào nhóm (mst_saki_approval_group_user.created_at ASC). Số lượng bước duyệt (steps_count) tương đương với tổng số người dùng được gán vào nhóm phê duyệt này. |

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
4. Kiểm tra người dùng có sở hữu quyền "approval.view" hay không.
5. Nếu hợp lệ, cho phép xử lý tiếp. Ngược lại, trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Logic truy vấn SQL |
| --- | --- | --- | --- |
| keyword | tenant_saki.mst_saki_approval_group | group_name_ja | WHERE group_name_ja LIKE %keyword% |
| status | tenant_saki.mst_saki_approval_group | status | WHERE status = :status |
| limit | - | - | LIMIT :limit |
| page | - | - | OFFSET (:page - 1) * :limit |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_saki.mst_saki_approval_group | approval_group_id | data[].approval_group_id | Mã nhóm phê duyệt |
| tenant_saki.mst_saki_approval_group | group_name_ja | data[].group_name_ja | Tên tuyến phê duyệt |
| tenant_saki.mst_saki_approval_group | group_name_en | data[].group_name_en | Tên tuyến phê duyệt tiếng Anh |
| - | - | data[].steps_count | Đếm số lượng approver của group (COUNT) |
| - | - | data[].approver_flow | Chuỗi luồng người duyệt (ghép theo thời gian gán) |
| tenant_saki.mst_saki_approval_group_user | user_id | data[].approvers[].user_id | ID người duyệt |
| tenant_saki.mst_saki_user | last_name_ja, first_name_ja | data[].approvers[].user_name | Họ tên người duyệt |
| tenant_saki.mst_saki_approval_group_user | created_at | data[].approvers[].assigned_at | Thời điểm gán người duyệt vào nhóm |
| tenant_saki.mst_saki_approval_group | status | data[].status | Trạng thái (0: Vô hiệu, 1: Hiệu lực) |
| tenant_saki.mst_saki_approval_group | created_at | data[].created_at | Thời điểm tạo |
| tenant_saki.mst_saki_approval_group | updated_at | data[].updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate query parameters | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | Query approval groups and relationship users | Ngoài transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Không áp dụng | Không áp dụng |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| Không áp dụng | Không áp dụng | Không áp dụng | Không áp dụng |

---

# 13. Sequence / Processing Flow

```
1. Client gửi request: GET /api/v1/saki/settings/approval-groups kèm theo query parameters.
2. Middleware thực hiện kiểm tra xác thực (Authentication) và giải mã JWT token.
3. Middleware thực hiện kiểm tra phân quyền (Authorization): Xác minh user thuộc SAKI Portal và có quyền "approval.view".
4. Controller nhận request, thực hiện Validate Query Parameters theo Validation Rules (Section 7).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm thông tin chi tiết lỗi.
5. Controller xác định tenant của user hiện tại (ví dụ: tenant_id/company_id).
6. Controller thiết lập kết nối động (Dynamic Connection) tới schema tương ứng của tenant (`tenant_saki_<company_id>`).
7. Controller gọi Service thực hiện xây dựng câu lệnh SQL truy vấn:
   - SELECT thông tin từ bảng `mst_saki_approval_group`.
   - Áp dụng các điều kiện lọc: `keyword`, `status` nếu có giá trị.
   - Sắp xếp và phân trang theo tham số yêu cầu.
   - Lấy danh sách kết quả tuyến phê duyệt từ database.
8. Đối với mỗi tuyến phê duyệt, Service truy vấn bảng `mst_saki_approval_group_user` (kết hợp JOIN với `mst_saki_user` để lấy họ tên) để lấy danh sách người phê duyệt của tuyến đó và sắp xếp theo thời gian gán `created_at` tăng dần.
9. Backend thực hiện logic:
   - Đếm số lượng người phê duyệt (`steps_count`).
   - Ghép chuỗi luồng phê duyệt (`approver_flow`).
10. Format dữ liệu kết quả thông qua API Resource và trả về response JSON kèm HTTP 200 OK.
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
| 1 | Authentication & Authorization | Phải xác thực JWT token và kiểm tra quyền approval.view của người dùng. |
| 2 | Tenant Isolation | Thiết lập kết nối động tới đúng tenant schema dựa trên thông tin công ty của người dùng đăng nhập để cách ly hoàn toàn dữ liệu giữa các tenant. |
| 3 | SQL Injection Prevention | Sử dụng Eloquent ORM hoặc Parameterized Query cho tất cả các điều kiện lọc, tìm kiếm. |
| 4 | Whitelist-based Sorting | Chỉ cho phép sắp xếp theo danh sách cột an toàn được định nghĩa trước. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.8 giây |
| Lượng truy cập dự kiến | Trung bình |
| Pagination | Bắt buộc |
| Index cần thiết | Index composite (company_id, approval_group_id) trên các bảng liên quan. |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 User Management Flow |
| Wireframe | SA-SET-003 承認ルート一覧 |
| Screen Detail Design | SA-SET-003 |
| ERD | Carima-Staffing Data Dictionary / ERD |
| Permission Matrix | Portal Permission Matrix |
| Validation Rule | Section 7. Validation Rules |
| NFR | Performance Requirement |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| - | Không có. | - | - | - | - |

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
