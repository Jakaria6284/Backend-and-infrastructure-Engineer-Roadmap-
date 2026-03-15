# SQL Mastery — Subqueries & JOINs: Backend Engineer's Complete Guide

> **Target:** Standard ANSI SQL — runs on MySQL 8+, SQL Server, SQLite, MariaDB without modification.
> Engine-specific differences are noted inline.

---

## Table of Contents

1. [What Is a Subquery?](#1-what-is-a-subquery)
2. [What Is a JOIN?](#2-what-is-a-join)
3. [Schema Setup — Our Practice Database](#3-schema-setup--our-practice-database)
4. [Non-Correlated Subqueries](#4-non-correlated-subqueries)
5. [Correlated Subqueries](#5-correlated-subqueries)
6. [JOIN Types — All Concepts](#6-join-types--all-concepts)
   - 6.1 INNER JOIN
   - 6.2 LEFT JOIN
   - 6.3 RIGHT JOIN
   - 6.4 FULL OUTER JOIN
   - 6.5 CROSS JOIN
   - 6.6 SELF JOIN
   - 6.7 Multiple JOINs (Chaining)
   - 6.8 JOIN with Subqueries (Bridge Pattern)
   - 6.9 JOIN vs Subquery — When to Use Which
7. [Subquery Placement Cheatsheet](#7-subquery-placement-cheatsheet)
8. [JOIN Cheatsheet](#8-join-cheatsheet)
9. [Practice Questions](#9-practice-questions)
10. [Solutions](#10-solutions)
11. [Performance and Pitfalls](#11-performance-and-pitfalls)
12. [Quick Reference](#12-quick-reference)

---

## 1. What Is a Subquery?

A **subquery** is a `SELECT` statement embedded inside another SQL statement.
The outer statement is called the **outer query**.

```sql
SELECT column_name
FROM   some_table
WHERE  column_name = (SELECT column_name FROM other_table WHERE condition);
--                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
--                          This is the subquery (inner query)
```

### The Two Fundamental Types

| Type | Definition | Runs How Many Times? | Key Trait |
|---|---|---|---|
| **Non-Correlated** | Completely independent of the outer query | **Once** | Can be cut out and run on its own |
| **Correlated** | References at least one column from the outer query | **Once per outer row** | Cannot run alone — needs outer row data |

### Mental Model

```
NON-CORRELATED
--------------------------------------------------
  Outer Query  ->  "Inner query, compute a result."
  Inner Query  ->  Runs ONCE. Returns one value or a list.
  Outer Query  ->  Uses that fixed result for every row it checks.

CORRELATED
--------------------------------------------------
  Outer Query: for Row 1  ->  "Run using my Row 1 data."
  Inner Query             ->  Executes. Returns result for Row 1.
  Outer Query: for Row 2  ->  "Run using my Row 2 data."
  Inner Query             ->  Executes again for Row 2.
  ... (repeats once per outer row)
```

### One Rule to Tell Them Apart

> Does the subquery reference any column from the outer query's table?
> **Yes** -> Correlated. **No** -> Non-Correlated.

```sql
-- NON-CORRELATED: inner query has no reference to outer table
SELECT full_name FROM employees
WHERE  salary > (SELECT AVG(salary) FROM employees);

-- CORRELATED: e1.department_id comes from the outer row
SELECT full_name FROM employees e1
WHERE  salary > (SELECT AVG(salary) FROM employees e2
                 WHERE  e2.department_id = e1.department_id);
--                                        ^^^^^^^^^^^^^^^^  <-- outer reference
```

---

## 2. What Is a JOIN?

A **JOIN** combines rows from two or more tables based on a related column between them.
Where a subquery computes something and passes it in, a JOIN pulls the data side by side.

```sql
SELECT a.col, b.col
FROM   table_a a
JOIN   table_b b ON a.shared_key = b.shared_key;
--                  ^^^^^^^^^^^^^^^^^^^^^^^^^
--                  The join condition
```

### The JOIN Family

| JOIN Type | What It Returns |
|---|---|
| `INNER JOIN` | Only rows where the join condition matches in BOTH tables |
| `LEFT JOIN` | All rows from the left table; NULLs for unmatched right rows |
| `RIGHT JOIN` | All rows from the right table; NULLs for unmatched left rows |
| `FULL OUTER JOIN` | All rows from both tables; NULLs where either side has no match |
| `CROSS JOIN` | Every row in left × every row in right (Cartesian product) |
| `SELF JOIN` | A table joined to itself (uses two aliases for the same table) |

### Visual Representation

```
Table A         Table B
--------        --------
  1               1
  2               2
  3               4  (no 3)
(no 5)            5

INNER JOIN  ->  rows: 1, 2          (only matches)
LEFT JOIN   ->  rows: 1, 2, 3       (all of A; 3 has NULL for B side)
RIGHT JOIN  ->  rows: 1, 2, 4, 5    (all of B; 5 has NULL for A side)
FULL OUTER  ->  rows: 1, 2, 3, 4, 5 (everything; NULLs where no match)
CROSS JOIN  ->  3 × 4 = 12 rows     (every A row paired with every B row)
```

---

## 3. Schema Setup — Our Practice Database

### 3.1 Compatibility Notes

| Feature | MySQL 8 | SQL Server | SQLite | MariaDB |
|---|---|---|---|---|
| Auto-increment | `AUTO_INCREMENT` | `IDENTITY(1,1)` | `AUTOINCREMENT` | `AUTO_INCREMENT` |
| Current timestamp | `CURRENT_TIMESTAMP` | `GETDATE()` | `CURRENT_TIMESTAMP` | `CURRENT_TIMESTAMP` |
| String concat | `CONCAT(a,b)` | `CONCAT(a,b)` | `a \|\| b` | `CONCAT(a,b)` |
| Limit rows | `LIMIT n` | `TOP n` / `FETCH FIRST n ROWS ONLY` | `LIMIT n` | `LIMIT n` |
| FULL OUTER JOIN | Not supported — emulate with UNION | Supported | Not supported — emulate | Supported |

---

### 3.2 Create Tables

```sql
-- ================================================================
-- TABLE: departments
-- ================================================================
CREATE TABLE departments (
    department_id   INT          NOT NULL,
    dept_name       VARCHAR(100) NOT NULL,
    budget          DECIMAL(12,2),
    location        VARCHAR(100),
    CONSTRAINT pk_departments PRIMARY KEY (department_id)
);

-- ================================================================
-- TABLE: employees  (self-referential: manager_id -> employee_id)
-- ================================================================
CREATE TABLE employees (
    employee_id     INT          NOT NULL,
    full_name       VARCHAR(150) NOT NULL,
    email           VARCHAR(200) NOT NULL,
    department_id   INT,
    manager_id      INT,
    salary          DECIMAL(10,2),
    hire_date       DATE,
    job_title       VARCHAR(100),
    CONSTRAINT pk_employees   PRIMARY KEY (employee_id),
    CONSTRAINT uq_emp_email   UNIQUE      (email),
    CONSTRAINT fk_emp_dept    FOREIGN KEY (department_id) REFERENCES departments(department_id),
    CONSTRAINT fk_emp_manager FOREIGN KEY (manager_id)   REFERENCES employees(employee_id)
);

-- ================================================================
-- TABLE: customers
-- ================================================================
CREATE TABLE customers (
    customer_id     INT          NOT NULL,
    full_name       VARCHAR(150) NOT NULL,
    email           VARCHAR(200) NOT NULL,
    country         VARCHAR(100),
    created_at      DATE,
    CONSTRAINT pk_customers  PRIMARY KEY (customer_id),
    CONSTRAINT uq_cust_email UNIQUE      (email)
);

-- ================================================================
-- TABLE: products
-- ================================================================
CREATE TABLE products (
    product_id      INT          NOT NULL,
    product_name    VARCHAR(200) NOT NULL,
    category        VARCHAR(100),
    unit_price      DECIMAL(10,2),
    stock_qty       INT DEFAULT 0,
    CONSTRAINT pk_products PRIMARY KEY (product_id)
);

-- ================================================================
-- TABLE: orders
-- ================================================================
CREATE TABLE orders (
    order_id        INT          NOT NULL,
    customer_id     INT,
    order_date      DATE         NOT NULL,
    status          VARCHAR(20),
    total_amount    DECIMAL(12,2),
    CONSTRAINT pk_orders      PRIMARY KEY (order_id),
    CONSTRAINT fk_ord_cust    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    CONSTRAINT chk_ord_status CHECK (status IN ('pending','processing','shipped','delivered','cancelled'))
);

-- ================================================================
-- TABLE: order_items
-- ================================================================
CREATE TABLE order_items (
    item_id         INT NOT NULL,
    order_id        INT,
    product_id      INT,
    quantity        INT          NOT NULL,
    unit_price      DECIMAL(10,2),
    CONSTRAINT pk_order_items PRIMARY KEY (item_id),
    CONSTRAINT fk_oi_order    FOREIGN KEY (order_id)   REFERENCES orders(order_id),
    CONSTRAINT fk_oi_product  FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- ================================================================
-- TABLE: reviews
-- ================================================================
CREATE TABLE reviews (
    review_id       INT NOT NULL,
    product_id      INT,
    customer_id     INT,
    rating          INT,
    review_text     VARCHAR(1000),
    reviewed_at     DATE,
    CONSTRAINT pk_reviews      PRIMARY KEY (review_id),
    CONSTRAINT fk_rev_product  FOREIGN KEY (product_id)  REFERENCES products(product_id),
    CONSTRAINT fk_rev_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    CONSTRAINT chk_rating      CHECK (rating BETWEEN 1 AND 5)
);
```

---

### 3.3 Seed Data

```sql
-- departments
INSERT INTO departments (department_id, dept_name, budget, location) VALUES
(1, 'Engineering', 500000.00, 'New York'),
(2, 'Marketing',   200000.00, 'San Francisco'),
(3, 'Sales',       350000.00, 'Chicago'),
(4, 'HR',          150000.00, 'New York'),
(5, 'Finance',     300000.00, 'Boston');

-- employees (managers inserted first)
INSERT INTO employees (employee_id, full_name, email, department_id, manager_id, salary, hire_date, job_title) VALUES
(1,  'Alice Thompson', 'alice@company.com',  1, NULL, 120000, '2018-03-15', 'CTO'),
(2,  'Bob Martinez',   'bob@company.com',    3, NULL,  95000, '2017-06-01', 'VP Sales'),
(3,  'Carol White',    'carol@company.com',  1,    1,  85000, '2019-07-20', 'Senior Engineer'),
(4,  'David Kim',      'david@company.com',  1,    1,  80000, '2020-01-10', 'Backend Engineer'),
(5,  'Eve Johnson',    'eve@company.com',    1,    1,  75000, '2020-09-15', 'Frontend Engineer'),
(6,  'Frank Lee',      'frank@company.com',  1,    3,  70000, '2021-03-01', 'Junior Engineer'),
(7,  'Grace Hall',     'grace@company.com',  1,    3,  72000, '2021-05-12', 'Junior Engineer'),
(8,  'Henry Brown',    'henry@company.com',  3,    2,  65000, '2019-08-10', 'Sales Lead'),
(9,  'Iris Davis',     'iris@company.com',   3,    2,  62000, '2020-02-14', 'Sales Rep'),
(10, 'Jack Wilson',    'jack@company.com',   3,    2,  60000, '2021-07-01', 'Sales Rep'),
(11, 'Karen Moore',    'karen@company.com',  2, NULL,  88000, '2018-11-01', 'Marketing Director'),
(12, 'Leo Garcia',     'leo@company.com',    2,   11,  68000, '2020-06-15', 'Marketing Manager'),
(13, 'Mia Taylor',     'mia@company.com',    4, NULL,  78000, '2017-04-20', 'HR Director'),
(14, 'Nate Anderson',  'nate@company.com',   4,   13,  58000, '2021-09-01', 'HR Specialist'),
(15, 'Olivia Thomas',  'olivia@company.com', 5, NULL,  92000, '2016-01-15', 'CFO'),
(16, 'Paul Jackson',   'paul@company.com',   5,   15,  72000, '2019-03-25', 'Senior Analyst');

-- customers
INSERT INTO customers (customer_id, full_name, email, country, created_at) VALUES
(1,  'Samantha Reed', 'sam@example.com',    'USA',       '2022-01-10'),
(2,  'Tom Harris',    'tom@example.com',    'Canada',    '2022-03-15'),
(3,  'Uma Patel',     'uma@example.com',    'India',     '2022-05-20'),
(4,  'Victor Chen',   'victor@example.com', 'USA',       '2022-07-05'),
(5,  'Wendy Scott',   'wendy@example.com',  'UK',        '2022-08-30'),
(6,  'Xander Young',  'xander@example.com', 'Australia', '2022-09-15'),
(7,  'Yara Green',    'yara@example.com',   'Germany',   '2023-01-20'),
(8,  'Zach Baker',    'zach@example.com',   'USA',       '2023-03-10'),
(9,  'Amy Clark',     'amy@example.com',    'Canada',    '2023-05-05'),
(10, 'Brian Lewis',   'brian@example.com',  'USA',       '2023-06-20');

-- products
INSERT INTO products (product_id, product_name, category, unit_price, stock_qty) VALUES
(1,  'Wireless Keyboard',           'Electronics', 89.99,  200),
(2,  'USB-C Hub',                   'Electronics', 49.99,  350),
(3,  'Noise-Cancelling Headphones', 'Electronics', 199.99, 120),
(4,  'Mechanical Keyboard',         'Electronics', 149.99,  80),
(5,  'Webcam HD 1080p',             'Electronics', 79.99,  260),
(6,  'Laptop Stand',                'Accessories', 39.99,  400),
(7,  'Monitor Arm',                 'Accessories', 69.99,  150),
(8,  'Cable Management Kit',        'Accessories', 19.99,  600),
(9,  'Ergonomic Mouse',             'Electronics', 59.99,  310),
(10, 'Standing Desk Mat',           'Accessories', 49.99,  180),
(11, 'Blue Light Glasses',          'Wellness',    29.99,  500),
(12, 'Desk Lamp LED',               'Accessories', 44.99,  220),
(13, 'Portable SSD 1TB',            'Storage',     109.99,  90),
(14, 'Cloud Backup Plan 1Y',        'Software',    59.99,  999),
(15, 'VPN Subscription 1Y',         'Software',    39.99,  999);

-- orders
INSERT INTO orders (order_id, customer_id, order_date, status, total_amount) VALUES
(1,  1,  '2023-01-15', 'delivered',  339.97),
(2,  1,  '2023-04-20', 'delivered',  199.99),
(3,  2,  '2023-02-10', 'delivered',  129.98),
(4,  3,  '2023-03-05', 'delivered',  259.98),
(5,  4,  '2023-04-01', 'shipped',     89.99),
(6,  4,  '2023-07-10', 'delivered',  219.98),
(7,  5,  '2023-05-15', 'delivered',   49.99),
(8,  6,  '2023-06-20', 'processing', 289.98),
(9,  7,  '2023-07-01', 'delivered',  159.98),
(10, 8,  '2023-08-15', 'pending',    109.99),
(11, 1,  '2023-09-01', 'delivered',  449.97),
(12, 9,  '2023-09-10', 'delivered',   79.99),
(13, 10, '2023-10-05', 'cancelled',  149.99),
(14, 2,  '2023-10-20', 'delivered',  219.98),
(15, 3,  '2023-11-15', 'shipped',    109.99);

-- order_items
INSERT INTO order_items (item_id, order_id, product_id, quantity, unit_price) VALUES
(1,  1,  1, 1,  89.99), (2,  1,  3, 1, 199.99), (3,  1,  8, 2,  19.99),
(4,  2,  3, 1, 199.99), (5,  3,  1, 1,  89.99), (6,  3,  6, 1,  39.99),
(7,  4,  4, 1, 149.99), (8,  4,  9, 1,  59.99), (9,  4, 11, 1,  29.99),
(10, 5,  1, 1,  89.99), (11, 6,  5, 1,  79.99), (12, 6,  7, 1,  69.99),
(13, 6, 12, 1,  44.99), (14, 7, 10, 1,  49.99), (15, 8,  2, 2,  49.99),
(16, 8,  9, 1,  59.99), (17, 9, 14, 1,  59.99), (18, 9, 15, 1,  39.99),
(19, 9,  6, 1,  39.99), (20,10, 13, 1, 109.99), (21,11,  3, 1, 199.99),
(22,11,  4, 1, 149.99), (23,11, 13, 1, 109.99), (24,12,  5, 1,  79.99),
(25,13,  4, 1, 149.99), (26,14,  7, 1,  69.99), (27,14,  5, 1,  79.99),
(28,14, 12, 1,  44.99), (29,15, 13, 1, 109.99);

-- reviews
INSERT INTO reviews (review_id, product_id, customer_id, rating, review_text, reviewed_at) VALUES
(1,  3, 1, 5, 'Amazing noise cancellation!',      '2023-01-25'),
(2,  1, 2, 4, 'Great keyboard, fast delivery.',   '2023-02-20'),
(3,  4, 3, 5, 'Best mechanical keyboard ever.',   '2023-03-20'),
(4,  9, 3, 4, 'Comfortable and precise.',         '2023-03-22'),
(5,  1, 4, 3, 'Decent product, average quality.', '2023-04-15'),
(6,  3, 1, 4, 'Still great on second purchase.',  '2023-09-10'),
(7,  5, 6, 5, 'Crystal clear video quality.',     '2023-07-05'),
(8, 13,10, 5, 'Fast transfer speeds.',            '2023-10-20'),
(9,  7, 2, 4, 'Sturdy monitor arm.',              '2023-11-05'),
(10,14, 7, 5, 'Great cloud service.',             '2023-07-15');
```

---

### 3.4 Table Relationships

```
departments  ---<  employees    (employees.department_id  -> departments.department_id)
employees    ---<  employees    (employees.manager_id     -> employees.employee_id)  [self-join]
customers    ---<  orders       (orders.customer_id       -> customers.customer_id)
orders       ---<  order_items  (order_items.order_id     -> orders.order_id)
products     ---<  order_items  (order_items.product_id   -> products.product_id)
products     ---<  reviews      (reviews.product_id       -> products.product_id)
customers    ---<  reviews      (reviews.customer_id      -> customers.customer_id)
```

---

## 4. Non-Correlated Subqueries

### Definition

Self-contained. No reference to the outer query. Runs **once**. Result substituted into the outer query.

```
1. Inner query executes  ->  produces a fixed result
2. Outer query runs using that result
Inner never runs again.
```

---

### 4.1 Scalar Subquery — Single Value in WHERE

Find employees who earn above the company average. Show how much above average.

```sql
SELECT
    full_name,
    salary,
    ROUND(salary - (SELECT AVG(salary) FROM employees), 2) AS above_avg_by
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
ORDER BY salary DESC;
```

The subquery runs once, returns 76625.00. Every outer row is compared against that fixed number.

---

### 4.2 List Subquery — IN / NOT IN

Find customers who have placed at least one delivered order.

```sql
SELECT customer_id, full_name, country
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id
    FROM   orders
    WHERE  status = 'delivered'
)
ORDER BY customer_id;
```

> **NULL trap:** `NOT IN` breaks silently when the subquery returns any NULL.
> Always add `WHERE col IS NOT NULL` inside the subquery, or use `NOT EXISTS` instead.

---

### 4.3 Derived Table — Subquery in FROM

Departments where average salary exceeds $70,000.

```sql
SELECT
    dept_summary.dept_name,
    dept_summary.avg_salary,
    dept_summary.headcount
FROM (
    SELECT
        d.dept_name,
        ROUND(AVG(e.salary), 2) AS avg_salary,
        COUNT(e.employee_id)    AS headcount
    FROM   employees e
    JOIN   departments d ON e.department_id = d.department_id
    GROUP  BY d.dept_name
) AS dept_summary          -- alias is MANDATORY on all derived tables
WHERE dept_summary.avg_salary > 70000
ORDER BY dept_summary.avg_salary DESC;
```

---

### 4.4 Subquery in HAVING

Departments whose total payroll exceeds the average department payroll.

```sql
SELECT d.dept_name, SUM(e.salary) AS total_payroll
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id
GROUP  BY d.dept_name
HAVING SUM(e.salary) > (
    SELECT AVG(dept_total)
    FROM (
        SELECT SUM(salary) AS dept_total
        FROM   employees
        GROUP  BY department_id
    ) AS payroll_by_dept
)
ORDER BY total_payroll DESC;
```

---

### 4.5 Subquery in SELECT Clause

Every order row plus the platform-wide average order value for comparison.

```sql
SELECT
    c.full_name,
    o.order_id,
    o.total_amount,
    (SELECT ROUND(AVG(total_amount), 2) FROM orders) AS platform_avg
FROM   orders o
JOIN   customers c ON o.customer_id = c.customer_id
ORDER  BY o.order_date;
```

---

### 4.6 Second-Level Nesting

Customers who spent more than double the minimum-spending customer.

```sql
SELECT c.full_name, SUM(o.total_amount) AS total_spend
FROM   customers c
JOIN   orders o ON o.customer_id = c.customer_id
GROUP  BY c.customer_id, c.full_name
HAVING SUM(o.total_amount) > (
    SELECT MIN(customer_total) * 2
    FROM (
        SELECT SUM(total_amount) AS customer_total
        FROM   orders
        GROUP  BY customer_id
    ) AS spend_per_customer
)
ORDER BY total_spend DESC;
```

---

## 5. Correlated Subqueries

### Definition

References at least one column from the outer query. Re-evaluated **once per outer row**.

```
For EACH outer row R:
  1. Take R's column values
  2. Inject into inner query
  3. Execute inner query  ->  result for R
  4. Use result in outer condition
  Move to next row.
```

---

### 5.1 Correlated WHERE

Find employees who earn more than the average salary in their own department.

```sql
SELECT e1.full_name, e1.salary, e1.job_title
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM   employees e2
    WHERE  e2.department_id = e1.department_id   -- correlation
);
```

The inner query uses `e1.department_id` from the outer row, so it re-executes per employee.

---

### 5.2 EXISTS — Check Presence

Customers who have at least one cancelled order.

```sql
SELECT c.full_name, c.country
FROM customers c
WHERE EXISTS (
    SELECT 1                               -- columns ignored; only existence matters
    FROM   orders o
    WHERE  o.customer_id = c.customer_id   -- correlation
      AND  o.status = 'cancelled'
);
```

`EXISTS` stops scanning as soon as one matching row is found (short-circuit).

---

### 5.3 NOT EXISTS — Safe Anti-Join

Products that have never been ordered.

```sql
SELECT p.product_name, p.category, p.unit_price
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM   order_items oi
    WHERE  oi.product_id = p.product_id    -- correlation
);
```

Always prefer `NOT EXISTS` over `NOT IN` when the inner column can be NULL.

---

### 5.4 Latest Record per Group

Each customer's most recent order.

```sql
SELECT c.full_name, o.order_id, o.order_date, o.total_amount, o.status
FROM   customers c
JOIN   orders o ON o.customer_id = c.customer_id
WHERE  o.order_date = (
    SELECT MAX(o2.order_date)
    FROM   orders o2
    WHERE  o2.customer_id = c.customer_id    -- correlation
)
ORDER BY c.customer_id;
```

---

### 5.5 Correlated Scalar in SELECT

Every employee's salary alongside their department's average.

```sql
SELECT
    e.full_name,
    e.salary,
    (SELECT ROUND(AVG(e2.salary), 2) FROM employees e2
     WHERE  e2.department_id = e.department_id) AS dept_avg,
    e.salary - (SELECT ROUND(AVG(e2.salary), 2) FROM employees e2
                WHERE  e2.department_id = e.department_id) AS diff
FROM employees e
ORDER BY e.department_id, e.salary DESC;
```

---

### 5.6 ALL and ANY

`> ALL` — must beat every value in the set.
`> ANY` — must beat at least one value.

```sql
-- Employees earning more than every Sales Rep
SELECT full_name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary FROM employees WHERE job_title = 'Sales Rep'
);
-- > ALL({62000,60000})  =  salary > 62000  (equivalent to > MAX)

-- Employees earning more than at least one top-level manager
SELECT full_name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary FROM employees WHERE manager_id IS NULL
);
-- > ANY  =  salary > MIN of the set
```

---

### 5.7 Relational Division — Double NOT EXISTS

Customers who have ordered every Electronics product.

```sql
SELECT c.full_name, c.email
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM products p
    WHERE  p.category = 'Electronics'
      AND  NOT EXISTS (
          SELECT 1
          FROM   order_items oi
          JOIN   orders o ON oi.order_id = o.order_id
          WHERE  o.customer_id = c.customer_id   -- outer correlated
            AND  oi.product_id = p.product_id    -- middle correlated
      )
);
-- "There is no Electronics product this customer has NOT ordered"
```

---

### 5.8 Correlated UPDATE and DELETE

```sql
-- UPDATE: recalculate each order's total from its actual items
UPDATE orders
SET total_amount = (
    SELECT COALESCE(SUM(quantity * unit_price), 0)
    FROM   order_items
    WHERE  order_items.order_id = orders.order_id    -- correlation
);

-- DELETE: remove reviews for out-of-stock products
DELETE FROM reviews
WHERE EXISTS (
    SELECT 1 FROM products
    WHERE  products.product_id = reviews.product_id  -- correlation
      AND  products.stock_qty = 0
);
```

---

## 6. JOIN Types — All Concepts

### 6.1 INNER JOIN

Returns only rows where the join condition is satisfied in **both** tables.
Unmatched rows from either side are dropped entirely.

**Syntax:**
```sql
SELECT a.col, b.col
FROM   table_a a
INNER JOIN table_b b ON a.key = b.key;

-- INNER is optional: JOIN by itself is INNER JOIN
SELECT a.col, b.col
FROM   table_a a
JOIN   table_b b ON a.key = b.key;
```

**Example:** Show every employee with their department name.

```sql
SELECT
    e.employee_id,
    e.full_name,
    e.job_title,
    e.salary,
    d.dept_name,
    d.location
FROM   employees e
INNER JOIN departments d ON e.department_id = d.department_id
ORDER  BY d.dept_name, e.salary DESC;
```

Result: 16 rows — one per employee (every employee has a valid department_id).

**Example:** Orders with customer and product detail.

```sql
SELECT
    o.order_id,
    c.full_name     AS customer,
    o.order_date,
    p.product_name,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total
FROM   orders o
JOIN   customers c    ON o.customer_id   = c.customer_id
JOIN   order_items oi ON o.order_id      = oi.order_id
JOIN   products p     ON oi.product_id   = p.product_id
ORDER  BY o.order_date, o.order_id;
```

This chains three JOINs. The engine processes them left to right.

---

### 6.2 LEFT JOIN (LEFT OUTER JOIN)

Returns **all rows from the left table** plus matched rows from the right table.
Where there is no match on the right, all right-side columns are NULL.

**Syntax:**
```sql
SELECT a.col, b.col
FROM   table_a a
LEFT JOIN table_b b ON a.key = b.key;
```

**Diagram:**
```
Left Table   Right Table      Result
----------   -----------      ------
   A              A           A, A   <- matched
   B              C           B, NULL <- B has no match in right
   D                          D, NULL <- D has no match in right
```

**Example:** Show all customers and their orders — include customers with no orders.

```sql
SELECT
    c.customer_id,
    c.full_name,
    c.country,
    o.order_id,
    o.order_date,
    o.status
FROM   customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
ORDER  BY c.customer_id, o.order_date;
```

Customers with no orders appear once with NULL values for all order columns.

**Example:** Show all products and their total sales (including products never sold).

```sql
SELECT
    p.product_name,
    p.category,
    COALESCE(SUM(oi.quantity), 0)                AS total_units_sold,
    COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS total_revenue
FROM   products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
GROUP  BY p.product_id, p.product_name, p.category
ORDER  BY total_revenue DESC;
```

`COALESCE(SUM(...), 0)` converts NULL (from products with no sales) to 0.

**Finding rows with no match — the NULL check pattern:**

```sql
-- Products that have NEVER been ordered
SELECT p.product_name, p.category
FROM   products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE  oi.product_id IS NULL;    -- NULL means no matching order_item row existed
```

This is the JOIN equivalent of `NOT EXISTS`. Both give the same result — pick whichever reads clearer.

---

### 6.3 RIGHT JOIN (RIGHT OUTER JOIN)

Returns **all rows from the right table** plus matched rows from the left.
Where there is no match on the left, all left-side columns are NULL.

**Syntax:**
```sql
SELECT a.col, b.col
FROM   table_a a
RIGHT JOIN table_b b ON a.key = b.key;
```

**Practical note:** Most engineers use `LEFT JOIN` for everything by flipping the table order.
`A RIGHT JOIN B` is identical to `B LEFT JOIN A`. Pick whichever reads more naturally.

**Example:** Show all departments — include departments with no employees.

```sql
-- RIGHT JOIN version
SELECT
    e.full_name,
    e.salary,
    d.dept_name,
    d.budget
FROM   employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
ORDER  BY d.dept_name, e.salary DESC;

-- Equivalent LEFT JOIN (identical result, more readable to most engineers):
SELECT
    e.full_name,
    e.salary,
    d.dept_name,
    d.budget
FROM   departments d
LEFT JOIN employees e ON d.department_id = e.department_id
ORDER  BY d.dept_name, e.salary DESC;
```

If a department has no employees, it appears with NULL for full_name and salary.

---

### 6.4 FULL OUTER JOIN

Returns **all rows from both tables**.
Where there is no match on either side, that side's columns are NULL.

**Syntax:**
```sql
SELECT a.col, b.col
FROM   table_a a
FULL OUTER JOIN table_b b ON a.key = b.key;
```

> **Engine support:** MySQL and SQLite do not support FULL OUTER JOIN directly.
> Emulate it with `LEFT JOIN UNION RIGHT JOIN` (see below).

**Example:** Show all customers and all orders — including customers without orders and orders without valid customers.

```sql
-- SQL Server / MariaDB (supports FULL OUTER JOIN directly)
SELECT
    c.full_name,
    c.country,
    o.order_id,
    o.order_date,
    o.status
FROM   customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id
ORDER  BY c.customer_id;
```

**MySQL / SQLite emulation using UNION:**
```sql
-- LEFT JOIN (all customers + matched orders)
SELECT c.full_name, c.country, o.order_id, o.order_date, o.status
FROM   customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id

UNION

-- RIGHT JOIN (all orders + matched customers), keeping only the right-only rows
SELECT c.full_name, c.country, o.order_id, o.order_date, o.status
FROM   customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id
WHERE  c.customer_id IS NULL;
```

**When to use FULL OUTER JOIN:**
- Data reconciliation: "what exists in table A but not B, and vice versa?"
- Merging two similar datasets to find gaps.
- Audit queries where you need every record from both sides.

---

### 6.5 CROSS JOIN

Returns the **Cartesian product** — every row in the left table paired with every row in the right table.
No join condition is needed (or used).

**Syntax:**
```sql
SELECT a.col, b.col
FROM   table_a a
CROSS JOIN table_b b;

-- Old implicit syntax (same result, avoid in new code):
SELECT a.col, b.col FROM table_a a, table_b b;
```

**Row count:** `rows_in_A × rows_in_B`

**Example:** Generate every possible product-category combination (for a coverage matrix).

```sql
-- What if we want to pair every customer with every product for a recommendation grid?
SELECT
    c.full_name   AS customer,
    p.product_name,
    p.category
FROM   customers c
CROSS JOIN products p
ORDER  BY c.customer_id, p.product_id;
-- 10 customers × 15 products = 150 rows
```

**Practical backend use case — generate a date or slot grid:**

```sql
-- All departments × all job titles that exist, for a staffing matrix
SELECT
    d.dept_name,
    job_titles.job_title
FROM departments d
CROSS JOIN (
    SELECT DISTINCT job_title FROM employees
) AS job_titles
ORDER BY d.dept_name, job_titles.job_title;
```

> Use CROSS JOIN intentionally, never accidentally. An accidental `FROM a, b` without a WHERE clause is a surprise Cartesian product waiting to explode on large tables.

---

### 6.6 SELF JOIN

A table joined **to itself** using two different aliases.
Used for hierarchical or comparison data within the same table.

**Syntax:**
```sql
SELECT a.col AS col_from_a, b.col AS col_from_b
FROM   same_table a
JOIN   same_table b ON a.related_key = b.primary_key;
```

**Example:** Show every employee alongside their manager's name.

```sql
SELECT
    e.full_name      AS employee,
    e.job_title,
    e.salary,
    mgr.full_name    AS manager,
    mgr.job_title    AS manager_title
FROM   employees e
LEFT JOIN employees mgr ON e.manager_id = mgr.employee_id
ORDER  BY mgr.full_name NULLS LAST, e.salary DESC;
```

LEFT JOIN is used so top-level employees (no manager) still appear — their manager columns are NULL.

**Example:** Find all pairs of employees in the same department earning within $10,000 of each other.

```sql
SELECT
    e1.full_name AS employee_1,
    e1.salary    AS salary_1,
    e2.full_name AS employee_2,
    e2.salary    AS salary_2,
    ABS(e1.salary - e2.salary) AS salary_gap
FROM   employees e1
JOIN   employees e2 ON  e1.department_id = e2.department_id
                    AND e1.employee_id < e2.employee_id   -- avoid duplicates and self-pairs
WHERE  ABS(e1.salary - e2.salary) <= 10000
ORDER  BY e1.department_id, salary_gap;
```

`e1.employee_id < e2.employee_id` ensures each pair appears only once and eliminates the row where an employee is matched against themselves.

**Example:** Multi-level hierarchy — employees, their manager, and their manager's manager.

```sql
SELECT
    e.full_name    AS employee,
    mgr.full_name  AS manager,
    gm.full_name   AS grand_manager
FROM      employees e
LEFT JOIN employees mgr ON e.manager_id   = mgr.employee_id
LEFT JOIN employees gm  ON mgr.manager_id = gm.employee_id
ORDER  BY gm.full_name NULLS LAST, mgr.full_name NULLS LAST;
```

---

### 6.7 Multiple JOINs — Chaining

You can chain as many JOINs as needed. The engine processes them left to right.
Each JOIN adds another table to the working result set.

**Example:** Full order report — customer, order, product, category, review rating.

```sql
SELECT
    c.full_name       AS customer,
    c.country,
    o.order_id,
    o.order_date,
    o.status,
    p.product_name,
    p.category,
    oi.quantity,
    oi.quantity * oi.unit_price   AS line_total,
    r.rating,
    r.review_text
FROM   orders o
JOIN   customers    c  ON o.customer_id   = c.customer_id
JOIN   order_items  oi ON o.order_id      = oi.order_id
JOIN   products     p  ON oi.product_id   = p.product_id
LEFT JOIN reviews   r  ON r.product_id    = p.product_id
                      AND r.customer_id   = c.customer_id   -- match review to THIS customer
ORDER  BY o.order_date, o.order_id, p.product_name;
```

> **Mix JOIN types freely.** The LEFT JOIN on reviews ensures order lines without a review still appear.
> The two-condition ON clause ties the review to the exact customer-product combination.

**Example:** Department report with employee count, payroll, and budget usage.

```sql
SELECT
    d.dept_name,
    d.budget,
    d.location,
    COUNT(e.employee_id)        AS headcount,
    COALESCE(SUM(e.salary), 0)  AS total_payroll,
    d.budget - COALESCE(SUM(e.salary), 0) AS remaining_budget
FROM   departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP  BY d.department_id, d.dept_name, d.budget, d.location
ORDER  BY remaining_budget;
```

---

### 6.8 JOIN with Subqueries — The Bridge Pattern

JOINs and subqueries are not competing tools. They combine powerfully.

**Pattern A — JOIN onto a derived table (pre-aggregate then join):**

```sql
-- Show each customer's total spend alongside their details
-- MUCH faster than a correlated subquery when customer count is large
SELECT
    c.full_name,
    c.country,
    COALESCE(spend.total_spend, 0) AS total_spend,
    COALESCE(spend.order_count, 0) AS order_count
FROM   customers c
LEFT JOIN (
    SELECT
        customer_id,
        SUM(total_amount) AS total_spend,
        COUNT(*)          AS order_count
    FROM   orders
    GROUP  BY customer_id
) AS spend ON c.customer_id = spend.customer_id
ORDER  BY total_spend DESC;
```

The subquery aggregates once. The JOIN distributes those aggregates to each customer row.
Compare this to a correlated subquery — that would run one `SUM()` per customer row.

**Pattern B — Filter with subquery, enrich with JOIN:**

```sql
-- Start with a subquery to identify high-value customers,
-- then JOIN to get their full profile
SELECT
    c.full_name,
    c.country,
    c.email,
    vip.total_spend
FROM   customers c
JOIN (
    SELECT customer_id, SUM(total_amount) AS total_spend
    FROM   orders
    GROUP  BY customer_id
    HAVING SUM(total_amount) > 300          -- filter in the subquery
) AS vip ON c.customer_id = vip.customer_id
ORDER  BY vip.total_spend DESC;
```

**Pattern C — JOIN to validate, subquery to compute:**

```sql
-- For each employee, show their salary and the top salary in their department
SELECT
    e.full_name,
    e.salary,
    d.dept_name,
    dept_top.max_salary,
    e.salary - dept_top.max_salary AS gap_to_top
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id
JOIN (
    SELECT department_id, MAX(salary) AS max_salary
    FROM   employees
    GROUP  BY department_id
) AS dept_top ON e.department_id = dept_top.department_id
ORDER  BY d.dept_name, e.salary DESC;
```

---

### 6.9 JOIN vs Subquery — When to Use Which

| Scenario | Recommended Approach | Why |
|---|---|---|
| "Show me columns from two tables together" | JOIN | Natural multi-table output |
| "Does a related row exist?" | `EXISTS` subquery | Short-circuits on first match |
| "Does no related row exist?" | `NOT EXISTS` subquery | NULL-safe anti-join |
| "Filter on an aggregated value per group" | JOIN + derived table or CTE | Aggregate once, not per row |
| "Add a single scalar value to every row" | Non-correlated subquery in SELECT | Cleaner than a JOIN for one number |
| "Per-row comparison against its own group" | Correlated subquery OR JOIN + derived table | JOIN version is faster at scale |
| "Running total / rank" | Window function (if available) | Built for sequential computation |
| "Cartesian product intentionally" | CROSS JOIN | Only correct tool for this |
| "Hierarchy traversal (parent-child)" | SELF JOIN | Only way without recursive CTE |

---

## 7. Subquery Placement Cheatsheet

```
SQL Statement
|
+-- SELECT clause    ->  scalar subquery only (1 row, 1 column)
|
+-- FROM clause      ->  any subquery as derived table (MUST have alias)
|
+-- WHERE clause     ->  scalar (= > <), list (IN / NOT IN),
|                        or existence (EXISTS / NOT EXISTS / ALL / ANY)
|
+-- HAVING clause    ->  same rules as WHERE, after GROUP BY
|
+-- UPDATE SET       ->  scalar correlated subquery
|
+-- DELETE WHERE     ->  IN or EXISTS subquery
|
+-- INSERT source    ->  INSERT INTO t SELECT ... (full subquery as data source)
```

---

## 8. JOIN Cheatsheet

```sql
-- INNER JOIN: only matched rows
FROM a JOIN b ON a.key = b.key

-- LEFT JOIN: all of a, matched of b (NULLs where no b match)
FROM a LEFT JOIN b ON a.key = b.key

-- RIGHT JOIN: all of b, matched of a (NULLs where no a match)
FROM a RIGHT JOIN b ON a.key = b.key

-- FULL OUTER JOIN: all of both (NULLs on either side where no match)
FROM a FULL OUTER JOIN b ON a.key = b.key

-- CROSS JOIN: every a row x every b row (no condition)
FROM a CROSS JOIN b

-- SELF JOIN: table compared/linked to itself
FROM t alias1 JOIN t alias2 ON alias1.fk = alias2.pk

-- Multiple conditions in ON
FROM a JOIN b ON a.key = b.key AND a.type = b.type

-- JOIN + filter in ON vs WHERE (affects LEFT JOIN behaviour!)
-- This filters BEFORE the join (keeps all left rows):
FROM a LEFT JOIN b ON a.key = b.key AND b.status = 'active'

-- This filters AFTER the join (removes left rows with no active match):
FROM a LEFT JOIN b ON a.key = b.key
WHERE b.status = 'active'

-- Find rows in left with NO match in right (anti-join)
FROM a LEFT JOIN b ON a.key = b.key
WHERE b.key IS NULL
```

---

## 9. Practice Questions

---

### Level 1 — Subquery Basics (Non-Correlated)

#### Q1. Above-Average Salary
Find every employee whose salary is above the overall company average.
Show name, salary, and how many dollars above average they are.
Use only a subquery — no JOINs.

#### Q2. Most Expensive Product per Category
Show the product name, category, and unit price for the most expensive product in each category.

#### Q3. Derived Table — Departments Above $75k Average
Using a derived table in FROM, list departments where average employee salary exceeds $75,000.
Show: department name, average salary, headcount.

#### Q4. Customers Who Have Ordered
Find all customers who appear at least once in the orders table. Use `IN`.

#### Q5. Products Never Reviewed — Two Ways
Write the query using `NOT IN`. Then rewrite with `NOT EXISTS`. Explain the difference.

---

### Level 2 — Mixed Subqueries (Intermediate)

#### Q6. Employees Managed by a High Earner
Find employees whose direct manager earns above $100,000.
Show: employee name, salary, manager name, manager salary.

#### Q7. Above-Average Spend per Customer
Find customers whose total spend is above the average total spend per customer.
Show: customer name, country, total spend.

#### Q8. Second Highest Salary
Find employees with the second highest salary. No LIMIT/OFFSET — use only subqueries.

#### Q9. Departments Where No Employee Earns Below $60,000
Use `NOT EXISTS` with a correlated subquery.

#### Q10. Employees Earning More Than All Sales Reps
Use `> ALL` with a subquery.

---

### Level 3 — JOIN Basics (Beginner-Intermediate)

#### Q11. INNER JOIN — Employee Full Profile
Show each employee's full name, job title, salary, department name, and department location.

#### Q12. LEFT JOIN — All Products with Sales Summary
Show all 15 products. For each, show total units sold and total revenue.
Products with no sales should show 0 for both values.

#### Q13. LEFT JOIN — Customers with No Orders
Find customers who have never placed any order. Use a LEFT JOIN with a NULL check.
Then rewrite using NOT EXISTS and compare.

#### Q14. SELF JOIN — Employee + Manager
Show every employee paired with their manager's name and salary.
Employees with no manager should still appear.

#### Q15. Multiple JOIN — Full Order Line Report
For every order line, show: customer name, order date, product name, category, quantity, unit price, line total.
Use three JOINs.

---

### Level 4 — Advanced JOIN and Combined Patterns

#### Q16. RIGHT JOIN — Departments with No Employees
Find departments that have no employees assigned to them. Use a RIGHT JOIN.

#### Q17. CROSS JOIN — Product × Category Coverage Matrix
Generate a grid of every department × every unique job title that exists in the company.
Show which department-title combinations exist and which do not.

#### Q18. SELF JOIN — Salary Peers
Find all pairs of employees in the same department whose salaries are within $5,000 of each other.
Each pair should appear only once.

#### Q19. JOIN + Derived Table (Bridge Pattern)
Show each customer's name, country, total spend, and order count.
Use a LEFT JOIN onto a derived table (do NOT use a correlated subquery).

#### Q20. Running Cumulative Spend
For each order, show customer name, order date, this order's total, and cumulative spend to date.
Use a correlated subquery in SELECT.

#### Q21. Multi-level Hierarchy
Show each employee, their direct manager, and their manager's manager (grand-manager).
Use two LEFT SELF JOINs.

#### Q22. JOIN with Subquery Filter — VIP Customers
Identify customers who have spent over $300 total and show their full profile.
Use a subquery to identify them, then JOIN for the profile.

#### Q23. Most Recent Order per Customer (JOIN + Correlated)
Show each customer's most recent order details.
Use a JOIN plus a correlated subquery — not a window function.

#### Q24. Department Budget Status per Employee
For every employee show: name, salary, dept name, dept total payroll, dept budget, and budget status (OVER/UNDER).
Use a non-correlated derived table + JOIN + CASE.

#### Q25. FULL OUTER JOIN — Customer-Order Gap Report
Show all customers and all orders side by side.
For customers without orders, show NULL on the order side.
For orders without a valid customer (hypothetically), show NULL on the customer side.
*(Use UNION-based emulation for MySQL/SQLite.)*

---

## 10. Solutions

---

### Solution Q1 — Above-Average Salary

```sql
SELECT
    full_name,
    salary,
    ROUND(salary - (SELECT AVG(salary) FROM employees), 2) AS above_avg_by
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
ORDER BY salary DESC;
```

**Explanation:** Subquery runs once, returns 76625.00. Outer compares every row against that constant.

---

### Solution Q2 — Most Expensive Product per Category

```sql
SELECT p1.product_name, p1.category, p1.unit_price
FROM products p1
WHERE p1.unit_price = (
    SELECT MAX(p2.unit_price)
    FROM   products p2
    WHERE  p2.category = p1.category    -- correlated by category
)
ORDER BY p1.category;
```

**Note:** This is a correlated subquery — `p1.category` comes from the outer row.

---

### Solution Q3 — Derived Table Above $75k

```sql
SELECT dept_stats.dept_name, dept_stats.avg_salary, dept_stats.headcount
FROM (
    SELECT
        d.dept_name,
        ROUND(AVG(e.salary), 2) AS avg_salary,
        COUNT(e.employee_id)    AS headcount
    FROM   employees e
    JOIN   departments d ON e.department_id = d.department_id
    GROUP  BY d.dept_name
) AS dept_stats
WHERE dept_stats.avg_salary > 75000
ORDER BY dept_stats.avg_salary DESC;
```

---

### Solution Q4 — Customers Who Have Ordered

```sql
SELECT customer_id, full_name, country
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id FROM orders
)
ORDER BY customer_id;
```

---

### Solution Q5 — Products Never Reviewed

```sql
-- NOT IN version (guard with IS NOT NULL)
SELECT product_name, category, unit_price
FROM products
WHERE product_id NOT IN (
    SELECT DISTINCT product_id FROM reviews WHERE product_id IS NOT NULL
);

-- NOT EXISTS version (recommended)
SELECT p.product_name, p.category, p.unit_price
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM reviews r WHERE r.product_id = p.product_id
);
```

**Why NOT EXISTS is better:** NULL-safe, short-circuits on first review found, intent is unambiguous.

---

### Solution Q6 — Employees Managed by a High Earner

```sql
-- IN version
SELECT e.full_name AS employee_name, e.salary, e.job_title
FROM employees e
WHERE e.manager_id IN (
    SELECT employee_id FROM employees WHERE salary > 100000
)
ORDER BY e.salary DESC;

-- EXISTS version (also shows manager detail via JOIN)
SELECT
    e.full_name   AS employee_name,
    e.salary      AS employee_salary,
    mgr.full_name AS manager_name,
    mgr.salary    AS manager_salary
FROM   employees e
JOIN   employees mgr ON e.manager_id = mgr.employee_id
WHERE EXISTS (
    SELECT 1 FROM employees m
    WHERE  m.employee_id = e.manager_id AND m.salary > 100000
)
ORDER BY mgr.salary DESC;
```

---

### Solution Q7 — Above-Average Spend per Customer

```sql
SELECT c.full_name, c.country, cust_spend.total_spend
FROM customers c
JOIN (
    SELECT customer_id, SUM(total_amount) AS total_spend
    FROM   orders
    GROUP  BY customer_id
) AS cust_spend ON c.customer_id = cust_spend.customer_id
WHERE cust_spend.total_spend > (
    SELECT AVG(customer_total)
    FROM (
        SELECT SUM(total_amount) AS customer_total
        FROM   orders
        GROUP  BY customer_id
    ) AS all_spend
)
ORDER BY cust_spend.total_spend DESC;
```

---

### Solution Q8 — Second Highest Salary

```sql
SELECT DISTINCT full_name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM   employees
    WHERE  salary < (SELECT MAX(salary) FROM employees)
)
ORDER BY full_name;
```

**Trace:** Innermost -> 120000. Middle -> MAX below 120000 = 95000. Outer -> employees with salary = 95000.

---

### Solution Q9 — No Employee Below $60,000

```sql
SELECT d.dept_name, COUNT(e.employee_id) AS headcount, MIN(e.salary) AS lowest_salary
FROM departments d
JOIN employees e ON e.department_id = d.department_id
GROUP BY d.department_id, d.dept_name
HAVING NOT EXISTS (
    SELECT 1 FROM employees e2
    WHERE  e2.department_id = d.department_id AND e2.salary < 60000
)
ORDER BY lowest_salary DESC;
```

---

### Solution Q10 — Earn More Than All Sales Reps

```sql
SELECT full_name, salary, job_title
FROM employees
WHERE salary > ALL (
    SELECT salary FROM employees WHERE job_title = 'Sales Rep'
)
ORDER BY salary DESC;
-- Equivalent: WHERE salary > (SELECT MAX(salary) FROM employees WHERE job_title = 'Sales Rep')
```

---

### Solution Q11 — INNER JOIN Employee Full Profile

```sql
SELECT
    e.employee_id,
    e.full_name,
    e.job_title,
    e.salary,
    d.dept_name,
    d.location
FROM   employees e
INNER JOIN departments d ON e.department_id = d.department_id
ORDER  BY d.dept_name, e.salary DESC;
```

**Why INNER JOIN here?** Every employee has a valid `department_id`, so INNER and LEFT give the same result. INNER is semantically correct since we only want employees that have a department.

---

### Solution Q12 — LEFT JOIN All Products with Sales

```sql
SELECT
    p.product_name,
    p.category,
    p.unit_price,
    COALESCE(SUM(oi.quantity), 0)                 AS total_units_sold,
    COALESCE(SUM(oi.quantity * oi.unit_price), 0)  AS total_revenue
FROM   products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
GROUP  BY p.product_id, p.product_name, p.category, p.unit_price
ORDER  BY total_revenue DESC;
```

`LEFT JOIN` keeps all 15 products. Products with no order_items get NULL from `SUM()`, which `COALESCE` converts to 0.

---

### Solution Q13 — Customers with No Orders

```sql
-- LEFT JOIN + NULL check
SELECT c.customer_id, c.full_name, c.country
FROM   customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE  o.order_id IS NULL
ORDER  BY c.customer_id;

-- NOT EXISTS equivalent
SELECT c.customer_id, c.full_name, c.country
FROM   customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
)
ORDER BY c.customer_id;
```

**Both produce the same result.** The LEFT JOIN + IS NULL pattern is common when you are already doing other JOINs and want to filter in the same query. `NOT EXISTS` is clearer in intent as a standalone check.

---

### Solution Q14 — SELF JOIN Employee + Manager

```sql
SELECT
    e.full_name     AS employee,
    e.job_title     AS employee_title,
    e.salary        AS employee_salary,
    mgr.full_name   AS manager,
    mgr.salary      AS manager_salary
FROM   employees e
LEFT JOIN employees mgr ON e.manager_id = mgr.employee_id
ORDER  BY mgr.full_name, e.salary DESC;
```

`LEFT JOIN` ensures employees with no manager (Alice, Bob, Karen, Mia, Olivia) still appear — their manager columns show NULL.

---

### Solution Q15 — Full Order Line Report

```sql
SELECT
    c.full_name                         AS customer,
    o.order_date,
    o.status,
    p.product_name,
    p.category,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price          AS line_total
FROM   orders o
JOIN   customers   c  ON o.customer_id  = c.customer_id
JOIN   order_items oi ON o.order_id     = oi.order_id
JOIN   products    p  ON oi.product_id  = p.product_id
ORDER  BY o.order_date, o.order_id, p.product_name;
```

Three INNER JOINs. Every order_item must have a valid order, customer, and product — so INNER is correct.

---

### Solution Q16 — RIGHT JOIN Departments with No Employees

```sql
-- RIGHT JOIN to find departments with no employees
SELECT
    d.department_id,
    d.dept_name,
    d.budget,
    e.full_name AS employee
FROM   employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
WHERE  e.employee_id IS NULL;      -- NULL means no employee matched
```

With our seed data every department has at least one employee, so this returns 0 rows.
Add a department to test:
```sql
INSERT INTO departments VALUES (6, 'Legal', 100000.00, 'New York');
-- Now rerun the query -> Legal appears with NULL for employee
```

**Equivalent LEFT JOIN:**
```sql
SELECT d.department_id, d.dept_name
FROM   departments d
LEFT JOIN employees e ON d.department_id = e.department_id
WHERE  e.employee_id IS NULL;
```

---

### Solution Q17 — CROSS JOIN Department × Job Title Matrix

```sql
SELECT
    d.dept_name,
    jt.job_title,
    CASE WHEN e.employee_id IS NOT NULL THEN 'EXISTS' ELSE '---' END AS has_employee
FROM departments d
CROSS JOIN (
    SELECT DISTINCT job_title FROM employees
) AS jt
LEFT JOIN employees e ON e.department_id = d.department_id
                     AND e.job_title     = jt.job_title
ORDER BY d.dept_name, jt.job_title;
```

**How it works:**
1. `CROSS JOIN` produces every department × job_title pair (5 × 13 = 65 rows).
2. `LEFT JOIN` on employees checks if that exact combination exists.
3. `CASE` labels it accordingly.

---

### Solution Q18 — SELF JOIN Salary Peers

```sql
SELECT
    e1.full_name                AS employee_1,
    e1.salary                   AS salary_1,
    e2.full_name                AS employee_2,
    e2.salary                   AS salary_2,
    ABS(e1.salary - e2.salary)  AS salary_gap,
    d.dept_name
FROM   employees e1
JOIN   employees e2 ON  e1.department_id = e2.department_id
                    AND e1.employee_id   < e2.employee_id      -- each pair once, no self-pairs
JOIN   departments d ON e1.department_id = d.department_id
WHERE  ABS(e1.salary - e2.salary) <= 5000
ORDER  BY d.dept_name, salary_gap;
```

`e1.employee_id < e2.employee_id` is the key trick:
- Eliminates (Alice, Alice) — the self-pair.
- Eliminates (Bob, Alice) when (Alice, Bob) already exists — the duplicate pair.

---

### Solution Q19 — JOIN + Derived Table Customer Spend

```sql
SELECT
    c.full_name,
    c.country,
    COALESCE(spend.total_spend, 0) AS total_spend,
    COALESCE(spend.order_count, 0) AS order_count
FROM   customers c
LEFT JOIN (
    SELECT
        customer_id,
        SUM(total_amount) AS total_spend,
        COUNT(*)          AS order_count
    FROM   orders
    GROUP  BY customer_id
) AS spend ON c.customer_id = spend.customer_id
ORDER  BY total_spend DESC;
```

**Why this beats a correlated subquery for large data:** The subquery aggregates all orders once in a single pass. The JOIN distributes results. A correlated subquery would run `SUM()` and `COUNT()` once per customer — N separate scans.

---

### Solution Q20 — Running Cumulative Spend

```sql
SELECT
    c.full_name,
    o.order_date,
    o.order_id,
    o.total_amount AS this_order,
    (
        SELECT SUM(o2.total_amount)
        FROM   orders o2
        WHERE  o2.customer_id = o.customer_id   -- same customer
          AND  o2.order_id   <= o.order_id       -- up to this order
    ) AS cumulative_spend
FROM   orders o
JOIN   customers c ON o.customer_id = c.customer_id
ORDER  BY c.customer_id, o.order_date, o.order_id;
```

**How it works:** For each order row `o`, the correlated subquery sums all orders by the same customer with `order_id <= current`. As `order_id` increases, the cumulative total grows.

---

### Solution Q21 — Three-Level Hierarchy

```sql
SELECT
    e.full_name    AS employee,
    e.job_title,
    mgr.full_name  AS manager,
    gm.full_name   AS grand_manager
FROM      employees e
LEFT JOIN employees mgr ON e.manager_id   = mgr.employee_id
LEFT JOIN employees gm  ON mgr.manager_id = gm.employee_id
ORDER  BY gm.full_name, mgr.full_name, e.full_name;
```

Two independent LEFT SELF JOINs on the same table, each with its own alias.
`LEFT JOIN` at each level keeps employees whose manager or grand-manager is NULL.

---

### Solution Q22 — VIP Customers via JOIN + Subquery

```sql
SELECT
    c.full_name,
    c.country,
    c.email,
    vip.total_spend
FROM   customers c
JOIN (
    SELECT customer_id, SUM(total_amount) AS total_spend
    FROM   orders
    GROUP  BY customer_id
    HAVING SUM(total_amount) > 300
) AS vip ON c.customer_id = vip.customer_id
ORDER  BY vip.total_spend DESC;
```

**Why INNER JOIN here?** We want only the qualifying customers. INNER JOIN on the derived table automatically excludes customers not in the VIP subquery.

---

### Solution Q23 — Most Recent Order per Customer

```sql
SELECT
    c.full_name,
    o.order_id,
    o.order_date,
    o.total_amount,
    o.status
FROM   customers c
JOIN   orders o ON o.customer_id = c.customer_id
WHERE  o.order_date = (
    SELECT MAX(o2.order_date)
    FROM   orders o2
    WHERE  o2.customer_id = c.customer_id    -- correlated
)
ORDER  BY c.customer_id;
```

The JOIN brings in order data. The correlated subquery in WHERE limits to just the most recent row per customer.

---

### Solution Q24 — Department Budget Status per Employee

```sql
SELECT
    e.full_name,
    e.salary,
    d.dept_name,
    dept_payroll.total_payroll,
    d.budget,
    CASE
        WHEN dept_payroll.total_payroll > d.budget THEN 'OVER BUDGET'
        ELSE 'UNDER BUDGET'
    END AS budget_status
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id
JOIN (
    -- non-correlated derived table: aggregates all departments once
    SELECT department_id, SUM(salary) AS total_payroll
    FROM   employees
    GROUP  BY department_id
) AS dept_payroll ON dept_payroll.department_id = d.department_id
ORDER  BY d.dept_name, e.salary DESC;
```

---

### Solution Q25 — FULL OUTER JOIN Customer-Order Gap Report

```sql
-- SQL Server / MariaDB (native FULL OUTER JOIN)
SELECT
    c.full_name,
    c.country,
    o.order_id,
    o.order_date,
    o.status
FROM   customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id
ORDER  BY c.customer_id, o.order_date;

-- MySQL / SQLite emulation (UNION of LEFT + RIGHT-only rows)
SELECT c.full_name, c.country, o.order_id, o.order_date, o.status
FROM   customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id

UNION ALL

SELECT c.full_name, c.country, o.order_id, o.order_date, o.status
FROM   customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id
WHERE  c.customer_id IS NULL;     -- only rows where right has no left match
```

**`UNION ALL` vs `UNION`:** Use `UNION ALL` here because we already filtered `WHERE c.customer_id IS NULL` in the second half — there are no duplicates to remove, and `UNION ALL` is faster (no dedup pass).

---

## 11. Performance and Pitfalls

### 11.1 Correlated Subquery vs JOIN + Derived Table

```sql
-- SLOW: correlated subquery runs once per employee row
SELECT full_name FROM employees e
WHERE salary > (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id);

-- FAST: aggregate once, join
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM   employees GROUP BY department_id
)
SELECT e.full_name FROM employees e
JOIN dept_avg d ON e.department_id = d.department_id
WHERE e.salary > d.avg_sal;
```

Rule: if the correlated subquery does aggregation and the outer table is large, convert to JOIN.

---

### 11.2 The NULL Trap with NOT IN

```sql
-- BROKEN silently if any order has customer_id = NULL
WHERE customer_id NOT IN (SELECT customer_id FROM orders)

-- Always safe
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = customers.customer_id)
```

---

### 11.3 ON vs WHERE with LEFT JOIN

```sql
-- Filter in ON: keeps all left rows (filters only on the right side before joining)
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id AND o.status = 'delivered'
-- Result: all customers; those with only non-delivered orders get NULL for order columns

-- Filter in WHERE: eliminates left rows with no match after the join
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.status = 'delivered'
-- Result: only customers who have at least one delivered order (same as INNER JOIN in effect)
```

This is one of the most common LEFT JOIN bugs. When you want "all customers, with delivered orders if they exist," put the filter in ON. When you want "only customers who have a delivered order," put it in WHERE.

---

### 11.4 Accidental Cartesian Product

```sql
-- WRONG: forgot the join condition -> 16 × 5 = 80 rows instead of 16
SELECT e.full_name, d.dept_name
FROM employees e, departments d;     -- old implicit JOIN syntax with no WHERE

-- CORRECT
SELECT e.full_name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

Always use explicit `JOIN ... ON` syntax. Never rely on implicit comma joins.

---

### 11.5 Derived Table Alias is Mandatory

```sql
-- ERROR on every database
SELECT * FROM (SELECT dept_name FROM departments);

-- CORRECT
SELECT * FROM (SELECT dept_name FROM departments) AS d;
```

---

### 11.6 Scalar Subquery Must Return One Row

```sql
-- RUNTIME ERROR: returns 7 rows (all engineers)
WHERE salary = (SELECT salary FROM employees WHERE department_id = 1)

-- Fix: use IN or an aggregate
WHERE salary IN (SELECT salary FROM employees WHERE department_id = 1)
WHERE salary =  (SELECT MAX(salary) FROM employees WHERE department_id = 1)
```

---

### 11.7 FULL OUTER JOIN on MySQL/SQLite

MySQL and SQLite do not implement FULL OUTER JOIN. The emulation pattern is:
```sql
SELECT ... FROM a LEFT JOIN b ON ...
UNION ALL
SELECT ... FROM a RIGHT JOIN b ON ...
WHERE a.key IS NULL
```

The second half selects only the right-only rows (those with no left match), preventing duplicates.

---

### 11.8 Read the Query Plan

```sql
-- Check how your query executes
EXPLAIN SELECT ...;

-- Look for:
-- MySQL: "DEPENDENT SUBQUERY" = correlated subquery, runs N times
-- MySQL: "SUBQUERY" = runs once (good)
-- MySQL: "ALL" in type column = full table scan (add an index)
-- SQL Server: "Nested Loops" on inner side = correlated, may be slow at scale
-- SQL Server: "Hash Match" / "Merge Join" = optimizer already joined (efficient)
```

---

## 12. Quick Reference

### Subquery Syntax Templates

```sql
-- Non-correlated scalar
WHERE col = (SELECT MAX(col) FROM tbl)

-- Non-correlated list
WHERE col IN (SELECT col FROM tbl WHERE condition)
WHERE col NOT IN (SELECT col FROM tbl WHERE col IS NOT NULL)

-- Derived table
FROM (SELECT col, AGG(col2) AS alias FROM tbl GROUP BY col) AS vt

-- Correlated WHERE
WHERE col > (SELECT AVG(col) FROM tbl WHERE tbl.fk = outer.pk)

-- EXISTS / NOT EXISTS
WHERE EXISTS     (SELECT 1 FROM tbl WHERE tbl.fk = outer.pk AND cond)
WHERE NOT EXISTS (SELECT 1 FROM tbl WHERE tbl.fk = outer.pk)

-- ALL / ANY
WHERE col > ALL (SELECT col FROM tbl WHERE cond)
WHERE col > ANY (SELECT col FROM tbl WHERE cond)

-- Correlated SELECT clause
SELECT (SELECT MAX(col) FROM tbl WHERE tbl.fk = outer.pk) AS computed

-- UPDATE / DELETE
UPDATE t SET col = (SELECT AGG(col) FROM t2 WHERE t2.fk = t.pk)
DELETE FROM t WHERE EXISTS (SELECT 1 FROM t2 WHERE t2.fk = t.pk AND cond)
```

---

### JOIN Syntax Templates

```sql
-- INNER JOIN
FROM a JOIN b ON a.key = b.key

-- LEFT JOIN
FROM a LEFT JOIN b ON a.key = b.key

-- RIGHT JOIN
FROM a RIGHT JOIN b ON a.key = b.key

-- FULL OUTER JOIN
FROM a FULL OUTER JOIN b ON a.key = b.key
-- MySQL/SQLite emulation:
-- SELECT ... FROM a LEFT JOIN b ON ... UNION ALL SELECT ... FROM a RIGHT JOIN b ON ... WHERE a.key IS NULL

-- CROSS JOIN
FROM a CROSS JOIN b

-- SELF JOIN
FROM tbl t1 JOIN tbl t2 ON t1.fk = t2.pk

-- Multiple JOINs
FROM a JOIN b ON a.k = b.k  JOIN c ON b.k = c.k  LEFT JOIN d ON c.k = d.k

-- Anti-join via LEFT JOIN
FROM a LEFT JOIN b ON a.key = b.key WHERE b.key IS NULL

-- JOIN + derived table
FROM a JOIN (SELECT fk, AGG(col) AS val FROM b GROUP BY fk) AS agg ON a.pk = agg.fk
```

---

### Decision Flowchart

```
WHAT DO I NEED?
|
+-- Combine columns from two or more tables side by side?
|         -> JOIN (choose type based on which side needs all rows)
|
+-- Does a related row exist?
|         -> WHERE EXISTS (correlated subquery)
|
+-- Does NO related row exist?
|         -> WHERE NOT EXISTS  (never NOT IN on nullable columns)
|         -> OR LEFT JOIN ... WHERE b.key IS NULL
|
+-- Compare row to its group aggregate?
|         -> JOIN + derived table/CTE  (fast)
|         -> OR correlated subquery    (simple, slower on large tables)
|
+-- Add one global computed value to every row?
|         -> Non-correlated scalar subquery in SELECT
|
+-- Every row in A matched with every row in B?
|         -> CROSS JOIN
|
+-- Parent-child or peer comparison within the same table?
|         -> SELF JOIN
|
+-- Find gaps — rows in A with no match in B?
|         -> LEFT JOIN ... WHERE b.key IS NULL
|         -> OR NOT EXISTS
|
+-- All rows from both sides including non-matches?
|         -> FULL OUTER JOIN
|            (MySQL/SQLite: LEFT JOIN UNION ALL RIGHT JOIN WHERE a.key IS NULL)
```

---

### Common Mistakes

| Mistake | Effect | Fix |
|---|---|---|
| `NOT IN` on nullable column | Returns 0 rows silently | Use `NOT EXISTS` |
| Filter in WHERE after LEFT JOIN | Turns LEFT into INNER | Move filter to ON clause |
| No alias on derived table | Syntax error | Add `AS alias` |
| Missing join condition in FROM | Cartesian product explosion | Always use explicit `JOIN ... ON` |
| Correlated aggregate on large table | N full scans | JOIN + derived table/CTE |
| Scalar subquery returns 2+ rows | Runtime error | Use `IN` or an aggregate |
| Nesting 4+ subquery levels | Unreadable | Refactor with CTEs |
| `FULL OUTER JOIN` in MySQL/SQLite | Unsupported error | Use LEFT UNION ALL RIGHT emulation |

---

> **Final note for backend engineers:**
> JOINs and subqueries are complementary, not competing tools.
> JOINs excel at combining relational data into a flat result set.
> Subqueries excel at filtering based on derived conditions or existence checks.
> The bridge pattern — joining onto a derived table — is the most common performance upgrade
> you will make when a correlated subquery starts showing up in slow query logs.
> Master both, understand when each fits, and let `EXPLAIN` guide every production decision.

---

*All SQL follows the ANSI standard. Compatible with MySQL 8+, MariaDB 10.5+, SQL Server 2016+, SQLite 3.35+.
Engine-specific notes are called out inline.*
