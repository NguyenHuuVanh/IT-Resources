# SQL Functions - Hướng dẫn đầy đủ

## Mục lục

- [Tổng quan các loại hàm trong SQL](#tổng-quan-các-loại-hàm-trong-sql)
- [PHẦN 1: SCALAR FUNCTIONS](#phần-1-scalar-functions)
  - [1.1 String Functions](#11-string-functions)
  - [1.2 Numeric Functions](#12-numeric-functions)
  - [1.3 Date/Time Functions](#13-datetime-functions)
  - [1.4 Conversion Functions](#14-conversion-functions)
- [PHẦN 2: AGGREGATE FUNCTIONS](#phần-2-aggregate-functions)
  - [2.1 Các hàm Aggregate cơ bản](#21-các-hàm-aggregate-cơ-bản)
  - [2.2 GROUP BY và HAVING](#22-group-by-và-having)
  - [2.3 GROUPING SETS, ROLLUP, CUBE](#23-grouping-sets-rollup-cube)
- [PHẦN 3: WINDOW FUNCTIONS](#phần-3-window-functions-analytic-functions)
  - [3.1 Cú pháp Window Function](#31-cú-pháp-window-function)
  - [3.2 Ranking Functions](#32-ranking-functions)
  - [3.3 Aggregate Window Functions](#33-aggregate-window-functions)
  - [3.4 Navigation Functions](#34-navigation-functions)
  - [3.5 Ví dụ thực tế Window Functions](#35-ví-dụ-thực-tế-window-functions)
- [PHẦN 4: CONDITIONAL FUNCTIONS](#phần-4-conditional-functions)
  - [4.1 CASE Expression](#41-case-expression)
  - [4.2 NULL Handling Functions](#42-null-handling-functions)
  - [4.3 IIF / IF / DECODE](#43-iif--if--decode)
- [PHẦN 5: JSON FUNCTIONS](#phần-5-json-functions-modern-sql)
  - [5.1 SQL Server JSON](#51-sql-server-json)
  - [5.2 MySQL JSON](#52-mysql-json)
  - [5.3 PostgreSQL JSON](#53-postgresql-json)
- [PHẦN 6: SYSTEM FUNCTIONS](#phần-6-system-functions)
- [PHẦN 7: USER-DEFINED FUNCTIONS](#phần-7-user-defined-functions-udf)
  - [7.1 Scalar Function](#71-scalar-function)
  - [7.2 Table-Valued Function](#72-table-valued-function)
- [TÓM TẮT](#tóm-tắt)

---

## Tổng quan các loại hàm trong SQL

```
┌─────────────────────────────────────────────────────────────┐
│                      SQL FUNCTIONS                          │
├─────────────────┬─────────────────┬─────────────────────────┤
│  Scalar         │  Aggregate      │  Window Functions       │
│  Functions      │  Functions      │  (Analytic Functions)   │
├─────────────────┼─────────────────┼─────────────────────────┤
│ - String        │ - COUNT()       │ - Ranking: ROW_NUMBER,  │
│ - Numeric       │ - SUM()         │   RANK, DENSE_RANK      │
│ - Date/Time     │ - AVG()         │ - Aggregate: SUM, AVG   │
│ - Conversion    │ - MIN/MAX()     │ - Navigation: LAG, LEAD │
│ - NULL handling │ - GROUP_CONCAT  │ - Distribution: NTILE   │
└─────────────────┴─────────────────┴─────────────────────────┘
```

---

# PHẦN 1: SCALAR FUNCTIONS

Scalar functions trả về **một giá trị** cho mỗi dòng.

## 1.1 String Functions

### Các hàm xử lý chuỗi phổ biến

```sql
-- Độ dài chuỗi
SELECT LEN('Hello World');           -- SQL Server: 11
SELECT LENGTH('Hello World');        -- MySQL/PostgreSQL: 11
SELECT CHAR_LENGTH('Xin chào');      -- Đếm ký tự (Unicode): 8

-- Chuyển đổi chữ hoa/thường
SELECT UPPER('hello');               -- HELLO
SELECT LOWER('HELLO');               -- hello
SELECT INITCAP('hello world');       -- Hello World (PostgreSQL/Oracle)

-- Cắt khoảng trắng
SELECT TRIM('  hello  ');            -- 'hello'
SELECT LTRIM('  hello');             -- 'hello'
SELECT RTRIM('hello  ');             -- 'hello'

-- Trích xuất chuỗi con
SELECT SUBSTRING('Hello World', 1, 5);    -- 'Hello'
SELECT LEFT('Hello World', 5);            -- 'Hello'
SELECT RIGHT('Hello World', 5);           -- 'World'

-- Tìm vị trí
SELECT CHARINDEX('o', 'Hello');           -- SQL Server: 5
SELECT POSITION('o' IN 'Hello');          -- PostgreSQL: 5
SELECT INSTR('Hello', 'o');               -- MySQL/Oracle: 5

-- Thay thế
SELECT REPLACE('Hello World', 'World', 'SQL');  -- 'Hello SQL'

-- Nối chuỗi
SELECT CONCAT('Hello', ' ', 'World');     -- 'Hello World'
SELECT 'Hello' || ' ' || 'World';         -- PostgreSQL/Oracle
SELECT 'Hello' + ' ' + 'World';           -- SQL Server

-- Lặp chuỗi
SELECT REPLICATE('Ab', 3);                -- SQL Server: 'AbAbAb'
SELECT REPEAT('Ab', 3);                   -- MySQL: 'AbAbAb'

-- Đảo ngược
SELECT REVERSE('Hello');                  -- 'olleH'
```

### Ví dụ thực tế

```sql
-- Format tên
SELECT
    CONCAT(UPPER(LEFT(first_name, 1)), LOWER(SUBSTRING(first_name, 2, LEN(first_name)))) AS formatted_name
FROM employees;

-- Tạo email từ tên
SELECT
    LOWER(CONCAT(first_name, '.', last_name, '@company.com')) AS email
FROM employees;

-- Ẩn số điện thoại
SELECT
    CONCAT('***-***-', RIGHT(phone, 4)) AS masked_phone
FROM customers;
```

---

## 1.2 Numeric Functions

```sql
-- Làm tròn
SELECT ROUND(123.456, 2);        -- 123.46
SELECT ROUND(123.456, 0);        -- 123
SELECT ROUND(123.456, -1);       -- 120

-- Làm tròn lên/xuống
SELECT CEILING(123.1);           -- 124
SELECT FLOOR(123.9);             -- 123

-- Giá trị tuyệt đối
SELECT ABS(-123);                -- 123

-- Lũy thừa và căn
SELECT POWER(2, 3);              -- 8
SELECT SQRT(16);                 -- 4
SELECT EXP(1);                   -- 2.718... (e^1)
SELECT LOG(10);                  -- Logarit tự nhiên
SELECT LOG10(100);               -- 2

-- Chia lấy dư
SELECT MOD(10, 3);               -- MySQL/PostgreSQL: 1
SELECT 10 % 3;                   -- SQL Server: 1

-- Dấu số
SELECT SIGN(-5);                 -- -1
SELECT SIGN(0);                  -- 0
SELECT SIGN(5);                  -- 1

-- Random
SELECT RAND();                   -- SQL Server/MySQL: 0.0 - 1.0
SELECT RANDOM();                 -- PostgreSQL
```

### Ví dụ thực tế

```sql
-- Tính giá sau giảm giá (làm tròn 2 chữ số)
SELECT
    product_name,
    price,
    ROUND(price * 0.9, 2) AS discounted_price
FROM products;

-- Phân loại theo khoảng giá
SELECT
    product_name,
    price,
    FLOOR(price / 100) * 100 AS price_range
FROM products;
```

---

## 1.3 Date/Time Functions

```sql
-- Lấy ngày giờ hiện tại
SELECT GETDATE();                -- SQL Server
SELECT NOW();                    -- MySQL/PostgreSQL
SELECT CURRENT_TIMESTAMP;        -- Standard SQL
SELECT SYSDATE;                  -- Oracle

-- Chỉ lấy ngày hoặc giờ
SELECT CAST(GETDATE() AS DATE);  -- Chỉ ngày
SELECT CAST(GETDATE() AS TIME);  -- Chỉ giờ

-- Trích xuất thành phần
SELECT YEAR('2024-03-15');       -- 2024
SELECT MONTH('2024-03-15');      -- 3
SELECT DAY('2024-03-15');        -- 15
SELECT DATEPART(QUARTER, '2024-03-15');  -- SQL Server: 1
SELECT EXTRACT(MONTH FROM '2024-03-15'); -- PostgreSQL

-- Cộng/trừ ngày
SELECT DATEADD(DAY, 7, '2024-03-15');      -- SQL Server: 2024-03-22
SELECT DATEADD(MONTH, 1, '2024-03-15');    -- 2024-04-15
SELECT DATE_ADD('2024-03-15', INTERVAL 7 DAY);  -- MySQL

-- Khoảng cách giữa 2 ngày
SELECT DATEDIFF(DAY, '2024-01-01', '2024-03-15');   -- SQL Server: 74
SELECT DATEDIFF('2024-03-15', '2024-01-01');        -- MySQL: 74

-- Format ngày
SELECT FORMAT(GETDATE(), 'dd/MM/yyyy');             -- SQL Server
SELECT DATE_FORMAT(NOW(), '%d/%m/%Y');              -- MySQL
SELECT TO_CHAR(NOW(), 'DD/MM/YYYY');                -- PostgreSQL

-- Ngày đầu/cuối tháng
SELECT EOMONTH('2024-03-15');                       -- SQL Server: 2024-03-31
SELECT LAST_DAY('2024-03-15');                      -- MySQL/Oracle
```

### Ví dụ thực tế

```sql
-- Tính tuổi
SELECT
    name,
    birth_date,
    DATEDIFF(YEAR, birth_date, GETDATE()) AS age
FROM employees;

-- Đơn hàng trong 30 ngày gần nhất
SELECT * FROM orders
WHERE order_date >= DATEADD(DAY, -30, GETDATE());

-- Nhóm theo tháng
SELECT
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    SUM(total) AS monthly_revenue
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date);
```

---

## 1.4 Conversion Functions

```sql
-- CAST (Standard SQL)
SELECT CAST(123 AS VARCHAR(10));         -- '123'
SELECT CAST('123' AS INT);               -- 123
SELECT CAST('2024-03-15' AS DATE);       -- Date type
SELECT CAST(123.456 AS DECIMAL(10,2));   -- 123.46

-- CONVERT (SQL Server)
SELECT CONVERT(VARCHAR(10), 123);        -- '123'
SELECT CONVERT(VARCHAR, GETDATE(), 103); -- 'dd/mm/yyyy'
SELECT CONVERT(VARCHAR, GETDATE(), 108); -- 'hh:mm:ss'

-- TRY_CAST / TRY_CONVERT (không lỗi nếu fail)
SELECT TRY_CAST('abc' AS INT);           -- NULL (không lỗi)
SELECT TRY_CONVERT(INT, 'abc');          -- NULL

-- COALESCE (trả về giá trị không NULL đầu tiên)
SELECT COALESCE(NULL, NULL, 'default');  -- 'default'
SELECT COALESCE(phone, mobile, 'N/A') FROM customers;

-- NULLIF (trả về NULL nếu 2 giá trị bằng nhau)
SELECT NULLIF(10, 10);                   -- NULL
SELECT NULLIF(10, 20);                   -- 10

-- IIF (SQL Server) / IF (MySQL)
SELECT IIF(score >= 50, 'Pass', 'Fail'); -- SQL Server
SELECT IF(score >= 50, 'Pass', 'Fail');  -- MySQL
```

---

# PHẦN 2: AGGREGATE FUNCTIONS

Aggregate functions tính toán trên **tập hợp các dòng** và trả về **một giá trị**.

## 2.1 Các hàm Aggregate cơ bản

```sql
-- COUNT - Đếm số dòng
SELECT COUNT(*) FROM employees;              -- Đếm tất cả dòng
SELECT COUNT(email) FROM employees;          -- Đếm dòng có email (không NULL)
SELECT COUNT(DISTINCT department) FROM employees;  -- Đếm giá trị unique

-- SUM - Tổng
SELECT SUM(salary) FROM employees;
SELECT SUM(quantity * price) AS total_revenue FROM order_items;

-- AVG - Trung bình
SELECT AVG(salary) FROM employees;
SELECT AVG(CAST(salary AS DECIMAL(10,2))) FROM employees;  -- Chính xác hơn

-- MIN / MAX
SELECT MIN(salary), MAX(salary) FROM employees;
SELECT MIN(hire_date), MAX(hire_date) FROM employees;

-- GROUP_CONCAT (MySQL) / STRING_AGG (SQL Server/PostgreSQL)
SELECT
    department_id,
    STRING_AGG(employee_name, ', ') AS employees  -- SQL Server 2017+
FROM employees
GROUP BY department_id;

SELECT
    department_id,
    GROUP_CONCAT(employee_name SEPARATOR ', ') AS employees  -- MySQL
FROM employees
GROUP BY department_id;
```

## 2.2 GROUP BY và HAVING

```sql
-- GROUP BY cơ bản
SELECT
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department_id;

-- HAVING - Filter sau khi GROUP BY
SELECT
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5 AND AVG(salary) > 50000;

-- GROUP BY với nhiều cột
SELECT
    department_id,
    job_title,
    COUNT(*) AS count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id, job_title
ORDER BY department_id, avg_salary DESC;
```

## 2.3 GROUPING SETS, ROLLUP, CUBE

```sql
-- ROLLUP - Tạo subtotals theo thứ tự
SELECT
    COALESCE(region, 'All Regions') AS region,
    COALESCE(city, 'All Cities') AS city,
    SUM(sales) AS total_sales
FROM sales_data
GROUP BY ROLLUP(region, city);

-- Kết quả:
-- | region    | city      | total_sales |
-- |-----------|-----------|-------------|
-- | North     | Hanoi     | 1000        |
-- | North     | Haiphong  | 800         |
-- | North     | All Cities| 1800        |  <- Subtotal
-- | South     | HCMC      | 2000        |
-- | South     | All Cities| 2000        |  <- Subtotal
-- | All Regions| All Cities| 3800       |  <- Grand Total

-- CUBE - Tạo tất cả combinations
SELECT
    region,
    product,
    SUM(sales) AS total_sales
FROM sales_data
GROUP BY CUBE(region, product);

-- GROUPING SETS - Chỉ định cụ thể các nhóm
SELECT
    region,
    product,
    SUM(sales) AS total_sales
FROM sales_data
GROUP BY GROUPING SETS (
    (region, product),
    (region),
    (product),
    ()
);
```

---

# PHẦN 3: WINDOW FUNCTIONS (Analytic Functions)

Window Functions thực hiện tính toán trên **một tập hợp các dòng liên quan** đến dòng hiện tại, **không gộp các dòng lại**.

## 3.1 Cú pháp Window Function

```sql
function_name(expression) OVER (
    [PARTITION BY partition_expression]
    [ORDER BY sort_expression [ASC|DESC]]
    [frame_clause]
)
```

**Frame clause:**

```sql
ROWS BETWEEN frame_start AND frame_end

-- frame_start / frame_end có thể là:
-- UNBOUNDED PRECEDING  : Từ đầu partition
-- n PRECEDING          : n dòng trước
-- CURRENT ROW          : Dòng hiện tại
-- n FOLLOWING          : n dòng sau
-- UNBOUNDED FOLLOWING  : Đến cuối partition
```

## 3.2 Ranking Functions

```sql
SELECT
    employee_name,
    department_id,
    salary,
    -- Đánh số tuần tự (không trùng)
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,

    -- Xếp hạng (trùng thì bỏ số)
    RANK() OVER (ORDER BY salary DESC) AS rank,

    -- Xếp hạng (trùng không bỏ số)
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank,

    -- Phần trăm xếp hạng (0-1)
    PERCENT_RANK() OVER (ORDER BY salary DESC) AS percent_rank,

    -- Chia thành n nhóm
    NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;
```

**So sánh kết quả:**
| employee | salary | row_num | rank | dense_rank |
|----------|--------|---------|------|------------|
| Alice | 100000 | 1 | 1 | 1 |
| Bob | 90000 | 2 | 2 | 2 |
| Charlie | 90000 | 3 | 2 | 2 |
| David | 80000 | 4 | 4 | 3 |

## 3.3 Aggregate Window Functions

```sql
SELECT
    order_date,
    amount,
    -- Tổng tích lũy (running total)
    SUM(amount) OVER (ORDER BY order_date) AS running_total,

    -- Tổng theo partition
    SUM(amount) OVER (PARTITION BY customer_id) AS customer_total,

    -- Trung bình động 3 ngày
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3day,

    -- Tổng toàn bộ
    SUM(amount) OVER () AS grand_total,

    -- Phần trăm so với tổng
    amount * 100.0 / SUM(amount) OVER () AS percentage
FROM orders;
```

## 3.4 Navigation Functions

```sql
SELECT
    order_date,
    amount,
    -- Giá trị dòng trước
    LAG(amount, 1, 0) OVER (ORDER BY order_date) AS prev_amount,

    -- Giá trị dòng sau
    LEAD(amount, 1, 0) OVER (ORDER BY order_date) AS next_amount,

    -- Giá trị đầu tiên trong partition
    FIRST_VALUE(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS first_order_amount,

    -- Giá trị cuối cùng trong partition
    LAST_VALUE(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_order_amount,

    -- Giá trị thứ n
    NTH_VALUE(amount, 2) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS second_order_amount
FROM orders;
```

## 3.5 Ví dụ thực tế Window Functions

### Tính tăng trưởng so với tháng trước

```sql
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month,
    revenue - LAG(revenue) OVER (ORDER BY month) AS growth,
    ROUND(
        (revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0
        / LAG(revenue) OVER (ORDER BY month), 2
    ) AS growth_percent
FROM monthly_sales;
```

### Running Total và Cumulative Percentage

```sql
SELECT
    product_name,
    sales,
    SUM(sales) OVER (ORDER BY sales DESC) AS running_total,
    SUM(sales) OVER () AS total_sales,
    ROUND(
        SUM(sales) OVER (ORDER BY sales DESC) * 100.0
        / SUM(sales) OVER (), 2
    ) AS cumulative_percent
FROM product_sales;
```

### Top N mỗi nhóm

```sql
WITH RankedProducts AS (
    SELECT
        category,
        product_name,
        sales,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY sales DESC
        ) AS rank
    FROM products
)
SELECT * FROM RankedProducts WHERE rank <= 3;
```

### Moving Average (Trung bình động)

```sql
SELECT
    date,
    price,
    -- MA 7 ngày
    AVG(price) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS ma_7,
    -- MA 30 ngày
    AVG(price) OVER (
        ORDER BY date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS ma_30
FROM stock_prices;
```

### Year-over-Year Comparison

```sql
SELECT
    year,
    month,
    revenue,
    LAG(revenue, 12) OVER (ORDER BY year, month) AS same_month_last_year,
    revenue - LAG(revenue, 12) OVER (ORDER BY year, month) AS yoy_change
FROM monthly_revenue;
```

---

# PHẦN 4: CONDITIONAL FUNCTIONS

## 4.1 CASE Expression

```sql
-- Simple CASE
SELECT
    product_name,
    category_id,
    CASE category_id
        WHEN 1 THEN 'Electronics'
        WHEN 2 THEN 'Clothing'
        WHEN 3 THEN 'Food'
        ELSE 'Other'
    END AS category_name
FROM products;

-- Searched CASE (linh hoạt hơn)
SELECT
    employee_name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'Executive'
        WHEN salary >= 70000 THEN 'Senior'
        WHEN salary >= 40000 THEN 'Mid-level'
        ELSE 'Junior'
    END AS level
FROM employees;

-- CASE trong ORDER BY
SELECT * FROM products
ORDER BY
    CASE WHEN stock = 0 THEN 1 ELSE 0 END,  -- Out of stock cuối
    product_name;

-- CASE trong Aggregate
SELECT
    COUNT(CASE WHEN status = 'completed' THEN 1 END) AS completed,
    COUNT(CASE WHEN status = 'pending' THEN 1 END) AS pending,
    COUNT(CASE WHEN status = 'cancelled' THEN 1 END) AS cancelled
FROM orders;
```

## 4.2 NULL Handling Functions

```sql
-- COALESCE - Trả về giá trị không NULL đầu tiên
SELECT COALESCE(phone, mobile, email, 'No contact') AS contact
FROM customers;

-- ISNULL (SQL Server) / IFNULL (MySQL)
SELECT ISNULL(phone, 'N/A') FROM customers;      -- SQL Server
SELECT IFNULL(phone, 'N/A') FROM customers;      -- MySQL

-- NVL (Oracle)
SELECT NVL(phone, 'N/A') FROM customers;         -- Oracle

-- NULLIF - Trả về NULL nếu 2 giá trị bằng nhau
SELECT NULLIF(quantity, 0);  -- Tránh chia cho 0
SELECT total / NULLIF(quantity, 0) AS unit_price;

-- IS NULL / IS NOT NULL
SELECT * FROM customers WHERE phone IS NULL;
SELECT * FROM customers WHERE phone IS NOT NULL;
```

## 4.3 IIF / IF / DECODE

```sql
-- IIF (SQL Server 2012+)
SELECT
    product_name,
    stock,
    IIF(stock > 0, 'In Stock', 'Out of Stock') AS availability
FROM products;

-- IF (MySQL)
SELECT
    product_name,
    IF(stock > 0, 'In Stock', 'Out of Stock') AS availability
FROM products;

-- DECODE (Oracle)
SELECT
    product_name,
    DECODE(category_id, 1, 'Electronics', 2, 'Clothing', 'Other') AS category
FROM products;
```

---

# PHẦN 5: JSON FUNCTIONS (Modern SQL)

## 5.1 SQL Server JSON

```sql
-- Parse JSON
SELECT JSON_VALUE('{"name":"John","age":30}', '$.name');  -- 'John'
SELECT JSON_QUERY('{"items":[1,2,3]}', '$.items');        -- '[1,2,3]'

-- Kiểm tra JSON hợp lệ
SELECT ISJSON('{"name":"John"}');  -- 1

-- Tạo JSON từ query
SELECT
    employee_id,
    employee_name,
    salary
FROM employees
FOR JSON PATH;

-- Kết quả: [{"employee_id":1,"employee_name":"John","salary":50000},...]
```

## 5.2 MySQL JSON

```sql
-- Trích xuất giá trị
SELECT JSON_EXTRACT('{"name":"John"}', '$.name');  -- "John"
SELECT '{"name":"John"}'->>'$.name';               -- John (không có quotes)

-- Tạo JSON
SELECT JSON_OBJECT('name', 'John', 'age', 30);
SELECT JSON_ARRAY(1, 2, 3);

-- Kiểm tra
SELECT JSON_VALID('{"name":"John"}');  -- 1
SELECT JSON_CONTAINS('{"a":1,"b":2}', '1', '$.a');  -- 1
```

## 5.3 PostgreSQL JSON

```sql
-- Trích xuất
SELECT '{"name":"John"}'::json->>'name';           -- John
SELECT '{"a":{"b":"c"}}'::json->'a'->>'b';         -- c

-- Tạo JSON
SELECT json_build_object('name', 'John', 'age', 30);
SELECT json_agg(row_to_json(t)) FROM my_table t;

-- Array operations
SELECT jsonb_array_elements('[1,2,3]'::jsonb);
```

---

# PHẦN 6: SYSTEM FUNCTIONS

```sql
-- Thông tin database
SELECT DB_NAME();                    -- SQL Server: Tên database hiện tại
SELECT DATABASE();                   -- MySQL: Tên database hiện tại
SELECT CURRENT_DATABASE();           -- PostgreSQL

-- Thông tin user
SELECT USER_NAME();                  -- SQL Server
SELECT USER();                       -- MySQL
SELECT CURRENT_USER;                 -- PostgreSQL

-- Thông tin version
SELECT @@VERSION;                    -- SQL Server/MySQL
SELECT VERSION();                    -- PostgreSQL

-- Identity/Auto-increment
SELECT SCOPE_IDENTITY();             -- SQL Server: ID vừa insert
SELECT LAST_INSERT_ID();             -- MySQL
SELECT LASTVAL();                    -- PostgreSQL

-- Row count
SELECT @@ROWCOUNT;                   -- SQL Server: Số dòng affected
SELECT ROW_COUNT();                  -- MySQL
```

---

# PHẦN 7: USER-DEFINED FUNCTIONS (UDF)

## 7.1 Scalar Function

```sql
-- SQL Server
CREATE FUNCTION dbo.CalculateAge(@BirthDate DATE)
RETURNS INT
AS
BEGIN
    RETURN DATEDIFF(YEAR, @BirthDate, GETDATE()) -
           CASE WHEN DATEADD(YEAR, DATEDIFF(YEAR, @BirthDate, GETDATE()), @BirthDate) > GETDATE()
                THEN 1 ELSE 0 END;
END;

-- Sử dụng
SELECT dbo.CalculateAge('1990-05-15') AS age;
SELECT name, dbo.CalculateAge(birth_date) AS age FROM employees;
```

## 7.2 Table-Valued Function

```sql
-- Inline Table-Valued Function
CREATE FUNCTION dbo.GetEmployeesByDept(@DeptId INT)
RETURNS TABLE
AS
RETURN (
    SELECT employee_id, employee_name, salary
    FROM employees
    WHERE department_id = @DeptId
);

-- Sử dụng
SELECT * FROM dbo.GetEmployeesByDept(1);

-- JOIN với function
SELECT d.department_name, e.*
FROM departments d
CROSS APPLY dbo.GetEmployeesByDept(d.department_id) e;
```

---

# TÓM TẮT

## Bảng so sánh các loại Function

| Loại      | Input      | Output         | GROUP BY  |
| --------- | ---------- | -------------- | --------- |
| Scalar    | 1 dòng     | 1 giá trị      | Không cần |
| Aggregate | Nhiều dòng | 1 giá trị      | Cần       |
| Window    | Nhiều dòng | 1 giá trị/dòng | Không cần |

## Khi nào dùng gì?

| Mục đích               | Function                     |
| ---------------------- | ---------------------------- |
| Xử lý text             | String functions             |
| Tính toán số           | Numeric functions            |
| Xử lý ngày             | Date functions               |
| Tổng/Đếm/TB            | Aggregate + GROUP BY         |
| Ranking                | ROW_NUMBER, RANK, DENSE_RANK |
| So sánh dòng trước/sau | LAG, LEAD                    |
| Running total          | SUM() OVER()                 |
| Moving average         | AVG() OVER(ROWS...)          |
| Top N mỗi nhóm         | ROW_NUMBER + PARTITION BY    |

## Performance Tips

1. **Tránh function trong WHERE** - không dùng được index

```sql
-- Chậm
WHERE YEAR(order_date) = 2024

-- Nhanh
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'
```

2. **Index cho Window Functions** - tạo index cho PARTITION BY và ORDER BY columns

3. **Tránh Scalar UDF trong SELECT** - gọi nhiều lần, chậm

4. **Dùng Inline TVF** thay vì Multi-statement TVF
