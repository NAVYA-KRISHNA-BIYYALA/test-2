# SQL Phase 2: Advanced Concepts Handbook
## Master Window Functions, CTEs, and Data Manipulation

---

## 📑 Table of Contents

### Window Functions
1. [Window Functions Overview](#window-functions-overview)
2. [Ranking Functions](#ranking-functions)
   - [ROW_NUMBER](#row_number)
   - [RANK](#rank)
   - [DENSE_RANK](#dense_rank)
3. [LAG and LEAD Functions](#lag-and-lead-functions)
4. [Window Aggregates](#window-aggregates)
5. [Value Functions](#value-functions)
6. [Distribution Functions](#distribution-functions)
7. [Frame Clauses](#frame-clauses)

### Subqueries & CTEs
8. [Subqueries Overview](#subqueries-overview)
9. [Scalar Subqueries](#scalar-subqueries)
10. [Inline View Subqueries](#inline-view-subqueries)
11. [Correlated Subqueries](#correlated-subqueries)
12. [CTEs (WITH Clause)](#ctes-with-clause)
13. [Recursive CTEs](#recursive-ctes)
14. [EXISTS vs IN](#exists-vs-in)

### String & Date Manipulation
15. [String Functions](#string-functions)
16. [Date/Time Functions](#datetime-functions)
17. [CASE WHEN Expressions](#case-when-expressions)
18. [Type Casting](#type-casting)
19. [Production Best Practices](#production-best-practices)
20. [Interview Questions](#interview-questions)

---

## Window Functions Overview

### What Are Window Functions?

Window functions perform calculations across rows related to the current row without collapsing the result set. They operate on a "window" of rows defined by the OVER clause.

### Basic Syntax

```sql
function_name(column) OVER (
    [PARTITION BY column1, column2]
    [ORDER BY column3 [ASC|DESC]]
    [ROWS|RANGE frame_specification]
)
```

### Key Components

```sql
PARTITION BY    -- Divides rows into groups (optional)
ORDER BY        -- Defines order within partition (usually required)
ROWS/RANGE      -- Defines frame (optional, used with aggregates)
```

### Simple Example

```sql
SELECT 
    employee_id,
    salary,
    AVG(salary) OVER () AS avg_salary_all,
    AVG(salary) OVER (PARTITION BY department) AS avg_salary_dept
FROM employees;
```

---

## Ranking Functions

### ROW_NUMBER

#### Definition
Assigns a unique sequential integer to each row within a partition.

#### Standard Syntax

```sql
ROW_NUMBER() OVER (
    [PARTITION BY column1, column2, ...]
    ORDER BY column3 [ASC|DESC]
)
```

#### Key Characteristics
- Always produces unique numbers (1, 2, 3, 4...)
- Restarts at 1 for each partition
- Requires ORDER BY clause
- Never skips numbers

#### Dialect Variations

All major SQL dialects support ROW_NUMBER() identically.

```sql
-- All databases use the same syntax
SELECT 
    employee_id,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank
FROM employees;
```

#### Use Cases

**Case 1: Ranking Employees by Salary**
```sql
SELECT 
    employee_id,
    employee_name,
    salary,
    department,
    ROW_NUMBER() OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS salary_rank_in_dept
FROM employees;
```

**Case 2: Find Top N Per Group**
```sql
-- Get top 3 products by sales per category
WITH ranked_products AS (
    SELECT 
        product_id,
        product_name,
        category,
        total_sales,
        ROW_NUMBER() OVER (
            PARTITION BY category 
            ORDER BY total_sales DESC
        ) AS rank
    FROM products
)
SELECT *
FROM ranked_products
WHERE rank <= 3;
```

**Case 3: Assign Unique Row Numbers**
```sql
SELECT 
    ROW_NUMBER() OVER (ORDER BY order_date) AS order_sequence,
    order_id,
    customer_id,
    order_date,
    order_amount
FROM orders;
```

**Case 4: Pagination**
```sql
WITH numbered_rows AS (
    SELECT 
        product_id,
        product_name,
        price,
        ROW_NUMBER() OVER (ORDER BY product_name) AS row_num
    FROM products
)
SELECT *
FROM numbered_rows
WHERE row_num BETWEEN 11 AND 20;  -- Page 2, 10 items per page
```

#### Common Mistakes

❌ **Mistake 1: ROW_NUMBER Without ORDER BY**
```sql
-- WRONG - results are unpredictable
SELECT 
    employee_id,
    ROW_NUMBER() OVER (PARTITION BY department) AS rank
FROM employees;

-- CORRECT - specify order
SELECT 
    employee_id,
    ROW_NUMBER() OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS rank
FROM employees;
```

❌ **Mistake 2: Expecting ROW_NUMBER to Handle Ties**
```sql
-- ROW_NUMBER doesn't skip for ties
SELECT 
    employee_id,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary) AS rank
FROM employees;
-- If two employees have same salary, they get 2 and 3, not 1 and 1

-- Use RANK or DENSE_RANK for ties
SELECT 
    employee_id,
    salary,
    RANK() OVER (ORDER BY salary) AS rank
FROM employees;
```

❌ **Mistake 3: Forgetting PARTITION BY Scope**
```sql
-- WRONG - row number resets for entire table
SELECT 
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS overall_rank,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- Both are correct but show different things
```

#### Production Tips

✅ **Tip 1: Use for De-duplication**
```sql
-- Good - remove duplicates keeping first occurrence
WITH ranked AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (
            PARTITION BY email 
            ORDER BY created_date ASC
        ) AS rank
    FROM users
)
SELECT * FROM ranked WHERE rank = 1;
```

✅ **Tip 2: Combine with Filtering**
```sql
-- Good - find top performers
SELECT 
    employee_id,
    employee_name,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS rank
FROM employees
HAVING rank <= 5;
```

✅ **Tip 3: Use for Pagination Efficiently**
```sql
-- Good - for web pagination
WITH paged_results AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (ORDER BY created_date DESC) AS row_num
    FROM articles
)
SELECT * FROM paged_results
WHERE row_num BETWEEN @start_row AND @end_row;
```

### RANK

#### Definition
Assigns a rank to each row with gaps for ties. Ties receive the same rank.

#### Standard Syntax

```sql
RANK() OVER (
    [PARTITION BY column1, column2, ...]
    ORDER BY column3 [ASC|DESC]
)
```

#### Key Characteristics
- Ties receive the same rank
- Next rank skips (1, 1, 3, 4...)
- Requires ORDER BY clause
- Numbers can repeat

#### Examples

```sql
-- Rank employees by salary (ties handled)
SELECT 
    employee_id,
    employee_name,
    salary,
    RANK() OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;

-- Results might be:
-- emp_id, name, salary, rank
-- 1, John, 50000, 1
-- 2, Jane, 50000, 1
-- 3, Bob, 48000, 3      <- skips 2
-- 4, Alice, 48000, 3
-- 5, Charlie, 45000, 5  <- skips 4
```

#### Use Cases

**Case 1: Finding Top Performers with Ties**
```sql
SELECT 
    employee_id,
    employee_name,
    performance_score,
    RANK() OVER (ORDER BY performance_score DESC) AS performance_rank
FROM employees
WHERE RANK() OVER (ORDER BY performance_score DESC) <= 3;

-- This won't work - use CTE instead:
WITH ranked_employees AS (
    SELECT 
        employee_id,
        employee_name,
        performance_score,
        RANK() OVER (ORDER BY performance_score DESC) AS performance_rank
    FROM employees
)
SELECT * FROM ranked_employees WHERE performance_rank <= 3;
```

**Case 2: Medal Distribution (Gold, Silver, Bronze)**
```sql
SELECT 
    athlete_name,
    sport,
    points,
    RANK() OVER (
        PARTITION BY sport 
        ORDER BY points DESC
    ) AS medal_rank,
    CASE 
        WHEN RANK() OVER (PARTITION BY sport ORDER BY points DESC) = 1 THEN 'Gold'
        WHEN RANK() OVER (PARTITION BY sport ORDER BY points DESC) = 2 THEN 'Silver'
        WHEN RANK() OVER (PARTITION BY sport ORDER BY points DESC) = 3 THEN 'Bronze'
        ELSE NULL
    END AS medal
FROM athletes;
```

#### Common Mistakes

❌ **Mistake 1: Confusing RANK with ROW_NUMBER**
```sql
-- These are different!
ROW_NUMBER() OVER (ORDER BY salary)  -- 1, 2, 3, 4, 5
RANK() OVER (ORDER BY salary)        -- 1, 1, 3, 4, 5 (if ties exist)
DENSE_RANK() OVER (ORDER BY salary)  -- 1, 1, 2, 3, 4
```

❌ **Mistake 2: Using RANK in WHERE**
```sql
-- WRONG - window functions evaluated after WHERE
SELECT *
FROM employees
WHERE RANK() OVER (ORDER BY salary DESC) <= 3;

-- CORRECT - use CTE
WITH ranked AS (
    SELECT 
        *,
        RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT * FROM ranked WHERE rank <= 3;
```

### DENSE_RANK

#### Definition
Assigns ranks to rows with no gaps. Ties receive the same rank, next rank continues without skipping.

#### Standard Syntax

```sql
DENSE_RANK() OVER (
    [PARTITION BY column1, column2, ...]
    ORDER BY column3 [ASC|DESC]
)
```

#### Key Characteristics
- Ties receive the same rank
- No gaps in ranking (1, 1, 2, 3...)
- Requires ORDER BY clause
- Most intuitive for human-readable rankings

#### Comparison: ROW_NUMBER vs RANK vs DENSE_RANK

```sql
SELECT 
    employee_id,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- Results:
-- emp_id, salary, row_num, rank, dense_rank
-- 1, 50000, 1, 1, 1
-- 2, 50000, 2, 1, 1       <- same rank
-- 3, 48000, 3, 3, 2       <- RANK skips, DENSE_RANK continues
-- 4, 48000, 4, 3, 2
-- 5, 45000, 5, 5, 3       <- gap in RANK
```

#### Examples

```sql
-- Rank products by sales (no gaps)
SELECT 
    product_id,
    product_name,
    total_sales,
    DENSE_RANK() OVER (ORDER BY total_sales DESC) AS sales_rank
FROM products;

-- Find products at rank 1, 2, 3 (no need to worry about ties)
WITH ranked_products AS (
    SELECT 
        product_id,
        product_name,
        total_sales,
        DENSE_RANK() OVER (ORDER BY total_sales DESC) AS sales_rank
    FROM products
)
SELECT * FROM ranked_products WHERE sales_rank <= 3;
```

#### Use Cases

**Case 1: Grade Distribution**
```sql
SELECT 
    student_id,
    student_name,
    test_score,
    DENSE_RANK() OVER (ORDER BY test_score DESC) AS score_rank,
    CASE 
        WHEN DENSE_RANK() OVER (ORDER BY test_score DESC) = 1 THEN 'Excellent'
        WHEN DENSE_RANK() OVER (ORDER BY test_score DESC) = 2 THEN 'Good'
        WHEN DENSE_RANK() OVER (ORDER BY test_score DESC) = 3 THEN 'Average'
        ELSE 'Below Average'
    END AS grade
FROM students;
```

**Case 2: Performance Tiers**
```sql
SELECT 
    employee_id,
    employee_name,
    department,
    performance_score,
    DENSE_RANK() OVER (
        PARTITION BY department 
        ORDER BY performance_score DESC
    ) AS performance_tier
FROM employees
WHERE DENSE_RANK() OVER (
    PARTITION BY department 
    ORDER BY performance_score DESC
) <= 2;  -- Top 2 performers per department
```

#### Production Tips

✅ **Tip 1: Use DENSE_RANK for Human-Readable Results**
```sql
-- Good - no confusing gaps
SELECT 
    customer_id,
    total_spent,
    DENSE_RANK() OVER (ORDER BY total_spent DESC) AS customer_rank
FROM customers;
-- Results: rank 1, 1, 2, 3, 4 (no gap at rank 3)
```

✅ **Tip 2: Combine with PARTITION BY for Complex Rankings**
```sql
-- Good - rank within groups without gaps
SELECT 
    product_id,
    product_name,
    category,
    revenue,
    DENSE_RANK() OVER (
        PARTITION BY category 
        ORDER BY revenue DESC
    ) AS category_rank
FROM products;
```

---

## LAG and LEAD Functions

### Overview
LAG and LEAD access data from previous or subsequent rows within a partition without using self-joins.

### LAG Function

#### Definition
Accesses data from the previous row in the window.

#### Standard Syntax

```sql
LAG(column [, offset] [, default_value]) OVER (
    [PARTITION BY column1, column2, ...]
    ORDER BY column3
)
```

#### Parameters
```
column          -- Column to retrieve from previous row
offset          -- How many rows back (default: 1)
default_value   -- Value if no previous row exists (default: NULL)
```

#### Examples

```sql
-- Compare salary with previous employee
SELECT 
    employee_id,
    employee_name,
    salary,
    LAG(salary) OVER (ORDER BY employee_id) AS previous_salary,
    salary - LAG(salary) OVER (ORDER BY employee_id) AS salary_increase
FROM employees;

-- Month-over-month sales
SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS previous_month_revenue,
    revenue - LAG(revenue) OVER (ORDER BY month) AS revenue_change
FROM monthly_sales;

-- With default value
SELECT 
    date,
    closing_price,
    LAG(closing_price, 1, closing_price) OVER (ORDER BY date) AS previous_price
FROM stock_prices;
```

#### Use Cases

**Case 1: Calculate Period-over-Period Growth**
```sql
SELECT 
    quarter,
    revenue,
    LAG(revenue) OVER (ORDER BY quarter) AS previous_quarter_revenue,
    ROUND(
        ((revenue - LAG(revenue) OVER (ORDER BY quarter)) 
        / LAG(revenue) OVER (ORDER BY quarter) * 100), 
        2
    ) AS growth_percentage
FROM quarterly_revenue;
```

**Case 2: Find Data Anomalies**
```sql
-- Detect unusual jumps in values
SELECT 
    date,
    value,
    LAG(value) OVER (ORDER BY date) AS previous_value,
    ABS(value - LAG(value) OVER (ORDER BY date)) AS change,
    CASE 
        WHEN ABS(value - LAG(value) OVER (ORDER BY date)) > 100 
        THEN 'Anomaly Detected'
        ELSE 'Normal'
    END AS status
FROM sensor_data;
```

**Case 3: Identify Consecutive Values**
```sql
-- Find when value changes
SELECT 
    date,
    status,
    LAG(status) OVER (ORDER BY date) AS previous_status,
    CASE 
        WHEN status <> LAG(status) OVER (ORDER BY date) 
        OR LAG(status) OVER (ORDER BY date) IS NULL
        THEN 1 
        ELSE 0 
    END AS status_changed
FROM status_log;
```

#### Dialect Variations

All major databases support LAG identically.

```sql
-- PostgreSQL, MySQL 8.0+, SQL Server, Oracle
LAG(column) OVER (ORDER BY column)
```

### LEAD Function

#### Definition
Accesses data from the next row in the window. Opposite of LAG.

#### Standard Syntax

```sql
LEAD(column [, offset] [, default_value]) OVER (
    [PARTITION BY column1, column2, ...]
    ORDER BY column3
)
```

#### Examples

```sql
-- Compare salary with next employee
SELECT 
    employee_id,
    employee_name,
    salary,
    LEAD(salary) OVER (ORDER BY salary DESC) AS next_higher_salary
FROM employees;

-- Find when customer's next purchase is
SELECT 
    customer_id,
    order_date,
    order_amount,
    LEAD(order_date) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS next_purchase_date,
    DATEDIFF(
        day,
        order_date,
        LEAD(order_date) OVER (PARTITION BY customer_id ORDER BY order_date)
    ) AS days_to_next_purchase
FROM orders;
```

#### Use Cases

**Case 1: Calculate Time Between Events**
```sql
SELECT 
    customer_id,
    event_date,
    event_type,
    LEAD(event_date) OVER (
        PARTITION BY customer_id 
        ORDER BY event_date
    ) AS next_event_date,
    DATEDIFF(
        day,
        event_date,
        LEAD(event_date) OVER (PARTITION BY customer_id ORDER BY event_date)
    ) AS days_between_events
FROM customer_events;
```

**Case 2: Detect Session End**
```sql
-- Identify when customer left (last activity)
SELECT 
    customer_id,
    activity_timestamp,
    activity_type,
    LEAD(activity_timestamp) OVER (
        PARTITION BY customer_id 
        ORDER BY activity_timestamp
    ) AS next_activity,
    CASE 
        WHEN LEAD(activity_timestamp) OVER (
            PARTITION BY customer_id 
            ORDER BY activity_timestamp
        ) IS NULL
        THEN 'Session End'
        ELSE NULL
    END AS session_status
FROM user_activity;
```

**Case 3: Forecast Next Value**
```sql
SELECT 
    month,
    actual_value,
    LEAD(actual_value) OVER (ORDER BY month) AS next_month_actual,
    LEAD(actual_value) OVER (ORDER BY month) - actual_value AS simple_forecast
FROM forecast_data;
```

#### Common Mistakes

❌ **Mistake 1: Accessing LAG/LEAD Outside Window**
```sql
-- WRONG - LAG/LEAD only works in OVER clause
SELECT 
    employee_id,
    LAG(salary)  -- Error!
FROM employees;

-- CORRECT
SELECT 
    employee_id,
    LAG(salary) OVER (ORDER BY employee_id) AS previous_salary
FROM employees;
```

❌ **Mistake 2: Not Handling NULL from LAG/LEAD**
```sql
-- WRONG - first row's LAG is NULL
SELECT 
    date,
    value,
    LAG(value) OVER (ORDER BY date) AS previous_value,
    value / LAG(value) OVER (ORDER BY date) AS ratio  -- Error for first row!
FROM data;

-- CORRECT - handle NULL
SELECT 
    date,
    value,
    LAG(value) OVER (ORDER BY date) AS previous_value,
    CASE 
        WHEN LAG(value) OVER (ORDER BY date) IS NULL THEN NULL
        ELSE value / LAG(value) OVER (ORDER BY date)
    END AS ratio
FROM data;
```

❌ **Mistake 3: Wrong Offset Direction**
```sql
-- LAG goes backward, LEAD goes forward
-- Confusing these leads to wrong results
LAG(salary, 2) OVER (ORDER BY date)   -- 2 rows BACK
LEAD(salary, 2) OVER (ORDER BY date)  -- 2 rows FORWARD
```

#### Production Tips

✅ **Tip 1: Cache Window Function Results**
```sql
-- Good - avoid recalculating the same window function
WITH sales_with_change AS (
    SELECT 
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS prev_revenue
    FROM monthly_sales
)
SELECT 
    month,
    revenue,
    prev_revenue,
    revenue - prev_revenue AS change,
    (revenue - prev_revenue) / prev_revenue * 100 AS pct_change
FROM sales_with_change;
```

✅ **Tip 2: Use Proper Offset**
```sql
-- Good - clearly specify offset
LAG(salary, 1) OVER (ORDER BY hire_date)   -- Previous row
LAG(salary, 12) OVER (ORDER BY month)      -- Same month last year

-- Avoid
LAG(salary) OVER (ORDER BY hire_date)      -- Unclear what offset is
```

---

## Window Aggregates

### Overview
Use aggregate functions within window functions to calculate running totals, moving averages, and more without collapsing rows.

### Running Totals

#### Definition
Sum of all values from start of partition to current row.

#### Syntax

```sql
SUM(column) OVER (
    [PARTITION BY column1]
    ORDER BY column2
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

#### Examples

```sql
-- Running total of sales
SELECT 
    order_date,
    order_amount,
    SUM(order_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM orders
ORDER BY order_date;

-- Running total per customer
SELECT 
    customer_id,
    order_date,
    order_amount,
    SUM(order_amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS customer_lifetime_total
FROM orders
ORDER BY customer_id, order_date;
```

#### Use Cases

**Case 1: Cumulative Sales by Month**
```sql
SELECT 
    month,
    monthly_revenue,
    SUM(monthly_revenue) OVER (
        ORDER BY month
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_revenue,
    SUM(monthly_revenue) OVER (ORDER BY month) AS ytd_sales
FROM monthly_revenue
ORDER BY month;
```

**Case 2: Cash Flow Tracking**
```sql
SELECT 
    transaction_date,
    transaction_type,
    amount,
    SUM(
        CASE 
            WHEN transaction_type = 'Income' THEN amount
            ELSE -amount
        END
    ) OVER (
        ORDER BY transaction_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS balance
FROM transactions
ORDER BY transaction_date;
```

### Moving Averages

#### Definition
Average of values within a sliding window of rows.

#### Syntax

```sql
AVG(column) OVER (
    [PARTITION BY column1]
    ORDER BY column2
    ROWS BETWEEN N PRECEDING AND N FOLLOWING
)
```

#### Examples

```sql
-- 7-day moving average
SELECT 
    date,
    closing_price,
    AVG(closing_price) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM stock_prices
ORDER BY date;

-- 30-day moving average
SELECT 
    date,
    revenue,
    ROUND(
        AVG(revenue) OVER (
            ORDER BY date
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ), 
        2
    ) AS moving_avg_30day
FROM daily_revenue
ORDER BY date;
```

#### Use Cases

**Case 1: Smooth Out Volatile Data**
```sql
SELECT 
    date,
    daily_users,
    AVG(daily_users) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS weekly_avg,
    AVG(daily_users) OVER (
        ORDER BY date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS monthly_avg
FROM website_stats
ORDER BY date;
```

**Case 2: Anomaly Detection**
```sql
SELECT 
    date,
    value,
    AVG(value) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg,
    ABS(value - AVG(value) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    )) AS deviation,
    CASE 
        WHEN ABS(value - AVG(value) OVER (
            ORDER BY date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        )) > 2 * STDDEV(value) OVER (
            ORDER BY date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        )
        THEN 'Anomaly'
        ELSE 'Normal'
    END AS status
FROM sensor_readings
ORDER BY date;
```

### Window Aggregates Without Frame Clause

#### Default Frame Behavior

```sql
-- Without explicit frame, default differs by function

-- For RANK, ROW_NUMBER, etc. - entire partition
RANK() OVER (ORDER BY salary DESC)

-- For aggregate functions - entire partition (unless ORDER BY specified)
SUM(amount) OVER (PARTITION BY customer_id)  -- Sums all rows in partition

-- With ORDER BY - defaults to UNBOUNDED PRECEDING to CURRENT ROW
SUM(amount) OVER (
    PARTITION BY customer_id
    ORDER BY date
)  -- Running sum from start to current row
```

#### Examples

```sql
-- Total count per department
SELECT 
    employee_id,
    employee_name,
    department,
    COUNT(*) OVER (PARTITION BY department) AS dept_employee_count
FROM employees;

-- Average salary per department
SELECT 
    employee_id,
    employee_name,
    salary,
    department,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary,
    salary - AVG(salary) OVER (PARTITION BY department) AS salary_vs_avg
FROM employees;

-- Max salary per category
SELECT 
    product_id,
    product_name,
    category,
    price,
    MAX(price) OVER (PARTITION BY category) AS max_category_price
FROM products;
```

#### Production Tips

✅ **Tip 1: Be Explicit with Frame Specification**
```sql
-- Good - explicit about range
SUM(amount) OVER (
    ORDER BY date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)

-- Avoid - implicit behavior
SUM(amount) OVER (ORDER BY date)
```

✅ **Tip 2: Understand Frame Defaults**
```sql
-- All rows in partition (no ORDER BY)
SUM(sales) OVER (PARTITION BY region)

-- Running total (with ORDER BY)
SUM(sales) OVER (PARTITION BY region ORDER BY date)
```

✅ **Tip 3: Use RANGE for Date-Based Windows**
```sql
-- Good - finds all rows within 30 days
AVG(price) OVER (
    ORDER BY date
    RANGE BETWEEN INTERVAL 30 DAY PRECEDING AND CURRENT ROW
)

-- Instead of
AVG(price) OVER (
    ORDER BY date
    ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
)  -- Uses row count, not date range
```

---

## Value Functions

### FIRST_VALUE Function

#### Definition
Returns the first value in the window frame.

#### Standard Syntax

```sql
FIRST_VALUE(column) OVER (
    [PARTITION BY column1]
    ORDER BY column2
    [ROWS frame_specification]
)
```

#### Examples

```sql
-- Get first salary for each employee in their department
SELECT 
    employee_id,
    employee_name,
    hire_date,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS first_hire_salary
FROM employees;

-- Get price at start of year for each product
SELECT 
    date,
    product_id,
    price,
    FIRST_VALUE(price) OVER (
        PARTITION BY product_id
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS annual_start_price
FROM daily_prices;
```

#### Use Cases

**Case 1: Track Value Changes**
```sql
SELECT 
    date,
    product_id,
    price,
    FIRST_VALUE(price) OVER (
        PARTITION BY product_id
        ORDER BY date
    ) AS initial_price,
    price - FIRST_VALUE(price) OVER (
        PARTITION BY product_id
        ORDER BY date
    ) AS price_change
FROM product_prices
ORDER BY product_id, date;
```

**Case 2: Compare to Baseline**
```sql
SELECT 
    month,
    region,
    revenue,
    FIRST_VALUE(revenue) OVER (
        PARTITION BY region
        ORDER BY month
    ) AS baseline_revenue,
    (revenue / FIRST_VALUE(revenue) OVER (
        PARTITION BY region
        ORDER BY month
    ) - 1) * 100 AS pct_vs_baseline
FROM regional_revenue;
```

### LAST_VALUE Function

#### Definition
Returns the last value in the window frame.

#### Standard Syntax

```sql
LAST_VALUE(column) OVER (
    [PARTITION BY column1]
    ORDER BY column2
    [ROWS frame_specification]
)
```

#### Examples

```sql
-- Get most recent salary for each employee
SELECT 
    employee_id,
    employee_name,
    salary_effective_date,
    salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY employee_id
        ORDER BY salary_effective_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS current_salary
FROM salary_history;

-- Get latest status per order
SELECT 
    order_id,
    status_date,
    status,
    LAST_VALUE(status) OVER (
        PARTITION BY order_id
        ORDER BY status_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS current_status
FROM order_status_log;
```

#### Use Cases

**Case 1: Track Final Status**
```sql
SELECT 
    order_id,
    status_date,
    status,
    LAST_VALUE(status) OVER (
        PARTITION BY order_id
        ORDER BY status_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS final_status
FROM order_history
WHERE LAST_VALUE(status) OVER (
    PARTITION BY order_id
    ORDER BY status_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) = 'Delivered';
```

**Case 2: Compare Current to Latest**
```sql
SELECT 
    date,
    employee_id,
    daily_productivity,
    LAST_VALUE(daily_productivity) OVER (
        PARTITION BY employee_id
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS latest_productivity
FROM employee_performance;
```

#### Common Mistakes

❌ **Mistake 1: Not Specifying Frame for LAST_VALUE**
```sql
-- WRONG - with ORDER BY, frame defaults to CURRENT ROW
SELECT 
    date,
    value,
    LAST_VALUE(value) OVER (ORDER BY date)  -- Always equals current row!
FROM data;

-- CORRECT - specify frame
SELECT 
    date,
    value,
    LAST_VALUE(value) OVER (
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_value
FROM data;
```

❌ **Mistake 2: Confusion Between FIRST_VALUE and FIRST()**
```sql
-- FIRST_VALUE is window function
FIRST_VALUE(column) OVER (ORDER BY ...)

-- Some databases have aggregate FIRST() (different purpose)
```

#### Production Tips

✅ **Tip 1: Always Specify Frame for LAST_VALUE**
```sql
-- Good - explicit frame
LAST_VALUE(value) OVER (
    ORDER BY date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)

-- Avoid - confusing default behavior
LAST_VALUE(value) OVER (ORDER BY date)
```

✅ **Tip 2: Use with Partition for Baseline Comparisons**
```sql
-- Good - compare each to first in partition
SELECT 
    month,
    region,
    sales,
    FIRST_VALUE(sales) OVER (
        PARTITION BY region
        ORDER BY month
    ) AS region_baseline,
    sales - FIRST_VALUE(sales) OVER (
        PARTITION BY region
        ORDER BY month
    ) AS improvement
FROM regional_sales;
```

---

## Distribution Functions

### NTILE Function

#### Definition
Divides rows into N approximately equal buckets.

#### Standard Syntax

```sql
NTILE(num_buckets) OVER (
    [PARTITION BY column1]
    ORDER BY column2
)
```

#### Examples

```sql
-- Divide employees into quartiles by salary
SELECT 
    employee_id,
    employee_name,
    salary,
    NTILE(4) OVER (ORDER BY salary) AS salary_quartile
FROM employees;

-- Results: quartile 1, 2, 3, or 4

-- Divide customers into 5 groups by spending
SELECT 
    customer_id,
    total_spent,
    NTILE(5) OVER (ORDER BY total_spent) AS customer_quintile
FROM customers;
```

#### Use Cases

**Case 1: Segment Customers by Spending**
```sql
SELECT 
    customer_id,
    customer_name,
    total_spent,
    NTILE(4) OVER (ORDER BY total_spent DESC) AS customer_segment,
    CASE 
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 1 THEN 'Premium'
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 2 THEN 'Gold'
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 3 THEN 'Silver'
        ELSE 'Bronze'
    END AS segment_name
FROM customers;
```

**Case 2: Create Performance Tiers**
```sql
SELECT 
    employee_id,
    employee_name,
    performance_score,
    NTILE(3) OVER (
        PARTITION BY department
        ORDER BY performance_score DESC
    ) AS performance_tier
FROM employees
WHERE performance_tier = 1;  -- Top third
```

**Case 3: A/B Testing Groups**
```sql
SELECT 
    user_id,
    email,
    NTILE(2) OVER (ORDER BY user_id) AS test_group,
    CASE 
        WHEN NTILE(2) OVER (ORDER BY user_id) = 1 THEN 'Control'
        ELSE 'Treatment'
    END AS group_name
FROM users;
```

### PERCENT_RANK Function

#### Definition
Returns the percentile rank (0 to 1) of a row within the partition.

#### Standard Syntax

```sql
PERCENT_RANK() OVER (
    [PARTITION BY column1]
    ORDER BY column2
)
```

#### Formula
```
(rank - 1) / (total_rows - 1)
```

#### Examples

```sql
-- Get percentile rank of salaries
SELECT 
    employee_id,
    employee_name,
    salary,
    PERCENT_RANK() OVER (ORDER BY salary) AS percentile,
    ROUND(PERCENT_RANK() OVER (ORDER BY salary) * 100, 2) AS percentile_pct
FROM employees;

-- Results: 0.0 (minimum) to 1.0 (maximum)
```

#### Use Cases

**Case 1: Find Top Percentile Performers**
```sql
SELECT 
    employee_id,
    employee_name,
    performance_score,
    PERCENT_RANK() OVER (ORDER BY performance_score) AS percentile
FROM employees
WHERE PERCENT_RANK() OVER (ORDER BY performance_score) >= 0.75;
-- Top 25% performers
```

**Case 2: Revenue Distribution Analysis**
```sql
SELECT 
    month,
    revenue,
    PERCENT_RANK() OVER (ORDER BY revenue) AS revenue_percentile,
    CASE 
        WHEN PERCENT_RANK() OVER (ORDER BY revenue) < 0.25 THEN 'Bottom Quarter'
        WHEN PERCENT_RANK() OVER (ORDER BY revenue) < 0.5 THEN 'Below Average'
        WHEN PERCENT_RANK() OVER (ORDER BY revenue) < 0.75 THEN 'Above Average'
        ELSE 'Top Quarter'
    END AS performance_tier
FROM monthly_revenue;
```

### CUME_DIST Function

#### Definition
Returns cumulative distribution (0 to 1) - percentage of rows with values less than or equal to current row.

#### Standard Syntax

```sql
CUME_DIST() OVER (
    [PARTITION BY column1]
    ORDER BY column2
)
```

#### Formula
```
(number of rows with values <= current value) / total_rows
```

#### Examples

```sql
-- Cumulative distribution of test scores
SELECT 
    student_name,
    test_score,
    CUME_DIST() OVER (ORDER BY test_score) AS cumulative_dist,
    ROUND(CUME_DIST() OVER (ORDER BY test_score) * 100, 2) AS percentile
FROM student_scores;

-- Results: Students with same score have same CUME_DIST
```

#### Use Cases

**Case 1: Percentile Ranking**
```sql
SELECT 
    employee_id,
    salary,
    CUME_DIST() OVER (ORDER BY salary) AS cumulative_percentile,
    CASE 
        WHEN CUME_DIST() OVER (ORDER BY salary) <= 0.25 THEN 'Q1 (Bottom 25%)'
        WHEN CUME_DIST() OVER (ORDER BY salary) <= 0.5 THEN 'Q2 (25-50%)'
        WHEN CUME_DIST() OVER (ORDER BY salary) <= 0.75 THEN 'Q3 (50-75%)'
        ELSE 'Q4 (Top 25%)'
    END AS salary_quartile
FROM employees;
```

**Case 2: Find Cumulative Threshold**
```sql
SELECT 
    date,
    sales,
    CUME_DIST() OVER (ORDER BY sales DESC) AS cumulative_from_top,
    CASE 
        WHEN CUME_DIST() OVER (ORDER BY sales DESC) <= 0.2 THEN 'Top 20%'
        ELSE 'Other'
    END AS category
FROM daily_sales;
```

#### Comparison: Distribution Functions

```sql
SELECT 
    value,
    NTILE(4) OVER (ORDER BY value) AS ntile_4,
    PERCENT_RANK() OVER (ORDER BY value) AS percent_rank,
    CUME_DIST() OVER (ORDER BY value) AS cume_dist
FROM data;

-- NTILE: 1, 2, 3, or 4 (equal-sized buckets)
-- PERCENT_RANK: 0.0 to 1.0 (individual rank)
-- CUME_DIST: 0.0 to 1.0 (cumulative percentage)
```

#### Production Tips

✅ **Tip 1: Use NTILE for Segmentation**
```sql
-- Good - for customer segments
NTILE(4) OVER (ORDER BY lifetime_value)

-- Avoids manual cutoff calculation
```

✅ **Tip 2: Use PERCENT_RANK for Top N%**
```sql
-- Good - get top 10% without knowing absolute value
WHERE PERCENT_RANK() OVER (ORDER BY score DESC) <= 0.1

-- More flexible than fixed thresholds
```

✅ **Tip 3: Use CUME_DIST for Percentile Matching**
```sql
-- Good - find where values fit in distribution
WHERE CUME_DIST() OVER (ORDER BY value DESC) BETWEEN 0.25 AND 0.75
-- Middle 50%
```

---

## Frame Clauses

### Overview
Frame clauses define the set of rows used in window functions. Essential for aggregates and value functions.

### ROWS vs RANGE

#### ROWS
Counts by physical row position.

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
-- Previous 2 rows + current row
```

#### RANGE
Counts by value range (for dates, numbers).

```sql
RANGE BETWEEN INTERVAL 7 DAY PRECEDING AND CURRENT ROW
-- All rows within 7 days of current row
```

### Frame Specifications

#### UNBOUNDED PRECEDING
Start from first row in partition.

```sql
SUM(amount) OVER (
    ORDER BY date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
-- Running total from start
```

#### CURRENT ROW
The current row.

```sql
SUM(amount) OVER (
    ORDER BY date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
-- Sum of: 2 rows before + current
```

#### UNBOUNDED FOLLOWING
End at last row in partition.

```sql
SUM(amount) OVER (
    ORDER BY date
    ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
)
-- Sum from current row to end
```

### Common Frame Patterns

#### Running Total
```sql
SUM(amount) OVER (
    PARTITION BY customer_id
    ORDER BY date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

#### Moving Average (N Rows)
```sql
AVG(price) OVER (
    ORDER BY date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)
-- 7-day moving average (6 prior + current)
```

#### Moving Average (Date Range)
```sql
AVG(price) OVER (
    ORDER BY date
    RANGE BETWEEN INTERVAL 30 DAY PRECEDING AND CURRENT ROW
)
-- 30-day moving average by date
```

#### Entire Partition
```sql
AVG(salary) OVER (PARTITION BY department)
-- Average for entire department
```

#### Centered Window
```sql
AVG(value) OVER (
    ORDER BY date
    ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
)
-- 3 rows before + current + 3 rows after
```

### Examples

**Case 1: Multi-Window Analysis**
```sql
SELECT 
    date,
    value,
    -- Running total
    SUM(value) OVER (
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_sum,
    -- 3-day moving average
    AVG(value) OVER (
        ORDER BY date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3day,
    -- 30-day moving average by date
    AVG(value) OVER (
        ORDER BY date
        RANGE BETWEEN INTERVAL 30 DAY PRECEDING AND CURRENT ROW
    ) AS moving_avg_30day
FROM daily_data
ORDER BY date;
```

**Case 2: YTD vs YoY Comparison**
```sql
SELECT 
    month_date,
    revenue,
    -- Year-to-date
    SUM(revenue) OVER (
        PARTITION BY YEAR(month_date)
        ORDER BY month_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS ytd_revenue,
    -- Rolling 12-month
    SUM(revenue) OVER (
        ORDER BY month_date
        ROWS BETWEEN 11 PRECEDING AND CURRENT ROW
    ) AS rolling_12m_revenue
FROM monthly_sales;
```

**Case 3: Rank with Percentiles**
```sql
SELECT 
    score,
    COUNT(*) OVER () AS total_students,
    RANK() OVER (ORDER BY score DESC) AS rank,
    PERCENT_RANK() OVER (ORDER BY score DESC) AS percentile,
    COUNT(*) OVER (
        ORDER BY score DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_count
FROM test_scores;
```

#### Common Mistakes

❌ **Mistake 1: Wrong Frame for Moving Average**
```sql
-- WRONG - includes future rows (not standard moving avg)
AVG(price) OVER (
    ORDER BY date
    ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
)

-- CORRECT - look back only
AVG(price) OVER (
    ORDER BY date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)
```

❌ **Mistake 2: Confusing ROWS and RANGE**
```sql
-- WRONG - if dates have gaps, ROWS and RANGE differ
ROWS BETWEEN 30 PRECEDING AND CURRENT ROW
-- Last 30 rows (might span months if data is sparse)

RANGE BETWEEN INTERVAL 30 DAY PRECEDING AND CURRENT ROW
-- Exactly 30 days (better for date ranges)
```

❌ **Mistake 3: Missing UNBOUNDED for Aggregates**
```sql
-- WRONG - default with ORDER BY is UNBOUNDED PRECEDING to CURRENT ROW
SELECT 
    date,
    value,
    MAX(value) OVER (ORDER BY date)  -- Only max up to current row!
FROM data;

-- CORRECT - full partition
SELECT 
    date,
    value,
    MAX(value) OVER (
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS overall_max
FROM data;
```

#### Production Tips

✅ **Tip 1: Use RANGE for Date-Based Windows**
```sql
-- Good - handles date gaps intelligently
AVG(value) OVER (
    ORDER BY date
    RANGE BETWEEN INTERVAL 7 DAY PRECEDING AND CURRENT ROW
)
```

✅ **Tip 2: Be Explicit with Frame Boundaries**
```sql
-- Good - crystal clear intent
SUM(amount) OVER (
    PARTITION BY customer_id
    ORDER BY date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)

-- Avoid implicit behavior
SUM(amount) OVER (ORDER BY date)
```

✅ **Tip 3: Test Edge Cases**
```sql
-- Test first row (should have NULL or proper default)
-- Test last row (should have all previous values)
-- Test ties (ensure consistent ordering)
```

---

## Subqueries Overview

### What Are Subqueries?

Subqueries are queries nested within another query. They help break complex logic into manageable pieces.

### Types of Subqueries

1. **Scalar Subqueries** - Return single row, single column
2. **Inline Views** - Return multiple rows/columns, used in FROM
3. **Correlated Subqueries** - Reference outer query columns
4. **Dependent Subqueries** - Execute once per outer row

### Subquery Scoping

```sql
-- Subquery can access outer query
SELECT * FROM outer_table
WHERE column IN (
    SELECT column FROM inner_table
    WHERE inner_table.id = outer_table.id
)

-- But outer query cannot access subquery
SELECT column FROM (
    SELECT * FROM inner_table
) AS subquery
-- subquery can't reference outer query
```

---

## Scalar Subqueries

### Definition
Returns exactly one row with one column.

### Syntax

```sql
SELECT column1, (SELECT column2 FROM table2 WHERE condition) AS subquery_result
FROM table1;
```

### Examples

```sql
-- Get employee with their department's average salary
SELECT 
    employee_id,
    employee_name,
    salary,
    (SELECT AVG(salary) FROM employees e2 
     WHERE e2.department = e1.department) AS dept_avg_salary
FROM employees e1;

-- Get order with total sold that day
SELECT 
    order_id,
    customer_id,
    order_amount,
    (SELECT SUM(order_amount) FROM orders o2 
     WHERE o2.order_date = o1.order_date) AS daily_total
FROM orders o1;

-- In SELECT list
SELECT 
    product_id,
    (SELECT COUNT(*) FROM order_items WHERE product_id = p.product_id) AS times_ordered,
    (SELECT AVG(unit_price) FROM order_items WHERE product_id = p.product_id) AS avg_price
FROM products p;
```

### Use Cases

**Case 1: Add Contextual Information**
```sql
SELECT 
    employee_id,
    employee_name,
    salary,
    -- Add context from other tables
    (SELECT department_name FROM departments WHERE dept_id = e.department_id) AS dept_name,
    (SELECT COUNT(*) FROM employees e2 
     WHERE e2.department_id = e.department_id) AS team_size,
    (SELECT MAX(salary) FROM employees e2 
     WHERE e2.department_id = e.department_id) AS max_dept_salary
FROM employees e;
```

**Case 2: Include Aggregates in Detail View**
```sql
SELECT 
    customer_id,
    customer_name,
    email,
    (SELECT COUNT(*) FROM orders WHERE customer_id = c.customer_id) AS total_orders,
    (SELECT SUM(order_amount) FROM orders WHERE customer_id = c.customer_id) AS lifetime_value,
    (SELECT MAX(order_date) FROM orders WHERE customer_id = c.customer_id) AS last_order_date
FROM customers c;
```

**Case 3: Conditional Aggregates**
```sql
SELECT 
    order_id,
    customer_id,
    order_amount,
    (SELECT COUNT(*) FROM order_items WHERE order_id = o.order_id) AS item_count,
    (SELECT MAX(quantity) FROM order_items WHERE order_id = o.order_id) AS max_qty_item
FROM orders o;
```

### Common Mistakes

❌ **Mistake 1: Subquery Returns Multiple Rows**
```sql
-- WRONG - subquery returns multiple rows
SELECT 
    employee_id,
    (SELECT salary FROM employees e2 WHERE e2.department = e1.department) AS dept_salary
FROM employees e1;
-- Error: subquery returned more than 1 row

-- CORRECT - aggregate to single row
SELECT 
    employee_id,
    (SELECT AVG(salary) FROM employees e2 WHERE e2.department = e1.department) AS dept_avg
FROM employees e1;
```

❌ **Mistake 2: Performance Issues with Scalar Subqueries**
```sql
-- SLOW - executes subquery for every row
SELECT 
    employee_id,
    employee_name,
    (SELECT COUNT(*) FROM orders WHERE customer_id = e.employee_id) AS order_count
FROM employees e;  -- N+1 query problem

-- BETTER - use JOIN and aggregate
SELECT 
    e.employee_id,
    e.employee_name,
    COUNT(o.order_id) AS order_count
FROM employees e
LEFT JOIN orders o ON e.employee_id = o.customer_id
GROUP BY e.employee_id, e.employee_name;
```

❌ **Mistake 3: NULL Handling in Subqueries**
```sql
-- WRONG - NULL from subquery may cause issues
SELECT 
    employee_id,
    employee_name,
    salary + (SELECT bonus FROM bonuses WHERE emp_id = e.employee_id) AS total_comp
FROM employees e;
-- If bonus is NULL, total_comp is NULL

-- CORRECT - handle NULL
SELECT 
    employee_id,
    employee_name,
    salary + COALESCE(
        (SELECT bonus FROM bonuses WHERE emp_id = e.employee_id), 
        0
    ) AS total_comp
FROM employees e;
```

### Production Tips

✅ **Tip 1: Cache Subquery Results if Used Multiple Times**
```sql
-- Good - calculate once
WITH dept_stats AS (
    SELECT 
        department,
        AVG(salary) AS avg_salary,
        COUNT(*) AS emp_count
    FROM employees
    GROUP BY department
)
SELECT 
    e.employee_id,
    e.salary,
    ds.avg_salary,
    e.salary - ds.avg_salary AS diff
FROM employees e
JOIN dept_stats ds ON e.department = ds.department;

-- Avoid - repeated calculation
SELECT 
    employee_id,
    salary,
    (SELECT AVG(salary) FROM employees e2 WHERE e2.department = e1.department),
    salary - (SELECT AVG(salary) FROM employees e2 WHERE e2.department = e1.department)
FROM employees e1;
```

✅ **Tip 2: Use JOIN Instead When Possible**
```sql
-- Scalar subquery
SELECT 
    e.employee_id,
    (SELECT department_name FROM departments d WHERE d.dept_id = e.department_id) AS dept_name
FROM employees e;

-- Better - use JOIN
SELECT 
    e.employee_id,
    d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.dept_id;
```

---

## Inline View Subqueries

### Definition
Subqueries used in FROM clause that return multiple rows and columns.

### Syntax

```sql
SELECT columns
FROM (
    SELECT ... FROM ...
) AS subquery_alias
WHERE condition;
```

### Examples

```sql
-- Filter aggregated results
SELECT 
    category,
    avg_price,
    product_count
FROM (
    SELECT 
        category,
        AVG(price) AS avg_price,
        COUNT(*) AS product_count
    FROM products
    GROUP BY category
) AS category_stats
WHERE product_count > 10;

-- Create derived table for ranking
SELECT 
    employee_id,
    employee_name,
    salary,
    salary_rank
FROM (
    SELECT 
        employee_id,
        employee_name,
        salary,
        RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employees
) AS ranked_employees
WHERE salary_rank <= 5;
```

### Use Cases

**Case 1: Multi-Level Aggregation**
```sql
SELECT 
    region,
    AVG(quarterly_sales) AS avg_regional_sales
FROM (
    SELECT 
        region,
        quarter,
        SUM(sales) AS quarterly_sales
    FROM sales
    GROUP BY region, quarter
) AS quarterly_data
GROUP BY region;
```

**Case 2: Combining Multiple Datasets**
```sql
SELECT 
    employee_id,
    employee_name,
    total_sales,
    total_returns,
    net_sales
FROM (
    SELECT 
        e.employee_id,
        e.employee_name,
        COALESCE(s.total_sales, 0) AS total_sales,
        COALESCE(r.total_returns, 0) AS total_returns,
        COALESCE(s.total_sales, 0) - COALESCE(r.total_returns, 0) AS net_sales
    FROM employees e
    LEFT JOIN (
        SELECT employee_id, SUM(amount) AS total_sales
        FROM sales
        GROUP BY employee_id
    ) s ON e.employee_id = s.employee_id
    LEFT JOIN (
        SELECT employee_id, SUM(amount) AS total_returns
        FROM returns
        GROUP BY employee_id
    ) r ON e.employee_id = r.employee_id
) AS employee_financials
WHERE net_sales > 0;
```

**Case 3: Rank Within Group**
```sql
SELECT 
    category,
    product_name,
    revenue,
    category_rank
FROM (
    SELECT 
        category,
        product_name,
        revenue,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS category_rank
    FROM products
) AS ranked_products
WHERE category_rank <= 5;
```

### Common Mistakes

❌ **Mistake 1: Missing Alias for Subquery**
```sql
-- WRONG
SELECT * FROM (SELECT * FROM employees);

-- CORRECT
SELECT * FROM (SELECT * FROM employees) AS emp;
```

❌ **Mistake 2: Complex Nested Subqueries (Readability)**
```sql
-- Hard to read and debug
SELECT * FROM (
    SELECT * FROM (
        SELECT * FROM (
            SELECT * FROM data
        ) AS level3
    ) AS level2
) AS level1;

-- Use CTE instead
WITH level1 AS (SELECT * FROM data),
     level2 AS (SELECT * FROM level1),
     level3 AS (SELECT * FROM level2)
SELECT * FROM level3;
```

### Production Tips

✅ **Tip 1: Use CTEs Instead of Nested Subqueries**
```sql
-- Good - readable
WITH sales_summary AS (
    SELECT region, SUM(amount) AS total FROM sales GROUP BY region
),
top_regions AS (
    SELECT * FROM sales_summary WHERE total > 100000
)
SELECT * FROM top_regions;

-- Less readable
SELECT * FROM (
    SELECT * FROM (
        SELECT region, SUM(amount) AS total FROM sales GROUP BY region
    ) AS ss
    WHERE total > 100000
) AS tr;
```

✅ **Tip 2: Test Subquery Independently**
```sql
-- First, test the subquery alone
SELECT 
    employee_id,
    employee_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;

-- Then wrap in outer query
SELECT 
    employee_id,
    employee_name,
    salary
FROM (
    SELECT 
        employee_id,
        employee_name,
        salary,
        RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employees
) AS ranked
WHERE salary_rank <= 5;
```

---

## Correlated Subqueries

### Definition
Subqueries that reference columns from the outer query. Executes once per outer row (slower).

### Syntax

```sql
SELECT column1, column2
FROM table1 outer_alias
WHERE EXISTS / IN / = (
    SELECT ... 
    FROM table2 
    WHERE table2.id = outer_alias.id
);
```

### Examples

```sql
-- Find employees earning more than their department average
SELECT 
    employee_id,
    employee_name,
    salary,
    department
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department = e.department
);

-- Find products above category average price
SELECT 
    product_id,
    product_name,
    price,
    category
FROM products p
WHERE price > (
    SELECT AVG(price)
    FROM products p2
    WHERE p2.category = p.category
);
```

### Use Cases

**Case 1: Identify Outliers**
```sql
SELECT 
    date,
    value,
    (SELECT AVG(value) FROM daily_data d2 
     WHERE MONTH(d2.date) = MONTH(d1.date)
     AND YEAR(d2.date) = YEAR(d1.date)) AS monthly_avg,
    value - (SELECT AVG(value) FROM daily_data d2 
     WHERE MONTH(d2.date) = MONTH(d1.date)
     AND YEAR(d2.date) = YEAR(d1.date)) AS deviation
FROM daily_data d1
WHERE ABS(value - (SELECT AVG(value) FROM daily_data d2 
    WHERE MONTH(d2.date) = MONTH(d1.date)
    AND YEAR(d2.date) = YEAR(d1.date))) > 2;
```

**Case 2: Comparing to Aggregate**
```sql
SELECT 
    customer_id,
    customer_name,
    total_spent,
    (SELECT AVG(total_spent) FROM customers c2) AS overall_avg
FROM customers c
WHERE total_spent > (SELECT AVG(total_spent) FROM customers);
```

**Case 3: Complex Filtering**
```sql
SELECT 
    department,
    employee_id,
    salary
FROM employees e1
WHERE NOT EXISTS (
    SELECT 1 FROM employees e2
    WHERE e2.department = e1.department
    AND e2.salary > e1.salary
    LIMIT 1  -- Ensure no one in dept earns more
);
-- Top earner in each department
```

### Performance Considerations

❌ **Correlated subqueries execute for EVERY row**

```sql
-- Executes subquery 1000 times (if 1000 employees)
SELECT 
    employee_id,
    salary,
    (SELECT AVG(salary) FROM employees e2 
     WHERE e2.department = e1.department) AS dept_avg
FROM employees e1;
```

✅ **Better: Use JOIN with GROUP BY**

```sql
-- Executes subquery once
SELECT 
    e.employee_id,
    e.salary,
    e_stats.dept_avg
FROM employees e
JOIN (
    SELECT department, AVG(salary) AS dept_avg
    FROM employees
    GROUP BY department
) e_stats ON e.department = e_stats.department;
```

### Common Mistakes

❌ **Mistake 1: N+1 Query Problem**
```sql
-- SLOW - correlated subquery executes for every row
SELECT 
    customer_id,
    customer_name,
    (SELECT COUNT(*) FROM orders WHERE customer_id = c.customer_id) AS order_count
FROM customers c;

-- BETTER - single query
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

❌ **Mistake 2: Complex Conditions in Correlated Subquery**
```sql
-- SLOW and hard to read
SELECT 
    order_id,
    customer_id,
    (SELECT SUM(amount) FROM order_items oi
     WHERE oi.order_id = o.order_id
     AND oi.product_id IN (SELECT product_id FROM products WHERE category = 'Electronics'))
FROM orders o;

-- Better - use CTE
WITH electronics_items AS (
    SELECT order_id, SUM(amount) AS electronics_total
    FROM order_items
    WHERE product_id IN (SELECT product_id FROM products WHERE category = 'Electronics')
    GROUP BY order_id
)
SELECT 
    o.order_id,
    o.customer_id,
    COALESCE(ei.electronics_total, 0) AS electronics_total
FROM orders o
LEFT JOIN electronics_items ei ON o.order_id = ei.order_id;
```

---

## CTEs (WITH Clause)

### Overview
Common Table Expressions (CTEs) make complex queries readable by breaking them into named steps.

### Standard Syntax

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ... FROM cte_name;
```

### Basic Example

```sql
WITH employee_departments AS (
    SELECT 
        department,
        AVG(salary) AS avg_salary,
        COUNT(*) AS emp_count
    FROM employees
    GROUP BY department
)
SELECT 
    department,
    avg_salary,
    emp_count
FROM employee_departments
WHERE emp_count > 5;
```

### Multiple CTEs

```sql
WITH sales_summary AS (
    SELECT 
        employee_id,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY employee_id
),
top_salespeople AS (
    SELECT 
        employee_id,
        total_sales
    FROM sales_summary
    WHERE total_sales > 10000
)
SELECT *
FROM top_salespeople
ORDER BY total_sales DESC;
```

### Use Cases

**Case 1: Multi-Step Logic**
```sql
WITH monthly_revenue AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
),
revenue_with_growth AS (
    SELECT 
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
        (revenue - LAG(revenue) OVER (ORDER BY month)) / 
        LAG(revenue) OVER (ORDER BY month) * 100 AS growth_pct
    FROM monthly_revenue
)
SELECT *
FROM revenue_with_growth
WHERE growth_pct > 10;
```

**Case 2: Data Preparation**
```sql
WITH cleaned_data AS (
    SELECT 
        customer_id,
        TRIM(customer_name) AS customer_name,
        COALESCE(email, 'unknown@email.com') AS email,
        CAST(created_date AS DATE) AS created_date
    FROM raw_customers
    WHERE customer_id IS NOT NULL
),
active_customers AS (
    SELECT 
        customer_id,
        customer_name,
        email,
        created_date,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY created_date) AS email_rank
    FROM cleaned_data
)
SELECT *
FROM active_customers
WHERE email_rank = 1;  -- Keep first occurrence only
```

**Case 3: Complex Joins**
```sql
WITH customer_orders AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        COUNT(o.order_id) AS order_count,
        SUM(o.amount) AS total_spent
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name
),
customer_reviews AS (
    SELECT 
        customer_id,
        COUNT(*) AS review_count,
        AVG(rating) AS avg_rating
    FROM reviews
    GROUP BY customer_id
)
SELECT 
    co.customer_id,
    co.customer_name,
    co.order_count,
    co.total_spent,
    COALESCE(cr.review_count, 0) AS review_count,
    COALESCE(cr.avg_rating, 0) AS avg_rating
FROM customer_orders co
LEFT JOIN customer_reviews cr ON co.customer_id = cr.customer_id;
```

### Common Mistakes

❌ **Mistake 1: CTE References Itself Incorrectly**
```sql
-- WRONG - forward reference (CTE can't reference itself)
WITH cte AS (
    SELECT 
        value,
        LAG(value) OVER (ORDER BY date) AS prev_value
    FROM cte  -- Error!
)
SELECT * FROM cte;

-- CORRECT - for recursion, use RECURSIVE
WITH RECURSIVE cte AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM cte WHERE n < 10
)
SELECT * FROM cte;
```

❌ **Mistake 2: CTE Not Referenced in Main Query**
```sql
-- The CTE is defined but never used
WITH unused_cte AS (
    SELECT * FROM employees
)
SELECT * FROM departments;  -- CTE not used!
```

### Advantages of CTEs

1. **Readability** - Break complex logic into steps
2. **Reusability** - Reference CTE multiple times in same query
3. **Debugging** - Test each CTE independently
4. **Maintainability** - Easier to modify and understand

### Production Tips

✅ **Tip 1: Name CTEs Descriptively**
```sql
-- Good
WITH monthly_sales_summary AS (...),
     ytd_calculations AS (...),
     top_products AS (...)

-- Avoid
WITH cte1 AS (...),
     cte2 AS (...),
     cte3 AS (...)
```

✅ **Tip 2: Order CTEs Logically**
```sql
-- Good - dependencies flow downward
WITH raw_data AS (...),           -- Base data
     cleaned_data AS (...),       -- Data cleaning
     aggregated AS (...),         -- Aggregation
     final_results AS (...)       -- Final transformation
SELECT * FROM final_results;
```

✅ **Tip 3: Reference CTE Multiple Times**
```sql
-- Good - calculate once, use many times
WITH dept_stats AS (
    SELECT department, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
)
SELECT 
    e.employee_id,
    e.salary,
    ds.avg_sal AS dept_avg,
    e.salary - ds.avg_sal AS difference
FROM employees e
JOIN dept_stats ds ON e.department = ds.department
WHERE e.salary > ds.avg_sal;
```

---

## Recursive CTEs

### Definition
CTEs that reference themselves to generate sequences or hierarchical data.

### Syntax

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor: Base case
    SELECT ... 
    WHERE condition
    UNION ALL
    -- Recursive: References itself
    SELECT ... FROM cte_name 
    WHERE termination_condition
)
SELECT * FROM cte_name;
```

### Two Parts

1. **Anchor Member** - Base case, executes once
2. **Recursive Member** - References CTE, generates iteratively

### Examples

**Case 1: Generate Number Sequence**
```sql
WITH RECURSIVE numbers AS (
    -- Anchor: start at 1
    SELECT 1 AS n
    UNION ALL
    -- Recursive: increment until 10
    SELECT n + 1
    FROM numbers
    WHERE n < 10
)
SELECT * FROM numbers;
-- Results: 1, 2, 3, ..., 10
```

**Case 2: Date Series**
```sql
WITH RECURSIVE date_series AS (
    -- Start date
    SELECT '2024-01-01'::DATE AS date
    UNION ALL
    -- Add one day each iteration
    SELECT date + INTERVAL 1 DAY
    FROM date_series
    WHERE date < '2024-12-31'
)
SELECT * FROM date_series;
```

**Case 3: Hierarchical Organization**
```sql
WITH RECURSIVE org_hierarchy AS (
    -- Anchor: CEOs (no manager)
    SELECT 
        employee_id,
        employee_name,
        manager_id,
        1 AS hierarchy_level,
        CAST(employee_name AS VARCHAR(500)) AS path
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    -- Recursive: find reports for each manager
    SELECT 
        e.employee_id,
        e.employee_name,
        e.manager_id,
        oh.hierarchy_level + 1,
        CONCAT(oh.path, ' > ', e.employee_name)
    FROM employees e
    JOIN org_hierarchy oh ON e.manager_id = oh.employee_id
    WHERE oh.hierarchy_level < 10  -- Prevent infinite loop
)
SELECT *
FROM org_hierarchy
ORDER BY hierarchy_level, employee_id;
```

**Case 4: Hierarchical Categories**
```sql
WITH RECURSIVE category_tree AS (
    -- Anchor: top-level categories
    SELECT 
        category_id,
        category_name,
        parent_category_id,
        1 AS level,
        CAST(category_name AS VARCHAR(1000)) AS full_path
    FROM categories
    WHERE parent_category_id IS NULL
    UNION ALL
    -- Recursive: find subcategories
    SELECT 
        c.category_id,
        c.category_name,
        c.parent_category_id,
        ct.level + 1,
        CONCAT(ct.full_path, ' > ', c.category_name)
    FROM categories c
    JOIN category_tree ct ON c.parent_category_id = ct.category_id
    WHERE ct.level < 10
)
SELECT *
FROM category_tree
ORDER BY full_path;
```

### Common Mistakes

❌ **Mistake 1: Missing UNION ALL**
```sql
-- WRONG - UNION removes duplicates, incomplete results
WITH RECURSIVE cte AS (
    SELECT 1 AS n
    UNION  -- Should be UNION ALL
    SELECT n + 1 FROM cte WHERE n < 10
)
SELECT * FROM cte;
```

❌ **Mistake 2: No Termination Condition**
```sql
-- WRONG - infinite loop!
WITH RECURSIVE infinite_loop AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM infinite_loop
    -- WHERE clause missing - never stops!
)
SELECT * FROM infinite_loop;

-- CORRECT
WITH RECURSIVE controlled_recursion AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM controlled_recursion
    WHERE n < 100  -- Termination condition
)
SELECT * FROM controlled_recursion;
```

❌ **Mistake 3: Modifying Anchor/Recursive Members Incorrectly**
```sql
-- WRONG - changing structure between anchor and recursive
WITH RECURSIVE broken_cte AS (
    SELECT id, name FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT employee_id FROM employees  -- Different columns!
    JOIN broken_cte ON ...
)
SELECT * FROM broken_cte;

-- CORRECT - consistent structure
WITH RECURSIVE cte AS (
    SELECT id, name FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name
    FROM employees e
    JOIN cte ON e.manager_id = cte.id
)
SELECT * FROM cte;
```

### Production Tips

✅ **Tip 1: Always Set Recursion Limit**
```sql
-- Good - prevent runaway queries
WITH RECURSIVE cte AS (
    SELECT ..., 1 AS depth
    UNION ALL
    SELECT ..., depth + 1
    FROM cte
    WHERE depth < 20  -- Maximum depth
)
```

✅ **Tip 2: Index Columns Used in Joins**
```sql
-- For recursive CTEs with hierarchies, index the join column
CREATE INDEX idx_manager_id ON employees(manager_id);
```

✅ **Tip 3: Test Recursive Logic Carefully**
```sql
-- Test with LIMIT to see results gradually
WITH RECURSIVE cte AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM cte WHERE n < 1000
)
SELECT * FROM cte LIMIT 50;
```

---

## EXISTS vs IN

### EXISTS

#### Definition
Checks if subquery returns any rows. More efficient for "does it exist" checks.

#### Syntax

```sql
SELECT *
FROM table1
WHERE EXISTS (
    SELECT 1 FROM table2 WHERE table1.id = table2.id
);
```

#### Examples

```sql
-- Find customers with orders
SELECT 
    customer_id,
    customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- Find products not in any order (anti-join)
SELECT 
    product_id,
    product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi
    WHERE oi.product_id = p.product_id
);
```

### IN

#### Definition
Checks if value is in a list/subquery result set.

#### Syntax

```sql
SELECT *
FROM table1
WHERE column IN (SELECT column FROM table2);
```

#### Examples

```sql
-- Find orders from specific customers
SELECT 
    order_id,
    customer_id,
    order_amount
FROM orders
WHERE customer_id IN (
    SELECT customer_id FROM customers
    WHERE country = 'USA'
);

-- Find products in specific categories
SELECT *
FROM products
WHERE category_id IN (1, 5, 10, 15);
```

### EXISTS vs IN Comparison

| Aspect | EXISTS | IN |
|--------|--------|-----|
| **Return Value** | TRUE/FALSE | TRUE/FALSE/NULL |
| **NULL Handling** | NULL in subquery ok | NULL causes unknown |
| **Performance** | Better for large sets | Better for small sets |
| **Short Circuits** | Stops on first match | Evaluates all |
| **Subquery Columns** | Any (ignores SELECT list) | Must select the check column |

#### Examples Showing Difference

```sql
-- EXISTS: Stops after finding one match
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders WHERE customer_id = c.customer_id
);

-- IN: Checks against entire list
SELECT * FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id FROM orders
);
```

### NOT EXISTS vs NOT IN

❌ **Mistake: NULL Handling in NOT IN**

```sql
-- WRONG - returns no results if subquery contains NULL
SELECT * FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM orders
    WHERE customer_id IS NULL  -- Included!
);
-- NULL makes the NOT IN false for all rows

-- CORRECT - use NOT EXISTS
SELECT * FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

### Use Cases

**Case 1: Semi-Join (EXISTS)**
```sql
-- Find employees with direct reports
SELECT 
    employee_id,
    employee_name
FROM employees e
WHERE EXISTS (
    SELECT 1 FROM employees e2
    WHERE e2.manager_id = e.employee_id
);
```

**Case 2: Anti-Join (NOT EXISTS)**
```sql
-- Find products never ordered
SELECT 
    product_id,
    product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi
    WHERE oi.product_id = p.product_id
);
```

**Case 3: Complex Exists Condition**
```sql
-- Find customers with high-value orders
SELECT DISTINCT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.order_amount > 5000
);
```

### Production Tips

✅ **Tip 1: Prefer EXISTS for Large Datasets**
```sql
-- Good - stops on first match
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders WHERE customer_id = c.customer_id
);

-- Slower for large order list
SELECT * FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);
```

✅ **Tip 2: Use EXISTS for Correlated Checks**
```sql
-- Good - efficient correlated check
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.order_date >= CURRENT_DATE - 30
);
```

✅ **Tip 3: Avoid NOT IN with NULLs**
```sql
-- Good - explicit NULL handling
SELECT * FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items WHERE product_id = p.product_id
);

-- Risky if subquery might return NULL
SELECT * FROM products
WHERE product_id NOT IN (SELECT product_id FROM order_items);
```

---

## String Functions

### CONCAT

#### Definition
Combines multiple strings into one.

#### Syntax

```sql
CONCAT(string1, string2, [string3, ...])
```

#### Dialect Variations

```sql
-- Standard SQL / Most databases
SELECT CONCAT('Hello', ' ', 'World');

-- MySQL
SELECT CONCAT(first_name, ' ', last_name) FROM employees;

-- PostgreSQL (also supports ||)
SELECT first_name || ' ' || last_name FROM employees;

-- SQL Server (+ operator)
SELECT first_name + ' ' + last_name FROM employees;

-- SQLite (||)
SELECT first_name || ' ' || last_name FROM employees;

-- Oracle (||)
SELECT first_name || ' ' || last_name FROM employees;
```

#### Examples

```sql
-- Combine name parts
SELECT 
    CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;

-- Build email
SELECT 
    CONCAT(
        LOWER(first_name), 
        '.', 
        LOWER(last_name), 
        '@company.com'
    ) AS email
FROM employees;

-- Multiple concatenations
SELECT 
    CONCAT(
        customer_name,
        ' (ID: ',
        customer_id,
        ') - ',
        email
    ) AS customer_info
FROM customers;
```

#### Handling NULLs

❌ **Mistake: NULL Propagation**
```sql
-- WRONG - if any part is NULL, result is NULL
SELECT CONCAT(first_name, ' ', middle_name, ' ', last_name)
FROM employees;
-- Returns NULL if middle_name is NULL

-- CORRECT - handle NULLs
SELECT CONCAT(
    first_name,
    CASE WHEN middle_name IS NOT NULL THEN CONCAT(' ', middle_name) ELSE '' END,
    ' ',
    last_name
) AS full_name
FROM employees;
```

### SUBSTRING / SUBSTR

#### Definition
Extracts part of a string.

#### Syntax

```sql
SUBSTRING(string, start [, length])
-- or
SUBSTR(string, start [, length])
```

#### Parameters
```
string      -- Original string
start       -- Starting position (1-based in most databases)
length      -- Number of characters (optional, default: to end)
```

#### Dialect Variations

```sql
-- PostgreSQL
SUBSTRING('Hello World' FROM 1 FOR 5)  -- 'Hello'

-- MySQL
SUBSTRING('Hello World', 1, 5)  -- 'Hello'

-- SQL Server
SUBSTRING('Hello World', 1, 5)  -- 'Hello'

-- Oracle
SUBSTR('Hello World', 1, 5)  -- 'Hello'

-- SQLite
SUBSTR('Hello World', 1, 5)  -- 'Hello'
```

#### Examples

```sql
-- Extract area code from phone
SELECT 
    SUBSTRING(phone_number, 1, 3) AS area_code
FROM employees;

-- Extract last 4 digits
SELECT 
    SUBSTRING(phone_number, -4) AS last_4_digits
FROM employees;

-- Extract domain from email
SELECT 
    SUBSTRING(email, POSITION('@' IN email) + 1) AS domain
FROM customers;
```

#### Use Cases

**Case 1: Format Data**
```sql
SELECT 
    SUBSTRING(ssn, 1, 3) || '-' ||
    SUBSTRING(ssn, 4, 2) || '-' ||
    SUBSTRING(ssn, 6, 4) AS formatted_ssn
FROM employees;
```

**Case 2: Parse Structured Strings**
```sql
-- Extract parts from code like "USANewYork123"
SELECT 
    SUBSTRING(location_code, 1, 3) AS country_code,
    SUBSTRING(location_code, 4, 8) AS region,
    SUBSTRING(location_code, 12) AS location_id
FROM locations;
```

### TRIM, LTRIM, RTRIM

#### Definition
Removes leading/trailing whitespace or specified characters.

#### Syntax

```sql
TRIM([characters FROM] string)
LTRIM(string [, characters])  -- Left trim
RTRIM(string [, characters])  -- Right trim
```

#### Examples

```sql
-- Remove leading/trailing spaces
SELECT TRIM('  Hello World  ');  -- 'Hello World'

-- Remove specific characters
SELECT TRIM('0' FROM '00123400');  -- '1234'

-- Remove leading zeros
SELECT LTRIM(product_code, '0') FROM products;

-- Remove trailing spaces
SELECT RTRIM(description) FROM product_descriptions;
```

#### Dialect Variations

```sql
-- PostgreSQL
TRIM(string)
LTRIM(string [, characters])
RTRIM(string [, characters])

-- MySQL
TRIM(string)
LTRIM(string)
RTRIM(string)

-- SQL Server
TRIM(string)
LTRIM(string)
RTRIM(string)

-- Oracle
TRIM(string)
-- For LTRIM/RTRIM, use LTRIM(string, characters) / RTRIM(string, characters)

-- SQLite
TRIM(string)
LTRIM(string)
RTRIM(string)
```

#### Use Cases

**Case 1: Data Cleaning**
```sql
SELECT 
    TRIM(customer_name) AS clean_name,
    TRIM(email) AS clean_email,
    TRIM(phone_number) AS clean_phone
FROM raw_customer_data;
```

**Case 2: Remove Leading Zeros**
```sql
SELECT 
    CAST(LTRIM(order_id, '0') AS INT) AS order_id
FROM orders
WHERE order_id LIKE '000%';
```

### SPLIT / STRING_SPLIT

#### Definition
Splits string by delimiter into multiple rows or an array.

#### Syntax

```sql
-- PostgreSQL (returns array)
STRING_TO_ARRAY(string, delimiter)

-- SQL Server (returns table)
STRING_SPLIT(string, delimiter)

-- MySQL (not directly, use alternative method)
SUBSTRING_INDEX(string, delimiter, position)
```

#### Dialect Variations

```sql
-- PostgreSQL
SELECT UNNEST(STRING_TO_ARRAY('apple,banana,cherry', ','));
-- Results: apple, banana, cherry (3 rows)

-- SQL Server
SELECT value FROM STRING_SPLIT('apple,banana,cherry', ',');
-- Results: apple, banana, cherry (3 rows)

-- MySQL (alternative)
SELECT SUBSTRING_INDEX('apple,banana,cherry', ',', 1);  -- apple

-- Oracle (alternative)
SELECT TRIM(REGEXP_SUBSTR('apple,banana,cherry', '[^,]+', 1, 1)) 
FROM dual;
```

#### Examples

```sql
-- PostgreSQL: Parse comma-separated values
SELECT 
    id,
    UNNEST(STRING_TO_ARRAY(tags, ',')) AS tag
FROM products;

-- SQL Server: Parse pipe-separated values
SELECT 
    product_id,
    value AS category
FROM products
CROSS APPLY STRING_SPLIT(categories, '|');

-- MySQL: Extract first element
SELECT 
    SUBSTRING_INDEX(tags, ',', 1) AS primary_tag
FROM products;
```

#### Use Cases

**Case 1: Normalize Comma-Separated Values**
```sql
-- PostgreSQL
SELECT 
    product_id,
    product_name,
    UNNEST(STRING_TO_ARRAY(features, ', ')) AS feature
FROM products;

-- SQL Server
SELECT 
    product_id,
    product_name,
    TRIM(value) AS feature
FROM products
CROSS APPLY STRING_SPLIT(features, ',');
```

**Case 2: Parse Address Components**
```sql
-- SQL Server
SELECT 
    customer_id,
    TRIM(value) AS address_part
FROM customers
CROSS APPLY STRING_SPLIT(address, ',')
ORDER BY customer_id;
```

### UPPER, LOWER, INITCAP

#### Definition
Changes string case.

#### Examples

```sql
-- Convert to uppercase
SELECT UPPER('hello world');  -- 'HELLO WORLD'

-- Convert to lowercase
SELECT LOWER('HELLO WORLD');  -- 'hello world'

-- Initial capital (PostgreSQL)
SELECT INITCAP('hello world');  -- 'Hello World'

-- SQL Server (manual INITCAP)
SELECT 
    UPPER(SUBSTRING(name, 1, 1)) + LOWER(SUBSTRING(name, 2))
FROM employees;
```

#### Use Cases

**Case 1: Case-Insensitive Search**
```sql
SELECT * FROM employees
WHERE UPPER(last_name) = 'SMITH';
```

**Case 2: Normalize Data**
```sql
SELECT 
    LOWER(email) AS normalized_email,
    COUNT(*) AS count
FROM users
GROUP BY LOWER(email);  -- Find duplicate emails ignoring case
```

---

## Date/Time Functions

### DATE_TRUNC

#### Definition
Truncates date/timestamp to specified precision.

#### Syntax

```sql
DATE_TRUNC(precision, timestamp)
```

#### Precision Options

```
'day'       -- Truncate to day (00:00:00)
'month'     -- Truncate to first day of month
'year'      -- Truncate to January 1
'hour'      -- Truncate to hour
'minute'    -- Truncate to minute
'second'    -- Truncate to second
```

#### Dialect Variations

```sql
-- PostgreSQL
DATE_TRUNC('day', timestamp_column)

-- MySQL (alternative)
DATE(timestamp_column)  -- Truncate to day
DATE_FORMAT(timestamp_column, '%Y-%m-01')  -- Truncate to month

-- SQL Server (alternative)
DATEPART(day, timestamp_column)
CONVERT(DATE, timestamp_column)  -- Truncate to day

-- Oracle (alternative)
TRUNC(timestamp_column, 'day')

-- SQLite (alternative)
DATE(timestamp_column)
STRFTIME('%Y-%m-%d', timestamp_column)
```

#### Examples

```sql
-- Group by day
SELECT 
    DATE_TRUNC('day', created_at) AS day,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE_TRUNC('day', created_at);

-- Group by month
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    SUM(order_amount) AS monthly_revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date);

-- Get start of month
SELECT 
    DATE_TRUNC('month', CURRENT_DATE) AS month_start
FROM orders;
```

### EXTRACT

#### Definition
Extracts part of a date or timestamp.

#### Syntax

```sql
EXTRACT(field FROM date_expression)
```

#### Field Options

```
YEAR        -- Extract year
MONTH       -- Extract month (1-12)
DAY         -- Extract day (1-31)
HOUR        -- Extract hour (0-23)
MINUTE      -- Extract minute (0-59)
SECOND      -- Extract second (0-59)
QUARTER     -- Extract quarter (1-4)
WEEK        -- Extract week number
DOW         -- Day of week (0-6 or 1-7)
```

#### Dialect Variations

```sql
-- PostgreSQL
EXTRACT(MONTH FROM order_date)

-- MySQL
MONTH(order_date)
YEAR(order_date)
DAY(order_date)

-- SQL Server
MONTH(order_date)
YEAR(order_date)
DAY(order_date)
DATEPART(MONTH, order_date)

-- Oracle
EXTRACT(MONTH FROM order_date)
TO_CHAR(order_date, 'MM')

-- SQLite
STRFTIME('%m', order_date)
```

#### Examples

```sql
-- Find orders by month
SELECT 
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS order_count
FROM orders
GROUP BY EXTRACT(MONTH FROM order_date);

-- Find Q1 orders
SELECT 
    order_id,
    order_date
FROM orders
WHERE EXTRACT(QUARTER FROM order_date) = 1;

-- Find weekend orders
SELECT 
    order_id,
    order_date
FROM orders
WHERE EXTRACT(DOW FROM order_date) IN (0, 6);  -- Saturday, Sunday
```

### DATEDIFF

#### Definition
Calculates difference between two dates.

#### Syntax

```sql
DATEDIFF(unit, date1, date2)
```

#### Dialect Variations

**PostgreSQL**
```sql
-- Use subtraction operator
SELECT order_date2 - order_date1 AS days_diff
FROM orders;

-- Or use age() function
SELECT AGE(order_date2, order_date1) AS interval
FROM orders;
```

**MySQL**
```sql
DATEDIFF(date1, date2)  -- Returns number of days
TIMEDIFF(datetime1, datetime2)  -- Returns time difference
```

**SQL Server**
```sql
DATEDIFF(unit, date1, date2)
-- Units: year, quarter, month, week, day, hour, minute, second
```

**Oracle**
```sql
-- Simple subtraction for days
SELECT date1 - date2 FROM orders;
```

**SQLite**
```sql
JULIANDAY(date1) - JULIANDAY(date2)  -- Days difference
```

#### Examples

```sql
-- SQL Server: Days between order and delivery
SELECT 
    order_id,
    order_date,
    delivery_date,
    DATEDIFF(DAY, order_date, delivery_date) AS delivery_days
FROM orders;

-- SQL Server: Hours between timestamps
SELECT 
    event_id,
    start_time,
    end_time,
    DATEDIFF(HOUR, start_time, end_time) AS duration_hours
FROM events;

-- PostgreSQL: Days between dates
SELECT 
    order_id,
    delivery_date - order_date AS delivery_days
FROM orders;
```

#### Use Cases

**Case 1: Calculate Tenure**
```sql
SELECT 
    employee_id,
    employee_name,
    hire_date,
    DATEDIFF(YEAR, hire_date, CURRENT_DATE) AS years_employed
FROM employees;
```

**Case 2: Service Level Compliance**
```sql
SELECT 
    support_ticket_id,
    submitted_date,
    resolved_date,
    DATEDIFF(HOUR, submitted_date, resolved_date) AS resolution_hours,
    CASE 
        WHEN DATEDIFF(HOUR, submitted_date, resolved_date) <= 24 THEN 'On Time'
        ELSE 'Late'
    END AS sla_status
FROM support_tickets;
```

**Case 3: Age Calculation**
```sql
SELECT 
    employee_id,
    employee_name,
    birth_date,
    DATEDIFF(YEAR, birth_date, CURRENT_DATE) AS age
FROM employees;
```

### DATE_ADD / DATE_SUB / DATE_FORMAT

#### DATE_ADD / DATE_SUB (MySQL)

```sql
DATE_ADD(date, INTERVAL value unit)
DATE_SUB(date, INTERVAL value unit)

-- Examples
DATE_ADD('2024-01-15', INTERVAL 30 DAY)    -- 2024-02-14
DATE_SUB('2024-01-15', INTERVAL 1 MONTH)   -- 2023-12-15
```

#### INTERVAL (PostgreSQL, SQL Server)

```sql
-- PostgreSQL
DATE '2024-01-15' + INTERVAL '30 days'
DATE '2024-01-15' - INTERVAL '1 month'

-- SQL Server
DATEADD(DAY, 30, '2024-01-15')
DATEADD(MONTH, -1, '2024-01-15')
```

#### Examples

```sql
-- Find orders from last 30 days
SELECT * FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL 30 DAY;

-- Calculate due dates
SELECT 
    invoice_date,
    DATE_ADD(invoice_date, INTERVAL 30 DAY) AS due_date
FROM invoices;
```

### Common Date Patterns

**Today**
```sql
SELECT CURRENT_DATE;
SELECT TODAY();  -- Some databases
SELECT DATE('now');  -- SQLite
```

**Now**
```sql
SELECT CURRENT_TIMESTAMP;
SELECT NOW();  -- MySQL
```

**Start/End of Period**
```sql
-- First day of month
SELECT DATE_TRUNC('month', CURRENT_DATE);

-- Last day of month
SELECT DATE_TRUNC('month', CURRENT_DATE + INTERVAL 1 MONTH) - INTERVAL 1 DAY;

-- First day of year
SELECT DATE_TRUNC('year', CURRENT_DATE);
```

---

## CASE WHEN Expressions

### Overview
Conditional logic in SQL. Similar to IF/ELSE in programming.

### Simple CASE

#### Syntax

```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE result_default
END
```

#### Examples

```sql
-- Map status codes to labels
SELECT 
    order_id,
    status,
    CASE status
        WHEN 'P' THEN 'Pending'
        WHEN 'S' THEN 'Shipped'
        WHEN 'D' THEN 'Delivered'
        WHEN 'C' THEN 'Cancelled'
        ELSE 'Unknown'
    END AS status_label
FROM orders;

-- Grade mapping
SELECT 
    student_id,
    test_score,
    CASE test_score
        WHEN 90 THEN 'A'
        WHEN 80 THEN 'B'
        WHEN 70 THEN 'C'
        WHEN 60 THEN 'D'
        ELSE 'F'
    END AS grade
FROM student_scores;
```

### Searched CASE

#### Syntax

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE result_default
END
```

#### Examples

```sql
-- Categorize employees by salary
SELECT 
    employee_id,
    employee_name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'Executive'
        WHEN salary >= 70000 THEN 'Manager'
        WHEN salary >= 40000 THEN 'Senior Staff'
        WHEN salary >= 25000 THEN 'Staff'
        ELSE 'Entry Level'
    END AS salary_level
FROM employees;

-- Complex conditional logic
SELECT 
    customer_id,
    total_spent,
    order_count,
    CASE
        WHEN total_spent >= 10000 AND order_count >= 20 THEN 'Premium'
        WHEN total_spent >= 5000 OR order_count >= 10 THEN 'Gold'
        WHEN total_spent >= 1000 THEN 'Silver'
        ELSE 'Bronze'
    END AS customer_tier
FROM customer_summary;
```

### Use Cases

**Case 1: Conditional Aggregation**
```sql
SELECT 
    month,
    SUM(CASE WHEN status = 'Completed' THEN amount ELSE 0 END) AS completed_amount,
    SUM(CASE WHEN status = 'Pending' THEN amount ELSE 0 END) AS pending_amount,
    SUM(CASE WHEN status = 'Cancelled' THEN amount ELSE 0 END) AS cancelled_amount,
    COUNT(CASE WHEN status = 'Completed' THEN 1 END) AS completed_count
FROM orders
GROUP BY month;
```

**Case 2: Data Transformation**
```sql
SELECT 
    employee_id,
    employee_name,
    CASE department
        WHEN 'IT' THEN 'Technology'
        WHEN 'HR' THEN 'Human Resources'
        WHEN 'FIN' THEN 'Finance'
        WHEN 'OPS' THEN 'Operations'
        ELSE 'Other'
    END AS department_full_name
FROM employees;
```

**Case 3: Risk Scoring**
```sql
SELECT 
    customer_id,
    total_spent,
    last_purchase_days_ago,
    CASE
        WHEN last_purchase_days_ago > 180 THEN 'High'
        WHEN last_purchase_days_ago > 90 THEN 'Medium'
        WHEN last_purchase_days_ago <= 30 THEN 'Low'
        ELSE 'Medium'
    END AS churn_risk
FROM customers;
```

### Nested CASE

```sql
SELECT 
    order_id,
    CASE
        WHEN order_amount > 1000 THEN
            CASE status
                WHEN 'Completed' THEN 'High Value Completed'
                WHEN 'Pending' THEN 'High Value Pending'
                ELSE 'High Value Other'
            END
        ELSE
            CASE status
                WHEN 'Completed' THEN 'Regular Completed'
                ELSE 'Regular Other'
            END
    END AS order_category
FROM orders;
```

### Common Mistakes

❌ **Mistake 1: CASE in WHERE Instead of HAVING**
```sql
-- WRONG - CASE evaluates, doesn't filter
SELECT 
    department,
    COUNT(*)
FROM employees
WHERE CASE WHEN salary > 50000 THEN 1 END  -- Returns NULL or 1, not boolean
GROUP BY department;

-- CORRECT - use for filtering aggregates
SELECT 
    department,
    COUNT(*) AS emp_count
FROM employees
WHERE salary > 50000
GROUP BY department;
```

❌ **Mistake 2: Different Data Types in THEN/ELSE**
```sql
-- WRONG - mixed types can cause implicit conversion
SELECT 
    CASE status
        WHEN 1 THEN 'Active'
        WHEN 0 THEN 0  -- Returns number instead of text!
    END
FROM users;

-- CORRECT - consistent types
SELECT 
    CASE status
        WHEN 1 THEN 'Active'
        WHEN 0 THEN 'Inactive'
    END
FROM users;
```

❌ **Mistake 3: Missing ELSE**
```sql
-- WRONG - unmapped values return NULL
SELECT 
    status,
    CASE status
        WHEN 'A' THEN 'Active'
        WHEN 'I' THEN 'Inactive'
        -- What about 'S' for Suspended?
    END AS status_label
FROM users;

-- CORRECT - handle all cases
SELECT 
    status,
    CASE status
        WHEN 'A' THEN 'Active'
        WHEN 'I' THEN 'Inactive'
        WHEN 'S' THEN 'Suspended'
        ELSE 'Unknown'
    END AS status_label
FROM users;
```

### Production Tips

✅ **Tip 1: Use CASE for Conditional Counts**
```sql
-- Good - single pass
SELECT 
    COUNT(*) AS total,
    COUNT(CASE WHEN status = 'Active' THEN 1 END) AS active_count,
    COUNT(CASE WHEN status = 'Inactive' THEN 1 END) AS inactive_count
FROM users;

-- Avoid - multiple queries
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM users WHERE status = 'Active';
```

✅ **Tip 2: Order Conditions by Frequency**
```sql
-- Good - most common conditions first (minor perf benefit)
SELECT 
    CASE
        WHEN status = 'Completed' THEN 1  -- Most common
        WHEN status = 'Pending' THEN 2
        WHEN status = 'Cancelled' THEN 3  -- Rare
        ELSE 4
    END AS priority
FROM orders;
```

✅ **Tip 3: Use Meaningful Labels**
```sql
-- Good - descriptive
CASE status
    WHEN 'P' THEN 'Pending - Awaiting Fulfillment'
    WHEN 'S' THEN 'Shipped - In Transit'
    WHEN 'D' THEN 'Delivered - Complete'
ELSE 'Unknown'
END

-- Avoid - ambiguous
CASE status WHEN 'P' THEN '1' WHEN 'S' THEN '2' END
```

---

## Type Casting

### CAST Function

#### Definition
Explicitly converts value from one data type to another.

#### Syntax

```sql
CAST(expression AS target_type)
```

#### Examples

```sql
-- Convert string to number
SELECT CAST('123' AS INT);

-- Convert number to string
SELECT CAST(employee_id AS VARCHAR(10));

-- Convert to date
SELECT CAST('2024-01-15' AS DATE);

-- Convert timestamp to date
SELECT CAST(created_at AS DATE);

-- Truncate decimals
SELECT CAST(price AS INT);
```

#### Use Cases

**Case 1: Data Type Consistency**
```sql
SELECT 
    CAST(order_id AS VARCHAR) AS order_id_str,
    CAST(quantity AS DECIMAL(10,2)) AS quantity_decimal,
    CAST(order_date AS VARCHAR) AS order_date_str
FROM orders;
```

**Case 2: Division with Decimals**
```sql
-- Without CAST: integer division
SELECT 10 / 3;  -- 3

-- With CAST: decimal division
SELECT CAST(10 AS DECIMAL(10,2)) / 3;  -- 3.33
```

**Case 3: Date Extraction**
```sql
SELECT 
    CAST(order_date AS DATE) AS order_date_only,
    CAST(EXTRACT(YEAR FROM order_date) AS VARCHAR) AS year_str
FROM orders;
```

### Dialect Variations

```sql
-- CAST (standard across all databases)
CAST(value AS type)

-- MySQL (:: operator in some functions)
CONVERT(type, value)

-- SQL Server (:: operator sometimes)
CAST(value AS type)

-- PostgreSQL (:: shorthand operator)
value::type

-- Oracle
CAST(value AS type)

-- SQLite
CAST(value AS type)
```

#### Examples by Dialect

```sql
-- PostgreSQL (multiple styles)
SELECT '123'::INT;
SELECT CAST('123' AS INT);

-- MySQL
SELECT CAST('123' AS UNSIGNED);
SELECT CONVERT(INT, '123');

-- SQL Server
SELECT CAST('123' AS INT);
SELECT CONVERT(INT, '123');
```

### TRY_CAST Function

#### Definition
Like CAST but returns NULL instead of error on failure.

#### Syntax

```sql
TRY_CAST(expression AS target_type)
```

#### Support
- **SQL Server**: Full support
- **PostgreSQL**: Not available (use CASE with error handling)
- **MySQL**: Not available (use CASE)
- **Oracle**: Not available

#### Examples

```sql
-- SQL Server
SELECT 
    TRY_CAST('123' AS INT) AS valid_cast,    -- 123
    TRY_CAST('abc' AS INT) AS invalid_cast   -- NULL
FROM table_name;

-- PostgreSQL alternative (using CASE)
SELECT 
    CASE 
        WHEN value ~ '^[0-9]+$' THEN CAST(value AS INT)
        ELSE NULL
    END
FROM table_name;
```

#### Use Cases

**Case 1: Safe Type Conversion**
```sql
-- SQL Server
SELECT 
    id,
    TRY_CAST(user_input AS INT) AS converted_value,
    CASE WHEN TRY_CAST(user_input AS INT) IS NULL 
        THEN 'Invalid Number'
        ELSE 'Valid Number'
    END AS validation_status
FROM user_input_table;
```

**Case 2: Data Validation**
```sql
-- SQL Server
SELECT 
    record_id,
    TRY_CAST(amount AS DECIMAL(10,2)) AS amount,
    CASE 
        WHEN TRY_CAST(amount AS DECIMAL(10,2)) IS NULL THEN 'Invalid Amount'
        WHEN TRY_CAST(amount AS DECIMAL(10,2)) < 0 THEN 'Negative Amount'
        ELSE 'Valid'
    END AS validation_result
FROM raw_data;
```

### Implicit Conversion

#### What Is It?
Automatic type conversion that SQL performs behind the scenes.

#### Examples

```sql
-- String to number (implicit in comparison)
SELECT * FROM orders WHERE order_id = '12345';

-- Number to string (implicit in concatenation)
SELECT 'Order #' + order_id FROM orders;

-- Boolean to integer
WHERE is_active = 1;  -- is_active might be BOOLEAN, 1 is INT
```

#### Risks

❌ **Implicit conversions can cause:**

```sql
-- Unexpected results
SELECT * FROM data WHERE amount = '100.50';  -- Might match 100.50, 100.500, etc.

-- Performance issues (index not used)
SELECT * FROM data WHERE CAST(numeric_id AS VARCHAR) = '123';

-- Data loss
SELECT CAST(3.9 AS INT);  -- Becomes 3, loses 0.9
```

### Common Mistakes

❌ **Mistake 1: Implicit Conversion in Performance-Critical Code**
```sql
-- SLOW - prevents index use
SELECT * FROM orders
WHERE CAST(order_id AS VARCHAR) = '12345';

-- BETTER - explicit type in WHERE
SELECT * FROM orders
WHERE order_id = 12345;
```

❌ **Mistake 2: CAST Changing Data Meaning**
```sql
-- Wrong precision
SELECT CAST(price AS INT) FROM products;
-- Loses cents!

-- Better
SELECT CAST(price AS DECIMAL(10,2)) FROM products;
```

❌ **Mistake 3: Casting in Aggregates**
```sql
-- WRONG - aggregates on cast values
SELECT AVG(CAST(salary AS VARCHAR))  -- Doesn't make sense!
FROM employees;

-- CORRECT - cast result if needed
SELECT CAST(AVG(salary) AS INT)
FROM employees;
```

---

## Production Best Practices

### Window Functions in Production

✅ **Use Window Functions Instead of Aggregates When Detail Rows Needed**

```sql
-- Good - keep row details
SELECT 
    product_id,
    date,
    sales,
    SUM(sales) OVER (
        PARTITION BY product_id
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_sales
FROM sales;

-- Avoid - loses detail rows with GROUP BY
SELECT 
    product_id,
    SUM(sales) AS cumulative_sales
FROM sales
GROUP BY product_id;
```

### CTEs vs Subqueries

✅ **Prefer CTEs for Readability**

```sql
-- Good - clear structure
WITH monthly_data AS (
    SELECT MONTH(order_date) AS month, SUM(amount) AS revenue
    FROM orders
    GROUP BY MONTH(order_date)
),
growth_calc AS (
    SELECT 
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS prev_revenue
    FROM monthly_data
)
SELECT * FROM growth_calc WHERE revenue > prev_revenue;

-- Harder to read
SELECT * FROM (
    SELECT 
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS prev_revenue
    FROM (
        SELECT MONTH(order_date) AS month, SUM(amount) AS revenue
        FROM orders
        GROUP BY MONTH(order_date)
    ) monthly_data
) growth_calc;
```

### String Function Performance

✅ **Minimize String Functions in WHERE**

```sql
-- SLOW
SELECT * FROM customers
WHERE SUBSTRING(phone_number, 1, 3) = '415';

-- BETTER
SELECT * FROM customers
WHERE phone_number LIKE '415%';
```

### Date Function Best Practices

✅ **Use Date Types, Not Strings**

```sql
-- Good
created_date DATE NOT NULL

-- Avoid
created_date VARCHAR(10)

-- Query correctly
SELECT * FROM orders WHERE created_date = '2024-01-15';

-- Not this
SELECT * FROM orders WHERE CAST(created_date AS DATE) = '2024-01-15';
```

---

## Interview Questions

### Window Functions

**Q1: Explain ROW_NUMBER vs RANK vs DENSE_RANK with an example where they differ.**

A: With salaries [50000, 50000, 48000, 48000, 45000]:
- ROW_NUMBER: 1, 2, 3, 4, 5 (always unique)
- RANK: 1, 1, 3, 3, 5 (skips for ties)
- DENSE_RANK: 1, 1, 2, 2, 3 (no gaps)

**Q2: How do you find the top 3 products per category using window functions?**

A: 
```sql
WITH ranked AS (
    SELECT 
        category,
        product_id,
        revenue,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS rank
    FROM products
)
SELECT * FROM ranked WHERE rank <= 3;
```

**Q3: What's the difference between ROWS BETWEEN and RANGE BETWEEN?**

A: ROWS counts physical rows, RANGE uses value comparisons. ROWS BETWEEN 30 PRECEDING = 30 rows back. RANGE BETWEEN INTERVAL 30 DAY = all rows within 30 days.

**Q4: How do you calculate a moving average efficiently?**

A:
```sql
SELECT 
    date,
    price,
    AVG(price) OVER (
        ORDER BY date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS moving_avg_30
FROM prices;
```

**Q5: When would you use LEAD/LAG instead of self-joining?**

A: When you need previous/next row values without duplicating data. LAG/LEAD is cleaner and usually faster than self-joins for this purpose.

### CTEs and Subqueries

**Q6: What's the advantage of CTEs over subqueries?**

A: Readability - CTEs break complex logic into named steps. Reusability - CTEs can be referenced multiple times. Debugging - test CTEs independently.

**Q7: How do you prevent infinite loops in recursive CTEs?**

A: Include a termination condition (WHERE clause) that becomes false at some point, usually checking a depth/level counter.

**Q8: What's the difference between EXISTS and IN?**

A: EXISTS stops after finding first match (better for large sets). IN evaluates entire list. EXISTS handles NULLs better in subquery (NULL doesn't affect result). NOT IN with NULL returns no rows.

**Q9: When should you use scalar subqueries vs window functions?**

A: Scalar subqueries for single context values. Window functions for per-row calculations. Window functions usually faster as they don't execute per row.

**Q10: How do you find rows that don't have matches using NOT EXISTS?**

A:
```sql
SELECT c.* FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

### String & Date Functions

**Q11: How do you extract the domain from email addresses?**

A:
```sql
SELECT SUBSTRING(email, POSITION('@' IN email) + 1) AS domain
FROM users;
```

**Q12: What's the safest way to handle NULLs in string concatenation?**

A:
```sql
SELECT CONCAT(first_name, COALESCE(CONCAT(' ', middle_name), ''), ' ', last_name)
FROM employees;
```

**Q13: How do you calculate age from birthdate?**

A:
```sql
-- SQL Server
SELECT DATEDIFF(YEAR, birth_date, GETDATE()) AS age

-- PostgreSQL
SELECT EXTRACT(YEAR FROM AGE(birth_date))

-- MySQL
SELECT YEAR(CURDATE()) - YEAR(birth_date) AS age
```

**Q14: What's the difference between CAST and TRY_CAST?**

A: CAST throws error if conversion fails. TRY_CAST returns NULL. TRY_CAST is safer but SQL Server only.

**Q15: How would you handle date gaps in time series data?**

A:
```sql
WITH series AS (
    SELECT DATE_TRUNC('day', GENERATE_SERIES(min_date, max_date, '1 day')) AS date
),
actual_data AS (
    SELECT DATE_TRUNC('day', created_at) AS date, COUNT(*) AS count
    FROM events
    GROUP BY DATE_TRUNC('day', created_at)
)
SELECT s.date, COALESCE(a.count, 0) AS count
FROM series s
LEFT JOIN actual_data a ON s.date = a.date;
```

---

## Conclusion

Phase 2 covers advanced SQL concepts critical for data engineering work. Window functions enable complex analytical queries without self-joins. CTEs improve code readability. Subqueries solve specific problems. String and date functions handle real-world data.

**Key Takeaways:**
1. ✅ Use window functions for per-row calculations
2. ✅ Prefer CTEs over nested subqueries
3. ✅ Avoid N+1 query problems with correlated subqueries
4. ✅ Use EXISTS for efficient existence checks
5. ✅ Handle NULLs explicitly in string operations
6. ✅ Use correct date types, not strings
7. ✅ Test complex queries incrementally
8. ✅ Consider performance implications of string functions

---

**Document Version**: 2.0  
**Last Updated**: 2024  
**Author**: Advanced SQL & Data Engineering Team
