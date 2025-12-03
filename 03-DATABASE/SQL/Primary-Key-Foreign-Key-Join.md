# SQL: Primary Key, Foreign Key và JOIN

## 📌 Mục lục

1. [Primary Key](#1-primary-key-khóa-chính)
2. [Foreign Key](#2-foreign-key-khóa-ngoại)
3. [Mối quan hệ giữa các bảng](#3-mối-quan-hệ-giữa-các-bảng)
4. [JOIN](#4-join)
5. [Ví dụ thực tế](#5-ví-dụ-thực-tế-tổng-hợp)

---

## 1. Primary Key (Khóa chính)

### 🎯 Định nghĩa

Primary Key là một cột (hoặc tập hợp các cột) **định danh duy nhất** cho mỗi bản ghi trong bảng.

### 📋 Đặc điểm

| Đặc điểm              | Mô tả                                              |
| --------------------- | -------------------------------------------------- |
| **Duy nhất (UNIQUE)** | Không có 2 bản ghi nào có cùng giá trị Primary Key |
| **Không NULL**        | Không được phép để trống                           |
| **Chỉ có 1**          | Mỗi bảng chỉ có 1 Primary Key                      |
| **Tự động index**     | MySQL tự động tạo index cho Primary Key            |

### 💻 Cú pháp

```sql
-- Cách 1: Khai báo khi tạo cột
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50),
    email VARCHAR(100)
);

-- Cách 2: Khai báo riêng
CREATE TABLE users (
    id INT AUTO_INCREMENT,
    username VARCHAR(50),
    email VARCHAR(100),
    PRIMARY KEY (id)
);

-- Cách 3: Composite Primary Key (nhiều cột)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

### 🖼️ Minh họa trực quan

```
┌─────────────────────────────────────────┐
│              BẢNG: users                │
├─────────┬──────────────┬────────────────┤
│   id    │   username   │     email      │
│   (PK)  │              │                │
├─────────┼──────────────┼────────────────┤
│    1    │   john_doe   │ john@mail.com  │  ← id=1 là DUY NHẤT
│    2    │   jane_doe   │ jane@mail.com  │  ← id=2 là DUY NHẤT
│    3    │   bob_smith  │ bob@mail.com   │  ← id=3 là DUY NHẤT
└─────────┴──────────────┴────────────────┘
         ↑
    PRIMARY KEY
    (Không trùng, không NULL)
```

---

## 2. Foreign Key (Khóa ngoại)

### 🎯 Định nghĩa

Foreign Key là một cột (hoặc tập hợp các cột) trong bảng này **tham chiếu** đến Primary Key của bảng khác, tạo nên **mối quan hệ** giữa 2 bảng.

### 📋 Đặc điểm

| Đặc điểm             | Mô tả                                    |
| -------------------- | ---------------------------------------- |
| **Tham chiếu**       | Trỏ đến Primary Key của bảng khác        |
| **Có thể NULL**      | Cho phép giá trị NULL (tùy thiết kế)     |
| **Có thể trùng**     | Nhiều bản ghi có thể có cùng Foreign Key |
| **Đảm bảo toàn vẹn** | Ngăn chặn dữ liệu "mồ côi"               |

### 💻 Cú pháp

```sql
-- Tạo bảng cha (Parent table)
CREATE TABLE departments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100)
);

-- Tạo bảng con (Child table) với Foreign Key
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- Với các tùy chọn ON DELETE và ON UPDATE
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
        ON DELETE CASCADE      -- Xóa department → xóa luôn employees
        ON UPDATE CASCADE      -- Cập nhật id department → cập nhật FK
);
```

### 🔄 Các tùy chọn ON DELETE / ON UPDATE

| Tùy chọn      | Hành vi                                                      |
| ------------- | ------------------------------------------------------------ |
| `CASCADE`     | Xóa/cập nhật bản ghi cha → tự động xóa/cập nhật bản ghi con  |
| `SET NULL`    | Xóa/cập nhật bản ghi cha → đặt FK của bản ghi con thành NULL |
| `RESTRICT`    | Không cho phép xóa/cập nhật nếu còn bản ghi con tham chiếu   |
| `NO ACTION`   | Tương tự RESTRICT (trong MySQL)                              |
| `SET DEFAULT` | Đặt về giá trị mặc định (MySQL InnoDB không hỗ trợ)          |

### 🖼️ Minh họa trực quan

```
┌─────────────────────────┐          ┌─────────────────────────────────┐
│   BẢNG: departments     │          │        BẢNG: employees          │
├─────────┬───────────────┤          ├─────────┬──────────┬────────────┤
│   id    │     name      │          │   id    │   name   │ dept_id    │
│   (PK)  │               │          │   (PK)  │          │   (FK)     │
├─────────┼───────────────┤          ├─────────┼──────────┼────────────┤
│    1    │      IT       │◄─────────│    1    │   John   │     1      │
│    2    │      HR       │◄─────────│    2    │   Jane   │     2      │
│    3    │    Sales      │◄────┬────│    3    │   Bob    │     1      │
└─────────┴───────────────┘     │    │    4    │   Alice  │     3      │
                                │    │    5    │   Tom    │     3      │
                                │    └─────────┴──────────┴────────────┘
                                │
                    Foreign Key tham chiếu đến Primary Key
```

---

## 3. Mối quan hệ giữa các bảng

### 📊 Các loại quan hệ

#### 3.1. One-to-One (1:1)

Một bản ghi trong bảng A chỉ liên kết với một bản ghi trong bảng B.

```sql
-- Ví dụ: Mỗi user có 1 profile
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50)
);

CREATE TABLE user_profiles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT UNIQUE,  -- UNIQUE đảm bảo 1:1
    bio TEXT,
    avatar VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

```
┌──────────────┐         ┌──────────────────┐
│    users     │         │  user_profiles   │
├──────┬───────┤         ├──────┬───────────┤
│  id  │ name  │         │  id  │  user_id  │
├──────┼───────┤   1:1   ├──────┼───────────┤
│  1   │ John  │◄────────│  1   │     1     │
│  2   │ Jane  │◄────────│  2   │     2     │
└──────┴───────┘         └──────┴───────────┘
```

#### 3.2. One-to-Many (1:N)

Một bản ghi trong bảng A có thể liên kết với nhiều bản ghi trong bảng B.

```sql
-- Ví dụ: Một department có nhiều employees
CREATE TABLE departments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100)
);

CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

```
┌──────────────────┐         ┌──────────────────────┐
│   departments    │         │      employees       │
├──────┬───────────┤         ├──────┬───────┬───────┤
│  id  │   name    │         │  id  │ name  │dept_id│
├──────┼───────────┤   1:N   ├──────┼───────┼───────┤
│  1   │    IT     │◄────────│  1   │ John  │   1   │
│      │           │◄────────│  2   │ Bob   │   1   │
│      │           │◄────────│  3   │ Tom   │   1   │
│  2   │    HR     │◄────────│  4   │ Jane  │   2   │
└──────┴───────────┘         └──────┴───────┴───────┘
```

#### 3.3. Many-to-Many (N:N)

Nhiều bản ghi trong bảng A có thể liên kết với nhiều bản ghi trong bảng B.
→ Cần **bảng trung gian** (Junction/Pivot table)

```sql
-- Ví dụ: Students và Courses (sinh viên đăng ký nhiều môn, môn có nhiều sinh viên)
CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100)
);

CREATE TABLE courses (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200)
);

-- Bảng trung gian
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    enrolled_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

```
┌────────────────┐      ┌─────────────────────┐      ┌────────────────┐
│    students    │      │   student_courses   │      │    courses     │
├──────┬─────────┤      ├────────────┬────────┤      ├──────┬─────────┤
│  id  │  name   │      │ student_id │course_id│     │  id  │  title  │
├──────┼─────────┤      ├────────────┼────────┤      ├──────┼─────────┤
│  1   │  John   │◄─────│     1      │   1    │─────►│  1   │  Math   │
│      │         │◄─────│     1      │   2    │──┐   │      │         │
│  2   │  Jane   │◄─────│     2      │   1    │──│──►│  2   │ English │
│      │         │◄─────│     2      │   3    │──│──►│  3   │ Science │
└──────┴─────────┘      └────────────┴────────┘  │   └──────┴─────────┘
                                                 │
                        John học Math + English ─┘
                        Jane học Math + Science
```

---

## 4. JOIN

### 🎯 Định nghĩa

JOIN là phép kết hợp dữ liệu từ 2 hoặc nhiều bảng dựa trên điều kiện liên kết (thường là Foreign Key - Primary Key).

### 📊 Các loại JOIN

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         CÁC LOẠI JOIN                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   INNER JOIN          LEFT JOIN           RIGHT JOIN        FULL JOIN   │
│   ┌───┬───┐          ┌───┬───┐           ┌───┬───┐         ┌───┬───┐   │
│   │ A │ B │          │ A │ B │           │ A │ B │         │ A │ B │   │
│   │ ┌─┼─┐ │          │███│   │           │   │███│         │███│███│   │
│   │ │█│█│ │          │███│   │           │   │███│         │███│███│   │
│   │ └─┼─┘ │          │███│   │           │   │███│         │███│███│   │
│   └───┴───┘          └───┴───┘           └───┴───┘         └───┴───┘   │
│   Chỉ phần           Toàn bộ A           Toàn bộ B         Toàn bộ     │
│   giao nhau          + phần giao         + phần giao       A và B      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📝 Dữ liệu mẫu cho các ví dụ

```sql
-- Tạo bảng departments
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

INSERT INTO departments VALUES
(1, 'IT'),
(2, 'HR'),
(3, 'Sales'),
(4, 'Marketing');  -- Không có nhân viên nào

-- Tạo bảng employees
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT
);

INSERT INTO employees VALUES
(1, 'John', 1),
(2, 'Jane', 2),
(3, 'Bob', 1),
(4, 'Alice', 3),
(5, 'Tom', NULL);  -- Chưa được phân công department
```

**Dữ liệu hiện tại:**

```
departments                    employees
┌────┬───────────┐            ┌────┬───────┬─────────────┐
│ id │   name    │            │ id │ name  │department_id│
├────┼───────────┤            ├────┼───────┼─────────────┤
│ 1  │    IT     │            │ 1  │ John  │      1      │
│ 2  │    HR     │            │ 2  │ Jane  │      2      │
│ 3  │   Sales   │            │ 3  │ Bob   │      1      │
│ 4  │ Marketing │            │ 4  │ Alice │      3      │
└────┴───────────┘            │ 5  │ Tom   │    NULL     │
                              └────┴───────┴─────────────┘
```

---

### 4.1. INNER JOIN

Chỉ trả về các bản ghi **có dữ liệu khớp ở CẢ HAI bảng**.

```sql
SELECT
    e.id,
    e.name AS employee_name,
    d.name AS department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

**Kết quả:**

```
┌────┬───────────────┬─────────────────┐
│ id │ employee_name │ department_name │
├────┼───────────────┼─────────────────┤
│ 1  │     John      │       IT        │
│ 2  │     Jane      │       HR        │
│ 3  │     Bob       │       IT        │
│ 4  │     Alice     │      Sales      │
└────┴───────────────┴─────────────────┘

❌ Tom không xuất hiện (department_id = NULL)
❌ Marketing không xuất hiện (không có nhân viên)
```

**Minh họa:**

```
employees                              departments
┌───────┬─────────────┐               ┌────┬───────────┐
│ name  │department_id│               │ id │   name    │
├───────┼─────────────┤               ├────┼───────────┤
│ John  │      1      │───────────────│ 1  │    IT     │ ✓
│ Jane  │      2      │───────────────│ 2  │    HR     │ ✓
│ Bob   │      1      │───────────────│    │           │
│ Alice │      3      │───────────────│ 3  │   Sales   │ ✓
│ Tom   │    NULL     │      ✗        │ 4  │ Marketing │ ✗
└───────┴─────────────┘               └────┴───────────┘
                    ↑
            INNER JOIN chỉ lấy
            những cặp khớp nhau
```

---

### 4.2. LEFT JOIN (LEFT OUTER JOIN)

Trả về **TẤT CẢ bản ghi từ bảng TRÁI** + bản ghi khớp từ bảng phải.
Nếu không khớp → cột bảng phải = NULL.

```sql
SELECT
    e.id,
    e.name AS employee_name,
    d.name AS department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

**Kết quả:**

```
┌────┬───────────────┬─────────────────┐
│ id │ employee_name │ department_name │
├────┼───────────────┼─────────────────┤
│ 1  │     John      │       IT        │
│ 2  │     Jane      │       HR        │
│ 3  │     Bob       │       IT        │
│ 4  │     Alice     │      Sales      │
│ 5  │     Tom       │      NULL       │  ← Tom vẫn xuất hiện
└────┴───────────────┴─────────────────┘

✓ Tất cả employees đều xuất hiện
❌ Marketing không xuất hiện (không phải bảng trái)
```

**Minh họa:**

```
employees (BẢNG TRÁI)                  departments (BẢNG PHẢI)
┌───────┬─────────────┐               ┌────┬───────────┐
│ name  │department_id│               │ id │   name    │
├───────┼─────────────┤               ├────┼───────────┤
│ John  │      1      │───────────────│ 1  │    IT     │ ✓
│ Jane  │      2      │───────────────│ 2  │    HR     │ ✓
│ Bob   │      1      │───────────────│    │           │
│ Alice │      3      │───────────────│ 3  │   Sales   │ ✓
│ Tom   │    NULL     │──────► NULL   │ 4  │ Marketing │ ✗
└───────┴─────────────┘               └────┴───────────┘
    ↑
  TẤT CẢ bản ghi
  bảng trái được giữ
```

**Ứng dụng:** Tìm employees chưa có department:

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;

-- Kết quả: Tom
```

---

### 4.3. RIGHT JOIN (RIGHT OUTER JOIN)

Trả về **TẤT CẢ bản ghi từ bảng PHẢI** + bản ghi khớp từ bảng trái.
Nếu không khớp → cột bảng trái = NULL.

```sql
SELECT
    e.id,
    e.name AS employee_name,
    d.name AS department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

**Kết quả:**

```
┌──────┬───────────────┬─────────────────┐
│  id  │ employee_name │ department_name │
├──────┼───────────────┼─────────────────┤
│  1   │     John      │       IT        │
│  3   │     Bob       │       IT        │
│  2   │     Jane      │       HR        │
│  4   │     Alice     │      Sales      │
│ NULL │     NULL      │    Marketing    │  ← Marketing vẫn xuất hiện
└──────┴───────────────┴─────────────────┘

❌ Tom không xuất hiện (không phải bảng phải)
✓ Tất cả departments đều xuất hiện
```

**Minh họa:**

```
employees (BẢNG TRÁI)                  departments (BẢNG PHẢI)
┌───────┬─────────────┐               ┌────┬───────────┐
│ name  │department_id│               │ id │   name    │
├───────┼─────────────┤               ├────┼───────────┤
│ John  │      1      │───────────────│ 1  │    IT     │ ✓
│ Jane  │      2      │───────────────│ 2  │    HR     │ ✓
│ Bob   │      1      │───────────────│    │           │
│ Alice │      3      │───────────────│ 3  │   Sales   │ ✓
│ Tom   │    NULL     │ ✗      NULL ◄─│ 4  │ Marketing │
└───────┴─────────────┘               └────┴───────────┘
                                           ↑
                                      TẤT CẢ bản ghi
                                      bảng phải được giữ
```

**Ứng dụng:** Tìm departments không có employee:

```sql
SELECT d.name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id
WHERE e.id IS NULL;

-- Kết quả: Marketing
```

---

### 4.4. FULL OUTER JOIN

Trả về **TẤT CẢ bản ghi từ CẢ HAI bảng**.
Nếu không khớp → cột tương ứng = NULL.

> ⚠️ **Lưu ý:** MySQL không hỗ trợ FULL OUTER JOIN trực tiếp. Cần dùng UNION.

```sql
-- MySQL: Mô phỏng FULL OUTER JOIN
SELECT
    e.id,
    e.name AS employee_name,
    d.name AS department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id

UNION

SELECT
    e.id,
    e.name AS employee_name,
    d.name AS department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

**Kết quả:**

```
┌──────┬───────────────┬─────────────────┐
│  id  │ employee_name │ department_name │
├──────┼───────────────┼─────────────────┤
│  1   │     John      │       IT        │
│  2   │     Jane      │       HR        │
│  3   │     Bob       │       IT        │
│  4   │     Alice     │      Sales      │
│  5   │     Tom       │      NULL       │  ← Employee không có dept
│ NULL │     NULL      │    Marketing    │  ← Dept không có employee
└──────┴───────────────┴─────────────────┘

✓ Tất cả employees xuất hiện
✓ Tất cả departments xuất hiện
```

---

### 4.5. CROSS JOIN (Cartesian Product)

Kết hợp **MỖI bản ghi** của bảng A với **MỖI bản ghi** của bảng B.
Số bản ghi kết quả = số bản ghi A × số bản ghi B.

```sql
SELECT
    e.name AS employee,
    d.name AS department
FROM employees e
CROSS JOIN departments d;
```

**Kết quả:** 5 employees × 4 departments = 20 bản ghi

```
┌──────────┬───────────┐
│ employee │department │
├──────────┼───────────┤
│   John   │    IT     │
│   John   │    HR     │
│   John   │   Sales   │
│   John   │ Marketing │
│   Jane   │    IT     │
│   Jane   │    HR     │
│   ...    │    ...    │  (20 dòng)
└──────────┴───────────┘
```

**Ứng dụng:** Tạo tất cả các tổ hợp có thể (ví dụ: size × color cho sản phẩm).

---

### 4.6. SELF JOIN

Bảng tự JOIN với chính nó. Thường dùng cho dữ liệu có cấu trúc phân cấp.

```sql
-- Ví dụ: Bảng employees có cột manager_id
CREATE TABLE employees_v2 (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    manager_id INT
);

INSERT INTO employees_v2 VALUES
(1, 'CEO', NULL),
(2, 'CTO', 1),
(3, 'Developer', 2),
(4, 'Designer', 2);

-- Tìm tên manager của mỗi employee
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees_v2 e
LEFT JOIN employees_v2 m ON e.manager_id = m.id;
```

**Kết quả:**

```
┌───────────┬─────────┐
│ employee  │ manager │
├───────────┼─────────┤
│    CEO    │  NULL   │
│    CTO    │   CEO   │
│ Developer │   CTO   │
│ Designer  │   CTO   │
└───────────┴─────────┘
```

---

### 📊 So sánh tổng hợp các loại JOIN

| JOIN Type      | Bảng trái | Bảng phải | Không khớp              |
| -------------- | --------- | --------- | ----------------------- |
| **INNER JOIN** | Chỉ khớp  | Chỉ khớp  | Bỏ qua                  |
| **LEFT JOIN**  | Tất cả    | Chỉ khớp  | NULL cho bảng phải      |
| **RIGHT JOIN** | Chỉ khớp  | Tất cả    | NULL cho bảng trái      |
| **FULL JOIN**  | Tất cả    | Tất cả    | NULL cho bên không khớp |
| **CROSS JOIN** | Tất cả    | Tất cả    | Tạo mọi tổ hợp          |

---

## 5. Ví dụ thực tế tổng hợp

### 🛒 Hệ thống E-commerce

#### Thiết kế Database

```sql
-- 1. Bảng khách hàng
CREATE TABLE customers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Bảng danh mục sản phẩm
CREATE TABLE categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    parent_id INT NULL,
    FOREIGN KEY (parent_id) REFERENCES categories(id)  -- Self-reference cho subcategory
);

-- 3. Bảng sản phẩm
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(200) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    stock INT DEFAULT 0,
    category_id INT,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

-- 4. Bảng đơn hàng
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    total_amount DECIMAL(12, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- 5. Bảng chi tiết đơn hàng (Junction table cho N:N giữa orders và products)
CREATE TABLE order_items (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id),
    UNIQUE KEY unique_order_product (order_id, product_id)
);
```

#### Sơ đồ quan hệ (ERD)

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│   categories    │       │    products     │       │   order_items   │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ id (PK)         │◄──────│ category_id(FK) │       │ id (PK)         │
│ name            │       │ id (PK)         │◄──────│ product_id (FK) │
│ parent_id (FK)──┼──┐    │ name            │       │ order_id (FK)───┼──┐
└─────────────────┘  │    │ price           │       │ quantity        │  │
                     │    │ stock           │       │ unit_price      │  │
                     │    └─────────────────┘       └─────────────────┘  │
                     │                                                   │
                     └──► Self-reference                                 │
                                                                         │
┌─────────────────┐       ┌─────────────────┐                           │
│   customers     │       │     orders      │◄──────────────────────────┘
├─────────────────┤       ├─────────────────┤
│ id (PK)         │◄──────│ customer_id(FK) │
│ name            │       │ id (PK)         │
│ email           │       │ order_date      │
│ created_at      │       │ status          │
└─────────────────┘       │ total_amount    │
                          └─────────────────┘
```

#### Dữ liệu mẫu

```sql
-- Thêm categories
INSERT INTO categories (id, name, parent_id) VALUES
(1, 'Electronics', NULL),
(2, 'Phones', 1),
(3, 'Laptops', 1),
(4, 'Clothing', NULL),
(5, 'Men', 4);

-- Thêm products
INSERT INTO products (name, price, stock, category_id) VALUES
('iPhone 15', 999.00, 50, 2),
('Samsung Galaxy S24', 899.00, 30, 2),
('MacBook Pro', 1999.00, 20, 3),
('Dell XPS 15', 1499.00, 15, 3),
('T-Shirt Basic', 29.99, 100, 5);

-- Thêm customers
INSERT INTO customers (name, email) VALUES
('Nguyen Van A', 'nguyenvana@email.com'),
('Tran Thi B', 'tranthib@email.com'),
('Le Van C', 'levanc@email.com');

-- Thêm orders
INSERT INTO orders (customer_id, status, total_amount) VALUES
(1, 'delivered', 1998.00),
(1, 'processing', 899.00),
(2, 'shipped', 2028.99);

-- Thêm order_items
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(1, 1, 2, 999.00),      -- Order 1: 2 iPhone 15
(2, 2, 1, 899.00),      -- Order 2: 1 Samsung
(3, 3, 1, 1999.00),     -- Order 3: 1 MacBook
(3, 5, 1, 29.99);       -- Order 3: 1 T-Shirt
```

---

### 📝 Các truy vấn thực tế

#### 1. Lấy thông tin đơn hàng đầy đủ

```sql
SELECT
    o.id AS order_id,
    c.name AS customer_name,
    c.email,
    o.order_date,
    o.status,
    o.total_amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
ORDER BY o.order_date DESC;
```

**Kết quả:**

```
┌──────────┬───────────────┬──────────────────────┬─────────────────────┬────────────┬──────────────┐
│ order_id │ customer_name │        email         │     order_date      │   status   │ total_amount │
├──────────┼───────────────┼──────────────────────┼─────────────────────┼────────────┼──────────────┤
│    3     │  Tran Thi B   │ tranthib@email.com   │ 2024-01-15 10:30:00 │  shipped   │   2028.99    │
│    2     │ Nguyen Van A  │ nguyenvana@email.com │ 2024-01-14 15:20:00 │ processing │    899.00    │
│    1     │ Nguyen Van A  │ nguyenvana@email.com │ 2024-01-10 09:00:00 │ delivered  │   1998.00    │
└──────────┴───────────────┴──────────────────────┴─────────────────────┴────────────┴──────────────┘
```

#### 2. Chi tiết đơn hàng với sản phẩm

```sql
SELECT
    o.id AS order_id,
    c.name AS customer,
    p.name AS product,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) AS subtotal
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id
WHERE o.id = 3;
```

**Kết quả:**

```
┌──────────┬───────────┬─────────────┬──────────┬────────────┬──────────┐
│ order_id │ customer  │   product   │ quantity │ unit_price │ subtotal │
├──────────┼───────────┼─────────────┼──────────┼────────────┼──────────┤
│    3     │ Tran Thi B│ MacBook Pro │    1     │  1999.00   │ 1999.00  │
│    3     │ Tran Thi B│ T-Shirt Basic│   1     │   29.99    │  29.99   │
└──────────┴───────────┴─────────────┴──────────┴────────────┴──────────┘
```

#### 3. Sản phẩm theo danh mục (với subcategory)

```sql
SELECT
    p.name AS product,
    p.price,
    c.name AS category,
    parent.name AS parent_category
FROM products p
INNER JOIN categories c ON p.category_id = c.id
LEFT JOIN categories parent ON c.parent_id = parent.id;
```

**Kết quả:**

```
┌────────────────────┬─────────┬──────────┬─────────────────┐
│      product       │  price  │ category │ parent_category │
├────────────────────┼─────────┼──────────┼─────────────────┤
│     iPhone 15      │  999.00 │  Phones  │   Electronics   │
│ Samsung Galaxy S24 │  899.00 │  Phones  │   Electronics   │
│    MacBook Pro     │ 1999.00 │ Laptops  │   Electronics   │
│    Dell XPS 15     │ 1499.00 │ Laptops  │   Electronics   │
│   T-Shirt Basic    │  29.99  │   Men    │    Clothing     │
└────────────────────┴─────────┴──────────┴─────────────────┘
```

#### 4. Khách hàng chưa có đơn hàng nào

```sql
SELECT
    c.id,
    c.name,
    c.email
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

**Kết quả:**

```
┌────┬──────────┬──────────────────┐
│ id │   name   │      email       │
├────┼──────────┼──────────────────┤
│ 3  │ Le Van C │ levanc@email.com │
└────┴──────────┴──────────────────┘
```

#### 5. Sản phẩm chưa được mua

```sql
SELECT
    p.id,
    p.name,
    p.price,
    p.stock
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
WHERE oi.id IS NULL;
```

**Kết quả:**

```
┌────┬─────────────┬─────────┬───────┐
│ id │    name     │  price  │ stock │
├────┼─────────────┼─────────┼───────┤
│ 4  │ Dell XPS 15 │ 1499.00 │  15   │
└────┴─────────────┴─────────┴───────┘
```

#### 6. Thống kê doanh thu theo khách hàng

```sql
SELECT
    c.name AS customer,
    COUNT(DISTINCT o.id) AS total_orders,
    SUM(o.total_amount) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name
ORDER BY total_spent DESC;
```

**Kết quả:**

```
┌───────────────┬──────────────┬─────────────┐
│   customer    │ total_orders │ total_spent │
├───────────────┼──────────────┼─────────────┤
│ Nguyen Van A  │      2       │   2897.00   │
│  Tran Thi B   │      1       │   2028.99   │
│   Le Van C    │      0       │    NULL     │
└───────────────┴──────────────┴─────────────┘
```

#### 7. Top sản phẩm bán chạy

```sql
SELECT
    p.name AS product,
    SUM(oi.quantity) AS total_sold,
    SUM(oi.quantity * oi.unit_price) AS revenue
FROM products p
INNER JOIN order_items oi ON p.id = oi.product_id
GROUP BY p.id, p.name
ORDER BY total_sold DESC
LIMIT 5;
```

**Kết quả:**

```
┌────────────────────┬────────────┬─────────┐
│      product       │ total_sold │ revenue │
├────────────────────┼────────────┼─────────┤
│     iPhone 15      │     2      │ 1998.00 │
│    MacBook Pro     │     1      │ 1999.00 │
│ Samsung Galaxy S24 │     1      │  899.00 │
│   T-Shirt Basic    │     1      │   29.99 │
└────────────────────┴────────────┴─────────┘
```

---

## 📌 Tổng kết

| Khái niệm       | Mục đích                       | Lưu ý                          |
| --------------- | ------------------------------ | ------------------------------ |
| **Primary Key** | Định danh duy nhất mỗi bản ghi | Không NULL, không trùng        |
| **Foreign Key** | Tạo quan hệ giữa các bảng      | Đảm bảo toàn vẹn dữ liệu       |
| **INNER JOIN**  | Lấy dữ liệu khớp cả 2 bảng     | Dùng nhiều nhất                |
| **LEFT JOIN**   | Giữ tất cả bảng trái           | Tìm dữ liệu "mồ côi"           |
| **RIGHT JOIN**  | Giữ tất cả bảng phải           | Ít dùng, có thể đổi thành LEFT |
| **FULL JOIN**   | Giữ tất cả 2 bảng              | MySQL cần dùng UNION           |

---

## 🔗 Tài liệu tham khảo

- [MySQL Documentation](https://dev.mysql.com/doc/)
- [W3Schools SQL Tutorial](https://www.w3schools.com/sql/)
- [SQLBolt - Interactive SQL Lessons](https://sqlbolt.com/)
