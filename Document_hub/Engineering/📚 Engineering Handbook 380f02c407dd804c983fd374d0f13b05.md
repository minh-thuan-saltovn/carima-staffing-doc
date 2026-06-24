## API Convention

## Mục đích

Đảm bảo API được thiết kế nhất quán, dễ hiểu, dễ bảo trì và dễ mở rộng.

---

## HTTP Methods

| Operation | Method |
| --- | --- |
| Query | GET |
| Create | POST |
| Update | PATCH |
| Delete | DELETE |
| Business Action | POST |

### Rules

- GET chỉ dùng để đọc dữ liệu.
- POST dùng cho Create hoặc Business Action.
- PATCH là method mặc định cho Update.
- DELETE dùng cho xóa dữ liệu.
- PUT chỉ sử dụng khi cần thay thế toàn bộ resource.

---

## Resource Naming

### Collection

```
GET /staffs
POST /staffs
```

### Single Resource

```
GET /staffs/{id}
PATCH /staffs/{id}
DELETE /staffs/{id}
```

### Nested Resource

```
GET /jobs/{id}/applications
GET /companies/{id}/staffs
```

### Rules

- Sử dụng danh từ số nhiều.
- Không sử dụng động từ trong URL.
- URL phải thể hiện Resource.

### Good

```
/staffs
/jobs
/requests
/companies
```

### Bad

```
/getStaffs
/createJob
/updateRequest
/deleteCompany
```

---

## Business Action

Sử dụng POST cho các hành động nghiệp vụ.

### Examples

```
POST /requests/{id}/approve

POST /requests/{id}/reject

POST /jobs/{id}/publish

POST /jobs/{id}/close

POST /attendance/check-in

POST /attendance/check-out

POST /users/{id}/reset-password
```

### Rules

- Action phải được đặt ở cuối URL.
- Không dùng PATCH hoặc PUT cho Business Action.

---

## Request Convention

Use raw JSON.

Good:

```json
{
  "full_name": "Nguyen Van A",
  "email": "staff@example.com"
}
```

Avoid:

```json
{
  "data": {
    "full_name": "Nguyen Van A",
    "email": "staff@example.com"
  }
}
```

---

## Response Convention

### Single Resource

```json
{
  "data": {
    "id": 1
  }
}
```

### Collection

```json
{
  "data": []
}
```

### Paginated Collection

```json
{
  "data": [],
  "links": {},
  "meta": {}
}
```

### Error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters."
  }
}
```

---

## HTTP Status Code

| Status | Meaning |
| --- | --- |
| 200 | Success |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Validation Error |
| 500 | Internal Server Error |

---

## Versioning

```
/api/v1/staffs
/api/v1/jobs
```

### Rules

- Version nằm trong URL.
- Chỉ tăng version khi breaking change.

---

## Naming Convention

### JSON Key

Sử dụng snake_case.

```json
{
  "first_name": "Carima Staffing",
  "created_at": "2026-06-15T10:00:00Z"
}
```

### Rules

- API Request và Response phải thống nhất format.
- Không trộn camelCase và snake_case.

---

## Summary

| Category | Standard |
| --- | --- |
| Query | GET |
| Create | POST |
| Update | PATCH |
| Delete | DELETE |
| Business Action | POST |
| Resource Name | Plural Noun |
| API Version | `/api/v1` |
| JSON Key | snake_case |
| Pagination | page, per_page |
| Sort | sort=name, sort=-name |
| Response | data, meta, error |