# SA-AUTH-003-API-01-Change Password

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Change Password |
| Description | Đổi mật khẩu cho tài khoản đang đăng nhập. |
| Endpoint | /api/v1/saki/change-password |
| Menu | Common |
| Method | POST |
| Related Tables | tenant_db.mst_saki_user, tenant_db.saki_user_credential |
| Screen list | パスワード変更 SA-AUTH-003 Change Password |
| System | SAKI Portal |
| 説明 | ログイン中ユーザーのパスワードを変更する。 |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design |
| Business Flow áp dụng | BF-001 Login Flow / Account Security |
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
| v1.0 | 2026/06/23 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Saki Change Password. |

---

## 3. Tổng quan API

### 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-AUTH-003-API-01-Change Password |
| Tên API | Change Password |
| Tên tiếng Nhật | パスワード変更 |
| Business Flow liên quan | BF-001 Login Flow / Account Security |
| Screen ID liên quan | SA-AUTH-003 |
| Chức năng liên quan | Đổi mật khẩu người dùng đang đăng nhập |
| Mục đích API | Cho phép người dùng SAKI Portal tự thay đổi mật khẩu hiện tại của mình để bảo mật tài khoản. |
| Loại API | Frontend API |
| Method | POST |
| Endpoint | /api/v1/saki/change-password |
| Authentication | Có |
| Authorization | Có (Bất kỳ user nào đang đăng nhập hợp lệ) |
| Phạm vi Tenant | Tenant được xác định từ Access Token / request context |
| Có Transaction không | Có |
| Xử lý file | Không |
| Response format | JSON |
| Trạng thái | VN Review |

### 3.2 Mô tả API

API này xử lý việc đổi mật khẩu cho người dùng SAKI Portal đang đăng nhập.

Hệ thống xác thực Access Token để lấy ID người dùng và tenant tương ứng. Người dùng gửi lên mật khẩu hiện tại cùng mật khẩu mới và xác nhận mật khẩu mới.

Backend thực hiện kiểm tra tính hợp lệ của mật khẩu hiện tại, kiểm tra định dạng và độ phức tạp của mật khẩu mới. Nếu hợp lệ, mật khẩu mới sẽ được băm bằng thuật toán an toàn và cập nhật vào bảng xác thực tài khoản. Các cờ yêu cầu đổi mật khẩu ở lần tiếp theo (nếu có) cũng được reset về 0.

### 3.3 Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User nhấn nút 「パスワード変更」 trên màn hình |
| Actor | SAKI User / Admin đang đăng nhập |
| Trước khi gọi API | User đang ở màn hình Đổi mật khẩu và điền đầy đủ các trường dữ liệu |
| Sau khi gọi API thành công | Hiển thị thông báo thành công, giữ phiên đăng nhập hoặc yêu cầu đăng nhập lại (tùy thuộc frontend) |
| Sau khi gọi API thất bại | Hiển thị thông báo lỗi chi tiết trên màn hình |
| Thay đổi trạng thái liên quan | Cập nhật mật khẩu băm mới và reset cờ yêu cầu đổi mật khẩu lần đầu |

---

## 4. Định nghĩa Endpoint

### 4.1 Endpoint

```
POST /api/v1/saki/change-password
```

### 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Content-Type | Có | application/json | Loại dữ liệu request |
| Accept | Có | application/json | Dữ liệu response mong muốn |
| Authorization | Có | Bearer eyJhbGciOi... | Access Token của user đang đăng nhập |
| Accept-Language | Không | ja | Ngôn ngữ câu thông báo lỗi |

### 4.3 Path Parameters

Không áp dụng.

### 4.4 Query Parameters

Không áp dụng.

---

## 5. Request Body

### 5.1 Request Body JSON

```json
{
  "current_password": "oldPassword123!",
  "new_password": "NewStrongPassword456!",
  "new_password_confirmation": "NewStrongPassword456!"
}
```

### 5.2 Định nghĩa field request

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| current_password | string | Có | 255 | Không rỗng | Mật khẩu hiện tại của người dùng |
| new_password | string | Có | 255 | Tối thiểu 8 ký tự, chữ hoa, chữ thường, số, ký tự đặc biệt | Mật khẩu mới muốn thay đổi |
| new_password_confirmation | string | Có | 255 | Khớp với new_password | Xác nhận lại mật khẩu mới |

### 5.3 Ghi chú request

* Các trường mật khẩu thô gửi lên bắt buộc phải đi qua kết nối HTTPS bảo mật để tránh rò rỉ thông tin trên đường truyền.

---

## 6. Response Definition

### 6.1 Success Response

HTTP Status: 200 OK

```json
{
  "success": true,
  "message": "パスワードを変更しました"
}
```

### 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| success | boolean | Trạng thái xử lý thành công (true) |
| message | string | Thông báo kết quả bằng tiếng Nhật |
### 6.3 Error Response - Validation Error

### 6.4 Error Response - Invalid Credentials

HTTP Status: 401 Unauthorized

### 6.5 Error Response - Account Locked

HTTP Status: 403 Forbidden

### 6.6 Error Response - Account Inactive

### 6.7 Error Response - Tenant Not Found

HTTP Status: 404 Not Found

### 6.8 Error Response - System Error

HTTP Status: 500 Internal Server Error

### 6.9 Error Response - Bad Request

HTTP Status: 400 Bad Request

### 6.10 Error Response - Method Not Allowed

HTTP Status: 405 Method Not Allowed

### 6.11 Error Response - Too Many Requests

HTTP Status: 429 Too Many Requests

### 6.12 Error Response - Service Unavailable

HTTP Status: 503 Service Unavailable

### 6.13 Error Response - Gateway Timeout

HTTP Status: 504 Gateway Timeout

Untitled


## 7. Validation Rules

[Message list](https://app.notion.com/p/sokucom/Message-list-374f02c407dd8037808eea01e93be8aa?source=copy_link) 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | current_password | required | - | required |
| 2 | current_password | max | 255 | max |
| 3 | new_password | required | - | required |
| 4 | new_password | min | 8 | min |
| 5 | new_password | max | 255 | max |
| 6 | new_password | regex | /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]+$/ | regex |
| 7 | new_password | string | - | string |
| 8 | new_password_confirmation | required | - | required |
| 9 | new_password_confirmation | string | - | string |

### 7.1 Validation xử lý trước Business Rule

| Step | Nội dung |
| --- | --- |
| 1 | Kiểm tra Authorization Header (Access Token hợp lệ) |
| 2 | Kiểm tra Content-Type là application/json |
| 3 | Parse JSON request body |
| 4 | Validate các trường bắt buộc, độ dài, tính khớp và độ phức tạp mật khẩu |
| 5 | Chuyển sang xử lý Business Rule |

---

## 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Token Verification & Tenant Resolution | Lấy thông tin user_id và tenant_id từ Access Token. Toàn bộ các bước sau phải được thực hiện trong schema của tenant này. |
| BR-002 | Verify Current Password | Hệ thống truy vấn thông tin password_hash từ bảng saki_user_credential của người dùng hiện tại và thực hiện so khớp mật khẩu hiện tại qua Bcrypt. |
| BR-003 | Incorrect Password Handler | Nếu so khớp mật khẩu hiện tại thất bại, trả về lỗi 422 Unprocessable Entity với mã lỗi INCORRECT ở field current_password. |
| BR-004 | New Password Validation | Mật khẩu mới bắt buộc phải khác mật khẩu hiện tại (SAME_AS_OLD). |
| BR-005 | Password Hashing | Mật khẩu mới phải được băm bằng thuật toán Bcrypt an toàn trước khi cập nhật vào cơ sở dữ liệu. |
| BR-006 | Reset Password Change Requirement Flag | Nếu tài khoản hiện tại có cờ require_password_change_flg = 1 trong bảng saki_user_credential, cập nhật cờ này về 0. |
| BR-007 | Security Log Activity | Ghi nhận lịch sử đổi mật khẩu thành công. Tuyệt đối không ghi mật khẩu thô hoặc chuỗi hash mật khẩu mới vào log hệ thống. |

---

## 9. Permission / Authorization

### 9.1 Authentication

| Hạng mục | Nội dung |
| --- | --- |
| API yêu cầu Access Token không | Có |
| Token type | Bearer |

### 9.2 Authorization

| Role | Cho phép đổi mật khẩu | Ghi chú |
| --- | --- | --- |
| SAKI Admin | Có | Chỉ được đổi mật khẩu cho chính tài khoản của mình |
| SAKI User / Staff | Có | Chỉ được đổi mật khẩu cho chính tài khoản của mình |
| MOTO Portal Users | Không | Không thuộc scope phân quyền của SAKI Portal |
| Platform Admin | Không | Cổng quản trị Platform Admin riêng biệt |

### 9.3 Logic kiểm tra quyền

| Step | Nội dung |
| --- | --- |
| 1 | Xác thực token hiện tại, lấy tenant_id và user_id của actor đăng nhập. |
| 2 | Đối chiếu user_id trong token với user_id cần đổi mật khẩu (chính mình), nếu đúng cho phép đi tiếp. |

---

## 10. Data Mapping

### 10.1 Request To DB Mapping

| Request Field | DB | Table | Column | Process | Mô tả |
| --- | --- | --- | --- | --- | --- |
| Token.user_id | tenant_db | saki_user_credential | user_id | WHERE | Xác định dòng dữ liệu của chính user |
| new_password (hashed) | tenant_db | saki_user_credential | password_hash | UPDATE | Cập nhật mật khẩu băm mới |
| - | tenant_db | saki_user_credential | require_change_flg | UPDATE | Cập nhật giá trị thành 0 |
| - | tenant_db | saki_user_credential | updated_at | UPDATE | Thời điểm cập nhật |
| Token.user_id | tenant_db | saki_user_credential | updated_by | UPDATE | Người thực hiện cập nhật |

### 10.2 DB To Response Mapping

Không trả dữ liệu DB nào ra Response thành công ngoại trừ trạng thái success và thông báo message.

### 10.3 Index cần thiết

| DB | Table | Index Name | Column | Mục đích |
| --- | --- | --- | --- | --- |
| tenant_db | saki_user_credential | idx_saki_cred_user | user_id | Tra cứu nhanh thông tin credential của user |

---

## 11. Database Transaction

### 11.1 Phạm vi transaction khi đổi mật khẩu thành công

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Xác định Tenant, Validate request body | Ngoài transaction |
| 2 | Kiểm tra token và so khớp mật khẩu hiện tại | Ngoài transaction |
| 3 | Bắt đầu Database Transaction | Trong transaction |
| 4 | Cập nhật password_hash và require_change_flg trong saki_user_credential | Trong transaction |
| 5 | Ghi nhận Audit Log đổi mật khẩu thành công | Trong transaction |
| 6 | Commit Transaction | Trong transaction |
| 7 | Trả về Response thành công | Ngoài transaction |

### 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Cập nhật bảng saki_user_credential thất bại | Rollback toàn bộ transaction |
| 2 | Ghi nhận Audit Log thất bại | Rollback toàn bộ transaction |

---

## 12. Status Transition

Không áp dụng cho trạng thái tài khoản. Trạng thái tài khoản vẫn giữ nguyên là hoạt động.

---

## 13. Sequence / Processing Flow

### 13.1 Change Password Success Flow

1. Frontend gửi request đổi mật khẩu chứa current_password, new_password, new_password_confirmation.
2. Backend xác thực Access Token, giải mã lấy tenant_id và user_id của người dùng.
3. Validate định dạng request body và kiểm tra độ phức tạp của mật khẩu mới.
4. Truy vấn bảng saki_user_credential để so khớp mật khẩu hiện tại qua Bcrypt.
5. So khớp thành công và mật khẩu mới khác mật khẩu cũ.
6. Khởi tạo Database Transaction:
   - Cập nhật password_hash mới đã băm và cập nhật require_change_flg = 0.
   - Ghi nhận Audit Log hoạt động đổi mật khẩu.
7. Commit Transaction thành công.
8. Trả về response thành công HTTP 200 OK.

### 13.2 Change Password Failed Flow

1. Frontend gửi request đổi mật khẩu.
2. Backend validate thất bại (mật khẩu mới quá ngắn, không đủ độ phức tạp, mật khẩu xác nhận không khớp, hoặc mật khẩu mới trùng mật khẩu cũ).
3. Hoặc so khớp mật khẩu hiện tại thất bại (mật khẩu hiện tại nhập sai).
4. Hệ thống trả về lỗi 422 Unprocessable Entity kèm thông điệp lỗi cụ thể.

---

## 14. Notification

Không yêu cầu gửi Email/SMS thông báo khi tự đổi mật khẩu thành công trong màn hình này (hoặc tùy thuộc thiết kế có gửi email bảo mật thông báo mật khẩu thay đổi). Hiện tại là Không.

---

## 15. Audit Log

| Hạng mục | Nội dung |
| --- | --- |
| Có cần Audit Log không | Có |
| Loại thao tác | UPDATE |
| Bảng target | saki_user_credential |
| ID record target | user_id |
| Dữ liệu ghi log | user_id, action: CHANGE_PASSWORD, ip_address, user_agent, updated_at |
| Chính sách lưu trữ | Ghi vào bảng log hoạt động bảo mật chung của tenant schema |

---

## 16. Xử lý file

Không áp dụng.

---

## 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Mật khẩu thô | Tuyệt đối không ghi mật khẩu hiện tại hay mật khẩu mới ở dạng thô vào file log hệ thống hoặc audit log. |
| 2 | Băm mật khẩu | Sử dụng thuật toán băm mạnh Bcrypt với salt đầy đủ trước khi lưu trữ vào DB. |
| 3 | Token Validation | Phải kiểm tra tính hợp lệ của token ở mỗi request. |
| 4 | SQL Injection Prevention | Sử dụng cơ chế binding parameter của Eloquent ORM. |

---

## 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian phản hồi | Dưới 1 giây (do xử lý băm mật khẩu bằng Bcrypt mất khoảng 100-300ms tùy cấu hình rounds) |
| Index cần thiết | Index theo user_id trong bảng credential để truy xuất nhanh bản ghi cần cập nhật. |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 Login Flow / Account Security |
| Wireframe | SA-AUTH-003 Change Password |
| Screen Detail Design | SA-AUTH-003 |
| ERD | Carima-Staffing Data Dictionary / ERD (mst_saki_user) |
| Permission Matrix | Portal Permission Matrix |
| Validation Rule | Section 7. Validation Rules |
| NFR | Performance Requirement |
| API Design | SA-AUTH-001-API-01-Login |

---

# 20. Open Issues

| No. | Issue | Ảnh hưởng | Owner | Due Date | Status |
| --- | --- | --- | --- | --- | --- |
| 1 | Chưa xác định bảng credential chính thức dành cho saki_user trong Data Dictionary (hiện tại giả định tên là saki_user_credential). | Low | PM / SA | TBD | Open |

---

## 21. Checklist Review

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
