# SCREEN SPECIFICATION

# Màn hình Loại nơi làm việc Master

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | MO-SET-007 |
| Tên màn hình | Quản lý loại nơi làm việc |
| Tên tiếng Nhật | 就業場所区分マスタ |
| Module | Company Settings |
| URL | /moto/settings/workplace-types |
| Actor | Admin, Sales, Operation |
| Priority | P1 |

---

# 2. Mục đích

Cho phép người dùng quản lý, tìm kiếm, thêm mới, xem lịch sử thay đổi và cập nhật danh mục loại nơi làm việc.

Sau khi lưu thành công:

- Cập nhật/Thêm mới bản ghi loại nơi làm việc vào Database
- Ghi log hệ thống (Audit Log)
- Hiển thị Toast thông báo thành công và reload danh sách.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập MOTO Portal
- Có quyền workplace_type.view

## Điều kiện sau

- Hiển thị danh sách loại nơi làm việc thành công

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên |
| --- | --- |
| MO-SET-007 | Workplace Type Master |

---

## Màn hình đích

| Action | Screen ID | Screen Name |
| --- | --- | --- |
| Create | MO-SET-007 | Workplace Type Master |
| Update | MO-SET-007 | Workplace Type Master |
| Cancel | MO-SET-007 | Workplace Type Master |

---

# 5. UI/UX Layout

![list_workplace_type.png](list_workplace_type.png)

![update_workplace_type.png](update_workplace_type.png)

---

# 6. Quy tắc UI/UX

## Tab điều hướng
- Tab Danh sách: Hiển thị danh mục chính.
- Tab Lịch sử thay đổi: Hiển thị log cập nhật master.

---

## Bộ lọc tìm kiếm
- Trạng thái: Mặc định chọn "Hoạt động".
- Từ khóa: Tìm kiếm tương đối theo Mã hoặc Tên loại nơi làm việc.

---

## Bảng dữ liệu
- Hiển thị 6 cột dữ liệu và cột Thao tác.
- Trạng thái: Badge màu xanh (Hoạt động) hoặc xám (Vô hiệu).
- Hình thức làm việc: Nhãn màu tương ứng (Thường trú: Xanh dương, Tại nhà: Xanh lá, Hybrid: Tím, Công tác: Cam).
- Phí đi lại: Hiển thị "Có" hoặc "Không".

---

## Form nhập liệu (Thêm mới/Cập nhật)
- Mã loại: Cho phép nhập khi thêm mới; Readonly khi chỉnh sửa.
- Trường bắt buộc: Hiển thị dấu * đỏ sau nhãn.

---

# 7. Định nghĩa Item

## Bộ lọc tìm kiếm

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 1 | Trạng thái | Radio | No | 有効 / 無効 | status |
| 2 | Từ khóa | Textbox | No | 100 ký tự | workplace_type_code / workplace_type_name |

---

## Thông tin cơ bản (Form Thêm mới / Cập nhật)

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 3 | Mã loại | Textbox | Yes | 20 ký tự | workplace_type_code |
| 4 | Trạng thái | Dropdown | Yes | 有効 / 無効 | status |
| 5 | Tên loại | Textbox | Yes | 100 ký tự | workplace_type_name |
| 6 | Tên tiếng Anh | Textbox | No | 100 ký tự | workplace_type_name_en |
| 7 | Hình thức làm việc | Dropdown | Yes | 1: On-site, 2: Remote, 3: Hybrid, 4: Business Trip | work_style_type |
| 8 | Đối tượng phí đi lại | Radio | Yes | 0: Không, 1: Có | is_transportation_eligible |

---

## Các nút hành động

| Item | Type | Required | Mô tả |
| --- | --- | --- | --- |
| Create | Button | - | Chuyển hướng sang màn hình Thêm mới |
| Save | Button | - | Submit dữ liệu |
| Update | Button | - | Chuyển tiếp sang màn hình Chỉnh sửa |
| Lịch sử (履歴) | Button | - | Xem popup lịch sử thay đổi dòng tương ứng |
| Quay lại (<-) | Link/Button | - | Hủy bỏ và quay lại danh sách |

---

# 8. Validation

## Mã loại nơi làm việc (workplace_type_code)

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 種別コードを入力してください。 (Vui lòng không để trống trường Mã loại nơi làm việc.) |
| Max 20 | CMS-VAL-6 | 種別コードは20文字以内で入力してください。 (Vui lòng nhập Mã loại nơi làm việc trong vòng 20 ký tự trở xuống.) |
| Format | CMS-VAL-24 | 種別コードに正しい形式を指定してください。 (Vui lòng nhập Mã loại nơi làm việc đúng định dạng yêu cầu.) |
| Unique | CMS-VAL-11 | 種別コードの値は既に存在しています。 (Giá trị của Mã loại nơi làm việc đã tồn tại trong hệ thống (không được trùng lặp).) |

---

## Tên loại nơi làm việc (workplace_type_name)

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 種別名を入力してください。 (Vui lòng không để trống trường Tên loại nơi làm việc.) |
| Max 100 | CMS-VAL-6 | 種別名は100文字以内で入力してください。 (Vui lòng nhập Tên loại nơi làm việc trong vòng 100 ký tự trở xuống.) |

---

## Tên tiếng Anh (workplace_type_name_en)

| Rule | Message Code | Message |
| --- | --- | --- |
| Max 100 | CMS-VAL-6 | 種別名（英字）は100文字以内で入力してください。 (Vui lòng nhập Tên tiếng Anh trong vòng 100 ký tự trở xuống.) |

---

## Hình thức làm việc (work_style_type)

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 勤務形態を入力してください。 (Vui lòng không để trống trường Hình thức làm việc.) |
| In | CMS-VAL-41 | 選択された勤務形態は正しくありません。 (Hình thức làm việc được chọn không hợp lệ.) |

---

## Đối tượng phí đi lại (is_transportation_eligible)

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 交通費対象を入力してください。 (Vui lòng không để trống trường Đối tượng phí đi lại.) |
| In | CMS-VAL-41 | 選択された交通費対象は正しくありません。 (Đối tượng phí đi lại được chọn không hợp lệ.) |

---

## Trạng thái (status)

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | ステータスを入力してください。 (Vui lòng không để trống trường Trạng thái.) |
| In | CMS-VAL-41 | 選択されたステータスは正しくありません。 (Trạng thái được chọn không hợp lệ.) |

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình danh sách 

### Process

1. Gọi API Get Workplace Type Master.
2. Mặc định tải trang 1, lọc theo trạng thái hoạt động.
3. Hiển thị Grid danh sách.

---

## Search

### Trigger

Thay đổi bộ lọc Trạng thái hoặc nhập Từ khóa.

### Process

1. Gửi tham số tìm kiếm qua API MO-SET-007-API-01.
2. Cập nhật dữ liệu hiển thị trên Grid và phân trang.

---

## Save 

### Trigger

Click 保存する trên form thêm mới / cập nhật.

### Process

1. Validate toàn bộ các trường dữ liệu trên form.
2. Hiển thị popup xác nhận cập nhật (CMS-VAL-85).
3. Call API Update MO-SET-007-API-02 (PATCH) hoặc API Create (POST).
4. Ghi Audit Log.
5. Hiển thị Toast thông báo thành công (CMS-VAL-79).
6. Chuyển hướng quay lại màn hình danh sách và reload.

---

## Back

### Trigger

Click nút Back

### Process

Quay lại màn hình danh sách, hủy bỏ mọi thay đổi chưa lưu.

---

# 10. Mapping Database

## mst_moto_workplace_type

| Column | Type |
| --- | --- |
| company_id | varchar(20) |
| workplace_type_code | varchar(20) |
| workplace_type_name | varchar(100) |
| workplace_type_name_en | varchar(100) |
| work_style_type | smallint |
| is_transportation_eligible | smallint |
| status | smallint |
| created_at | timestamptz |
| updated_at | timestamptz |
| created_by | varchar(100) |
| updated_by | varchar(100) |

---

# 11. API Mapping

## Get Workplace Type Master

```
GET /api/v1/moto/settings/workplace-type-master
```

Request

```json
{
  "page": 1,
  "limit": 20,
  "keyword": "常駐",
  "status": 1
}
```

Response

```json
{
  "data": [
    {
      "workplace_type_code": "WPT001",
      "workplace_type_name": "クライアント常駐",
      "workplace_type_name_en": "Client On-site",
      "work_style_type": 1,
      "is_transportation_eligible": 1,
      "status": 1,
      "created_at": "2026-06-23T08:00:00+09:00",
      "updated_at": "2026-06-23T08:00:00+09:00"
    }
  ]
}
```

---

## Update Workplace Type Master

```
PATCH /api/v1/moto/settings/workplace-type-master/{workplace_type_code}
```

Request

```json
{
  "workplace_type_name": "クライアント常駐(更新)",
  "workplace_type_name_en": "Client On-site (Updated)",
  "work_style_type": 1,
  "is_transportation_eligible": 1,
  "status": 1
}
```

Response

```json
{
  "data": {
    "workplace_type_code": "WPT001",
    "workplace_type_name": "クライアント常駐(更新)",
    "workplace_type_name_en": "Client On-site (Updated)",
    "work_style_type": 1,
    "is_transportation_eligible": 1,
    "status": 1,
    "created_at": "2026-06-23T08:00:00+09:00",
    "updated_at": "2026-06-24T17:08:00+09:00"
  }
}
```

---

# 12. Notification

## Trigger

Không áp dụng.

---

# 13. Message Definition

| Code | Message (Tiếng Nhật) | Message (Tiếng Việt) | Loại hiển thị |
| --- | --- | --- | --- |
| **CMS-VAL-23** | {0}を入力してください。 | Vui lòng không để trống trường {0}. | Inline Validation |
| **CMS-VAL-6** | {0}は{1}文字以内で入力してください。 | Vui lòng nhập {0} trong vòng {1} ký tự trở xuống. | Inline Validation |
| **CMS-VAL-24** | {0}に正しい形式を指定してください。 | Vui lòng nhập {0} đúng định dạng yêu cầu. | Inline Validation |
| **CMS-VAL-11** | {0}の値は既に存在しています。 | Giá trị của {0} đã tồn tại trong hệ thống (không được trùng lặp). | Inline Validation |
| **CMS-VAL-40** | {0}は整数で指定してください。 | Vui lòng chỉ định {0} là một số nguyên. | Inline Validation |
| **CMS-VAL-41** | 選択された{0}は正しくありません。 | {0} được chọn không hợp lệ. | Inline Validation |
| **CMS-VAL-79** | {Screen name}を更新しました。 | Đã cập nhật {Screen name}. | Toast Success |
| **CMS-VAL-85** | {Target}を更新します。よろしいですか。 | Sẽ tiến hành cập nhật {Target}. Bạn có chắc chắn không? | Dialog Confirm |
| **CMS-VAL-95** | この機能・リソースへのアクセス権限がありません。 | Bạn không có quyền truy cập vào chức năng/tài nguyên này. | Toast Error |
| **CMS-VAL-99** | システムエラーが発生しました。管理者へお問い合わせください。 | Đã xảy ra lỗi hệ thống. Vui lòng liên hệ với người quản trị. | Toast Error / Popup |

---

# 14. Permission

| Action | Admin | Sales | Operation |
| --- | --- | --- | --- |
| Create | O | X | X |
| Update | O | X | X |
| View | O | O | O |

---

# 15. Audit Log

| Action | Log |
| --- | --- |
| Create | Yes |
| Update | Yes |

---

# 16. Error Handling

| HTTP Code | Message |
| --- | --- |
| 401 | Phiên đăng nhập đã hết hạn |
| 403 | Bạn không có quyền thực hiện thao tác này |
| 404 | Không tìm thấy dữ liệu |
| 409 | Dữ liệu đã tồn tại |
| 422 | Dữ liệu không hợp lệ |
| 500 | Hệ thống đang gặp sự cố |

---

# 17. Related Documents

- Business Flow 
- ERD 
- API Specification 
- Portal Permission Matrix
