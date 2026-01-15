# SQL Basics (Sections 01–04)

This file covers the **fundamental SQL concepts**, including query basics, DDL (Data Definition Language), and DML (Data Manipulation Language).

---

## 2. Query Data — SELECT
Used to **retrieve data** from a dataset.

```sql
SELECT column1, column2
FROM table_name;
```

---

## 3. Data Definition Language (DDL)
Used to **manage table structures**.

| Command | Description |
|----------|--------------|
| **CREATE** | Start a new table |
| **ALTER** | Edit an existing table |
| **DROP** | Delete an existing table |

### 🧱 CREATE
Create a new table in the database.

```sql
CREATE TABLE persons (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    age INT
);
```

### ✏️ ALTER
Modify an existing table — add, remove, or modify columns.  
_Remember: Column Name, Data Type, Constraint_

```sql
ALTER TABLE persons 
ADD email VARCHAR(50) NOT NULL;
```

### 🗑️ DROP
Delete a column or table entirely.

```sql
ALTER TABLE persons 
DROP COLUMN phone;

DROP TABLE persons;
```

---

## 4. Data Manipulation Language (DML)
Used to **manage data within a table**.

| Command | Action |
|----------|---------|
| **INSERT** | Add new data |
| **UPDATE** | Edit existing data |
| **DELETE** | Remove existing data |
| **TRUNCATE** | Empty the table completely |

---

### ➕ INSERT
Add rows to a table.

#### Manually
```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

#### Using SELECT
```sql
INSERT INTO persons
SELECT 
    id,
    first_name,
    NULL,
    'Unknown'
FROM customers;
```

---

### 🔄 UPDATE
Modify existing records.  
⚠️ **Always include a WHERE clause** to avoid updating every row.

```sql
UPDATE customers
SET score = 0, country = 'UK'
WHERE id = 10;
```

Good practice — verify before/after:

```sql
SELECT *
FROM customers
WHERE id = 10;
```

---

### ❌ DELETE
Remove rows that match a condition.  
⚠️ **Always use WHERE** to avoid deleting everything.

```sql
DELETE FROM table_name
WHERE condition;
```

---

### 🧹 TRUNCATE
Reset the table (removes all rows, but structure remains).  
Much faster than DELETE for large tables.

```sql
TRUNCATE TABLE table_name;
```

---

## ⚙️ Common Clauses

| Clause | Purpose |
|---------|----------|
| **SELECT** | Retrieve columns or aggregated results |
| **DISTINCT** | Remove duplicates (can slow performance) |
| **TOP # /** | Limit number of returned rows |
| **FROM** | Define the data source |
| **WHERE** | Filter rows before aggregation |
| **GROUP BY** | Combine rows and apply aggregate functions |
| **HAVING** | Filter results after aggregation (on aggregated values) |
| **ORDER BY** | Sort results |

🧠 **Tip:** Use `WHERE` for normal filters and `HAVING` for aggregated results.

---

> _End of Basics (Sections 01–04)_  
> Continue to [INTERMEDIATE_05-08.md](./INTERMEDIATE_05-08.md)
