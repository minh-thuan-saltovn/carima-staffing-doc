# MO-SET-001-API-01-Get Company Master

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Get Company Master |
| Description | Lấy thông tin chi tiết master công ty MOTO (bao gồm thông tin cơ bản, pháp lý/thuế và người phụ trách). |
| Endpoint | `/api/v1/moto/settings/company-master` |
| Menu | Company Settings |
| Method | GET |
| Related Tables | `tenant_db.mst_moto_company`, `tenant_db.mst_moto_company_contact` |
| Screen list | 企業マスタ (MO-SET-001) - Company Master |
| System | MOTO Portal |
| 説明 | 会社マスタの情報を取得する (Lấy thông tin master công ty). |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Get Company Master |
| Diagram / Flow áp dụng | Tenant Master Management Flow |
| Portal áp dụng | MOTO Portal |
| Version | v1.0 |
| Ngày tạo | 2026/06/23 |
| Ngày cập nhật | 2026/06/23 |
| Người tạo | Thuận |
| Người review | Duy |
| Trạng thái | Draft |

---

## 2. Lịch sử chỉnh sửa

| Version | Ngày | Người chỉnh sửa | Nội dung thay đổi |
| --- | --- | --- | --- |
| v1.0 | 2026/06/23 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Get Company Master (GET) bám sát ERD thực tế và giao diện UI. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | MO-SET-001-API-01 |
| Tên API | Get Company Master |
| Business Flow liên quan | Tenant Master Management Flow |
| Screen ID liên quan | MO-SET-001 |
| Chức năng liên quan | Company Master Management |
| Mục đích API | Lấy thông tin chi tiết của công ty MOTO hiện tại (bao gồm các thông tin cơ bản, thông tin pháp lý/thuế và danh sách tối đa 2 người phụ trách) để hiển thị lên màn hình cấu hình. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/moto/settings/company-master |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này được gọi khi người dùng truy cập màn hình **企業マスタ (Company Master - MO-SET-001)** trên MOTO Portal. API thực hiện:
1. Truy vấn bản ghi duy nhất trong bảng `mst_moto_company` của tenant schema tương ứng.
2. Truy vấn danh sách người liên hệ phụ trách (tối đa 2 bản ghi) từ bảng `mst_moto_company_contact` sắp xếp theo thứ tự `seq_no` ASC.
3. Trả về response JSON cấu trúc Single Resource đầy đủ thông tin để hiển thị lên Form.

### Ngữ cảnh nghiệp vụ

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | Người dùng truy cập vào mục "Cài đặt doanh nghiệp / 企業マスタ" |
| Actor | MOTO Admin / MOTO Staff |
| Trước khi gọi API | Đăng nhập thành công và có vai trò quản trị/xem cài đặt MOTO Portal |
| Sau khi gọi API | Dữ liệu được điền tự động vào các ô input trên màn hình 企業マスタ |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/moto/settings/company-master
```

## 4.2 Request Header

| Header Name | Bắt buộc | Ví dụ | Mô tả |
| --- | --- | --- | --- |
| Authorization | Có | Bearer xxx | JWT access token |
| Content-Type | Có | application/json | Định dạng dữ liệu |
| Accept-Language | Không | ja / vi / en | Ngôn ngữ message trả về |

## 4.3 Path Parameters

Không áp dụng.

## 4.4 Query Parameters

Không áp dụng.

---

# 5. Request Body

Không áp dụng cho phương thức GET.

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
    "created_at": "2026-06-23T08:00:00+09:00",
    "updated_at": "2026-06-23T08:00:00+09:00",
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
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data | object | Thông tin chi tiết master công ty |
| data.company_id | string | Mã công ty |
| data.official_name_ja | string | Tên công ty chính thức (tiếng Nhật) |
| data.official_name_kana | string | Tên công ty chính thức (katakana) |
| data.display_name_ja | string | Tên công ty hiển thị (tiếng Nhật) |
| data.postal_code | string | Mã bưu chính (7 chữ số không gạch nối) |
| data.address_ja | string | Địa chỉ chính thức 1 (tiếng Nhật) |
| data.address2_ja | string \| null | Địa chỉ bổ sung 2 (tiếng Nhật) |
| data.official_name_en | string | Tên công ty chính thức (tiếng Anh) |
| data.display_name_en | string | Tên công ty hiển thị (tiếng Anh) |
| data.dispatch_permit_first | string \| null | Phần đầu số giấy phép phái cử (Ví dụ: "派") |
| data.dispatch_permit_last | string \| null | Phần sau số giấy phép phái cử (Ví dụ: "123456") |
| data.manage_permit_by_office_flg | number | Cờ quản lý số giấy phép theo sự nghiệp sở (0: quản lý ở công ty, 1: quản lý ở sự nghiệp sở) |
| data.qualified_invoice_status | number | Trạng thái đăng ký hóa đơn hợp lệ (0: không có số đăng ký, 1: có số đăng ký) |
| data.qualified_invoice_no | string \| null | Số đăng ký doanh nghiệp phát hành hóa đơn hợp lệ (13 chữ số) |
| data.created_at | datetime | Thời điểm tạo bản ghi công ty |
| data.updated_at | datetime | Thời điểm cập nhật bản ghi công ty |
| data.contacts | array | Danh sách thông tin tối đa 2 người phụ trách liên hệ |
| data.contacts[].seq_no | number | Thứ tự người phụ trách (1 hoặc 2) |
| data.contacts[].department | string \| null | Phòng ban / Đơn vị trực thuộc của người phụ trách |
| data.contacts[].name_ja | string | Họ tên người phụ trách (tiếng Nhật) |
| data.contacts[].name_kana | string \| null | Họ tên người phụ trách (katakana) |
| data.contacts[].tel | string | Số điện thoại liên hệ |
| data.contacts[].email | string | Địa chỉ email liên hệ |
| data.contacts[].remarks | string \| null | Ghi chú thêm |

---

## 6.3 Error Response

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

Không áp dụng (GET không nhận tham số).

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Dữ liệu chỉ được truy vấn từ bảng `mst_moto_company` và `mst_moto_company_contact` thuộc schema `tenant_moto_<company_id>` của MOTO tenant hiện tại dựa trên JWT context của user đăng nhập. |
| BR-002 | Role authorization | Chỉ cho phép người dùng thuộc MOTO Portal có vai trò phù hợp và sở hữu quyền `company_master.view` thực hiện API này. |
| BR-003 | Sort Contacts | Danh sách `contacts` đi kèm phải được sắp xếp tăng dần theo trường `seq_no` (`seq_no` ASC) để hiển thị đúng thứ tự người phụ trách 1 và người phụ trách 2. |

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

---

## 9.2 Logic kiểm tra quyền

```
1. Validate JWT token gửi kèm trong header Authorization.
2. Trích xuất thông tin người dùng và quyền hạn từ token.
3. Xác minh người dùng thuộc MOTO Portal và sở hữu quyền "company_master.view".
4. Nếu hợp lệ, cho phép xử lý tiếp. Ngược lại, trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

Không áp dụng.

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_moto.mst_moto_company | company_id | data.company_id | Mã công ty |
| tenant_moto.mst_moto_company | official_name_ja | data.official_name_ja | Tên công ty tiếng Nhật |
| tenant_moto.mst_moto_company | official_name_kana | data.official_name_kana | Tên công ty katakana |
| tenant_moto.mst_moto_company | display_name_ja | data.display_name_ja | Tên công ty hiển thị tiếng Nhật |
| tenant_moto.mst_moto_company | postal_code | data.postal_code | Mã bưu chính |
| tenant_moto.mst_moto_company | address_ja | data.address_ja | Địa chỉ chính thức 1 |
| tenant_moto.mst_moto_company | address2_ja | data.address2_ja | Địa chỉ bổ sung 2 |
| tenant_moto.mst_moto_company | official_name_en | data.official_name_en | Tên công ty tiếng Anh |
| tenant_moto.mst_moto_company | display_name_en | data.display_name_en | Tên công ty hiển thị tiếng Anh |
| tenant_moto.mst_moto_company | dispatch_permit_first | data.dispatch_permit_first | Đầu số giấy phép phái cử |
| tenant_moto.mst_moto_company | dispatch_permit_last | data.dispatch_permit_last | Đuôi số giấy phép phái cử |
| tenant_moto.mst_moto_company | manage_permit_by_office_flg | data.manage_permit_by_office_flg | Cờ quản lý số giấy phép |
| tenant_moto.mst_moto_company | qualified_invoice_status | data.qualified_invoice_status | Trạng thái hóa đơn |
| tenant_moto.mst_moto_company | qualified_invoice_no | data.qualified_invoice_no | Số đăng ký hóa đơn |
| tenant_moto.mst_moto_company | created_at | data.created_at | Thời điểm tạo |
| tenant_moto.mst_moto_company | updated_at | data.updated_at | Thời điểm cập nhật |
| tenant_moto.mst_moto_company_contact | seq_no | data.contacts[].seq_no | Thứ tự liên hệ |
| tenant_moto.mst_moto_company_contact | department | data.contacts[].department | Đơn vị trực thuộc |
| tenant_moto.mst_moto_company_contact | name_ja | data.contacts[].name_ja | Họ tên tiếng Nhật |
| tenant_moto.mst_moto_company_contact | name_kana | data.contacts[].name_kana | Họ tên katakana |
| tenant_moto.mst_moto_company_contact | tel | data.contacts[].tel | Điện thoại liên hệ |
| tenant_moto.mst_moto_company_contact | email | data.contacts[].email | Email liên hệ |
| tenant_moto.mst_moto_company_contact | remarks | data.contacts[].remarks | Ghi chú |

---

# 11. Database Transaction

Không áp dụng cho phương thức GET.

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu: GET /api/v1/moto/settings/company-master.
2. Middleware kiểm tra xác thực JWT và phân quyền (yêu cầu quyền "company_master.view").
3. Xác định tenant từ user context, thiết lập kết nối động tới schema `tenant_moto_<company_id>`.
4. Thực hiện các câu lệnh truy vấn dữ liệu:
   - SELECT bản ghi đầu tiên từ bảng `mst_moto_company` (WHERE company_id = company_id).
   - Nếu không tìm thấy: Trả về HTTP 404 Not Found.
   - SELECT toàn bộ các bản ghi trong bảng `mst_moto_company_contact` (WHERE company_id = company_id) sắp xếp theo `seq_no` ASC.
5. Đóng gói thông tin công ty và danh sách liên hệ vào Single Resource thông qua API Resource.
6. Trả về response JSON thành công kèm HTTP 200 OK.
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

Không áp dụng cho phương thức GET.

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
| 1 | Authentication & Authorization | Phải xác thực JWT token và kiểm tra quyền `company_master.view` của người dùng trước khi xử lý. |
| 2 | Tenant Isolation | Thiết lập kết nối động tới đúng tenant schema dựa trên thông tin công ty của người dùng đăng nhập để cách ly dữ liệu. Không cho phép truyền tham số `company_id` từ client để thay đổi tenant truy vấn dữ liệu. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.2 giây |
| Lượng truy cập dự kiến | Thấp |
| Pagination | Không áp dụng |
| Index cần thiết | Khóa chính trên bảng `mst_moto_company` và `mst_moto_company_contact`. |
| Cache cần thiết | Có thể cache ở mức ứng dụng vì thông tin công ty rất ít khi thay đổi. |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | Tenant Master Management Flow |
| Wireframe | MO-SET-001 企業マスタ |
| Screen Detail Design | MO-SET-001 |
| ERD | Carima-Staffing Data Dictionary / ERD |
| Permission Matrix | Portal Permission Matrix |
| Validation Rule | Section 7. Validation Rules |
| NFR | Performance Requirement |

---

# 20. Open Issues

Không có.

---

# 21. Checklist Review

| No. | Check Item | Status |
| --- | --- | --- |
| 1 | Endpoint đã được định nghĩa | Done |
| 2 | Request body đã được định nghĩa | Done (Không áp dụng) |
| 3 | Response body đã được định nghĩa | Done |
| 4 | Error response đã được định nghĩa | Done |
| 5 | Validation rule đã được định nghĩa | Done (Không áp dụng) |
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
