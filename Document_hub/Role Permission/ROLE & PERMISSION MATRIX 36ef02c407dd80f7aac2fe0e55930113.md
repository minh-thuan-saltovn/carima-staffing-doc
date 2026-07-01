# Hệ thống Carima Staffing

---

# 1. Tổng quan

## 1.1 Mục đích

Tài liệu này định nghĩa kiến trúc RBAC (Role Based Access Control), Data Scope và Approval Runtime Gate cho hệ thống Carima Staffing.

Mục tiêu:

- Kiểm soát truy cập theo Role cho Platform/SAKI/MOTO internal users
- Phân tách quyền theo portal, tenant và actor
- Giới hạn phạm vi dữ liệu bằng runtime data scope
- Kiểm soát quyền phê duyệt bằng approval route/assignment
- Đáp ứng yêu cầu audit/compliance
- Ngăn thao tác trái phép hoặc vượt trạng thái nghiệp vụ
- Hỗ trợ kiến trúc SaaS multi-tenant enterprise

---

# 2. Kiến trúc RBAC & Data Scope

## 2.1 Cấu trúc RBAC

```
Permission:
System -> Module -> Screen/Capability -> Action

Role:
Role -> Permission Set

Runtime Authorization:
Permission Grant
+ Tenant/Portal Boundary
+ Active SAKI-MOTO Link
+ User Data Scope
+ Object State Gate
+ Approval Route / Assignment
+ Segregation of Duty
+ Audit Requirement
```

Role chỉ là tập hợp permission. Role không tự quyết định user được xem bản ghi nào. Data Scope và Approval Route là lớp runtime/config riêng.

---

## 2.2 Cấu trúc Permission Key

```
system.module.screen.action
```

- `system`: `platform`, `moto`, `saki` trong 3 RBAC matrix hiện tại.
- Permission key áp dụng cho Platform/MOTO/SAKI internal users.
- Staff App không dùng Role/Permission Matrix động. Staff dùng fixed capability.

### Ví dụ

| Permission Key | Ý nghĩa |
| --- | --- |
| `moto.contract.contract_request_cancel.cancel` | MOTO hủy contract request khi state cho phép |
| `moto.billing.invoice.finalize` | MOTO chốt invoice trước khi submit nếu workflow cho phép |
| `saki.attendance.daily_attendance.approve` | SAKI duyệt chấm công hằng ngày theo route/assignment |
| `saki.attendance.attendance_approver.assign` | Gán/cấu hình người duyệt chấm công, không phải approve attendance |
| `platform.tenant.saki_linkage_metadata.edit` | Platform sửa metadata liên kết SAKI-MOTO, không sửa dữ liệu nghiệp vụ tenant |

---

# 3. Phân chia hệ thống

## 3.1 Platform Admin

Quản trị cấp Platform/SaaS provider.

### Chức năng

- Quản lý tenant lifecycle
- Quản lý metadata liên kết SAKI-MOTO
- Cấp feature/package cho tenant
- Quản lý SaaS billing và hợp đồng sử dụng hệ thống
- Quản lý global/platform master
- Monitoring, batch, backup, system operations
- Audit platform metadata/operation log

---

## 3.2 MOTO Tenant (派遣元)

Nghiệp vụ phía công ty phái cử/staffing agency.

### Chức năng

- Nhận và trả lời 派遣照会 từ SAKI đã liên kết
- Tạo/gửi/gia hạn/kết thúc contract theo workflow
- Quản lý Staff account tối thiểu thuộc MOTO
- Đối soát/chỉnh sửa/proxy attendance theo quyền
- Tạo, finalize, submit, cancel invoice theo state
- Quản lý master/settings tenant MOTO

---

## 3.3 SAKI Tenant (派遣先)

Nghiệp vụ phía khách hàng tiếp nhận phái cử.

### Chức năng

- Tạo 派遣照会 và chọn MOTO/quote
- Duyệt nội bộ request nếu workflow yêu cầu
- Xác nhận/review/ký hợp đồng
- Duyệt/reject daily attendance và period attendance
- Xem/duyệt/reject invoice từ MOTO
- Quản lý settings/master tenant SAKI
- Quản lý compliance evidence liên quan 36協定, 3年抵触日, Equal Pay

SAKI chỉ được thao tác dữ liệu tenant SAKI, MOTO đã active link và object thuộc data scope nội bộ.

---

## 3.4 Staff App

Ứng dụng cho Staff.

- Staff App không tham gia RBAC matrix như User nội bộ MOTO/SAKI.
- Staff không được gán Role, Custom Permission, hoặc Data Scope động.
- Quyền Staff là fixed business capability theo Staff account/staff code/contract.
- Staff App dùng fixed self-filter theo Staff account/staff code/contract; đây không phải `参照範囲` của e-staffing internal users.

### Chức năng

- Nhập attendance
- Xem shift
- Xem hợp đồng
- Quản lý profile

---

# 4. Action Permission

## 4.1 Danh sách Action

| Action | JP | Ý nghĩa |
| --- | --- | --- |
| view | 参照 | Xem dữ liệu/màn hình |
| create | 作成 | Tạo mới |
| edit | 修正 | Chỉnh sửa |
| delete | 削除 | Xóa |
| cancel | 取消 | Hủy nghiệp vụ |
| submit | 提出 | Submit workflow |
| approve | 承認 | Phê duyệt |
| reject | 却下 / 差戻し | Từ chối/Trả lại |
| finalize | 確定 | Chốt dữ liệu |
| upload | アップロード | Upload file |
| download | ダウンロード | Download file/PDF |
| export | エクスポート | Export CSV/Excel |
| import | インポート | Import dữ liệu |
| assign | 割当 | Gán user/quyền |
| issue | 発行 | Phát hành account |
| reset | 初期化 | Reset password |
| lock | ロック | Khóa account |
| unlock | ロック解除 | Mở khóa account |
| hide | 非表示 | Ẩn dữ liệu |
| restore | 復帰 | Khôi phục dữ liệu |
| execute | 実行 | Chạy batch/process |
| provide | 提供 | SAKI cung cấp "Thông tin đãi ngộ" (待遇情報提供) cho MOTO theo Luật phái cử |
| notify | 通知 | SAKI thông báo "Ngày giới hạn phái cử" (事業所単位抵触日通知) cho MOTO |
| proxy | 代理操作 / 代理申請 | Nghiệp vụ "Làm thay/Duyệt thay” |

---

# 5. Kiểm soát phạm vi dữ liệu

## 5.1 Các loại Scope

### Đối với SAKI

| Scope | JP | Ý nghĩa |
| --- | --- | --- |
| ASSIGNED_ONLY | 自分が担当の範囲のみ | Chỉ dữ liệu do user đó phụ trách (VD: 契約確認者, 勤怠承認者). |
| DEPARTMENT | 所属部署の範囲のみ | Dữ liệu thuộc phòng ban của user. |
| BRANCH | 所属事業所の範囲のみ | Dữ liệu thuộc chi nhánh của user. |
| BRANCH_OR_DEPARTMENT | 所属事業所もしくは所属部署の範囲 | Gom của 2 và 3 (Rất quan trọng, e-staffing có option này). |
| ALL | 全て | Toàn quyền xem. |
| CUSTOM_REFERENCE_SCOPE | ユーザーに任意で設定された範囲 | Tùy chỉnh (Map đích danh với một hoặc nhiều phòng ban/chi nhánh khác). |

### Đối với MOTO

Không dùng chung bảng này. Phạm vi của MOTO được xác định dựa trên Khách hàng giao dịch (取引先) và quyền phụ trách Staff. (Bạn có thể giữ lại khái niệm Data Scope cho MOTO ở mặt kỹ thuật Backend, nhưng trong tài liệu BA phải làm rõ sự khác biệt này).

---

## 5.2 Ví dụ Scope

### Ví dụ 1

```
Role: Attendance Approver
Action: approve
Scope: DEPARTMENT
```

User chỉ approve attendance trong phòng ban được assign.

---

### Ví dụ 2

```
Role: Sales User
Action: view
Scope: ASSIGNED_ONLY
```

User chỉ xem contract/customer được assign.

---

# 6. Matrix Role của Platform

| Module | Màn hình | Super Admin | Operator | Billing Admin | Security Admin | Support |
| --- | --- | --- | --- | --- | --- | --- |
| Tenant | Danh sách tenant | ○ | ○ | △ | △ | ○ |
| Tenant | Tạo tenant | ○ | ○ | × | × | × |
| Tenant | Dừng tenant | ○ | △ | × | ○ | × |
| Tenant | Khôi phục tenant | ○ | △ | × | ○ | × |
| Contract | Hợp đồng tenant | ○ | ○ | ○ | × | △ |
| User | Danh sách user admin | ○ | ○ | × | ○ | △ |
| User | Reset password | ○ | △ | × | ○ | ○ |
| Security | API Key | ○ | × | × | ○ | × |
| Security | IP Restriction | ○ | × | × | ○ | × |
| Security | SSO Setting | ○ | × | × | ○ | × |
| Master | Global Master | ○ | △ | × | × | × |
| Audit | Audit Log | ○ | △ | △ | ○ | △ |
| Monitoring | System Monitoring | ○ | ○ | × | ○ | ○ |

---

# 7. Matrix Role của MOTO

| Module | Màn hình | Admin | Sales | Contract | Attendance | Billing | Readonly |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 派遣照会 | Danh sách/Chi tiết | ○ | ○ | ○ | △ | △ | ○ |
| 派遣照会 | Trả lời/Báo giá | ○ | ○ | △ | × | × | × |
| Contract | Danh sách hợp đồng | ○ | ○ | ○ | △ | △ | ○ |
| Contract | Tạo hợp đồng | ○ | △ | ○ | × | × | × |
| Contract | Sửa hợp đồng | ○ | △ | ○ | × | × | × |
| Contract | Gia hạn hợp đồng | ○ | △ | ○ | × | × | × |
| Contract | Kết thúc hợp đồng | ○ | △ | ○ | × | × | × |
| Staff | Danh sách staff | ○ | ○ | ○ | ○ | △ | ○ |
| Staff | Cấp account | ○ | × | ○ | × | × | × |
| Attendance | Danh sách attendance | ○ | △ | △ | ○ | △ | ○ |
| Attendance | Proxy attendance | ○ | × | × | ○ | × | × |
| Attendance | Attendance closing | ○ | × | × | ○ | × | × |
| Attendance | Attendance download | ○ | △ | △ | ○ | △ | ○ |
| Billing | Danh sách invoice | ○ | △ | △ | △ | ○ | ○ |
| Billing | Tạo invoice | ○ | × | × | × | ○ | × |
| Billing | Confirm invoice | ○ | × | × | × | ○ | × |
| Billing | Submit invoice | ○ | × | × | × | ○ | × |
| Billing | Cancel invoice | ○ | × | × | × | ○ | × |
| Master | Company Setting | ○ | × | × | × | × | × |
| User | User Management | ○ | × | × | × | × | × |
| Audit | Operation History | ○ | △ | △ | △ | △ | ○ |

---

# 8. Matrix Role của SAKI

| Module | Màn hình | Admin | Request User | Contract Approver | Attendance Approver | Billing Viewer | Readonly |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 派遣照会 | Tạo request | ○ | ○ | △ | × | × | × |
| 派遣照会 | So sánh báo giá | ○ | ○ | △ | × | × | ○ |
| 派遣照会 | Chọn MOTO | ○ | ○ | ○ | × | × | × |
| Contract | Danh sách hợp đồng | ○ | ○ | ○ | △ | △ | ○ |
| Contract | Approve hợp đồng | ○ | × | ○ | × | × | × |
| Contract | Reject hợp đồng | ○ | × | ○ | × | × | × |
| Attendance | Danh sách attendance | ○ | △ | △ | ○ | × | ○ |
| Attendance | Approve attendance | ○ | × | × | ○ | × | × |
| Attendance | Reject attendance | ○ | × | × | ○ | × | × |
| Attendance | Closing approve | ○ | × | × | ○ | × | × |
| Billing | Danh sách invoice | ○ | × | △ | △ | ○ | ○ |
| Billing | Confirm invoice | ○ | × | × | × | ○ | ○ |
| Billing | PDF download | ○ | × | △ | △ | ○ | ○ |
| User | User Management | ○ | × | × | × | × | × |
| Master | Department Setting | ○ | × | × | △ | × | × |
| Audit | Operation History | ○ | △ | △ | △ | △ | ○ |

---

# 9. Matrix của Staff App

| Module | Màn hình | Staff User |
| --- | --- | --- |
| Profile | Hồ sơ cá nhân | ○ |
| Attendance | Nhập attendance | ○ |
| Attendance | Danh sách attendance | ○ |
| Attendance | Xem shift | ○ |
| Contract | Xem hợp đồng | ○ |
| Notification | Danh sách thông báo | ○ |
| Account | Đổi password | ○ |

---

# 10. Thiết kế UI Permission Matrix

## 10.1 Flow tạo Role

```
Thông tin Role
↓
Chọn hệ thống / tenant type
↓
Permission Matrix
↓
Review dependency / high-risk permission warning
↓
Xác nhận
↓
Lưu role
```

---

## 10.2 Layout Permission Matrix

| Module | Screen | View | Create | Edit | Delete | Submit | Approve | Reject | Finalize | Cancel | Upload | Download | Export |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Contract | Contract List | ☑ |  |  |  |  |  |  |  |  |  | ☑ | ☑ |
| Contract | Contract Create | ☑ | ☑ | ☑ |  | ☑ |  |  |  | ☑ |  |  |  |
| Attendance | Attendance List | ☑ |  |  |  |  |  |  |  |  |  | ☑ | ☑ |
| Attendance | Attendance Approve | ☑ |  |  |  |  | ☑ | ☑ |  |  |  |  |  |
| Billing | Invoice Create | ☑ | ☑ | ☑ |  |  |  |  | ☑ | ☑ |  |  |  |
| Billing | Invoice Submit | ☑ |  |  |  | ☑ |  |  |  | ☑ |  | ☑ |  |

### 10.3 Layout thiết lập Scope dữ liệu

#### データ範囲設定レイアウト

---

```
------------------------------------------------
[ Data Scope Setting ]
------------------------------------------------

○ ALL
○ BRANCH
○ DEPARTMENT
○ ASSIGNED_ONLY
○ SELF_ONLY

------------------------------------------------
Conditional Scope Area
------------------------------------------------

[Division Selection]
☑ Shinjuku
☐ Umeda
☑ FUKUOKA

[Assigned Target]
☑ Assigned Contracts Only
☑ Assigned Staff Only

------------------------------------------------
Preview Scope Summary
------------------------------------------------

This role can:
- view contracts in OSAKA/TOKYO
- approve attendance in Billing dept
- access assigned staff only

------------------------------------------------
```

---

# Các field đề xuất

| Field | Type | Description |
| --- | --- | --- |
| Scope Type | radio | loại scope |
| Branch List | checkbox multi | chi nhánh |
| Department List | checkbox multi | phòng ban |
| Assigned Only Flag | checkbox | chỉ assigned |
| Cross Department | checkbox | cho phép cross dept |
| Cross Branch | checkbox | cho phép cross branch |
| Personal Info Restriction | checkbox | hạn chế PII |
| Effective Date | datetime | ngày hiệu lực |
| Expire Date | datetime | ngày hết hạn |

### Layout Approval

| Flag | JP | Ý nghĩa |
| --- | --- | --- |
| can_approve_contract | 契約確認の承認を許可 | Approve contract |
| can_reject_contract | 契約確認の却下を許可 | Reject contract |
| can_approve_daily_attendance | 勤怠日々承認を許可 | Approve daily attendance |
| can_reject_daily_attendance | 勤怠日々差戻しを許可 | Return/reject daily attendance |
| can_approve_attendance_closing | 勤怠締め承認を許可 | Approve monthly closing |
| can_reopen_attendance_closing | 勤怠締め解除を許可 | Reopen attendance closing |
| can_approve_master_change | マスタ変更承認を許可 | Approve master changes |
| can_approve_36_agreement | 36協定・特別条項申請の承認を許可 | Approve 36 agreement |
| can_finalize_invoice | 請求確定を許可 | Finalize invoice |
| can_submit_invoice | 請求提出を許可 | Submit invoice |
| can_cancel_invoice_submission | 請求提出取消を許可 | Cancel submitted invoice |
| can_approve_overtime | 残業申請承認を許可 | Approve overtime |
| can_approve_holiday_work | 休日出勤承認を許可 | Approve holiday work |
| can_approve_expense | 経費承認を許可 | Approve expense/reimbursement |
| can_unlock_account | アカウントロック解除を許可 | Unlock user account |
| can_reset_password | パスワード初期化を許可 | Reset password |
| can_execute_batch | バッチ実行を許可 | Execute batch/import |
| can_restore_deleted_data | 削除データ復旧を許可 | Restore deleted/cancelled data |

---

# Scope logic

## ALL

```
Xem toàn tenant
```

---

## BRANCH

```
Chỉ xem dữ liệu branch được chọn
```

---

## DEPARTMENT

```
Chỉ xem dữ liệu phòng ban được chọn
```

---

## ASSIGNED_ONLY

```
Chỉ dữ liệu được assign
```

Ví dụ:

- sales chỉ xem contract assigned
- approver chỉ approve staff assigned

---

## SELF_ONLY

```
Chỉ dữ liệu chính mình
```

---

# Rule Engine đề xuất

| Rule | Description |
| --- | --- |
| Role Scope Priority | SELF < ASSIGNED < DEPARTMENT < BRANCH < ALL |
| Deny Cross Tenant | bắt buộc |
| Deny Undefined Scope | default deny |
| Scope Required for Approve | approve bắt buộc có scope |
| Audit Required | thay đổi scope phải log |

---

# 11. Rule phụ thuộc Permission

| Action được chọn | Permission bắt buộc |
| --- | --- |
| create | view |
| edit | view |
| delete | view |
| submit | view |
| approve | view |
| reject | view |
| finalize | view |
| download | view |
| export | view |
| upload | view |
| assign | view |

---

# 12. Security Rules

| Rule | Nội dung |
| --- | --- |
| Tenant Isolation | Không được access cross-tenant |
| Backend Enforcement | API bắt buộc validate permission |
| Audit Mandatory | Mọi thay đổi quyền phải lưu log |
| System Role Protected | Không được xóa role hệ thống |
| Approval Separation | Tách create/edit với approve |
| Least Privilege | Chỉ cấp quyền tối thiểu cần thiết |
| MFA Required | Admin bắt buộc MFA |
| IP Restriction | Có thể cấu hình theo tenant |

---

# 13. Audit Requirement

## Các thao tác cần log

Operation

---

Tạo role

---

Sửa role

---

Gán permission

---

Gán role cho user

---

Đổi scope

---

Đổi quyền approve

---

---

## Audit Log Fields

Field

---

Operator

---

Tenant

---

Timestamp

---

Before value

---

After value

---

IP Address

---

Device/User Agent

---

---

# 14. Thiết kế Database

## Các table chính

| Table | Ý nghĩa |
| --- | --- |
| roles | Master role |
| permissions | Master permission |
| role_permissions | Mapping role-permission |
| user_roles | Mapping user-role |
| data_scopes | Master scope |
| role_data_scopes | Mapping role-scope |
| approval_authorities | Quyền approve |
| permission_dependencies | Rule phụ thuộc action |
| permission_audit_logs | Audit history |

---

# Build Full matrix permission

```markdown
system.module.screen.action
```

## 1. System Code chuẩn

| System | System Code |
| --- | --- |
| Platform SaaS Admin | `platform` |
| MOTO Portal / 派遣元ポータル | `moto` |
| SAKI Portal / 派遣先ポータル | `saki` |
| Staff Mobile App / スタッフアプリ | `staff` |

### 3. Module Code chuẩn

| Module | Module Code |
| --- | --- |
| Common / Authentication | `common` |
| Tenant Management | `tenant` |
| Request Management / 派遣照会 | `request` |
| Contract Management | `contract` |
| Staff Management | `staff` |
| Attendance Management | `attendance` |
| Billing Management | `billing` |
| Company Settings | `settings` |
| Batch Processing | `batch` |
| Global Master Management | `global_master` |
| System Management | `system` |
| User Management | `user` |
| Dashboard | `dashboard` |

### 4. Action

| Action | JP | Ý nghĩa |
| --- | --- | --- |
| view | 参照 | Xem màn hình / dữ liệu |
| create | 作成 | Tạo mới |
| edit | 修正 | Chỉnh sửa |
| delete | 削除 | Xóa |
| cancel | 取消 | Hủy nghiệp vụ |
| submit | 提出 | Submit workflow |
| approve | 承認 | Phê duyệt |
| reject | 却下 / 差戻し | Từ chối / trả lại |
| finalize | 確定 | Chốt dữ liệu |
| upload | アップロード | Upload file |
| download | ダウンロード | Download file/PDF |
| export | エクスポート | Export CSV/Excel |
| import | インポート | Import dữ liệu |
| assign | 割当 | Gán user/quyền |
| issue | 発行 | Phát hành account |
| reset | 初期化 | Reset password |
| lock | ロック | Khóa account |
| unlock | ロック解除 | Mở khóa account |
| hide | 非表示 | Ẩn dữ liệu |
| restore | 復帰 | Khôi phục dữ liệu |
| execute | 実行 | Chạy process/batch |
| remind | 督促 / 連絡 | Gửi nhắc nhở |
| provide | 提供 | SAKI cung cấp "Thông tin đãi ngộ" (待遇情報提供) cho MOTO theo Luật phái cử |
| notify | 通知 | SAKI thông báo "Ngày giới hạn phái cử" (事業所単位抵触日通知) cho MOTO |
| proxy | 代理操作 / 代理申請 | Nghiệp vụ "Làm thay/Duyệt thay” |# Hệ thống Carima Staffing

---

# 1. Tổng quan

## 1.1 Mục đích

Tài liệu này định nghĩa kiến trúc RBAC (Role Based Access Control), Data Scope và Approval Runtime Gate cho hệ thống Carima Staffing.

Mục tiêu:

- Kiểm soát truy cập theo Role cho Platform/SAKI/MOTO internal users
- Phân tách quyền theo portal, tenant và actor
- Giới hạn phạm vi dữ liệu bằng runtime data scope
- Kiểm soát quyền phê duyệt bằng approval route/assignment
- Đáp ứng yêu cầu audit/compliance
- Ngăn thao tác trái phép hoặc vượt trạng thái nghiệp vụ
- Hỗ trợ kiến trúc SaaS multi-tenant enterprise

---

# 2. Kiến trúc RBAC & Data Scope

## 2.1 Cấu trúc RBAC

```
Permission:
System -> Module -> Screen/Capability -> Action

Role:
Role -> Permission Set

Runtime Authorization:
Permission Grant
+ Tenant/Portal Boundary
+ Active SAKI-MOTO Link
+ User Data Scope
+ Object State Gate
+ Approval Route / Assignment
+ Segregation of Duty
+ Audit Requirement
```

Role chỉ là tập hợp permission. Role không tự quyết định user được xem bản ghi nào. Data Scope và Approval Route là lớp runtime/config riêng.

---

## 2.2 Cấu trúc Permission Key

```
system.module.screen.action
```

- `system`: `platform`, `moto`, `saki` trong 3 RBAC matrix hiện tại.
- Permission key áp dụng cho Platform/MOTO/SAKI internal users.
- Staff App không dùng Role/Permission Matrix động. Staff dùng fixed capability.

### Ví dụ

| Permission Key | Ý nghĩa |
| --- | --- |
| `moto.contract.contract_request_cancel.cancel` | MOTO hủy contract request khi state cho phép |
| `moto.billing.invoice.finalize` | MOTO chốt invoice trước khi submit nếu workflow cho phép |
| `saki.attendance.daily_attendance.approve` | SAKI duyệt chấm công hằng ngày theo route/assignment |
| `saki.attendance.attendance_approver.assign` | Gán/cấu hình người duyệt chấm công, không phải approve attendance |
| `platform.tenant.saki_linkage_metadata.edit` | Platform sửa metadata liên kết SAKI-MOTO, không sửa dữ liệu nghiệp vụ tenant |

---

# 3. Phân chia hệ thống

## 3.1 Platform Admin

Quản trị cấp Platform/SaaS provider.

### Chức năng

- Quản lý tenant lifecycle
- Quản lý metadata liên kết SAKI-MOTO
- Cấp feature/package cho tenant
- Quản lý SaaS billing và hợp đồng sử dụng hệ thống
- Quản lý global/platform master
- Monitoring, batch, backup, system operations
- Audit platform metadata/operation log

---

## 3.2 MOTO Tenant (派遣元)

Nghiệp vụ phía công ty phái cử/staffing agency.

### Chức năng

- Nhận và trả lời 派遣照会 từ SAKI đã liên kết
- Tạo/gửi/gia hạn/kết thúc contract theo workflow
- Quản lý Staff account tối thiểu thuộc MOTO
- Đối soát/chỉnh sửa/proxy attendance theo quyền
- Tạo, finalize, submit, cancel invoice theo state
- Quản lý master/settings tenant MOTO

---

## 3.3 SAKI Tenant (派遣先)

Nghiệp vụ phía khách hàng tiếp nhận phái cử.

### Chức năng

- Tạo 派遣照会 và chọn MOTO/quote
- Duyệt nội bộ request nếu workflow yêu cầu
- Xác nhận/review/ký hợp đồng
- Duyệt/reject daily attendance và period attendance
- Xem/duyệt/reject invoice từ MOTO
- Quản lý settings/master tenant SAKI
- Quản lý compliance evidence liên quan 36協定, 3年抵触日, Equal Pay

SAKI chỉ được thao tác dữ liệu tenant SAKI, MOTO đã active link và object thuộc data scope nội bộ.

---

## 3.4 Staff App

Ứng dụng cho Staff.

- Staff App không tham gia RBAC matrix như User nội bộ MOTO/SAKI.
- Staff không được gán Role, Custom Permission, hoặc Data Scope động.
- Quyền Staff là fixed business capability theo Staff account/staff code/contract.
- Staff App dùng fixed self-filter theo Staff account/staff code/contract; đây không phải `参照範囲` của e-staffing internal users.

### Chức năng

- Nhập attendance
- Xem shift
- Xem hợp đồng
- Quản lý profile

---

# 4. Action Permission

## 4.1 Danh sách Action

| Action | JP | Ý nghĩa |
| --- | --- | --- |
| view | 参照 | Xem dữ liệu/màn hình |
| create | 作成 | Tạo mới |
| edit | 修正 | Chỉnh sửa |
| delete | 削除 | Xóa |
| cancel | 取消 | Hủy nghiệp vụ |
| submit | 提出 | Submit workflow |
| approve | 承認 | Phê duyệt |
| reject | 却下 / 差戻し | Từ chối/Trả lại |
| finalize | 確定 | Chốt dữ liệu |
| upload | アップロード | Upload file |
| download | ダウンロード | Download file/PDF |
| export | エクスポート | Export CSV/Excel |
| import | インポート | Import dữ liệu |
| assign | 割当 | Gán user/quyền |
| issue | 発行 | Phát hành account |
| reset | 初期化 | Reset password |
| lock | ロック | Khóa account |
| unlock | ロック解除 | Mở khóa account |
| hide | 非表示 | Ẩn dữ liệu |
| restore | 復帰 | Khôi phục dữ liệu |
| execute | 実行 | Chạy batch/process |
| provide | 提供 | SAKI cung cấp "Thông tin đãi ngộ" (待遇情報提供) cho MOTO theo Luật phái cử |
| notify | 通知 | SAKI thông báo "Ngày giới hạn phái cử" (事業所単位抵触日通知) cho MOTO |
| proxy | 代理操作 / 代理申請 | Nghiệp vụ "Làm thay/Duyệt thay” |

---

# 5. Kiểm soát phạm vi dữ liệu

## 5.1 Các loại Scope

### Đối với SAKI

| Scope | JP | Ý nghĩa |
| --- | --- | --- |
| ASSIGNED_ONLY | 自分が担当の範囲のみ | Chỉ dữ liệu do user đó phụ trách (VD: 契約確認者, 勤怠承認者). |
| DEPARTMENT | 所属部署の範囲のみ | Dữ liệu thuộc phòng ban của user. |
| BRANCH | 所属事業所の範囲のみ | Dữ liệu thuộc chi nhánh của user. |
| BRANCH_OR_DEPARTMENT | 所属事業所もしくは所属部署の範囲 | Gom của 2 và 3 (Rất quan trọng, e-staffing có option này). |
| ALL | 全て | Toàn quyền xem. |
| CUSTOM_REFERENCE_SCOPE | ユーザーに任意で設定された範囲 | Tùy chỉnh (Map đích danh với một hoặc nhiều phòng ban/chi nhánh khác). |

### Đối với MOTO

Không dùng chung bảng này. Phạm vi của MOTO được xác định dựa trên Khách hàng giao dịch (取引先) và quyền phụ trách Staff. (Bạn có thể giữ lại khái niệm Data Scope cho MOTO ở mặt kỹ thuật Backend, nhưng trong tài liệu BA phải làm rõ sự khác biệt này).

---

## 5.2 Ví dụ Scope

### Ví dụ 1

```
Role: Attendance Approver
Action: approve
Scope: DEPARTMENT
```

User chỉ approve attendance trong phòng ban được assign.

---

### Ví dụ 2

```
Role: Sales User
Action: view
Scope: ASSIGNED_ONLY
```

User chỉ xem contract/customer được assign.

---

# 6. Matrix Role của Platform

| Module | Màn hình | Super Admin | Operator | Billing Admin | Security Admin | Support |
| --- | --- | --- | --- | --- | --- | --- |
| Tenant | Danh sách tenant | ○ | ○ | △ | △ | ○ |
| Tenant | Tạo tenant | ○ | ○ | × | × | × |
| Tenant | Dừng tenant | ○ | △ | × | ○ | × |
| Tenant | Khôi phục tenant | ○ | △ | × | ○ | × |
| Contract | Hợp đồng tenant | ○ | ○ | ○ | × | △ |
| User | Danh sách user admin | ○ | ○ | × | ○ | △ |
| User | Reset password | ○ | △ | × | ○ | ○ |
| Security | API Key | ○ | × | × | ○ | × |
| Security | IP Restriction | ○ | × | × | ○ | × |
| Security | SSO Setting | ○ | × | × | ○ | × |
| Master | Global Master | ○ | △ | × | × | × |
| Audit | Audit Log | ○ | △ | △ | ○ | △ |
| Monitoring | System Monitoring | ○ | ○ | × | ○ | ○ |

---

# 7. Matrix Role của MOTO

| Module | Màn hình | Admin | Sales | Contract | Attendance | Billing | Readonly |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 派遣照会 | Danh sách/Chi tiết | ○ | ○ | ○ | △ | △ | ○ |
| 派遣照会 | Trả lời/Báo giá | ○ | ○ | △ | × | × | × |
| Contract | Danh sách hợp đồng | ○ | ○ | ○ | △ | △ | ○ |
| Contract | Tạo hợp đồng | ○ | △ | ○ | × | × | × |
| Contract | Sửa hợp đồng | ○ | △ | ○ | × | × | × |
| Contract | Gia hạn hợp đồng | ○ | △ | ○ | × | × | × |
| Contract | Kết thúc hợp đồng | ○ | △ | ○ | × | × | × |
| Staff | Danh sách staff | ○ | ○ | ○ | ○ | △ | ○ |
| Staff | Cấp account | ○ | × | ○ | × | × | × |
| Attendance | Danh sách attendance | ○ | △ | △ | ○ | △ | ○ |
| Attendance | Proxy attendance | ○ | × | × | ○ | × | × |
| Attendance | Attendance closing | ○ | × | × | ○ | × | × |
| Attendance | Attendance download | ○ | △ | △ | ○ | △ | ○ |
| Billing | Danh sách invoice | ○ | △ | △ | △ | ○ | ○ |
| Billing | Tạo invoice | ○ | × | × | × | ○ | × |
| Billing | Confirm invoice | ○ | × | × | × | ○ | × |
| Billing | Submit invoice | ○ | × | × | × | ○ | × |
| Billing | Cancel invoice | ○ | × | × | × | ○ | × |
| Master | Company Setting | ○ | × | × | × | × | × |
| User | User Management | ○ | × | × | × | × | × |
| Audit | Operation History | ○ | △ | △ | △ | △ | ○ |

---

# 8. Matrix Role của SAKI

| Module | Màn hình | Admin | Request User | Contract Approver | Attendance Approver | Billing Viewer | Readonly |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 派遣照会 | Tạo request | ○ | ○ | △ | × | × | × |
| 派遣照会 | So sánh báo giá | ○ | ○ | △ | × | × | ○ |
| 派遣照会 | Chọn MOTO | ○ | ○ | ○ | × | × | × |
| Contract | Danh sách hợp đồng | ○ | ○ | ○ | △ | △ | ○ |
| Contract | Approve hợp đồng | ○ | × | ○ | × | × | × |
| Contract | Reject hợp đồng | ○ | × | ○ | × | × | × |
| Attendance | Danh sách attendance | ○ | △ | △ | ○ | × | ○ |
| Attendance | Approve attendance | ○ | × | × | ○ | × | × |
| Attendance | Reject attendance | ○ | × | × | ○ | × | × |
| Attendance | Closing approve | ○ | × | × | ○ | × | × |
| Billing | Danh sách invoice | ○ | × | △ | △ | ○ | ○ |
| Billing | Confirm invoice | ○ | × | × | × | ○ | ○ |
| Billing | PDF download | ○ | × | △ | △ | ○ | ○ |
| User | User Management | ○ | × | × | × | × | × |
| Master | Department Setting | ○ | × | × | △ | × | × |
| Audit | Operation History | ○ | △ | △ | △ | △ | ○ |

---

# 9. Matrix của Staff App

| Module | Màn hình | Staff User |
| --- | --- | --- |
| Profile | Hồ sơ cá nhân | ○ |
| Attendance | Nhập attendance | ○ |
| Attendance | Danh sách attendance | ○ |
| Attendance | Xem shift | ○ |
| Contract | Xem hợp đồng | ○ |
| Notification | Danh sách thông báo | ○ |
| Account | Đổi password | ○ |

---

# 10. Thiết kế UI Permission Matrix

## 10.1 Flow tạo Role

```
Thông tin Role
↓
Chọn hệ thống / tenant type
↓
Permission Matrix
↓
Review dependency / high-risk permission warning
↓
Xác nhận
↓
Lưu role
```

---

## 10.2 Layout Permission Matrix

| Module | Screen | View | Create | Edit | Delete | Submit | Approve | Reject | Finalize | Cancel | Upload | Download | Export |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Contract | Contract List | ☑ |  |  |  |  |  |  |  |  |  | ☑ | ☑ |
| Contract | Contract Create | ☑ | ☑ | ☑ |  | ☑ |  |  |  | ☑ |  |  |  |
| Attendance | Attendance List | ☑ |  |  |  |  |  |  |  |  |  | ☑ | ☑ |
| Attendance | Attendance Approve | ☑ |  |  |  |  | ☑ | ☑ |  |  |  |  |  |
| Billing | Invoice Create | ☑ | ☑ | ☑ |  |  |  |  | ☑ | ☑ |  |  |  |
| Billing | Invoice Submit | ☑ |  |  |  | ☑ |  |  |  | ☑ |  | ☑ |  |

### 10.3 Layout thiết lập Scope dữ liệu

#### データ範囲設定レイアウト

---

```
------------------------------------------------
[ Data Scope Setting ]
------------------------------------------------

○ ALL
○ BRANCH
○ DEPARTMENT
○ ASSIGNED_ONLY
○ SELF_ONLY

------------------------------------------------
Conditional Scope Area
------------------------------------------------

[Division Selection]
☑ Shinjuku
☐ Umeda
☑ FUKUOKA

[Assigned Target]
☑ Assigned Contracts Only
☑ Assigned Staff Only

------------------------------------------------
Preview Scope Summary
------------------------------------------------

This role can:
- view contracts in OSAKA/TOKYO
- approve attendance in Billing dept
- access assigned staff only

------------------------------------------------
```

---

# Các field đề xuất

| Field | Type | Description |
| --- | --- | --- |
| Scope Type | radio | loại scope |
| Branch List | checkbox multi | chi nhánh |
| Department List | checkbox multi | phòng ban |
| Assigned Only Flag | checkbox | chỉ assigned |
| Cross Department | checkbox | cho phép cross dept |
| Cross Branch | checkbox | cho phép cross branch |
| Personal Info Restriction | checkbox | hạn chế PII |
| Effective Date | datetime | ngày hiệu lực |
| Expire Date | datetime | ngày hết hạn |

### Layout Approval

| Flag | JP | Ý nghĩa |
| --- | --- | --- |
| can_approve_contract | 契約確認の承認を許可 | Approve contract |
| can_reject_contract | 契約確認の却下を許可 | Reject contract |
| can_approve_daily_attendance | 勤怠日々承認を許可 | Approve daily attendance |
| can_reject_daily_attendance | 勤怠日々差戻しを許可 | Return/reject daily attendance |
| can_approve_attendance_closing | 勤怠締め承認を許可 | Approve monthly closing |
| can_reopen_attendance_closing | 勤怠締め解除を許可 | Reopen attendance closing |
| can_approve_master_change | マスタ変更承認を許可 | Approve master changes |
| can_approve_36_agreement | 36協定・特別条項申請の承認を許可 | Approve 36 agreement |
| can_finalize_invoice | 請求確定を許可 | Finalize invoice |
| can_submit_invoice | 請求提出を許可 | Submit invoice |
| can_cancel_invoice_submission | 請求提出取消を許可 | Cancel submitted invoice |
| can_approve_overtime | 残業申請承認を許可 | Approve overtime |
| can_approve_holiday_work | 休日出勤承認を許可 | Approve holiday work |
| can_approve_expense | 経費承認を許可 | Approve expense/reimbursement |
| can_unlock_account | アカウントロック解除を許可 | Unlock user account |
| can_reset_password | パスワード初期化を許可 | Reset password |
| can_execute_batch | バッチ実行を許可 | Execute batch/import |
| can_restore_deleted_data | 削除データ復旧を許可 | Restore deleted/cancelled data |

---

# Scope logic

## ALL

```
Xem toàn tenant
```

---

## BRANCH

```
Chỉ xem dữ liệu branch được chọn
```

---

## DEPARTMENT

```
Chỉ xem dữ liệu phòng ban được chọn
```

---

## ASSIGNED_ONLY

```
Chỉ dữ liệu được assign
```

Ví dụ:

- sales chỉ xem contract assigned
- approver chỉ approve staff assigned

---

## SELF_ONLY

```
Chỉ dữ liệu chính mình
```

---

# Rule Engine đề xuất

| Rule | Description |
| --- | --- |
| Role Scope Priority | SELF < ASSIGNED < DEPARTMENT < BRANCH < ALL |
| Deny Cross Tenant | bắt buộc |
| Deny Undefined Scope | default deny |
| Scope Required for Approve | approve bắt buộc có scope |
| Audit Required | thay đổi scope phải log |

---

# 11. Rule phụ thuộc Permission

| Action được chọn | Permission bắt buộc |
| --- | --- |
| create | view |
| edit | view |
| delete | view |
| submit | view |
| approve | view |
| reject | view |
| finalize | view |
| download | view |
| export | view |
| upload | view |
| assign | view |

---

# 12. Security Rules

| Rule | Nội dung |
| --- | --- |
| Tenant Isolation | Không được access cross-tenant |
| Backend Enforcement | API bắt buộc validate permission |
| Audit Mandatory | Mọi thay đổi quyền phải lưu log |
| System Role Protected | Không được xóa role hệ thống |
| Approval Separation | Tách create/edit với approve |
| Least Privilege | Chỉ cấp quyền tối thiểu cần thiết |
| MFA Required | Admin bắt buộc MFA |
| IP Restriction | Có thể cấu hình theo tenant |

---

# 13. Audit Requirement

## Các thao tác cần log

Operation

---

Tạo role

---

Sửa role

---

Gán permission

---

Gán role cho user

---

Đổi scope

---

Đổi quyền approve

---

---

## Audit Log Fields

Field

---

Operator

---

Tenant

---

Timestamp

---

Before value

---

After value

---

IP Address

---

Device/User Agent

---

---

# 14. Thiết kế Database

## Các table chính

| Table | Ý nghĩa |
| --- | --- |
| roles | Master role |
| permissions | Master permission |
| role_permissions | Mapping role-permission |
| user_roles | Mapping user-role |
| data_scopes | Master scope |
| role_data_scopes | Mapping role-scope |
| approval_authorities | Quyền approve |
| permission_dependencies | Rule phụ thuộc action |
| permission_audit_logs | Audit history |

---

# Build Full matrix permission

```markdown
system.module.screen.action
```

## 1. System Code chuẩn

| System | System Code |
| --- | --- |
| Platform SaaS Admin | `platform` |
| MOTO Portal / 派遣元ポータル | `moto` |
| SAKI Portal / 派遣先ポータル | `saki` |
| Staff Mobile App / スタッフアプリ | `staff` |

### 3. Module Code chuẩn

| Module | Module Code |
| --- | --- |
| Common / Authentication | `common` |
| Tenant Management | `tenant` |
| Request Management / 派遣照会 | `request` |
| Contract Management | `contract` |
| Staff Management | `staff` |
| Attendance Management | `attendance` |
| Billing Management | `billing` |
| Company Settings | `settings` |
| Batch Processing | `batch` |
| Global Master Management | `global_master` |
| System Management | `system` |
| User Management | `user` |
| Dashboard | `dashboard` |

### 4. Action

| Action | JP | Ý nghĩa |
| --- | --- | --- |
| view | 参照 | Xem màn hình / dữ liệu |
| create | 作成 | Tạo mới |
| edit | 修正 | Chỉnh sửa |
| delete | 削除 | Xóa |
| cancel | 取消 | Hủy nghiệp vụ |
| submit | 提出 | Submit workflow |
| approve | 承認 | Phê duyệt |
| reject | 却下 / 差戻し | Từ chối / trả lại |
| finalize | 確定 | Chốt dữ liệu |
| upload | アップロード | Upload file |
| download | ダウンロード | Download file/PDF |
| export | エクスポート | Export CSV/Excel |
| import | インポート | Import dữ liệu |
| assign | 割当 | Gán user/quyền |
| issue | 発行 | Phát hành account |
| reset | 初期化 | Reset password |
| lock | ロック | Khóa account |
| unlock | ロック解除 | Mở khóa account |
| hide | 非表示 | Ẩn dữ liệu |
| restore | 復帰 | Khôi phục dữ liệu |
| execute | 実行 | Chạy process/batch |
| remind | 督促 / 連絡 | Gửi nhắc nhở |
| provide | 提供 | SAKI cung cấp "Thông tin đãi ngộ" (待遇情報提供) cho MOTO theo Luật phái cử |
| notify | 通知 | SAKI thông báo "Ngày giới hạn phái cử" (事業所単位抵触日通知) cho MOTO |
| proxy | 代理操作 / 代理申請 | Nghiệp vụ "Làm thay/Duyệt thay” |