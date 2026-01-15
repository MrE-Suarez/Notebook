# SQL Basics (Sections 01–04)

> **Level:** Foundation  
> **Topics:** SELECT queries, DDL (Table Structure), DML (Data Manipulation)

---

## 📋 Quick Reference

| Section | Topic | Commands |
|---------|-------|----------|
| [2](#2-query-data--select) | Query Data | `SELECT`, `FROM`, `WHERE` |
| [3](#3-data-definition-language-ddl) | Table Structure (DDL) | `CREATE`, `ALTER`, `DROP` |
| [4](#4-data-manipulation-language-dml) | Data Management (DML) | `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE` |
| [5](#5-common-clauses) | Query Clauses | `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY` |

---

## 2. Query Data – SELECT

**Purpose:** Retrieve data from a database table.

### Basic Syntax
```sql
SELECT column1, column2, column3
FROM table_name;
```

### Select All Columns
```sql
SELECT *
FROM table_name;
```

> **💡 Tip:** Avoid `SELECT *` in production queries. Specify only needed columns for better performance.

---

## 3. Data Definition Language (DDL)

**Purpose:** Define and manage the **structure** of database objects (tables, columns, constraints).

| Command | Description | Use Case |
|---------|-------------|----------|
| `CREATE` | Create a new table | Define initial structure |
| `ALTER` | Modify existing table | Add/remove/modify columns |
| `DROP` | Delete table or column | Remove objects permanently |

---

### 🧱 CREATE TABLE

Define a new table with columns, data types, and constraints.

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype CONSTRAINT,
    column2 datatype CONSTRAINT,
    ...
);
```

**Example:**
```sql
CREATE TABLE persons (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    age INT,
    email VARCHAR(100)
);
```

**Key Elements:**
- **Column Name:** Identifier for the data field
- **Data Type:** `INT`, `VARCHAR`, `DATE`, etc.
- **Constraint:** `PRIMARY KEY`, `NOT NULL`, `UNIQUE`, `DEFAULT`

---

### ✏️ ALTER TABLE

Modify an existing table's structure.

#### Add a Column
```sql
ALTER TABLE persons
ADD email VARCHAR(100) NOT NULL;
```

#### Modify a Column
```sql
ALTER TABLE persons
ALTER COLUMN age SMALLINT;
```

#### Drop a Column
```sql
ALTER TABLE persons
DROP COLUMN email;
```

> **⚠️ Warning:** Dropping columns deletes all data in that column permanently.

---

### 🗑️ DROP

Delete database objects entirely.

#### Drop a Column
```sql
ALTER TABLE persons
DROP COLUMN phone;
```

#### Drop a Table
```sql
DROP TABLE persons;
```

> **⚠️ Danger Zone:** `DROP TABLE` deletes the entire table and all its data irreversibly. Always backup first.

---

## 4. Data Manipulation Language (DML)

**Purpose:** Manage the **data** inside tables (add, modify, delete records).

| Command | Action | Risk Level |
|---------|--------|------------|
| `INSERT` | Add new rows | Low |
| `UPDATE` | Modify existing rows | ⚠️ High (without WHERE) |
| `DELETE` | Remove specific rows | ⚠️ High (without WHERE) |
| `TRUNCATE` | Remove all rows | ⚠️ Critical |

---

### ➕ INSERT

Add new records to a table.

#### Insert Single Row (Manual Values)
```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

**Example:**
```sql
INSERT INTO persons (id, name, age, email)
VALUES (1, 'John Doe', 30, 'john@example.com');
```

---

#### Insert Multiple Rows
```sql
INSERT INTO persons (id, name, age)
VALUES 
    (2, 'Jane Smith', 25),
    (3, 'Bob Johnson', 35),
    (4, 'Alice Brown', 28);
```

---

#### Insert from SELECT Query
Populate a table using data from another table.

```sql
INSERT INTO persons (id, first_name, last_name, country)
SELECT 
    customer_id,
    first_name,
    last_name,
    'Unknown'
FROM customers
WHERE registration_date > '2024-01-01';
```

> **💡 Use Case:** Useful for data migration, creating test datasets, or populating staging tables.

---

### 🔄 UPDATE

Modify existing records in a table.

**Syntax:**
```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;
```

**Example:**
```sql
UPDATE customers
SET score = 0, status = 'inactive'
WHERE last_purchase_date < '2023-01-01';
```

---

#### ⚠️ Critical Rule: Always Use WHERE

```sql
-- ❌ DANGEROUS: Updates ALL rows
UPDATE customers
SET country = 'Unknown';

-- ✅ SAFE: Updates only matching rows
UPDATE customers
SET country = 'Unknown'
WHERE country IS NULL;
```

---

#### Best Practice: Verify Before/After
```sql
-- Step 1: Check current state
SELECT *
FROM customers
WHERE id = 10;

-- Step 2: Update
UPDATE customers
SET score = 100, country = 'USA'
WHERE id = 10;

-- Step 3: Verify change
SELECT *
FROM customers
WHERE id = 10;
```

---

### ❌ DELETE

Remove specific rows from a table.

**Syntax:**
```sql
DELETE FROM table_name
WHERE condition;
```

**Example:**
```sql
DELETE FROM orders
WHERE order_status = 'cancelled'
AND order_date < '2023-01-01';
```

---

#### ⚠️ Critical Rule: Always Use WHERE

```sql
-- ❌ CATASTROPHIC: Deletes ALL rows
DELETE FROM customers;

-- ✅ SAFE: Deletes only matching rows
DELETE FROM customers
WHERE account_status = 'deleted'
AND last_login < '2022-01-01';
```

---

### 🧹 TRUNCATE

Remove **all rows** from a table but keep the structure.

**Syntax:**
```sql
TRUNCATE TABLE table_name;
```

**Example:**
```sql
TRUNCATE TABLE temp_staging_data;
```

---

#### DELETE vs TRUNCATE

| Feature | DELETE | TRUNCATE |
|---------|--------|----------|
| **Speed** | Slower (row-by-row) | ⚡ Much faster |
| **WHERE clause** | ✅ Supported | ❌ Not allowed |
| **Rollback** | ✅ Can be rolled back | ⚠️ Cannot be rolled back (DDL) |
| **Reset Identity** | ❌ No | ✅ Yes |
| **Triggers** | ✅ Fires DELETE triggers | ❌ No triggers |

**When to Use:**
- `DELETE` → Remove specific rows with conditions
- `TRUNCATE` → Quickly empty entire table (staging, temp tables)

---

## 5. Common Clauses

SQL clauses control how queries retrieve and process data.

### Clause Execution Order

| Logical Order | Clause | Purpose |
|---------------|--------|---------|
| 1️⃣ | `FROM` | Define data source |
| 2️⃣ | `WHERE` | Filter rows **before** aggregation |
| 3️⃣ | `GROUP BY` | Group rows for aggregation |
| 4️⃣ | `HAVING` | Filter groups **after** aggregation |
| 5️⃣ | `SELECT` | Choose columns to return |
| 6️⃣ | `DISTINCT` | Remove duplicate rows |
| 7️⃣ | `ORDER BY` | Sort final results |
| 8️⃣ | `TOP` / `LIMIT` | Limit number of rows returned |

---

### Clause Reference Table

| Clause | Purpose | Example |
|--------|---------|---------|
| **SELECT** | Retrieve columns | `SELECT first_name, last_name` |
| **DISTINCT** | Remove duplicates | `SELECT DISTINCT country` |
| **TOP** / **LIMIT** | Limit results | `SELECT TOP 10 *` (SQL Server)<br>`SELECT * LIMIT 10` (MySQL/PostgreSQL) |
| **FROM** | Specify table | `FROM customers` |
| **WHERE** | Filter rows | `WHERE age > 18` |
| **GROUP BY** | Aggregate rows | `GROUP BY country` |
| **HAVING** | Filter aggregated results | `HAVING COUNT(*) > 5` |
| **ORDER BY** | Sort results | `ORDER BY last_name ASC` |

---

### WHERE vs HAVING

```sql
-- ✅ WHERE: Filter rows before grouping
SELECT country, COUNT(*) as customer_count
FROM customers
WHERE age > 18  -- Filter individual rows
GROUP BY country;

-- ✅ HAVING: Filter groups after aggregation
SELECT country, COUNT(*) as customer_count
FROM customers
GROUP BY country
HAVING COUNT(*) > 100;  -- Filter aggregated results
```

**Key Difference:**
- `WHERE` → Filters **individual rows** (cannot use aggregate functions)
- `HAVING` → Filters **grouped results** (can use aggregate functions like `COUNT`, `SUM`, `AVG`)

---

### TOP / LIMIT

Restrict the number of rows returned.

#### SQL Server
```sql
SELECT TOP 5 *
FROM orders
ORDER BY order_date DESC;
```

#### MySQL / PostgreSQL
```sql
SELECT *
FROM orders
ORDER BY order_date DESC
LIMIT 5;
```

#### With Percentage (SQL Server)
```sql
SELECT TOP 10 PERCENT *
FROM customers
ORDER BY total_purchases DESC;
```

---

### DISTINCT

Remove duplicate rows from results.

```sql
-- Without DISTINCT: May return duplicate countries
SELECT country
FROM customers;

-- With DISTINCT: Each country appears once
SELECT DISTINCT country
FROM customers;
```

> **⚠️ Performance Note:** `DISTINCT` can slow queries on large datasets. Use only when necessary.

---

### Complete Query Example

Combining multiple clauses:

```sql
SELECT DISTINCT 
    country,
    COUNT(*) as customer_count,
    AVG(age) as avg_age
FROM customers
WHERE registration_date >= '2024-01-01'
    AND status = 'active'
GROUP BY country
HAVING COUNT(*) > 50
ORDER BY customer_count DESC;
```

**Execution Steps:**
1. `FROM customers` → Get data source
2. `WHERE` → Filter active customers from 2024
3. `GROUP BY` → Group by country
4. `HAVING` → Keep only countries with 50+ customers
5. `SELECT` → Calculate count and average
6. `DISTINCT` → Remove duplicates (if any)
7. `ORDER BY` → Sort by customer count descending

---

## 🎯 Key Takeaways

### DDL (Structure)
- ✅ Use `CREATE` to define new tables
- ✅ Use `ALTER` to modify existing structures
- ⚠️ Use `DROP` carefully—it's permanent

### DML (Data)
- ✅ Use `INSERT` to add records
- ⚠️ Always use `WHERE` with `UPDATE` and `DELETE`
- ⚠️ Use `TRUNCATE` only when you want to delete ALL rows

### Query Construction
- ✅ Use `WHERE` to filter rows
- ✅ Use `HAVING` to filter aggregated results
- ✅ Use `ORDER BY` to sort results
- ⚠️ Use `DISTINCT` sparingly (performance impact)

---

> **Next:** Continue to [INTERMEDIATE_05-08.md](./INTERMEDIATE_05-08.md) for filtering, joins, and window functions.
