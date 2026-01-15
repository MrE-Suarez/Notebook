# SQL Advanced (Sections 09–12)

> **Level:** Advanced  
> **Topics:** Advanced Techniques, Performance Optimization, AI Integration, Projects

---

## 📋 Quick Reference

| Section | Topic | Key Concepts |
|---------|-------|--------------|
| [9](#9-advanced-techniques) | Advanced SQL Techniques | Subqueries, CTEs, Views, Temp Tables, CTAS |
| [10](#10-performance-optimization) | Performance Optimization | Indexes, Execution Plans, Partitions |
| [11](#11-ai--sql) | AI & SQL | ChatGPT Prompts for SQL Development |
| [12](#12-sql-projects) | SQL Projects | Real-world project templates |

---

## 9. Advanced Techniques

### 9.1 Database Architecture Overview

Understanding database architecture helps optimize query performance.

**Database Engine Components:**

```
DATABASE ENGINE
├── Storage Layer
│   ├── Disk Storage (permanent, slow)
│   │   ├── USER → Main database content
│   │   ├── CATALOG → Metadata (schema, tables, columns)
│   │   └── TEMP → Temporary query processing space
│   └── Cache Storage (temporary, fast)
│       └── Recently accessed data
└── Query Processor
    └── Executes, optimizes queries
```

**Data Warehouse Challenges:**
- Redundancy
- Performance issues
- Complexity
- Maintenance overhead
- Database stress
- Security concerns

**Solutions:**
1. **Subqueries** → Break complex logic into steps
2. **CTEs** → Organize and reuse query logic
3. **Views** → Abstract complex queries
4. **Temp Tables** → Store intermediate results
5. **CTAS** → Create tables from queries

---

### 9.2 Subqueries

A **subquery** is a query nested inside another query.

**Flow:**
```
Database Tables → Outer Query → Subquery → Result
```

**Why use subqueries?**
- Break complex logic into readable steps
- Enable dynamic filtering
- Simplify joins and aggregations

---

#### Subquery Categories

**By Correlation:**
| Type | Description |
|------|-------------|
| **Non-correlated** | Independent from outer query |
| **Correlated** | Dependent on outer query (runs row-by-row) |

**By Result Type:**
| Type | Returns | Example Use |
|------|---------|-------------|
| **Scalar** | Single value | Aggregations in SELECT |
| **Row** | Multiple rows (single column) | Use with IN, ANY, ALL |
| **Table** | Multiple columns | Use in FROM clause |

**By Location:**
- SELECT clause
- FROM clause
- JOIN clause
- WHERE clause

---

#### Subqueries in SELECT Clause

**Rule:** Must return a **single value** (scalar subquery).

**Syntax:**
```sql
SELECT 
    column1,
    (SELECT aggregate_function(column) 
     FROM table2 
     WHERE condition) AS alias
FROM table1;
```

**Example:**
```sql
-- Show customer details with total orders
SELECT 
    customer_id,
    first_name,
    (SELECT COUNT(*) 
     FROM orders o 
     WHERE o.customer_id = c.customer_id) AS total_orders
FROM customers c;
```

---

#### Subqueries in FROM Clause

**Most common usage** → Create temporary result set for outer query.

**Syntax:**
```sql
SELECT column1, column2
FROM (
    SELECT column 
    FROM table1 
    WHERE condition
) AS alias;
```

**Example:**
```sql
-- Get average sales per product category
SELECT 
    category,
    AVG(total_sales) AS avg_category_sales
FROM (
    SELECT 
        product_category AS category,
        SUM(sales) AS total_sales
    FROM orders
    GROUP BY product_category
) AS category_totals
GROUP BY category;
```

---

#### Subqueries in JOIN Clause

Prepare and aggregate data before joining.

**Example:**
```sql
-- Show customers with their total order count
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    o.total_orders
FROM customers c
LEFT JOIN (
    SELECT 
        customer_id,
        COUNT(*) AS total_orders
    FROM orders
    GROUP BY customer_id
) o
ON c.customer_id = o.customer_id;
```

---

#### Subqueries in WHERE Clause

**Rule:** Only **scalar** subqueries with comparison operators.

**With Comparison Operators:**
```sql
SELECT column1, column2
FROM table1
WHERE column = (SELECT column FROM table2 WHERE condition);
```

**Example:**
```sql
-- Find products priced above average
SELECT 
    product_id,
    product_name,
    price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

---

**With IN Operator:**

Check if value matches any value in a list.

```sql
SELECT *
FROM orders
WHERE customer_id IN (
    SELECT customer_id
    FROM customers
    WHERE country = 'Germany'
);
```

---

**With ANY / ALL Operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| `ANY` | At least one match | `WHERE salary > ANY (subquery)` |
| `ALL` | All values match | `WHERE salary > ALL (subquery)` |

**Example:**
```sql
-- Female employees earning more than any male employee
SELECT 
    employee_id,
    first_name,
    gender,
    salary
FROM employees
WHERE gender = 'F'
    AND salary > ANY (
        SELECT salary 
        FROM employees 
        WHERE gender = 'M'
    );
```

---

#### Correlated Subqueries

A **correlated subquery** references columns from the outer query. Executes **once per row** in the outer query.

**Example:**
```sql
-- Show customers with their total orders
SELECT 
    *,
    (SELECT COUNT(*) 
     FROM orders o 
     WHERE o.customer_id = c.customer_id) AS total_orders
FROM customers c;
```

**Execution Flow:**
1. Outer query processes row 1
2. Subquery runs with data from row 1
3. Result added to row 1
4. Repeat for all rows

---

**With EXISTS Operator:**

Check if subquery returns any rows (more efficient than IN for large datasets).

**Syntax:**
```sql
SELECT column1, column2
FROM table1
WHERE EXISTS (
    SELECT 1 
    FROM table2 
    WHERE table1.id = table2.id
);
```

**Example:**
```sql
-- Orders from German customers
SELECT *
FROM orders o
WHERE EXISTS (
    SELECT 1
    FROM customers c
    WHERE c.country = 'Germany'
        AND o.customer_id = c.customer_id
);
```

> **💡 Performance Tip:** `EXISTS` is faster than `IN` for large datasets because it stops at first match.

---

### 9.3 Common Table Expressions (CTEs)

A **CTE** is a temporary named result set that exists only within a single query.

**Benefits:**
- Improves readability
- Enables reuse within query
- Supports recursion
- Easier to debug than nested subqueries

**Limitations:**
- Cannot use `ORDER BY` in CTE definition
- Recommendation: Avoid using more than 5 CTEs

---

#### Non-Recursive CTE

Execute once, used for organizing complex queries.

---

##### Standalone CTE

Independent CTE used by the main query.

**Syntax:**
```sql
WITH CTE_Name AS (
    SELECT ...
    FROM ...
    WHERE ...
)
SELECT ...
FROM CTE_Name
WHERE ...;
```

**Example:**
```sql
-- Calculate total sales per customer
WITH CTE_Total_Sales AS (
    SELECT 
        customer_id,
        SUM(sales) AS total_sales
    FROM orders
    GROUP BY customer_id
)
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    ts.total_sales
FROM customers c
LEFT JOIN CTE_Total_Sales ts
    ON c.customer_id = ts.customer_id;
```

---

##### Multiple CTEs

Define multiple CTEs in a single query (use one `WITH`, separate with commas).

**Syntax:**
```sql
WITH CTE_1 AS (
    SELECT ...
),
CTE_2 AS (
    SELECT ...
),
CTE_3 AS (
    SELECT ...
)
SELECT ...
FROM CTE_1
JOIN CTE_2 ...
JOIN CTE_3 ...;
```

**Example:**
```sql
-- Customer analysis with multiple metrics
WITH CTE_Total_Sales AS (
    SELECT 
        customer_id,
        SUM(sales) AS total_sales
    FROM orders
    GROUP BY customer_id
),
CTE_Last_Order AS (
    SELECT 
        customer_id,
        MAX(order_date) AS last_order
    FROM orders
    GROUP BY customer_id
)
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    ts.total_sales,
    lo.last_order
FROM customers c
LEFT JOIN CTE_Total_Sales ts
    ON c.customer_id = ts.customer_id
LEFT JOIN CTE_Last_Order lo
    ON c.customer_id = lo.customer_id;
```

---

##### Nested CTE

CTE that references another CTE.

**Syntax:**
```sql
WITH CTE_1 AS (
    SELECT ...
),
CTE_2 AS (
    SELECT ...
    FROM CTE_1  -- References CTE_1
)
SELECT ...
FROM CTE_2;
```

**Complex Example:**
```sql
-- Customer segmentation with ranking
WITH CTE_Total_Sales AS (
    SELECT 
        customer_id,
        SUM(sales) AS total_sales
    FROM orders
    GROUP BY customer_id
),
CTE_Last_Order AS (
    SELECT 
        customer_id,
        MAX(order_date) AS last_order
    FROM orders
    GROUP BY customer_id
),
CTE_Customer_Rank AS (
    -- Nested: Uses CTE_Total_Sales
    SELECT 
        customer_id,
        total_sales,
        RANK() OVER(ORDER BY total_sales DESC) AS customer_rank
    FROM CTE_Total_Sales
),
CTE_Segment AS (
    -- Nested: Uses CTE_Total_Sales
    SELECT 
        customer_id,
        CASE
            WHEN total_sales >= 100 THEN 'Great'
            WHEN total_sales >= 90 THEN 'Good'
            WHEN total_sales >= 50 THEN 'OK'
            ELSE 'Bad'
        END AS segment
    FROM CTE_Total_Sales
)
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    ts.total_sales,
    lo.last_order,
    cr.customer_rank,
    s.segment
FROM customers c
LEFT JOIN CTE_Total_Sales ts ON c.customer_id = ts.customer_id
LEFT JOIN CTE_Last_Order lo ON c.customer_id = lo.customer_id
LEFT JOIN CTE_Customer_Rank cr ON c.customer_id = cr.customer_id
LEFT JOIN CTE_Segment s ON c.customer_id = s.customer_id;
```

---

#### Recursive CTE

Self-referencing query that repeats until a condition is met.

**Syntax:**
```sql
WITH CTE_Name AS (
    -- Anchor Query (base case)
    SELECT ...
    FROM ...
    WHERE ...
    
    UNION ALL
    
    -- Recursive Query (references CTE itself)
    SELECT ...
    FROM CTE_Name
    WHERE [break_condition]
)
SELECT *
FROM CTE_Name
OPTION (MAXRECURSION n);  -- SQL Server: Control iterations
```

---

**Example 1: Number Series**
```sql
-- Generate numbers 1 to 100
WITH Series AS (
    -- Anchor: Start with 1
    SELECT 1 AS my_number
    
    UNION ALL
    
    -- Recursive: Add 1 each iteration
    SELECT my_number + 1
    FROM Series
    WHERE my_number < 100
)
SELECT *
FROM Series
OPTION (MAXRECURSION 150);
```

---

**Example 2: Employee Hierarchy**
```sql
-- Build organizational hierarchy
WITH CTE_emp_hierarchy AS (
    -- Anchor: Top-level employees (no manager)
    SELECT 
        employee_id,
        first_name,
        manager_id,
        1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Add subordinates
    SELECT 
        e.employee_id,
        e.first_name,
        e.manager_id,
        level + 1
    FROM employees e
    INNER JOIN CTE_emp_hierarchy ceh
        ON e.manager_id = ceh.employee_id
)
SELECT *
FROM CTE_emp_hierarchy
ORDER BY level, first_name;
```

**Use Cases:**
- Organizational charts
- Bill of materials (BOM)
- Category trees
- File system structures

---

### 9.4 Views

A **view** is a virtual table based on a SQL query. It does not store data physically.

**Database Structure:**
```
SQL SERVER
└── DATABASE
    └── SCHEMA
        ├── TABLES
        │   ├── Columns
        │   └── Keys
        ├── VIEWS
        │   └── Columns (virtual)
        └── Other Objects
```

---

#### Database Architecture Layers

| Layer | Description | Users |
|-------|-------------|-------|
| **Physical (Internal)** | Storage: files, partitions, logs, blocks | DBA |
| **Logical (Conceptual)** | Structure: tables, relationships, indexes | Data Engineers |
| **View (External)** | User-specific views of data | Analysts, Business Users |

---

#### View Use Cases

| Use Case | Description |
|----------|-------------|
| **1. Centralize Logic** | Store complex queries for reuse |
| **2. Simplify Access** | Hide complexity from end users |
| **3. Security** | Restrict access to specific columns |
| **4. Flexibility** | Dynamic data presentation |
| **5. Multi-language** | Support international teams |
| **6. Data Marts** | Virtual data marts in DWH |

---

#### CTE vs View vs Table

| Feature | CTE | View | Table |
|---------|-----|------|-------|
| **Definition** | Temporary result set in query | Saved query in database | Physical data storage |
| **Persistence** | Query execution only | Persistent | Persistent |
| **Storage** | No physical storage | No physical storage | Physical storage |
| **Lifetime** | One query | Until dropped | Until dropped |
| **Performance** | Recomputed every time | Recomputed every time | Fastest (data stored) |
| **Indexable** | ❌ No | ❌ No (unless materialized) | ✅ Yes |
| **Reusable** | ❌ No | ✅ Yes | ✅ Yes |
| **Recursion** | ✅ Yes | ❌ No | ❌ N/A |
| **Security** | None | ✅ Column-level | ✅ Full control |
| **Best For** | Breaking complex queries | Reusing logic | Storing data |

---

#### View Syntax

##### CREATE VIEW

**Syntax:**
```sql
CREATE VIEW schema_name.view_name AS
(
    SELECT ...
    FROM ...
    WHERE ...
);
```

**Example:**
```sql
-- Monthly sales summary view
CREATE VIEW Sales.V_Monthly_Summary AS
(
    SELECT 
        DATETRUNC(month, order_date) AS order_month,
        SUM(sales) AS total_sales,
        COUNT(order_id) AS total_orders,
        SUM(quantity) AS total_quantities
    FROM Sales.Orders
    GROUP BY DATETRUNC(month, order_date)
);
```

**Using the View:**
```sql
SELECT 
    order_month,
    total_sales,
    SUM(total_sales) OVER(ORDER BY order_month) AS running_total
FROM Sales.V_Monthly_Summary;
```

---

##### UPDATE View Logic

**PostgreSQL:**
```sql
CREATE OR REPLACE VIEW Sales.V_Monthly_Summary AS ...
```

**SQL Server:**
```sql
-- Option 1: Drop and recreate
DROP VIEW Sales.V_Monthly_Summary;
GO
CREATE VIEW Sales.V_Monthly_Summary AS ...

-- Option 2: T-SQL with existence check
IF OBJECT_ID('Sales.V_Monthly_Summary', 'V') IS NOT NULL
    DROP VIEW Sales.V_Monthly_Summary;
GO
CREATE VIEW Sales.V_Monthly_Summary AS ...
```

---

### 9.5 CTAS & Temp Tables

**CTAS** = CREATE TABLE AS SELECT

Create a new table based on query results.

---

#### Table Types

| Type | Persistence | Use Case |
|------|-------------|----------|
| **Permanent** | Until dropped | Production tables |
| **Temporary** | Session only | Intermediate results |

---

#### Creating Permanent Tables

**Method 1: CREATE + INSERT**
```sql
-- Define structure
CREATE TABLE table_name (
    id INT,
    name VARCHAR(50)
);

-- Insert data
INSERT INTO table_name VALUES (1, 'Frank');
```

---

**Method 2: CTAS (Most SQL Engines)**
```sql
CREATE TABLE new_table AS
(
    SELECT ...
    FROM ...
    WHERE ...
);
```

---

**Method 3: SELECT INTO (SQL Server Shortcut)**
```sql
SELECT ...
INTO new_table
FROM ...
WHERE ...;
```

**Example:**
```sql
-- Create monthly orders summary table
SELECT 
    DATENAME(month, order_date) AS order_month,
    COUNT(order_id) AS total_orders
INTO Sales.Monthly_Orders
FROM Sales.Orders
GROUP BY DATENAME(month, order_date);
```

---

#### CTAS Use Cases

**1. Optimize Performance**
When views are too slow, create physical tables.

**2. Create Snapshots**
Preserve data state for historical analysis.

**3. Physical Data Marts**
Improve performance with materialized tables instead of views.

---

#### Refresh CTAS Tables

**Option 1: Drop and Recreate**
```sql
DROP TABLE Sales.Monthly_Orders;

SELECT ...
INTO Sales.Monthly_Orders
FROM ...;
```

**Option 2: T-SQL with Check**
```sql
IF OBJECT_ID('Sales.Monthly_Orders', 'U') IS NOT NULL
    DROP TABLE Sales.Monthly_Orders;
GO

SELECT ...
INTO Sales.Monthly_Orders
FROM ...;
```

---

#### Temporary Tables

Store intermediate results during session (automatically dropped when session ends).

**Syntax (SQL Server):**
```sql
SELECT ...
INTO #temp_table_name  -- Hash (#) makes it temporary
FROM ...
WHERE ...;
```

**Example:**
```sql
-- Create temp table for ETL process
SELECT 
    customer_id,
    first_name,
    last_name,
    UPPER(country) AS country_clean
INTO #cleaned_customers
FROM staging_customers
WHERE registration_date >= '2024-01-01';

-- Use temp table
SELECT *
FROM #cleaned_customers
WHERE country_clean = 'USA';
```

**Use Case:**
ETL workflows where you extract, transform, then load into permanent tables.

---

### 9.6 Stored Procedures (Overview)

**Stored Procedures** are precompiled SQL programs stored in the database.

**Advantages:**
- ✅ High performance (execution plan caching)
- ✅ Enhanced security (permission abstraction)
- ✅ Consistent business rules
- ✅ Reduced network overhead

**Disadvantages:**
- ❌ Harder to version control
- ❌ Difficult to test/debug
- ❌ Vendor-specific
- ❌ Not ideal for complex business logic

**When to Use:**
- Data manipulation inside database
- Performance-critical operations
- Transactional integrity

**When NOT to Use:**
- ETL orchestration (use Python/Airflow)
- Complex business logic (use application layer)
- Cross-database operations

> **Note:** Stored procedures are becoming less common in modern data architectures. Focus on CTEs and Views for most use cases.

---

### 9.7 Triggers (Overview)

**Triggers** are special stored procedures that automatically execute in response to events.

#### Trigger Types

**DML Triggers:**
- INSERT
- UPDATE
- DELETE

**Timing:**
- `AFTER` → Runs after the event
- `INSTEAD OF` → Replaces the event

**DDL Triggers:**
- CREATE
- ALTER
- DROP

**Logon Triggers:**
- Execute on user login

---

**Syntax:**
```sql
CREATE TRIGGER trigger_name ON table_name
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    -- SQL statements
END;
```

**Example:**
```sql
-- Log new employee additions
CREATE TRIGGER trg_AfterInsertEmployee ON Sales.Employees
AFTER INSERT
AS
BEGIN
    INSERT INTO Sales.EmployeeLogs (employee_id, log_message, log_date)
    SELECT 
        employee_id,
        'New Employee Added = ' + CAST(employee_id AS VARCHAR),
        GETDATE()
    FROM INSERTED;
END;
```

> **Note:** Triggers add complexity and can impact performance. Use sparingly and document thoroughly.

---

## 🎯 Key Takeaways - Advanced Techniques

### Subqueries
- ✅ Break complex logic into manageable steps
- ✅ Use EXISTS instead of IN for large datasets
- ⚠️ Correlated subqueries run once per row (can be slow)

### CTEs
- ✅ Improve query readability and maintainability
- ✅ Enable recursion for hierarchical data
- ⚠️ Limit to 5 CTEs per query

### Views
- ✅ Abstract complex queries for end users
- ✅ Enhance security with column-level access
- ⚠️ May impact performance (consider CTAS for large datasets)

### Temp Tables & CTAS
- ✅ Store intermediate results for complex workflows
- ✅ Create snapshots for historical analysis
- ✅ Materialize views for better performance

---

> **Next:** Continue to Part 2 of Advanced topics for Performance Optimization.
