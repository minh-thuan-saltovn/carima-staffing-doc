Screen 1 (Screen hoặc Popup / Modal)

# 1. UI/UX Layout

{image}

---

# 2. Định nghĩa Item

## 1. Thông tin cơ bản

| No | Item | Type | Required | Format | DB | Default | Điều kiện hiển thị |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Contract No | Label | No | Auto | contract_no |  |  |
| 2 | Contract Name | Textbox | Yes | 200 ký tự | contract_name |  |  |
| 3 | Client | Dropdown | Yes | FK | client_id |  |  |
| 4 | Staff | Dropdown | Yes | FK | staff_id |  |  |
| 5 | Contract Type | Dropdown | Yes | Enum | contract_type |  |  |

---

# 3. Validation

Reference Link

---

# 4. Event Definition (Screen)

## Cancel

### Trigger

Click Cancel

### Process

Hiển thị popup

```
Dữ liệu chưa lưu sẽ bị mất.
Bạn có muốn thoát không?
```

---

# 5. Event Definition (API)

## Initial Load

### Trigger

Mở màn hình

### Process

1. Call API <API ID>

---

# 6. API Mapping

## Create Contract

```
POST /api/contracts
```

Request

```json
{
  "contract_name": "ABC Contract",
  "client_id": 1,
  "staff_id": 10,
  "contract_type": "DISPATCH",
  "start_date": "2026-06-01",
  "end_date": "2026-12-31"
}
```

Response

```json
{
  "id": 1001,
  "contract_no": "CTR-202606-0001"
}
```

---

# 7. Notification

## Trigger

Create Contract Success

### Receiver

- Contract Manager
- Operation Manager

### Channel

- Email
- In-App Notification

---

# 8. Approval Flow

## Nếu Approval ON

```
Draft
 ↓
Pending Approval
 ↓
Approved
 ↓
Active
```