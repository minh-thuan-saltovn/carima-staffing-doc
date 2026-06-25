# MO-SET-001-API-02-Update Company Master

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Update Company Master |
| Description | Cập nhật master công ty.. |
| Endpoint | `/api/v1/moto/settings/company-master` |
| Menu | Company Settings |
| Method | PATch |
| Related Tables | `tenant_db.mst_moto_company` |
| Screen list | 企業マスタ (MO-SET-001) - Company Master |
| System | MOTO Portal |
| 説明 | 会社マスタを更新する。. |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Update Company Master |
| Diagram / Flow áp dụng | Tenant Master Management Flow |
| Portal áp dụng | MOTO Portal |
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
| v1.0 | 2026/06/23 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Update Company Master. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | MO-SET-001-API-02 |
| Tên API | Update Company Master |
| Business Flow liên quan | Tenant Master Management Flow |
| Screen ID liên quan | MO-SET-001 |
| Chức năng liên quan | Company Master Management |
| Mục đích API | Cập nhật thông tin chi tiết của công ty MOTO hiện tại. |
| Loại API | Frontend API |
| Method | PATCH |
| Endpoint | /api/v1/moto/settings/company-master |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Có |
| Xử lý file | Không |
| Trạng thái | VN Review |

---

## 3.2 Mô tả API

### Mục đích

API này được gọi khi người dùng nhấn nút Lưu. API thực hiện:
1. Validate định dạng và nghiệp vụ của toàn bộ các trường gửi lên trong Request Body.
2. Cập nhật thông tin công ty vào bảng mst_moto_company của tenant schema tương ứng.
3. Cập nhật hoặc tạo mới thông tin người phụ trách liên hệ trong bảng mst_moto_company_contact.
4. Trả về response JSON thành công chứa company_id của công ty đã cập nhật.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Người dùng nhấn nút Lưu ở góc phải màn hình |
| Actor | MOTO Admin |
| Trước khi gọi API | Đã đăng nhập vào hệ thống và có quyền cập nhật master công ty |
| Sau khi gọi API | Dữ liệu mới được cập nhật vào database và hiển thị lại lên màn hình |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
PATCH /api/v1/moto/settings/company-master
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Định dạng dữ liệu gửi lên |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

## 4.3 Path Parameters

Không áp dụng.

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

## 5.1 Request Body Schema

```json
{
  "official_name_ja": "株式会社XYZ人材サービス",
  "official_name_kana": "カブシキガイシャエックスワイゼットジンザイサービス",
  "display_name_ja": "XYZ人材サービス",
  "postal_code": "1500001",
  "address_ja": "東京都渋谷区渋谷1-1-1",
  "address2_ja": "XYZビル 5階",
  "official_name_en": "XYZ Staffing Services Inc.",
  "display_name_en": "XYZ Staffing",
  "dispatch_permit_first": "派",
  "dispatch_permit_last": "123456",
  "manage_permit_by_office_flg": 0,
  "qualified_invoice_status": 1,
  "qualified_invoice_no": "1234567890123",
  "contacts": [
    {
      "seq_no": 1,
      "department": "人事部",
      "name_ja": "山田 太郎",
      "name_kana": "ヤマダ タロウ",
      "tel": "03-1234-5678",
      "email": "yamada@xyz-staffing.co.jp",
      "remarks": null
    },
    {
      "seq_no": 2,
      "department": "営業部",
      "name_ja": "佐藤 花子",
      "name_kana": "サトウ ハナコ",
      "tel": "03-1234-5679",
      "email": "sato@xyz-staffing.co.jp",
      "remarks": null
    }
  ]
}
```

## 5.2 Định nghĩa field request

### 5.2.1 Thông tin công ty chính

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| official_name_ja | string | Có | 100 | Không được rỗng | Tên công ty chính thức (tiếng Nhật) |
| official_name_kana | string | Có | 200 | Định dạng Katakana | Tên công ty chính thức (katakana) |
| display_name_ja | string | Có | 24 | Không được rỗng | Tên hiển thị (tiếng Nhật) |
| postal_code | string | Có | 7 | Regex: ^[0-9]{7}$ | Mã bưu chính (7 chữ số half-width) |
| address_ja | string | Có | 50 | Không được rỗng | Địa chỉ chính thức 1 |
| address2_ja | string | Không | 50 | - | Địa chỉ bổ sung 2 |
| official_name_en | string | Có | 100 | Half-width alphanumeric | Tên công ty chính thức (tiếng Anh) |
| display_name_en | string | Có | 24 | Half-width alphanumeric | Tên hiển thị (tiếng Anh) |
| manage_permit_by_office_flg | number | Có | - | Giá trị: 0 hoặc 1 | Cờ quản lý số giấy phép theo sự nghiệp sở |
| dispatch_permit_first | string | Không | 2 | Bắt buộc khi manage_permit_by_office_flg = 0 | Phần đầu số giấy phép phái cử (ví dụ: 派) |
| dispatch_permit_last | string | Không | 6 | Regex: ^[0-9]{6}$. Bắt buộc khi manage_permit_by_office_flg = 0 | Phần sau số giấy phép phái cử |
| qualified_invoice_status | number | Có | - | Giá trị: 0 hoặc 1 | Tình trạng đăng ký hóa đơn hợp lệ |
| qualified_invoice_no | string | Không | 13 | Regex: ^[0-9]{13}$. Bắt buộc khi qualified_invoice_status = 1 | Số đăng ký phát hành hóa đơn hợp lệ |

### 5.2.2 Thông tin người liên hệ phụ trách (Mảng contacts)

| Field | Type | Bắt buộc | Max Length | Validation | Mô tả |
| --- | --- | --- | --- | --- | --- |
| contacts | array | Có | - | Tối đa 2 items | Danh sách người phụ trách |
| contacts[].seq_no | number | Có | - | Giá trị: 1 hoặc 2 | Thứ tự người liên hệ |
| contacts[].department | string | Không | 100 | - | Phòng ban phụ trách |
| contacts[].name_ja | string | Có | 48 | Không được rỗng | Họ tên người phụ trách (tiếng Nhật) |
| contacts[].name_kana | string | Không | 48 | Định dạng Katakana | Họ tên người phụ trách (katakana) |
| contacts[].tel | string | Có | 15 | Chỉ chữ số và gạch ngang half-width | Số điện thoại liên hệ |
| contacts[].email | string | Có | 128 | Định dạng Email hợp lệ | Địa chỉ email liên hệ |
| contacts[].remarks | string | Không | 500 | - | Ghi chú thêm |

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
    "company_id": "COM0000000000000000001",
    "updated_at": "2026-06-23T08:15:00+09:00"
  }
}
```

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data | object | Wrapper chứa kết quả |
| data.company_id | string | Mã công ty đã cập nhật thành công |
| data.updated_at | datetime | Thời điểm cập nhật dữ liệu thành công |

---
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


# 7. Validation Rules

[Message list](https://app.notion.com/p/sokucom/Message-list-374f02c407dd8037808eea01e93be8aa?source=copy_link) 

| No. | Field | Rule | Params | Message Key |
| --- | --- | --- | --- | --- |
| 1 | official_name_ja | required | - | required |
| 2 | official_name_ja | max | 100 | max |
| 3 | official_name_kana | required | - | required |
| 4 | official_name_kana | regex | /^[ァ-ヶー]+$/u | regex |
| 5 | official_name_kana | max | 200 | max |
| 6 | display_name_ja | required | - | required |
| 7 | display_name_ja | max | 24 | max |
| 8 | official_name_en | required | - | required |
| 9 | official_name_en | string | - | string |
| 10 | official_name_en | max | 100 | max |
| 11 | display_name_en | required | - | required |
| 12 | display_name_en | string | - | string |
| 13 | display_name_en | max | 24 | max |
| 14 | postal_code | required | - | required |
| 15 | postal_code | digits | 7 | digits |
| 16 | address_ja | required | - | required |
| 17 | address_ja | max | 50 | max |
| 18 | address2_ja | max | 50 | max |
| 19 | manage_permit_by_office_flg | required | - | required |
| 20 | manage_permit_by_office_flg | in | 0, 1 | in |
| 21 | dispatch_permit_first | required | - | required |
| 22 | dispatch_permit_first | max | 2 | max |
| 23 | dispatch_permit_last | required | - | required |
| 24 | dispatch_permit_last | digits | 6 | digits |
| 25 | qualified_invoice_status | required | - | required |
| 26 | qualified_invoice_status | in | 0, 1 | in |
| 27 | qualified_invoice_no | required | - | required |
| 28 | qualified_invoice_no | digits | 13 | digits |
| 29 | contacts | required | - | required |
| 30 | contacts | max | 2 | max |
| 31 | contacts[].seq_no | required | - | required |
| 32 | contacts[].seq_no | in | 1, 2 | in |
| 33 | contacts[].department | max | 100 | max |
| 34 | contacts[].name_ja | required | - | required |
| 35 | contacts[].name_ja | max | 48 | max |
| 36 | contacts[].name_kana | regex | /^[ァ-ヶー]+$/u | regex |
| 37 | contacts[].name_kana | max | 48 | max |
| 38 | contacts[].tel | required | - | required |
| 39 | contacts[].tel | max | 15 | max |
| 40 | contacts[].tel | string | - | string |
| 41 | contacts[].email | required | - | required |
| 42 | contacts[].email | string | - | string |
| 43 | contacts[].email | max | 128 | max |
| 44 | contacts[].remarks | max | 500 | max |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ được phép cập nhật dữ liệu của tenant hiện tại dựa trên JWT token context. Không nhận company_id từ Client để xác định tenant. |
| BR-002 | Role authorization | Chỉ cho phép người dùng thuộc MOTO Portal có vai trò phù hợp và sở hữu quyền company_master.update thực hiện API này. |
| BR-003 | Permit field logic | Nếu manage_permit_by_office_flg = 1, hệ thống sẽ bỏ qua và clear (set null) giá trị của dispatch_permit_first và dispatch_permit_last trong database. |
| BR-004 | Invoice field logic | Nếu qualified_invoice_status = 0, hệ thống sẽ bỏ qua và clear (set null) giá trị của qualified_invoice_no trong database. |
| BR-005 | Contacts upsert | Đối với mảng contacts, hệ thống sẽ đồng bộ hóa dựa trên seq_no:<br>1. Nếu bản ghi có seq_no đã tồn tại trong database: thực hiện UPDATE.<br>2. Nếu bản ghi chưa có: thực hiện INSERT.<br>3. Nếu trước đó có 2 người phụ trách nhưng request gửi lên chỉ có 1 người: thực hiện DELETE bản ghi người phụ trách còn lại trong database. |
| BR-006 | Audit log | Ghi log chi tiết các trường thay đổi (Before / After) khi cập nhật thành công. |

---

# 9. Permission / Authorization

## 9.1 Quyền cần thiết

| Role | View | Create | Update | Delete | Approve | Export |
| --- | --- | --- | --- | --- | --- | --- |
| Platform Admin | Có | Không | Có | Không | Không | Không |
| Tenant Admin | Có | Không | Có | Không | Không | Không |
| MOTO Admin | Có | Không | Có | Không | Không | Không |
| MOTO User | Có | Không | Không | Không | Không | Không |
| SAKI Admin | Không | Không | Không | Không | Không | Không |
| Staff | Không | Không | Không | Không | Không | Không |

## 9.2 Logic kiểm tra quyền

```
1. Validate JWT token gửi kèm trong header Authorization.
2. Trích xuất thông tin người dùng và quyền hạn từ token.
3. Xác minh người dùng thuộc MOTO Portal và sở hữu quyền company_master.update.
4. Nếu hợp lệ, cho phép xử lý tiếp. Ngược lại, trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Mô tả |
| --- | --- | --- | --- |
| official_name_ja | tenant_db.mst_moto_company | official_name_ja | Tên công ty tiếng Nhật |
| official_name_kana | tenant_db.mst_moto_company | official_name_kana | Tên công ty Katakana |
| display_name_ja | tenant_db.mst_moto_company | display_name_ja | Tên hiển thị tiếng Nhật |
| postal_code | tenant_db.mst_moto_company | postal_code | Mã bưu chính |
| address_ja | tenant_db.mst_moto_company | address_ja | Địa chỉ chính thức 1 |
| address2_ja | tenant_db.mst_moto_company | address2_ja | Địa chỉ bổ sung 2 |
| official_name_en | tenant_db.mst_moto_company | official_name_en | Tên công ty tiếng Anh |
| display_name_en | tenant_db.mst_moto_company | display_name_en | Tên hiển thị tiếng Anh |
| manage_permit_by_office_flg | tenant_db.mst_moto_company | manage_permit_by_office_flg | Cờ quản lý số giấy phép |
| dispatch_permit_first | tenant_db.mst_moto_company | dispatch_permit_first | Phần đầu số giấy phép phái cử |
| dispatch_permit_last | tenant_db.mst_moto_company | dispatch_permit_last | Phần sau số giấy phép phái cử |
| qualified_invoice_status | tenant_db.mst_moto_company | qualified_invoice_status | Tình trạng hóa đơn |
| qualified_invoice_no | tenant_db.mst_moto_company | qualified_invoice_no | Số đăng ký hóa đơn |
| contacts[].seq_no | tenant_db.mst_moto_company_contact | seq_no | Thứ tự liên hệ |
| contacts[].department | tenant_db.mst_moto_company_contact | department | Phòng ban phụ trách |
| contacts[].name_ja | tenant_db.mst_moto_company_contact | name_ja | Họ tên tiếng Nhật |
| contacts[].name_kana | tenant_db.mst_moto_company_contact | name_kana | Họ tên Katakana |
| contacts[].tel | tenant_db.mst_moto_company_contact | tel | Số điện thoại liên hệ |
| contacts[].email | tenant_db.mst_moto_company_contact | email | Email liên hệ |
| contacts[].remarks | tenant_db.mst_moto_company_contact | remarks | Ghi chú thêm |

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_db.mst_moto_company | company_id | data.company_id | Mã công ty đã cập nhật |
| tenant_db.mst_moto_company | updated_at | data.updated_at | Thời điểm cập nhật dữ liệu |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

Toàn bộ thao tác cập nhật bảng mst_moto_company và mst_moto_company_contact phải được bọc trong một Database Transaction để đảm bảo tính toàn vẹn dữ liệu.

| Step | Process | Transaction |
| --- | --- | --- |
| 1 | Validate Authentication & Authorization | Ngoài transaction |
| 2 | Validate Request Body & Business logic | Ngoài transaction |
| 3 | Start DB Transaction | Bắt đầu |
| 4 | Lock & Select company record | Trong transaction |
| 5 | Update tenant_db.mst_moto_company | Trong transaction |
| 6 | Select current contacts of company | Trong transaction |
| 7 | Process Upsert / Delete contacts | Trong transaction |
| 8 | Write Audit Log (vào bảng t_moto_audit_log nếu có) | Trong transaction |
| 9 | Commit Transaction | Kết thúc |

## 11.2 Điều kiện rollback

| No. | Điều kiện | Hành động |
| --- | --- | --- |
| 1 | Update bảng mst_moto_company thất bại | Rollback toàn bộ transaction |
| 2 | Upsert/Delete bảng mst_moto_company_contact thất bại | Rollback toàn bộ transaction |
| 3 | Gặp lỗi hệ thống đột ngột (Database connection lost, Exception,...) | Rollback toàn bộ transaction |

---

# 12. Status Transition

Không áp dụng (Dữ liệu master cấu hình công ty không quản lý qua Status Workflow).

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu.
2. Middleware kiểm tra xác thực JWT và phân quyền.
3. Xác định tenant từ user context, thiết lập kết nối động tới schema tenant_moto_<company_id>.
4. Validate dữ liệu đầu vào.
5. Nếu có lỗi: Trả về HTTP 422 Unprocessable Entity kèm thông tin chi tiết lỗi.
6. Backend start DB Transaction.
7. Thực hiện SELECT khóa dòng và UPDATE bảng mst_moto_company.
8. So sánh danh sách contacts gửi lên và danh sách hiện có dưới DB:
   - SELECT contacts WHERE company_id = company_id.
   - Duyệt qua từng contact trong Request Body để INSERT hoặc UPDATE.
   - DELETE các contact trong DB có seq_no không nằm trong Request Body.
9. Ghi log.
10. Commit DB Transaction.
11. Trả về response JSON thành công chứa company_id và updated_at.
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
| Bảng target | mst_moto_company, mst_moto_company_contact |
| ID record target | company_id |
| Dữ liệu ghi log | Giá trị cũ và giá trị mới của các cột thay đổi dữ liệu |
| Chính sách lưu trữ | Ghi vào bảng log chung của tenant schema |

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
| 1 | Authentication & Authorization | Bắt buộc kiểm tra tính hợp lệ của JWT token và quyền company_master.update. |
| 2 | Tenant Isolation | Thiết lập kết nối động tới đúng tenant schema dựa trên thông tin công ty từ user context. Cấm hoàn toàn việc truyền company_id từ Client để xác định schema. |
| 3 | Input Validation & SQL Injection | Thực hiện validate chặt chẽ dữ liệu đầu vào (Regex mã bưu chính, số điện thoại, định dạng email, chữ Katakana toàn giác). Sử dụng Eloquent ORM / Parameterized Query để tránh SQL Injection. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.5 giây |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính trên bảng mst_moto_company (company_id) và mst_moto_company_contact (company_id, seq_no). |
| Cache cần thiết | Thực hiện xóa cache cấu hình thông tin công ty sau khi cập nhật thành công. |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | Tenant Master Management Flow (BF-032) |
| Wireframe | MO-SET-001 企業マスタ |
| Screen Detail Design | MO-SET-001 Screen Specification |
| ERD | Carima-Staffing Data Dictionary / ERD |
| Permission Matrix | Portal Permission Matrix |

---

# 20. Open Issues

Không có.

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
