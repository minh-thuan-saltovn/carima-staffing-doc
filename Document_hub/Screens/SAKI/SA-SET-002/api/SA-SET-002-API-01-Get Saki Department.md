# SA-SET-002-API-01-Get Saki Department

| Hạng mục | Nội dung |
| --- | --- |
| API Name | Get Saki Department |
| Description | Lấy danh sách phòng ban của SAKI. |
| Endpoint | /api/v1/saki/settings/departments |
| Menu | Settings |
| Method | GET |
| Related Tables | tenant_db.mst_saki_department, tenant_db.mst_saki_user, tenant_db.mst_saki_office |
| Screen list | 部署マスタ (SA-SET-002) - Department Master |
| System | SAKI Portal |
| 説明 | SAKI của danh sách phòng ban (bộ phận), hỗ trợ tìm kiếm, lọc theo sự nghiệp sở, lọc theo trạng thái và phân trang. |

---

## 1. Thông tin tài liệu

| Hạng mục | Nội dung |
| --- | --- |
| Dự án | CARIMA Staffing |
| Tên tài liệu | API Detail Design - Get Saki Department |
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
| v1.0 | 2026/06/22 | Thuận | Khởi tạo tài liệu đặc tả chi tiết API Get Saki Department (GET) bám sát 100% ERD thực tế, tuân thủ Template và API Convention. |

---

# 3. Tổng quan API

## 3.1 Thông tin API

| Hạng mục | Nội dung |
| --- | --- |
| API ID | SA-SET-002-API-01-Get Saki Department |
| Tên API | Get Saki Department |
| Business Flow liên quan | BF-001 User Management Flow |
| Screen ID liên quan | SA-SET-002 |
| Chức năng liên quan | Department Management (SAKI Department Master) |
| Mục đích API | Lấy danh sách phòng ban, hỗ trợ tìm kiếm theo tên/ID/tên người quản lý, lọc theo sự nghiệp sở trực thuộc, trạng thái và phân trang trong cùng Tenant. |
| Loại API | Frontend API |
| Method | GET |
| Endpoint | /api/v1/saki/settings/departments |
| Authentication | Có |
| Authorization | Có |
| Phạm vi Tenant | Tenant Schema / Không cho phép Cross Tenant |
| Có Transaction không | Không |
| Xử lý file | Không |
| Trạng thái | Draft |

---

## 3.2 Mô tả API

### Mục đích

API này dùng để lấy danh sách các phòng ban (部署) cấu hình trong SAKI Portal của tenant hiện tại phục vụ hiển thị lên bảng dữ liệu tại màn hình **部署マスタ管理 (Department Master - SA-SET-002)**. API hỗ trợ tìm kiếm tương đối theo tên/mã phòng ban hoặc tên trưởng phòng, lọc theo sự nghiệp sở trực thuộc (事業所), lọc theo trạng thái hoạt động và phân trang.

### Ngữ cảnh nghiệp vụ

API được gọi khi người dùng truy cập màn hình Danh sách phòng ban (`SA-SET-002`) hoặc thay đổi các bộ lọc, thực hiện tìm kiếm, chuyển trang trên màn hình này.

| Hạng mục | Nội dung |
| --- | --- |
| Hành động kích hoạt | User truy cập màn hình hoặc thay đổi bộ lọc, chuyển trang |
| Actor | SAKI Admin / SAKI User (những người có quyền quản trị cài đặt) |
| Trước khi gọi API | Đã đăng nhập vào SAKI Portal và có quyền department.view |
| Sau khi gọi API | Hiển thị danh sách phòng ban cùng thông tin phân trang lên bảng dữ liệu |
| Thay đổi trạng thái liên quan | Không |

---

# 4. Định nghĩa Endpoint

## 4.1 Endpoint

```
GET /api/v1/saki/settings/departments
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
| limit | number | Không | 10 | Số lượng bản ghi mỗi trang | 10 |
| keyword | string | Không | null | Tìm kiếm tương đối theo tên phòng ban (official_name_ja, display_name_ja), mã phòng ban (department_id), hoặc tên trưởng phòng (last_name_ja, first_name_ja) | 人事 |
| office_id | string | Không | null | Lọc theo sự nghiệp sở trực thuộc (lấy gián tiếp qua sự nghiệp sở của Trưởng phòng ban) | OFF-001 |
| status | number | Không | null | Lọc theo trạng thái hoạt động (0: Vô hiệu, 1: Hiệu lực/Hoạt động) | 1 |
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
      "department_id": "DEPT-001",
      "official_name_ja": "人事部",
      "display_name_ja": "人事部",
      "official_name_en": "Human Resources",
      "display_name_en": "Human Resources",
      "office_id": "OFF-001",
      "office_name": "SAKI 東京本社",
      "manager_user_id": "yamada.taro",
      "manager_name": "山田 太郎",
      "member_count": 12,
      "status": 1,
      "created_at": "2026-04-01T08:00:00+09:00",
      "updated_at": "2026-06-22T10:00:00+09:00"
    },
    {
      "department_id": "DEPT-005",
      "official_name_ja": "営業部",
      "display_name_ja": "営業部",
      "official_name_en": "Sales",
      "display_name_en": "Sales",
      "office_id": "OFF-002",
      "office_name": "SAKI 大阪支社",
      "manager_user_id": "takahashi.misaki",
      "manager_name": "高橋 美咲",
      "member_count": 15,
      "status": 1,
      "created_at": "2026-04-02T09:00:00+09:00",
      "updated_at": "2026-06-22T10:00:00+09:00"
    }
  ],
  "links": {
    "first": "http://example.com/api/v1/saki/settings/departments?page=1",
    "last": "http://example.com/api/v1/saki/settings/departments?page=1",
    "prev": null,
    "next": null
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 1,
    "path": "http://example.com/api/v1/saki/settings/departments",
    "per_page": 10,
    "to": 2,
    "total": 2
  }
}
```

---

## 6.2 Định nghĩa field response thành công

| Field | Type | Mô tả |
| --- | --- | --- |
| data[] | array | Danh sách các phòng ban |
| data[].department_id | string | Mã phòng ban |
| data[].official_name_ja | string | Tên phòng ban chính thức tiếng Nhật |
| data[].display_name_ja | string | Tên phòng ban hiển thị tiếng Nhật |
| data[].official_name_en | string \| null | Tên phòng ban chính thức tiếng Anh |
| data[].display_name_en | string \| null | Tên phòng ban hiển thị tiếng Anh |
| data[].office_id | string \| null | Mã sự nghiệp sở trực thuộc (lấy gián tiếp qua sự nghiệp sở của Trưởng phòng ban) |
| data[].office_name | string \| null | Tên sự nghiệp sở trực thuộc (lấy gián tiếp qua sự nghiệp sở của Trưởng phòng ban) |
| data[].manager_user_id | string \| null | ID của Trưởng phòng ban (lấy từ default_dispatch_supervisor_id) |
| data[].manager_name | string \| null | Tên Trưởng phòng ban (ghép từ last_name_ja và first_name_ja) |
| data[].member_count | number | Số lượng nhân sự trực thuộc phòng ban |
| data[].status | number | Trạng thái (0: Vô hiệu, 1: Hiệu lực/Hoạt động) |
| data[].created_at | datetime | Thời điểm tạo bản ghi phòng ban |
| data[].updated_at | datetime | Thời điểm cập nhật bản ghi phòng ban |
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
        "field": "limit",
        "message": "Limit must be one of 10, 20, 50."
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
| 1 | page | Không bắt buộc, số nguyên >= 1 | CMS-VAL-40 | ページは整数で指定してください。 |
| 2 | limit | Không bắt buộc, số nguyên thuộc danh sách: 10, 20, 50 | CMS-VAL-40 | 表示件数は整数で指定してください。 |
| 3 | keyword | Không bắt buộc, string, tối đa 100 ký tự | CMS-VAL-6 | keywordは100文字以内で入力してください。 |
| 4 | office_id | Không bắt buộc, string, tối đa 20 ký tự | CMS-VAL-6 | 事業所IDは20文字以内で入力してください。 |
| 5 | status | Không bắt buộc, số nguyên thuộc: 0, 1 | CMS-VAL-40 | ステータスは整数で指定してください。 |
| 6 | sort_column | Không bắt buộc, phải thuộc: department_id, display_name_ja, status, created_at | CMS-VAL-41 | 選択されたソートカラムは正しくありません。 |
| 7 | sort_direction | Không bắt buộc, phải là asc hoặc desc | CMS-VAL-41 | 選択されたソート方向は正しくありません。 |

---

# 8. Business Rules

| No. | Rule | Mô tả |
| --- | --- | --- |
| BR-001 | Tenant isolation | Chỉ người dùng thuộc tenant hiện tại mới có thể truy cập cấu hình của tenant đó. Kết nối động tới schema tenant_saki_<company_id> dựa trên JWT context. |
| BR-002 | Role authorization | Chỉ cho phép người dùng thuộc SAKI Portal có vai trò phù hợp và sở hữu quyền department.view truy cập API này. |
| BR-003 | Default pagination & sorting | Nếu page không được gửi, mặc định là 1. Nếu limit không được gửi, mặc định là 10. Sắp xếp mặc định theo created_at DESC. |
| BR-004 | Manager & Office Mapping Logic | Do bảng mst_saki_department không có liên kết trực tiếp tới mst_saki_office:<br>1. **Trưởng phòng ban (部署長):** Xác định thông qua cột default_dispatch_supervisor_id làm user_id liên kết sang bảng mst_saki_user để lấy họ tên.<br>2. **Sự nghiệp sở trực thuộc (所属事業所):** Lấy gián tiếp thông qua sự nghiệp sở của Trưởng phòng ban (mst_saki_user.office_id -> JOIN mst_saki_office.display_name_ja). Nếu Trưởng phòng ban chưa được cấu hình, thông tin sự nghiệp sở trực thuộc trả về null. |
| BR-005 | Member Count Calculation | Số lượng nhân sự (member_count) của phòng ban được đếm (COUNT) từ bảng mst_saki_user có department_id trùng khớp với department_id của phòng ban đó và có status = 1 (đang hoạt động). |
| BR-006 | Filter & Search Logic | - Lọc theo keyword: Tìm kiếm tương đối (LIKE %keyword%) trên department_id, display_name_ja, official_name_ja hoặc tìm kiếm tên Trưởng phòng ban.<br>- Lọc theo office_id: Lọc các phòng ban mà Trưởng phòng ban trực thuộc sự nghiệp sở đó (mst_saki_user.office_id = :office_id). |

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
4. Kiểm tra người dùng có sở hữu quyền "department.view" hay không.
5. Nếu hợp lệ, cho phép xử lý tiếp. Ngược lại, trả về HTTP 403 Forbidden.
```

---

# 10. Data Mapping

## 10.1 Mapping từ Request vào DB

| Request Field | Table | Column | Logic truy vấn SQL |
| --- | --- | --- | --- |
| keyword | tenant_saki.mst_saki_department | department_id, display_name_ja, official_name_ja | WHERE (department_id LIKE %keyword% OR display_name_ja LIKE %keyword% OR official_name_ja LIKE %keyword%) |
| office_id | tenant_saki.mst_saki_user | office_id | Lọc gián tiếp qua trưởng phòng: WHERE mst_saki_user.office_id = :office_id (kết hợp JOIN với default_dispatch_supervisor_id) |
| status | tenant_saki.mst_saki_department | status | WHERE status = :status |
| limit | - | - | LIMIT :limit |
| page | - | - | OFFSET (:page - 1) * :limit |

---

## 10.2 Mapping từ DB ra Response

| Table | Column | Response Field | Mô tả |
| --- | --- | --- | --- |
| tenant_saki.mst_saki_department | department_id | data[].department_id | Mã phòng ban |
| tenant_saki.mst_saki_department | official_name_ja | data[].official_name_ja | Tên phòng ban chính thức (tiếng Nhật) |
| tenant_saki.mst_saki_department | display_name_ja | data[].display_name_ja | Tên phòng ban hiển thị (tiếng Nhật) |
| tenant_saki.mst_saki_department | official_name_en | data[].official_name_en | Tên phòng ban chính thức (tiếng Anh) |
| tenant_saki.mst_saki_department | display_name_en | data[].display_name_en | Tên phòng ban hiển thị (tiếng Anh) |
| tenant_saki.mst_saki_user | office_id | data[].office_id | Mã sự nghiệp sở trực thuộc trưởng phòng |
| tenant_saki.mst_saki_office | display_name_ja | data[].office_name | Tên sự nghiệp sở hiển thị |
| tenant_saki.mst_saki_department | default_dispatch_supervisor_id | data[].manager_user_id | ID người dùng của Trưởng phòng ban |
| tenant_saki.mst_saki_user | last_name_ja, first_name_ja | data[].manager_name | Ghép Họ & Tên trưởng phòng (ví dụ "山田 太郎") |
| - | - | data[].member_count | Đếm số lượng user thuộc department (`status = 1`) |
| tenant_saki.mst_saki_department | status | data[].status | Trạng thái phòng ban |
| tenant_saki.mst_saki_department | created_at | data[].created_at | Thời điểm tạo |
| tenant_saki.mst_saki_department | updated_at | data[].updated_at | Thời điểm cập nhật |

---

# 11. Database Transaction

## 11.1 Phạm vi transaction

Không áp dụng (Phương thức GET chỉ đọc dữ liệu không thực hiện ghi DB).

---

## 11.2 Điều kiện rollback

Không áp dụng.

---

# 12. Status Transition

Không áp dụng.

---

# 13. Sequence / Processing Flow

```
1. Client gửi yêu cầu GET /api/v1/saki/settings/departments kèm bộ lọc và phân trang.
2. Middleware kiểm tra xác thực JWT và phân quyền (quyền "department.view").
3. Validate tham số đầu vào (page, limit, keyword, office_id, status). Trả về HTTP 422 nếu lỗi.
4. Xác định tenant từ user context, thiết lập kết nối động tới schema `tenant_saki_<company_id>`.
5. Truy vấn dữ liệu:
   - SELECT phòng ban LEFT JOIN `mst_saki_user` và `mst_saki_office` để lấy tên Trưởng phòng ban và Sự nghiệp sở trực thuộc.
   - COUNT số lượng nhân viên hoạt động (`status = 1`) có `department_id` tương ứng từ `mst_saki_user`.
   - Áp dụng các điều kiện lọc (keyword, office_id, status), sắp xếp và phân trang.
6. Trả về response JSON thành công (Paginated Collection) kèm HTTP 200 OK.
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
| Quy trình download | Không áp dụng |
| Thời gian hết hạn | Không áp dụng |
| Audit Log | Không áp dụng |

---

# 17. Security Considerations

| No. | Hạng mục | Mô tả |
| --- | --- | --- |
| 1 | Authentication & Authorization | Phải xác thực JWT token và kiểm tra quyền department.view của người dùng trước khi xử lý. |
| 2 | Tenant Isolation | Thiết lập kết nối động tới đúng tenant schema dựa trên thông tin công ty của người dùng đăng nhập để cách ly dữ liệu. Không cho phép truyền tham số company_id từ client để thay đổi tenant. |
| 3 | SQL Injection Prevention | Sử dụng Eloquent ORM / Parameterized Query cho tất cả các điều kiện lọc, tìm kiếm. |
| 4 | Whitelist-based Sorting | Chỉ cho phép sắp xếp theo danh sách cột an toàn được định nghĩa trước. |

---

# 18. Performance Considerations

| Hạng mục | Target |
| --- | --- |
| Thời gian response kỳ vọng | Trong vòng 0.8 giây |
| Lượng truy cập dự kiến | Trung bình |
| Pagination | Bắt buộc |
| Index cần thiết | Khóa chính trên các bảng liên quan. Index composite trên mst_saki_user(company_id, department_id) để tối ưu hóa việc đếm số lượng nhân viên. |
| Cache cần thiết | Không |
| Bulk Processing | Không |
| Async Processing | Không |

---

# 19. Tài liệu liên quan

| Tài liệu | Reference |
| --- | --- |
| Business Flow | BF-001 User Management Flow |
| Wireframe | SA-SET-002 部署マスタ管理 |
| Screen Detail Design | SA-SET-002 |
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
