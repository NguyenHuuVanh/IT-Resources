# SQL Row Numbering & Ranking Functions

## Tổng quan

SQL cung cấp các Window Functions để đánh số thứ tự và xếp hạng dữ liệu. Đây là các hàm rất mạnh mẽ để phân tích dữ liệu.

## Các hàm đánh số thứ tự

| Hàm            | Mô tả                         | Xử lý trùng     |
| -------------- | ----------------------------- | --------------- |
| `ROW_NUMBER()` | Đánh số tuần tự 1, 2, 3...    | Không trùng số  |
| `RANK()`       | Xếp hạng, bỏ qua số khi trùng | 1, 2, 2, 4...   |
| `DENSE_RANK()` | Xếp hạng, không bỏ qua số     | 1, 2, 2, 3...   |
| `NTILE(n)`     | Chia thành n nhóm bằng nhau   | Nhóm 1, 2, 3... |

## Cú pháp cơ bản

```sql
FUNCTION_NAME() OVER (
    [PARTITION BY column1, column2...]
    ORDER BY column3 [ASC|DESC]
)
```

- `PARTITION BY`: Chia dữ liệu thành các nhóm (tùy chọn)
- `ORDER BY`: Thứ tự sắp xếp để đánh số

---

## 1. ROW_NUMBER()

Đánh số thứ tự tuần tự, **không bao giờ trùng số**.

### Ví dụ cơ bản

```sql
SELECT
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS stt,
    employee_name,
    salary
FROM employees;
```

**Kết quả:**
| stt | employee_name | salary |
|-----|---------------|--------|
| 1 | Alice | 100000 |
| 2 | Bob | 90000 |
| 3 | Charlie | 90000 |
| 4 | David | 80000 |

### ROW_NUMBER với PARTITION BY

```sql
-- Đánh số thứ tự theo từng phòng ban
SELECT
    ROW_NUMBER() OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
    ) AS rank_in_dept,
    employee_name,
    department_id,
    salary
FROM employees;
```

**Kết quả:**
| rank_in_dept | employee_name | department_id | salary |
|--------------|---------------|---------------|--------|
| 1 | Alice | IT | 100000 |
| 2 | Bob | IT | 90000 |
| 1 | Charlie | HR | 85000 |
| 2 | David | HR | 75000 |

---

## 2. RANK()

Xếp hạng với **cùng thứ hạng cho giá trị trùng**, sau đó **bỏ qua số**.

```sql
SELECT
    RANK() OVER (ORDER BY score DESC) AS rank,
    student_name,
    score
FROM students;
```

**Kết quả:**
| rank | student_name | score |
|------|--------------|-------|
| 1 | An | 95 |
| 2 | Bình | 90 |
| 2 | Cường | 90 |
| 4 | Dũng | 85 |

> Lưu ý: Bỏ qua số 3 vì có 2 người cùng hạng 2

---

## 3. DENSE_RANK()

Xếp hạng với **cùng thứ hạng cho giá trị trùng**, **không bỏ qua số**.

```sql
SELECT
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank,
    student_name,
    score
FROM students;
```

**Kết quả:**
| dense_rank | student_name | score |
|------------|--------------|-------|
| 1 | An | 95 |
| 2 | Bình | 90 |
| 2 | Cường | 90 |
| 3 | Dũng | 85 |

> Lưu ý: Số 3 vẫn được sử dụng (không bỏ qua)

---

## 4. NTILE(n)

Chia dữ liệu thành **n nhóm bằng nhau**.

```sql
-- Chia thành 4 nhóm (quartiles)
SELECT
    NTILE(4) OVER (ORDER BY salary DESC) AS quartile,
    employee_name,
    salary
FROM employees;
```

**Kết quả:**
| quartile | employee_name | salary |
|----------|---------------|--------|
| 1 | Alice | 100000 |
| 1 | Bob | 95000 |
| 2 | Charlie | 90000 |
| 2 | David | 85000 |
| 3 | Eve | 80000 |
| 3 | Frank | 75000 |
| 4 | Grace | 70000 |
| 4 | Henry | 65000 |

---

## So sánh ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT
    employee_name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```

**Kết quả:**
| employee_name | salary | row_num | rank | dense_rank |
|---------------|--------|---------|------|------------|
| Alice | 100000 | 1 | 1 | 1 |
| Bob | 90000 | 2 | 2 | 2 |
| Charlie | 90000 | 3 | 2 | 2 |
| David | 80000 | 4 | 4 | 3 |
| Eve | 80000 | 5 | 4 | 3 |
| Frank | 70000 | 6 | 6 | 4 |

---

## Ứng dụng thực tế

### 1. Lấy Top N mỗi nhóm

```sql
-- Top 3 nhân viên lương cao nhất mỗi phòng ban
WITH RankedEmployees AS (
    SELECT
        employee_name,
        department_id,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT * FROM RankedEmployees WHERE rn <= 3;
```

### 2. Xóa bản ghi trùng lặp

```sql
-- Giữ lại bản ghi mới nhất, xóa các bản ghi trùng
WITH CTE AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY created_at DESC
        ) AS rn
    FROM users
)
DELETE FROM CTE WHERE rn > 1;
```

### 3. Phân trang (Pagination)

```sql
-- Lấy trang 2, mỗi trang 10 bản ghi
WITH NumberedRows AS (
    SELECT
        *,
        ROW_NUMBER() OVER (ORDER BY id) AS rn
    FROM products
)
SELECT * FROM NumberedRows
WHERE rn BETWEEN 11 AND 20;

-- Hoặc dùng OFFSET FETCH (SQL Server, PostgreSQL)
SELECT * FROM products
ORDER BY id
OFFSET 10 ROWS
FETCH NEXT 10 ROWS ONLY;
```

### 4. Tìm bản ghi có giá trị cao nhất/thấp nhất

```sql
-- Đơn hàng có giá trị cao nhất của mỗi khách hàng
WITH RankedOrders AS (
    SELECT
        *,
        RANK() OVER (
            PARTITION BY customer_id
            ORDER BY total_amount DESC
        ) AS rn
    FROM orders
)
SELECT * FROM RankedOrders WHERE rn = 1;
```

### 5. Tính phần trăm xếp hạng

```sql
SELECT
    employee_name,
    salary,
    NTILE(100) OVER (ORDER BY salary) AS percentile
FROM employees;
```

### 6. So sánh với bản ghi trước/sau

```sql
SELECT
    order_date,
    total_amount,
    ROW_NUMBER() OVER (ORDER BY order_date) AS order_num,
    LAG(total_amount) OVER (ORDER BY order_date) AS prev_amount,
    LEAD(total_amount) OVER (ORDER BY order_date) AS next_amount
FROM orders;
```

---

## Các hàm Window bổ sung

### LAG() và LEAD()

```sql
-- LAG: Lấy giá trị của dòng trước
-- LEAD: Lấy giá trị của dòng sau
SELECT
    month,
    revenue,
    LAG(revenue, 1, 0) OVER (ORDER BY month) AS prev_month_revenue,
    LEAD(revenue, 1, 0) OVER (ORDER BY month) AS next_month_revenue,
    revenue - LAG(revenue, 1, 0) OVER (ORDER BY month) AS growth
FROM monthly_sales;
```

### FIRST_VALUE() và LAST_VALUE()

```sql
SELECT
    employee_name,
    department_id,
    salary,
    FIRST_VALUE(employee_name) OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
    ) AS highest_paid,
    LAST_VALUE(employee_name) OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_paid
FROM employees;
```

---

## Đánh số thứ tự không dùng Window Function

### Cách 1: Biến (MySQL)

```sql
-- MySQL 5.x
SET @row_number = 0;
SELECT
    (@row_number := @row_number + 1) AS stt,
    employee_name,
    salary
FROM employees
ORDER BY salary DESC;
```

### Cách 2: Subquery

```sql
-- Đếm số bản ghi có giá trị lớn hơn
SELECT
    e1.employee_name,
    e1.salary,
    (SELECT COUNT(*) + 1
     FROM employees e2
     WHERE e2.salary > e1.salary) AS rank
FROM employees e1
ORDER BY rank;
```

### Cách 3: IDENTITY (SQL Server)

```sql
-- Tạo bảng tạm với số thứ tự
SELECT
    IDENTITY(INT, 1, 1) AS stt,
    employee_name,
    salary
INTO #TempTable
FROM employees
ORDER BY salary DESC;
```

---

## Performance Tips

1. **Tạo Index** cho cột trong ORDER BY và PARTITION BY

```sql
CREATE INDEX idx_emp_dept_salary
ON employees(department_id, salary DESC);
```

2. **Tránh PARTITION BY trên cột có nhiều giá trị unique** - sẽ tạo nhiều partition nhỏ

3. **Sử dụng CTE thay vì subquery lồng nhau** - dễ đọc và optimize tốt hơn

4. **Giới hạn dữ liệu trước khi đánh số** nếu có thể

```sql
-- Tốt hơn: Filter trước
WITH FilteredData AS (
    SELECT * FROM orders WHERE year = 2024
)
SELECT ROW_NUMBER() OVER (ORDER BY total DESC) AS rn, *
FROM FilteredData;
```

---

## Tương thích Database

| Hàm          | MySQL | PostgreSQL | SQL Server | Oracle |
| ------------ | ----- | ---------- | ---------- | ------ |
| ROW_NUMBER() | 8.0+  | ✓          | ✓          | ✓      |
| RANK()       | 8.0+  | ✓          | ✓          | ✓      |
| DENSE_RANK() | 8.0+  | ✓          | ✓          | ✓      |
| NTILE()      | 8.0+  | ✓          | ✓          | ✓      |
| LAG/LEAD     | 8.0+  | ✓          | 2012+      | ✓      |

---

## Tóm tắt

| Khi nào dùng                       | Hàm phù hợp                      |
| ---------------------------------- | -------------------------------- |
| Đánh số tuần tự, không trùng       | `ROW_NUMBER()`                   |
| Xếp hạng, cho phép trùng, bỏ số    | `RANK()`                         |
| Xếp hạng, cho phép trùng, liên tục | `DENSE_RANK()`                   |
| Chia nhóm đều                      | `NTILE(n)`                       |
| So sánh với dòng trước/sau         | `LAG()` / `LEAD()`               |
| Lấy giá trị đầu/cuối trong nhóm    | `FIRST_VALUE()` / `LAST_VALUE()` |
