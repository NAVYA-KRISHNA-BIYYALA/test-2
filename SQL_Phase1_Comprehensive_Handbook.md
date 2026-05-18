# SQL Phase 1: Comprehensive Handbook
## Master SQL Fundamentals for Data Engineering

---

## 📑 Table of Contents

1. [Introduction](#introduction)
2. [1. SELECT and FROM Clauses](#1-select-and-from-clauses)
3. [2. WHERE Clause](#2-where-clause)
4. [3. ORDER BY Clause](#3-order-by-clause)
5. [4. Data Types](#4-data-types)
6. [5. NULL Handling](#5-null-handling)
7. [6. Logical and Comparison Operators](#6-logical-and-comparison-operators)
8. [7. Aggregate Functions and GROUP BY](#7-aggregate-functions-and-group-by)
9. [8. JOIN Operations](#8-join-operations)
10. [9. Set Operations](#9-set-operations)
11. [Production Best Practices](#production-best-practices)
12. [Interview Questions](#interview-questions)

---

## Introduction

This handbook covers the fundamental SQL concepts essential for data engineering. Mastering these basics will give you a solid foundation for advanced SQL operations and data manipulation tasks.

---

## 1. SELECT and FROM Clauses

### Overview
The SELECT and FROM clauses are the foundation of every SQL query. SELECT specifies which columns to retrieve, while FROM specifies the table(s) to query.

### Standard Syntax

```sql
SELECT column1, column2, column3
FROM table_name;
```

#### Selecting All Columns
```sql
SELECT *
FROM table_name;
```

#### Selecting Specific Columns
```sql
SELECT first_name, last_name, email
FROM customers;
```

#### Column Aliasing
```sql
SELECT 
    first_name AS 'First Name',
    last_name AS 'Last Name',
    email AS 'Email Address'
FROM customers;
```

### Dialect Variations

#### MySQL
```sql
SELECT column1, column2
FROM table_name
LIMIT 10;
```

#### PostgreSQL
```sql
SELECT column1, column2
FROM table_name
LIMIT 10;
```

#### SQL Server
```sql
SELECT TOP 10 column1, column2
FROM table_name;
```

#### SQLite
```sql
SELECT column1, column2
FROM table_name
LIMIT 10;
```

#### Oracle
```sql
SELECT column1, column2
FROM table_name
WHERE ROWNUM <= 10;
```

### Use Cases

**Case 1: Basic Data Retrieval**
```sql
-- Retrieve all customer names
SELECT customer_id, first_name, last_name
FROM customers;
```

**Case 2: Expression in SELECT**
```sql
-- Calculate derived columns
SELECT 
    product_id,
    unit_price,
    quantity,
    (unit_price * quantity) AS total_amount
FROM order_items;
```

**Case 3: String Concatenation**
```sql
-- MySQL
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM customers;

-- PostgreSQL
SELECT first_name || ' ' || last_name AS full_name
FROM customers;

-- SQL Server
SELECT first_name + ' ' + last_name AS full_name
FROM customers;
```

**Case 4: Selecting Distinct Values**
```sql
SELECT DISTINCT country
FROM customers;
```

### Common Mistakes

❌ **Mistake 1: Selecting Non-Aggregated Columns with GROUP BY**
```sql
-- WRONG - department is not aggregated
SELECT department, employee_name, COUNT(*)
FROM employees
GROUP BY department;
```

❌ **Mistake 2: Using Column Aliases in WHERE Clause**
```sql
-- WRONG - aliases cannot be used in WHERE
SELECT first_name AS fname
FROM customers
WHERE fname = 'John';
```

❌ **Mistake 3: Ambiguous Column Names in JOIN**
```sql
-- WRONG - unclear which table's ID
SELECT id, name
FROM customers
JOIN orders ON customers.id = orders.customer_id;

-- CORRECT
SELECT customers.id, customers.name
FROM customers
JOIN orders ON customers.id = orders.customer_id;
```

### Production Tips

✅ **Tip 1: Always Use Column Names Explicitly**
```sql
-- Good - explicit and maintainable
SELECT customer_id, first_name, last_name, email
FROM customers;

-- Avoid - SELECT * can cause issues if schema changes
SELECT *
FROM customers;
```

✅ **Tip 2: Use Meaningful Aliases**
```sql
-- Good - clear and descriptive
SELECT 
    order_id,
    SUM(unit_price * quantity) AS total_order_amount
FROM order_items
GROUP BY order_id;
```

✅ **Tip 3: Always Qualify Column Names in JOINs**
```sql
-- Good - clear table reference
SELECT 
    c.customer_id,
    c.customer_name,
    o.order_id
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

### Examples

```sql
-- Example 1: Simple SELECT with aliases
SELECT 
    emp_id AS 'Employee ID',
    CONCAT(first_name, ' ', last_name) AS 'Full Name',
    salary AS 'Annual Salary',
    (salary / 12) AS 'Monthly Salary'
FROM employees;

-- Example 2: SELECT with DISTINCT
SELECT DISTINCT 
    department_id,
    job_title
FROM employees
ORDER BY department_id;

-- Example 3: Calculate average with expression
SELECT 
    product_id,
    product_name,
    price,
    (price * 0.9) AS 'Price After 10% Discount'
FROM products
WHERE price > 100;
```

### Interview Questions

**Q1: What is the difference between SELECT * and explicit column selection?**
A: SELECT * retrieves all columns, which can be problematic if the schema changes (new columns may be retrieved unexpectedly). Explicit column selection is better for production code as it's self-documenting and prevents unexpected data retrieval.

**Q2: Can you use column aliases in WHERE clause?**
A: No. Column aliases are created during SELECT execution, but WHERE is evaluated before SELECT. You must use the original column name in WHERE. However, you can use aliases in ORDER BY and HAVING.

**Q3: What is the purpose of aliasing?**
A: Aliasing improves readability, provides meaningful column names in results, and is necessary when joining tables with duplicate column names. It also helps with calculated columns.

---

## 2. WHERE Clause

### Overview
The WHERE clause filters rows based on specified conditions. It's evaluated before the SELECT list and determines which rows are returned.

### Standard Syntax

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

### Basic Conditions

#### Single Condition
```sql
SELECT *
FROM employees
WHERE department = 'Sales';
```

#### Comparison Operators
```sql
SELECT *
FROM products
WHERE price > 100;

SELECT *
FROM orders
WHERE order_date >= '2024-01-01';

SELECT *
FROM employees
WHERE salary <= 50000;

SELECT *
FROM customers
WHERE city != 'New York';  -- or <>
```

### Dialect Variations

#### Date Handling
```sql
-- MySQL
WHERE DATE(order_date) = '2024-01-15';

-- PostgreSQL
WHERE order_date::date = '2024-01-15';

-- SQL Server
WHERE CAST(order_date AS DATE) = '2024-01-15';

-- SQLite
WHERE DATE(order_date) = '2024-01-15';

-- Oracle
WHERE TRUNC(order_date) = TO_DATE('2024-01-15', 'YYYY-MM-DD');
```

#### Case Sensitivity
```sql
-- MySQL (case-insensitive by default)
WHERE country = 'USA';

-- PostgreSQL (case-sensitive)
WHERE LOWER(country) = 'usa';

-- SQL Server (usually case-insensitive)
WHERE UPPER(country) = 'USA';
```

### Use Cases

**Case 1: Single WHERE Condition**
```sql
SELECT employee_id, employee_name, salary
FROM employees
WHERE department_id = 5;
```

**Case 2: Numeric Range**
```sql
SELECT product_id, product_name, price
FROM products
WHERE price BETWEEN 50 AND 200;
```

**Case 3: Text Pattern Matching**
```sql
SELECT customer_id, email
FROM customers
WHERE email LIKE '%@gmail.com';
```

**Case 4: Multiple Conditions**
```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND order_date >= '2024-01-01'
  AND status = 'Completed';
```

**Case 5: NULL Checking**
```sql
SELECT *
FROM employees
WHERE phone_number IS NULL;
```

**Case 6: IN Operator**
```sql
SELECT *
FROM orders
WHERE status IN ('Pending', 'Processing', 'Completed');
```

### Common Mistakes

❌ **Mistake 1: Comparing with NULL using =**
```sql
-- WRONG - NULL comparisons don't work with =
SELECT *
FROM employees
WHERE commission = NULL;

-- CORRECT
SELECT *
FROM employees
WHERE commission IS NULL;
```

❌ **Mistake 2: String Quotes in WHERE**
```sql
-- WRONG in SQL Server and PostgreSQL
WHERE country = USA;

-- CORRECT
WHERE country = 'USA';
```

❌ **Mistake 3: Logic Error with OR**
```sql
-- WRONG - returns all employees (status is always one of these)
SELECT *
FROM employees
WHERE status = 'Active' OR status = 'Inactive' OR 1 = 1;

-- CORRECT
SELECT *
FROM employees
WHERE status IN ('Active', 'Inactive');
```

❌ **Mistake 4: Implicit Type Conversion Issues**
```sql
-- RISKY - relies on implicit conversion
SELECT *
FROM orders
WHERE order_id = '12345';  -- stored as INT

-- BETTER
SELECT *
FROM orders
WHERE order_id = 12345;
```

### Production Tips

✅ **Tip 1: Index Considerations**
```sql
-- Good - can use index if column is indexed
SELECT *
FROM products
WHERE category_id = 5;

-- Avoid - function calls prevent index usage
SELECT *
FROM products
WHERE UPPER(category_name) = 'ELECTRONICS';

-- Better alternative if needed
SELECT *
FROM products
WHERE category_name = 'Electronics';
```

✅ **Tip 2: Use BETWEEN for Ranges (More Readable)**
```sql
-- Good - clear and often optimized
SELECT *
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';

-- Less readable
SELECT *
FROM orders
WHERE order_date >= '2024-01-01' AND order_date <= '2024-12-31';
```

✅ **Tip 3: Explicit Type Comparison**
```sql
-- Good - explicit and safe
SELECT *
FROM orders
WHERE order_date > DATE '2024-01-01';

-- MySQL
WHERE DATE(created_at) > '2024-01-01';
```

✅ **Tip 4: Avoid Functions on Indexed Columns**
```sql
-- Bad - cannot use index
SELECT *
FROM products
WHERE YEAR(creation_date) = 2024;

-- Good - can use index
SELECT *
FROM products
WHERE creation_date >= '2024-01-01' 
  AND creation_date < '2025-01-01';
```

### Examples

```sql
-- Example 1: Complex WHERE conditions
SELECT 
    order_id,
    customer_id,
    order_date,
    total_amount
FROM orders
WHERE status = 'Completed'
  AND order_date >= '2024-01-01'
  AND total_amount > 1000
  AND customer_id NOT IN (1, 2, 3);

-- Example 2: Pattern matching
SELECT 
    employee_id,
    first_name,
    last_name,
    email
FROM employees
WHERE email LIKE '%@company.com'
  AND first_name LIKE 'J%';

-- Example 3: Multiple conditions with IN
SELECT *
FROM products
WHERE status IN ('Active', 'On Sale')
  AND category_id IN (1, 3, 5, 7)
  AND price BETWEEN 10 AND 500;

-- Example 4: NULL and NOT NULL checks
SELECT 
    customer_id,
    customer_name,
    phone,
    fax
FROM customers
WHERE phone IS NOT NULL
  AND fax IS NULL;
```

### Interview Questions

**Q1: Why can't you use = to compare with NULL?**
A: NULL represents an unknown value. In SQL, NULL = NULL evaluates to NULL (unknown), not TRUE. This is why IS NULL and IS NOT NULL operators were created specifically for NULL comparisons.

**Q2: What's the performance difference between BETWEEN and >= / <=?**
A: BETWEEN and >= / <= have similar performance if the query optimizer is smart. However, BETWEEN is more readable and sometimes more optimizable. The database engine typically treats them equivalently.

**Q3: Can you use wildcards (LIKE) with numeric columns?**
A: You can try, but it's not recommended. Numeric columns should be queried with comparison operators (=, <, >, BETWEEN). If you must use LIKE, you'd need to convert the number to a string first, which prevents index usage.

**Q4: What's the difference between WHERE and HAVING?**
A: WHERE filters rows before aggregation, while HAVING filters groups after aggregation. WHERE works on individual rows, HAVING works on aggregate results.

---

## 3. ORDER BY Clause

### Overview
The ORDER BY clause sorts the result set by one or more columns in ascending (ASC) or descending (DESC) order.

### Standard Syntax

```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 ASC, column2 DESC;
```

### Basic Sorting

#### Single Column Ascending
```sql
SELECT *
FROM customers
ORDER BY last_name;
```

#### Single Column Descending
```sql
SELECT *
FROM orders
ORDER BY order_date DESC;
```

#### Multiple Columns
```sql
SELECT *
FROM employees
ORDER BY department ASC, salary DESC;
```

### Dialect Variations

#### Sorting by Column Number
```sql
-- Works in most databases
SELECT first_name, last_name, salary
FROM employees
ORDER BY 1, 3 DESC;  -- Sort by 1st column ASC, 3rd column DESC

-- Note: Column numbers are less readable and not recommended in production
```

#### NULLS First/Last (PostgreSQL, Oracle)
```sql
-- PostgreSQL
SELECT *
FROM employees
ORDER BY commission DESC NULLS LAST;

-- Oracle
SELECT *
FROM employees
ORDER BY commission DESC NULLS LAST;

-- MySQL/SQL Server (use CASE for workaround)
SELECT *
FROM employees
ORDER BY CASE WHEN commission IS NULL THEN 1 ELSE 0 END,
         commission DESC;
```

#### Collation (for string sorting)
```sql
-- SQL Server
SELECT *
FROM customers
ORDER BY customer_name COLLATE SQL_Latin1_General_CI_AS;

-- PostgreSQL
SELECT *
FROM customers
ORDER BY customer_name COLLATE "C";
```

### Use Cases

**Case 1: Basic Sorting**
```sql
-- Sort customers by registration date (newest first)
SELECT customer_id, customer_name, registration_date
FROM customers
ORDER BY registration_date DESC;
```

**Case 2: Multi-Column Sorting**
```sql
-- Sort employees by department, then by salary
SELECT 
    employee_id,
    employee_name,
    department,
    salary
FROM employees
ORDER BY department ASC, salary DESC;
```

**Case 3: Sorting with Expressions**
```sql
-- Sort by calculated field
SELECT 
    product_id,
    unit_price,
    quantity,
    (unit_price * quantity) AS total_value
FROM order_items
ORDER BY (unit_price * quantity) DESC;
```

**Case 4: Sorting by Column Alias**
```sql
-- PostgreSQL and most databases allow this
SELECT 
    employee_name,
    salary,
    (salary * 1.1) AS projected_salary
FROM employees
ORDER BY projected_salary DESC;
```

**Case 5: Conditional Sorting**
```sql
-- Sort VIP customers first, then by name
SELECT *
FROM customers
ORDER BY CASE WHEN is_vip = 1 THEN 0 ELSE 1 END,
         customer_name ASC;
```

### Common Mistakes

❌ **Mistake 1: Using Non-Existent Column in ORDER BY**
```sql
-- WRONG - column doesn't exist
SELECT first_name, last_name
FROM employees
ORDER BY middle_name;
```

❌ **Mistake 2: ORDER BY with Implicit Type Conversion**
```sql
-- String column sorted as text (123, 20, 3)
SELECT *
FROM data
WHERE year = '2024'
ORDER BY month;  -- If month is VARCHAR, sorts as '1', '10', '2'

-- Better - ensure correct data type or convert explicitly
ORDER BY CAST(month AS INT);
```

❌ **Mistake 3: Missing ASC/DESC in Multi-Column Sort**
```sql
-- Unclear intent
SELECT *
FROM employees
ORDER BY department, salary;

-- Better - explicitly state direction
SELECT *
FROM employees
ORDER BY department ASC, salary DESC;
```

❌ **Mistake 4: ORDER BY Column Not in SELECT**
```sql
-- In some databases (PostgreSQL in strict mode), this may cause issues
SELECT first_name, last_name
FROM employees
ORDER BY hire_date;  -- hire_date not in SELECT

-- Better - include in SELECT if sorting by it
SELECT first_name, last_name, hire_date
FROM employees
ORDER BY hire_date DESC;
```

### Production Tips

✅ **Tip 1: Use Column Names, Not Numbers**
```sql
-- Good - clear and maintainable
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC;

-- Avoid - fragile if columns are reordered
SELECT first_name, last_name, salary
FROM employees
ORDER BY 3 DESC;
```

✅ **Tip 2: Be Explicit with ASC/DESC**
```sql
-- Good - clear intent
SELECT *
FROM products
ORDER BY price DESC, product_name ASC;

-- Could be confusing - is second column ASC or DESC?
SELECT *
FROM products
ORDER BY price DESC, product_name;
```

✅ **Tip 3: Handle NULL Values Explicitly**
```sql
-- Good - handles NULLs predictably
SELECT *
FROM employees
ORDER BY CASE WHEN bonus IS NULL THEN 1 ELSE 0 END,
         bonus DESC;

-- Better if database supports it (PostgreSQL, Oracle)
ORDER BY bonus DESC NULLS LAST;
```

✅ **Tip 4: Consider Performance Impact**
```sql
-- Large sorts without index may be slow
-- If sorting large datasets, ensure:
-- 1. Indexed columns where possible
-- 2. Use LIMIT to reduce result set
-- 3. Consider pagination

SELECT *
FROM orders
WHERE status = 'Completed'
ORDER BY order_date DESC
LIMIT 100;
```

### Examples

```sql
-- Example 1: Basic sorting
SELECT 
    product_id,
    product_name,
    unit_price,
    quantity_in_stock
FROM products
ORDER BY unit_price DESC;

-- Example 2: Multi-column sorting
SELECT 
    employee_id,
    first_name,
    last_name,
    department,
    salary
FROM employees
ORDER BY department ASC, salary DESC, last_name ASC;

-- Example 3: Sorting by expression
SELECT 
    order_id,
    order_date,
    total_amount,
    shipping_cost,
    (total_amount - shipping_cost) AS net_amount
FROM orders
ORDER BY (total_amount - shipping_cost) DESC;

-- Example 4: Complex conditional sorting
SELECT 
    customer_id,
    customer_name,
    is_vip,
    total_spent,
    registration_date
FROM customers
ORDER BY 
    CASE WHEN is_vip = 1 THEN 0 ELSE 1 END,
    total_spent DESC,
    registration_date ASC;

-- Example 5: Handling NULL in sort
SELECT 
    employee_id,
    employee_name,
    commission,
    salary
FROM employees
ORDER BY 
    CASE WHEN commission IS NULL THEN 1 ELSE 0 END,
    commission DESC;
```

### Interview Questions

**Q1: What happens if you ORDER BY a column that's not in the SELECT list?**
A: In most databases (MySQL, PostgreSQL, SQL Server), this is allowed and the results are still sorted correctly. However, the unsorted column won't appear in the output. Some databases may require the column in SELECT if DISTINCT is used.

**Q2: How do you sort NULL values to appear last?**
A: Different databases handle this differently. In PostgreSQL and Oracle, use "ORDER BY column DESC NULLS LAST". In MySQL and SQL Server, use "ORDER BY CASE WHEN column IS NULL THEN 1 ELSE 0 END, column DESC".

**Q3: What's the performance impact of ORDER BY?**
A: ORDER BY requires a sort operation, which can be slow for large datasets. If the column is indexed, the database may use an efficient sort. Complex multi-column sorts or sorts on non-indexed columns require full table scans and in-memory sorting, which impacts performance.

**Q4: Can you ORDER BY an alias created in SELECT?**
A: In most databases yes, but in PostgreSQL's strict mode (with DISTINCT), you may need to include the column in SELECT. It's safer to use the actual column name.

---

## 4. Data Types

### Overview
SQL data types define the kind of data a column can store and affect how data is processed, compared, and stored.

### Primary Data Types

### 1. INTEGER / INT / BIGINT

#### Definition
Numeric types for whole numbers without decimal places.

#### Variants
```sql
-- Different integer sizes (bytes and range)
TINYINT        -- 1 byte (-128 to 127)
SMALLINT       -- 2 bytes (-32,768 to 32,767)
INT            -- 4 bytes (-2,147,483,648 to 2,147,483,647)
INTEGER        -- 4 bytes (alias for INT)
BIGINT         -- 8 bytes (-9.2 quintillion to 9.2 quintillion)
```

#### Dialect Variations

```sql
-- MySQL
CREATE TABLE employees (
    emp_id INT AUTO_INCREMENT PRIMARY KEY,
    salary BIGINT
);

-- PostgreSQL
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    salary BIGINT
);

-- SQL Server
CREATE TABLE employees (
    emp_id INT IDENTITY(1,1) PRIMARY KEY,
    salary BIGINT
);

-- SQLite
CREATE TABLE employees (
    emp_id INTEGER PRIMARY KEY AUTOINCREMENT,
    salary INTEGER
);

-- Oracle
CREATE TABLE employees (
    emp_id NUMBER(10) PRIMARY KEY,
    salary NUMBER(15)
);
```

#### Use Cases

```sql
-- Good uses of INT
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    age INT,
    product_quantity INT,
    year_of_birth INT
);

-- Use BIGINT for large numbers
CREATE TABLE transactions (
    transaction_id BIGINT PRIMARY KEY,
    account_balance BIGINT
);
```

#### Common Mistakes

❌ **Mistake 1: Wrong Integer Size**
```sql
-- WRONG - year could need more space than TINYINT can hold
CREATE TABLE data (
    year TINYINT
);

-- CORRECT
CREATE TABLE data (
    year SMALLINT
);
```

❌ **Mistake 2: Using INT for IDs in Distributed Systems**
```sql
-- WRONG - may overflow in large systems
emp_id INT PRIMARY KEY

-- CORRECT for distributed systems
emp_id BIGINT PRIMARY KEY
```

### 2. VARCHAR / CHAR / TEXT

#### Definition
String data types for text storage with different characteristics.

#### Variants

```sql
CHAR(n)        -- Fixed length, padded with spaces if shorter
VARCHAR(n)     -- Variable length, maximum n characters
TEXT           -- Variable length, very large text (no size limit)
NVARCHAR(n)    -- Variable length with Unicode support (SQL Server)
NCHAR(n)       -- Fixed length with Unicode support (SQL Server)
```

#### Dialect Variations

```sql
-- MySQL
CREATE TABLE products (
    product_name VARCHAR(255),
    description TEXT,
    sku CHAR(10)
);

-- PostgreSQL
CREATE TABLE products (
    product_name VARCHAR(255),
    description TEXT,
    sku CHAR(10)
);

-- SQL Server (Unicode support)
CREATE TABLE products (
    product_name NVARCHAR(255),
    description NVARCHAR(MAX),
    sku CHAR(10)
);

-- SQLite (no length limit enforced)
CREATE TABLE products (
    product_name TEXT,
    description TEXT,
    sku TEXT
);

-- Oracle
CREATE TABLE products (
    product_name VARCHAR2(255),
    description CLOB,
    sku CHAR(10)
);
```

#### Use Cases

```sql
-- Fixed-length strings (use CHAR)
CREATE TABLE countries (
    country_code CHAR(2),    -- Always 2 characters
    country_name VARCHAR(100)
);

-- Product descriptions (use TEXT)
CREATE TABLE products (
    product_id INT,
    product_name VARCHAR(255),
    detailed_description TEXT
);

-- Email addresses (use VARCHAR)
CREATE TABLE users (
    email VARCHAR(255) UNIQUE
);
```

#### Common Mistakes

❌ **Mistake 1: Using TEXT for Small Strings**
```sql
-- Wasteful - TEXT uses more storage
CREATE TABLE config (
    setting_value TEXT
);

-- CORRECT
CREATE TABLE config (
    setting_value VARCHAR(100)
);
```

❌ **Mistake 2: Not Specifying VARCHAR Length**
```sql
-- WRONG - may behave unexpectedly
CREATE TABLE data (
    name VARCHAR
);

-- CORRECT
CREATE TABLE data (
    name VARCHAR(255)
);
```

❌ **Mistake 3: Using CHAR for Variable-Length Data**
```sql
-- WRONG - wastes space with padding
CREATE TABLE emails (
    email CHAR(255)
);

-- CORRECT
CREATE TABLE emails (
    email VARCHAR(255)
);
```

#### Production Tips

✅ **Tip 1: Be Conservative with VARCHAR Length**
```sql
-- Good - allows expansion but reasonable limit
email VARCHAR(254)  -- RFC 5321 standard for email

-- Set appropriate limits
first_name VARCHAR(50)
last_name VARCHAR(50)
street_address VARCHAR(255)
```

✅ **Tip 2: Use CHAR for Fixed Codes**
```sql
-- Good - known fixed length
country_code CHAR(2),
currency_code CHAR(3)
```

### 3. DATE

#### Definition
Stores date values in YYYY-MM-DD format.

#### Variants
```sql
DATE           -- Date only (YYYY-MM-DD)
DATETIME       -- Date and time (YYYY-MM-DD HH:MM:SS)
TIMESTAMP      -- Date, time, and timezone
TIME           -- Time only (HH:MM:SS)
YEAR           -- Year only (YYYY)
```

#### Dialect Variations

```sql
-- MySQL
CREATE TABLE orders (
    order_date DATE,
    created_at DATETIME,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- PostgreSQL
CREATE TABLE orders (
    order_date DATE,
    created_at TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE
);

-- SQL Server
CREATE TABLE orders (
    order_date DATE,
    created_at DATETIME2,
    updated_at DATETIMEOFFSET
);

-- SQLite
CREATE TABLE orders (
    order_date TEXT,  -- SQLite stores as TEXT
    created_at TEXT
);

-- Oracle
CREATE TABLE orders (
    order_date DATE,
    created_at TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE
);
```

#### Use Cases

```sql
-- DATE for business logic dates
CREATE TABLE employees (
    hire_date DATE,
    birth_date DATE
);

-- DATETIME/TIMESTAMP for audit trails
CREATE TABLE orders (
    order_id INT,
    order_date DATE,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    deleted_at TIMESTAMP NULL
);

-- Querying dates
SELECT *
FROM orders
WHERE order_date = '2024-01-15';

SELECT *
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

#### Common Mistakes

❌ **Mistake 1: Storing Dates as VARCHAR**
```sql
-- WRONG - can't compare or sort properly
CREATE TABLE data (
    date_field VARCHAR(10)
);

-- CORRECT
CREATE TABLE data (
    date_field DATE
);
```

❌ **Mistake 2: Incorrect Date Format in Comparison**
```sql
-- May fail depending on database setting
SELECT * FROM orders WHERE order_date = '15-01-2024';

-- CORRECT - use YYYY-MM-DD
SELECT * FROM orders WHERE order_date = '2024-01-15';
```

❌ **Mistake 3: Not Using TIMESTAMP for Audit Fields**
```sql
-- WRONG - won't automatically update
created_at DATETIME DEFAULT NOW(),
updated_at DATETIME

-- CORRECT
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
```

### 4. BOOLEAN

#### Definition
Stores TRUE/FALSE or 1/0 values.

#### Dialect Variations

```sql
-- PostgreSQL (native BOOLEAN)
CREATE TABLE users (
    is_active BOOLEAN,
    is_premium BOOLEAN
);

-- MySQL (TINYINT(1) or BOOL alias)
CREATE TABLE users (
    is_active BOOLEAN,      -- alias for TINYINT(1)
    is_premium TINYINT(1)
);

-- SQL Server (BIT)
CREATE TABLE users (
    is_active BIT,
    is_premium BIT
);

-- SQLite (INTEGER 0 or 1)
CREATE TABLE users (
    is_active INTEGER,
    is_premium INTEGER
);

-- Oracle (NUMBER(1) or CHAR(1))
CREATE TABLE users (
    is_active NUMBER(1),
    is_premium CHAR(1)
);
```

#### Use Cases

```sql
-- Boolean for status flags
CREATE TABLE customers (
    customer_id INT,
    is_active BOOLEAN,
    is_vip BOOLEAN,
    is_verified BOOLEAN
);

-- Querying boolean columns
SELECT *
FROM customers
WHERE is_active = TRUE;

-- MySQL/SQL Server
SELECT *
FROM customers
WHERE is_active = 1;

-- Negation
SELECT *
FROM customers
WHERE is_active != TRUE;

-- PostgreSQL style
SELECT *
FROM customers
WHERE NOT is_active;
```

#### Common Mistakes

❌ **Mistake 1: Using VARCHAR for Boolean**
```sql
-- WRONG
is_active VARCHAR(5)  -- 'true', 'false', 'yes', 'no'?

-- CORRECT
is_active BOOLEAN
```

❌ **Mistake 2: NULL Boolean Ambiguity**
```sql
-- WRONG - three-valued logic can be confusing
is_deleted BOOLEAN NULL

-- BETTER - use default
is_deleted BOOLEAN DEFAULT FALSE

-- OR use nullable with explicit NULL handling
is_deleted BOOLEAN NULL CHECK (is_deleted IS NOT NULL OR is_deleted IS NULL)
```

### Data Type Comparison Table

| Data Type | Size | Use Case | Storage |
|-----------|------|----------|---------|
| INT | 4 bytes | IDs, counts, ages | Efficient |
| BIGINT | 8 bytes | Large IDs, balances | More storage |
| VARCHAR(n) | Variable | Flexible text | Optimal |
| CHAR(n) | Fixed | Fixed codes (ISO codes) | Over-allocate |
| TEXT | Variable | Large text | Flexible but slow for comparison |
| DATE | 3 bytes | Dates without time | Efficient |
| DATETIME | 8 bytes | Dates with time | Standard for most |
| TIMESTAMP | 4-8 bytes | Audit trails with timezone | Good for UTC |
| BOOLEAN | 1 byte | Status flags | Very efficient |

### Production Tips

✅ **Tip 1: Choose Appropriate Data Types**
```sql
-- Good - right type for each column
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(255),
    description TEXT,
    price DECIMAL(10,2),
    quantity INT,
    is_active BOOLEAN,
    created_at TIMESTAMP
);
```

✅ **Tip 2: Use DECIMAL for Money, Not FLOAT**
```sql
-- WRONG - floating point errors
price FLOAT

-- CORRECT
price DECIMAL(10, 2)  -- 10 digits total, 2 after decimal
```

✅ **Tip 3: Always Use Timestamps for Audit Fields**
```sql
-- Good practice
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
deleted_at TIMESTAMP NULL
```

### Examples

```sql
-- Example 1: Complete table with appropriate types
CREATE TABLE employees (
    emp_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(254) UNIQUE,
    phone_number CHAR(10),
    birth_date DATE,
    hire_date DATE NOT NULL,
    salary DECIMAL(10, 2),
    is_active BOOLEAN DEFAULT TRUE,
    department_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Example 2: Using the right type for comparisons
SELECT 
    emp_id,
    CONCAT(first_name, ' ', last_name) AS full_name,
    salary,
    hire_date,
    DATEDIFF(CURDATE(), hire_date) AS days_employed
FROM employees
WHERE is_active = TRUE
  AND hire_date >= '2020-01-01'
  AND salary > 50000
ORDER BY salary DESC;
```

### Interview Questions

**Q1: Why should you use DECIMAL for money instead of FLOAT?**
A: FLOAT uses binary representation which cannot precisely represent all decimal values (e.g., 0.1 + 0.2 ≠ 0.3 in floating point). DECIMAL stores exact decimal values, which is critical for financial calculations to avoid rounding errors.

**Q2: What's the difference between VARCHAR and CHAR?**
A: CHAR is fixed-length and pads with spaces if the data is shorter, while VARCHAR is variable-length and only uses needed space. CHAR is better for fixed-length codes (country codes), VARCHAR for variable-length data (names, emails).

**Q3: When would you use TIMESTAMP instead of DATETIME?**
A: TIMESTAMP automatically updates when a record is modified, stores timezone information, and is more space-efficient (4 bytes vs 8 bytes). DATETIME is for storing date/time values without automatic updates.

**Q4: Can you store text longer than VARCHAR(255) allows?**
A: Yes, use TEXT type which can store very large text. However, TEXT doesn't allow indexing on the full column (only on a prefix) and is slower for comparisons.

---

## 5. NULL Handling

### Overview
NULL represents unknown, missing, or undefined values. Special operators are required for NULL comparisons because NULL doesn't equal anything, not even NULL.

### Core Concepts

#### NULL = NULL is Unknown, Not TRUE
```sql
-- These all return NULL (unknown), not TRUE or FALSE
SELECT NULL = NULL;        -- NULL
SELECT NULL <> NULL;       -- NULL
SELECT 5 = NULL;           -- NULL
SELECT NULL > 10;          -- NULL
```

### IS NULL and IS NOT NULL

#### Standard Syntax

```sql
SELECT *
FROM table_name
WHERE column_name IS NULL;

SELECT *
FROM table_name
WHERE column_name IS NOT NULL;
```

#### Basic Examples

```sql
-- Find customers with no phone number
SELECT customer_id, customer_name
FROM customers
WHERE phone_number IS NULL;

-- Find employees with commission
SELECT employee_id, employee_name, commission
FROM employees
WHERE commission IS NOT NULL;
```

#### Dialect Variations

All SQL dialects support IS NULL and IS NOT NULL identically.

```sql
-- All databases
WHERE column IS NULL
WHERE column IS NOT NULL
```

### Use Cases

**Case 1: Identifying Missing Data**
```sql
SELECT 
    customer_id,
    customer_name,
    email,
    phone
FROM customers
WHERE email IS NULL
   OR phone IS NULL;
```

**Case 2: Handling Optional Fields**
```sql
SELECT 
    order_id,
    customer_id,
    special_instructions
FROM orders
WHERE special_instructions IS NOT NULL;
```

**Case 3: Data Quality Checks**
```sql
-- Find records with incomplete data
SELECT *
FROM employees
WHERE phone_number IS NULL
   OR email IS NULL
   OR manager_id IS NULL;
```

### COALESCE Function

#### Definition
Returns the first non-NULL value in a list of arguments.

#### Standard Syntax

```sql
COALESCE(column1, column2, column3, default_value)
```

#### Examples

```sql
-- Return commission if not NULL, otherwise 0
SELECT 
    employee_id,
    employee_name,
    COALESCE(commission, 0) AS commission
FROM employees;

-- Use first available contact method
SELECT 
    customer_id,
    customer_name,
    COALESCE(email, phone, mailing_address) AS contact_method
FROM customers;
```

#### Dialect Variations

```sql
-- All databases support COALESCE
-- Standard SQL
SELECT COALESCE(email, phone, 'No contact')
FROM customers;

-- MySQL also has IFNULL (similar but only 2 arguments)
SELECT IFNULL(commission, 0) AS commission
FROM employees;

-- SQL Server also has ISNULL
SELECT ISNULL(commission, 0) AS commission
FROM employees;
```

#### Use Cases

**Case 1: Default Values in SELECT**
```sql
SELECT 
    order_id,
    COALESCE(discount_percentage, 0) AS discount,
    COALESCE(notes, 'No special notes') AS order_notes
FROM orders;
```

**Case 2: Handling Missing Data in Joins**
```sql
SELECT 
    c.customer_id,
    c.customer_name,
    COALESCE(SUM(o.order_amount), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id;
```

**Case 3: Multi-Level Fallback**
```sql
-- Try preferred email, then alternative, then use name at domain
SELECT 
    customer_id,
    COALESCE(
        preferred_email,
        work_email,
        personal_email,
        CONCAT(customer_name, '@company.com')
    ) AS email_to_use
FROM customers;
```

### NULLIF Function

#### Definition
Returns NULL if two expressions are equal; otherwise returns the first expression.

#### Standard Syntax

```sql
NULLIF(expression1, expression2)
```

#### Equivalent Logic
```sql
NULLIF(value1, value2) 
-- is equivalent to 
CASE WHEN value1 = value2 THEN NULL ELSE value1 END
```

#### Examples

```sql
-- Return commission only if it differs from salary
SELECT 
    employee_id,
    salary,
    NULLIF(commission, salary) AS commission
FROM employees;

-- Convert zero to NULL
SELECT 
    product_id,
    NULLIF(quantity_in_stock, 0) AS available_quantity
FROM products;
```

#### Common Uses

```sql
-- Avoid division by zero
SELECT 
    order_id,
    NULLIF(total_items, 0) AS item_count
FROM orders;

-- Better - in division
SELECT 
    employee_id,
    total_sales / NULLIF(num_customers, 0) AS avg_sale_per_customer
FROM sales_summary;
```

#### Dialect Variations

All SQL dialects support NULLIF.

```sql
-- PostgreSQL
SELECT NULLIF(status, 'inactive');

-- MySQL
SELECT NULLIF(status, 'inactive');

-- SQL Server
SELECT NULLIF(status, 'inactive');
```

### Common Mistakes

❌ **Mistake 1: Using = or <> with NULL**
```sql
-- WRONG - always returns NULL result
SELECT *
FROM employees
WHERE commission = NULL;      -- Never true!
WHERE bonus <> NULL;          -- Never true!

-- CORRECT
SELECT *
FROM employees
WHERE commission IS NULL;
WHERE bonus IS NOT NULL;
```

❌ **Mistake 2: Not Handling NULL in Calculations**
```sql
-- WRONG - if commission is NULL, result is NULL
SELECT 
    employee_id,
    salary + commission AS total_compensation
FROM employees;

-- CORRECT
SELECT 
    employee_id,
    salary + COALESCE(commission, 0) AS total_compensation
FROM employees;
```

❌ **Mistake 3: Forgetting NULL in Aggregates**
```sql
-- NULL values are excluded from COUNT except COUNT(*)
SELECT COUNT(commission) FROM employees;     -- Counts non-NULL
SELECT COUNT(*) FROM employees;              -- Counts all rows

-- If you want to count NULL values
SELECT COUNT(CASE WHEN commission IS NULL THEN 1 END) 
FROM employees;
```

❌ **Mistake 4: NULL in String Concatenation**
```sql
-- WRONG - if middle_name is NULL, whole result is NULL
SELECT first_name + ' ' + middle_name + ' ' + last_name
FROM employees;

-- CORRECT - remove NULLs
SELECT CONCAT(first_name, ' ', COALESCE(middle_name, ''), ' ', last_name)
FROM employees;

-- Or use NULL-safe concatenation
SELECT first_name || ' ' || COALESCE(middle_name, '') || ' ' || last_name
FROM employees;
```

### Production Tips

✅ **Tip 1: Always Use IS NULL / IS NOT NULL**
```sql
-- Good - clear and correct
WHERE email IS NOT NULL

-- Avoid
WHERE email != NULL       -- Wrong! Returns NULL
WHERE email IS NOT NULL   -- Wait, double negative
```

✅ **Tip 2: Use COALESCE for Default Values**
```sql
-- Good - clear intent
SELECT 
    COALESCE(discount_rate, 0) AS discount,
    COALESCE(notes, 'None') AS special_notes
FROM orders;
```

✅ **Tip 3: Handle NULL in GROUP BY**
```sql
-- Good - explicitly show NULL groups
SELECT 
    COALESCE(category, 'Uncategorized') AS category,
    COUNT(*) AS product_count
FROM products
GROUP BY category;
```

✅ **Tip 4: Document NULL Semantics**
```sql
-- Good practice - document what NULL means
CREATE TABLE employees (
    -- NULL means not yet assigned
    manager_id INT NULL REFERENCES employees(emp_id),
    -- NULL means no commission
    commission DECIMAL(10,2) NULL,
    -- NULL means not applicable
    clearance_level VARCHAR(50) NULL
);
```

### Examples

```sql
-- Example 1: Handling optional fields
SELECT 
    customer_id,
    customer_name,
    COALESCE(phone_number, 'No phone') AS phone,
    COALESCE(fax_number, 'No fax') AS fax,
    CASE WHEN email IS NULL THEN 'Missing' ELSE 'Present' END AS email_status
FROM customers;

-- Example 2: Avoiding NULL propagation
SELECT 
    order_id,
    customer_id,
    order_date,
    COALESCE(discount, 0) AS discount_amount,
    total_amount * (1 - COALESCE(discount, 0) / 100) AS final_amount
FROM orders;

-- Example 3: Conditional NULL handling in aggregates
SELECT 
    department_id,
    COUNT(*) AS total_employees,
    COUNT(commission) AS commissioned_employees,
    COUNT(CASE WHEN commission IS NULL THEN 1 END) AS non_commissioned,
    COALESCE(AVG(commission), 0) AS avg_commission
FROM employees
GROUP BY department_id;

-- Example 4: Using NULLIF for calculations
SELECT 
    order_id,
    total_items,
    total_amount,
    CASE WHEN NULLIF(total_items, 0) IS NULL 
         THEN NULL 
         ELSE total_amount / total_items 
    END AS price_per_item
FROM orders;

-- Example 5: Finding data quality issues
SELECT 
    customer_id,
    customer_name,
    CASE WHEN email IS NULL THEN 'Missing' ELSE 'OK' END AS email_status,
    CASE WHEN phone_number IS NULL THEN 'Missing' ELSE 'OK' END AS phone_status,
    CASE WHEN address IS NULL THEN 'Missing' ELSE 'OK' END AS address_status
FROM customers
WHERE email IS NULL OR phone_number IS NULL OR address IS NULL;
```

### Interview Questions

**Q1: Why does NULL = NULL return NULL instead of TRUE?**
A: NULL represents an unknown value. If both sides are unknown, we can't say they're equal—the result is also unknown (NULL). This is part of SQL's three-valued logic (TRUE, FALSE, NULL).

**Q2: What's the difference between COUNT(*) and COUNT(column_name)?**
A: COUNT(*) counts all rows, while COUNT(column_name) counts only non-NULL values in that column. This is important because NULL values are excluded from column counts.

**Q3: When would you use NULLIF instead of CASE?**
A: NULLIF is simpler and more readable for the specific case of converting one value to NULL based on equality. For complex conditions, CASE is more flexible. NULLIF(col, 0) is clearer than CASE WHEN col = 0 THEN NULL ELSE col END.

**Q4: How does NULL behave in string concatenation?**
A: In most SQL dialects, NULL concatenated with any string produces NULL. Use COALESCE or CONCAT_WS to safely handle NULL values in string operations.

---

## 6. Logical and Comparison Operators

### Overview
These operators are used in WHERE and HAVING clauses to build complex filter conditions.

### Comparison Operators

#### Definition and Usage

```sql
=       -- Equal
<>      -- Not equal (also !=)
<       -- Less than
>       -- Greater than
<=      -- Less than or equal
>=      -- Greater than or equal
```

#### Examples

```sql
-- Equality
SELECT * FROM products WHERE price = 99.99;

-- Not equal (two styles)
SELECT * FROM employees WHERE status <> 'Inactive';
SELECT * FROM employees WHERE status != 'Inactive';  -- MySQL style

-- Comparisons
SELECT * FROM orders WHERE order_amount > 1000;
SELECT * FROM employees WHERE hire_date <= '2020-01-01';
```

### Logical Operators

#### AND Operator

**Definition**: Both conditions must be TRUE.

```sql
SELECT *
FROM orders
WHERE status = 'Completed'
  AND order_date >= '2024-01-01'
  AND order_amount > 500;
```

#### OR Operator

**Definition**: At least one condition must be TRUE.

```sql
SELECT *
FROM customers
WHERE country = 'USA'
   OR country = 'Canada'
   OR country = 'Mexico';
```

#### NOT Operator

**Definition**: Negates a condition.

```sql
SELECT *
FROM employees
WHERE NOT status = 'Inactive';

-- Equivalent
SELECT *
FROM employees
WHERE status <> 'Inactive';
```

### IN Operator

#### Standard Syntax

```sql
SELECT *
FROM table_name
WHERE column IN (value1, value2, value3);
```

#### Equivalent Logic

```sql
-- These are equivalent
WHERE status IN ('Active', 'Pending', 'Processing')

WHERE status = 'Active' OR status = 'Pending' OR status = 'Processing'
```

#### Use Cases

**Case 1: Multiple Values**
```sql
SELECT *
FROM orders
WHERE status IN ('Shipped', 'Delivered', 'Completed');
```

**Case 2: IN with Subquery**
```sql
SELECT *
FROM employees
WHERE department_id IN (
    SELECT dept_id FROM departments WHERE location = 'New York'
);
```

**Case 3: NOT IN**
```sql
SELECT *
FROM products
WHERE product_id NOT IN (101, 102, 103);
```

#### Dialect Variations

All databases support IN operator. Syntax is identical.

```sql
-- All databases
WHERE id IN (1, 2, 3, 4, 5)
WHERE city IN ('New York', 'Los Angeles', 'Chicago')
```

#### Common Mistakes

❌ **Mistake 1: NULL in NOT IN**
```sql
-- WRONG - returns no results if list contains NULL
SELECT * FROM products WHERE id NOT IN (1, 2, NULL);

-- CORRECT
SELECT * FROM products 
WHERE id NOT IN (1, 2) 
  AND id IS NOT NULL;

-- OR use NOT EXISTS instead
SELECT p1.* FROM products p1
WHERE NOT EXISTS (
    SELECT 1 FROM products p2 
    WHERE p2.id IN (1, 2) AND p2.id = p1.id
);
```

❌ **Mistake 2: Using IN Instead of EXISTS for Subqueries (Performance)**
```sql
-- SLOW - scans entire table
SELECT * FROM orders
WHERE customer_id IN (
    SELECT customer_id FROM customers WHERE country = 'USA'
);

-- FASTER - stops when match found
SELECT * FROM orders o
WHERE EXISTS (
    SELECT 1 FROM customers c 
    WHERE c.customer_id = o.customer_id 
    AND c.country = 'USA'
);
```

### BETWEEN Operator

#### Standard Syntax

```sql
SELECT *
FROM table_name
WHERE column BETWEEN value1 AND value2;
```

#### Equivalent Logic

```sql
-- These are equivalent
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'

WHERE order_date >= '2024-01-01' AND order_date <= '2024-12-31'
```

#### Use Cases

**Case 1: Date Range**
```sql
SELECT *
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

**Case 2: Numeric Range**
```sql
SELECT *
FROM products
WHERE price BETWEEN 50 AND 200;
```

**Case 3: NOT BETWEEN**
```sql
SELECT *
FROM employees
WHERE salary NOT BETWEEN 40000 AND 60000;
```

#### Dialect Variations

All databases support BETWEEN with identical syntax.

```sql
-- All databases
WHERE age BETWEEN 18 AND 65
WHERE salary BETWEEN 40000 AND 100000
```

#### Production Tips

✅ **Tip 1: BETWEEN is Inclusive**
```sql
-- Both boundaries are included
WHERE price BETWEEN 10 AND 100  -- Includes 10 and 100

-- If you want exclusive upper bound
WHERE price BETWEEN 10 AND 99.99  -- Or use < 100
```

✅ **Tip 2: BETWEEN with Dates**
```sql
-- For dates with time components
WHERE created_at BETWEEN '2024-01-01' AND '2024-01-01 23:59:59'

-- Better - use date functions
WHERE DATE(created_at) BETWEEN '2024-01-01' AND '2024-01-31'
```

### LIKE Operator

#### Standard Syntax

```sql
SELECT *
FROM table_name
WHERE column LIKE 'pattern';
```

#### Wildcard Patterns

```sql
%   -- Matches zero or more characters
_   -- Matches exactly one character

-- Examples
'A%'      -- Starts with A
'%B'      -- Ends with B
'%C%'     -- Contains C
'A_C'     -- A, any character, C
'_A%'     -- Any character, then A, then anything
```

#### Use Cases

**Case 1: Prefix Matching**
```sql
SELECT *
FROM employees
WHERE last_name LIKE 'Smith%';
```

**Case 2: Contains Pattern**
```sql
SELECT *
FROM products
WHERE product_name LIKE '%organic%';
```

**Case 3: Suffix Matching**
```sql
SELECT *
FROM emails
WHERE email LIKE '%@gmail.com';
```

**Case 4: NOT LIKE**
```sql
SELECT *
FROM customers
WHERE customer_name NOT LIKE 'Mr.%';
```

#### Dialect Variations

**Case Sensitivity**
```sql
-- MySQL (case-insensitive by default)
WHERE name LIKE 'john'  -- Matches 'John', 'JOHN', 'john'

-- PostgreSQL (case-sensitive)
WHERE name LIKE 'john'  -- Only matches 'john'
WHERE name ILIKE 'john' -- Case-insensitive

-- SQL Server (case-insensitive by default)
WHERE name LIKE 'john'  -- Matches 'John', 'JOHN', etc.

-- Oracle (case-sensitive by default)
WHERE UPPER(name) LIKE 'JOHN'  -- Use UPPER() for case-insensitive
```

**Regular Expressions (if supported)**
```sql
-- MySQL
WHERE email REGEXP '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

-- PostgreSQL
WHERE email ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
```

#### Common Mistakes

❌ **Mistake 1: LIKE on Non-String Columns**
```sql
-- SLOW and problematic - requires conversion
SELECT * FROM orders
WHERE CAST(order_id AS VARCHAR) LIKE '123%';

-- BETTER - use comparison operators
SELECT * FROM orders
WHERE order_id LIKE 123000 AND order_id < 124000;

-- OR use numeric comparison
SELECT * FROM orders
WHERE order_id >= 123000 AND order_id < 124000;
```

❌ **Mistake 2: Performance Issues with Leading %**
```sql
-- SLOW - can't use index
WHERE product_name LIKE '%widget%';

-- FASTER if possible - uses index
WHERE product_name LIKE 'widget%';
```

❌ **Mistake 3: Not Escaping Special Characters**
```sql
-- WRONG - % and _ are special in LIKE
SELECT * FROM files
WHERE filename LIKE 'report_2024.txt';  -- _ matches any character!

-- CORRECT - escape special characters
SELECT * FROM files
WHERE filename LIKE 'report\_2024.txt' ESCAPE '\';

-- MySQL
WHERE filename LIKE 'report\\_2024.txt';

-- PostgreSQL
WHERE filename LIKE 'report\_2024.txt' ESCAPE '\';
```

### Production Tips

✅ **Tip 1: Prefer Comparison Operators**
```sql
-- Good for indexed columns
WHERE status = 'Active'
WHERE price > 100

-- LIKE is slower
WHERE status LIKE 'Active%'
```

✅ **Tip 2: Use BETWEEN for Ranges**
```sql
-- Clearer and often optimized
WHERE hire_date BETWEEN '2020-01-01' AND '2020-12-31'

-- Instead of
WHERE hire_date >= '2020-01-01' AND hire_date < '2021-01-01'
```

✅ **Tip 3: Be Careful with OR**
```sql
-- Good - can use index
WHERE status = 'Active' OR status = 'Pending'

-- Better - use IN
WHERE status IN ('Active', 'Pending')
```

✅ **Tip 4: Build Conditions Carefully**
```sql
-- Good - explicit and readable
WHERE (status = 'Active' AND priority = 'High')
   OR (status = 'Pending' AND priority = 'Critical')

-- Use parentheses to clarify intent and ensure correct precedence
```

### Examples

```sql
-- Example 1: Complex logical conditions
SELECT 
    employee_id,
    first_name,
    last_name,
    salary,
    department
FROM employees
WHERE (department IN ('Sales', 'Marketing') AND salary > 50000)
   OR (department = 'IT' AND salary > 60000)
ORDER BY salary DESC;

-- Example 2: Multiple filters with IN and BETWEEN
SELECT 
    order_id,
    customer_id,
    order_date,
    total_amount
FROM orders
WHERE status IN ('Completed', 'Shipped')
  AND order_date BETWEEN '2024-01-01' AND '2024-12-31'
  AND total_amount BETWEEN 100 AND 10000;

-- Example 3: Pattern matching
SELECT 
    customer_id,
    customer_name,
    email,
    phone
FROM customers
WHERE customer_name LIKE '%Inc%'
   OR customer_name LIKE '%Ltd%'
   OR email LIKE '%@company.com';

-- Example 4: NOT operators
SELECT 
    product_id,
    product_name,
    category
FROM products
WHERE NOT (category IN ('Discontinued', 'Coming Soon'))
  AND product_name NOT LIKE '%old%'
  AND price NOT BETWEEN 0 AND 10;

-- Example 5: Complex real-world filter
SELECT 
    order_id,
    customer_id,
    order_date,
    status,
    total_amount
FROM orders
WHERE (status IN ('Pending', 'Processing') 
       AND order_date <= DATE_SUB(CURDATE(), INTERVAL 7 DAY))
   OR (status = 'Completed' 
       AND total_amount > 5000 
       AND order_date BETWEEN '2024-01-01' AND '2024-03-31');
```

### Interview Questions

**Q1: What's the performance impact of IN vs multiple OR conditions?**
A: Generally, IN is optimized better by most query engines and is readable. Large IN lists may need optimization. EXISTS is often faster than IN for subqueries with large result sets.

**Q2: Why does NOT IN with NULL return no results?**
A: NOT IN (1, 2, NULL) returns no rows because NULL makes the comparison unknown. The logic is: if id NOT IN list, and NULL is in list, we can't say if id matches NULL (unknown), so the result is unknown, making the whole condition fail.

**Q3: What's the difference between LIKE and regular expressions?**
A: LIKE is simpler with basic wildcards (% and _), while regex is more powerful for complex patterns. LIKE is faster for simple patterns; regex for complex matching (email validation, complex formats).

**Q4: Why is LIKE with leading % slow?**
A: Because the database can't use a B-tree index efficiently when the pattern starts with %. Indexes work left-to-right, so '%pattern' requires a full table scan, while 'pattern%' can use the index.

---

## 7. Aggregate Functions and GROUP BY

### Overview
Aggregate functions compute a single value from a set of rows. GROUP BY divides rows into groups and applies aggregate functions to each group.

### Aggregate Functions

#### COUNT Function

**Definition**: Counts rows or non-NULL values.

```sql
-- Count all rows
SELECT COUNT(*)
FROM orders;

-- Count non-NULL values in a column
SELECT COUNT(commission)
FROM employees;

-- Count distinct values
SELECT COUNT(DISTINCT country)
FROM customers;
```

**Syntax Variations**
```sql
COUNT(*)                -- Count all rows
COUNT(column)           -- Count non-NULL values
COUNT(DISTINCT column)  -- Count unique non-NULL values
```

#### SUM Function

**Definition**: Adds numeric values.

```sql
-- Sum all values
SELECT SUM(order_amount)
FROM orders;

-- Sum with condition
SELECT SUM(salary)
FROM employees
WHERE department = 'Sales';

-- Sum distinct values
SELECT SUM(DISTINCT price)
FROM order_items;
```

**Syntax**
```sql
SUM(column)             -- Sum all values
SUM(DISTINCT column)    -- Sum unique values
SUM(CASE WHEN ... THEN ... END)  -- Conditional sum
```

#### AVG Function

**Definition**: Calculates average value.

```sql
-- Average value
SELECT AVG(salary)
FROM employees;

-- Average with filtering
SELECT AVG(price)
FROM products
WHERE category = 'Electronics';

-- Average distinct values
SELECT AVG(DISTINCT price)
FROM order_items;
```

**Syntax**
```sql
AVG(column)             -- Average of values
AVG(DISTINCT column)    -- Average of unique values
```

#### MIN and MAX Functions

**Definition**: Find minimum and maximum values.

```sql
-- Minimum and maximum
SELECT 
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary
FROM employees;

-- With multiple columns
SELECT 
    MIN(order_date) AS first_order,
    MAX(order_date) AS latest_order
FROM orders;
```

**Syntax**
```sql
MIN(column)  -- Minimum value
MAX(column)  -- Maximum value
```

### Use Cases for Aggregate Functions

**Case 1: Single Row Aggregates**
```sql
SELECT 
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_revenue,
    AVG(total_amount) AS average_order_value,
    MIN(total_amount) AS smallest_order,
    MAX(total_amount) AS largest_order
FROM orders;
```

**Case 2: Group Aggregates**
```sql
SELECT 
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS average_salary,
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary
FROM employees
GROUP BY department;
```

**Case 3: Conditional Aggregates**
```sql
SELECT 
    COUNT(CASE WHEN status = 'Completed' THEN 1 END) AS completed_orders,
    COUNT(CASE WHEN status = 'Pending' THEN 1 END) AS pending_orders,
    SUM(CASE WHEN status = 'Completed' THEN total_amount ELSE 0 END) AS revenue
FROM orders;
```

### GROUP BY Clause

#### Standard Syntax

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;
```

#### Basic Grouping

```sql
-- Group by single column
SELECT 
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;

-- Group by multiple columns
SELECT 
    department,
    job_title,
    COUNT(*) AS count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department, job_title;
```

#### Dialect Variations

```sql
-- All databases support GROUP BY similarly

-- MySQL (flexible GROUP BY)
SELECT 
    department,
    first_name,      -- Not aggregated - MySQL allows this
    COUNT(*)
FROM employees
GROUP BY department;

-- PostgreSQL (strict GROUP BY)
SELECT 
    department,
    COUNT(*) 
FROM employees
GROUP BY department;
-- first_name must be in GROUP BY if included in SELECT

-- SQL Server
SELECT 
    department,
    COUNT(*) 
FROM employees
GROUP BY department;

-- Oracle
SELECT 
    department,
    COUNT(*) 
FROM employees
GROUP BY department;
```

### HAVING Clause

#### Definition
HAVING filters groups (after GROUP BY), while WHERE filters rows (before GROUP BY).

#### Syntax

```sql
SELECT column1, COUNT(*) AS count
FROM table_name
WHERE condition1
GROUP BY column1
HAVING condition2;
```

#### Examples

```sql
-- Filter groups
SELECT 
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

-- HAVING with multiple conditions
SELECT 
    category,
    COUNT(*) AS product_count,
    AVG(price) AS avg_price
FROM products
GROUP BY category
HAVING COUNT(*) > 3 AND AVG(price) > 50;

-- Filter specific aggregates
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    SUM(order_amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(order_amount) > 5000;
```

#### WHERE vs HAVING

```sql
-- WHERE filters rows BEFORE grouping
-- HAVING filters groups AFTER grouping

SELECT 
    department,
    AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= '2020-01-01'   -- Filter rows first
GROUP BY department
HAVING AVG(salary) > 50000;       -- Then filter groups
```

### Common Mistakes

❌ **Mistake 1: Including Non-Aggregated Column in SELECT**
```sql
-- WRONG - employee_name not aggregated or in GROUP BY
SELECT 
    department,
    employee_name,
    AVG(salary)
FROM employees
GROUP BY department;

-- CORRECT
SELECT 
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

-- OR if you need names
SELECT 
    department,
    employee_name,
    salary
FROM employees
ORDER BY department, salary;
```

❌ **Mistake 2: Using WHERE Instead of HAVING**
```sql
-- WRONG - COUNT(*) not available in WHERE
SELECT 
    department,
    COUNT(*) AS emp_count,
    AVG(salary)
FROM employees
WHERE COUNT(*) > 5      -- Error!
GROUP BY department;

-- CORRECT
SELECT 
    department,
    COUNT(*) AS emp_count,
    AVG(salary)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

❌ **Mistake 3: NULL Behavior in GROUP BY**
```sql
-- NULL is treated as a distinct group value
SELECT 
    COALESCE(manager_id, 'Unassigned') AS manager,
    COUNT(*) AS employee_count
FROM employees
GROUP BY manager_id;
```

❌ **Mistake 4: SUM/AVG on NULL**
```sql
-- NULL values excluded from aggregates
SELECT 
    SUM(commission),    -- Excludes NULLs
    AVG(commission)     -- Excludes NULLs
FROM employees;

-- If you want to include NULL as 0
SELECT 
    SUM(COALESCE(commission, 0)) AS total_commission,
    AVG(COALESCE(commission, 0)) AS avg_commission
FROM employees;
```

### Production Tips

✅ **Tip 1: Always Use Aliases with Aggregates**
```sql
-- Good - clear what each value represents
SELECT 
    department,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

✅ **Tip 2: Use Conditional Aggregates Instead of UNION**
```sql
-- Good - single query, efficient
SELECT 
    department,
    SUM(CASE WHEN status = 'Active' THEN 1 ELSE 0 END) AS active_count,
    SUM(CASE WHEN status = 'Inactive' THEN 1 ELSE 0 END) AS inactive_count,
    COUNT(*) AS total_count
FROM employees
GROUP BY department;

-- Avoid - multiple queries with UNION
SELECT department, 'Active', COUNT(*)
FROM employees
WHERE status = 'Active'
GROUP BY department
UNION ALL
SELECT department, 'Inactive', COUNT(*)
FROM employees
WHERE status = 'Inactive'
GROUP BY department;
```

✅ **Tip 3: Filter with WHERE Before GROUP BY for Performance**
```sql
-- Good - reduce rows before grouping
SELECT 
    department,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= '2020-01-01'
GROUP BY department;

-- Less efficient - group then filter
SELECT 
    department,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING hire_date >= '2020-01-01';
```

✅ **Tip 4: Handle Aggregates with DISTINCT Carefully**
```sql
-- Good - clear what's being counted
SELECT 
    department,
    COUNT(DISTINCT employee_id) AS unique_employees,
    COUNT(*) AS total_records
FROM salary_history
GROUP BY department;
```

### Examples

```sql
-- Example 1: Sales summary by region
SELECT 
    region,
    COUNT(DISTINCT customer_id) AS unique_customers,
    COUNT(*) AS total_orders,
    SUM(order_amount) AS total_revenue,
    AVG(order_amount) AS avg_order_value,
    MIN(order_amount) AS min_order,
    MAX(order_amount) AS max_order
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY region
ORDER BY total_revenue DESC;

-- Example 2: Employee salary analysis with filtering
SELECT 
    department,
    job_title,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees
WHERE hire_date >= '2020-01-01'
GROUP BY department, job_title
HAVING COUNT(*) >= 2
ORDER BY department, avg_salary DESC;

-- Example 3: Conditional aggregates
SELECT 
    date_trunc('month', order_date)::date AS month,
    COUNT(*) AS total_orders,
    SUM(CASE WHEN status = 'Completed' THEN 1 ELSE 0 END) AS completed_orders,
    SUM(CASE WHEN status = 'Pending' THEN 1 ELSE 0 END) AS pending_orders,
    SUM(CASE WHEN status = 'Completed' THEN order_amount ELSE 0 END) AS revenue
FROM orders
GROUP BY date_trunc('month', order_date)
ORDER BY month DESC;

-- Example 4: Top performers
SELECT 
    employee_id,
    employee_name,
    department,
    COUNT(*) AS sales_count,
    SUM(amount) AS total_sales,
    AVG(amount) AS avg_sale
FROM sales
GROUP BY employee_id, employee_name, department
HAVING SUM(amount) > 50000
ORDER BY total_sales DESC
LIMIT 10;

-- Example 5: Data quality checks with aggregates
SELECT 
    category,
    COUNT(*) AS total_products,
    COUNT(CASE WHEN price IS NULL THEN 1 END) AS missing_prices,
    COUNT(CASE WHEN description IS NULL THEN 1 END) AS missing_descriptions,
    SUM(CASE WHEN price IS NULL OR description IS NULL THEN 1 ELSE 0 END) AS incomplete_records
FROM products
GROUP BY category
HAVING SUM(CASE WHEN price IS NULL OR description IS NULL THEN 1 ELSE 0 END) > 0;
```

### Interview Questions

**Q1: What's the difference between WHERE and HAVING?**
A: WHERE filters individual rows before grouping and applies to columns. HAVING filters groups after GROUP BY and works with aggregate functions. WHERE is more efficient because it reduces rows before aggregation.

**Q2: Why can't you use COUNT(*) in WHERE?**
A: WHERE clause is evaluated before GROUP BY, so aggregate functions aren't available yet. Use HAVING after GROUP BY for aggregate conditions.

**Q3: What happens to NULL values in GROUP BY?**
A: NULL is treated as a distinct group value. All NULL values are grouped together separately from other groups.

**Q4: How does COUNT(*) differ from COUNT(column)?**
A: COUNT(*) counts all rows including NULLs. COUNT(column) counts only non-NULL values in that column. This is important for understanding actual row counts vs non-NULL value counts.

---

## 8. JOIN Operations

### Overview
JOINs combine rows from multiple tables based on related columns. Different JOIN types return different result sets.

### INNER JOIN

#### Definition
Returns rows that have matching values in both tables.

#### Syntax

```sql
SELECT columns
FROM table1
INNER JOIN table2
ON table1.key_column = table2.key_column;

-- Alternative syntax
SELECT columns
FROM table1
JOIN table2
ON table1.key_column = table2.key_column;
```

#### Visualization
```
Table1: A=1,2,3
Table2: B=2,3,4
INNER JOIN result: 2,3 (common values)
```

#### Examples

```sql
-- Basic INNER JOIN
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    o.order_amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;

-- Multiple INNER JOINs
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id;

-- INNER JOIN with conditions
SELECT 
    e.employee_id,
    e.employee_name,
    d.department_name,
    e.salary
FROM employees e
INNER JOIN departments d ON e.department_id = d.dept_id
WHERE e.salary > 50000
AND d.department_name IN ('Sales', 'IT');
```

#### Dialect Variations

All SQL dialects support INNER JOIN identically.

```sql
-- All databases use the same syntax
INNER JOIN table2 ON condition
JOIN table2 ON condition  -- INNER is default
```

### LEFT JOIN (LEFT OUTER JOIN)

#### Definition
Returns all rows from the left table and matching rows from the right table. Non-matching rows have NULL in right table columns.

#### Syntax

```sql
SELECT columns
FROM table1
LEFT JOIN table2
ON table1.key_column = table2.key_column;

-- Alternative (same)
SELECT columns
FROM table1
LEFT OUTER JOIN table2
ON table1.key_column = table2.key_column;
```

#### Visualization
```
Table1: A=1,2,3
Table2: B=2,3,4
LEFT JOIN result: 1(null),2,3 (all from left)
```

#### Examples

```sql
-- All customers with their orders (or NULL if no orders)
SELECT 
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

-- Find customers with no orders
SELECT 
    c.customer_id,
    c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;

-- LEFT JOIN with aggregates
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(o.order_amount), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

### RIGHT JOIN (RIGHT OUTER JOIN)

#### Definition
Returns all rows from the right table and matching rows from the left table. Opposite of LEFT JOIN.

#### Syntax

```sql
SELECT columns
FROM table1
RIGHT JOIN table2
ON table1.key_column = table2.key_column;
```

#### Examples

```sql
-- All products with their orders (or NULL if not ordered)
SELECT 
    p.product_id,
    p.product_name,
    o.order_id,
    o.order_amount
FROM orders o
RIGHT JOIN products p ON o.product_id = p.product_id;

-- Find products never ordered
SELECT 
    p.product_id,
    p.product_name
FROM orders o
RIGHT JOIN products p ON o.product_id = p.product_id
WHERE o.order_id IS NULL;
```

#### Dialect Variations

```sql
-- Most databases support RIGHT JOIN
-- RIGHT JOIN is equivalent to LEFT JOIN with tables reversed

-- PostgreSQL, MySQL, SQL Server, Oracle all support it
-- SQLite does NOT support RIGHT JOIN
-- In SQLite, use LEFT JOIN with tables reversed

-- SQLite workaround
SELECT *
FROM products p
LEFT JOIN orders o ON o.product_id = p.product_id;
```

### FULL OUTER JOIN

#### Definition
Returns all rows from both tables. Rows without matches show NULL in unmatched columns.

#### Syntax

```sql
SELECT columns
FROM table1
FULL OUTER JOIN table2
ON table1.key_column = table2.key_column;

-- Alternative
SELECT columns
FROM table1
FULL JOIN table2
ON table1.key_column = table2.key_column;
```

#### Visualization
```
Table1: A=1,2,3
Table2: B=2,3,4
FULL OUTER JOIN: 1(null),2,3,null 4
```

#### Examples

```sql
-- All customers and all orders
SELECT 
    COALESCE(c.customer_id, o.customer_id) AS customer_id,
    c.customer_name,
    o.order_id,
    o.order_date
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;

-- Complex FULL OUTER JOIN
SELECT 
    COALESCE(e.employee_id, s.employee_id) AS emp_id,
    e.employee_name,
    COALESCE(e.salary, 0) AS salary,
    COALESCE(s.total_sales, 0) AS total_sales
FROM employees e
FULL OUTER JOIN sales_summary s ON e.employee_id = s.employee_id;
```

#### Dialect Variations

```sql
-- PostgreSQL
SELECT * FROM table1 FULL OUTER JOIN table2 ON condition;

-- MySQL (NOT supported - use UNION alternative)
SELECT * FROM table1 LEFT JOIN table2 ON condition
UNION
SELECT * FROM table1 RIGHT JOIN table2 ON condition;

-- SQL Server
SELECT * FROM table1 FULL OUTER JOIN table2 ON condition;

-- SQLite (NOT supported - use UNION alternative)
SELECT * FROM table1 LEFT JOIN table2 ON condition
UNION
SELECT * FROM table1 RIGHT JOIN table2 ON condition;

-- Oracle
SELECT * FROM table1 FULL OUTER JOIN table2 ON condition;
```

### CROSS JOIN

#### Definition
Returns the Cartesian product - every row from table1 matched with every row from table2.

#### Syntax

```sql
SELECT columns
FROM table1
CROSS JOIN table2;

-- Alternative syntax
SELECT columns
FROM table1, table2;
```

#### Warning
CROSS JOIN can create extremely large result sets!

```sql
-- If table1 has 1000 rows and table2 has 100 rows
-- CROSS JOIN returns 100,000 rows

SELECT COUNT(*)
FROM customers CROSS JOIN products;  -- Be careful!
```

#### Valid Use Cases

```sql
-- Generate combinations
SELECT 
    d.day,
    h.hour,
    CONCAT(d.day, ' ', h.hour) AS time_slot
FROM days d
CROSS JOIN hours h;

-- Generate sequences
SELECT 
    numbers.n
FROM (
    SELECT 1 AS n UNION ALL SELECT 2 UNION ALL SELECT 3
) numbers
CROSS JOIN periods;
```

### Self JOIN

#### Definition
Joins a table to itself to compare rows within the same table.

#### Examples

```sql
-- Find employees and their managers
SELECT 
    e.employee_id,
    e.employee_name,
    m.employee_id AS manager_id,
    m.employee_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;

-- Find products in the same category
SELECT 
    p1.product_id,
    p1.product_name,
    p2.product_id AS related_product_id,
    p2.product_name AS related_product_name
FROM products p1
INNER JOIN products p2 ON p1.category = p2.category
WHERE p1.product_id < p2.product_id;

-- Find duplicate values
SELECT 
    t1.id,
    t1.email,
    t2.id AS duplicate_id
FROM users t1
INNER JOIN users t2 ON t1.email = t2.email AND t1.id < t2.id;
```

### Common Mistakes

❌ **Mistake 1: Wrong JOIN Type**
```sql
-- WRONG - misses customers with no orders
SELECT *
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- CORRECT - shows all customers
SELECT *
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

❌ **Mistake 2: Ambiguous Column Names**
```sql
-- WRONG - unclear which table's ID
SELECT id, customer_name, order_amount
FROM customers
JOIN orders ON customers.id = orders.customer_id;

-- CORRECT - specify table
SELECT 
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

❌ **Mistake 3: JOIN Condition in WHERE**
```sql
-- WRONG - if using LEFT JOIN, WHERE nullifies it
SELECT *
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NOT NULL;  -- This defeats LEFT JOIN purpose

-- CORRECT - put condition in ON clause
SELECT *
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
  AND o.order_date >= '2024-01-01';
```

❌ **Mistake 4: Too Many JOINs**
```sql
-- SLOW - deep JOIN chain
SELECT *
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
JOIN suppliers s ON p.supplier_id = s.supplier_id
JOIN locations l ON s.location_id = l.location_id;

-- Better - break into multiple queries
-- OR use subqueries to reduce intermediate data
```

### Production Tips

✅ **Tip 1: Always Qualify Column Names**
```sql
-- Good - clear which table
SELECT 
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

✅ **Tip 2: Use Table Aliases**
```sql
-- Good - more readable
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id

-- Verbose
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id
```

✅ **Tip 3: Check for NULL Issues in LEFT JOINs**
```sql
-- Remember: unmatched rows have NULL in join columns
SELECT *
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE COALESCE(o.order_amount, 0) > 1000;

-- Or find unmatched specifically
WHERE o.order_id IS NULL;
```

✅ **Tip 4: Order JOINs by Size**
```sql
-- Good - join smaller result sets first (usually more efficient)
SELECT *
FROM huge_table h
JOIN small_table s ON h.id = s.id
JOIN medium_table m ON h.id = m.id;
```

### Examples

```sql
-- Example 1: Multi-table JOIN
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    c.country,
    SUM(oi.quantity * oi.unit_price) AS order_total,
    COUNT(oi.order_item_id) AS item_count
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_date BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY o.order_id, o.order_date, c.customer_name, c.country
ORDER BY order_total DESC;

-- Example 2: LEFT JOIN with aggregation
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(o.order_amount), 0) AS total_spent,
    MAX(o.order_date) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY total_spent DESC;

-- Example 3: Self JOIN
SELECT 
    e.employee_id,
    e.first_name || ' ' || e.last_name AS employee_name,
    m.first_name || ' ' || m.last_name AS manager_name,
    e.salary,
    ROUND((e.salary - m.salary), 2) AS salary_difference
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
ORDER BY employee_name;

-- Example 4: Complex JOIN with conditions
SELECT 
    p.product_id,
    p.product_name,
    c.category_name,
    COUNT(oi.order_item_id) AS times_ordered,
    SUM(oi.quantity) AS total_quantity_sold,
    AVG(oi.unit_price) AS avg_price
FROM products p
LEFT JOIN categories c ON p.category_id = c.category_id
LEFT JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name, c.category_name
HAVING COUNT(oi.order_item_id) > 0
ORDER BY times_ordered DESC;

-- Example 5: FULL OUTER JOIN with data reconciliation
SELECT 
    COALESCE(s.sale_id, r.return_id) AS transaction_id,
    COALESCE(s.sale_date, r.return_date) AS transaction_date,
    s.sale_amount,
    r.return_amount,
    CASE 
        WHEN s.sale_id IS NULL THEN 'Return Only'
        WHEN r.return_id IS NULL THEN 'Sale Only'
        ELSE 'Sale & Return'
    END AS transaction_type
FROM sales s
FULL OUTER JOIN returns r ON s.sale_id = r.original_sale_id
ORDER BY transaction_date;
```

### Interview Questions

**Q1: What's the difference between INNER JOIN and LEFT JOIN?**
A: INNER JOIN returns only rows with matches in both tables. LEFT JOIN returns all rows from the left table and matching rows from the right table, with NULL for unmatched right table columns.

**Q2: Why might LEFT JOIN performance differ from INNER JOIN?**
A: INNER JOIN can optimize better because it knows both sides must match. LEFT JOIN must keep all left table rows, which may prevent certain optimizations and index usage.

**Q3: How do you find rows in table1 that have no match in table2?**
A: Use LEFT JOIN and WHERE table2.key IS NULL. The NULL indicates no matching row in the right table.

**Q4: What's the difference between joining in ON clause vs WHERE clause?**
A: In INNER JOIN, they're equivalent. In LEFT/RIGHT/FULL OUTER JOINs, conditions in ON affect which rows join, while conditions in WHERE filter after joining. Use ON for join logic, WHERE for filtering.

---

## 9. Set Operations

### Overview
Set operations combine results from multiple queries. They treat query results as sets and apply set theory operations.

### UNION Operator

#### Definition
Combines results from two queries and removes duplicates.

#### Syntax

```sql
SELECT columns FROM table1
UNION
SELECT columns FROM table2;
```

#### Requirements
- Same number of columns
- Corresponding columns must have compatible data types
- Column names from first query are used in result

#### Examples

```sql
-- Combine two customer lists
SELECT customer_id, customer_name, 'Active' AS status
FROM active_customers
UNION
SELECT customer_id, customer_name, 'Inactive' AS status
FROM inactive_customers;

-- Combine different queries
SELECT employee_id, employee_name
FROM employees
WHERE department = 'Sales'
UNION
SELECT contractor_id, contractor_name
FROM contractors
WHERE status = 'Active';
```

#### UNION ALL

Includes duplicates (more efficient than UNION as no deduplication needed).

```sql
SELECT * FROM table1
UNION ALL
SELECT * FROM table2;
```

#### Examples with UNION ALL

```sql
-- Combine sales data with returns (keeping all)
SELECT 
    'Sale' AS transaction_type,
    order_id,
    customer_id,
    amount
FROM sales
UNION ALL
SELECT 
    'Return' AS transaction_type,
    return_id,
    customer_id,
    -amount
FROM returns;
```

#### Common Mistakes

❌ **Mistake 1: Mismatched Column Types**
```sql
-- WRONG - character vs number
SELECT id, name FROM customers
UNION
SELECT emp_id, salary FROM employees;

-- CORRECT - same types
SELECT id, CAST(id AS VARCHAR) FROM customers
UNION
SELECT emp_id, salary FROM employees;
```

❌ **Mistake 2: Different Number of Columns**
```sql
-- WRONG
SELECT customer_id, customer_name FROM customers
UNION
SELECT product_id, product_name, category FROM products;

-- CORRECT - same number
SELECT customer_id, customer_name, NULL FROM customers
UNION
SELECT product_id, product_name, category FROM products;
```

❌ **Mistake 3: Performance - Using UNION When UNION ALL Sufficient**
```sql
-- SLOWER - deduplicates unnecessarily
SELECT product_id FROM sales
UNION
SELECT product_id FROM returns;

-- FASTER - if you're certain there are no duplicates
SELECT DISTINCT product_id FROM sales
UNION ALL
SELECT DISTINCT product_id FROM returns;
```

### INTERSECT Operator

#### Definition
Returns only rows that appear in both query results.

#### Syntax

```sql
SELECT columns FROM table1
INTERSECT
SELECT columns FROM table2;
```

#### Examples

```sql
-- Find customers who are also suppliers
SELECT customer_id, company_name FROM customers
INTERSECT
SELECT supplier_id, company_name FROM suppliers;

-- Find products ordered by all top customers
SELECT product_id FROM order_items
WHERE order_id IN (SELECT order_id FROM orders WHERE customer_id = 1)
INTERSECT
SELECT product_id FROM order_items
WHERE order_id IN (SELECT order_id FROM orders WHERE customer_id = 2);
```

#### Dialect Variations

```sql
-- PostgreSQL, SQL Server, Oracle support INTERSECT
-- MySQL does NOT support INTERSECT natively
-- SQLite supports INTERSECT

-- MySQL alternative using JOIN
SELECT DISTINCT t1.column
FROM table1 t1
INNER JOIN table2 t2 ON t1.column = t2.column;
```

### EXCEPT Operator (or MINUS)

#### Definition
Returns rows from first query that don't appear in second query.

#### Syntax

```sql
SELECT columns FROM table1
EXCEPT
SELECT columns FROM table2;

-- Oracle syntax
SELECT columns FROM table1
MINUS
SELECT columns FROM table2;
```

#### Examples

```sql
-- Find customers who haven't placed orders
SELECT customer_id FROM customers
EXCEPT
SELECT DISTINCT customer_id FROM orders;

-- Find products never ordered
SELECT product_id FROM products
EXCEPT
SELECT DISTINCT product_id FROM order_items;
```

#### Dialect Variations

```sql
-- PostgreSQL, SQL Server support EXCEPT
-- Oracle uses MINUS instead
-- MySQL doesn't support EXCEPT (use LEFT JOIN NOT IN)
-- SQLite supports EXCEPT

-- MySQL alternative
SELECT DISTINCT p.product_id
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE oi.product_id IS NULL;
```

### Common Mistakes

❌ **Mistake 1: Wrong Operation for the Task**
```sql
-- WRONG - using INTERSECT when you want matching pairs
SELECT * FROM customers
INTERSECT
SELECT * FROM orders;

-- These have incompatible columns, should be JOIN
SELECT c.* FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

❌ **Mistake 2: Performance with Set Operations**
```sql
-- SLOW - materializes entire result sets
SELECT * FROM huge_table1
UNION
SELECT * FROM huge_table2;

-- BETTER - filter first
SELECT * FROM huge_table1 WHERE criteria
UNION
SELECT * FROM huge_table2 WHERE criteria;
```

❌ **Mistake 3: ORDER BY with Set Operations**
```sql
-- WRONG - ORDER BY applies to entire result
SELECT * FROM table1
ORDER BY column1
UNION
SELECT * FROM table2;

-- CORRECT
SELECT * FROM table1
UNION
SELECT * FROM table2
ORDER BY column1;

-- Or group with parentheses
(SELECT * FROM table1 ORDER BY column1)
UNION
(SELECT * FROM table2 ORDER BY column2);
```

### Production Tips

✅ **Tip 1: Use UNION ALL Unless Deduplication Needed**
```sql
-- Good - faster when duplicates acceptable
SELECT * FROM table1
UNION ALL
SELECT * FROM table2;

-- UNION is slower due to deduplication
SELECT * FROM table1
UNION
SELECT * FROM table2;
```

✅ **Tip 2: Consider JOINs vs Set Operations**
```sql
-- UNION for combining different entity types
SELECT customer_id, customer_name FROM customers
UNION
SELECT supplier_id, supplier_name FROM suppliers;

-- JOIN for related data
SELECT c.customer_id, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

✅ **Tip 3: Specify Distinct Column Names**
```sql
-- Good - clear result set structure
SELECT 
    customer_id,
    customer_name,
    'Customer' AS entity_type
FROM customers
UNION
SELECT 
    supplier_id,
    supplier_name,
    'Supplier' AS entity_type
FROM suppliers;
```

### Examples

```sql
-- Example 1: Combine entity types
SELECT 
    contact_id,
    contact_name,
    email,
    phone,
    'Customer' AS entity_type
FROM customers
UNION
SELECT 
    contact_id,
    contact_name,
    email,
    phone,
    'Supplier' AS entity_type
FROM suppliers
ORDER BY entity_type, contact_name;

-- Example 2: Complete transaction history
SELECT 
    transaction_id,
    transaction_date,
    CONCAT('Sale: ', amount) AS description,
    amount
FROM sales
UNION ALL
SELECT 
    transaction_id,
    transaction_date,
    CONCAT('Return: -', amount) AS description,
    -amount
FROM returns
ORDER BY transaction_date DESC;

-- Example 3: Find common values
SELECT DISTINCT product_id
FROM order_items
WHERE quantity > 5
INTERSECT
SELECT DISTINCT product_id
FROM inventory
WHERE stock > 100
ORDER BY product_id;

-- Example 4: Except for missing data
SELECT customer_id
FROM customers
EXCEPT
SELECT DISTINCT customer_id
FROM orders
WHERE order_date >= '2024-01-01';

-- Example 5: Multiple set operations
SELECT product_id FROM category_A_products
UNION
SELECT product_id FROM category_B_products
EXCEPT
SELECT product_id FROM discontinued_products
ORDER BY product_id;
```

### Interview Questions

**Q1: What's the difference between UNION and UNION ALL?**
A: UNION removes duplicate rows from the result set, while UNION ALL keeps all rows including duplicates. UNION ALL is faster because it doesn't need to perform deduplication, so use it when duplicates are acceptable.

**Q2: Why would you use INTERSECT instead of INNER JOIN?**
A: INTERSECT is used to find rows that exist in both result sets (comparing different queries or tables). INNER JOIN combines related data from tables. They solve different problems—INTERSECT is for "what's common," JOIN is for "how are they related."

**Q3: How would you perform a set operation in MySQL which doesn't support INTERSECT?**
A: Use INNER JOIN instead. For INTERSECT, join tables on the common column. For EXCEPT, use LEFT JOIN with WHERE column IS NULL.

**Q4: What happens if you have a UNION with different data types?**
A: SQL will attempt implicit type conversion. It's better to explicitly CAST columns to the same type to avoid unexpected results or errors.

---

## Production Best Practices

### Query Optimization

#### 1. Index Strategy
```sql
-- Good - queries on indexed columns
SELECT * FROM customers WHERE customer_id = 123;     -- Uses index
SELECT * FROM orders WHERE order_date = '2024-01-01'; -- Uses index

-- Avoid - prevents index usage
SELECT * FROM customers WHERE UPPER(name) = 'JOHN';   -- Function on column
SELECT * FROM orders WHERE YEAR(order_date) = 2024;   -- Function on column
```

#### 2. Select Only Needed Columns
```sql
-- Good
SELECT customer_id, customer_name, email
FROM customers;

-- Avoid - unnecessary data transfer
SELECT * FROM customers;
```

#### 3. Filter Early
```sql
-- Good - filter before aggregation
SELECT department, AVG(salary)
FROM employees
WHERE hire_date >= '2020-01-01'
GROUP BY department;

-- Less efficient - aggregate then filter
SELECT department, AVG(salary) AS avg_sal
FROM employees
GROUP BY department
HAVING hire_date >= '2020-01-01';
```

### Data Type Selection

#### Choose Appropriate Types
```sql
-- Good
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(255),
    price DECIMAL(10, 2),
    created_at TIMESTAMP,
    is_active BOOLEAN
);

-- Avoid
CREATE TABLE products (
    product_id VARCHAR(255),
    product_name TEXT,
    price FLOAT,
    created_at VARCHAR(50),
    is_active VARCHAR(10)
);
```

### NULL Handling

#### Default Values
```sql
-- Good - provide defaults
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(254) NOT NULL,
    phone VARCHAR(20) NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Use COALESCE
SELECT 
    customer_id,
    COALESCE(phone, 'No phone') AS contact_phone
FROM customers;
```

### Naming Conventions

#### Table and Column Names
```sql
-- Good - clear, descriptive, consistent
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(254),
    created_date DATE
);

-- Avoid - ambiguous or inconsistent
CREATE TABLE cust (
    id INT,
    fname VARCHAR(50),
    lname VARCHAR(50),
    em VARCHAR(254),
    cr_dt DATE
);
```

### Documentation

```sql
-- Good - comments document intent
-- Retrieves active customers from the last 30 days
-- ordered by spending to identify VIPs
SELECT 
    c.customer_id,
    c.customer_name,
    SUM(o.order_amount) AS total_spent
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE c.is_active = TRUE
  AND o.order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
GROUP BY c.customer_id, c.customer_name
ORDER BY total_spent DESC;
```

---

## Interview Questions

### Comprehensive Interview Questions

**Q1: Explain the order of execution in a SQL query.**
A: 
1. FROM (identify tables)
2. WHERE (filter rows)
3. GROUP BY (group rows)
4. HAVING (filter groups)
5. SELECT (select columns)
6. ORDER BY (sort results)
7. LIMIT/OFFSET (limit result set)

**Q2: What's the difference between COUNT(*), COUNT(column), and COUNT(DISTINCT column)?**
A:
- COUNT(*) counts all rows including those with NULL values
- COUNT(column) counts only non-NULL values in that column
- COUNT(DISTINCT column) counts unique non-NULL values

**Q3: How do you handle NULL values in comparisons?**
A: Use IS NULL or IS NOT NULL. Regular comparison operators (=, <>, <, >) return NULL when either operand is NULL because NULL represents an unknown value.

**Q4: What are the different types of JOINs and when would you use each?**
A:
- INNER JOIN: Only matching rows (default)
- LEFT JOIN: All left rows + matching right rows
- RIGHT JOIN: All right rows + matching left rows
- FULL OUTER JOIN: All rows from both tables
- CROSS JOIN: Every combination (Cartesian product)

**Q5: How do you find duplicate records in a table?**
A:
```sql
SELECT column, COUNT(*) as count
FROM table
GROUP BY column
HAVING COUNT(*) > 1;
```

**Q6: What's the performance impact of functions in WHERE clauses?**
A: Functions on indexed columns prevent index usage, forcing full table scans. Use functions sparingly and consider moving logic to the application layer if performance-critical.

**Q7: Explain the difference between UNION and UNION ALL.**
A: UNION removes duplicates (slower), UNION ALL keeps duplicates (faster). Use UNION ALL when you're certain there are no duplicates or duplicates are acceptable.

**Q8: How do you optimize a slow query with multiple JOINs?**
A:
- Check indexes on JOIN columns
- Filter with WHERE before JOINs
- Reduce rows before expensive operations
- Consider breaking into multiple queries
- Check query execution plan
- Denormalize if appropriate for specific scenarios

**Q9: What's a primary key vs unique key vs index?**
A:
- Primary Key: unique identifier, cannot be NULL, one per table
- Unique Key: unique values but can be NULL, multiple per table
- Index: data structure for fast lookups, not necessarily unique

**Q10: How do you write a query to find the nth highest salary?**
A:
```sql
-- SQL Server / PostgreSQL / MySQL 8.0+
SELECT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET n-1;

-- Generic approach
SELECT DISTINCT salary
FROM employees e1
WHERE (SELECT COUNT(DISTINCT salary) 
       FROM employees e2 
       WHERE e2.salary > e1.salary) = n-1;
```

---

## Conclusion

This handbook covers the fundamental SQL concepts needed for data engineering. Master these basics before moving to advanced topics like window functions, CTEs, and performance optimization. Practice regularly with real datasets to internalize these concepts.

**Key Takeaways:**
1. ✅ Always use appropriate data types
2. ✅ Handle NULL values explicitly
3. ✅ Choose the right JOIN type
4. ✅ Use aggregates with GROUP BY correctly
5. ✅ Optimize queries with indexes and WHERE filters
6. ✅ Write readable code with clear naming and aliases
7. ✅ Understand the execution order of SQL clauses
8. ✅ Test queries on sample data before production

---

**Document Version**: 1.0
**Last Updated**: 2024
**Author**: Data Engineering Team

