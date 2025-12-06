# SQL vs NoSQL: So sánh toàn diện

## 📌 Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [So sánh chi tiết](#2-so-sánh-chi-tiết)
3. [Hiệu suất Query](#3-hiệu-suất-query---cái-nào-nhanh-hơn)
4. [Khi nào dùng SQL vs NoSQL](#4-khi-nào-dùng-sql-vs-nosql)
5. [Ví dụ thực tế](#5-ví-dụ-thực-tế)

---

## 1. Tổng quan

### SQL (Relational Database)

```
┌─────────────────────────────────────────────────────────────┐
│                    SQL DATABASE                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Table A   │───►│   Table B   │───►│   Table C   │     │
│  │  (Users)    │    │  (Orders)   │    │ (Products)  │     │
│  ├─────────────┤    ├─────────────┤    ├─────────────┤     │
│  │ id (PK)     │    │ id (PK)     │    │ id (PK)     │     │
│  │ name        │    │ user_id(FK) │    │ name        │     │
│  │ email       │    │ product_id  │    │ price       │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                             │
│  ✓ Cấu trúc cố định (Schema)                               │
│  ✓ Quan hệ rõ ràng giữa các bảng                           │
│  ✓ ACID compliance                                          │
└─────────────────────────────────────────────────────────────┘

Ví dụ: MySQL, PostgreSQL, Oracle, SQL Server, SQLite
```

### NoSQL (Non-Relational Database)

```
┌─────────────────────────────────────────────────────────────┐
│                   NoSQL DATABASE                            │
│                                                             │
│  Document DB          Key-Value         Column-Family       │
│  ┌───────────┐       ┌─────┬─────┐     ┌─────────────┐     │
│  │ {         │       │ Key │Value│     │ Row Key     │     │
│  │  "_id":1, │       ├─────┼─────┤     ├──────┬──────┤     │
│  │  "name":  │       │user1│{...}│     │ Col1 │ Col2 │     │
│  │   "John", │       │user2│{...}│     │      │      │     │
│  │  "orders":│       └─────┴─────┘     └──────┴──────┘     │
│  │   [...]   │                                              │
│  │ }         │       Graph DB                               │
│  └───────────┘       ┌───┐   ┌───┐                         │
│                      │ A │──►│ B │                         │
│                      └───┘   └─┬─┘                         │
│                          ◄────┘                            │
│  ✓ Schema linh hoạt (Schemaless)                           │
│  ✓ Dữ liệu phi cấu trúc                                    │
│  ✓ Horizontal scaling                                       │
└─────────────────────────────────────────────────────────────┘

Ví dụ: MongoDB, Redis, Cassandra, Neo4j, DynamoDB
```

---

## 2. So sánh chi tiết

### 📊 Bảng so sánh tổng quan

| Tiêu chí              | SQL                        | NoSQL                              |
| --------------------- | -------------------------- | ---------------------------------- |
| **Cấu trúc dữ liệu**  | Bảng với hàng và cột       | Document, Key-Value, Graph, Column |
| **Schema**            | Cố định, định nghĩa trước  | Linh hoạt, động                    |
| **Quan hệ**           | Có (Foreign Key, JOIN)     | Không hoặc hạn chế                 |
| **Ngôn ngữ truy vấn** | SQL chuẩn                  | Khác nhau tùy DB                   |
| **Scaling**           | Vertical (nâng cấp server) | Horizontal (thêm server)           |
| **ACID**              | Đầy đủ                     | Thường là BASE                     |
| **Transactions**      | Hỗ trợ tốt                 | Hạn chế                            |
| **Consistency**       | Strong consistency         | Eventual consistency               |

### 🔄 ACID vs BASE

```
┌─────────────────────────────────────┐    ┌─────────────────────────────────────┐
│            ACID (SQL)               │    │           BASE (NoSQL)              │
├─────────────────────────────────────┤    ├─────────────────────────────────────┤
│ A - Atomicity                       │    │ BA - Basically Available            │
│     Giao dịch hoàn thành hoặc       │    │      Hệ thống luôn sẵn sàng        │
│     không có gì xảy ra              │    │      phản hồi (có thể không         │
│                                     │    │      phải data mới nhất)            │
│ C - Consistency                     │    │                                     │
│     Dữ liệu luôn hợp lệ             │    │ S - Soft state                      │
│     sau mỗi giao dịch               │    │     Trạng thái có thể thay đổi      │
│                                     │    │     theo thời gian                  │
│ I - Isolation                       │    │                                     │
│     Các giao dịch độc lập           │    │ E - Eventual consistency            │
│     không ảnh hưởng nhau            │    │     Dữ liệu sẽ nhất quán            │
│                                     │    │     sau một khoảng thời gian        │
│ D - Durability                      │    │                                     │
│     Dữ liệu được lưu vĩnh viễn      │    │                                     │
└─────────────────────────────────────┘    └─────────────────────────────────────┘

Ví dụ ACID: Chuyển tiền ngân hàng - PHẢI đảm bảo cả 2 tài khoản được cập nhật
Ví dụ BASE: Like Facebook - Không sao nếu số like hiển thị chậm vài giây
```

### 📈 Scaling

```
VERTICAL SCALING (SQL)                 HORIZONTAL SCALING (NoSQL)
        ↑
   ┌────┴────┐                         ┌─────┐ ┌─────┐ ┌─────┐
   │ Server  │                         │Node1│ │Node2│ │Node3│
   │ (Bigger)│                         └──┬──┘ └──┬──┘ └──┬──┘
   │ 64GB RAM│                            │      │      │
   │ 16 CPU  │                         ┌──┴──────┴──────┴──┐
   └─────────┘                         │   Load Balancer   │
        ↑                              └───────────────────┘
   ┌────┴────┐
   │ Server  │                         ✓ Thêm node khi cần
   │(Original)│                        ✓ Dữ liệu phân tán
   │ 16GB RAM│                         ✓ Fault tolerance
   │ 4 CPU   │                         ✓ Chi phí thấp hơn
   └─────────┘

✗ Giới hạn phần cứng
✗ Downtime khi nâng cấp
✗ Chi phí cao
```

---

## 3. Hiệu suất Query - Cái nào nhanh hơn?

### ⚡ Câu trả lời ngắn gọn

> **Không có câu trả lời tuyệt đối** - Tùy thuộc vào:
>
> - Loại truy vấn
> - Cấu trúc dữ liệu
> - Khối lượng dữ liệu
> - Cách thiết kế schema

### 📊 So sánh theo loại truy vấn

```
┌────────────────────────────────────────────────────────────────────────┐
│                    HIỆU SUẤT THEO LOẠI TRUY VẤN                        │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  1. ĐỌC ĐƠN GIẢN (Read by ID/Key)                                     │
│     ════════════════════════════                                       │
│     NoSQL  ████████████████████████████████████  NHANH HƠN            │
│     SQL    ████████████████████████                                    │
│                                                                        │
│     → NoSQL: O(1) với Key-Value store                                  │
│     → SQL: Cần index lookup                                            │
│                                                                        │
│  2. TRUY VẤN PHỨC TẠP (Multiple JOINs)                                │
│     ═══════════════════════════════════                                │
│     SQL    ████████████████████████████████████  NHANH HƠN            │
│     NoSQL  ████████████████                                            │
│                                                                        │
│     → SQL: Tối ưu hóa JOIN với query planner                          │
│     → NoSQL: Phải query nhiều lần hoặc denormalize                    │
│                                                                        │
│  3. GHI DỮ LIỆU (Write-heavy)                                         │
│     ═════════════════════════                                          │
│     NoSQL  ████████████████████████████████████  NHANH HƠN            │
│     SQL    ██████████████████████                                      │
│                                                                        │
│     → NoSQL: Không cần validate FK, không lock                        │
│     → SQL: ACID overhead, index maintenance                           │
│                                                                        │
│  4. AGGREGATION (SUM, COUNT, AVG)                                     │
│     ═════════════════════════════                                      │
│     SQL    ████████████████████████████████████  NHANH HƠN            │
│     NoSQL  ████████████████████                                        │
│                                                                        │
│     → SQL: Tối ưu hóa sẵn cho aggregation                             │
│     → NoSQL: Cần MapReduce hoặc aggregation pipeline                  │
│                                                                        │
│  5. TÌM KIẾM FULL-TEXT                                                │
│     ════════════════════                                               │
│     Tương đương (cả 2 cần search engine như Elasticsearch)            │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### 🔍 Phân tích chi tiết

#### Trường hợp NoSQL NHANH hơn:

```javascript
// 1. Đọc document đơn lẻ - MongoDB
db.users.findOne({ _id: ObjectId("...") })  // O(1)

// 2. Dữ liệu đã được denormalize (embedded)
// Một query lấy được tất cả
{
  "_id": 1,
  "name": "John",
  "orders": [                    // Embedded - không cần JOIN
    { "product": "iPhone", "price": 999 },
    { "product": "MacBook", "price": 1999 }
  ]
}

// 3. Write-heavy workload
// Không cần check FK constraints
// Không cần maintain nhiều indexes
db.logs.insertMany([...millions of documents...])
```

#### Trường hợp SQL NHANH hơn:

```sql
-- 1. Query với nhiều điều kiện và JOIN
SELECT
    u.name,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
WHERE u.created_at > '2024-01-01'
  AND o.status = 'completed'
GROUP BY u.id
HAVING total_spent > 1000
ORDER BY total_spent DESC;

-- Query planner tối ưu hóa execution plan
-- Sử dụng indexes hiệu quả

-- 2. Transactions phức tạp
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
INSERT INTO transactions (from_id, to_id, amount) VALUES (1, 2, 100);
COMMIT;
-- ACID đảm bảo data integrity
```

### 📉 Benchmark thực tế (Tham khảo)

```
┌─────────────────────────────────────────────────────────────────┐
│  Benchmark: 1 triệu records                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Simple Read (by ID):                                           │
│  ├─ Redis:      0.1ms   ████                                   │
│  ├─ MongoDB:    0.5ms   ████████                               │
│  └─ PostgreSQL: 1.2ms   ████████████████                       │
│                                                                 │
│  Complex Query (3 JOINs + aggregation):                        │
│  ├─ PostgreSQL: 45ms    ████████                               │
│  ├─ MongoDB:    120ms   ████████████████████████               │
│  └─ Redis:      N/A (không hỗ trợ)                             │
│                                                                 │
│  Bulk Insert (100K records):                                    │
│  ├─ MongoDB:    2.1s    ████████                               │
│  ├─ Redis:      3.5s    ████████████                           │
│  └─ PostgreSQL: 8.2s    ████████████████████████████           │
│                                                                 │
│  ⚠️ Lưu ý: Kết quả phụ thuộc vào cấu hình, hardware, indexes   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Khi nào dùng SQL vs NoSQL

### ✅ Chọn SQL khi:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         CHỌN SQL KHI                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 💰 HỆ THỐNG TÀI CHÍNH / NGÂN HÀNG                                  │
│     ─────────────────────────────────                                   │
│     • Cần ACID transactions                                             │
│     • Không được phép mất dữ liệu                                       │
│     • Cần audit trail chính xác                                         │
│     Ví dụ: Banking, Payment, Accounting                                 │
│                                                                         │
│  2. 📊 DỮ LIỆU CÓ QUAN HỆ PHỨC TẠP                                     │
│     ─────────────────────────────────                                   │
│     • Nhiều bảng liên kết với nhau                                      │
│     • Cần JOIN thường xuyên                                             │
│     • Cần đảm bảo data integrity                                        │
│     Ví dụ: ERP, CRM, Inventory Management                               │
│                                                                         │
│  3. 📈 BÁO CÁO & PHÂN TÍCH                                             │
│     ─────────────────────────                                           │
│     • Complex queries với GROUP BY, HAVING                              │
│     • Aggregations (SUM, AVG, COUNT)                                    │
│     • Ad-hoc queries                                                    │
│     Ví dụ: Business Intelligence, Data Warehouse                        │
│                                                                         │
│  4. 🏢 ỨNG DỤNG TRUYỀN THỐNG                                           │
│     ─────────────────────────                                           │
│     • Cấu trúc dữ liệu ổn định, ít thay đổi                            │
│     • Team quen thuộc với SQL                                           │
│     • Cần mature ecosystem                                              │
│     Ví dụ: E-commerce, HR Systems, Legacy Apps                          │
│                                                                         │
│  5. 🔒 YÊU CẦU CONSISTENCY CAO                                         │
│     ─────────────────────────────                                       │
│     • Dữ liệu phải chính xác tại mọi thời điểm                         │
│     • Không chấp nhận eventual consistency                              │
│     Ví dụ: Booking systems, Ticketing                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### ✅ Chọn NoSQL khi:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        CHỌN NoSQL KHI                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 📱 ỨNG DỤNG REAL-TIME / HIGH TRAFFIC                               │
│     ────────────────────────────────────                                │
│     • Millions of concurrent users                                      │
│     • Cần response time cực thấp                                        │
│     • Write-heavy workloads                                             │
│     Ví dụ: Social Media, Gaming, Chat Apps                              │
│                                                                         │
│  2. 📝 DỮ LIỆU PHI CẤU TRÚC / BÁN CẤU TRÚC                            │
│     ──────────────────────────────────────                              │
│     • Schema thay đổi thường xuyên                                      │
│     • Mỗi document có thể khác nhau                                     │
│     • JSON/XML data                                                     │
│     Ví dụ: CMS, Product Catalogs, User Profiles                         │
│                                                                         │
│  3. 🌐 HỆ THỐNG PHÂN TÁN / GLOBAL                                      │
│     ─────────────────────────────────                                   │
│     • Multi-region deployment                                           │
│     • Cần horizontal scaling                                            │
│     • High availability quan trọng hơn consistency                      │
│     Ví dụ: CDN, Global Apps, IoT                                        │
│                                                                         │
│  4. 📊 BIG DATA / LOGGING                                              │
│     ─────────────────────────                                           │
│     • Terabytes/Petabytes of data                                       │
│     • Time-series data                                                  │
│     • Log aggregation                                                   │
│     Ví dụ: Analytics, Monitoring, Log Management                        │
│                                                                         │
│  5. 🚀 RAPID DEVELOPMENT / PROTOTYPING                                 │
│     ────────────────────────────────────                                │
│     • Schema chưa xác định rõ                                           │
│     • Cần iterate nhanh                                                 │
│     • MVP development                                                   │
│     Ví dụ: Startups, POC, Hackathons                                    │
│                                                                         │
│  6. 💾 CACHING / SESSION STORAGE                                       │
│     ────────────────────────────────                                    │
│     • Key-value lookups                                                 │
│     • Temporary data                                                    │
│     • High-speed access                                                 │
│     Ví dụ: Redis for caching, Session management                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 🎯 Decision Matrix

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        DECISION MATRIX                                   │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Yêu cầu                              │  SQL  │ NoSQL │ Gợi ý            │
│  ─────────────────────────────────────┼───────┼───────┼─────────────────│
│  ACID Transactions bắt buộc           │  ✓✓✓  │   ✗   │ SQL              │
│  Schema thay đổi thường xuyên         │   ✗   │  ✓✓✓  │ NoSQL            │
│  Complex JOINs                        │  ✓✓✓  │   ✗   │ SQL              │
│  Horizontal scaling                   │   ✗   │  ✓✓✓  │ NoSQL            │
│  Strong consistency                   │  ✓✓✓  │   ✓   │ SQL              │
│  High write throughput                │   ✓   │  ✓✓✓  │ NoSQL            │
│  Complex aggregations                 │  ✓✓✓  │   ✓   │ SQL              │
│  Hierarchical/nested data             │   ✗   │  ✓✓✓  │ NoSQL            │
│  Mature tooling/ecosystem             │  ✓✓✓  │   ✓   │ SQL              │
│  Real-time analytics                  │   ✓   │  ✓✓✓  │ NoSQL            │
│                                                                          │
│  ✓✓✓ = Excellent   ✓ = Good   ✗ = Poor                                  │
└──────────────────────────────────────────────────────────────────────────┘
```

### 🔀 Hybrid Approach (Polyglot Persistence)

Nhiều hệ thống hiện đại sử dụng CẢ HAI:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    KIẾN TRÚC HYBRID                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│                         ┌─────────────┐                                 │
│                         │   Client    │                                 │
│                         └──────┬──────┘                                 │
│                                │                                        │
│                         ┌──────▼──────┐                                 │
│                         │   API Layer │                                 │
│                         └──────┬──────┘                                 │
│                                │                                        │
│         ┌──────────────────────┼──────────────────────┐                │
│         │                      │                      │                │
│         ▼                      ▼                      ▼                │
│  ┌─────────────┐       ┌─────────────┐       ┌─────────────┐          │
│  │ PostgreSQL  │       │   MongoDB   │       │    Redis    │          │
│  │             │       │             │       │             │          │
│  │ • Users     │       │ • Products  │       │ • Sessions  │          │
│  │ • Orders    │       │ • Reviews   │       │ • Cache     │          │
│  │ • Payments  │       │ • Logs      │       │ • Real-time │          │
│  │             │       │             │       │             │          │
│  │ Transactions│       │ Flexibility │       │   Speed     │          │
│  │ Consistency │       │ Scalability │       │   Caching   │          │
│  └─────────────┘       └─────────────┘       └─────────────┘          │
│                                                                         │
│  Ví dụ thực tế:                                                        │
│  • Uber: PostgreSQL (trips) + Redis (caching) + Cassandra (logs)       │
│  • Netflix: MySQL (billing) + Cassandra (viewing history)              │
│  • Airbnb: MySQL (bookings) + Redis (search) + Elasticsearch           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Ví dụ thực tế

### 📱 Case Study: Social Media App

#### Yêu cầu:

- Millions of users
- Posts, comments, likes
- Real-time feed
- User profiles với nhiều fields tùy chọn

#### Giải pháp: NoSQL (MongoDB) + Redis

```javascript
// MongoDB - User Profile (flexible schema)
{
  "_id": ObjectId("..."),
  "username": "john_doe",
  "email": "john@email.com",
  "profile": {
    "bio": "Developer",
    "avatar": "url...",
    "social_links": {           // Optional, flexible
      "twitter": "@john",
      "github": "johndoe"
    },
    "interests": ["coding", "music"]  // Array - easy in NoSQL
  },
  "followers_count": 1500,
  "following_count": 200
}

// MongoDB - Post (embedded comments for speed)
{
  "_id": ObjectId("..."),
  "user_id": ObjectId("..."),
  "content": "Hello World!",
  "likes_count": 42,
  "comments": [                 // Embedded - 1 query lấy tất cả
    { "user": "jane", "text": "Nice!", "created_at": "..." },
    { "user": "bob", "text": "Cool!", "created_at": "..." }
  ],
  "created_at": ISODate("...")
}

// Redis - Caching & Real-time
SET user:123:feed "[post_ids...]"     // Cached feed
ZADD trending:posts 100 "post:456"    // Sorted set for trending
PUBLISH notifications:user:123 "..."   // Real-time notifications
```

**Tại sao NoSQL?**

- Schema linh hoạt cho user profiles
- Embedded documents giảm JOINs
- Horizontal scaling cho millions of users
- Redis cho real-time features

---

### 💰 Case Study: Banking System

#### Yêu cầu:

- Chuyển tiền giữa các tài khoản
- Lịch sử giao dịch chính xác
- Audit trail
- Không được mất tiền!

#### Giải pháp: SQL (PostgreSQL)

```sql
-- Schema chặt chẽ
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(id),
    account_number VARCHAR(20) UNIQUE NOT NULL,
    balance DECIMAL(15, 2) NOT NULL DEFAULT 0,
    currency VARCHAR(3) NOT NULL DEFAULT 'VND',
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT positive_balance CHECK (balance >= 0)
);

CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    from_account_id INT REFERENCES accounts(id),
    to_account_id INT REFERENCES accounts(id),
    amount DECIMAL(15, 2) NOT NULL,
    type VARCHAR(20) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,

    CONSTRAINT positive_amount CHECK (amount > 0)
);

-- ACID Transaction cho chuyển tiền
BEGIN;

-- Lock rows để tránh race condition
SELECT * FROM accounts WHERE id IN (1, 2) FOR UPDATE;

-- Kiểm tra số dư
UPDATE accounts
SET balance = balance - 1000000
WHERE id = 1 AND balance >= 1000000;

-- Nếu update thành công (1 row affected)
UPDATE accounts
SET balance = balance + 1000000
WHERE id = 2;

-- Ghi log transaction
INSERT INTO transactions (from_account_id, to_account_id, amount, type, status)
VALUES (1, 2, 1000000, 'transfer', 'completed');

COMMIT;
-- Nếu bất kỳ bước nào fail → ROLLBACK tự động
```

**Tại sao SQL?**

- ACID đảm bảo tiền không bị mất
- Constraints đảm bảo data integrity
- Transactions cho operations phức tạp
- Audit trail với foreign keys

---

### 🛒 Case Study: E-commerce (Hybrid)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    E-COMMERCE ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐                                                   │
│  │   PostgreSQL    │  ← Orders, Payments, Users                        │
│  │   (Primary DB)  │    ACID transactions                              │
│  └────────┬────────┘    Strong consistency                             │
│           │                                                             │
│  ┌────────▼────────┐                                                   │
│  │    MongoDB      │  ← Product Catalog                                │
│  │  (Product DB)   │    Flexible attributes                            │
│  └────────┬────────┘    (size, color, specs vary by category)          │
│           │                                                             │
│  ┌────────▼────────┐                                                   │
│  │     Redis       │  ← Shopping Cart, Sessions                        │
│  │    (Cache)      │    Product cache                                  │
│  └────────┬────────┘    Real-time inventory                            │
│           │                                                             │
│  ┌────────▼────────┐                                                   │
│  │  Elasticsearch  │  ← Product Search                                 │
│  │   (Search)      │    Full-text search                               │
│  └─────────────────┘    Faceted navigation                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

```javascript
// MongoDB - Product với attributes linh hoạt
// Điện thoại
{
  "_id": "phone-001",
  "name": "iPhone 15",
  "category": "phones",
  "price": 999,
  "attributes": {
    "screen_size": "6.1 inch",
    "storage": "128GB",
    "color": "Blue",
    "5g_support": true
  }
}

// Quần áo - attributes hoàn toàn khác
{
  "_id": "shirt-001",
  "name": "T-Shirt Basic",
  "category": "clothing",
  "price": 29.99,
  "attributes": {
    "size": ["S", "M", "L", "XL"],
    "color": ["White", "Black"],
    "material": "Cotton 100%"
  }
}
```

```sql
-- PostgreSQL - Orders (cần ACID)
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    status VARCHAR(20) NOT NULL,
    total_amount DECIMAL(12, 2) NOT NULL,
    shipping_address JSONB,  -- PostgreSQL hỗ trợ JSON
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Transaction khi đặt hàng
BEGIN;
INSERT INTO orders (...) VALUES (...);
INSERT INTO order_items (...) VALUES (...);
UPDATE inventory SET stock = stock - 1 WHERE product_id = '...';
INSERT INTO payments (...) VALUES (...);
COMMIT;
```

---

## 📌 Tổng kết

| Câu hỏi                    | Trả lời                                                                                   |
| -------------------------- | ----------------------------------------------------------------------------------------- |
| **Cái nào nhanh hơn?**     | Tùy use case. NoSQL nhanh hơn cho simple reads/writes. SQL nhanh hơn cho complex queries. |
| **Cái nào tốt hơn?**       | Không có "tốt hơn" tuyệt đối. Chọn đúng tool cho đúng job.                                |
| **Nên học cái nào trước?** | SQL - vì fundamental và được dùng rộng rãi hơn.                                           |
| **Có thể dùng cả hai?**    | Có! Polyglot persistence là xu hướng hiện đại.                                            |

### 🎯 Quick Decision Guide

```
Cần ACID + Complex queries + Relationships → SQL
Cần Flexibility + Scale + Speed → NoSQL
Cần cả hai → Hybrid approach
Không chắc → Bắt đầu với SQL, migrate sau nếu cần
```

---

## 🔗 Tài liệu tham khảo

- [MongoDB vs PostgreSQL](https://www.mongodb.com/compare/mongodb-postgresql)
- [CAP Theorem Explained](https://www.ibm.com/topics/cap-theorem)
- [Polyglot Persistence](https://martinfowler.com/bliki/PolyglotPersistence.html)
