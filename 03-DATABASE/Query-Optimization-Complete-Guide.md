# Tối Ưu Truy Vấn Database - SQL & MongoDB

## Mục Lục

1. [SQL Query Optimization](#sql-query-optimization)
2. [MongoDB Query Optimization](#mongodb-query-optimization)
3. [So Sánh SQL vs MongoDB](#so-sánh-sql-vs-mongodb)
4. [Best Practices](#best-practices)

---

# SQL Query Optimization

## 1. Indexing - Đánh Index

### Khái Niệm

Index giống như mục lục sách, giúp database tìm dữ liệu nhanh hơn mà không cần scan toàn bộ bảng.

### Các Loại Index

```sql
-- 1. Single Column Index
CREATE INDEX idx_users_email ON users(email);

-- 2. Composite Index (Multi-column)
CREATE INDEX idx_users_name_age ON users(last_name, first_name, age);

-- 3. Unique Index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- 4. Partial Index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- 5. Full-Text Index (MySQL)
CREATE FULLTEXT INDEX idx_posts_content ON posts(title, content);

-- 6. Covering Index (Include columns)
CREATE INDEX idx_users_covering ON users(email) INCLUDE (name, age);
```

### Khi Nào Nên Tạo Index?

```sql
-- ✅ Nên tạo index cho:

-- 1. Primary Key (tự động)
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(255)
);

-- 2. Foreign Key
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 3. Columns trong WHERE clause
SELECT * FROM users WHERE email = 'john@example.com';
-- → CREATE INDEX idx_users_email ON users(email);

-- 4. Columns trong JOIN
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id;
-- → CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 5. Columns trong ORDER BY
SELECT * FROM products ORDER BY created_at DESC;
-- → CREATE INDEX idx_products_created ON products(created_at);

-- 6. Columns trong GROUP BY
SELECT category, COUNT(*) FROM products GROUP BY category;
-- → CREATE INDEX idx_products_category ON products(category);
```

### Khi Nào KHÔNG Nên Tạo Index?

```sql
-- ❌ Không nên tạo index cho:

-- 1. Bảng nhỏ (< 1000 rows)
-- Full table scan nhanh hơn

-- 2. Columns có ít giá trị distinct (low cardinality)
CREATE INDEX idx_users_gender ON users(gender); -- BAD!
-- gender chỉ có 2-3 giá trị

-- 3. Columns thường xuyên UPDATE
-- Index phải rebuild mỗi lần update

-- 4. Bảng có nhiều INSERT/UPDATE/DELETE
-- Index làm chậm write operations
```

### Index Selectivity

```sql
-- Kiểm tra selectivity (càng cao càng tốt)
SELECT
    COUNT(DISTINCT email) / COUNT(*) as selectivity
FROM users;
-- > 0.9: Excellent
-- 0.5-0.9: Good
-- < 0.5: Poor

-- Ví dụ:
-- email: 10000 distinct / 10000 total = 1.0 (Perfect)
-- gender: 2 distinct / 10000 total = 0.0002 (Poor)
```

### Composite Index Order

```sql
-- ❌ BAD: Sai thứ tự
CREATE INDEX idx_users_bad ON users(age, last_name, first_name);

SELECT * FROM users
WHERE last_name = 'Smith' AND first_name = 'John';
-- Index không được sử dụng!

-- ✅ GOOD: Đúng thứ tự
CREATE INDEX idx_users_good ON users(last_name, first_name, age);

-- Rule: Đặt columns có selectivity cao nhất trước
-- Rule: Đặt columns trong WHERE trước ORDER BY
```

---

## 2. Query Optimization Techniques

### SELECT Optimization

```sql
-- ❌ BAD: SELECT *
SELECT * FROM users WHERE id = 1;
-- Lấy tất cả columns, tốn bandwidth

-- ✅ GOOD: Chỉ lấy columns cần thiết
SELECT id, name, email FROM users WHERE id = 1;

-- ❌ BAD: SELECT DISTINCT không cần thiết
SELECT DISTINCT name FROM users;

-- ✅ GOOD: Dùng GROUP BY nếu cần aggregate
SELECT name FROM users GROUP BY name;
```

### WHERE Clause Optimization

```sql
-- ❌ BAD: Function trên indexed column
SELECT * FROM users WHERE YEAR(created_at) = 2024;
-- Index không được sử dụng!

-- ✅ GOOD: Dùng range
SELECT * FROM users
WHERE created_at >= '2024-01-01'
  AND created_at < '2025-01-01';

-- ❌ BAD: Leading wildcard
SELECT * FROM users WHERE email LIKE '%@gmail.com';
-- Index không được sử dụng!

-- ✅ GOOD: Trailing wildcard
SELECT * FROM users WHERE email LIKE 'john%';

-- ❌ BAD: OR với columns khác nhau
SELECT * FROM users WHERE name = 'John' OR email = 'john@example.com';

-- ✅ GOOD: Dùng UNION
SELECT * FROM users WHERE name = 'John'
UNION
SELECT * FROM users WHERE email = 'john@example.com';

-- ❌ BAD: NOT IN với subquery lớn
SELECT * FROM users WHERE id NOT IN (SELECT user_id FROM banned_users);

-- ✅ GOOD: LEFT JOIN với NULL check
SELECT u.* FROM users u
LEFT JOIN banned_users b ON u.id = b.user_id
WHERE b.user_id IS NULL;
```

### JOIN Optimization

```sql
-- ❌ BAD: Cartesian product
SELECT * FROM users, orders;
-- Tạo ra users_count × orders_count rows!

-- ✅ GOOD: Proper JOIN
SELECT * FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- ✅ GOOD: Join với indexed columns
-- Đảm bảo cả 2 columns đều có index
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_id ON users(id); -- Primary key tự động có

-- ✅ GOOD: Join order (nhỏ trước, lớn sau)
SELECT * FROM small_table s
JOIN large_table l ON s.id = l.small_id;

-- ❌ BAD: Multiple JOINs không cần thiết
SELECT u.name, o.total, p.name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;

-- ✅ GOOD: Chỉ JOIN khi cần
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id;
```

### Subquery Optimization

```sql
-- ❌ BAD: Correlated subquery
SELECT u.name,
    (SELECT COUNT(*) FROM orders WHERE user_id = u.id) as order_count
FROM users u;
-- Subquery chạy cho mỗi row!

-- ✅ GOOD: JOIN với GROUP BY
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- ❌ BAD: IN với subquery lớn
SELECT * FROM products
WHERE category_id IN (SELECT id FROM categories WHERE is_active = true);

-- ✅ GOOD: JOIN
SELECT p.* FROM products p
INNER JOIN categories c ON p.category_id = c.id
WHERE c.is_active = true;

-- ✅ GOOD: EXISTS (nhanh hơn IN)
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id AND o.status = 'completed'
);
```

### LIMIT & OFFSET Optimization

```sql
-- ❌ BAD: OFFSET lớn
SELECT * FROM products
ORDER BY created_at DESC
LIMIT 10 OFFSET 100000;
-- Phải scan 100010 rows!

-- ✅ GOOD: Keyset pagination
SELECT * FROM products
WHERE created_at < '2024-01-01 00:00:00'
ORDER BY created_at DESC
LIMIT 10;

-- ✅ GOOD: Cursor-based pagination
SELECT * FROM products
WHERE id > 12345
ORDER BY id
LIMIT 10;
```

---

## 3. Aggregate Optimization

```sql
-- ❌ BAD: COUNT(*) trên bảng lớn
SELECT COUNT(*) FROM users;
-- Scan toàn bộ bảng!

-- ✅ GOOD: Approximate count (PostgreSQL)
SELECT reltuples::bigint AS estimate
FROM pg_class
WHERE relname = 'users';

-- ❌ BAD: Multiple COUNT
SELECT
    (SELECT COUNT(*) FROM users WHERE status = 'active') as active,
    (SELECT COUNT(*) FROM users WHERE status = 'inactive') as inactive;

-- ✅ GOOD: Single query với CASE
SELECT
    COUNT(CASE WHEN status = 'active' THEN 1 END) as active,
    COUNT(CASE WHEN status = 'inactive' THEN 1 END) as inactive
FROM users;

-- ✅ GOOD: Covering index cho aggregate
CREATE INDEX idx_users_status_covering ON users(status) INCLUDE (id);

SELECT status, COUNT(*) FROM users GROUP BY status;
```

---

## 4. Query Execution Plan

### EXPLAIN ANALYZE

```sql
-- PostgreSQL
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'john@example.com';

-- Output:
-- Seq Scan on users  (cost=0.00..180.00 rows=1 width=100) (actual time=0.050..5.234 rows=1 loops=1)
--   Filter: (email = 'john@example.com'::text)
-- Planning Time: 0.123 ms
-- Execution Time: 5.345 ms

-- Sau khi tạo index:
CREATE INDEX idx_users_email ON users(email);

EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'john@example.com';

-- Output:
-- Index Scan using idx_users_email on users  (cost=0.29..8.30 rows=1 width=100) (actual time=0.015..0.016 rows=1 loops=1)
--   Index Cond: (email = 'john@example.com'::text)
-- Planning Time: 0.089 ms
-- Execution Time: 0.032 ms
```

### Đọc Execution Plan

```sql
-- Các loại scan (từ nhanh đến chậm):
-- 1. Index Seek/Scan: Tốt nhất
-- 2. Index Scan: Tốt
-- 3. Table Scan: Chấp nhận được với bảng nhỏ
-- 4. Full Table Scan: Tệ với bảng lớn

-- Cost: Ước tính chi phí
-- Rows: Số rows ước tính
-- Width: Kích thước row (bytes)
-- Actual time: Thời gian thực tế
-- Loops: Số lần lặp
```

---

## 5. Database Design Optimization

### Normalization vs Denormalization

```sql
-- Normalized (3NF)
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    total DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Query cần JOIN
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id;

-- Denormalized (cho read-heavy)
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    user_name VARCHAR(100), -- Denormalized!
    total DECIMAL(10,2)
);

-- Query không cần JOIN
SELECT user_name, total FROM orders;
```

### Partitioning

```sql
-- Range Partitioning (PostgreSQL)
CREATE TABLE orders (
    id SERIAL,
    user_id INT,
    total DECIMAL(10,2),
    created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- Query chỉ scan partition cần thiết
SELECT * FROM orders WHERE created_at >= '2024-01-01';
-- Chỉ scan orders_2024
```

### Materialized Views

```sql
-- Tạo materialized view cho query phức tạp
CREATE MATERIALIZED VIEW user_stats AS
SELECT
    u.id,
    u.name,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- Tạo index trên materialized view
CREATE INDEX idx_user_stats_id ON user_stats(id);

-- Query nhanh
SELECT * FROM user_stats WHERE id = 1;

-- Refresh khi cần
REFRESH MATERIALIZED VIEW user_stats;

-- Refresh concurrent (không block reads)
REFRESH MATERIALIZED VIEW CONCURRENTLY user_stats;
```

---

## 6. Connection & Transaction Optimization

### Connection Pooling

```javascript
// Node.js với pg
const { Pool } = require("pg");

// ❌ BAD: Tạo connection mới mỗi query
const client = new Client();
await client.connect();
await client.query("SELECT * FROM users");
await client.end();

// ✅ GOOD: Dùng connection pool
const pool = new Pool({
  max: 20, // Max connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

const result = await pool.query("SELECT * FROM users");
```

### Transaction Optimization

```sql
-- ❌ BAD: Long transaction
BEGIN;
SELECT * FROM users; -- Slow query
UPDATE orders SET status = 'processed'; -- Many rows
-- ... more queries ...
COMMIT;

-- ✅ GOOD: Short transaction
BEGIN;
UPDATE orders SET status = 'processed' WHERE id = 123;
COMMIT;

-- ✅ GOOD: Read-only transaction
BEGIN TRANSACTION READ ONLY;
SELECT * FROM users;
COMMIT;
```

---

# MongoDB Query Optimization

## 1. Indexing trong MongoDB

### Các Loại Index

```javascript
// 1. Single Field Index
db.users.createIndex({ email: 1 }); // 1: ascending, -1: descending

// 2. Compound Index
db.users.createIndex({ lastName: 1, firstName: 1, age: 1 });

// 3. Multikey Index (cho arrays)
db.users.createIndex({ tags: 1 });

// 4. Text Index (full-text search)
db.posts.createIndex({ title: "text", content: "text" });

// 5. Geospatial Index
db.places.createIndex({ location: "2dsphere" });

// 6. Hashed Index (cho sharding)
db.users.createIndex({ _id: "hashed" });

// 7. Wildcard Index
db.products.createIndex({ "attributes.$**": 1 });

// 8. Partial Index
db.users.createIndex({ email: 1 }, { partialFilterExpression: { isActive: true } });

// 9. TTL Index (tự động xóa documents)
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }, // 1 hour
);

// 10. Unique Index
db.users.createIndex({ username: 1 }, { unique: true });
```

### Index Properties

```javascript
// Sparse Index (chỉ index documents có field)
db.users.createIndex({ phone: 1 }, { sparse: true });

// Case-insensitive Index
db.users.createIndex({ email: 1 }, { collation: { locale: "en", strength: 2 } });

// Background Index (không block operations)
db.users.createIndex({ email: 1 }, { background: true });
```

### Xem và Quản Lý Index

```javascript
// Xem tất cả indexes
db.users.getIndexes();

// Xem index usage stats
db.users.aggregate([{ $indexStats: {} }]);

// Drop index
db.users.dropIndex("email_1");
db.users.dropIndex({ email: 1 });

// Drop tất cả indexes (trừ _id)
db.users.dropIndexes();

// Rebuild indexes
db.users.reIndex();
```

---

## 2. Query Optimization

### Find Optimization

```javascript
// ❌ BAD: Không có index
db.users.find({ email: "john@example.com" });
// Collection scan!

// ✅ GOOD: Có index
db.users.createIndex({ email: 1 });
db.users.find({ email: "john@example.com" });

// ❌ BAD: Lấy tất cả fields
db.users.find({ email: "john@example.com" });

// ✅ GOOD: Projection (chỉ lấy fields cần thiết)
db.users.find({ email: "john@example.com" }, { name: 1, email: 1, _id: 0 });

// ❌ BAD: Regex không có anchor
db.users.find({ email: /gmail.com/ });

// ✅ GOOD: Regex với anchor
db.users.find({ email: /^john/ });

// ❌ BAD: $where operator
db.users.find({
  $where: "this.age > 18",
});

// ✅ GOOD: Standard operators
db.users.find({ age: { $gt: 18 } });
```

### Covered Queries

```javascript
// Covered query: Tất cả fields trong index
db.users.createIndex({ email: 1, name: 1, age: 1 });

// ✅ Query được cover hoàn toàn bởi index
db.users.find({ email: "john@example.com" }, { email: 1, name: 1, age: 1, _id: 0 });
// Không cần đọc documents!

// ❌ Không covered (có _id)
db.users.find({ email: "john@example.com" }, { email: 1, name: 1 });
```

### Sort Optimization

```javascript
// ❌ BAD: Sort không có index
db.users.find().sort({ createdAt: -1 });
// In-memory sort, limit 32MB!

// ✅ GOOD: Sort với index
db.users.createIndex({ createdAt: -1 });
db.users.find().sort({ createdAt: -1 });

// ✅ GOOD: Compound index cho filter + sort
db.users.createIndex({ status: 1, createdAt: -1 });
db.users.find({ status: "active" }).sort({ createdAt: -1 });

// ❌ BAD: Sort direction khác với index
db.users.createIndex({ lastName: 1, firstName: 1 });
db.users.find().sort({ lastName: 1, firstName: -1 });
// Không dùng được index!

// ✅ GOOD: Sort direction giống index
db.users.find().sort({ lastName: 1, firstName: 1 });
```

### Limit & Skip Optimization

```javascript
// ❌ BAD: Skip lớn
db.products.find().skip(100000).limit(10);
// Phải scan 100010 documents!

// ✅ GOOD: Range-based pagination
db.products.find({ _id: { $gt: lastSeenId } }).limit(10);

// ✅ GOOD: Indexed field pagination
db.products.createIndex({ createdAt: -1 });
db.products
  .find({ createdAt: { $lt: lastSeenDate } })
  .sort({ createdAt: -1 })
  .limit(10);
```

---

## 3. Aggregation Optimization

### Pipeline Optimization

```javascript
// ❌ BAD: $match sau $sort
db.orders.aggregate([{ $sort: { total: -1 } }, { $match: { status: "completed" } }, { $limit: 10 }]);

// ✅ GOOD: $match trước (dùng index)
db.orders.aggregate([{ $match: { status: "completed" } }, { $sort: { total: -1 } }, { $limit: 10 }]);

// ❌ BAD: $project sau $group
db.orders.aggregate([
  { $group: { _id: "$userId", total: { $sum: "$amount" } } },
  { $project: { userId: "$_id", total: 1, _id: 0 } },
]);

// ✅ GOOD: $project trước $group (giảm data)
db.orders.aggregate([
  { $project: { userId: 1, amount: 1 } },
  { $group: { _id: "$userId", total: { $sum: "$amount" } } },
]);

// ✅ GOOD: Dùng $limit sớm
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $sort: { createdAt: -1 } },
  { $limit: 100 }, // Limit sớm
  {
    $lookup: {
      /* ... */
    },
  },
  { $unwind: "$items" },
]);
```

### Index Usage trong Aggregation

```javascript
// ✅ GOOD: $match đầu tiên dùng index
db.orders.createIndex({ status: 1, createdAt: -1 });

db.orders.aggregate([
  { $match: { status: "completed" } }, // Dùng index
  { $sort: { createdAt: -1 } }, // Dùng index
  { $group: { _id: "$userId", total: { $sum: "$amount" } } },
]);

// ❌ BAD: $match với $or không hiệu quả
db.orders.aggregate([{ $match: { $or: [{ status: "completed" }, { status: "pending" }] } }]);

// ✅ GOOD: Dùng $in
db.orders.aggregate([{ $match: { status: { $in: ["completed", "pending"] } } }]);
```

### $lookup Optimization

```javascript
// ❌ BAD: $lookup không có index
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user",
    },
  },
]);

// ✅ GOOD: Tạo index trên foreign field
db.users.createIndex({ _id: 1 }); // _id tự động có index

// ✅ BETTER: $lookup với pipeline và index
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      let: { userId: "$userId" },
      pipeline: [
        { $match: { $expr: { $eq: ["$_id", "$$userId"] } } },
        { $project: { name: 1, email: 1 } }, // Chỉ lấy fields cần thiết
      ],
      as: "user",
    },
  },
]);

// ✅ BEST: Denormalize nếu có thể
// Thay vì $lookup, embed user data vào orders
db.orders.insertOne({
  _id: ObjectId(),
  userId: ObjectId(),
  userName: "John Doe", // Denormalized
  userEmail: "john@example.com", // Denormalized
  total: 100,
});
```

---

## 4. Schema Design Optimization

### Embedding vs Referencing

```javascript
// Embedding (One-to-Few)
// ✅ GOOD: Khi có ít sub-documents và thường query cùng nhau
{
  _id: ObjectId(),
  name: "John Doe",
  addresses: [
    { street: "123 Main St", city: "NYC" },
    { street: "456 Oak Ave", city: "LA" }
  ]
}

// Referencing (One-to-Many)
// ✅ GOOD: Khi có nhiều sub-documents hoặc query riêng
// Users collection
{
  _id: ObjectId("user1"),
  name: "John Doe"
}

// Orders collection
{
  _id: ObjectId(),
  userId: ObjectId("user1"),
  total: 100
}

// Hybrid (Two-Way Referencing)
// ✅ GOOD: Khi cần query từ cả 2 phía
// Users collection
{
  _id: ObjectId("user1"),
  name: "John Doe",
  orderIds: [ObjectId("order1"), ObjectId("order2")]
}

// Orders collection
{
  _id: ObjectId("order1"),
  userId: ObjectId("user1"),
  total: 100
}
```

### Denormalization

```javascript
// ❌ BAD: Normalized (nhiều queries)
// Users
{ _id: 1, name: "John" }

// Posts
{ _id: 1, userId: 1, title: "Hello" }

// Query cần 2 requests
const user = db.users.findOne({ _id: 1 });
const posts = db.posts.find({ userId: 1 });

// ✅ GOOD: Denormalized (1 query)
{
  _id: 1,
  userId: 1,
  userName: "John", // Denormalized
  title: "Hello"
}

// Query chỉ cần 1 request
const posts = db.posts.find({ userId: 1 });
```

### Array Size Limits

```javascript
// ❌ BAD: Array quá lớn
{
  _id: ObjectId(),
  name: "Product",
  reviews: [ /* 10000 reviews */ ] // Document > 16MB!
}

// ✅ GOOD: Bucketing pattern
// Products collection
{
  _id: ObjectId("product1"),
  name: "Product",
  reviewCount: 10000
}

// Reviews collection (bucketed)
{
  _id: ObjectId(),
  productId: ObjectId("product1"),
  bucket: 1,
  reviews: [ /* 100 reviews */ ]
}
```

---

## 5. Explain & Performance Analysis

### Explain Query

```javascript
// Basic explain
db.users.find({ email: "john@example.com" }).explain();

// Execution stats
db.users.find({ email: "john@example.com" }).explain("executionStats");

// All plans
db.users.find({ email: "john@example.com" }).explain("allPlansExecution");

// Đọc explain output
{
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 1,              // Số documents trả về
    "executionTimeMillis": 0,    // Thời gian thực thi
    "totalKeysExamined": 1,      // Số index keys scan
    "totalDocsExamined": 1,      // Số documents scan
    "executionStages": {
      "stage": "FETCH",          // IXSCAN: index scan (good)
                                 // COLLSCAN: collection scan (bad)
      "nReturned": 1,
      "executionTimeMillisEstimate": 0
    }
  }
}

// ✅ GOOD: Index scan
// totalKeysExamined ≈ nReturned
// totalDocsExamined ≈ nReturned

// ❌ BAD: Collection scan
// totalDocsExamined >> nReturned
```

### Profiler

```javascript
// Enable profiler
db.setProfilingLevel(2); // 0: off, 1: slow queries, 2: all queries

// Set slow query threshold
db.setProfilingLevel(1, { slowms: 100 });

// View slow queries
db.system.profile.find().sort({ ts: -1 }).limit(10);

// Analyze slow queries
db.system.profile
  .find({
    millis: { $gt: 100 },
  })
  .sort({ millis: -1 });

// Disable profiler
db.setProfilingLevel(0);
```

### Current Operations

```javascript
// Xem operations đang chạy
db.currentOp();

// Xem slow operations
db.currentOp({
  active: true,
  secs_running: { $gt: 3 },
});

// Kill operation
db.killOp(opId);
```

---

## 6. Write Optimization

### Bulk Operations

```javascript
// ❌ BAD: Multiple single inserts
for (let i = 0; i < 1000; i++) {
  db.users.insertOne({ name: `User ${i}` });
}

// ✅ GOOD: Bulk insert
const users = [];
for (let i = 0; i < 1000; i++) {
  users.push({ name: `User ${i}` });
}
db.users.insertMany(users);

// ✅ GOOD: Bulk write với mixed operations
db.users.bulkWrite(
  [
    { insertOne: { document: { name: "John" } } },
    { updateOne: { filter: { _id: 1 }, update: { $set: { age: 30 } } } },
    { deleteOne: { filter: { _id: 2 } } },
  ],
  { ordered: false },
); // ordered: false = parallel execution
```

### Update Optimization

```javascript
// ❌ BAD: Update toàn bộ document
db.users.updateOne({ _id: 1 }, { name: "John", email: "john@example.com", age: 30 /* ... */ });

// ✅ GOOD: Chỉ update fields cần thiết
db.users.updateOne({ _id: 1 }, { $set: { age: 30 } });

// ✅ GOOD: Atomic operators
db.users.updateOne(
  { _id: 1 },
  {
    $inc: { loginCount: 1 },
    $set: { lastLogin: new Date() },
    $push: { loginHistory: new Date() },
  },
);

// ❌ BAD: Update trong loop
for (let id of userIds) {
  db.users.updateOne({ _id: id }, { $set: { status: "active" } });
}

// ✅ GOOD: Update nhiều documents
db.users.updateMany({ _id: { $in: userIds } }, { $set: { status: "active" } });
```

### Write Concern

```javascript
// Default write concern
db.users.insertOne(
  { name: "John" },
  { writeConcern: { w: 1 } }, // Acknowledge từ primary
);

// Majority write concern (safer)
db.users.insertOne(
  { name: "John" },
  { writeConcern: { w: "majority" } }, // Acknowledge từ majority
);

// No acknowledgment (fastest, riskiest)
db.users.insertOne({ name: "John" }, { writeConcern: { w: 0 } });
```

---

## 7. Connection & Resource Management

### Connection Pooling

```javascript
// Node.js với MongoDB driver
const { MongoClient } = require("mongodb");

// ✅ GOOD: Connection pool
const client = new MongoClient(uri, {
  maxPoolSize: 50,
  minPoolSize: 10,
  maxIdleTimeMS: 30000,
  waitQueueTimeoutMS: 5000,
});

await client.connect();
const db = client.db("mydb");

// Reuse connection
const users = await db.collection("users").find().toArray();
```

### Read Preference

```javascript
// Primary (default)
db.users.find().readPref("primary");

// Secondary (cho read-heavy workloads)
db.users.find().readPref("secondary");

// Nearest (lowest latency)
db.users.find().readPref("nearest");
```

---

# So Sánh SQL vs MongoDB

## Performance Comparison

| Aspect                | SQL             | MongoDB                          |
| --------------------- | --------------- | -------------------------------- |
| **Read Performance**  | Tốt với index   | Tốt với index                    |
| **Write Performance** | Chậm hơn (ACID) | Nhanh hơn (eventual consistency) |
| **Join Performance**  | Tốt (optimized) | Chậm ($lookup)                   |
| **Aggregation**       | Tốt (mature)    | Tốt (flexible)                   |
| **Full-Text Search**  | Có (limited)    | Tốt (text index)                 |
| **Geospatial**        | Có (PostGIS)    | Native support                   |
| **Scalability**       | Vertical        | Horizontal (sharding)            |

## Index Comparison

```sql
-- SQL: Composite Index
CREATE INDEX idx_users ON users(last_name, first_name, age);

-- Query sử dụng index (left-to-right)
SELECT * FROM users WHERE last_name = 'Smith'; -- ✅ Uses index
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John'; -- ✅ Uses index
SELECT * FROM users WHERE first_name = 'John'; -- ❌ Doesn't use index
```

```javascript
// MongoDB: Compound Index
db.users.createIndex({ lastName: 1, firstName: 1, age: 1 });

// Query sử dụng index (prefix rule)
db.users.find({ lastName: "Smith" }); // ✅ Uses index
db.users.find({ lastName: "Smith", firstName: "John" }); // ✅ Uses index
db.users.find({ firstName: "John" }); // ❌ Doesn't use index
```

## Join vs Lookup

```sql
-- SQL: Efficient JOIN
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.id = 1;
-- Fast với proper indexes
```

```javascript
// MongoDB: $lookup (slower)
db.users.aggregate([
  { $match: { _id: 1 } },
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "userId",
      as: "orders",
    },
  },
]);
// Chậm hơn SQL JOIN

// MongoDB: Denormalization (faster)
db.orders.find({ userId: 1 });
// Nhanh hơn nếu embed user data
```

## Transaction Support

```sql
-- SQL: Full ACID transactions
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- Atomic, Consistent, Isolated, Durable
```

```javascript
// MongoDB: Multi-document transactions (4.0+)
const session = client.startSession();
session.startTransaction();

try {
  await db.collection("accounts").updateOne({ _id: 1 }, { $inc: { balance: -100 } }, { session });

  await db.collection("accounts").updateOne({ _id: 2 }, { $inc: { balance: 100 } }, { session });

  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
} finally {
  session.endSession();
}
// Có ACID nhưng performance impact lớn hơn
```

---

# Best Practices

## SQL Best Practices

### 1. Index Strategy

```sql
-- ✅ DO: Index foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- ✅ DO: Index WHERE clause columns
CREATE INDEX idx_users_email ON users(email);

-- ✅ DO: Composite index cho multiple columns
CREATE INDEX idx_users_name ON users(last_name, first_name);

-- ❌ DON'T: Over-index
-- Mỗi index tốn space và làm chậm writes

-- ❌ DON'T: Index low-cardinality columns
CREATE INDEX idx_users_gender ON users(gender); -- BAD!
```

### 2. Query Patterns

```sql
-- ✅ DO: Use prepared statements
PREPARE stmt FROM 'SELECT * FROM users WHERE id = ?';
EXECUTE stmt USING @user_id;

-- ✅ DO: Batch operations
INSERT INTO users (name, email) VALUES
  ('John', 'john@example.com'),
  ('Jane', 'jane@example.com'),
  ('Bob', 'bob@example.com');

-- ❌ DON'T: N+1 queries
-- BAD:
SELECT * FROM users;
-- For each user:
SELECT * FROM orders WHERE user_id = ?;

-- GOOD:
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

### 3. Schema Design

```sql
-- ✅ DO: Normalize to reduce redundancy
-- Users table
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Orders table
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- ✅ DO: Denormalize for read-heavy workloads
-- Add frequently accessed data
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    user_name VARCHAR(100), -- Denormalized
    user_email VARCHAR(255)  -- Denormalized
);
```

### 4. Monitoring

```sql
-- PostgreSQL: Check slow queries
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- MySQL: Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;

-- Check index usage
SELECT * FROM pg_stat_user_indexes
WHERE idx_scan = 0;
```

---

## MongoDB Best Practices

### 1. Index Strategy

```javascript
// ✅ DO: Index query patterns
db.users.createIndex({ email: 1 });
db.orders.createIndex({ userId: 1, createdAt: -1 });

// ✅ DO: Use compound indexes
db.products.createIndex({ category: 1, price: -1, name: 1 });

// ✅ DO: Use partial indexes
db.users.createIndex({ email: 1 }, { partialFilterExpression: { isActive: true } });

// ❌ DON'T: Create too many indexes
// Each index costs memory and slows writes

// ✅ DO: Monitor index usage
db.users.aggregate([{ $indexStats: {} }]);
```

### 2. Schema Design

```javascript
// ✅ DO: Embed for one-to-few
{
  _id: ObjectId(),
  name: "John",
  addresses: [
    { street: "123 Main", city: "NYC" },
    { street: "456 Oak", city: "LA" }
  ]
}

// ✅ DO: Reference for one-to-many
// Users
{ _id: ObjectId("user1"), name: "John" }

// Orders (many)
{ _id: ObjectId(), userId: ObjectId("user1"), total: 100 }

// ✅ DO: Denormalize frequently accessed data
{
  _id: ObjectId(),
  userId: ObjectId("user1"),
  userName: "John", // Denormalized
  total: 100
}

// ❌ DON'T: Embed unbounded arrays
{
  _id: ObjectId(),
  name: "Product",
  reviews: [ /* Could grow to millions! */ ]
}
```

### 3. Query Patterns

```javascript
// ✅ DO: Use projection
db.users.find({ email: "john@example.com" }, { name: 1, email: 1, _id: 0 });

// ✅ DO: Use covered queries
db.users.createIndex({ email: 1, name: 1 });
db.users.find({ email: "john@example.com" }, { email: 1, name: 1, _id: 0 });

// ✅ DO: Batch operations
db.users.insertMany([{ name: "John" }, { name: "Jane" }, { name: "Bob" }]);

// ❌ DON'T: Use $where
db.users.find({ $where: "this.age > 18" }); // Slow!

// ✅ DO: Use standard operators
db.users.find({ age: { $gt: 18 } });
```

### 4. Aggregation

```javascript
// ✅ DO: $match early
db.orders.aggregate([
  { $match: { status: "completed" } }, // Filter first
  { $sort: { total: -1 } },
  { $limit: 10 },
]);

// ✅ DO: $project to reduce data
db.orders.aggregate([
  { $project: { userId: 1, total: 1 } }, // Only needed fields
  { $group: { _id: "$userId", total: { $sum: "$total" } } },
]);

// ❌ DON'T: $lookup without indexes
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user",
    },
  },
]);

// ✅ DO: Ensure indexes on lookup fields
db.users.createIndex({ _id: 1 }); // Usually automatic
```

### 5. Monitoring

```javascript
// Enable profiler for slow queries
db.setProfilingLevel(1, { slowms: 100 });

// View slow queries
db.system.profile
  .find({ millis: { $gt: 100 } })
  .sort({ ts: -1 })
  .limit(10);

// Check current operations
db.currentOp({
  active: true,
  secs_running: { $gt: 3 },
});

// Database stats
db.stats();

// Collection stats
db.users.stats();
```

---

## General Best Practices

### 1. Caching

```javascript
// Redis caching layer
const redis = require("redis");
const client = redis.createClient();

async function getUser(id) {
  // Check cache first
  const cached = await client.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  // Query database
  const user = await db.users.findOne({ _id: id });

  // Cache result
  await client.setex(`user:${id}`, 3600, JSON.stringify(user));

  return user;
}
```

### 2. Pagination

```javascript
// ❌ BAD: Offset pagination
// SQL
SELECT * FROM products LIMIT 10 OFFSET 100000; // Slow!

// MongoDB
db.products.find().skip(100000).limit(10); // Slow!

// ✅ GOOD: Cursor-based pagination
// SQL
SELECT * FROM products WHERE id > 12345 ORDER BY id LIMIT 10;

// MongoDB
db.products.find({ _id: { $gt: lastSeenId } }).limit(10);
```

### 3. Batch Processing

```javascript
// ✅ Process in batches
const BATCH_SIZE = 1000;

async function processAllUsers() {
  let lastId = null;

  while (true) {
    const query = lastId ? { _id: { $gt: lastId } } : {};
    const users = await db.users.find(query).sort({ _id: 1 }).limit(BATCH_SIZE).toArray();

    if (users.length === 0) break;

    // Process batch
    await processBatch(users);

    lastId = users[users.length - 1]._id;
  }
}
```

### 4. Connection Management

```javascript
// ✅ Use connection pooling
const pool = new Pool({
  max: 20,
  min: 5,
  idleTimeoutMillis: 30000,
});

// ✅ Close connections properly
process.on("SIGTERM", async () => {
  await pool.end();
  await mongoClient.close();
  process.exit(0);
});
```

### 5. Monitoring & Alerting

```javascript
// Monitor query performance
const queryTime = Date.now();
const result = await db.users.find({ email });
const duration = Date.now() - queryTime;

if (duration > 100) {
  console.warn(`Slow query: ${duration}ms`);
  // Send alert
}

// Monitor connection pool
setInterval(() => {
  console.log("Pool stats:", {
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount,
  });
}, 60000);
```

---

## Performance Checklist

### SQL

- [ ] Indexes trên foreign keys
- [ ] Indexes trên WHERE/JOIN/ORDER BY columns
- [ ] Composite indexes cho multiple columns
- [ ] Analyze query execution plans
- [ ] Avoid SELECT \*
- [ ] Use LIMIT appropriately
- [ ] Batch INSERT/UPDATE operations
- [ ] Use connection pooling
- [ ] Monitor slow queries
- [ ] Regular VACUUM/ANALYZE (PostgreSQL)
- [ ] Optimize table statistics

### MongoDB

- [ ] Indexes trên query patterns
- [ ] Compound indexes cho multiple fields
- [ ] Use projection to limit fields
- [ ] Covered queries khi có thể
- [ ] $match early trong aggregation
- [ ] Avoid $where operator
- [ ] Use cursor-based pagination
- [ ] Batch write operations
- [ ] Monitor with profiler
- [ ] Check index usage stats
- [ ] Appropriate schema design (embed vs reference)

---

## Tools & Resources

### SQL Tools

- **EXPLAIN/EXPLAIN ANALYZE**: Query execution plans
- **pg_stat_statements**: Query statistics (PostgreSQL)
- **MySQL Workbench**: Visual query analysis
- **pgAdmin**: PostgreSQL administration
- **DataGrip**: Multi-database IDE

### MongoDB Tools

- **MongoDB Compass**: GUI with explain plans
- **mongo shell**: Command-line interface
- **MongoDB Atlas**: Cloud monitoring
- **Studio 3T**: Advanced MongoDB IDE
- **mongostat/mongotop**: Real-time stats

### Monitoring

- **Prometheus + Grafana**: Metrics visualization
- **New Relic**: APM monitoring
- **Datadog**: Infrastructure monitoring
- **ELK Stack**: Log analysis

---

## Tổng Kết

### Key Takeaways

1. **Indexing is crucial**: Đúng index = 100x faster
2. **Measure first**: Dùng EXPLAIN/profiler trước khi optimize
3. **Schema matters**: Design cho use case cụ thể
4. **Batch operations**: Giảm round trips
5. **Monitor continuously**: Catch issues early
6. **Cache strategically**: Giảm database load
7. **Connection pooling**: Reuse connections
8. **Test with production data**: Dev data không đại diện

### When to Optimize

1. **Slow queries** (> 100ms)
2. **High CPU/Memory** usage
3. **Connection pool exhaustion**
4. **Increasing response times**
5. **User complaints**

### Optimization Process

1. **Identify**: Monitor và tìm slow queries
2. **Analyze**: EXPLAIN/profiler
3. **Plan**: Xác định optimization strategy
4. **Implement**: Tạo indexes, refactor queries
5. **Test**: Verify improvements
6. **Monitor**: Ensure sustained performance

---

**Lưu Ý**: Optimization là quá trình liên tục. Luôn measure trước và sau khi optimize để verify improvements.
