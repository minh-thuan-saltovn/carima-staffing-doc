# SA-SET-003-API-03-Update approval group

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Update Approval Group |
| Description | Cập nhật thông tin nhóm phê duyệt theo ID. |
| Endpoint | /api/v1/saki/settings/approval-groups/{id} |
| Menu | Settings |
| Method | PUT |
| Related Tables | tenant_db.mst_saki_approval_group, tenant_db.mst_saki_approval_group_user |
| Screen list | 承認ルート一覧 (SA-SET-003) - Approval Route List / Workplace List |
| System | SAKI Portal |
| 説明 | SAKI của cập nhật nhóm phê duyệt (tuyến phê duyệt) hiện tại theo ID. |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Update Approval Group |
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
| v1.0 | 2026/06/22 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Update Approval Group (PUT) bám sát ERD thực tế, tuân thủ Template và API Convention. |
| v1.1 | 2026/06/22 | Thuận | Bổ sung trường status vào Request Body để hỗ trợ cập nhật trạng thái hoạt động (利用状態) của tuyến phê duyệt theo UI thực tế. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-SET-003-API-03-Update approval group |
| Tên API | Update Approval Group |
| Business Flow liên quan | BF-001 User Management Flow |
| Screen ID liên quan | SA-SET-003 |
| Chức năng liên quan | Approval Route Management (SAKI Approval Group) |
| Mục đích API | Cập nhật tên tuyến phê duyệt, phân loại module và thay thế danh sách người phê duyệt tương ứng theo ID. |
| Loại API | Frontend API |
| Method | PUT |
| Endpoint | /api/v1/saki/settings/approval-groups/{id} |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này được gọi khi người dùng chỉnh sửa và nhấn lưu một tuyến phê duyệt hiện có trên màn hình **新規承認ルート作成/編集 (Tạo mới/Chỉnh sửa tuyến phê duyệt - SA-SET-003)**. API thực hiện:
1. Cập nhật tên tuyến phê duyệt (`group_name_ja`) và phân loại module (`group_name_en`) trong bảng `mst_saki_approval_group`.
2. Thay thế toàn bộ danh sách người phê duyệt cũ bằng danh sách mới trong bảng `mst_saki_approval_group_user`.
3. Toàn bộ quá trình cập nhật phải được thực hiện trong Database Transaction để đảm bảo tính toàn vẹn dữ liệu.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User bấm nút “Lưu tuyến phê duyệt này” (このルートを保存する) khi đang chỉnh sửa |
| Actor | SAKI Admin / SAKI User (những người có quyền quản trị cài đặt) |
| Trước khi gọi API | Nhập đầy đủ thông tin bắt buộc (Tên tuyến, Chọn module, và danh sách tối thiểu 1 người duyệt) |
| Sau khi gọi API | Tuyến phê duyệt được cập nhật vào DB, chuyển hướng về màn hình Danh sách tuyến phê duyệt |
| Thay đổi trạng thái liên quan | Trạng thái tuyến phê duyệt được giữ nguyên hoặc cập nhật nếu có |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PUT /api/v1/saki/settings/approval-groups/{id}
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Định dạng dữ liệu |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | Mã nhóm phê duyệt cần cập nhật (approval_group_id) | APG-001 |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "route_name": "特別条項申請・2段階承認ルート（更新）",
  "target_module": "36_agreement",
  "status": 1,
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
| status | number | Có | - | Phải thuộc: 0, 1 | Trạng thái hoạt động của nhóm phê duyệt (0: Vô hiệu, 1: Hoạt động) |
| approver_user_ids | array | Có | - | Tối thiểu 1 phần tử, tối đa 3 phần tử | Danh sách mã ID người phê duyệt của tuyến |
| approver_user_ids[] | string | Có | 100 | Tồn tại trong mst_saki_user | ID của người phê duyệt |

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
  "data": {
    "approval_group_id": "APG-001",
    "group_name_ja": "特別条項申請・2段階承認ルート（更新）",
    "group_name_en": "36_agreement",
    "status": 1,
    "created_at": "2026-06-22T13:40:00+09:00",
    "updated_at": "2026-06-22T17:00:00+09:00"
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data | object | Thông tin nhóm phê duyệt vừa được cập nhật |
| data.approval_group_id | string | Mã nhóm phê duyệt |
| data.group_name_ja | string | Tên nhóm phê duyệt tiếng Nhật |
| data.group_name_en | string \| null | Tên nhóm phê duyệt tiếng Anh (mã module) |
| data.status | number | Trạng thái nhóm phê duyệt (0: Vô hiệu, 1: Hoạt động) |
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
    "message": "Approval group not found."
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
| 1 | id (Path) | Bắt buộc | CMS-VAL-23 | IDを入力してください。 |
| 2 | id (Path) | Tồn tại trong mst_saki_approval_group | CMS-VAL-25 | IDが存在していません。 |
| 3 | route_name | Bắt buộc | CMS-VAL-23 | ルート名を入力してください。 |
| 4 | route_name | Tối đa 24 ký tự | CMS-VAL-6 | ルート名は24文字以内で入力してください。 |
| 5 | target_module | Bắt buộc | CMS-VAL-23 | 対象モジュールを入力してください。 |
| 6 | target_module | Phải thuộc danh sách cho phép | CMS-VAL-41 | 選択された対象モジュールは正しくありません。 |
| 7 | status | Bắt buộc | CMS-VAL-23 | ステータスを入力してください。 |
| 8 | status | Phải là số nguyên | CMS-VAL-40 | ステータスは整数で指定してください。 |
| 9 | status | Phải thuộc: 0 hoặc 1 | CMS-VAL-41 | 選択されたステータスは正しくありません。 |
| 10 | approver_user_ids | Bắt buộc | CMS-VAL-23 | 承認者ユーザーID一覧を入力してください。 |
| 11 | approver_user_ids | Phải là mảng | CMS-VAL-62 | 承認者ユーザーID一覧は配列でなくてはなりません。 |
| 12 | approver_user_ids | Tối đa 3 phần tử | CMS-VAL-33 | 承認者ユーザーID一覧には、3以下の値を指定してください。 |
| 13 | approver_user_ids[] | Bắt buộc | CMS-VAL-23 | 承認者ユーザーIDを入力してください。 |
| 14 | approver_user_ids[] | Phải tồn tại trong bảng mst_saki_user của tenant | CMS-VAL-25 | 承認者ユーザーIDが存在していません。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Dữ liệu chỉ được ghi vào schema tenant_saki_<company_id> của tenant hiện tại dựa trên JWT context của user đăng nhập. Không cho phép cập nhật chéo tenant. |
| BR-002 | Role authorization | Chỉ cho phép người dùng thuộc SAKI Portal có vai trò phù hợp và sở hữu quyền approval.edit thực hiện API này. |
| BR-003 | Target Module Mapping | Lưu trực tiếp target_module vào cột group_name_en để phân loại module sử dụng của nhóm phê duyệt mà không làm thay đổi ERD. |
| BR-004 | Transaction control | Mọi thao tác cập nhật vào mst_saki_approval_group và thay thế danh sách người phê duyệt bắt buộc phải nằm trong cùng một Database Transaction. Nếu xảy ra lỗi ở bất kỳ bước nào, rollback toàn bộ thay đổi. |
| BR-005 | Re-creation of Approvers | Để cập nhật danh sách người phê duyệt và giữ được thứ tự duyệt chính xác theo thời gian tạo (created_at ASC):<br>1. Thực hiện DELETE toàn bộ bản ghi người duyệt cũ trong bảng mst_saki_approval_group_user có approval_group_id trùng khớp.<br>2. Thực hiện INSERT lại lần lượt danh sách người duyệt mới từ mảng approver_user_ids[] của request body. |

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

| Request Field / Param | Table | Column | Mô tả |
| --- | --- | --- | --- |
| id (Path) | tenant_saki.mst_saki_approval_group | approval_group_id | Điều kiện WHERE cập nhật |
| route_name | tenant_saki.mst_saki_approval_group | group_name_ja | Tên tuyến phê duyệt (tiếng Nhật) |
| target_module | tenant_saki.mst_saki_approval_group | group_name_en | Mã module áp dụng |
| status | tenant_saki.mst_saki_approval_group | status | Trạng thái hoạt động |
| approver_user_ids[] | tenant_saki.mst_saki_approval_group_user | user_id | ID người dùng gán vào nhóm |
| - | tenant_saki.mst_saki_approval_group_user | company_id | Mã công ty (lấy từ context đăng nhập) |
| - | tenant_saki.mst_saki_approval_group_user | approval_group_id | Mã nhóm phê duyệt cập nhật |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_saki.mst_saki_approval_group | approval_group_id | data.approval_group_id | Mã nhóm phê duyệt |
| tenant_saki.mst_saki_approval_group | group_name_ja | data.group_name_ja | Tên nhóm phê duyệt tiếng Nhật |
| tenant_saki.mst_saki_approval_group | group_name_en | data.group_name_en | Tên nhóm phê duyệt tiếng Anh (mã module) |
| tenant_saki.mst_saki_approval_group | status | data.status | Trạng thái |
| tenant_saki.mst_saki_approval_group | created_at | data.created_at | Thời điểm tạo |
| tenant_saki.mst_saki_approval_group | updated_at | data.updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate request path & body | Ngoài transaction |
| 2 | Check permission | Ngoài transaction |
| 3 | Check target approval group & approver users exist | Ngoài transaction |
| 4 | Open Database Transaction | Trong transaction |
| 5 | UPDATE bản ghi trong mst_saki_approval_group | Trong transaction |
| 6 | DELETE toàn bộ bản ghi cũ trong mst_saki_approval_group_user theo approval_group_id | Trong transaction |
| 7 | INSERT danh sách bản ghi mới vào mst_saki_approval_group_user | Trong transaction |
| 8 | Commit Database Transaction | Trong transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Gặp lỗi ở bất kỳ thao tác UPDATE, DELETE hoặc INSERT nào trong transaction | Rollback toàn bộ các thay đổi trên DB |

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: PUT /api/v1/saki/settings/approval-groups/{id} kèm Request Body.
2. Middleware kiểm tra xác thực JWT và phân quyền (yêu cầu quyền "approval.edit").
3. Validate tham số đầu vào (id, route_name, target_module, status, approver_user_ids). Trả về HTTP 422 nếu lỗi.
4. Xác định tenant từ user context, thiết lập kết nối động tới schema `tenant_saki_<company_id>`.
5. Service thực hiện các thao tác trong Database Transaction:
   - UPDATE bảng `mst_saki_approval_group` (`group_name_ja = route_name`, `group_name_en = target_module`, `status = status`).
   - DELETE toàn bộ người phê duyệt cũ trong `mst_saki_approval_group_user` theo `approval_group_id`.
   - INSERT danh sách người phê duyệt mới từ `approver_user_ids[]` để giữ thứ tự duyệt (`created_at` ASC).
6. Định dạng dữ liệu qua API Resource và trả về response JSON thành công kèm HTTP 200 OK.
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
| Loại thao tác | UPDATE |
| Bảng target | tenant_saki.mst_saki_approval_group, tenant_saki.mst_saki_approval_group_user |
| ID record target | approval_group_id |
| Dữ liệu ghi log | Diff / Before & After |

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
| Quy trình download | Không áp dụng |
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
| Wireframe | SA-SET-003 承認ルート編集 |
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
| 11 | Status transition đã được định nghĩa | Done (Không áp dụng) |
| 12 | Audit log đã được định nghĩa | Done |
| 13 | Notification rule đã được định nghĩa | Done (Không áp dụng) |
| 14 | File handling rule đã được định nghĩa nếu cần | Done (Không áp dụng) |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
