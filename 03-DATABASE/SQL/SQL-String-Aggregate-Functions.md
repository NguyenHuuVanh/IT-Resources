# SQL String Aggregation & Related Functions

## Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [STRING_AGG](#2-string_agg)
3. [QUOTENAME](#3-quotename)
4. [GROUP_CONCAT (MySQL)](#4-group_concat-mysql)
5. [LISTAGG (Oracle)](#5-listagg-oracle)
6. [ARRAY_AGG & ARRAY_TO_STRING (PostgreSQL)](#6-array_agg--array_to_string-postgresql)
7. [Các hàm String liên quan](#7-các-hàm-string-liên-quan)
8. [Use Cases thực tế](#8-use-cases-thực-tế)
9. [Performance Considerations](#9-performance-considerations)
10. [Best Practices](#10-best-practices)

---

## 1. Tổng quan

### String Aggregation là gì?

**Định nghĩa:** String aggregation là quá trình gộp nhiều giá trị từ nhiều rows thành một chuỗi duy nhất, thường được phân tách bởi delimiter.

### Vấn đề cần giải quyết

```sql
-- Dữ liệu gốc
SELECT user_id, skill_name
FROM user_skills;

/*
user_id | skill_name
--------|------------
1       | JavaScript
1       | Python
1       | SQL
2       | Java
2       | C++
*/

-- Mong muốn: Gộp skills của mỗi user thành 1 row
/*
user_id | skills
--------|------------------------
1       | JavaScript, Python, SQL
2       | Java, C++
*/
```

### So sánh các Database Systems

| Function     | SQL Server | MySQL | PostgreSQL | Oracle | SQLite     |
| ------------ | ---------- | ----- | ---------- | ------ | ---------- |
| STRING_AGG   | ✅ (2017+) | ❌    | ✅ (9.0+)  | ❌     | ✅ (3.44+) |
| GROUP_CONCAT | ❌         | ✅    | ❌         | ❌     | ✅         |
| LISTAGG      | ❌         | ❌    | ❌         | ✅     | ❌         |
| ARRAY_AGG    | ❌         | ❌    | ✅         | ✅     | ❌         |
| QUOTENAME    | ✅         | ❌    | ❌         | ❌     | ❌         |

---

## 2. STRING_AGG

### Cú pháp (SQL Server / PostgreSQL)

```sql
STRING_AGG(expression, separator)
  [WITHIN GROUP (ORDER BY order_expression)]
```

### Basic Usage

```sql
-- ═══════════════════════════════════════════════════════════
-- SQL Server / PostgreSQL
-- ═══════════════════════════════════════════════════════════

-- 1. Basic aggregation
SELECT
    user_id,
    STRING_AGG(skill_name, ', ') AS skills
FROM user_skills
GROUP BY user_id;

/*
user_id | skills
--------|------------------------
1       | JavaScript, Python, SQL
2       | Java, C++
*/


-- 2. Với ORDER BY
SELECT
    user_id,
    STRING_AGG(skill_name, ', ' ORDER BY skill_name) AS skills
FROM user_skills
GROUP BY user_id;

/*
user_id | skills
--------|------------------------
1       | JavaScript, Python, SQL  (alphabetically sorted)
2       | C++, Java
*/


-- 3. Với DISTINCT
SELECT
    department_id,
    STRING_AGG(DISTINCT job_title, ', ') AS unique_positions
FROM employees
GROUP BY department_id;


-- 4. Multiple columns
SELECT
    user_id,
    STRING_AGG(skill_name || ' (' || level || ')', ', '
               ORDER BY level DESC, skill_name) AS skills_with_level
FROM user_skills
GROUP BY user_id;

/*
user_id | skills_with_level
--------|----------------------------------
1       | Python (Expert), SQL (Advanced), JavaScript (Intermediate)
*/
```

### Advanced Examples

```sql
-- ═══════════════════════════════════════════════════════════
-- Conditional aggregation
-- ═══════════════════════════════════════════════════════════

-- Chỉ aggregate skills có level >= 'Intermediate'
SELECT
    user_id,
    STRING_AGG(
        CASE
            WHEN level IN ('Intermediate', 'Advanced', 'Expert')
            THEN skill_name
        END,
        ', '
    ) AS qualified_skills
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Nested aggregation
-- ═══════════════════════════════════════════════════════════

-- Aggregate theo category, rồi aggregate tất cả categories
SELECT
    user_id,
    STRING_AGG(category_skills, ' | ') AS all_skills
FROM (
    SELECT
        user_id,
        category,
        category || ': ' || STRING_AGG(skill_name, ', ') AS category_skills
    FROM user_skills
    GROUP BY user_id, category
) AS subquery
GROUP BY user_id;

/*
user_id | all_skills
--------|------------------------------------------------
1       | Programming: JavaScript, Python | Database: SQL
*/


-- ═══════════════════════════════════════════════════════════
-- Với JOIN
-- ═══════════════════════════════════════════════════════════

SELECT
    u.user_id,
    u.username,
    STRING_AGG(s.skill_name, ', ' ORDER BY s.skill_name) AS skills,
    COUNT(s.skill_id) AS skill_count
FROM users u
LEFT JOIN user_skills s ON u.user_id = s.user_id
GROUP BY u.user_id, u.username;


-- ═══════════════════════════════════════════════════════════
-- Với FILTER (PostgreSQL)
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    STRING_AGG(skill_name, ', ') FILTER (WHERE level = 'Expert') AS expert_skills,
    STRING_AGG(skill_name, ', ') FILTER (WHERE level = 'Beginner') AS beginner_skills
FROM user_skills
GROUP BY user_id;
```

### Handling NULL values

```sql
-- ═══════════════════════════════════════════════════════════
-- STRING_AGG bỏ qua NULL values
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    STRING_AGG(skill_name, ', ') AS skills
FROM user_skills
GROUP BY user_id;
-- NULL values trong skill_name sẽ bị bỏ qua


-- ═══════════════════════════════════════════════════════════
-- Xử lý khi tất cả values đều NULL
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    COALESCE(STRING_AGG(skill_name, ', '), 'No skills') AS skills
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Replace NULL với default value trước khi aggregate
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    STRING_AGG(COALESCE(skill_name, 'Unknown'), ', ') AS skills
FROM user_skills
GROUP BY user_id;
```

### Length Limitations

```sql
-- ═══════════════════════════════════════════════════════════
-- SQL Server: Max 8000 chars (VARCHAR) hoặc 2GB (VARCHAR(MAX))
-- ═══════════════════════════════════════════════════════════

-- Sử dụng VARCHAR(MAX) cho chuỗi dài
SELECT
    user_id,
    CAST(STRING_AGG(skill_name, ', ') AS VARCHAR(MAX)) AS skills
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Truncate nếu quá dài
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    CASE
        WHEN LEN(STRING_AGG(skill_name, ', ')) > 100
        THEN LEFT(STRING_AGG(skill_name, ', '), 97) + '...'
        ELSE STRING_AGG(skill_name, ', ')
    END AS skills
FROM user_skills
GROUP BY user_id;
```

---

## 3. QUOTENAME

### Định nghĩa (SQL Server)

**QUOTENAME** là hàm SQL Server dùng để bao quanh chuỗi bằng delimiters (thường là `[]` hoặc `""`) để tạo identifier hợp lệ.

### Cú pháp

```sql
QUOTENAME(character_string [, quote_character])
```

### Basic Usage

```sql
-- ═══════════════════════════════════════════════════════════
-- Default delimiter: []
-- ═══════════════════════════════════════════════════════════

SELECT QUOTENAME('TableName');
-- Result: [TableName]

SELECT QUOTENAME('My Table');
-- Result: [My Table]

SELECT QUOTENAME('Table]Name');
-- Result: [Table]]Name]  (escape ] bằng ]])


-- ═══════════════════════════════════════════════════════════
-- Custom delimiter
-- ═══════════════════════════════════════════════════════════

SELECT QUOTENAME('ColumnName', '"');
-- Result: "ColumnName"

SELECT QUOTENAME('Value', '''');
-- Result: 'Value'

SELECT QUOTENAME('Path', '`');
-- Result: `Path`


-- ═══════════════════════════════════════════════════════════
-- Với NULL
-- ═══════════════════════════════════════════════════════════

SELECT QUOTENAME(NULL);
-- Result: NULL
```

### Use Cases

```sql
-- ═══════════════════════════════════════════════════════════
-- 1. Dynamic SQL - Table/Column names
-- ═══════════════════════════════════════════════════════════

DECLARE @TableName NVARCHAR(128) = 'Users';
DECLARE @ColumnName NVARCHAR(128) = 'First Name';  -- Có space

DECLARE @SQL NVARCHAR(MAX);
SET @SQL = 'SELECT ' + QUOTENAME(@ColumnName) +
           ' FROM ' + QUOTENAME(@TableName);

EXEC sp_executesql @SQL;
-- Executes: SELECT [First Name] FROM [Users]


-- ═══════════════════════════════════════════════════════════
-- 2. Prevent SQL Injection
-- ═══════════════════════════════════════════════════════════

-- ❌ Vulnerable
DECLARE @UserInput NVARCHAR(128) = 'Users; DROP TABLE Users--';
DECLARE @SQL NVARCHAR(MAX) = 'SELECT * FROM ' + @UserInput;
-- Dangerous!

-- ✅ Safe với QUOTENAME
DECLARE @SQL NVARCHAR(MAX) = 'SELECT * FROM ' + QUOTENAME(@UserInput);
-- Result: SELECT * FROM [Users; DROP TABLE Users--]
-- Sẽ error vì table name không tồn tại, nhưng không execute DROP


-- ═══════════════════════════════════════════════════════════
-- 3. Generate CREATE TABLE script
-- ═══════════════════════════════════════════════════════════

SELECT
    'CREATE TABLE ' + QUOTENAME(TABLE_SCHEMA) + '.' + QUOTENAME(TABLE_NAME) + ' (' +
    STRING_AGG(
        QUOTENAME(COLUMN_NAME) + ' ' + DATA_TYPE +
        CASE
            WHEN CHARACTER_MAXIMUM_LENGTH IS NOT NULL
            THEN '(' + CAST(CHARACTER_MAXIMUM_LENGTH AS VARCHAR) + ')'
            ELSE ''
        END,
        ', '
    ) + ');' AS create_script
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Users'
GROUP BY TABLE_SCHEMA, TABLE_NAME;


-- ═══════════════════════════════════════════════════════════
-- 4. Build column list dynamically
-- ═══════════════════════════════════════════════════════════

DECLARE @ColumnList NVARCHAR(MAX);

SELECT @ColumnList = STRING_AGG(QUOTENAME(COLUMN_NAME), ', ')
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Users'
  AND COLUMN_NAME NOT IN ('Password', 'Salt');

DECLARE @SQL NVARCHAR(MAX);
SET @SQL = 'SELECT ' + @ColumnList + ' FROM Users';

EXEC sp_executesql @SQL;


-- ═══════════════════════════════════════════════════════════
-- 5. CSV values với proper quoting
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    STRING_AGG(QUOTENAME(skill_name, ''''), ', ') AS skills_csv
FROM user_skills
GROUP BY user_id;

/*
user_id | skills_csv
--------|---------------------------
1       | 'JavaScript', 'Python', 'SQL'
*/
```

### QUOTENAME Alternatives (Other Databases)

```sql
-- ═══════════════════════════════════════════════════════════
-- PostgreSQL: quote_ident() và quote_literal()
-- ═══════════════════════════════════════════════════════════

-- quote_ident: Cho identifiers (table, column names)
SELECT quote_ident('table name');
-- Result: "table name"

-- quote_literal: Cho string literals
SELECT quote_literal('O''Reilly');
-- Result: 'O''Reilly'


-- ═══════════════════════════════════════════════════════════
-- MySQL: Backticks
-- ═══════════════════════════════════════════════════════════

-- Manual quoting
SELECT CONCAT('`', table_name, '`') AS quoted_name
FROM information_schema.tables;


-- ═══════════════════════════════════════════════════════════
-- Oracle: DBMS_ASSERT.ENQUOTE_NAME
-- ═══════════════════════════════════════════════════════════

SELECT DBMS_ASSERT.ENQUOTE_NAME('TABLE_NAME') FROM DUAL;
-- Result: "TABLE_NAME"
```

---

## 4. GROUP_CONCAT (MySQL)

### Cú pháp

```sql
GROUP_CONCAT([DISTINCT] expression
    [ORDER BY expression]
    [SEPARATOR separator])
```

### Basic Usage

```sql
-- ═══════════════════════════════════════════════════════════
-- Basic aggregation
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    GROUP_CONCAT(skill_name) AS skills
FROM user_skills
GROUP BY user_id;
-- Default separator: ','


-- ═══════════════════════════════════════════════════════════
-- Custom separator
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    GROUP_CONCAT(skill_name SEPARATOR ', ') AS skills
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Với ORDER BY
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    GROUP_CONCAT(skill_name ORDER BY skill_name ASC SEPARATOR ', ') AS skills
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Với DISTINCT
-- ═══════════════════════════════════════════════════════════

SELECT
    department_id,
    GROUP_CONCAT(DISTINCT job_title ORDER BY job_title SEPARATOR ' | ') AS positions
FROM employees
GROUP BY department_id;
```

### Advanced Examples

```sql
-- ═══════════════════════════════════════════════════════════
-- Multiple columns
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    GROUP_CONCAT(
        CONCAT(skill_name, ' (', level, ')')
        ORDER BY level DESC
        SEPARATOR ', '
    ) AS skills_with_level
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Conditional aggregation
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    GROUP_CONCAT(
        IF(level IN ('Advanced', 'Expert'), skill_name, NULL)
        SEPARATOR ', '
    ) AS advanced_skills
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Length limit (default: 1024 bytes)
-- ═══════════════════════════════════════════════════════════

-- Tăng limit trong session
SET SESSION group_concat_max_len = 10000;

-- Hoặc trong query
SELECT
    user_id,
    GROUP_CONCAT(skill_name SEPARATOR ', ') AS skills
FROM user_skills
GROUP BY user_id;
```

---

## 5. LISTAGG (Oracle)

### Cú pháp

```sql
LISTAGG(expression [, separator])
    WITHIN GROUP (ORDER BY order_expression)
    [ON OVERFLOW {ERROR | TRUNCATE [literal] {WITH | WITHOUT} COUNT}]
```

### Basic Usage

```sql
-- ═══════════════════════════════════════════════════════════
-- Basic aggregation (WITHIN GROUP là bắt buộc)
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    LISTAGG(skill_name, ', ') WITHIN GROUP (ORDER BY skill_name) AS skills
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Với DISTINCT (Oracle 19c+)
-- ═══════════════════════════════════════════════════════════

SELECT
    department_id,
    LISTAGG(DISTINCT job_title, ', ') WITHIN GROUP (ORDER BY job_title) AS positions
FROM employees
GROUP BY department_id;


-- ═══════════════════════════════════════════════════════════
-- Multiple columns
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    LISTAGG(skill_name || ' (' || level || ')', ', ')
        WITHIN GROUP (ORDER BY level DESC, skill_name) AS skills_with_level
FROM user_skills
GROUP BY user_id;
```

### ON OVERFLOW Clause (Oracle 12.2+)

```sql
-- ═══════════════════════════════════════════════════════════
-- ERROR (default): Raise error nếu vượt quá 4000 chars
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    LISTAGG(skill_name, ', ') WITHIN GROUP (ORDER BY skill_name)
        ON OVERFLOW ERROR
FROM user_skills
GROUP BY user_id;
```

-- ═══════════════════════════════════════════════════════════
-- TRUNCATE: Cắt bớt nếu quá dài
-- ═══════════════════════════════════════════════════════════

SELECT
user_id,
LISTAGG(skill_name, ', ') WITHIN GROUP (ORDER BY skill_name)
ON OVERFLOW TRUNCATE '...' WITH COUNT
FROM user_skills
GROUP BY user_id;
-- Result nếu overflow: "JavaScript, Python, SQL...(+5)"

-- ═══════════════════════════════════════════════════════════
-- TRUNCATE WITHOUT COUNT
-- ═══════════════════════════════════════════════════════════

SELECT
user_id,
LISTAGG(skill_name, ', ') WITHIN GROUP (ORDER BY skill_name)
ON OVERFLOW TRUNCATE '...' WITHOUT COUNT
FROM user_skills
GROUP BY user_id;
-- Result nếu overflow: "JavaScript, Python, SQL..."

````

---

## 6. ARRAY_AGG & ARRAY_TO_STRING (PostgreSQL)

### ARRAY_AGG

```sql
-- ═══════════════════════════════════════════════════════════
-- Tạo array từ nhiều rows
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    ARRAY_AGG(skill_name) AS skills_array
FROM user_skills
GROUP BY user_id;

/*
user_id | skills_array
--------|---------------------------
1       | {JavaScript,Python,SQL}
*/


-- ═══════════════════════════════════════════════════════════
-- Với ORDER BY
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    ARRAY_AGG(skill_name ORDER BY skill_name) AS skills_array
FROM user_skills
GROUP BY user_id;
````

-- ═══════════════════════════════════════════════════════════
-- Với DISTINCT
-- ═══════════════════════════════════════════════════════════

SELECT
department_id,
ARRAY_AGG(DISTINCT job_title ORDER BY job_title) AS positions
FROM employees
GROUP BY department_id;

-- ═══════════════════════════════════════════════════════════
-- Với FILTER
-- ═══════════════════════════════════════════════════════════

SELECT
user_id,
ARRAY_AGG(skill_name) FILTER (WHERE level = 'Expert') AS expert_skills
FROM user_skills
GROUP BY user_id;

````

### ARRAY_TO_STRING

```sql
-- ═══════════════════════════════════════════════════════════
-- Convert array thành string
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    ARRAY_TO_STRING(ARRAY_AGG(skill_name ORDER BY skill_name), ', ') AS skills
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Với NULL replacement
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    ARRAY_TO_STRING(
        ARRAY_AGG(skill_name ORDER BY skill_name),
        ', ',
        'Unknown'  -- Replace NULL với 'Unknown'
    ) AS skills
FROM user_skills
GROUP BY user_id;
````

### Kết hợp ARRAY_AGG với Array Functions

```sql
-- ═══════════════════════════════════════════════════════════
-- Array length
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    ARRAY_AGG(skill_name) AS skills,
    ARRAY_LENGTH(ARRAY_AGG(skill_name), 1) AS skill_count
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Array contains
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    ARRAY_AGG(skill_name) AS skills,
    'Python' = ANY(ARRAY_AGG(skill_name)) AS has_python
FROM user_skills
GROUP BY user_id;


-- ═══════════════════════════════════════════════════════════
-- Array operations
-- ═══════════════════════════════════════════════════════════

SELECT
    user_id,
    ARRAY_AGG(skill_name) || ARRAY['Communication', 'Teamwork'] AS all_skills
FROM user_skills
GROUP BY user_id;
```

---

## 7. Các hàm String liên quan

### CONCAT & CONCAT_WS

```sql
-- ═══════════════════════════════════════════════════════════
-- CONCAT: Nối chuỗi (bỏ qua NULL)
-- ═══════════════════════════════════════════════════════════

SELECT CONCAT('Hello', ' ', 'World');
-- Result: 'Hello World'

SELECT CONCAT('Hello', NULL, 'World');
-- SQL Server: NULL
-- MySQL/PostgreSQL: 'HelloWorld'
```

-- ═══════════════════════════════════════════════════════════
-- CONCAT_WS: Nối với separator (MySQL/PostgreSQL)
-- ═══════════════════════════════════════════════════════════

SELECT CONCAT_WS(', ', 'JavaScript', 'Python', 'SQL');
-- Result: 'JavaScript, Python, SQL'

SELECT CONCAT_WS(', ', 'JavaScript', NULL, 'SQL');
-- Result: 'JavaScript, SQL' (bỏ qua NULL)

-- ═══════════════════════════════════════════════════════════
-- Sử dụng trong aggregation
-- ═══════════════════════════════════════════════════════════

SELECT
user_id,
STRING_AGG(
CONCAT_WS(' - ', skill_name, level, category),
'; '
) AS detailed_skills
FROM user_skills
GROUP BY user_id;

````

### STRING_SPLIT (SQL Server)

```sql
-- ═══════════════════════════════════════════════════════════
-- Tách chuỗi thành rows (ngược lại với STRING_AGG)
-- ═══════════════════════════════════════════════════════════

DECLARE @skills VARCHAR(100) = 'JavaScript, Python, SQL';

SELECT value AS skill
FROM STRING_SPLIT(@skills, ',');

/*
skill
----------
JavaScript
 Python
 SQL
*/


-- ═══════════════════════════════════════════════════════════
-- Với TRIM để loại bỏ spaces
-- ═══════════════════════════════════════════════════════════

SELECT TRIM(value) AS skill
FROM STRING_SPLIT(@skills, ',');
````

-- ═══════════════════════════════════════════════════════════
-- Kết hợp với JOIN
-- ═══════════════════════════════════════════════════════════

SELECT
u.user_id,
u.username,
s.value AS skill
FROM users u
CROSS APPLY STRING_SPLIT(u.skills_csv, ',') s;

````

### STUFF (SQL Server)

```sql
-- ═══════════════════════════════════════════════════════════
-- STUFF: Insert/replace chuỗi tại vị trí cụ thể
-- ═══════════════════════════════════════════════════════════

SELECT STUFF('Hello World', 7, 5, 'SQL');
-- Result: 'Hello SQL' (replace 'World' bằng 'SQL')


-- ═══════════════════════════════════════════════════════════
-- Trick: Loại bỏ separator đầu tiên
-- ═══════════════════════════════════════════════════════════

-- Cách cũ (trước SQL Server 2017)
SELECT
    user_id,
    STUFF((
        SELECT ', ' + skill_name
        FROM user_skills us
        WHERE us.user_id = u.user_id
        ORDER BY skill_name
        FOR XML PATH(''), TYPE
    ).value('.', 'NVARCHAR(MAX)'), 1, 2, '') AS skills
FROM users u;
-- STUFF loại bỏ 2 ký tự đầu tiên (', ')
````

### REPLACE

```sql
-- ═══════════════════════════════════════════════════════════
-- Thay thế chuỗi con
-- ═══════════════════════════════════════════════════════════

SELECT REPLACE('JavaScript, Python, SQL', ', ', ' | ');
-- Result: 'JavaScript | Python | SQL'
```
