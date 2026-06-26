# SCREEN SPECIFICATION

# Màn hình Chủng loại chấm công Master

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | MO-SET-006 |
| Tên màn hình | Quản lý chủng loại chấm công |
| Tên tiếng Nhật | 勤怠種別マスタ |
| Module | Company Settings |
| URL | /moto/settings/attendance-categories |
| Actor | Admin, Sales, Operation |
| Priority | P1 |

---

# 2. Mục đích

Cho phép người dùng quản lý, tìm kiếm, thêm mới, xem lịch sử thay đổi và cập nhật danh mục chủng loại chấm công, bao gồm thiết lập phân loại tính toán và tỷ lệ phụ trội phục vụ cho việc tính hóa đơn và liên kết lương.

Sau khi lưu thành công:

- Cập nhật/Thêm mới bản ghi chủng loại chấm công vào Database
- Ghi log hệ thống (Audit Log)
- Hiển thị Toast thông báo thành công và reload danh sách.

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập MOTO Portal
- Có quyền attendance_type.view

## Điều kiện sau

- Hiển thị danh sách chủng loại chấm công thành công

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên |
| --- | --- |
| MO-SET-006 | Attendance Type Master |

---

## Màn hình đích

| Action | Screen ID | Screen Name |
| --- | --- | --- |
| Create | MO-SET-006 | Attendance Type Master |
| Update | MO-SET-006 | Attendance Type Master |
| Cancel | MO-SET-006 | Attendance Type Master |

---

# 5. UI/UX Layout

- [Danh sách] list.png
- [Cập nhật] update.png

---

# 6. Quy tắc UI/UX

## Tab điều hướng
- Tab Danh sách (一覧): Hiển thị danh mục chủng loại chấm công chính.
- Tab Lịch sử thay đổi (変更履歴): Hiển thị log cập nhật master.

---

## Bộ lọc tìm kiếm
- Trạng thái (ステータス): Lọc theo Trạng thái (Radio Button). Mặc định chọn "Hoạt động" (有効). Các tùy chọn: 有効 (Hoạt động) / 無効 (Vô hiệu) / Tất cả.
- Từ khóa (キーワード): Tìm kiếm tương đối theo Mã chủng loại hoặc Tên chủng loại chấm công.

---

## Bảng dữ liệu
- Cột hiển thị: Mã chủng loại (種別コード), Tên chủng loại (種別名), Tên viết tắt (略称), Phân loại tính toán (計算区分), Tỷ lệ phụ trội (割増率), Trạng thái (ステータス), Thao tác (操作).
- Tỷ lệ phụ trội: Hiển thị dạng phần trăm (ví dụ 25%) đối với các phân loại có áp dụng; hiển thị "—" đối với phân loại không áp dụng (như 通常, 特別).
- Trạng thái hiệu lực: Badge màu xanh (有効) hoặc màu xám (無効).
- Thao tác: Gồm nút 編集 (Chỉnh sửa) và 履歴 (Lịch sử).

---

## Form nhập liệu (Thêm mới/Cập nhật)

### 1. 基本情報 (Thông tin cơ bản)
- Mã chủng loại (種別コード): Cho phép nhập khi thêm mới; Readonly khi chỉnh sửa. Chỉ cho phép chữ và số Half-width, tối đa 16 ký tự.
- Tên viết tắt (略称): Tên viết tắt của chủng loại chấm công, tối đa 50 ký tự.
- Tên chủng loại (種別名): Tên chủng loại chấm công, tối đa 100 ký tự.
- Trạng thái (ステータス): Dropdown/Select để chọn 有効 (1: Hoạt động) hoặc 無効 (0: Vô hiệu). Mặc định chọn 有効.

### 2. 計算設定 (Thiết lập tính toán)
- Phân loại tính toán (計算区分): Dropdown lựa chọn phân loại tính toán. Các lựa chọn: 通常 (Thường), 残業 (Tăng ca), 深夜 (Ca đêm), 休日 (Ngày nghỉ), 特別 (Đặc biệt).
- Tỷ lệ phụ trội (%) (割増率 (%)): Cho phép nhập số nguyên không âm (0 - 100) khi Phân loại tính toán là 残業, 深夜, 休日. Disable và hiển thị trống nếu chọn 通常 hoặc 特別.
- Mã liên kết lương (給与連携コード): TextBox cho phép nhập mã liên kết với phần mềm tính lương, tối đa 50 ký tự (chữ và số Half-width).

### 3. 備考 (Ghi chú)
- Ghi chú (備考): TextArea cho phép nhập ghi chú tự do, tối đa 255 ký tự.

Trường bắt buộc: Hiển thị dấu * đỏ sau nhãn.

---

# 7. Định nghĩa Item

## Bộ lọc tìm kiếm

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 1 | Trạng thái | Radio | No | 有効 / 無効 / Tất cả | status |
| 2 | Từ khóa | Textbox | No | 100 ký tự | category_code / category_name |

---

## Thông tin cơ bản & Thiết lập (Form Thêm mới / Cập nhật)

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 3 | Mã chủng loại | Textbox | Yes | 16 ký tự, Half-width | category_code |
| 4 | Tên viết tắt | Textbox | Yes | 50 ký tự | short_name |
| 5 | Tên chủng loại | Textbox | Yes | 100 ký tự | category_name |
| 6 | Trạng thái | Select | Yes | 1: 有効, 0: 無効 (Mặc định 1) | status |
| 7 | Phân loại tính toán | Select | Yes | 1: 通常, 2: 残業, 3: 深夜, 4: 休日, 5: 特別 | calc_category |
| 8 | Tỷ lệ phụ trội (%) | Textbox (Number) | No | Số nguyên từ 0 - 100 | premium_rate |
| 9 | Mã liên kết lương | Textbox | No | 50 ký tự, Half-width | payroll_link_code |
| 10 | Ghi chú | Textarea | No | 255 ký tự | remarks |

---

## Các nút hành động

| Item | Type | Required | Mô tả |
| --- | --- | --- | --- |
| Create (新規追加) | Button | - | Chuyển hướng sang màn hình Thêm mới |
| Save (保存する) | Button | - | Submit dữ liệu cập nhật |
| Update (編集) | Button | - | Mở form chỉnh sửa dòng tương ứng |
| Lịch sử (履歴) | Button | - | Xem popup lịch sử thay đổi dòng tương ứng |
| Quay lại (<-) | Link/Button | - | Hủy bỏ và quay lại danh sách |

---

# 8. Validation

## category_code

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 区分コードを入力してください。 (Vui lòng không để trống trường Mã chủng loại.) |
| Max 16 | CMS-VAL-6 | 区分コードは16文字以内で入力してください。 (Vui lòng nhập Mã chủng loại trong vòng 16 ký tự trở lâu hơn.) |
| Format | CMS-VAL-24 | 区分コードに正しい形式を指定してください。 (Vui lòng nhập Mã chủng loại đúng định dạng yêu cầu.) |
| Unique | CMS-VAL-11 | 区分コードの値は既に存在しています。 (Giá trị của Mã chủng loại đã tồn tại trong hệ thống (không được trùng lặp).) |

---

## short_name

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 略称を入力してください。 (Vui lòng không để trống trường Tên viết tắt.) |
| Max 50 | CMS-VAL-6 | 略称は50文字以内で入力してください。 (Vui lòng nhập Tên viết tắt trong vòng 50 ký tự trở xuống.) |

---

## category_name

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 区分名を入力してください。 (Vui lòng không để trống trường Tên chủng loại.) |
| Max 100 | CMS-VAL-6 | 区分名は100文字以内で入力してください。 (Vui lòng nhập Tên chủng loại trong vòng 100 ký tự trở xuống.) |

---

## status

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | ステータスを入力してください。 (Vui lòng không để trống Trạng thái.) |
| In | CMS-VAL-41 | 選択されたステータスは正しくありません。 (Trạng thái được chọn không hợp lệ.) |

---

## calc_category

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 計算区分を入力してください。 (Vui lòng không để trống Phân loại tính toán.) |
| In | CMS-VAL-41 | 選択された計算区分は正しくありません。 (Phân loại tính toán được chọn không hợp lệ.) |

---

## premium_rate

| Rule | Message Code | Message |
| --- | --- | --- |
| Integer | CMS-VAL-40 | 割増率は整数で指定してください。 (Vui lòng nhập Tỷ lệ phụ trội dưới dạng số nguyên.) |
| Range 0-100 | CMS-VAL-24 | 割増率に正しい形式を指定してください。 (Vui lòng nhập Tỷ lệ phụ trội đúng định dạng yêu cầu.) |

---

## payroll_link_code

| Rule | Message Code | Message |
| --- | --- | --- |
| Max 50 | CMS-VAL-6 | 給与連携コードは50文字以内で入力してください。 (Vui lòng nhập Mã liên kết lương trong vòng 50 ký tự trở xuống.) |
| Format | CMS-VAL-24 | 給与連携コードに正しい形式を指定してください。 (Vui lòng nhập Mã liên kết lương đúng định dạng yêu cầu.) |

---

## remarks

| Rule | Message Code | Message |
| --- | --- | --- |
| Max 255 | CMS-VAL-6 | 備考は255文字以内で入力してください。 (Vui lòng nhập Ghi chú trong vòng 255 ký tự trở xuống.) |

---

# 9. Event Definition

## Initial Load

### Trigger

Mở màn hình danh sách 

### Process

1. Gọi API Get Attendance Type Master (List).
2. Mặc định tải trang 1, lọc theo trạng thái hoạt động (status = 1).
3. Hiển thị Grid danh sách chủng loại chấm công.

---

## Search

### Trigger

Thay đổi bộ lọc Trạng thái hoặc nhập Từ khóa.

### Process

1. Gửi tham số tìm kiếm qua API.
2. Cập nhật dữ liệu hiển thị trên Grid và phân trang.

---

## Save 

### Trigger

Click 保存する trên form thêm mới / cập nhật.

### Process

1. Validate toàn bộ các trường dữ liệu trên form.
2. Hiển thị popup xác nhận cập nhật (CMS-VAL-85).
3. Call API Update MO-SET-006-API-02 (PATCH) hoặc API Create (POST).
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

## mst_moto_attendance_category

| Column | Type | Description |
| --- | --- | --- |
| company_id | varchar(20) | Mã công ty |
| category_code | varchar(16) | Mã chủng loại chấm công |
| short_name | varchar(50) | Tên viết tắt |
| category_name | varchar(100) | Tên chủng loại chấm công |
| status | smallint | Trạng thái (0: Vô hiệu, 1: Hoạt động) |
| calc_category | smallint | Phân loại tính toán (1: 通常, 2: 残業, 3: 深夜, 4: 休日, 5: 特別) |
| premium_rate | smallint | Tỷ lệ phụ trội (%) (Cho phép NULL) |
| payroll_link_code | varchar(50) | Mã liên kết lương (Cho phép NULL) |
| remarks | text | Ghi chú (Cho phép NULL) |
| created_at | timestamptz | Thời điểm tạo |
| updated_at | timestamptz | Thời điểm cập nhật |
| created_by | varchar(100) | Người tạo |
| updated_by | varchar(100) | Người cập nhật |

---

# 11. API Mapping

## Get Attendance Type Master

```
GET /api/v1/moto/settings/attendance-type-master
```

Request

```json
{
  "page": 1,
  "limit": 20,
  "keyword": "通常",
  "status": 1
}
```

Response

```json
{
  "data": [
    {
      "category_code": "ATT002",
      "short_name": "残業",
      "category_name": "時間外勤務",
      "status": 1,
      "calc_category": 2,
      "premium_rate": 25,
      "payroll_link_code": "PAY-OT-001",
      "remarks": "",
      "created_at": "2026-06-25T08:00:00+09:00",
      "updated_at": "2026-06-25T08:00:00+09:00"
    }
  ]
}
```

---

## Update Attendance Type Master

```
PATCH /api/v1/moto/settings/attendance-type-master/{category_code}
```

Request

```json
{
  "short_name": "残業",
  "category_name": "時間外勤務(更新)",
  "status": 1,
  "calc_category": 2,
  "premium_rate": 25,
  "payroll_link_code": "PAY-OT-001",
  "remarks": "Cập nhật ghi chú"
}
```

Response

```json
{
  "data": {
    "category_code": "ATT002",
    "short_name": "残業",
    "category_name": "時間外勤務(更新)",
    "status": 1,
    "calc_category": 2,
    "premium_rate": 25,
    "payroll_link_code": "PAY-OT-001",
    "remarks": "Cập nhật ghi chú",
    "created_at": "2026-06-25T08:00:00+09:00",
    "updated_at": "2026-06-25T15:20:00+09:00"
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
| CMS-VAL-23 | {0}を入力してください。 | Vui lòng không để trống trường {0}. | Inline Validation |
| CMS-VAL-6 | {0}は{1}文字以内で入力してください。 | Vui lòng nhập {0} trong vòng {1} ký tự trở xuống. | Inline Validation |
| CMS-VAL-24 | {0}に正しい形式を指定してください。 | Vui lòng nhập {0} đúng định dạng yêu cầu. | Inline Validation |
| CMS-VAL-11 | {0}の値は既に存在しています。 | Giá trị của {0} đã tồn tại trong hệ thống (không được trùng lặp). | Inline Validation |
| CMS-VAL-40 | {0}は整数で指定してください。 | Vui lòng chỉ định {0} là một số nguyên. | Inline Validation |
| CMS-VAL-41 | 選択された{0}は正しくありません。 | {0} được chọn không hợp lệ. | Inline Validation |
| CMS-VAL-79 | {Screen name}を更新しました。 | Đã cập nhật {Screen name}. | Toast Success |
| CMS-VAL-85 | {Target}を更新します。よろしいですか。 | Sẽ tiến hành cập nhật {Target}. Bạn có chắc chắn không? | Dialog Confirm |
| CMS-VAL-95 | この機能・リソースへのアクセス権限がありません。 | Bạn không có quyền truy cập vào chức năng/tài nguyên này. | Toast Error |
| CMS-VAL-99 | システムエラーが発生しました。システム管理者に連絡してください。 | Đã xảy ra lỗi hệ thống. Vui lòng liên hệ với người quản trị. | Toast Error / Popup |

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
