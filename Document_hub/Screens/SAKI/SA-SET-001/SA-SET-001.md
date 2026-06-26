# SCREEN SPECIFICATION

# Màn hình Quản lý người dùng SAKI

---

# 1. Thông tin màn hình

| Item | Nội dung |
| --- | --- |
| Screen ID | SA-SET-001 |
| Tên màn hình | Quản lý người dùng |
| Tên tiếng Nhật | ユーザー管理 |
| Module | Company Settings |
| URL | /saki/settings/users |
| Actor | SAKI Admin, SAKI User tùy thuộc quyền hạn |
| Priority | P1 |

---

# 2. Mục đích

Cho phép Admin của công ty phái cử đối tác SAKI Portal quản lý danh sách tài khoản người dùng, tìm kiếm, thêm mới mời người dùng, phân quyền chi tiết, reset mật khẩu, khóa/mở khóa tài khoản và thiết lập quyền phê duyệt của từng thành viên trong cùng Tenant.

Sau khi lưu thành công:

- Cập nhật/Thêm mới bản ghi người dùng vào Database
- Cập nhật phạm vi tham chiếu dữ liệu và phân quyền chi tiết
- Cập nhật thiết lập quyền phê duyệt
- Ghi log

---

# 3. Điều kiện truy cập

## Điều kiện trước

- Đã đăng nhập SAKI Portal
- Có quyền user.view

## Điều kiện sau

- Hiển thị danh sách người dùng SAKI thành công

---

# 4. Di chuyển màn hình

## Màn hình nguồn

| Screen ID | Tên |
| --- | --- |
| SA-SET-001 | User Management |

---

## Màn hình đích

| Action | Screen ID | Screen Name |
| --- | --- | --- |
| Create | SA-SET-001 | User Management |
| Update | SA-SET-001 | User Management |
| View Detail | SA-SET-001 | User Management |

---

# 5. UI/UX Layout

![- \[Danh sách\] saki_user_list.png](saki_user_list.png)
![- \[Cập nhật\] saki_user_update.png](saki_user_update.png)

---

# 6. Quy tắc UI/UX

## 6.1 Màn hình Danh sách

### KPI Cards
- Thống kê các chỉ số:
  - Tổng người dùng
  - Người dùng hoạt động
  - Người dùng đang bị khóa
  - Người dùng giữ quyền phê duyệt
  - Người dùng bị vô hiệu
- Các số liệu này được tải động từ API Saki User List.

### Bộ lọc tìm kiếm
- Bộ lọc cho phép mở rộng/thu gọn.
- Hỗ trợ các tiêu chí lọc:
  - Tên người dùng
  - ID người dùng
  - Vai trò: Dropdown chọn Admin / Approver / Staff / Viewer...
  - Bộ phận trực thuộc: Dropdown chọn phòng ban.
  - Trạng thái: Dropdown chọn 有効 / 無効 / ロック中.
  - Tùy chọn hiển thị: Checkbox "Chỉ người giữ quyền phê duyệt" và Checkbox "Chỉ người dùng hoạt động".
- Nút リセット: Xóa toàn bộ điều kiện lọc và quay về trạng thái mặc định.

### Thao tác chọn nhiều
- Khi check chọn ít nhất một dòng trong bảng, thanh hành động chọn nhiều sẽ xuất hiện:
  - Hiển thị số lượng dòng được chọn.
  - Nút 選択解除: Bỏ check toàn bộ các dòng.
  - Nút 一括ロック: Bấm để khóa các tài khoản được chọn.
  - Nút 一括無効化: Bấm để vô hiệu hóa các tài khoản được chọn.

### Bảng dữ liệu
- Các cột: Checkbox chọn dòng, Tên người dùng, ID người dùng, Bộ phận, Vai trò, Quyền phê duyệt, Trạng thái, Đăng nhập cuối, Thao tác.
- Quyền phê duyệt: Nếu user có quyền phê duyệt, hiển thị icon hình cái khiên + chữ あり. Nếu không, hiển thị dấu "-".
- Cột Thao tác:
  - Nút 編集: Mở màn hình cập nhật chi tiết của user tương ứng.
  - Icon Ba chấm dọc: Click để hiển thị các tùy chọn thao tác phụ.
  - Đối với user đang bị khóa: Cột thao tác hiển thị nút 解除.
  - Đối với user đang vô hiệu: Cột thao tác hiển thị nút 有効化.

---

## 6.2 Màn hình Chi tiết / Cập nhật

### Breadcrumb & Header
- Hiển thị đường dẫn điều hướng: 派遣先株式会社 > ユーザー管理 / 詳細.
- Bấm <- ユーザー管理に戻る để hủy bỏ và quay lại danh sách.
- Header hiển thị Badge trạng thái hiện tại, Login ID và Họ tên người dùng dạng chữ lớn.

### Panel Thao tác bên phải
- Panel Thao tác:
  - 変更を保存: Submit form cập nhật dữ liệu.
  - パスワードリセット: Gửi email hướng dẫn reset mật khẩu hoặc tạo mới mật khẩu tạm thời.
  - アカウントロック解除 / アカウントロック: Mở khóa nếu tài khoản đang bị khóa, hoặc thực hiện khóa tài khoản.
  - ユーザー無効化 / ユーザー有効化: Chuyển đổi trạng thái hoạt động của user.
- Panel Tóm tắt người dùng:
  - Hiển thị tóm tắt Trạng thái, Vai trò, Phòng ban trực thuộc, và icon Quyền phê duyệt.
- Panel Lưu ý:
  - Nhắc nhở cẩn trọng khi chỉnh sửa tài khoản có quyền Admin.
- Panel Thao tác nhanh:
  - Link xem lịch sử thao tác, xem lịch sử đăng nhập.

### Form thông tin chi tiết (Main Form)
Form được chia làm các khối thông tin rõ ràng:
1. **基本情報:**
   - Họ tên, ID đăng nhập, Email, Bộ phận, Chức vụ, Trạng thái.
   - ID đăng nhập: Chế độ readonly không cho sửa khi cập nhật.
2. **役割・権限設定:**
   - Vai trò: Dropdown chọn Admin / Approver / Staff / Viewer...
   - Module có thể truy cập: Checkbox chọn 依頼管理, 候補者管理, 契約管理, 勤怠管理, 請求書管理.
   - Quyền thao tác chi tiết: Checkbox bật/tắt quyền tạo yêu cầu, duyệt hợp đồng, duyệt chấm công, xác nhận hóa đơn, download/xuất excel.
3. **承認権限設定:**
   - Checkbox "Thiết lập làm người duyệt".
   - Nếu check chọn, cho phép thiết lập tiếp: Cấp độ duyệt, Phạm vi đối tượng duyệt, Người duyệt thay thế, Quy tắc leo thang.
4. **ログイン・セキュリティ:**
   - Đăng nhập cuối, Số lần đăng nhập, Trạng thái khóa, Trạng thái mật khẩu, Ngày tạo, Cập nhật cuối.
5. **メモ・監査ログ:**
   - Quản lý ghi chú: Textarea để ghi chú nội bộ của Admin.
   - Lịch sử thay đổi quyền: Danh sách thời gian, nội dung và người thực hiện thay đổi quyền.

---

# 7. Định nghĩa Item

## 7.1 Bộ lọc tìm kiếm (Màn hình Danh sách)

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 1 | Tên người dùng | Textbox | No | 100 ký tự | user_name |
| 2 | ID người dùng | Textbox | No | 100 ký tự | user_id |
| 3 | Vai trò | Select | No | Chọn từ danh sách Role | execution_role_id |
| 4 | Bộ phận | Select | No | Chọn từ danh sách Department | department_id |
| 5 | Trạng thái | Select | No | 有効 / 無効 / ロック | status |
| 6 | Chỉ người giữ quyền phê duyệt | Checkbox | No | 0: Tất cả, 1: Chỉ người phê duyệt | show_as_approver_flg |
| 7 | Chỉ người dùng hoạt động | Checkbox | No | 0: Tất cả, 1: Chỉ người hoạt động | status |

---

## 7.2 Khối Thông tin cơ bản (Form Nhập liệu / Chi tiết)

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 8 | Họ tiếng Nhật | Textbox | Yes | 24 ký tự | last_name_ja |
| 9 | Tên tiếng Nhật | Textbox | Yes | 24 ký tự | first_name_ja |
| 10 | Họ Katakana | Textbox | Yes | 24 ký tự, Katakana | last_name_kana |
| 11 | Tên Katakana | Textbox | Yes | 24 ký tự, Katakana | first_name_kana |
| 12 | ID người dùng | Textbox | Yes | 100 ký tự, Half-width | user_id |
| 13 | Email | Textbox | Yes | Email format, 128 ký tự | email |
| 14 | Số điện thoại | Textbox | Yes | Half-width, 15 ký tự | tel |
| 15 | Bộ phận | Select | Yes | Chọn từ danh sách phòng ban | department_id |
| 16 | Văn phòng | Select | Yes | Chọn từ danh sách văn phòng | office_id |
| 17 | Chức vụ | Textbox | No | 50 ký tự | position_ja |
| 18 | Trạng thái | Dropdown | Yes | 1: 有効, 0: 無効 | status |

---

## 7.3 Khối Vai trò & Phân quyền (Form Nhập liệu / Chi tiết)

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 19 | Vai trò | Select | Yes | Chọn từ danh sách Role | execution_role_id |
| 20 | Phạm vi tham chiếu | Select | Yes | 1-6 với 1 là cá nhân, 6 là tùy chọn | reference_scope |
| 21 | Quyền tạo yêu cầu | Checkbox | No | 0: Vô hiệu, 1: Có quyền | create_inquiry_flg |
| 22 | Quyền duyệt hợp đồng | Checkbox | No | 0: Vô hiệu, 1: Có quyền | approve_contract_flg |
| 23 | Quyền duyệt chấm công | Checkbox | No | 0: Vô hiệu, 1: Có quyền | approve_attendance_flg |
| 24 | Quyền xác nhận hóa đơn | Checkbox | No | 0: Vô hiệu, 1: Có quyền | confirm_invoice_flg |
| 25 | Quyền download/xuất file | Checkbox | No | 0: Vô hiệu, 1: Có quyền | export_file_flg |
| 26 | Cờ cho sửa thông tin khi duyệt | Checkbox | No | 0: Không, 1: Có | edit_on_approval_flg |
| 27 | Cờ tham chiếu nhóm công ty | Checkbox | No | 0: Không, 1: Có | view_group_company_flg |
| 28 | Cờ cho sửa phòng ban hợp đồng | Checkbox | No | 0: Không, 1: Có | edit_contract_dept_flg |

---

## 7.4 Khối Cấu hình quyền phê duyệt (Form Nhập liệu / Chi tiết)

| No | Item | Type | Required | Format | DB |
| --- | --- | --- | --- | --- | --- |
| 29 | Thiết lập làm người duyệt | Checkbox | No | 0: Không, 1: Có | show_as_approver_flg |
| 30 | Cấp độ duyệt | Select | No | Final / 1st / 2nd... | final_approver_flg |
| 31 | Phạm vi đối tượng duyệt | Select | No | 1: Toàn công ty, 2: Chỉ phòng ban | approval_target_range |
| 32 | Người duyệt thay thế | Select | No | Chọn user khác | deputy_approver_id |
| 33 | Quy tắc leo thang | Select | No | Quy tắc chuyển giao duyệt thay thế | escalation_rule_id |

---

## 7.5 Các nút hành động

| Item | Type | Required | Mô tả |
| --- | --- | --- | --- |
| 新規ユーザー作成 | Button | - | Mở form thêm mới mời người dùng |
| 一括取込 | Button | - | Mở popup/màn hình import file CSV |
| エクスポート | Button | - | Xuất Excel/CSV danh sách người dùng |
| 変更を保存 | Button | - | Lưu toàn bộ thông tin cập nhật trên form |
| パスワードリセット | Button | - | Thực hiện reset mật khẩu và gửi mail |
| アカウントロック解除 | Button | - | Thực hiện mở khóa tài khoản |
| ユーザー無効化 | Button | - | Đổi trạng thái status về 0 |
| ユーザー有効化 | Button | - | Đổi trạng thái status về 1 |

---

# 8. Validation

## Họ & Tên

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 氏名を入力してください。 |
| Max 24 | CMS-VAL-6 | 氏名は24文字以内で入力してください。 |
| Kana Format | CMS-VAL-24 | カナ氏名に正しい形式を指定してください。 |

---

## user_id

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | ユーザーIDを入力してください。 |
| Max 100 | CMS-VAL-6 | ユーザーIDは100文字以内で入力してください。 |
| Format | CMS-VAL-24 | ユーザーIDに正しい形式を指定してください。 |
| Unique | CMS-VAL-11 | ユーザーIDの値は既に存在しています。 |

---

## email

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | メールアドレスを入力してください。 |
| Max 128 | CMS-VAL-6 | メールアドレスは128文字以内で入力してください |
| Format | CMS-VAL-24 | メールアドレスに正しい形式を指定してください。 |
| Unique | CMS-VAL-11 | メールアドレスの値は既に存在しています。 |

---

## tel

| Rule | Message Code | Message |
| --- | --- | --- |
| Required | CMS-VAL-23 | 電話番号を入力してください。 |
| Max 15 | CMS-VAL-6 | 電話番号は15文字以内で入力してください。 |
| Format | CMS-VAL-24 | 電話番号に正しい形式を指定してください。 |

---

# 9. Event Definition

## Initial Load (Màn hình danh sách)

### Trigger
Người dùng click vào menu "ユーザー管理" (User Management).

### Process
1. Gọi API Saki User List (GET `/api/v1/saki/settings/users`).
2. Mặc định tải trang 1, sắp xếp theo thời gian tạo mới nhất.
3. Hiển thị Grid danh sách tài khoản cùng thông tin KPI thống kê số lượng tài khoản phía trên.

---

## Search & Filter

### Trigger
Admin nhập Từ khóa/ID, chọn bộ lọc vai trò, bộ phận, trạng thái, check tùy chọn hoặc bấm nút tìm kiếm.

### Process
1. Thu thập các tham số tìm kiếm trên UI.
2. Gọi API Saki User List với các tham số tương ứng.
3. Cập nhật Grid kết quả hiển thị và phân trang.

---

## Mở khóa tài khoản (Unlock Account)

### Trigger
Admin click nút アカウントロック解除 trên panel Thao tác bên phải.

### Process
1. Hiển thị Popup xác nhận mở khóa tài khoản.
2. Khi xác nhận, gọi API Update Saki User (PUT `/api/v1/saki/settings/users/{id}` hoặc API Unlock chuyên dụng) truyền trạng thái không bị khóa và reset số lần đăng nhập sai.
3. Trả về thông báo thành công và reload lại dữ liệu form/dòng.

---

## Save 

### Trigger
Admin click 変更を保存 (Lưu thay đổi) trên form cập nhật.

### Process
1. Thực hiện validate toàn bộ các trường nhập liệu trên form. Nếu có lỗi, hiển thị thông báo lỗi inline tại trường tương ứng và dừng xử lý.
2. Hiển thị popup xác nhận lưu.
3. Gọi API Update Saki User (PUT `/api/v1/saki/settings/users/{id}`) hoặc API Create Saki User (POST `/api/v1/saki/settings/users` nếu thêm mới).
4. Hệ thống thực hiện cập nhật database trong Transaction.
5. Ghi Audit Log.
6. Hiển thị Toast thông báo thành công.
7. Reload dữ liệu và cập nhật giao diện form.

---

# 10. Mapping Database

## mst_saki_user

| Column | Type | Description |
| --- | --- | --- |
| company_id | varchar | Mã công ty (Tenant ID) |
| user_id | varchar | ID đăng nhập người dùng |
| last_name_ja | varchar | Họ tiếng Nhật |
| first_name_ja | varchar | Tên tiếng Nhật |
| last_name_kana | varchar | Họ Katakana |
| first_name_kana | varchar | Tên Katakana |
| email | varchar | Email liên hệ |
| tel | varchar | Số điện thoại |
| office_id | varchar | Văn phòng trực thuộc |
| department_id | varchar | Phòng ban trực thuộc |
| position_ja | varchar | Chức vụ |
| execution_role_id | varchar | Vai trò phân quyền |
| reference_scope | smallint | Phạm vi tham chiếu dữ liệu |
| show_as_approver_flg | smallint | Cờ hiển thị làm người phê duyệt |
| final_approver_flg | smallint | Cờ người duyệt cuối cùng |
| status | smallint | Trạng thái (0: Vô hiệu, 1: Hoạt động) |
| remarks | text | Ghi chú của admin |
| created_at | timestamptz | Thời điểm tạo |
| updated_at | timestamptz | Thời điểm cập nhật |
| created_by | varchar | Người tạo |
| updated_by | varchar | Người cập nhật |

---

# 11. API Mapping

## 11.1 Saki User List

```
GET /api/v1/saki/settings/users
```

Request Query

```json
{
  "page": 1,
  "limit": 20,
  "user_name": "山田",
  "status": 1
}
```

Response

```json
{
  "success": true,
  "data": [
    {
      "user_id": "yamada.taro",
      "user_name": "山田 太郎",
      "last_name_ja": "山田",
      "first_name_ja": "太郎",
      "email": "yamada.taro@saki.co.jp",
      "tel": "09012345678",
      "department_id": "DEP-001",
      "department_name": "人事部",
      "office_id": "OFF-001",
      "office_name": "本社",
      "execution_role_id": "role_admin",
      "execution_role_name": "管理者",
      "has_approval_authority": true,
      "show_as_approver_flg": 1,
      "final_approver_flg": 1,
      "status": 1,
      "last_login_at": "2026-04-22T10:30:00+09:00",
      "created_at": "2024-05-15T09:00:00+09:00"
    }
  ],
  "meta": {
    "total": 45,
    "kpi": {
      "total_users": 45,
      "active_users": 42,
      "locked_users": 2,
      "approver_users": 12,
      "inactive_users": 3
    }
  }
}
```

---

## 11.2 Create Saki User

```
POST /api/v1/saki/settings/users
```

Request Body

```json
{
  "last_name_ja": "山田",
  "first_name_ja": "太郎",
  "last_name_kana": "ヤマダ",
  "first_name_kana": "タロウ",
  "user_id": "yamada.taro",
  "email": "yamada.taro@saki.co.jp",
  "tel": "09012345678",
  "office_id": "OFF-001",
  "department_id": "DEP-001",
  "position_ja": "部長",
  "execution_role_id": "role_admin",
  "reference_scope": 1,
  "status": 1,
  "send_invitation_email": true
}
```

Response

```json
{
  "success": true,
  "data": {
    "user_id": "yamada.taro",
    "email": "yamada.taro@saki.co.jp",
    "status": 1,
    "created_at": "2026-06-25T17:10:00+09:00"
  }
}
```

---

## 11.3 Update Saki User

```
PUT /api/v1/saki/settings/users/{id}
```

Request Body

```json
{
  "last_name_ja": "山田",
  "first_name_ja": "太郎",
  "last_name_kana": "ヤマダ",
  "first_name_kana": "タロウ",
  "email": "yamada.taro@saki.co.jp",
  "tel": "09012345678",
  "office_id": "OFF-001",
  "department_id": "DEP-001",
  "position_ja": "部長",
  "execution_role_id": "role_admin",
  "reference_scope": 1,
  "status": 1,
  "show_as_approver_flg": 1,
  "final_approver_flg": 1,
  "remarks": "Cập nhật ghi chú của admin"
}
```

Response

```json
{
  "success": true,
  "data": {
    "user_id": "yamada.taro",
    "email": "yamada.taro@saki.co.jp",
    "status": 1,
    "updated_at": "2026-06-25T17:15:00+09:00"
  }
}
```

---

# 12. Notification

## Trigger
Khi tài khoản mới được tạo thành công và checkbox "Gửi email mời ngay" được chọn.

## Process
Hệ thống kích hoạt queue gửi email chào mừng và link kích hoạt tài khoản đăng nhập cho user.

---

# 13. Message Definition

| Code | Message tiếng Nhật | Message tiếng Việt | Loại hiển thị |
| --- | --- | --- | --- |
| CMS-VAL-23 | {0}を入力してください。 | Vui lòng không để trống trường {0}. | Inline Validation |
| CMS-VAL-6 | {0}は{1}文字以内で入力してください。 | Vui lòng nhập {0} trong vòng {1} ký tự trở xuống. | Inline Validation |
| CMS-VAL-24 | {0}に正しい形式を指定してください。 | Vui lòng nhập {0} đúng định dạng yêu cầu. | Inline Validation |
| CMS-VAL-11 | {0}の値は既に存在しています。 | Giá trị của {0} đã tồn tại trong hệ thống, không được trùng lặp. | Inline Validation |
| CMS-VAL-40 | {0}は整数で指定してください。 | Vui lòng chỉ định {0} là một số nguyên. | Inline Validation |
| CMS-VAL-41 | 選択された{0}は正しくありません。 | {0} được chọn không hợp lệ. | Inline Validation |
| CMS-VAL-79 | {Screen name}を更新しました/登録しました。 | Đã cập nhật/đăng ký {Screen name}. | Toast Success |
| CMS-VAL-85 | {Target}を更新します/登録します。よろしいですか。 | Sẽ tiến hành cập nhật/đăng ký {Target}. Bạn có chắc chắn không? | Dialog Confirm |
| CMS-VAL-95 | この機能・リソースへのアクセス権限がありません。 | Bạn không có quyền truy cập vào chức năng/tài nguyên này. | Toast Error |
| CMS-VAL-99 | システムエラーが発生しました。 | Đã xảy ra lỗi hệ thống. Vui lòng liên hệ với người quản trị. | Toast Error / Popup |

---

# 14. Permission

| Action | Admin | Approver | Staff | Viewer |
| --- | --- | --- | --- | --- |
| Create | O | X | X | X |
| Update | O | X | X | X |
| View List | O | O | O | O |
| View Detail | O | O | O | O |

---

# 15. Audit Log

| Action | Log |
| --- | --- |
| Create | Yes |
| Update | Yes |
| Reset Password | Yes |
| Suspend / Enable | Yes |

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
- API Specification SA-SET-001-API-01 / SA-SET-001-API-02 / SA-SET-001-API-03
- Portal Permission Matrix
