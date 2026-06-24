# SA-SET-001-API-03-Update Saki User

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Update Saki User |
| Description | Cập nhật thông tin tài khoản người dùng SAKI theo ID. |
| Endpoint | /api/v1/saki/settings/users/{id} |
| Menu | Settings |
| Method | PUT |
| Related Tables | tenant_db.mst_saki_user, tenant_db.mst_saki_user_scope |
| Screen list | ユーザー管理 (SA-SET-001) / User Management (Detail/Edit) |
| System | SAKI Portal |
| 説明 | SAKIユーザー情報を更新する。 |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Update Saki User |
| Diagram / Flow áp dụng | BF-001 User Management Flow |
| Portal áp dụng | SAKI Portal |
| Version | v1.0 |
| Ngày tạo | 2026/06/23 |
| Ngày cập nhật | 2026/06/23 |
| Người tạo | Thuận |
| Người review | Sang |
| Trạng thái | VN Review |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/23 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Update Saki User. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-SET-001-API-03-Update Saki User |
| Tên API | Update Saki User |
| Business Flow liên quan | BF-001 User Management Flow |
| Screen ID liên quan | SA-SET-001 |
| Chức năng liên quan | User Management (SAKI) |
| Mục đích API | Cập nhật thông tin chi tiết tài khoản người dùng SAKI. |
| Loại API | Frontend API |
| Method | PUT |
| Endpoint | /api/v1/saki/settings/users/{id} |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | VN Review |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để cập nhật thông tin và phân quyền. API thực hiện:
1. Xác thực người dùng và kiểm tra quyền quản trị của SAKI Admin.
2. Validate dữ liệu gửi lên bao gồm thông tin cơ bản, thiết lập phê duyệt và danh sách scope liên quan.
3. Cập nhật dữ liệu vào bảng mst_saki_user trong tenant schema tương ứng.
4. Cập nhật danh sách phạm vi truy cập dữ liệu của user trong bảng mst_saki_user_scope.

### Ngữ cảnh nghiệp vụ

API được gọi khi quản trị viên thực hiện chỉnh sửa cấu hình một user và nhấn nút Lưu thay đổi trên màn hình chỉnh sửa người dùng SAKI.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User click nút Lưu trên UI |
| Actor | SAKI Admin |
| Trước khi gọi API | Màn hình hiển thị form chỉnh sửa thông tin user, cho phép admin cấu hình thông tin và phân quyền |
| Sau khi gọi API | Hiển thị Toast thông báo thành công, tải lại dữ liệu mới nhất của user |
| Thay đổi trạng thái liên quan | mst_saki_user được cập nhật thông tin mới nhất |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PUT /api/v1/saki/settings/users/{id}
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer eyJhbGciOi... | JWT access token của SAKI Admin |
| X-Tenant-ID | Có | tenant_saki_001 | Mã định danh tenant hiện tại |
| Content-Type | Có | application/json | Định dạng dữ liệu |
| Accept-Language | Không | ja | Ngôn ngữ thông báo lỗi |

---

## 4.3 Path Parameters

| Parameter | Type | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- |
| id | string | Có | ID của user SAKI cần chỉnh sửa (user_id) | yamada.taro |

---

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "last_name_ja": "山田",
  "first_name_ja": "太郎",
  "last_name_kana": "ヤマダ",
  "first_name_kana": "タロウ",
  "last_name_en": "Yamada",
  "middle_name_en": null,
  "first_name_en": "Taro",
  "office_id": "OFF00000000000000001",
  "department_id": "DEP00000000000000001",
  "position_ja": "部長",
  "position_en": "Manager",
  "tel": "03-1234-5678",
  "fax": "03-1234-5679",
  "email": "yamada.taro@saki.co.jp",
  "mail_language": "ja",
  "execution_role_id": "role_saki_admin",
  "reference_scope": 6,
  "edit_on_approval_flg": 0,
  "view_group_company_flg": 0,
  "edit_contract_dept_flg": 0,
  "show_as_approver_flg": 1,
  "final_approver_flg": 1,
  "approval_group_id": "APG00001",
  "attendance_approver1_id": "approver_user_01",
  "attendance_approver2_id": null,
  "attendance_approver3_id": null,
  "cost_center_code": "CC001",
  "cost_center_comment": "Nhân sự",
  "status": 1,
  "user_scopes": [
    {
      "scope_type": 1,
      "target_id": "OFF00000000000000001"
    },
    {
      "scope_type": 2,
      "target_id": "DEP00000000000000001"
    }
  ]
}
```

---

## 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| last_name_ja | string | Có | 24 | Không rỗng | Họ tiếng Nhật |
| first_name_ja | string | Có | 24 | Không rỗng | Tên tiếng Nhật |
| last_name_kana | string | Có | 24 | Katakana toàn giác | Họ Katakana |
| first_name_kana | string | Có | 24 | Katakana toàn giác | Tên Katakana |
| last_name_en | string | Không | 24 | Half-width | Họ tiếng Anh |
| middle_name_en | string | Không | 12 | Half-width | Tên đệm tiếng Anh |
| first_name_en | string | Không | 24 | Half-width | Tên tiếng Anh |
| office_id | string | Có | 20 | Tồn tại trong mst_saki_office | ID sự nghiệp sở trực thuộc |
| department_id | string | Có | 20 | Tồn tại trong mst_saki_department | ID phòng ban trực thuộc |
| position_ja | string | Không | 50 | - | Chức vụ tiếng Nhật |
| position_en | string | Không | 50 | Half-width | Chức vụ tiếng Anh |
| tel | string | Có | 15 | Half-width chữ số và gạch | Số điện thoại |
| fax | string | Không | 15 | Half-width chữ số và gạch | Số Fax |
| email | string | Có | 128 | Định dạng email hợp lệ | Địa chỉ email |
| mail_language | string | Có | 2 | ja hoặc en | Ngôn ngữ gửi mail thông báo |
| execution_role_id | string | Có | 64 | Tồn tại trong master role | Mã vai trò phân quyền thực thi |
| reference_scope | number | Có | - | 1, 2, 3, 4, 5, 6 | Phạm vi truy cập dữ liệu |
| edit_on_approval_flg | number | Có | - | 0 hoặc 1 | Cờ cho sửa thông tin khi duyệt hợp đồng |
| view_group_company_flg | number | Có | - | 0 hoặc 1 | Cờ tham chiếu nhóm công ty |
| edit_contract_dept_flg | number | Có | - | 0 hoặc 1 | Cờ sửa phòng ban trên hợp đồng |
| show_as_approver_flg | number | Có | - | 0 hoặc 1 | Cờ hiển thị làm người phê duyệt |
| final_approver_flg | number | Có | - | 0 hoặc 1 | Cờ duyệt cuối |
| approval_group_id | string | Không | 16 | Tồn tại trong mst_saki_approval_group | Mã nhóm nhận yêu cầu phê duyệt |
| attendance_approver1_id | string | Không | 100 | Tồn tại trong mst_saki_user | Người duyệt chấm công 1 |
| attendance_approver2_id | string | Không | 100 | Tồn tại trong mst_saki_user | Người duyệt chấm công 2 |
| attendance_approver3_id | string | Không | 100 | Tồn tại trong mst_saki_user | Người duyệt chấm công 3 |
| cost_center_code | string | Không | 20 | Không chứa Katakana half-width | Mã cost center |
| cost_center_comment | string | Không | 500 | - | Ghi chú mã cost center |
| status | number | Có | - | 0 hoặc 1 | Trạng thái (0: vô hiệu, 1: hoạt động) |
| user_scopes | array | Không | - | Validate các phần tử | Mảng scope (chỉ bắt buộc khi reference_scope = 6) |
| user_scopes[].scope_type | number | Có (trong array) | - | 1 (sự nghiệp sở), 2 (phòng ban) | Loại phạm vi tham chiếu tùy chọn |
| user_scopes[].target_id | string | Có (trong array) | 20 | Tồn tại trong office/department tương ứng | ID đối tượng tham chiếu |

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
    "user_id": "yamada.taro",
    "updated_at": "2026-06-23T11:45:00+09:00"
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data.user_id | string | ID của user SAKI đã được cập nhật thành công |
| data.updated_at | datetime | Thời điểm ghi nhận cập nhật thành công |

---

## 6.3 Error Response

### Validation Error

```
422 Unprocessable Entity
```

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dữ liệu không hợp lệ.",
    "details": [
      {
        "field": "email",
        "message": "Địa chỉ email không đúng định dạng."
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
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Yêu cầu xác thực tài khoản."
  }
}
```

### Forbidden

```
403 Forbidden
```

```json
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "Bạn không có quyền thực hiện thao tác này."
  }
}
```

### Not Found

```
404 Not Found
```

```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Không tìm thấy người dùng SAKI cần cập nhật."
  }
}
```

### System Error

```
500 Internal Server Error
```

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "Đã xảy ra lỗi hệ thống nghiêm trọng."
  }
}
```

---

# 7. Validation Rules

| No. | Field | Rule | Error Code | Error Message |
| --- | --- | --- | --- | --- |
| 1 | id (path) | Bắt buộc | CMS-VAL-23 | IDを入力してください。 |
| 2 | id (path) | Tồn tại trong mst_saki_user | CMS-VAL-25 | IDが存在していません。 |
| 3 | last_name_ja | Bắt buộc nhập | CMS-VAL-23 | 姓（和文）を入力してください。 |
| 4 | last_name_ja | Tối đa 24 ký tự | CMS-VAL-6 | 姓（和文）は24文字以内で入力してください。 |
| 5 | first_name_ja | Bắt buộc nhập | CMS-VAL-23 | 名（和文）を入力してください。 |
| 6 | first_name_ja | Tối đa 24 ký tự | CMS-VAL-6 | 名（和文）は24文字以内で入力してください。 |
| 7 | last_name_kana | Bắt buộc nhập | CMS-VAL-23 | 姓（カナ）を入力してください。 |
| 8 | last_name_kana | Phải là Katakana toàn giác | CMS-VAL-72 | 姓（カナ）は全角カタカナで入力してください。 |
| 9 | first_name_kana | Bắt buộc nhập | CMS-VAL-23 | 名（カナ）を入力してください。 |
| 10 | first_name_kana | Phải là Katakana toàn giác | CMS-VAL-72 | 名（カナ）は全角カタカナで入力してください。 |
| 11 | office_id | Bắt buộc nhập | CMS-VAL-23 | 事業所IDを入力してください。 |
| 12 | office_id | Phải tồn tại trong mst_saki_office | CMS-VAL-25 | 事業所IDが存在していません。 |
| 13 | department_id | Bắt buộc nhập | CMS-VAL-23 | 部門IDを入力してください。 |
| 14 | department_id | Phải tồn tại trong mst_saki_department | CMS-VAL-25 | 部門IDが存在していません。 |
| 15 | email | Bắt buộc nhập | CMS-VAL-23 | メールアドレスを入力してください。 |
| 16 | email | Đúng định dạng email | CMS-VAL-48 | メールアドレスには、有効なメールアドレスを指定してください。 |
| 17 | tel | Bắt buộc nhập | CMS-VAL-23 | 電話番号を入力してください。 |
| 18 | tel | Tối đa 15 ký tự half-width | CMS-VAL-6 | 電話番号は15文字以内で入力してください。 |
| 19 | reference_scope | Bắt buộc nhập | CMS-VAL-23 | 参照範囲を入力してください。 |
| 20 | reference_scope | Giá trị từ 1 đến 6 | CMS-VAL-41 | 選択された参照範囲は正しくありません。 |
| 21 | user_scopes | Bắt buộc nhập khi reference_scope = 6 | CMS-VAL-23 | user_scopesを入力してください。 |
| 22 | user_scopes[].target_id | Tồn tại trong office_id hoặc department_id | SCOPE_TARGET_NOT_FOUND | Đối tượng tham chiếu chỉ định không tồn tại. |
| 23 | cost_center_code | Không chứa ký tự Katakana half-width | CMS-VAL-24 | コストセンターコードに正しい形式を指定してください。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant Isolation | Người dùng chỉ được cập nhật thông tin user thuộc cùng công ty (company_id) trong tenant schema hiện hành. Không cho phép sửa đổi user chéo tenant. |
| BR-002 | Email Duplicate Check | Kiểm tra email cập nhật không được trùng với các user khác đang hoạt động trong cùng tenant (ngoại trừ chính user đó). |
| BR-003 | Admin Permission Check | Chỉ những tài khoản SAKI có vai trò quản trị (SAKI Admin) sở hữu quyền quản lý user (user.update) mới được gọi API này. |
| BR-004 | Scope Sync Logic | Khi reference_scope = 6 (Tùy chọn), thực hiện đồng bộ dữ liệu vào bảng mst_saki_user_scope. Nếu reference_scope khác 6, tự động xóa tất cả bản ghi scope liên kết cũ của user_id này. |
| BR-005 | Audit Logging | Lưu vết toàn bộ thay đổi trước (Before) và sau (After) của các cột được cập nhật dữ liệu. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| SAKI Admin | Có | Có | Có (Quyền user.update) | Không | Có | Có |
| SAKI Staff | Có | Không | Không | Không | Không | Không |
| MOTO Portal Users | Không | Không | Không | Không | Không | Không |
| Staff App Users | Không | Không | Không | Không | Không | Không |

---

## 9.2 Logic kiểm tra quyền

```
1. Middleware kiểm tra xác thực JWT token từ Header Authorization.
2. Trích xuất thông tin user và tenant context.
3. Kiểm tra user.role trong tenant hiện tại có thuộc nhóm SAKI Admin hay không.
4. Kiểm tra user hiện tại có sở hữu quyền "user.update" hay không.
5. Nếu không đáp ứng bất kỳ điều kiện nào, từ chối yêu cầu và trả về lỗi HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Mô tả |
| --- | --- | --- | --- |
| last_name_ja | mst_saki_user | last_name_ja | Họ tiếng Nhật |
| first_name_ja | mst_saki_user | first_name_ja | Tên tiếng Nhật |
| last_name_kana | mst_saki_user | last_name_kana | Họ Katakana |
| first_name_kana | mst_saki_user | first_name_kana | Tên Katakana |
| last_name_en | mst_saki_user | last_name_en | Họ tiếng Anh |
| middle_name_en | mst_saki_user | middle_name_en | Tên đệm tiếng Anh |
| first_name_en | mst_saki_user | first_name_en | Tên tiếng Anh |
| office_id | mst_saki_user | office_id | ID sự nghiệp sở trực thuộc |
| department_id | mst_saki_user | department_id | ID phòng ban trực thuộc |
| position_ja | mst_saki_user | position_ja | Chức vụ tiếng Nhật |
| position_en | mst_saki_user | position_en | Chức vụ tiếng Anh |
| tel | mst_saki_user | tel | Số điện thoại |
| fax | mst_saki_user | fax | Số Fax |
| email | mst_saki_user | email | Email |
| mail_language | mst_saki_user | mail_language | Ngôn ngữ email |
| execution_role_id | mst_saki_user | execution_role_id | Mã vai trò thực thi |
| reference_scope | mst_saki_user | reference_scope | Phạm vi tham chiếu |
| edit_on_approval_flg | mst_saki_user | edit_on_approval_flg | Cờ sửa thông tin khi duyệt hợp đồng |
| view_group_company_flg | mst_saki_user | view_group_company_flg | Cờ tham chiếu nhóm công ty |
| edit_contract_dept_flg | mst_saki_user | edit_contract_dept_flg | Cờ sửa phòng ban trên hợp đồng |
| show_as_approver_flg | mst_saki_user | show_as_approver_flg | Cờ hiển thị làm người phê duyệt |
| final_approver_flg | mst_saki_user | final_approver_flg | Cờ duyệt cuối |
| approval_group_id | mst_saki_user | approval_group_id | Mã nhóm nhận yêu cầu phê duyệt |
| attendance_approver1_id | mst_saki_user | attendance_approver1_id | Người duyệt chấm công 1 |
| attendance_approver2_id | mst_saki_user | attendance_approver2_id | Người duyệt chấm công 2 |
| attendance_approver3_id | mst_saki_user | attendance_approver3_id | Người duyệt chấm công 3 |
| cost_center_code | mst_saki_user | cost_center_code | Mã cost center |
| cost_center_comment | mst_saki_user | cost_center_comment | Ghi chú cost center |
| status | mst_saki_user | status | Trạng thái (0: vô hiệu, 1: hoạt động) |
| user_scopes[].scope_type | mst_saki_user_scope | scope_type | Loại phạm vi tham chiếu tùy chọn |
| user_scopes[].target_id | mst_saki_user_scope | target_id | ID đối tượng (office_id hoặc department_id) |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| mst_saki_user | user_id | data.user_id | ID người dùng được cập nhật |
| mst_saki_user | updated_at | data.updated_at | Thời điểm cập nhật thành công |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate dữ liệu đầu vào và các kiểm tra nghiệp vụ cơ bản | Ngoài transaction |
| 2 | Khởi động Database Transaction | Bắt đầu transaction |
| 3 | Khóa dòng dữ liệu (SELECT FOR UPDATE) bản ghi user hiện tại | Trong transaction |
| 4 | Cập nhật dữ liệu chính vào bảng mst_saki_user | Trong transaction |
| 5 | Đồng bộ dữ liệu scope vào bảng mst_saki_user_scope (nếu reference_scope = 6) hoặc xóa hết scope cũ (nếu reference_scope != 6) | Trong transaction |
| 6 | Ghi nhật ký thay đổi (Audit Log) | Trong transaction |
| 7 | Commit Transaction | Kết thúc transaction |
| 8 | Trả kết quả response cho client | Ngoài transaction |

---

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Cập nhật bảng mst_saki_user thất bại | Rollback toàn bộ transaction |
| 2 | Cập nhật bảng mst_saki_user_scope gặp lỗi | Rollback toàn bộ transaction |
| 3 | Ghi Audit Log thất bại | Rollback toàn bộ transaction |
| 4 | Mất kết nối DB trong quá trình xử lý | Tự động rollback |

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Frontend gửi yêu cầu.
2. Middleware thực hiện xác thực và phân quyền.
3. Thiết lập kết nối động tới database schema của tenant.
4. Validate dữ liệu đầu vào.
5. Nếu validation thất bại, trả về HTTP 422.
6. Kiểm tra sự tồn tại của các khóa ngoại liên quan.
7. Kiểm tra trùng lặp email với các tài khoản khác trong cùng tenant.
8. Khởi động DB Transaction.
9. Thực hiện cập nhật thông tin user trong bảng mst_saki_user.
10. Xử lý đồng bộ danh sách scope tùy chọn:
    - Nếu reference_scope = 6:
      a. SELECT danh sách scope hiện tại của user trong DB.
      b. So sánh để thực hiện INSERT các scope mới và DELETE các scope không còn nằm trong request.
    - Nếu reference_scope != 6:
      a. DELETE toàn bộ các bản ghi scope cũ của user này trong mst_saki_user_scope.
11. Ghi nhận thông tin log thay đổi vào bảng nhật ký.
12. Commit DB Transaction.
13. Trả về response thành công.
```

---

# 14. Notification

Không áp dụng.

---

# 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | UPDATE |
| Bảng target | mst_saki_user, mst_saki_user_scope |
| ID record target | user_id |
| Dữ liệu ghi log | Giá trị cũ và giá trị mới của các trường thay đổi dữ liệu |
| Chính sách lưu trữ | Ghi nhận vào bảng log hoạt động chung của tenant schema |

---

# 16. Xử lý file

## 16.1 Upload

Không áp dụng.

---

## 16.2 Download

Không áp dụng.

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication | Bắt buộc phải xác thực thông qua JWT token hợp lệ trước khi thực thi xử lý. |
| 2 | Authorization | Chỉ tài khoản có vai trò SAKI Admin và có quyền user.update mới được phép cập nhật. |
| 3 | Tenant Isolation | Ràng buộc chặt chẽ việc kiểm tra cross-tenant để đảm bảo admin tenant này không thể cập nhật user thuộc tenant khác. |
| 4 | Input Validation | Validate nghiêm ngặt tất cả các trường dữ liệu ở Request Body, kiểm tra kiểu dữ liệu và ký tự đặc biệt để phòng tránh SQL Injection và XSS. |
| 5 | Data Protection | Các thông tin nhạy cảm (như mật khẩu/credential) không được truyền qua API này và không được ghi thô ra log file. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian phản hồi kỳ vọng | Dưới 1.5 giây |
| Index cần thiết | index theo user_id trong bảng mst_saki_user_scope để tối ưu hóa quá trình đồng bộ scope |
| Lock contention | Sử dụng SELECT FOR UPDATE để tránh race condition khi có nhiều admin cùng sửa đổi một tài khoản cùng lúc |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 User Management Flow |
| Database Dictionary | Carima-Staffing Data Dictionary / ERD (mst_saki_user, mst_saki_user_scope) |
| API Specification | SA-SET-001-API-01-Saki user list |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| - | Không áp dụng. | - | - | - | - |

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
| 13 | Notification rule đã được định nghĩa | Done |
| 14 | File handling rule đã được định nghĩa nếu cần | Done |
| 15 | Security considerations đã được review | Done |
| 16 | Performance considerations đã được review | Done |
