# SQL PIVOT - Hướng dẫn Toàn diện

## Mục lục

1. [Giới thiệu PIVOT](#1-giới-thiệu-pivot)
2. [PIVOT cơ bản](#2-pivot-cơ-bản)
3. [UNPIVOT](#3-unpivot)
4. [Dynamic PIVOT](#4-dynamic-pivot)
5. [PIVOT với Multiple Columns](#5-pivot-với-multiple-columns)
6. [PIVOT trong các Database khác nhau](#6-pivot-trong-các-database-khác-nhau)
7. [Use Cases thực tế](#7-use-cases-thực-tế)
8. [Performance và Optimization](#8-performance-và-optimization)
9. [Best Practices](#9-best-practices)

---

## 1. Giới thiệu PIVOT

### PIVOT là gì?

**Định nghĩa:** PIVOT là kỹ thuật chuyển đổi dữ liệu từ dạng rows (hàng) sang columns (cột), thường dùng để tạo báo cáo cross-tabulation hoặc summary reports.

### Tại sao cần PIVOT?

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRƯỚC VÀ SAU KHI PIVOT                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TRƯỚC PIVOT (Dữ liệu dạng rows):                               │
│  ┌──────────┬─────────┬────────┐                                │
│  │ Employee │ Quarter │ Sales  │                                │
│  ├──────────┼─────────┼────────┤                                │
│  │ John     │ Q1      │ 1000   │                                │
│  │ John     │ Q2      │ 1500   │                                │
│  │ John     │ Q3      │ 1200   │                                │
│  │ Mary     │ Q1      │ 2000   │                                │
│  │ Mary     │ Q2      │ 2500   │                                │
│  │ Mary     │ Q3      │ 2200   │                                │
│  └──────────┴─────────┴────────┘                                │
│                                                                  │
│  SAU PIVOT (Dữ liệu dạng columns):                              │
│  ┌──────────┬──────┬──────┬──────┐                              │
│  │ Employee │  Q1  │  Q2  │  Q3  │                              │
│  ├──────────┼──────┼──────┼──────┤                              │
│  │ John     │ 1000 │ 1500 │ 1200 │                              │
│  │ Mary     │ 2000 │ 2500 │ 2200 │                              │
│  └──────────┴──────┴──────┴──────┘                              │
│                                                                  │
│  → Dễ đọc hơn, phù hợp cho báo cáo                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Khi nào dùng PIVOT?

- **Báo cáo tài chính** theo tháng/quý/năm
- **Dashboard** với dữ liệu tổng hợp
- **So sánh** dữ liệu theo nhiều chiều
- **Cross-tabulation** reports
- **Chuyển đổi** dữ liệu từ EAV (Entity-Attribute-Value) model

---

## 2. PIVOT cơ bản

### Cú pháp PIVOT (SQL Server)

```sql
SELECT <non-pivoted column>,
       [first pivoted column] AS <column name>,
       [second pivoted column] AS <column name>,
       ...
FROM
(
    SELECT <columns>
    FROM <table>
) AS SourceTable
PIVOT
(
    <aggregation function>(<column to aggregate>)
    FOR <column that contains values to become column headers>
    IN ([first pivoted column], [second pivoted column], ...)
) AS PivotTable;
```

### Ví dụ cơ bản

```sql
-- ═══════════════════════════════════════════════════════════
-- Setup: Tạo bảng Sales
-- ═══════════════════════════════════════════════════════════
CREATE TABLE Sales (
    Employee VARCHAR(50),
    Quarter VARCHAR(10),
    Amount DECIMAL(10, 2)
);

INSERT INTO Sales VALUES
('John', 'Q1', 1000),
('John', 'Q2', 1500),
('John', 'Q3', 1200),
('John', 'Q4', 1800),
('Mary', 'Q1', 2000),
('Mary', 'Q2', 2500),
('Mary', 'Q3', 2200),
('Mary', 'Q4', 2800),
('Peter', 'Q1', 1500),
('Peter', 'Q2', 1800),
('Peter', 'Q3', 1600),
('Peter', 'Q4', 2000);


-- ═══════════════════════════════════════════════════════════
-- PIVOT Example 1: Sales by Quarter
-- ═══════════════════════════════════════════════════════════
SELECT Employee, [Q1], [Q2], [Q3], [Q4]
FROM
(
    SELECT Employee, Quarter, Amount
    FROM Sales
) AS SourceTable
PIVOT
(
    SUM(Amount)
    FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])
) AS PivotTable
ORDER BY Employee;

/*
Result:
Employee    Q1      Q2      Q3      Q4
---------------------------------------
John        1000    1500    1200    1800
Mary        2000    2500    2200    2800
Peter       1500    1800    1600    2000
*/


-- ═══════════════════════════════════════════════════════════
-- PIVOT Example 2: Với WHERE clause
-- ═══════════════════════════════════════════════════════════
SELECT Employee, [Q1], [Q2], [Q3], [Q4]
FROM
(
    SELECT Employee, Quarter, Amount
    FROM Sales
    WHERE Amount > 1000  -- Filter trước khi PIVOT
) AS SourceTable
PIVOT
(
    SUM(Amount)
    FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])
) AS PivotTable
ORDER BY Employee;


-- ═══════════════════════════════════════════════════════════
-- PIVOT Example 3: Với nhiều aggregate functions
-- ═══════════════════════════════════════════════════════════
-- Tạo bảng với nhiều metrics
CREATE TABLE ProductSales (
    Product VARCHAR(50),
    Month VARCHAR(10),
    Quantity INT,
    Revenue DECIMAL(10, 2)
);

INSERT INTO ProductSales VALUES
('Laptop', 'Jan', 10, 10000),
('Laptop', 'Feb', 15, 15000),
('Laptop', 'Mar', 12, 12000),
('Phone', 'Jan', 50, 25000),
('Phone', 'Feb', 60, 30000),
('Phone', 'Mar', 55, 27500);

-- PIVOT cho Quantity
SELECT Product, [Jan], [Feb], [Mar]
FROM
(
    SELECT Product, Month, Quantity
    FROM ProductSales
) AS SourceTable
PIVOT
(
    SUM(Quantity)
    FOR Month IN ([Jan], [Feb], [Mar])
) AS PivotTable;

/*
Result:
Product     Jan     Feb     Mar
--------------------------------
Laptop      10      15      12
Phone       50      60      55
*/


-- ═══════════════════════════════════════════════════════════
-- PIVOT Example 4: Với AVG, MAX, MIN
-- ═══════════════════════════════════════════════════════════
-- AVG
SELECT Employee, [Q1], [Q2], [Q3], [Q4]
FROM
(
    SELECT Employee, Quarter, Amount
    FROM Sales
) AS SourceTable
PIVOT
(
    AVG(Amount)
    FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])
) AS PivotTable;

-- MAX
SELECT Employee, [Q1], [Q2], [Q3], [Q4]
FROM
(
    SELECT Employee, Quarter, Amount
    FROM Sales
) AS SourceTable
PIVOT
(
    MAX(Amount)
    FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])
) AS PivotTable;

-- COUNT
SELECT Employee, [Q1], [Q2], [Q3], [Q4]
FROM
(
    SELECT Employee, Quarter, Amount
    FROM Sales
) AS SourceTable
PIVOT
(
    COUNT(Amount)
    FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])
) AS PivotTable;
```

### PIVOT với CASE WHEN (Alternative)

```sql
-- ═══════════════════════════════════════════════════════════
-- Không dùng PIVOT - Dùng CASE WHEN
-- ═══════════════════════════════════════════════════════════
SELECT
    Employee,
    SUM(CASE WHEN Quarter = 'Q1' THEN Amount ELSE 0 END) AS Q1,
    SUM(CASE WHEN Quarter = 'Q2' THEN Amount ELSE 0 END) AS Q2,
    SUM(CASE WHEN Quarter = 'Q3' THEN Amount ELSE 0 END) AS Q3,
    SUM(CASE WHEN Quarter = 'Q4' THEN Amount ELSE 0 END) AS Q4
FROM Sales
GROUP BY Employee
ORDER BY Employee;

-- Kết quả giống PIVOT nhưng:
-- ✅ Hoạt động trên mọi database
-- ✅ Dễ hiểu hơn cho người mới
-- ❌ Dài dòng hơn
-- ❌ Khó maintain khi có nhiều columns
```

---

## 3. UNPIVOT

### UNPIVOT là gì?

**Định nghĩa:** UNPIVOT là thao tác ngược lại với PIVOT - chuyển đổi dữ liệu từ columns sang rows.

### Cú pháp UNPIVOT

```sql
SELECT <columns>
FROM
(
    SELECT <columns>
    FROM <table>
) AS SourceTable
UNPIVOT
(
    <value column>
    FOR <name column> IN ([column1], [column2], ...)
) AS UnpivotTable;
```

### Ví dụ UNPIVOT

```sql
-- ═══════════════════════════════════════════════════════════
-- Setup: Bảng dữ liệu đã PIVOT
-- ═══════════════════════════════════════════════════════════
CREATE TABLE QuarterlySales (
    Employee VARCHAR(50),
    Q1 DECIMAL(10, 2),
    Q2 DECIMAL(10, 2),
    Q3 DECIMAL(10, 2),
    Q4 DECIMAL(10, 2)
);

INSERT INTO QuarterlySales VALUES
('John', 1000, 1500, 1200, 1800),
('Mary', 2000, 2500, 2200, 2800),
('Peter', 1500, 1800, 1600, 2000);

/*
Current data:
Employee    Q1      Q2      Q3      Q4
---------------------------------------
John        1000    1500    1200    1800
Mary        2000    2500    2200    2800
Peter       1500    1800    1600    2000
*/


-- ═══════════════════════════════════════════════════════════
-- UNPIVOT: Chuyển về dạng rows
-- ═══════════════════════════════════════════════════════════
SELECT Employee, Quarter, Amount
FROM
(
    SELECT Employee, Q1, Q2, Q3, Q4
    FROM QuarterlySales
) AS SourceTable
UNPIVOT
(
    Amount FOR Quarter IN (Q1, Q2, Q3, Q4)
) AS UnpivotTable
ORDER BY Employee, Quarter;

/*
Result:
Employee    Quarter     Amount
--------------------------------
John        Q1          1000
John        Q2          1500
John        Q3          1200
John        Q4          1800
Mary        Q1          2000
Mary        Q2          2500
Mary        Q3          2200
Mary        Q4          2800
Peter       Q1          1500
Peter       Q2          1800
Peter       Q3          1600
Peter       Q4          2000
*/


-- ═══════════════════════════════════════════════════════════
-- UNPIVOT Alternative: UNION ALL
-- ═══════════════════════════════════════════════════════════
SELECT Employee, 'Q1' AS Quarter, Q1 AS Amount FROM QuarterlySales
UNION ALL
SELECT Employee, 'Q2', Q2 FROM QuarterlySales
UNION ALL
SELECT Employee, 'Q3', Q3 FROM QuarterlySales
UNION ALL
SELECT Employee, 'Q4', Q4 FROM QuarterlySales
ORDER BY Employee, Quarter;

-- Kết quả giống UNPIVOT
```

---

## 4. Dynamic PIVOT

### Vấn đề với Static PIVOT

```sql
-- ❌ Problem: Phải biết trước tất cả values
SELECT Employee, [Q1], [Q2], [Q3], [Q4]
FROM ...
PIVOT (SUM(Amount) FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])) ...

-- Nếu có thêm Q5, Q6... phải sửa query
```

### Dynamic PIVOT với Dynamic SQL

```sql
-- ═══════════════════════════════════════════════════════════
-- Dynamic PIVOT - Tự động lấy tất cả quarters
-- ═══════════════════════════════════════════════════════════
DECLARE @columns NVARCHAR(MAX), @sql NVARCHAR(MAX);

-- Lấy danh sách quarters từ data
SELECT @columns = STRING_AGG(QUOTENAME(Quarter), ',')
FROM (SELECT DISTINCT Quarter FROM Sales) AS Quarters;

-- Build dynamic SQL
SET @sql = N'
SELECT Employee, ' + @columns + '
FROM
(
    SELECT Employee, Quarter, Amount
    FROM Sales
) AS SourceTable
PIVOT
(
    SUM(Amount)
    FOR Quarter IN (' + @columns + ')
) AS PivotTable
ORDER BY Employee;
';

-- Execute
EXEC sp_executesql @sql;


-- ═══════════════════════════════════════════════════════════
-- Dynamic PIVOT - Với parameters
-- ═══════════════════════════════════════════════════════════
CREATE PROCEDURE sp_DynamicPivot
    @TableName NVARCHAR(128),
    @PivotColumn NVARCHAR(128),
    @AggregateColumn NVARCHAR(128),
    @GroupByColumn NVARCHAR(128)
AS
BEGIN
    DECLARE @columns NVARCHAR(MAX), @sql NVARCHAR(MAX);

    -- Get distinct values for pivot
    SET @sql = N'
    SELECT @cols = STRING_AGG(QUOTENAME(' + @PivotColumn + '), '','')
    FROM (SELECT DISTINCT ' + @PivotColumn + ' FROM ' + @TableName + ') AS T';

    EXEC sp_executesql @sql, N'@cols NVARCHAR(MAX) OUTPUT', @columns OUTPUT;

    -- Build pivot query
    SET @sql = N'
    SELECT ' + @GroupByColumn + ', ' + @columns + '
    FROM
    (
        SELECT ' + @GroupByColumn + ', ' + @PivotColumn + ', ' + @AggregateColumn + '
        FROM ' + @TableName + '
    ) AS SourceTable
    PIVOT
    (
        SUM(' + @AggregateColumn + ')
        FOR ' + @PivotColumn + ' IN (' + @columns + ')
    ) AS PivotTable
    ORDER BY ' + @GroupByColumn;

    EXEC sp_executesql @sql;
END;

-- Usage
EXEC sp_DynamicPivot
    @TableName = 'Sales',
    @PivotColumn = 'Quarter',
    @AggregateColumn = 'Amount',
    @GroupByColumn = 'Employee';
```

### Dynamic PIVOT với Date Ranges

```sql
-- ═══════════════════════════════════════════════════════════
-- Dynamic PIVOT cho months trong năm
-- ═══════════════════════════════════════════════════════════
CREATE TABLE MonthlySales (
    Product VARCHAR(50),
    SaleDate DATE,
    Amount DECIMAL(10, 2)
);

INSERT INTO MonthlySales VALUES
('Laptop', '2024-01-15', 1000),
('Laptop', '2024-02-20', 1500),
('Laptop', '2024-03-10', 1200),
('Phone', '2024-01-05', 2000),
('Phone', '2024-02-15', 2500),
('Phone', '2024-03-25', 2200);

-- Dynamic PIVOT by month
DECLARE @columns NVARCHAR(MAX), @sql NVARCHAR(MAX);

SELECT @columns = STRING_AGG(QUOTENAME(MonthName), ',')
FROM (
    SELECT DISTINCT FORMAT(SaleDate, 'yyyy-MM') AS MonthName
    FROM MonthlySales
) AS Months;

SET @sql = N'
SELECT Product, ' + @columns + '
FROM
(
    SELECT Product, FORMAT(SaleDate, ''yyyy-MM'') AS MonthName, Amount
    FROM MonthlySales
) AS SourceTable
PIVOT
(
    SUM(Amount)
    FOR MonthName IN (' + @columns + ')
) AS PivotTable
ORDER BY Product;
';

EXEC sp_executesql @sql;

/*
Result:
Product     2024-01     2024-02     2024-03
-------------------------------------------
Laptop      1000        1500        1200
Phone       2000        2500        2200
*/
```

---

## 5. PIVOT với Multiple Columns

### PIVOT nhiều columns cùng lúc

```sql
-- ═══════════════════════════════════════════════════════════
-- Setup: Bảng với nhiều metrics
-- ═══════════════════════════════════════════════════════════
CREATE TABLE SalesMetrics (
    Employee VARCHAR(50),
    Quarter VARCHAR(10),
    Sales DECIMAL(10, 2),
    Commission DECIMAL(10, 2),
    Bonus DECIMAL(10, 2)
);

INSERT INTO SalesMetrics VALUES
('John', 'Q1', 1000, 100, 50),
('John', 'Q2', 1500, 150, 75),
('Mary', 'Q1', 2000, 200, 100),
('Mary', 'Q2', 2500, 250, 125);


-- ═══════════════════════════════════════════════════════════
-- Method 1: Multiple PIVOT operations
-- ═══════════════════════════════════════════════════════════
SELECT
    Employee,
    [Q1_Sales], [Q2_Sales],
    [Q1_Commission], [Q2_Commission],
    [Q1_Bonus], [Q2_Bonus]
FROM
(
    SELECT
        Employee,
        Quarter + '_Sales' AS Quarter_Sales,
        Quarter + '_Commission' AS Quarter_Commission,
        Quarter + '_Bonus' AS Quarter_Bonus,
        Sales,
        Commission,
        Bonus
    FROM SalesMetrics
) AS SourceTable
PIVOT (SUM(Sales) FOR Quarter_Sales IN ([Q1_Sales], [Q2_Sales])) AS P1
PIVOT (SUM(Commission) FOR Quarter_Commission IN ([Q1_Commission], [Q2_Commission])) AS P2
PIVOT (SUM(Bonus) FOR Quarter_Bonus IN ([Q1_Bonus], [Q2_Bonus])) AS P3;


-- ═══════════════════════════════════════════════════════════
-- Method 2: UNPIVOT then PIVOT
-- ═══════════════════════════════════════════════════════════
SELECT Employee, [Q1_Sales], [Q1_Commission], [Q1_Bonus],
                 [Q2_Sales], [Q2_Commission], [Q2_Bonus]
FROM
(
    SELECT Employee, Quarter, MetricType, Value
    FROM
    (
        SELECT Employee, Quarter, Sales, Commission, Bonus
        FROM SalesMetrics
    ) AS SourceTable
    UNPIVOT
    (
        Value FOR MetricType IN (Sales, Commission, Bonus)
    ) AS UnpivotTable
) AS PreparedData
PIVOT
(
    SUM(Value)
    FOR Quarter + '_' + MetricType IN (
        [Q1_Sales], [Q1_Commission], [Q1_Bonus],
        [Q2_Sales], [Q2_Commission], [Q2_Bonus]
    )
) AS PivotTable;


-- ═══════════════════════════════════════════════════════════
-- Method 3: CASE WHEN (Đơn giản nhất)
-- ═══════════════════════════════════════════════════════════
SELECT
    Employee,
    SUM(CASE WHEN Quarter = 'Q1' THEN Sales END) AS Q1_Sales,
    SUM(CASE WHEN Quarter = 'Q1' THEN Commission END) AS Q1_Commission,
    SUM(CASE WHEN Quarter = 'Q1' THEN Bonus END) AS Q1_Bonus,
    SUM(CASE WHEN Quarter = 'Q2' THEN Sales END) AS Q2_Sales,
    SUM(CASE WHEN Quarter = 'Q2' THEN Commission END) AS Q2_Commission,
    SUM(CASE WHEN Quarter = 'Q2' THEN Bonus END) AS Q2_Bonus
FROM SalesMetrics
GROUP BY Employee;

/*
Result:
Employee  Q1_Sales  Q1_Commission  Q1_Bonus  Q2_Sales  Q2_Commission  Q2_Bonus
-----------------------------------------------------------------------------
John      1000      100            50        1500      150            75
Mary      2000      200            100       2500      250            125
*/
```

### PIVOT với Calculated Columns

```sql
-- ═══════════════════════════════════════════════════════════
-- PIVOT với tính toán
-- ═══════════════════════════════════════════════════════════
SELECT
    Employee,
    [Q1], [Q2], [Q3], [Q4],
    ([Q1] + [Q2] + [Q3] + [Q4]) AS Total,
    ([Q1] + [Q2] + [Q3] + [Q4]) / 4.0 AS Average
FROM
(
    SELECT Employee, Quarter, Amount
    FROM Sales
) AS SourceTable
PIVOT
(
    SUM(Amount)
    FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])
) AS PivotTable;


-- ═══════════════════════════════════════════════════════════
-- PIVOT với percentage
-- ═══════════════════════════════════════════════════════════
WITH PivotedData AS (
    SELECT Employee, [Q1], [Q2], [Q3], [Q4]
    FROM
    (
        SELECT Employee, Quarter, Amount
        FROM Sales
    ) AS SourceTable
    PIVOT
    (
        SUM(Amount)
        FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])
    ) AS PivotTable
)
SELECT
    Employee,
    [Q1],
    [Q2],
    [Q3],
    [Q4],
    ([Q1] + [Q2] + [Q3] + [Q4]) AS Total,
    CAST([Q1] * 100.0 / ([Q1] + [Q2] + [Q3] + [Q4]) AS DECIMAL(5,2)) AS Q1_Pct,
    CAST([Q2] * 100.0 / ([Q1] + [Q2] + [Q3] + [Q4]) AS DECIMAL(5,2)) AS Q2_Pct,
    CAST([Q3] * 100.0 / ([Q1] + [Q2] + [Q3] + [Q4]) AS DECIMAL(5,2)) AS Q3_Pct,
    CAST([Q4] * 100.0 / ([Q1] + [Q2] + [Q3] + [Q4]) AS DECIMAL(5,2)) AS Q4_Pct
FROM PivotedData;
```

---

## 6. PIVOT trong các Database khác nhau

### MySQL (Không có PIVOT native)

```sql
-- ═══════════════════════════════════════════════════════════
-- MySQL: Dùng CASE WHEN
-- ═══════════════════════════════════════════════════════════
SELECT
    Employee,
    SUM(CASE WHEN Quarter = 'Q1' THEN Amount ELSE 0 END) AS Q1,
    SUM(CASE WHEN Quarter = 'Q2' THEN Amount ELSE 0 END) AS Q2,
    SUM(CASE WHEN Quarter = 'Q3' THEN Amount ELSE 0 END) AS Q3,
    SUM(CASE WHEN Quarter = 'Q4' THEN Amount ELSE 0 END) AS Q4
FROM Sales
GROUP BY Employee
ORDER BY Employee;


-- ═══════════════════════════════════════════════════════════
-- MySQL: Dynamic PIVOT với Prepared Statement
-- ═══════════════════════════════════════════════════════════
SET @sql = NULL;

SELECT
  GROUP_CONCAT(DISTINCT
    CONCAT(
      'SUM(CASE WHEN Quarter = ''',
      Quarter,
      ''' THEN Amount ELSE 0 END) AS `',
      Quarter, '`'
    )
  ) INTO @sql
FROM Sales;

SET @sql = CONCAT('SELECT Employee, ', @sql, '
                   FROM Sales
                   GROUP BY Employee');

PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```

### PostgreSQL

```sql
-- ═══════════════════════════════════════════════════════════
-- PostgreSQL: Dùng crosstab (tablefunc extension)
-- ═══════════════════════════════════════════════════════════
CREATE EXTENSION IF NOT EXISTS tablefunc;

-- Static crosstab
SELECT *
FROM crosstab(
    'SELECT Employee, Quarter, SUM(Amount)
     FROM Sales
     GROUP BY Employee, Quarter
     ORDER BY Employee, Quarter',
    'SELECT DISTINCT Quarter FROM Sales ORDER BY Quarter'
) AS ct(Employee VARCHAR, Q1 NUMERIC, Q2 NUMERIC, Q3 NUMERIC, Q4 NUMERIC);


-- ═══════════════════════════════════════════════════════════
-- PostgreSQL: FILTER clause (Modern approach)
-- ═══════════════════════════════════════════════════════════
SELECT
    Employee,
    SUM(Amount) FILTER (WHERE Quarter = 'Q1') AS Q1,
    SUM(Amount) FILTER (WHERE Quarter = 'Q2') AS Q2,
    SUM(Amount) FILTER (WHERE Quarter = 'Q3') AS Q3,
    SUM(Amount) FILTER (WHERE Quarter = 'Q4') AS Q4
FROM Sales
GROUP BY Employee
ORDER BY Employee;


-- ═══════════════════════════════════════════════════════════
-- PostgreSQL: Dynamic PIVOT
-- ═══════════════════════════════════════════════════════════
DO $$
DECLARE
    quarters TEXT;
    query TEXT;
BEGIN
    SELECT string_agg(DISTINCT
        format('SUM(Amount) FILTER (WHERE Quarter = %L) AS %I',
               Quarter, Quarter), ', ')
    INTO quarters
    FROM Sales;

    query := format('SELECT Employee, %s FROM Sales GROUP BY Employee', quarters);

    RAISE NOTICE '%', query;
    EXECUTE query;
END $$;
```

### Oracle

```sql
-- ═══════════════════════════════════════════════════════════
-- Oracle: PIVOT syntax (11g+)
-- ═══════════════════════════════════════════════════════════
SELECT *
FROM (
    SELECT Employee, Quarter, Amount
    FROM Sales
)
PIVOT (
    SUM(Amount)
    FOR Quarter IN ('Q1' AS Q1, 'Q2' AS Q2, 'Q3' AS Q3, 'Q4' AS Q4)
)
ORDER BY Employee;


-- ═══════════════════════════════════════════════════════════
-- Oracle: Dynamic PIVOT
-- ═══════════════════════════════════════════════════════════
DECLARE
    v_sql VARCHAR2(4000);
    v_quarters VARCHAR2(1000);
BEGIN
    SELECT LISTAGG('''' || Quarter || ''' AS ' || Quarter, ', ')
           WITHIN GROUP (ORDER BY Quarter)
    INTO v_quarters
    FROM (SELECT DISTINCT Quarter FROM Sales);

    v_sql := 'SELECT * FROM (
                SELECT Employee, Quarter, Amount FROM Sales
              )
              PIVOT (
                SUM(Amount) FOR Quarter IN (' || v_quarters || ')
              )
              ORDER BY Employee';

    EXECUTE IMMEDIATE v_sql;
END;
```

### SQLite

```sql
-- ═══════════════════════════════════════════════════════════
-- SQLite: Chỉ có CASE WHEN
-- ═══════════════════════════════════════════════════════════
SELECT
    Employee,
    SUM(CASE WHEN Quarter = 'Q1' THEN Amount ELSE 0 END) AS Q1,
    SUM(CASE WHEN Quarter = 'Q2' THEN Amount ELSE 0 END) AS Q2,
    SUM(CASE WHEN Quarter = 'Q3' THEN Amount ELSE 0 END) AS Q3,
    SUM(CASE WHEN Quarter = 'Q4' THEN Amount ELSE 0 END) AS Q4
FROM Sales
GROUP BY Employee
ORDER BY Employee;
```

---

## 7. Use Cases thực tế

### Use Case 1: Sales Report by Month

```sql
-- ═══════════════════════════════════════════════════════════
-- Báo cáo doanh số theo tháng
-- ═══════════════════════════════════════════════════════════
CREATE TABLE Orders (
    OrderID INT,
    OrderDate DATE,
    CustomerID INT,
    Amount DECIMAL(10, 2)
);

-- Sample data
INSERT INTO Orders VALUES
(1, '2024-01-15', 101, 1000),
(2, '2024-01-20', 102, 1500),
(3, '2024-02-10', 101, 2000),
(4, '2024-02-15', 103, 2500),
(5, '2024-03-05', 102, 1800);

-- PIVOT by month
SELECT
    CustomerID,
    [Jan], [Feb], [Mar], [Apr], [May], [Jun],
    [Jul], [Aug], [Sep], [Oct], [Nov], [Dec],
    ([Jan] + [Feb] + [Mar] + [Apr] + [May] + [Jun] +
     [Jul] + [Aug] + [Sep] + [Oct] + [Nov] + [Dec]) AS YearTotal
FROM
(
    SELECT
        CustomerID,
        FORMAT(OrderDate, 'MMM') AS Month,
        Amount
    FROM Orders
    WHERE YEAR(OrderDate) = 2024
) AS SourceTable
PIVOT
(
    SUM(Amount)
    FOR Month IN ([Jan], [Feb], [Mar], [Apr], [May], [Jun],
                  [Jul], [Aug], [Sep], [Oct], [Nov], [Dec])
) AS PivotTable
ORDER BY CustomerID;
```

### Use Case 2: Product Comparison

```sql
-- ═══════════════════════════════════════════════════════════
-- So sánh sản phẩm theo nhiều metrics
-- ═══════════════════════════════════════════════════════════
CREATE TABLE ProductMetrics (
    Product VARCHAR(50),
    Metric VARCHAR(50),
    Value DECIMAL(10, 2)
);

INSERT INTO ProductMetrics VALUES
('Laptop', 'Sales', 50000),
('Laptop', 'Cost', 30000),
('Laptop', 'Profit', 20000),
('Laptop', 'Units', 50),
('Phone', 'Sales', 80000),
('Phone', 'Cost', 40000),
('Phone', 'Profit', 40000),
('Phone', 'Units', 100),
('Tablet', 'Sales', 30000),
('Tablet', 'Cost', 18000),
('Tablet', 'Profit', 12000),
('Tablet', 'Units', 30);

-- PIVOT metrics
SELECT
    Product,
    Sales,
    Cost,
    Profit,
    Units,
    CAST(Profit * 100.0 / Sales AS DECIMAL(5,2)) AS ProfitMargin,
    CAST(Sales / Units AS DECIMAL(10,2)) AS AvgPrice
FROM
(
    SELECT Product, Metric, Value
    FROM ProductMetrics
) AS SourceTable
PIVOT
(
    SUM(Value)
    FOR Metric IN ([Sales], [Cost], [Profit], [Units])
) AS PivotTable
ORDER BY Profit DESC;

/*
Result:
Product  Sales   Cost    Profit  Units  ProfitMargin  AvgPrice
---------------------------------------------------------------
Phone    80000   40000   40000   100    50.00         800.00
Laptop   50000   30000   20000   50     40.00         1000.00
Tablet   30000   18000   12000   30     40.00         1000.00
*/
```

### Use Case 3: Employee Attendance

```sql
-- ═══════════════════════════════════════════════════════════
-- Bảng chấm công nhân viên
-- ═══════════════════════════════════════════════════════════
CREATE TABLE Attendance (
    EmployeeID INT,
    AttendanceDate DATE,
    Status VARCHAR(20) -- Present, Absent, Leave
);

-- Sample data for January 2024
INSERT INTO Attendance VALUES
(1, '2024-01-01', 'Present'),
(1, '2024-01-02', 'Present'),
(1, '2024-01-03', 'Absent'),
(2, '2024-01-01', 'Present'),
(2, '2024-01-02', 'Leave'),
(2, '2024-01-03', 'Present');

-- PIVOT attendance by day
DECLARE @columns NVARCHAR(MAX), @sql NVARCHAR(MAX);

SELECT @columns = STRING_AGG(QUOTENAME(DAY(AttendanceDate)), ',')
FROM (SELECT DISTINCT AttendanceDate FROM Attendance) AS Dates;

SET @sql = N'
SELECT EmployeeID, ' + @columns + '
FROM
(
    SELECT
        EmployeeID,
        DAY(AttendanceDate) AS Day,
        LEFT(Status, 1) AS StatusCode
    FROM Attendance
) AS SourceTable
PIVOT
(
    MAX(StatusCode)
    FOR Day IN (' + @columns + ')
) AS PivotTable
ORDER BY EmployeeID;
';

EXEC sp_executesql @sql;

/*
Result:
EmployeeID  1   2   3
------------------------
1           P   P   A
2           P   L   P

P = Present, A = Absent, L = Leave
*/
```

### Use Case 4: Survey Results

```sql
-- ═══════════════════════════════════════════════════════════
-- Kết quả khảo sát
-- ═══════════════════════════════════════════════════════════
CREATE TABLE SurveyResponses (
    RespondentID INT,
    Question VARCHAR(100),
    Answer INT -- Rating 1-5
);

INSERT INTO SurveyResponses VALUES
(1, 'Product Quality', 5),
(1, 'Customer Service', 4),
(1, 'Price', 3),
(2, 'Product Quality', 4),
(2, 'Customer Service', 5),
(2, 'Price', 4),
(3, 'Product Quality', 3),
(3, 'Customer Service', 3),
(3, 'Price', 5);

-- PIVOT survey results
SELECT
    RespondentID,
    [Product Quality],
    [Customer Service],
    [Price],
    ([Product Quality] + [Customer Service] + [Price]) / 3.0 AS AvgRating
FROM
(
    SELECT RespondentID, Question, Answer
    FROM SurveyResponses
) AS SourceTable
PIVOT
(
    AVG(Answer)
    FOR Question IN ([Product Quality], [Customer Service], [Price])
) AS PivotTable
ORDER BY AvgRating DESC;
```

---

## 8. Performance và Optimization

### Performance Considerations

```sql
-- ═══════════════════════════════════════════════════════════
-- 1. Index cho PIVOT performance
-- ════════════════════════════════════════
```
