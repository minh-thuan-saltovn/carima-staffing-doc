# SA-SET-003-API-02-Create approval group

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Create Approval Group |
| Description | Tạo mới nhóm phê duyệt (tuyến phê duyệt). |
| Endpoint | /api/v1/saki/settings/approval-groups |
| Menu | Settings |
| Method | POST |
| Related Tables | tenant_db.mst_saki_approval_group, tenant_db.mst_saki_approval_group_user |
| Screen list | 承認ルート作成 (SA-SET-003) - Create Approval Route / Workplace List |
| System | SAKI Portal |
| 説明 | SAKI của nhóm phê duyệt (tuyến phê duyệt) mới. |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Create Approval Group |
| Diagram / Flow áp dụng | BF-001 User Management Flow |
| Portal áp dụng | SAKI Portal |
| Version | v1.1 |
| Ngày tạo | 2026/06/22 |
| Ngày cập nhật | 2026/06/22 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/22 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Create Approval Group (POST). |
| v1.1 | 2026/06/22 | Thuận | Cập nhật cấu trúc API bám sát 100% cấu trúc cơ sở dữ liệu thực tế (ERD), loại bỏ hoàn toàn các trường không có trong DB ra khỏi Request Body và các tài liệu liên quan. |
| v1.2 | 2026/06/22 | Thuận | Cập nhật nâng cấp tài liệu: Tích hợp thêm trường target_module (bắt buộc) vào Request Body để map vào cột group_name_en trong DB nhằm đáp ứng nghiệp vụ phân loại tuyến phê duyệt theo UI thực tế mà vẫn giữ tính toàn vẹn 100% của ERD vật lý. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-SET-003-API-02-Create approval group |
| Tên API | Create Approval Group |
| Business Flow liên quan | BF-001 User Management Flow |
| Screen ID liên quan | SA-SET-003 |
| Chức năng liên quan | Approval Route Management (SAKI Approval Group) |
| Mục đích API | Tạo mới nhóm phê duyệt (tuyến phê duyệt) và gán danh sách người dùng phê duyệt tương ứng. |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/saki/settings/approval-groups |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này được gọi khi người dùng lưu tuyến phê duyệt mới trên màn hình **新規承認ルート作成 (Tạo tuyến phê duyệt mới)**. API thực hiện:
1. Thêm mới bản ghi vào bảng `mst_saki_approval_group` trong tenant schema của SAKI.
2. Thêm mới danh sách người dùng phê duyệt vào bảng `mst_saki_approval_group_user`.
3. Toàn bộ quá trình tạo mới phải được bọc trong Database Transaction để đảm bảo tính toàn vẹn dữ liệu.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User bấm nút “Lưu tuyến phê duyệt này” (このルートを保存する) |
| Actor | SAKI Admin / SAKI User (những người có quyền quản trị cài đặt) |
| Trước khi gọi API | Nhập đầy đủ thông tin bắt buộc (Tên tuyến và danh sách tối thiểu 1 người duyệt) |
| Sau khi gọi API | Tuyến phê duyệt được lưu vào DB, chuyển hướng về màn hình Danh sách tuyến phê duyệt |
| Thay đổi trạng thái liên quan | Tạo mới bản ghi, trạng thái mặc định của tuyến là 1 (Hiệu lực) |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
POST /api/v1/saki/settings/approval-groups
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

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "route_name": "特別条項申請・2段階承認ルート",
  "target_module": "36_agreement",
  "approver_user_ids": [
    "yamada.taro",
    "kanda.ichiro"
  ]
}
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| route_name | string | Có | 24 | Không rỗng, tối đa 24 ký tự | Tên tuyến phê duyệt (tương đương 12 ký tự toàn giác) |
| target_module | string | Có | 24 | Phải thuộc: dispatch_inquiry, contract, attendance, 36_agreement | Mã module áp dụng tuyến phê duyệt (lưu vào group_name_en) |
| approver_user_ids | array | Có | - | Tối thiểu 1 phần tử, tối đa 3 phần tử | Danh sách mã ID người phê duyệt của tuyến |
| approver_user_ids[] | string | Có | 100 | Tồn tại trong mst_saki_user | ID của người phê duyệt |

---

# 6. Response Definition

## 6.1 Success Response

### HTTP Status

```
201 Created
```

### Response Body

```json
{
  "data": {
    "approval_group_id": "APG-001",
    "group_name_ja": "特別条項申請・2段階承認ルート",
    "group_name_en": "36_agreement",
    "status": 1,
    "created_at": "2026-06-22T13:40:00+09:00",
    "updated_at": "2026-06-22T13:40:00+09:00"
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data | object | Thông tin nhóm phê duyệt vừa tạo |
| data.approval_group_id | string | Mã nhóm phê duyệt vừa được sinh tự động |
| data.group_name_ja | string | Tên nhóm phê duyệt tiếng Nhật |
| data.group_name_en | string \| null | Tên nhóm phê duyệt tiếng Anh |
| data.status | number | Trạng thái nhóm phê duyệt (mặc định 1: Hoạt động) |
| data.created_at | datetime | Thời điểm tạo bản ghi |
| data.updated_at | datetime | Thời điểm cập nhật gần nhất |

---

## 6.3 Error Response

### Validation Error

```
422 Unprocessable Entity
```

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed.",
    "details": [
      {
        "field": "route_name",
        "message": "The route name field is required."
      }
    ]
  }
}
```

### Unauthorized

```
401 Unauthorized
```

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication is required."
  }
}
```

### Forbidden

```
403 Forbidden
```

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to perform this operation."
  }
}
```

### Not Found

```
404 Not Found
```

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Target resource was not found."
  }
}
```

### System Error

```
500 Internal Server Error
```

```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred."
  }
}
```

---

# 7. Validation Rules

| No. | Field | Rule | Error Code | Error Message |
| --- | --- | --- | --- | --- |
| 1 | route_name | Bắt buộc | CMS-VAL-23 | ルート名を入力してください。 |
| 2 | route_name | Tối đa 24 ký tự | CMS-VAL-6 | ルート名は24文字以内で入力してください。 |
| 3 | target_module | Bắt buộc | CMS-VAL-23 | 対象モジュールを入力してください。 |
| 4 | target_module | Phải thuộc danh sách cho phép | CMS-VAL-41 | 選択された対象モジュールは正しくありません。 |
| 5 | approver_user_ids | Bắt buộc | CMS-VAL-23 | 承認者ユーザーID一覧を入力してください。 |
| 6 | approver_user_ids | Phải là mảng | CMS-VAL-62 | 承認者ユーザーID一覧は配列でなくてはなりません。 |
| 7 | approver_user_ids | Tối đa 3 phần tử | CMS-VAL-33 | 承認者ユーザーID一覧には、3以下の値を指定してください。 |
| 8 | approver_user_ids[] | Bắt buộc | CMS-VAL-23 | 承認者ユーザーIDを入力してください。 |
| 9 | approver_user_ids[] | Phải tồn tại trong bảng mst_saki_user của tenant | CMS-VAL-25 | 承認者ユーザーIDが存在していません。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Dữ liệu chỉ được ghi vào schema tenant_saki_<company_id> của tenant hiện tại dựa trên JWT context của user đăng nhập. |
| BR-002 | Role authorization | Chỉ cho phép người dùng thuộc SAKI Portal có vai trò phù hợp và sở hữu quyền approval.edit thực hiện API này. |
| BR-003 | ID Generation | Mã approval_group_id được sinh tự động bằng logic nghiệp vụ của Backend (ví dụ: tự động tăng hoặc dạng chuỗi APG-XXX duy nhất trong công ty). |
| BR-004 | Transaction control | Mọi thao tác ghi vào mst_saki_approval_group và mst_saki_approval_group_user bắt buộc phải nằm trong cùng một Database Transaction. Nếu xảy ra lỗi ở bất kỳ bước nào, rollback toàn bộ thay đổi. |
| BR-005 | Target Module Mapping | Do bảng mst_saki_approval_group không có cột module chuyên biệt, giá trị target_module từ request sẽ được lưu trực tiếp vào cột group_name_en trong database để phân loại và truy vấn sau này. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Admin | Có | Có | Có | Có | Có | Có |
| Tenant Admin | Có | Có | Có | Có | Có | Có |
| SAKI Admin | Có | Có | Có | Không | Có | Có |
| SAKI User | Có | Có | Chỉ dữ liệu của mình | Không | Không | Không |
| MOTO Admin | Không | Không | Không | Không | Không | Không |
| Staff | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Validate JWT token gửi kèm trong header Authorization.
2. Trích xuất thông tin người dùng và quyền hạn từ token.
3. Xác minh người dùng thuộc SAKI Portal và sở hữu quyền "approval.edit".
4. Nếu hợp lệ, cho phép xử lý tiếp. Ngược lại, trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Mô tả |
| --- | --- | --- | --- |
| route_name | tenant_saki.mst_saki_approval_group | group_name_ja | Tên tuyến phê duyệt (tiếng Nhật) |
| target_module | tenant_saki.mst_saki_approval_group | group_name_en | Mã module áp dụng (phục vụ phân loại/lọc) |
| - | tenant_saki.mst_saki_approval_group | status | Trạng thái hoạt động (mặc định 1: Hiệu lực) |
| approver_user_ids[] | tenant_saki.mst_saki_approval_group_user | user_id | ID người dùng gán vào nhóm |
| - | tenant_saki.mst_saki_approval_group_user | company_id | Mã công ty (lấy từ context đăng nhập) |
| - | tenant_saki.mst_saki_approval_group_user | approval_group_id | Mã nhóm phê duyệt vừa tạo |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_saki.mst_saki_approval_group | approval_group_id | data.approval_group_id | Mã nhóm phê duyệt |
| tenant_saki.mst_saki_approval_group | group_name_ja | data.group_name_ja | Tên nhóm phê duyệt tiếng Nhật |
| tenant_saki.mst_saki_approval_group | group_name_en | data.group_name_en | Tên nhóm phê duyệt tiếng Anh |
| tenant_saki.mst_saki_approval_group | status | data.status | Trạng thái |
| tenant_saki.mst_saki_approval_group | created_at | data.created_at | Thời điểm tạo |
| tenant_saki.mst_saki_approval_group | updated_at | data.updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request body | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | Check master data (approver users exist) | Ngoài transaction |
| 4 | Open Database Transaction | Trong transaction |
| 5 | INSERT bản ghi vào mst_saki_approval_group | Trong transaction |
| 6 | INSERT danh sách bản ghi vào mst_saki_approval_group_user (gán user_id, company_id, và approval_group_id) | Trong transaction |
| 7 | Commit Database Transaction | Trong transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Gặp lỗi ở bất kỳ thao tác INSERT nào trong transaction | Rollback toàn bộ các thay đổi trên DB |

---

# 12. Status Transition

| Current Status | Action | Next Status | Condition |
| --- | --- | --- | --- |
| - | Create | Active (1) | Dữ liệu hợp lệ |

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: POST /api/v1/saki/settings/approval-groups kèm Request Body.
2. Middleware thực hiện kiểm tra xác thực (Authentication) và giải mã JWT token.
3. Middleware thực hiện kiểm tra phân quyền (Authorization): Xác minh user thuộc SAKI Portal và có quyền "approval.edit".
4. Controller nhận request, thực hiện Validate dữ liệu Request Body theo Validation Rules (Section 7).
   - Nếu không hợp lệ: Trả về HTTP 422 Unprocessable Entity kèm thông tin chi tiết lỗi.
5. Controller xác định tenant của user hiện tại (ví dụ: tenant_id/company_id).
6. Controller thiết lập kết nối động (Dynamic Connection) tới schema tương ứng của tenant (`tenant_saki_<company_id>`).
7. Service kiểm tra xem các user truyền lên trong `approver_user_ids[]` có tồn tại trong hệ thống SAKI và có hoạt động hay không.
   - Nếu không hợp lệ: Trả về lỗi Validation.
8. Service mở Database Transaction.
9. Service thực hiện INSERT bản ghi tuyến phê duyệt mới vào bảng `mst_saki_approval_group` (lưu `route_name` vào `group_name_ja`, và `target_module` vào `group_name_en`).
10. Service thực hiện INSERT danh sách người duyệt vào bảng `mst_saki_approval_group_user` (vòng lặp gán từng `user_id` trong danh sách).
11. Nếu không có lỗi, Service commit transaction. Ngược lại, rollback toàn bộ và trả về HTTP 500 Internal Server Error.
12. Format dữ liệu kết quả thông qua API Resource và trả về response JSON Created kèm HTTP 201 Created.
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
| Có cần Audit Log không | Có |
| Loại thao tác | CREATE |
| Bảng target | tenant_saki.mst_saki_approval_group, tenant_saki.mst_saki_approval_group_user |
| ID record target | approval_group_id |
| Dữ liệu ghi log | After |

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
| 1 | Authentication & Authorization | Phải xác thực JWT token và kiểm tra quyền approval.edit của người dùng trước khi xử lý. |
| 2 | Tenant Isolation | Thiết lập kết nối động tới đúng tenant schema dựa trên thông tin công ty của người dùng đăng nhập để cách ly dữ liệu. Không cho phép truyền tham số company_id từ client để thay đổi tenant ghi dữ liệu. |
| 3 | Input Sanitization & SQL Injection | Sử dụng Eloquent ORM / Parameterized Query để chèn bản ghi vào cơ sở dữ liệu để ngăn chặn tấn công SQL injection. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.5 giây |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính trên các bảng liên quan. |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 User Management Flow |
| Wireframe | SA-SET-003 承認ルート作成 |
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
| 2 | Request body đã được định nghĩa | Done |
| 3 | Response body đã được định nghĩa | Done |
| 4 | Error response đã được định nghĩa | Done |
| 5 | Validation rule đã được định nghĩa | Done |
| 6 | Business rule đã được định nghĩa | Done |
| 7 | Permission check đã được định nghĩa | Done |
| 8 | Tenant isolation rule đã được định nghĩa | Done |
| 9 | DB mapping đã được định nghĩa | Done |
| 10 | Transaction scope đã được định nghĩa | Done |
| 11 | Status transition đã được định nghĩa | Done |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
