# SQL Functions - Hướng dẫn đầy đủ

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
