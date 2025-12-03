# HTTP, HTTPS, WWW - Kiến thức toàn diện về Web Protocols

## 📌 Mục lục

1. [Tổng quan về Internet và Web](#1-tổng-quan-về-internet-và-web)
2. [WWW là gì?](#2-www-là-gì)
3. [HTTP là gì?](#3-http-là-gì)
4. [HTTPS là gì?](#4-https-là-gì)
5. [HTTP Methods](#5-http-methods)
6. [HTTP Status Codes](#6-http-status-codes)
7. [HTTP Headers](#7-http-headers)
8. [HTTP Versions](#8-http-versions)
9. [URL và URI](#9-url-và-uri)
10. [DNS - Domain Name System](#10-dns---domain-name-system)
11. [SSL/TLS Certificates](#11-ssltls-certificates)
12. [Cookies và Sessions](#12-cookies-và-sessions)
13. [CORS](#13-cors---cross-origin-resource-sharing)
14. [Caching](#14-caching)
15. [Security Best Practices](#15-security-best-practices)

---

## 1. Tổng quan về Internet và Web

### Internet vs World Wide Web

Internet là một mạng lưới toàn cầu kết nối hàng tỷ thiết bị và mạng máy tính, cho phép trao đổi thông tin và dữ liệu. Nó hoạt động dựa trên các giao thức truyền tải dữ liệu chung như TCP/IP và cho phép người dùng truy cập thông tin, giao tiếp, mua sắm, học tập và giải trí trực tuyến. Internet có thể được coi là một hệ thống kết nối khổng lồ, cho phép mọi người kết nối với nhau bất kể khoảng cách địa lý.

| Khái niệm      | Internet                                | World Wide Web (WWW)             |
| -------------- | --------------------------------------- | -------------------------------- |
| **Định nghĩa** | Mạng lưới toàn cầu kết nối các máy tính | Hệ thống thông tin trên Internet |
| **Ra đời**     | 1969 (ARPANET)                          | 1989 (Tim Berners-Lee)           |
| **Bản chất**   | Hạ tầng vật lý (cables, routers)        | Dịch vụ chạy trên Internet       |
| **Protocols**  | TCP/IP, UDP, ICMP                       | HTTP, HTTPS                      |
| **Ví dụ**      | Email, FTP, VoIP, Web                   | Websites, Web apps               |

### Mô hình Client-Server

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLIENT-SERVER MODEL                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────┐         REQUEST          ┌──────────┐           │
│   │          │  ───────────────────────▶│          │           │
│   │  CLIENT  │     (HTTP Request)       │  SERVER  │           │
│   │ (Browser)│                          │  (Web)   │           │
│   │          │  ◀───────────────────────│          │           │
│   └──────────┘         RESPONSE         └──────────┘           │
│                     (HTTP Response)                             │
│                                                                 │
│   Examples:                              Examples:              │
│   • Chrome, Firefox                      • Apache, Nginx        │
│   • Mobile apps                          • Node.js, Express     │
│   • Postman, curl                        • AWS, Azure           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### TCP/IP Model

```
┌─────────────────────────────────────────┐
│           APPLICATION LAYER             │  ← HTTP, HTTPS, FTP, SMTP
├─────────────────────────────────────────┤
│            TRANSPORT LAYER              │  ← TCP, UDP
├─────────────────────────────────────────┤
│            INTERNET LAYER               │  ← IP, ICMP
├─────────────────────────────────────────┤
│         NETWORK ACCESS LAYER            │  ← Ethernet, Wi-Fi
└─────────────────────────────────────────┘
```

---

## 2. WWW là gì?

### Định nghĩa

**WWW** (World Wide Web) là hệ thống thông tin toàn cầu, nơi các tài liệu và tài nguyên được:

- Định danh bằng **URL** (Uniform Resource Locator)
- Liên kết với nhau qua **hyperlinks**
- Truy cập thông qua **Internet** bằng giao thức **HTTP/HTTPS**

### Lịch sử WWW

| Năm  | Sự kiện                                   |
| ---- | ----------------------------------------- |
| 1989 | Tim Berners-Lee đề xuất WWW tại CERN      |
| 1990 | Website đầu tiên ra đời                   |
| 1991 | WWW được công bố ra công chúng            |
| 1993 | Mosaic - trình duyệt đồ họa đầu tiên      |
| 1994 | W3C (World Wide Web Consortium) thành lập |

### Các thành phần của WWW

```
┌─────────────────────────────────────────────────────────────┐
│                    WORLD WIDE WEB                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │    HTML     │  │    URL      │  │    HTTP     │        │
│  │  (Content)  │  │ (Address)   │  │ (Protocol)  │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                             │
│  HTML: Ngôn ngữ đánh dấu để tạo nội dung web               │
│  URL:  Địa chỉ để định vị tài nguyên                       │
│  HTTP: Giao thức để truyền tải dữ liệu                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### www có bắt buộc không?

```
https://www.example.com  ←  Có www (subdomain)
https://example.com      ←  Không có www

Cả hai đều hoạt động! www chỉ là một subdomain truyền thống.
Ngày nay, nhiều website bỏ www để URL ngắn gọn hơn.
```

---

## 3. HTTP là gì?

### Định nghĩa

**HTTP** (HyperText Transfer Protocol) là giao thức truyền tải siêu văn bản, được sử dụng để truyền dữ liệu giữa client và server trên World Wide Web.

### Đặc điểm của HTTP

| Đặc điểm             | Mô tả                                               |
| -------------------- | --------------------------------------------------- |
| **Stateless**        | Mỗi request độc lập, server không nhớ request trước |
| **Text-based**       | Messages dạng text, dễ đọc                          |
| **Request-Response** | Client gửi request, server trả response             |
| **Port mặc định**    | 80                                                  |
| **Connectionless**   | Kết nối đóng sau mỗi request (HTTP/1.0)             |

### Cấu trúc HTTP Request

```
┌─────────────────────────────────────────────────────────────┐
│                     HTTP REQUEST                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ REQUEST LINE                                        │   │
│  │ GET /api/users HTTP/1.1                            │   │
│  │ ↑    ↑          ↑                                  │   │
│  │ Method  Path    Version                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ HEADERS                                             │   │
│  │ Host: api.example.com                              │   │
│  │ Content-Type: application/json                     │   │
│  │ Authorization: Bearer token123                     │   │
│  │ User-Agent: Mozilla/5.0                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ BODY (Optional)                                     │   │
│  │ {                                                   │   │
│  │   "name": "John",                                   │   │
│  │   "email": "john@example.com"                       │   │
│  │ }                                                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Cấu trúc HTTP Response

```
┌─────────────────────────────────────────────────────────────┐
│                     HTTP RESPONSE                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ STATUS LINE                                         │   │
│  │ HTTP/1.1 200 OK                                    │   │
│  │ ↑        ↑   ↑                                     │   │
│  │ Version Code Message                               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ HEADERS                                             │   │
│  │ Content-Type: application/json                     │   │
│  │ Content-Length: 256                                │   │
│  │ Cache-Control: max-age=3600                        │   │
│  │ Set-Cookie: session=abc123                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ BODY                                                │   │
│  │ {                                                   │   │
│  │   "id": 1,                                          │   │
│  │   "name": "John",                                   │   │
│  │   "email": "john@example.com"                       │   │
│  │ }                                                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Ví dụ HTTP Request/Response

```http
# REQUEST
GET /api/users/1 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9

# RESPONSE
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 89

{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "created_at": "2024-01-15"
}
```

---

## 4. HTTPS là gì?

### Định nghĩa

**HTTPS** (HyperText Transfer Protocol Secure) là phiên bản bảo mật của HTTP, sử dụng **SSL/TLS** để mã hóa dữ liệu truyền tải.

### HTTP vs HTTPS

| Tiêu chí        | HTTP               | HTTPS                |
| --------------- | ------------------ | -------------------- |
| **Bảo mật**     | ❌ Không mã hóa    | ✅ Mã hóa SSL/TLS    |
| **Port**        | 80                 | 443                  |
| **URL**         | `http://`          | `https://`           |
| **Certificate** | Không cần          | Cần SSL Certificate  |
| **SEO**         | Thấp hơn           | Google ưu tiên       |
| **Tốc độ**      | Nhanh hơn một chút | Chậm hơn (do mã hóa) |
| **Tin cậy**     | Không an toàn      | An toàn, có 🔒       |

### Cách HTTPS hoạt động

```
┌─────────────────────────────────────────────────────────────────┐
│                    HTTPS HANDSHAKE (TLS)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   CLIENT                                          SERVER        │
│     │                                               │           │
│     │  1. Client Hello                              │           │
│     │  ─────────────────────────────────────────▶  │           │
│     │  (Supported TLS versions, cipher suites)     │           │
│     │                                               │           │
│     │  2. Server Hello + Certificate                │           │
│     │  ◀─────────────────────────────────────────  │           │
│     │  (Chosen cipher, SSL certificate)            │           │
│     │                                               │           │
│     │  3. Client verifies certificate               │           │
│     │  (Check CA, expiry, domain)                  │           │
│     │                                               │           │
│     │  4. Key Exchange                              │           │
│     │  ─────────────────────────────────────────▶  │           │
│     │  (Pre-master secret encrypted with           │           │
│     │   server's public key)                       │           │
│     │                                               │           │
│     │  5. Both generate session keys                │           │
│     │                                               │           │
│     │  6. Encrypted Communication                   │           │
│     │  ◀════════════════════════════════════════▶  │           │
│     │  (All data encrypted with session key)       │           │
│     │                                               │           │
└─────────────────────────────────────────────────────────────────┘
```

### Các loại SSL Certificate

| Loại                             | Validation        | Thời gian | Giá           | Use Case              |
| -------------------------------- | ----------------- | --------- | ------------- | --------------------- |
| **DV** (Domain Validation)       | Chỉ domain        | Vài phút  | Miễn phí - Rẻ | Blog, website cá nhân |
| **OV** (Organization Validation) | Domain + Tổ chức  | 1-3 ngày  | Trung bình    | Business websites     |
| **EV** (Extended Validation)     | Kiểm tra kỹ lưỡng | 1-2 tuần  | Đắt           | Banks, E-commerce     |

### Wildcard và Multi-domain

```
Wildcard Certificate:
*.example.com → Bảo vệ tất cả subdomains
  ├── www.example.com     ✅
  ├── api.example.com     ✅
  ├── mail.example.com    ✅
  └── blog.example.com    ✅

Multi-domain (SAN) Certificate:
  ├── example.com         ✅
  ├── example.net         ✅
  └── example.org         ✅
```

---

## 5. HTTP Methods

### Các HTTP Methods phổ biến

| Method      | Mục đích                 | Idempotent | Safe | Body |
| ----------- | ------------------------ | ---------- | ---- | ---- |
| **GET**     | Lấy dữ liệu              | ✅         | ✅   | ❌   |
| **POST**    | Tạo mới                  | ❌         | ❌   | ✅   |
| **PUT**     | Cập nhật toàn bộ         | ✅         | ❌   | ✅   |
| **PATCH**   | Cập nhật một phần        | ❌         | ❌   | ✅   |
| **DELETE**  | Xóa                      | ✅         | ❌   | ❌   |
| **HEAD**    | Lấy headers (không body) | ✅         | ✅   | ❌   |
| **OPTIONS** | Lấy methods được hỗ trợ  | ✅         | ✅   | ❌   |

### Giải thích Idempotent và Safe

```
IDEMPOTENT: Gọi nhiều lần cho cùng kết quả
─────────────────────────────────────────
GET /users/1     → Luôn trả về user 1
PUT /users/1     → Luôn update user 1 thành cùng data
DELETE /users/1  → User 1 bị xóa (gọi lại vẫn "đã xóa")

POST /users      → Mỗi lần tạo user MỚI (không idempotent)

SAFE: Không thay đổi state của server
─────────────────────────────────────────
GET, HEAD, OPTIONS → Chỉ đọc, không thay đổi gì
POST, PUT, DELETE  → Thay đổi data trên server
```

### Ví dụ CRUD với HTTP Methods

```http
# CREATE - Tạo user mới
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "John",
  "email": "john@example.com"
}

# READ - Lấy danh sách users
GET /api/users HTTP/1.1

# READ - Lấy user cụ thể
GET /api/users/1 HTTP/1.1

# UPDATE - Cập nhật toàn bộ user
PUT /api/users/1 HTTP/1.1
Content-Type: application/json

{
  "name": "John Updated",
  "email": "john.updated@example.com"
}

# UPDATE - Cập nhật một phần
PATCH /api/users/1 HTTP/1.1
Content-Type: application/json

{
  "email": "newemail@example.com"
}

# DELETE - Xóa user
DELETE /api/users/1 HTTP/1.1
```

### PUT vs PATCH

```
PUT - Thay thế TOÀN BỘ resource
────────────────────────────────
Current: { "name": "John", "email": "john@ex.com", "age": 25 }

PUT /users/1
{ "name": "John Updated" }

Result: { "name": "John Updated" }  ← email và age bị MẤT!


PATCH - Cập nhật MỘT PHẦN resource
────────────────────────────────
Current: { "name": "John", "email": "john@ex.com", "age": 25 }

PATCH /users/1
{ "name": "John Updated" }

Result: { "name": "John Updated", "email": "john@ex.com", "age": 25 }
```

---

## 6. HTTP Status Codes

### Phân loại Status Codes

```
┌─────────────────────────────────────────────────────────────┐
│                   HTTP STATUS CODES                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1xx - INFORMATIONAL    │  Request đang được xử lý         │
│  ───────────────────────┼─────────────────────────────────  │
│  100 Continue           │  Tiếp tục gửi request            │
│  101 Switching Protocols│  Đổi protocol (WebSocket)        │
│                                                             │
│  2xx - SUCCESS          │  Request thành công              │
│  ───────────────────────┼─────────────────────────────────  │
│  200 OK                 │  Thành công                      │
│  201 Created            │  Tạo mới thành công              │
│  204 No Content         │  Thành công, không có body       │
│                                                             │
│  3xx - REDIRECTION      │  Cần thêm action                 │
│  ───────────────────────┼─────────────────────────────────  │
│  301 Moved Permanently  │  URL đã đổi vĩnh viễn            │
│  302 Found              │  Redirect tạm thời               │
│  304 Not Modified       │  Dùng cache                      │
│                                                             │
│  4xx - CLIENT ERROR     │  Lỗi từ phía client              │
│  ───────────────────────┼─────────────────────────────────  │
│  400 Bad Request        │  Request không hợp lệ            │
│  401 Unauthorized       │  Chưa xác thực                   │
│  403 Forbidden          │  Không có quyền                  │
│  404 Not Found          │  Không tìm thấy resource         │
│  405 Method Not Allowed │  Method không được phép          │
│  429 Too Many Requests  │  Rate limit exceeded             │
│                                                             │
│  5xx - SERVER ERROR     │  Lỗi từ phía server              │
│  ───────────────────────┼─────────────────────────────────  │
│  500 Internal Server Error │  Lỗi server chung             │
│  502 Bad Gateway        │  Gateway/proxy lỗi               │
│  503 Service Unavailable│  Server quá tải/bảo trì          │
│  504 Gateway Timeout    │  Gateway timeout                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Chi tiết các Status Codes quan trọng

#### 2xx Success

| Code    | Name       | Mô tả                             | Use Case         |
| ------- | ---------- | --------------------------------- | ---------------- |
| **200** | OK         | Request thành công                | GET, PUT, PATCH  |
| **201** | Created    | Resource được tạo                 | POST tạo mới     |
| **202** | Accepted   | Request được chấp nhận, xử lý sau | Async operations |
| **204** | No Content | Thành công, không có body         | DELETE           |

#### 3xx Redirection

| Code    | Name               | Mô tả               | SEO Impact       |
| ------- | ------------------ | ------------------- | ---------------- |
| **301** | Moved Permanently  | URL đổi vĩnh viễn   | Chuyển SEO juice |
| **302** | Found              | Redirect tạm thời   | Không chuyển SEO |
| **303** | See Other          | Redirect sau POST   | -                |
| **304** | Not Modified       | Dùng cached version | -                |
| **307** | Temporary Redirect | Giữ nguyên method   | -                |
| **308** | Permanent Redirect | Giữ nguyên method   | Chuyển SEO juice |

#### 4xx Client Errors

| Code    | Name                 | Nguyên nhân        | Cách fix                     |
| ------- | -------------------- | ------------------ | ---------------------------- |
| **400** | Bad Request          | Request sai format | Kiểm tra request body/params |
| **401** | Unauthorized         | Chưa đăng nhập     | Thêm authentication          |
| **403** | Forbidden            | Không có quyền     | Kiểm tra permissions         |
| **404** | Not Found            | URL không tồn tại  | Kiểm tra URL                 |
| **405** | Method Not Allowed   | Method sai         | Dùng đúng HTTP method        |
| **409** | Conflict             | Xung đột data      | Resolve conflict             |
| **422** | Unprocessable Entity | Validation failed  | Fix validation errors        |
| **429** | Too Many Requests    | Rate limited       | Chờ và retry                 |

#### 5xx Server Errors

| Code    | Name                  | Nguyên nhân         | Cách fix                  |
| ------- | --------------------- | ------------------- | ------------------------- |
| **500** | Internal Server Error | Bug trong code      | Check server logs         |
| **502** | Bad Gateway           | Upstream server lỗi | Check proxy/load balancer |
| **503** | Service Unavailable   | Server overload     | Scale up, retry later     |
| **504** | Gateway Timeout       | Upstream timeout    | Increase timeout          |

### 401 vs 403

```
401 Unauthorized - "Bạn là ai?"
─────────────────────────────────
• Chưa xác thực (chưa login)
• Token hết hạn hoặc không hợp lệ
• Cần gửi credentials

403 Forbidden - "Tôi biết bạn, nhưng bạn không được phép"
─────────────────────────────────
• Đã xác thực nhưng không có quyền
• Role không đủ
• IP bị block
```

---

## 7. HTTP Headers

### Request Headers phổ biến

```http
# General
Host: api.example.com              # Domain của server
Connection: keep-alive             # Giữ kết nối
User-Agent: Mozilla/5.0...         # Thông tin browser/client

# Content
Content-Type: application/json     # Kiểu dữ liệu gửi đi
Content-Length: 256                # Kích thước body
Accept: application/json           # Kiểu dữ liệu mong muốn nhận
Accept-Language: en-US,vi          # Ngôn ngữ ưu tiên
Accept-Encoding: gzip, deflate     # Encoding được hỗ trợ

# Authentication
Authorization: Bearer <token>      # Token xác thực
Cookie: session=abc123             # Cookies

# Caching
Cache-Control: no-cache            # Điều khiển cache
If-None-Match: "etag123"           # Conditional request
If-Modified-Since: Wed, 21 Oct...  # Conditional request

# CORS
Origin: https://example.com        # Origin của request

# Custom
X-Request-ID: uuid-123             # Request tracking
X-API-Key: key123                  # API key
```

### Response Headers phổ biến

```http
# Content
Content-Type: application/json; charset=utf-8
Content-Length: 1234
Content-Encoding: gzip

# Caching
Cache-Control: max-age=3600, public
ETag: "abc123"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
Expires: Thu, 22 Oct 2024 07:28:00 GMT

# Security
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'

# CORS
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400

# Cookies
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict

# Other
Server: nginx/1.18.0
Date: Wed, 21 Oct 2024 07:28:00 GMT
```

### Content-Type phổ biến

| Content-Type                        | Mô tả       | Ví dụ         |
| ----------------------------------- | ----------- | ------------- |
| `application/json`                  | JSON data   | API responses |
| `application/xml`                   | XML data    | SOAP APIs     |
| `application/x-www-form-urlencoded` | Form data   | HTML forms    |
| `multipart/form-data`               | File upload | Upload files  |
| `text/html`                         | HTML        | Web pages     |
| `text/plain`                        | Plain text  | Text files    |
| `text/css`                          | CSS         | Stylesheets   |
| `application/javascript`            | JavaScript  | JS files      |
| `image/png`, `image/jpeg`           | Images      | Pictures      |

---

## 8. HTTP Versions

### So sánh các phiên bản HTTP

| Feature                 | HTTP/1.0             | HTTP/1.1        | HTTP/2            | HTTP/3            |
| ----------------------- | -------------------- | --------------- | ----------------- | ----------------- |
| **Năm**                 | 1996                 | 1997            | 2015              | 2022              |
| **Connection**          | Đóng sau mỗi request | Keep-alive      | Multiplexing      | Multiplexing      |
| **Requests/Connection** | 1                    | Nhiều (tuần tự) | Nhiều (song song) | Nhiều (song song) |
| **Header Compression**  | ❌                   | ❌              | ✅ HPACK          | ✅ QPACK          |
| **Server Push**         | ❌                   | ❌              | ✅                | ✅                |
| **Protocol**            | TCP                  | TCP             | TCP               | QUIC (UDP)        |
| **Encryption**          | Optional             | Optional        | Thường có TLS     | Bắt buộc TLS      |

### HTTP/1.0 vs HTTP/1.1

```
HTTP/1.0 - Mỗi request một connection
──────────────────────────────────────
Client          Server
  │──── GET /page.html ────▶│
  │◀─── Response ───────────│
  │     [Connection closed] │
  │                         │
  │──── GET /style.css ────▶│  ← Mở connection MỚI
  │◀─── Response ───────────│
  │     [Connection closed] │


HTTP/1.1 - Keep-alive (Persistent Connection)
──────────────────────────────────────
Client          Server
  │══════ Connection ═══════│
  │──── GET /page.html ────▶│
  │◀─── Response ───────────│
  │──── GET /style.css ────▶│  ← Cùng connection
  │◀─── Response ───────────│
  │──── GET /script.js ────▶│  ← Cùng connection
  │◀─── Response ───────────│
  │══════════════════════════│
```

### HTTP/2 - Multiplexing

```
HTTP/1.1 - Head-of-line blocking
──────────────────────────────────────
Request 1 ────▶ [Processing...] ────▶ Response 1
                Request 2 ────▶ [Wait...] [Processing] ────▶ Response 2
                                Request 3 ────▶ [Wait...] ────▶ Response 3

HTTP/2 - Multiplexing (Song song)
──────────────────────────────────────
Request 1 ────▶ [Processing] ────▶ Response 1
Request 2 ────▶ [Processing] ────▶ Response 2    ← Đồng thời!
Request 3 ────▶ [Processing] ────▶ Response 3
```

### HTTP/3 - QUIC Protocol

```
┌─────────────────────────────────────────────────────────────┐
│                        HTTP/3                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  HTTP/2 over TCP:                                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ TCP Packet Loss → Tất cả streams bị block           │   │
│  │ (Head-of-line blocking ở transport layer)           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  HTTP/3 over QUIC (UDP):                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Packet Loss → Chỉ stream đó bị ảnh hưởng            │   │
│  │ Các streams khác tiếp tục bình thường               │   │
│  │ + 0-RTT connection establishment                    │   │
│  │ + Built-in encryption (TLS 1.3)                     │   │
│  │ + Connection migration (đổi network không mất)      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 9. URL và URI

### Cấu trúc URL

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              URL STRUCTURE                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  https://user:pass@www.example.com:443/path/to/page?query=value#section│
│  ──┬──  ───┬────  ───────┬────────  ┬  ─────┬───── ─────┬───── ───┬─── │
│    │       │             │          │       │           │         │     │
│ Scheme  UserInfo      Host        Port    Path       Query    Fragment │
│                                                                         │
│  Scheme:   Protocol (http, https, ftp, mailto)                         │
│  UserInfo: Username:password (hiếm dùng, không an toàn)                │
│  Host:     Domain name hoặc IP address                                 │
│  Port:     Port number (mặc định: 80 HTTP, 443 HTTPS)                  │
│  Path:     Đường dẫn đến resource                                      │
│  Query:    Parameters (?key=value&key2=value2)                         │
│  Fragment: Anchor trong page (#section)                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### URI vs URL vs URN

```
┌─────────────────────────────────────────────────────────────┐
│                         URI                                 │
│              (Uniform Resource Identifier)                  │
│                                                             │
│    ┌─────────────────────┐  ┌─────────────────────┐        │
│    │        URL          │  │        URN          │        │
│    │ (Uniform Resource   │  │ (Uniform Resource   │        │
│    │     Locator)        │  │      Name)          │        │
│    │                     │  │                     │        │
│    │ WHERE to find       │  │ WHAT it is          │        │
│    │                     │  │                     │        │
│    │ https://example.com │  │ urn:isbn:0451450523 │        │
│    │ ftp://files.com/doc │  │ urn:uuid:6e8bc430.. │        │
│    └─────────────────────┘  └─────────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘

URI = URL + URN
URL: Địa chỉ để TÌM resource
URN: Tên để ĐỊNH DANH resource
```

### URL Encoding

```
Ký tự đặc biệt cần encode:

Space  →  %20 hoặc +
!      →  %21
#      →  %23
$      →  %24
&      →  %26
'      →  %27
(      →  %28
)      →  %29
*      →  %2A
+      →  %2B
,      →  %2C
/      →  %2F
:      →  %3A
;      →  %3B
=      →  %3D
?      →  %3F
@      →  %40

Ví dụ:
"Hello World" → "Hello%20World"
"name=John&age=25" → "name%3DJohn%26age%3D25"
```

---

## 10. DNS - Domain Name System

### DNS là gì?

**DNS** (Domain Name System) là hệ thống phân giải tên miền, chuyển đổi domain name thành IP address.

```
Không có DNS:
Bạn phải nhớ: 142.250.190.78 để vào Google

Có DNS:
Bạn chỉ cần nhớ: google.com
DNS sẽ chuyển google.com → 142.250.190.78
```

### Quá trình DNS Resolution

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        DNS RESOLUTION PROCESS                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  User types: www.example.com                                           │
│       │                                                                 │
│       ▼                                                                 │
│  ┌─────────────┐                                                       │
│  │ 1. Browser  │ Check browser cache                                   │
│  │    Cache    │ Found? → Return IP                                    │
│  └──────┬──────┘                                                       │
│         │ Not found                                                     │
│         ▼                                                               │
│  ┌─────────────┐                                                       │
│  │ 2. OS Cache │ Check operating system cache                          │
│  │   (hosts)   │ Found? → Return IP                                    │
│  └──────┬──────┘                                                       │
│         │ Not found                                                     │
│         ▼                                                               │
│  ┌─────────────┐                                                       │
│  │ 3. Resolver │ ISP's DNS resolver                                    │
│  │    (ISP)    │ Check cache → Found? Return                           │
│  └──────┬──────┘                                                       │
│         │ Not found                                                     │
│         ▼                                                               │
│  ┌─────────────┐                                                       │
│  │ 4. Root DNS │ "Ai quản lý .com?"                                    │
│  │   Servers   │ → Trả về TLD server                                   │
│  └──────┬──────┘                                                       │
│         ▼                                                               │
│  ┌─────────────┐                                                       │
│  │ 5. TLD DNS  │ "Ai quản lý example.com?"                             │
│  │   (.com)    │ → Trả về Authoritative NS                             │
│  └──────┬──────┘                                                       │
│         ▼                                                               │
│  ┌─────────────┐                                                       │
│  │6.Authoritat-│ "IP của www.example.com?"                             │
│  │  ive DNS    │ → Trả về 93.184.216.34                                │
│  └──────┬──────┘                                                       │
│         │                                                               │
│         ▼                                                               │
│  IP: 93.184.216.34 → Browser connects to server                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### DNS Record Types

| Type      | Name               | Mô tả                 | Ví dụ                               |
| --------- | ------------------ | --------------------- | ----------------------------------- |
| **A**     | Address            | IPv4 address          | `example.com → 93.184.216.34`       |
| **AAAA**  | IPv6 Address       | IPv6 address          | `example.com → 2606:2800:220:1:...` |
| **CNAME** | Canonical Name     | Alias cho domain khác | `www.example.com → example.com`     |
| **MX**    | Mail Exchange      | Mail server           | `example.com → mail.example.com`    |
| **TXT**   | Text               | Text records          | SPF, DKIM, verification             |
| **NS**    | Name Server        | DNS servers           | `example.com → ns1.example.com`     |
| **SOA**   | Start of Authority | Zone info             | Primary NS, admin email             |
| **PTR**   | Pointer            | Reverse DNS           | IP → Domain                         |
| **SRV**   | Service            | Service location      | `_sip._tcp.example.com`             |

### Ví dụ DNS Records

```
; A Records
example.com.        IN  A       93.184.216.34
www.example.com.    IN  A       93.184.216.34

; AAAA Record (IPv6)
example.com.        IN  AAAA    2606:2800:220:1:248:1893:25c8:1946

; CNAME Record
blog.example.com.   IN  CNAME   example.com.
api.example.com.    IN  CNAME   lb.example.com.

; MX Records (Mail)
example.com.        IN  MX  10  mail1.example.com.
example.com.        IN  MX  20  mail2.example.com.

; TXT Records
example.com.        IN  TXT     "v=spf1 include:_spf.google.com ~all"

; NS Records
example.com.        IN  NS      ns1.example.com.
example.com.        IN  NS      ns2.example.com.
```

---

## 11. SSL/TLS Certificates

### SSL vs TLS

| Aspect        | SSL                          | TLS                      |
| ------------- | ---------------------------- | ------------------------ |
| **Full name** | Secure Sockets Layer         | Transport Layer Security |
| **Versions**  | SSL 2.0, 3.0 (deprecated)    | TLS 1.0, 1.1, 1.2, 1.3   |
| **Status**    | ❌ Deprecated, không an toàn | ✅ Đang sử dụng          |
| **Current**   | -                            | TLS 1.3 (2018)           |

> **Note**: Mọi người vẫn gọi "SSL Certificate" nhưng thực tế là TLS Certificate

### Cách lấy SSL Certificate

#### 1. Let's Encrypt (Miễn phí)

```bash
# Sử dụng Certbot
sudo apt install certbot python3-certbot-nginx

# Lấy certificate cho Nginx
sudo certbot --nginx -d example.com -d www.example.com

# Auto-renewal
sudo certbot renew --dry-run
```

#### 2. Cloudflare (Miễn phí)

```
1. Đăng ký Cloudflare
2. Thêm domain
3. Đổi nameservers
4. Bật SSL/TLS → Full (strict)
5. Done! Cloudflare cung cấp SSL miễn phí
```

#### 3. Mua từ CA (Certificate Authority)

```
Các CA phổ biến:
• DigiCert
• Comodo/Sectigo
• GlobalSign
• GoDaddy
• Thawte
```

### Cấu hình HTTPS trên Nginx

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL Certificate
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Other security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    root /var/www/example.com;
    index index.html;
}
```

---

## 12. Cookies và Sessions

### Cookies là gì?

**Cookies** là những đoạn dữ liệu nhỏ được server gửi và lưu trữ trên browser của user.

```
┌─────────────────────────────────────────────────────────────┐
│                    HOW COOKIES WORK                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   CLIENT                                    SERVER          │
│     │                                         │             │
│     │  1. Request (no cookie)                 │             │
│     │  ──────────────────────────────────▶   │             │
│     │                                         │             │
│     │  2. Response + Set-Cookie               │             │
│     │  ◀──────────────────────────────────   │             │
│     │  Set-Cookie: session=abc123             │             │
│     │                                         │             │
│     │  [Browser stores cookie]                │             │
│     │                                         │             │
│     │  3. Next Request + Cookie               │             │
│     │  ──────────────────────────────────▶   │             │
│     │  Cookie: session=abc123                 │             │
│     │                                         │             │
│     │  4. Server identifies user              │             │
│     │  ◀──────────────────────────────────   │             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Cookie Attributes

```http
Set-Cookie: name=value; Domain=example.com; Path=/; Expires=Thu, 01 Jan 2025 00:00:00 GMT; Max-Age=31536000; Secure; HttpOnly; SameSite=Strict
```

| Attribute      | Mô tả                       | Ví dụ                         |
| -------------- | --------------------------- | ----------------------------- |
| **name=value** | Tên và giá trị cookie       | `session=abc123`              |
| **Domain**     | Domain được phép gửi cookie | `Domain=example.com`          |
| **Path**       | Path được phép gửi cookie   | `Path=/api`                   |
| **Expires**    | Thời điểm hết hạn           | `Expires=Thu, 01 Jan 2025...` |
| **Max-Age**    | Thời gian sống (giây)       | `Max-Age=3600`                |
| **Secure**     | Chỉ gửi qua HTTPS           | `Secure`                      |
| **HttpOnly**   | Không truy cập được từ JS   | `HttpOnly`                    |
| **SameSite**   | Chống CSRF                  | `SameSite=Strict\|Lax\|None`  |

### SameSite Cookie

```
SameSite=Strict
─────────────────
Cookie CHỈ được gửi khi request từ CÙNG site
• User click link từ email → Cookie KHÔNG gửi
• Bảo mật cao nhất

SameSite=Lax (Default)
─────────────────
Cookie được gửi với top-level navigation (GET)
• User click link từ email → Cookie được gửi
• Form POST từ site khác → Cookie KHÔNG gửi

SameSite=None
─────────────────
Cookie được gửi với mọi request (cần Secure)
• Dùng cho cross-site requests
• Phải có Secure attribute
```

### Sessions

```
┌─────────────────────────────────────────────────────────────┐
│                   SESSION-BASED AUTH                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   CLIENT                                    SERVER          │
│     │                                         │             │
│     │  1. Login (username, password)          │             │
│     │  ──────────────────────────────────▶   │             │
│     │                                         │             │
│     │                              ┌──────────────────┐     │
│     │                              │ Create Session   │     │
│     │                              │ Store in DB/Redis│     │
│     │                              │ session_id: data │     │
│     │                              └──────────────────┘     │
│     │                                         │             │
│     │  2. Set-Cookie: session_id=xyz          │             │
│     │  ◀──────────────────────────────────   │             │
│     │                                         │             │
│     │  3. Request + Cookie: session_id=xyz    │             │
│     │  ──────────────────────────────────▶   │             │
│     │                                         │             │
│     │                              ┌──────────────────┐     │
│     │                              │ Lookup session   │     │
│     │                              │ Get user data    │     │
│     │                              └──────────────────┘     │
│     │                                         │             │
│     │  4. Response (authenticated)            │             │
│     │  ◀──────────────────────────────────   │             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Cookies vs Sessions vs Tokens

| Aspect          | Cookies          | Sessions                 | JWT Tokens                    |
| --------------- | ---------------- | ------------------------ | ----------------------------- |
| **Lưu trữ**     | Browser          | Server                   | Browser (localStorage/cookie) |
| **Stateful**    | -                | ✅ Yes                   | ❌ No (Stateless)             |
| **Scalability** | -                | Khó (cần shared storage) | Dễ                            |
| **Size**        | 4KB limit        | Không giới hạn           | Có thể lớn                    |
| **Security**    | HttpOnly, Secure | Server-side              | Signature verification        |

---

## 13. CORS - Cross-Origin Resource Sharing

### CORS là gì?

**CORS** là cơ chế cho phép web page từ một origin truy cập resources từ origin khác.

### Same-Origin Policy

```
Origin = Protocol + Host + Port

https://example.com:443/page
──┬──   ─────┬─────  ─┬─
Protocol   Host     Port

Same Origin:
https://example.com/page1  ✅
https://example.com/page2  ✅

Different Origin:
http://example.com         ❌ (khác protocol)
https://api.example.com    ❌ (khác host/subdomain)
https://example.com:8080   ❌ (khác port)
https://other.com          ❌ (khác domain)
```

### CORS Flow

```
┌─────────────────────────────────────────────────────────────┐
│                      CORS FLOW                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SIMPLE REQUEST (GET, POST với simple headers)             │
│  ─────────────────────────────────────────────             │
│                                                             │
│   Browser                                    Server         │
│     │  GET /api/data                           │            │
│     │  Origin: https://frontend.com            │            │
│     │  ─────────────────────────────────────▶ │            │
│     │                                          │            │
│     │  Access-Control-Allow-Origin: *          │            │
│     │  ◀───────────────────────────────────── │            │
│     │                                          │            │
│                                                             │
│  PREFLIGHT REQUEST (PUT, DELETE, custom headers)           │
│  ─────────────────────────────────────────────             │
│                                                             │
│   Browser                                    Server         │
│     │  OPTIONS /api/data (Preflight)           │            │
│     │  Origin: https://frontend.com            │            │
│     │  Access-Control-Request-Method: PUT      │            │
│     │  Access-Control-Request-Headers: X-Custom│            │
│     │  ─────────────────────────────────────▶ │            │
│     │                                          │            │
│     │  Access-Control-Allow-Origin: https://...│            │
│     │  Access-Control-Allow-Methods: PUT       │            │
│     │  Access-Control-Allow-Headers: X-Custom  │            │
│     │  Access-Control-Max-Age: 86400           │            │
│     │  ◀───────────────────────────────────── │            │
│     │                                          │            │
│     │  PUT /api/data (Actual Request)          │            │
│     │  ─────────────────────────────────────▶ │            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### CORS Headers

```http
# Response Headers
Access-Control-Allow-Origin: https://example.com  # Hoặc *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
Access-Control-Expose-Headers: X-Custom-Header
```

### Cấu hình CORS

#### Express.js

```javascript
const cors = require("cors");

// Allow all origins
app.use(cors());

// Specific configuration
app.use(
  cors({
    origin: ["https://frontend.com", "https://admin.com"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true,
    maxAge: 86400,
  })
);
```

#### Nginx

```nginx
location /api {
    add_header 'Access-Control-Allow-Origin' 'https://frontend.com' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization' always;
    add_header 'Access-Control-Allow-Credentials' 'true' always;

    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Max-Age' 86400;
        add_header 'Content-Length' 0;
        return 204;
    }
}
```

---

## 14. Caching

### Tại sao cần Caching?

```
Không có Cache:
User → Request → Server → Database → Response → User
                 (100ms)    (50ms)     (100ms)
                 Total: ~250ms mỗi request

Có Cache:
User → Request → Cache Hit → Response → User
                  (5ms)       (5ms)
                 Total: ~10ms (nhanh hơn 25x!)
```

### Các loại Cache

```
┌─────────────────────────────────────────────────────────────┐
│                    CACHING LAYERS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐                                           │
│  │   Browser   │  ← Browser Cache (localStorage, memory)   │
│  │    Cache    │                                           │
│  └──────┬──────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────┐                                           │
│  │     CDN     │  ← Edge Cache (Cloudflare, AWS CloudFront)│
│  │    Cache    │                                           │
│  └──────┬──────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────┐                                           │
│  │   Reverse   │  ← Nginx, Varnish                         │
│  │    Proxy    │                                           │
│  └──────┬──────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────┐                                           │
│  │ Application │  ← Redis, Memcached                       │
│  │    Cache    │                                           │
│  └──────┬──────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────┐                                           │
│  │  Database   │  ← Query Cache                            │
│  │    Cache    │                                           │
│  └─────────────┘                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### HTTP Cache Headers

#### Cache-Control

```http
# Response có thể cache ở mọi nơi, 1 giờ
Cache-Control: public, max-age=3600

# Chỉ browser được cache, 1 ngày
Cache-Control: private, max-age=86400

# Không cache
Cache-Control: no-store

# Cache nhưng phải validate trước khi dùng
Cache-Control: no-cache

# Cache nhưng phải revalidate khi hết hạn
Cache-Control: must-revalidate

# Immutable - không bao giờ thay đổi
Cache-Control: public, max-age=31536000, immutable
```

#### Cache-Control Directives

| Directive                  | Mô tả                           |
| -------------------------- | ------------------------------- |
| `public`                   | Có thể cache ở CDN, proxy       |
| `private`                  | Chỉ browser được cache          |
| `no-store`                 | Không cache gì cả               |
| `no-cache`                 | Cache nhưng validate mỗi lần    |
| `max-age=N`                | Cache trong N giây              |
| `s-maxage=N`               | max-age cho shared cache (CDN)  |
| `must-revalidate`          | Phải validate khi stale         |
| `immutable`                | Content không bao giờ đổi       |
| `stale-while-revalidate=N` | Dùng stale trong khi revalidate |

#### ETag và Last-Modified

```
┌─────────────────────────────────────────────────────────────┐
│                  CONDITIONAL REQUESTS                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  First Request:                                            │
│  ─────────────                                             │
│  GET /api/data                                             │
│                                                             │
│  Response:                                                 │
│  HTTP/1.1 200 OK                                           │
│  ETag: "abc123"                                            │
│  Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT              │
│  Cache-Control: max-age=0, must-revalidate                 │
│  { "data": "..." }                                         │
│                                                             │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  Subsequent Request (Validation):                          │
│  ─────────────────────────────────                         │
│  GET /api/data                                             │
│  If-None-Match: "abc123"                                   │
│  If-Modified-Since: Wed, 21 Oct 2024 07:28:00 GMT          │
│                                                             │
│  Response (Not Modified):                                  │
│  HTTP/1.1 304 Not Modified                                 │
│  (No body - use cached version)                            │
│                                                             │
│  Response (Modified):                                      │
│  HTTP/1.1 200 OK                                           │
│  ETag: "xyz789"                                            │
│  { "data": "new data..." }                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Caching Strategies

```
┌─────────────────────────────────────────────────────────────┐
│                  CACHING STRATEGIES                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Cache-First (Offline-first)                            │
│  ───────────────────────────────                           │
│  Check cache → Hit? Return : Fetch → Cache → Return        │
│  Use: Static assets, offline apps                          │
│                                                             │
│  2. Network-First                                          │
│  ───────────────────────────────                           │
│  Fetch → Success? Return : Check cache → Return            │
│  Use: API data, real-time content                          │
│                                                             │
│  3. Stale-While-Revalidate                                 │
│  ───────────────────────────────                           │
│  Return cached → Fetch in background → Update cache        │
│  Use: News feeds, social media                             │
│                                                             │
│  4. Cache-Only                                             │
│  ───────────────────────────────                           │
│  Only return from cache                                    │
│  Use: Offline-only resources                               │
│                                                             │
│  5. Network-Only                                           │
│  ───────────────────────────────                           │
│  Always fetch, never cache                                 │
│  Use: Real-time data, analytics                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 15. Security Best Practices

### HTTPS Everywhere

```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

# HSTS Header
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

### Security Headers

```http
# Prevent clickjacking
X-Frame-Options: DENY
# Hoặc
Content-Security-Policy: frame-ancestors 'none';

# Prevent MIME sniffing
X-Content-Type-Options: nosniff

# XSS Protection (legacy browsers)
X-XSS-Protection: 1; mode=block

# Content Security Policy
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:;

# Referrer Policy
Referrer-Policy: strict-origin-when-cross-origin

# Permissions Policy
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

### Common Vulnerabilities

```
┌─────────────────────────────────────────────────────────────┐
│                 COMMON WEB VULNERABILITIES                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. XSS (Cross-Site Scripting)                             │
│  ─────────────────────────────                             │
│  Attack: Inject malicious scripts                          │
│  Prevention:                                               │
│  • Escape output                                           │
│  • Content-Security-Policy                                 │
│  • HttpOnly cookies                                        │
│                                                             │
│  2. CSRF (Cross-Site Request Forgery)                      │
│  ─────────────────────────────                             │
│  Attack: Trick user to perform unwanted actions            │
│  Prevention:                                               │
│  • CSRF tokens                                             │
│  • SameSite cookies                                        │
│  • Check Origin/Referer headers                            │
│                                                             │
│  3. SQL Injection                                          │
│  ─────────────────────────────                             │
│  Attack: Inject SQL commands                               │
│  Prevention:                                               │
│  • Parameterized queries                                   │
│  • ORM                                                     │
│  • Input validation                                        │
│                                                             │
│  4. Man-in-the-Middle (MITM)                               │
│  ─────────────────────────────                             │
│  Attack: Intercept communication                           │
│  Prevention:                                               │
│  • HTTPS                                                   │
│  • HSTS                                                    │
│  • Certificate pinning                                     │
│                                                             │
│  5. Clickjacking                                           │
│  ─────────────────────────────                             │
│  Attack: Trick user to click hidden elements               │
│  Prevention:                                               │
│  • X-Frame-Options                                         │
│  • CSP frame-ancestors                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Rate Limiting

```javascript
// Express.js với express-rate-limit
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per windowMs
  message: "Too many requests, please try again later.",
  standardHeaders: true,
  legacyHeaders: false,
});

app.use("/api/", limiter);
```

```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

location /api/ {
    limit_req zone=api burst=20 nodelay;
    limit_req_status 429;
}
```

---

## 📚 Tổng kết

### Quick Reference

```
┌─────────────────────────────────────────────────────────────┐
│                    QUICK REFERENCE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  HTTP Methods:                                             │
│  GET (Read) | POST (Create) | PUT (Replace) |              │
│  PATCH (Update) | DELETE (Remove)                          │
│                                                             │
│  Status Codes:                                             │
│  2xx Success | 3xx Redirect | 4xx Client Error |           │
│  5xx Server Error                                          │
│                                                             │
│  Common Ports:                                             │
│  HTTP: 80 | HTTPS: 443 | FTP: 21 | SSH: 22                │
│                                                             │
│  Security Headers:                                         │
│  HSTS | CSP | X-Frame-Options | X-Content-Type-Options    │
│                                                             │
│  Cache Headers:                                            │
│  Cache-Control | ETag | Last-Modified | Expires           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Checklist cho Production

```
✅ HTTPS với TLS 1.2+
✅ HSTS enabled
✅ Security headers configured
✅ CORS properly configured
✅ Rate limiting enabled
✅ Input validation
✅ Output encoding
✅ CSRF protection
✅ Secure cookies (HttpOnly, Secure, SameSite)
✅ Error handling (không leak sensitive info)
✅ Logging và monitoring
✅ Regular security updates
```

---

## 📖 Tài liệu tham khảo

- [MDN Web Docs - HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [HTTP/2 Specification](https://httpwg.org/specs/rfc7540.html)
- [HTTP/3 Specification](https://www.rfc-editor.org/rfc/rfc9114.html)
- [OWASP Security Guidelines](https://owasp.org/www-project-web-security-testing-guide/)
- [Let's Encrypt](https://letsencrypt.org/)
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/)
