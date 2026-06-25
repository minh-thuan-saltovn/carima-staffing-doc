# Error Handling Definition

Bảng định nghĩa chi tiết các mã lỗi HTTP, mã lỗi hệ thống (Code) và thông báo hiển thị tương ứng (Message) được áp dụng trên toàn bộ hệ thống CARIMA Staffing.

| HTTP Code | Code | Message |
| --- | --- | --- |
| 400 | BAD_REQUEST | Yêu cầu không hợp lệ |
| 401 | UNAUTHORIZED | Phiên đăng nhập đã hết hạn |
| 403 | FORBIDDEN | Bạn không có quyền thực hiện thao tác này |
| 404 | NOT_FOUND | Không tìm thấy dữ liệu |
| 409 | CONFLICT | Dữ liệu đã tồn tại |
| 422 | VALIDATION_ERROR | Dữ liệu không hợp lệ |
| 429 | TOO_MANY_REQUESTS | Quá nhiều yêu cầu, vui lòng thử lại sau |
| 500 | INTERNAL_SERVER_ERROR | Hệ thống đang gặp sự cố |
| 503 | SERVICE_UNAVAILABLE | Hệ thống đang bảo trì |
