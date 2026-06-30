**Success Response - Dạng danh sách**

HTTP Status: 200 OK

```json
{
  "data": [
    {
      "id": 1
    }
  ],
  "meta": {
    "total": 10000,
    "page": 1,
    "limit": 20
  }
}
```

**Success Response - Dạng create**

HTTP Status: 201 Created

```json
{
  "data": {
    "contract_id": 7001,
    "title": "MO-CON-004",
    "current_step": 2
  }
}
```

**Success Response - Dạng update**

HTTP Status: 200 OK

```json

{
  "data": {
    "contract_id": 7001,
    "title": "MO-CON-004",
    "current_step": 2
  }
}
```