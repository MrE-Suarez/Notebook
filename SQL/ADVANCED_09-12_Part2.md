# SQL Advanced - Part 2

## 10. Performance Optimization

### 10.1 Indexes

Indexes are data structures that optimize data retrieval speed.

**Concept:** Like a book index pointing to specific page numbers, database indexes point to data locations.

---

#### Index Structure

##### Clustered Index

**Definition:** Defines the **physical order** of rows in a table.

**Structure:**
- Stored as B-Tree (Balanced Tree)
- Root node → Index pages → Data pages (leaf level)
- Leaf level contains **actual table rows** (sorted)
- **Only ONE clustered index per table**

**Diagram:**
```
        Root Node
       /    |    \
   Index  Index  Index
     |      |      |
   Data   Data   Data
   Page   Page   Page
```

**Characteristics:**
- ✅ Leaf = actual data rows
- ✅ Rows physically sorted by index key
- ✅ Faster range queries
- ❌ Only one per table

---

##### Non-Clustered Index

**Definition:** Separate B-Tree structure with **pointers** to data.

**Structure:**
- Stored as separate B-Tree
- Leaf level contains **pointers** (not data)
- Pointers can be:
  - **RID** (Row ID) for heap tables
  - **Clustered key** if table has clustered index

**Diagram:**
```
        Root Node
       /    |    \
   Index  Index  Index
     |      |      |
  Pointer Pointer Pointer
     ↓      ↓      ↓
   Data   Data   Data
   Page   Page   Page
```

**Characteristics:**
- ✅ Leaf = pointers to data
- ✅ Can have **multiple** per table
- ✅ Good for exact match lookups
- ❌ Slightly slower than clustered (pointer hop)

---

##### Clustered vs Non-Clustered Comparison

| Feature | Clustered | Non-Clustered |
|---------|-----------|---------------|
| **Quantity** | One per table | Many per table |
| **Storage** | Data pages | Pointer pages |
| **Physical Order** | Yes | No |
| **Speed (Reads)** | Faster for ranges | Faster for exact matches |
| **Speed (Writes)** | Slower (reorganize data) | Faster (update pointers) |
| **Analogy** | Table of contents | Book index |

---

**Best Practices:**

**Clustered Index:**
- ✅ Primary keys
- ✅ Unique or mostly unique columns
- ✅ Columns not frequently modified
- ✅ Range queries (dates, IDs)

**Non-Clustered Index:**
- ✅ Foreign keys
- ✅ Columns in JOIN conditions
- ✅ Columns in WHERE clauses
- ✅ Exact match lookups

---

#### Index Syntax

##### Basic Index Creation

**Syntax:**
```sql
CREATE [CLUSTERED | NONCLUSTERED] INDEX index_name
ON table_name (column1 [ASC|DESC], column2, ...);
```

**Examples:**
```sql
-- Clustered index on primary key
CREATE CLUSTERED INDEX IX_Customers_ID
ON Customers (customer_id);

-- Non-clustered index (default)
CREATE INDEX IX_Customers_Name
ON Customers (last_name ASC, first_name DESC);

-- Non-clustered on frequently queried column
CREATE INDEX idx_DBCustomer_FirstName
ON Sales.DBCustomers (first_name);
```

---

##### Composite Index

Index with multiple columns (column order matters!).

**Rule:** Query must match **leftmost columns** of index.

**Example:**
```sql
-- Frequent query
WHERE country = 'USA' AND score > 500

-- Matching index
CREATE INDEX idx_DBCustomers_CountryScore
ON DBCustomers (country, score);
```

**How SQL Uses Composite Indexes:**
- ✅ Works: `WHERE country = 'USA'` (leftmost column)
- ⚠️ Partial: `WHERE country = 'USA' AND city = 'NYC'` (city not in index)
- ❌ Doesn't work: `WHERE score > 500` (not leftmost column)

---

#### Index Storage Types

##### Rowstore vs Columnstore

| Feature | Rowstore | Columnstore |
|---------|----------|-------------|
| **Storage** | Row by row | Column by column |
| **Compression** | Less efficient | High compression |
| **Read Speed** | Fair | ⚡ Very fast |
| **Write Speed** | Fair | Slower |
| **I/O Efficiency** | Lower (retrieves all columns) | Higher (retrieves specific columns) |
| **Best For** | OLTP (transactions) | OLAP (analytics) |
| **Use Case** | Banking, e-commerce | Data warehouse, BI |

---

**Columnstore Process:**
1. Separate data into row groups
2. Create column segments
3. Compress using dictionary encoding
4. Store in data pages
5. Define as CLUSTERED or NONCLUSTERED

---

##### Columnstore Index Syntax

**Syntax:**
```sql
CREATE [CLUSTERED | NONCLUSTERED] COLUMNSTORE INDEX index_name
ON table_name;
-- Note: Do not specify columns for columnstore
```

**Example:**
```sql
CREATE CLUSTERED COLUMNSTORE INDEX idx_Orders_Columnstore
ON Sales.Orders;
```

**Limitations (SQL Server):**
- Only 1 clustered OR 1 nonclustered columnstore per table
- Rowstore can have: 1 clustered + multiple nonclustered

---

##### Unique Index

Enforces uniqueness constraint and slightly improves performance.

**Syntax:**
```sql
CREATE UNIQUE [CLUSTERED | NONCLUSTERED] INDEX index_name
ON table_name (column1, column2, ...);
```

**Example:**
```sql
CREATE UNIQUE NONCLUSTERED INDEX idx_Products_ProductName
ON Sales.Products (product_name);
```

**Characteristics:**
- ✅ Enforces uniqueness
- ✅ Slightly faster reads
- ❌ Slower writes (uniqueness check)

---

##### Filtered Index

Index that includes only rows meeting specific conditions.

**Syntax:**
```sql
CREATE [NONCLUSTERED] INDEX index_name
ON table_name (column1, column2, ...)
WHERE condition;
```

**Example:**
```sql
-- Frequently filter USA customers
CREATE NONCLUSTERED INDEX idx_Customers_USA
ON Sales.Customers (customer_id, country)
WHERE country = 'USA';
```

**SQL Server Limitations:**
- ❌ Cannot create filtered clustered index
- ❌ Cannot create filtered columnstore index

**Benefits:**
- ✅ Smaller index size
- ✅ Faster queries for filtered data
- ✅ Reduced maintenance overhead

---

#### Choosing the Right Index

| Index Type | Use Case |
|------------|----------|
| **HEAP** (no index) | Staging tables |
| **CLUSTERED** | Primary keys, OLTP systems |
| **COLUMNSTORE** | Analytical queries, OLAP/DWH |
| **NONCLUSTERED** | Foreign keys, JOIN/WHERE columns |
| **FILTERED** | Queries with consistent WHERE conditions |
| **UNIQUE** | Enforce uniqueness (emails, usernames) |

---

### 10.2 Index Management

Indexes require ongoing maintenance for optimal performance.

---

#### 1. Monitor Index Usage

**Check Index Metadata:**
```sql
-- Quick index info
EXEC sp_helpindex 'table_name';

-- Detailed index metadata
SELECT 
    tbl.name AS table_name,
    idx.name AS index_name,
    idx.type_desc AS index_type,
    idx.is_primary_key,
    idx.is_unique,
    idx.is_disabled
FROM sys.indexes idx
JOIN sys.tables tbl
    ON idx.object_id = tbl.object_id
ORDER BY tbl.name, idx.name;
```

---

**Check Index Usage Statistics:**
```sql
SELECT 
    tbl.name AS table_name,
    idx.name AS index_name,
    idx.type_desc AS index_type,
    s.user_seeks,
    s.user_scans,
    s.user_lookups,
    s.user_updates,
    COALESCE(s.last_user_seek, s.last_user_scan) AS last_used
FROM sys.indexes idx
JOIN sys.tables tbl
    ON idx.object_id = tbl.object_id
LEFT JOIN sys.dm_db_index_usage_stats s
    ON s.object_id = idx.object_id
    AND s.index_id = idx.index_id
WHERE tbl.name = 'your_table_name'
ORDER BY s.user_seeks DESC;
```

**Decision Points:**
- **Unused indexes** → Consider dropping (saves storage, improves writes)
- **High user_updates, low seeks** → Index may hurt more than help

---

#### 2. Monitor Missing Indexes

SQL Server provides recommendations for missing indexes.

```sql
SELECT 
    OBJECT_NAME(mid.object_id) AS table_name,
    mid.equality_columns,
    mid.inequality_columns,
    mid.included_columns,
    migs.avg_user_impact,
    migs.user_seeks,
    migs.user_scans
FROM sys.dm_db_missing_index_details mid
JOIN sys.dm_db_missing_index_groups mig
    ON mid.index_handle = mig.index_handle
JOIN sys.dm_db_missing_index_group_stats migs
    ON mig.index_group_handle = migs.group_handle
ORDER BY migs.avg_user_impact DESC;
```

**Action:** Create indexes for high-impact recommendations.

---

#### 3. Monitor Duplicate Indexes

Identify indexes on the same columns.

```sql
SELECT 
    tbl.name AS table_name,
    col.name AS indexed_column,
    idx.name AS index_name,
    idx.type_desc,
    COUNT(*) OVER(PARTITION BY tbl.name, col.name) AS index_count
FROM sys.indexes idx
JOIN sys.tables tbl
    ON idx.object_id = tbl.object_id
JOIN sys.index_columns ic
    ON idx.object_id = ic.object_id
    AND idx.index_id = ic.index_id
JOIN sys.columns col
    ON ic.object_id = col.object_id
    AND ic.column_id = col.column_id
WHERE idx.type > 0  -- Exclude heap
ORDER BY tbl.name, col.name;
```

**Action:** Remove duplicate indexes, keep the most appropriate type.

---

#### 4. Update Statistics

Database engines use statistics to choose optimal execution plans.

**Check Statistics Freshness:**
```sql
SELECT 
    SCHEMA_NAME(t.schema_id) AS schema_name,
    t.name AS table_name,
    s.name AS statistics_name,
    sp.last_updated,
    DATEDIFF(day, sp.last_updated, GETDATE()) AS days_old,
    sp.rows,
    sp.modification_counter AS changes_since_update
FROM sys.stats s
JOIN sys.tables t
    ON s.object_id = t.object_id
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
ORDER BY sp.modification_counter DESC;
```

---

**Update Statistics:**
```sql
-- Single statistic
UPDATE STATISTICS Sales.Employees PK_employees;

-- All statistics on a table
UPDATE STATISTICS Sales.Employees;

-- All statistics in database (can be slow!)
EXEC sp_updatestats;
```

**Recommendations:**
- ✅ Schedule weekly updates (weekends for large DBs)
- ✅ Update after bulk data loads
- ✅ Update after data migrations

---

#### 5. Monitor and Fix Fragmentation

**Fragmentation** = Unused spaces in data pages or pages out of order.

**Check Fragmentation:**
```sql
SELECT 
    tbl.name AS table_name,
    idx.name AS index_name,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(
    DB_ID(), 
    NULL, 
    NULL, 
    NULL, 
    'LIMITED'
) AS ips
INNER JOIN sys.tables tbl
    ON ips.object_id = tbl.object_id
INNER JOIN sys.indexes idx
    ON idx.object_id = ips.object_id
    AND idx.index_id = ips.index_id
WHERE ips.avg_fragmentation_in_percent > 10
ORDER BY ips.avg_fragmentation_in_percent DESC;
```

**Fragmentation Scale:**
- **0%** → No fragmentation
- **100%** → Completely fragmented

---

**Fix Fragmentation:**

**REORGANIZE (Light operation):**
```sql
ALTER INDEX index_name ON table_name REORGANIZE;
```
- Defragments leaf nodes
- Online operation
- Less resource-intensive

**REBUILD (Heavy operation):**
```sql
ALTER INDEX index_name ON table_name REBUILD;
```
- Recreates index from scratch
- Can be offline (blocks access)
- More resource-intensive

---

**Fragmentation Guidelines:**

| Fragmentation | Action | Method |
|---------------|--------|--------|
| < 10% | None | No action needed |
| 10-30% | Light | REORGANIZE |
| > 30% | Heavy | REBUILD |

**Best Practice:** Schedule maintenance windows for REBUILD operations.

---

### 10.3 Execution Plans

An **execution plan** is the database's roadmap for executing a query.

**Types:**

| Type | Description | Use Case |
|------|-------------|----------|
| **Estimated** | Predicted plan (no execution) | Query analysis |
| **Actual** | Real plan after execution | Performance troubleshooting |
| **Live** | Real-time plan during execution | Long-running queries |

---

#### Reading Execution Plans

**Direction:** Read **right to left** (bottom to top).

**Key Components to Check:**
1. **CPU cost** → Which operations consume most CPU
2. **Scan types** → How data is accessed
3. **Join algorithms** → How tables are combined

---

##### Scan Types

| Type | Description | Performance |
|------|-------------|-------------|
| **Table Scan** | Reads entire table row-by-row | ❌ Slowest |
| **Index Scan** | Scans all rows in index | ⚠️ Moderate |
| **Index Seek** | Quickly locates specific rows | ✅ Fastest |

**Goal:** Convert scans to seeks when possible (add indexes).

---

##### Join Algorithms

| Algorithm | Description | Best For |
|-----------|-------------|----------|
| **Nested Loops** | Row-by-row comparison | Small tables |
| **Hash Match** | Uses hash table | Large tables |
| **Merge Join** | Merges two sorted inputs | Both inputs sorted |

---

##### Estimated vs Actual Comparison

**Mismatch Indicators:**
- Estimated rows: 100
- Actual rows: 10,000

**Causes:**
- Outdated statistics
- Missing indexes
- Data skew

**Action:** Update statistics or add missing indexes.

---

#### Comparing Execution Plans

1. Save execution plan (right-click → Save)
2. Open second query execution plan
3. Right-click → Compare Execution Plans
4. Analyze differences in cost and operations

**Use Case:** Compare before/after optimization.

---

#### SQL Hints

Force specific execution behavior (use sparingly!).

**Force JOIN Algorithm:**
```sql
SELECT 
    o.sales,
    c.country
FROM Sales.Orders o
LEFT JOIN Sales.Customers c
    ON o.customer_id = c.customer_id
OPTION (HASH JOIN);  -- Force hash join
```

**Force Index Usage:**
```sql
SELECT 
    o.sales,
    c.country
FROM Sales.Orders o
LEFT JOIN Sales.Customers c WITH (INDEX(idx_customers_id))
    ON o.customer_id = c.customer_id;
```

**Force Index Seek:**
```sql
SELECT *
FROM Sales.Customers WITH (FORCESEEK);
```

**⚠️ Warnings:**
- Hints are **workarounds**, not solutions
- Test in DEV and PROD (performance may vary)
- Find and fix root cause (missing index, outdated statistics)

---

### 10.4 Index Strategy

**Golden Rule:** **Avoid over-indexing** (confuses query optimizer).

---

#### 1. Initial Strategy

**Determine System Type:**

**OLAP (Analytical):**
- Goal: Optimize **READ** performance
- Index: **Columnstore**

**OLTP (Transactional):**
- Goal: Optimize **WRITE** performance
- Index: **Clustered on primary key**

---

#### 2. Usage Pattern Indexing

**Process:**
1. Identify frequently queried tables and columns
2. Choose appropriate index type
3. Test and measure performance

**Analysis:**
- Check query logs
- Review slow query reports
- Ask users about common queries

---

#### 3. Scenario-Based Indexing

**Process:**
1. Identify slow queries (execution time > threshold)
2. Review execution plans
3. Add recommended indexes
4. Compare execution plans (before/after)

**Tools:**
- Query Store (SQL Server)
- Slow query log (MySQL)
- pg_stat_statements (PostgreSQL)

---

#### 4. Ongoing Monitoring & Maintenance

**Weekly Tasks:**
1. ✅ Monitor index usage frequency
2. ✅ Check for missing indexes
3. ✅ Identify duplicate indexes
4. ✅ Update statistics
5. ✅ Monitor fragmentation

**Monthly Tasks:**
1. ✅ Review and remove unused indexes
2. ✅ Reorganize fragmented indexes (10-30%)
3. ✅ Rebuild heavily fragmented indexes (>30%)

---

### 10.5 Table Partitioning

**Partitioning** = Dividing a large table into smaller, manageable pieces.

**Benefits:**
- ✅ Improved query performance
- ✅ Easier maintenance
- ✅ Better data management
- ✅ Parallel processing

---

#### Partitioning Setup (SQL Server)

**Step 1: Create Partition Function**

Define how to split data (by date, ID, region, etc.).

```sql
CREATE PARTITION FUNCTION PartitionByYear (DATE)
AS RANGE LEFT FOR VALUES ('2023-12-31', '2024-12-31', '2025-12-31');
```

**Verify:**
```sql
SELECT 
    name,
    function_id,
    type_desc,
    boundary_value_on_right
FROM sys.partition_functions;
```

---

**Step 2: Create File Groups**

Logical containers for data files.

```sql
ALTER DATABASE SalesDB ADD FILEGROUP FG_2023;
ALTER DATABASE SalesDB ADD FILEGROUP FG_2024;
ALTER DATABASE SalesDB ADD FILEGROUP FG_2025;
ALTER DATABASE SalesDB ADD FILEGROUP FG_2026;
```

**Verify:**
```sql
SELECT *
FROM sys.filegroups
WHERE type = 'FG';
```

---

**Step 3: Create Data Files**

Physical files for each filegroup.

```sql
ALTER DATABASE SalesDB ADD FILE
(
    NAME = P_2023,
    FILENAME = 'C:\SQLData\P_2023.ndf'
) TO FILEGROUP FG_2023;

-- Repeat for other years
```

**Verify:**
```sql
SELECT 
    fg.name AS filegroup_name,
    mf.name AS file_name,
    mf.physical_name AS file_path,
    mf.size / 128 AS size_mb
FROM sys.filegroups fg
JOIN sys.master_files mf
    ON fg.data_space_id = mf.data_space_id
WHERE mf.database_id = DB_ID('SalesDB');
```

---

**Step 4: Create Partition Scheme**

Map filegroups to partition function.

```sql
CREATE PARTITION SCHEME SchemePartitionByYear
AS PARTITION PartitionByYear
TO (FG_2023, FG_2024, FG_2025, FG_2026);
```

**Verify:**
```sql
SELECT 
    ps.name AS scheme_name,
    pf.name AS function_name,
    ds.destination_id AS partition_number,
    fg.name AS filegroup_name
FROM sys.partition_schemes ps
JOIN sys.partition_functions pf
    ON ps.function_id = pf.function_id
JOIN sys.destination_data_spaces ds
    ON ps.data_space_id = ds.partition_scheme_id
JOIN sys.filegroups fg
    ON ds.data_space_id = fg.data_space_id;
```

---

**Step 5: Create Partitioned Table**

```sql
CREATE TABLE Sales.Orders_Partitioned (
    order_id INT,
    order_date DATE,
    sales DECIMAL(10,2)
) ON SchemePartitionByYear (order_date);
```

**Insert Data:**
```sql
INSERT INTO Sales.Orders_Partitioned
VALUES (1, '2023-05-15', 100.00);
```

**Verify Data Distribution:**
```sql
SELECT 
    p.partition_number,
    f.name AS filegroup,
    p.rows AS row_count
FROM sys.partitions p
JOIN sys.destination_data_spaces dds
    ON p.partition_number = dds.destination_id
JOIN sys.filegroups f
    ON dds.data_space_id = f.data_space_id
WHERE OBJECT_NAME(p.object_id) = 'Orders_Partitioned';
```

---

### 10.6 Performance Best Practices

**Summary of 30 Optimization Tips**

---

#### Fetching Data (Tips 1-3)

**1. Select Only Needed Columns**
```sql
-- ❌ Bad
SELECT * FROM customers;

-- ✅ Good
SELECT customer_id, first_name, last_name FROM customers;
```

**2. Avoid Unnecessary DISTINCT/ORDER BY**
```sql
-- Only use if truly needed
SELECT DISTINCT country FROM customers;  -- Use only if duplicates exist
```

**3. Limit Rows for Exploration**
```sql
SELECT TOP 100 * FROM orders;  -- SQL Server
SELECT * FROM orders LIMIT 100;  -- MySQL/PostgreSQL
```

---

#### Filtering Data (Tips 4-7)

**4. Index Frequently Filtered Columns**
```sql
CREATE NONCLUSTERED INDEX idx_customers_country
ON customers (country);
```

**5. Avoid Functions on Indexed Columns**
```sql
-- ❌ Bad (index not used)
WHERE YEAR(order_date) = 2024

-- ✅ Good (index used)
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'
```

**6. Avoid Leading Wildcards**
```sql
-- ❌ Bad (index not used)
WHERE last_name LIKE '%Smith%'

-- ✅ Good (index used)
WHERE last_name LIKE 'Smith%'
```

**7. Use IN Instead of Multiple OR**
```sql
-- ❌ Bad
WHERE customer_id = 1 OR customer_id = 2 OR customer_id = 3

-- ✅ Good
WHERE customer_id IN (1, 2, 3)
```

---

#### Joining Data (Tips 8-14)

**8. Understand JOIN Performance**
- **INNER JOIN** → Best
- **LEFT/RIGHT JOIN** → Slightly slower
- **FULL OUTER JOIN** → Slowest

**9. Use Explicit JOINs (ANSI)**
```sql
-- ❌ Bad (implicit join)
FROM customers, orders
WHERE customers.id = orders.customer_id

-- ✅ Good (explicit join)
FROM customers
INNER JOIN orders ON customers.id = orders.customer_id
```

**10. Index JOIN Columns**
```sql
CREATE NONCLUSTERED INDEX idx_orders_customer
ON orders (customer_id);
```

**11. Filter Before Joining (Large Tables)**
```sql
-- ✅ Best for large tables
SELECT c.first_name, o.order_id
FROM customers c
INNER JOIN (
    SELECT order_id, customer_id
    FROM orders
    WHERE order_status = 'Delivered'
) o
ON c.customer_id = o.customer_id;
```

**12. Aggregate Before Joining (Large Tables)**
```sql
-- ✅ Pre-aggregate
SELECT c.customer_id, c.first_name, agg.order_count
FROM customers c
INNER JOIN (
    SELECT customer_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY customer_id
) agg
ON c.customer_id = agg.customer_id;
```

**13. Use UNION Instead of OR Joins**
```sql
-- ❌ Bad
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id OR c.customer_id = o.salesperson_id

-- ✅ Good
SELECT ... FROM customers c JOIN orders o ON c.customer_id = o.customer_id
UNION
SELECT ... FROM customers c JOIN orders o ON c.customer_id = o.salesperson_id
```

**14. Check for Nested Loops, Use Hints**
```sql
-- If execution plan shows nested loops on large tables
SELECT ...
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
OPTION (HASH JOIN);
```

---

#### UNION Operations (Tips 15-16)

**15. Use UNION ALL When Duplicates OK**
```sql
-- ✅ Faster
SELECT customer_id FROM orders_2023
UNION ALL
SELECT customer_id FROM orders_2024;
```

**16. Use UNION ALL + DISTINCT When Duplicates NOT OK**
```sql
-- ✅ Faster than plain UNION
SELECT DISTINCT customer_id
FROM (
    SELECT customer_id FROM orders_2023
    UNION ALL
    SELECT customer_id FROM orders_2024
) combined;
```

---

#### Aggregating Data (Tips 17-18)

**17. Use Columnstore for Large Aggregations**
```sql
CREATE CLUSTERED COLUMNSTORE INDEX idx_orders_cs
ON orders;
```

**18. Pre-Aggregate for Reporting**
```sql
-- Create summary table
SELECT 
    YEAR(order_date) AS order_year,
    SUM(sales) AS total_sales
INTO sales_summary
FROM orders
GROUP BY YEAR(order_date);

-- Query summary table instead
SELECT * FROM sales_summary;
```

---

#### Subqueries (Tips 19-20)

**19. EXISTS > IN for Large Datasets**
```sql
-- ✅ Best
SELECT *
FROM orders o
WHERE EXISTS (
    SELECT 1
    FROM customers c
    WHERE c.country = 'USA'
        AND c.customer_id = o.customer_id
);
```

**20. Avoid Redundant Logic**
```sql
-- ❌ Bad (AVG calculated twice)
SELECT ... WHERE salary > (SELECT AVG(salary) FROM employees)
UNION ALL
SELECT ... WHERE salary < (SELECT AVG(salary) FROM employees)

-- ✅ Good (AVG calculated once with window function)
SELECT 
    employee_id,
    salary,
    CASE
        WHEN salary > AVG(salary) OVER() THEN 'Above'
        ELSE 'Below'
    END AS category
FROM employees;
```

---

#### DDL Best Practices (Tips 21-25)

**21. Avoid VARCHAR(MAX) and TEXT**
```sql
-- ❌ Bad
first_name VARCHAR(MAX),
description TEXT

-- ✅ Good
first_name VARCHAR(50),
description VARCHAR(500)
```

**22. Use Appropriate Lengths**
```sql
-- ❌ Bad
country VARCHAR(255)

-- ✅ Good
country VARCHAR(50)
```

**23. Use NOT NULL**
```sql
-- ✅ Good
first_name VARCHAR(50) NOT NULL,
email VARCHAR(100) NOT NULL
```

**24. Always Have a Clustered Primary Key**
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY CLUSTERED,
    first_name VARCHAR(50) NOT NULL,
    ...
);
```

**25. Index Foreign Keys**
```sql
CREATE NONCLUSTERED INDEX idx_orders_customer
ON orders (customer_id);
```

---

#### Index Best Practices (Tips 26-30)

**26. Avoid Over-Indexing**
- Too many indexes slow INSERT/UPDATE/DELETE
- Confuses query optimizer

**27. Drop Unused Indexes**
```sql
-- Check usage
SELECT * FROM sys.dm_db_index_usage_stats
WHERE user_seeks = 0 AND user_scans = 0;
```

**28. Update Statistics Weekly**
```sql
EXEC sp_updatestats;
```

**29. Maintain Indexes Weekly**
```sql
-- Reorganize or rebuild fragmented indexes
ALTER INDEX ALL ON table_name REORGANIZE;
```

**30. Partition + Columnstore for Huge Tables**
```sql
-- Combine partitioning with columnstore
CREATE CLUSTERED COLUMNSTORE INDEX idx_orders_cs
ON orders_partitioned;
```

---

## 11. AI & SQL

### 11.1 Using AI Tools for SQL Development

**AI Tools:**
- **GitHub Copilot** → Code completion in IDE
- **ChatGPT / Claude** → Query design, optimization, learning

---

### 11.2 ChatGPT Prompt Structure

**Essential Components:**

| Priority | Component | Description |
|----------|-----------|-------------|
| **Must Have** | Task | Clear request |
| **Important** | Context | Database schema, tools |
| **Important** | Details | Specific requirements |
| **Nice to Have** | Role | "You are a senior SQL expert" |
| **Nice to Have** | Tone | "Conversational and direct" |

---

**Example Prompt:**
```
You are a senior SQL expert.                          [Role]

I am a data analyst working on SQL Server.            [Context]

Explain window functions and do the following:        [Task]
- Explain each function with syntax
- Describe when to use them
- List top 3 use cases                                [Details]

Use a conversational tone.                            [Tone]
```

---

### 11.3 Essential SQL Prompts

#### Prompt 1: Solve SQL Task

```
In my SQL Server database:
- Table 1: orders (order_id, sales, customer_id, product_id)
- Table 2: customers (customer_id, first_name, last_name, country)

Do the following:
- Rank customers by total sales
- Include: customer_id, full name, country, total sales, rank
- Add comments (avoid obvious parts)
- Write 3 different versions
- Explain which version is best (readability + performance)
```

---

#### Prompt 2: Improve Readability

```
The following SQL Server query is hard to understand.

Do the following:
- Improve readability
- Remove redundancy
- Add comments (avoid obvious parts)
- Explain each improvement

[PASTE QUERY]
```

---

#### Prompt 3: Optimize Performance

```
The following SQL Server query is slow.

Do the following:
- Propose optimizations
- Provide improved query
- Explain each improvement

[PASTE QUERY]
```

---

#### Prompt 4: Optimize Execution Plan

```
[ATTACH EXECUTION PLAN IMAGE]

Do the following:
- Describe execution plan step-by-step
- Identify bottlenecks
- Suggest performance improvements

[PASTE QUERY]
```

---

#### Prompt 5: Debug Errors

```
This SQL Server query causes error: [ERROR MESSAGE]

Do the following:
- Explain the error
- Find root cause
- Suggest fix

[PASTE QUERY]
```

---

#### Prompt 6: Explain Query Results

```
I don't understand the result of this query.

Do the following:
- Break down query processing step-by-step
- Explain how the result is formed

[PASTE QUERY]
```

---

#### Prompt 7: Format Code

```
This SQL query is hard to read.

Do the following:
- Restyle for readability
- Align column aliases
- Keep compact (no unnecessary lines)
- Follow best practices

[PASTE QUERY]
```

---

#### Prompt 8: Add Documentation

```
This query lacks documentation.

Do the following:
- Add leading comment (overall purpose)
- Add inline comments (only where needed)
- Create document: business rules
- Create document: how it works

[PASTE QUERY]
```

---

#### Prompt 9: Optimize DDL

```
This SQL Server DDL needs optimization.

Do the following:
- Check naming consistency
- Verify data types
- Check primary/foreign keys
- Review indexes (avoid redundancy)
- Check normalization

[PASTE DDL]
```

---

#### Prompt 10: Generate Test Data

```
I need test data for this DDL.

Do the following:
- Generate INSERT statements
- Make data realistic
- Keep dataset small
- Ensure valid FK relationships
- No NULL values

[PASTE DDL]
```

---

#### Prompt 11: Create Learning Roadmap

```
Create a comprehensive SQL course roadmap.

Do the following:
- Start with fundamentals → advanced
- Make it beginner-friendly
- Focus on data analytics
- Include real-world scenarios
```

---

#### Prompt 12: Understand Concept

```
Explain SQL Window Functions in detail.

Do the following:
- Define what they are
- Provide an analogy
- Explain when to use them
- Show syntax
- Provide simple examples
- List top 5 use cases
```

---

#### Prompt 13: Compare Concepts

```
Compare Window Functions vs GROUP BY.

Do the following:
- Explain key differences
- Describe when to use each (with examples)
- List pros/cons
- Create comparison table
```

---

#### Prompt 14: Practice Exercises

```
Topic: [SQL TOPIC]

Do the following:
- Make it interactive (you give tasks, I solve)
- Provide sample dataset
- Give tasks with increasing difficulty
- Act as SQL Server (show results)
- Review my queries and suggest improvements
```

---

#### Prompt 15: Interview Preparation

```
Act as a SQL interviewer.

Do the following:
- Ask common SQL interview questions
- Make it interactive
- Progress to advanced topics
- Evaluate my answers
- Give improvement feedback
```

---

## 12. SQL Projects

### Project Templates

**1. Data Warehousing Project**
- Design star/snowflake schema
- Implement ETL pipelines
- Create dimension and fact tables
- Build aggregated views

**2. Exploratory Data Analysis (EDA)**
- Analyze dataset patterns
- Identify data quality issues
- Create summary statistics
- Visualize distributions

**3. Advanced Analytics Project**
- Implement window functions
- Create customer segmentation
- Build forecasting models
- Generate executive dashboards

---

## 🎯 Key Takeaways - Performance Optimization

### Indexes
- ✅ Use clustered for primary keys (OLTP)
- ✅ Use columnstore for analytics (OLAP)
- ✅ Index foreign keys and JOIN columns
- ✅ Monitor usage and remove unused indexes
- ⚠️ Avoid over-indexing (confuses optimizer)

### Maintenance
- ✅ Update statistics weekly
- ✅ Reorganize indexes (10-30% fragmentation)
- ✅ Rebuild indexes (>30% fragmentation)
- ✅ Monitor missing indexes
- ✅ Check duplicate indexes

### Optimization
- ✅ Filter before joining (large tables)
- ✅ Aggregate before joining (large tables)
- ✅ Use EXISTS instead of IN (large datasets)
- ✅ Pre-aggregate for reporting
- ✅ Review execution plans regularly

### AI Integration
- ✅ Use AI for query optimization
- ✅ Generate test data
- ✅ Debug errors faster
- ✅ Learn new concepts
- ✅ Prepare for interviews

---

> **Congratulations!** You've completed the SQL Advanced section. Continue practicing with real-world projects to solidify these concepts.
