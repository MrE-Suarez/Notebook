# SQL Intermediate (Sections 05–08)

> **Level:** Intermediate  
> **Topics:** Filtering, Joins, Set Operators, Functions, Window Functions

---

## 📋 Quick Reference

| Section | Topic | Key Concepts |
|---------|-------|--------------|
| [5](#5-filtering-data) | Filtering Data | Operators, Pattern Matching |
| [6](#6-combining-data) | Combining Data | JOINs, Set Operators |
| [7](#7-row-level-functions) | Row-Level Functions | String, Number, Date/Time, NULL handling, CASE |
| [8](#8-aggregation--analytical-functions) | Aggregation & Analytics | GROUP BY, Window Functions |

---

## 5. Filtering Data

### 5.1 Comparison Operators

Used to compare values in `WHERE` clauses.

| Operator | Description | Example | Notes |
|----------|-------------|---------|-------|
| `=` | Equal to | `WHERE age = 30` | Exact match |
| `<>` or `!=` | Not equal to | `WHERE age <> 30` | `<>` is SQL standard |
| `<` | Less than | `WHERE score < 100` | Numeric/date comparison |
| `<=` | Less than or equal | `WHERE score <= 100` | Inclusive |
| `>` | Greater than | `WHERE score > 100` | Numeric/date comparison |
| `>=` | Greater than or equal | `WHERE score >= 100` | Inclusive |

**Example:**
```sql
SELECT *
FROM customers
WHERE age >= 18 AND score > 500;
```

---

### 5.2 Logical Operators

Combine multiple conditions in a query.

| Operator | Description | Example |
|----------|-------------|---------|
| `AND` | Both conditions must be true | `WHERE age > 18 AND country = 'US'` |
| `OR` | Either condition can be true | `WHERE age < 18 OR status = 'VIP'` |
| `NOT` | Negates a condition | `WHERE NOT country = 'US'` |

**Evaluation Order:**
1. `NOT` → evaluated first
2. `AND` → evaluated second
3. `OR` → evaluated last

Use parentheses `()` to control evaluation order:
```sql
WHERE (age < 18 OR age > 65) AND status = 'active'
```

---

### 5.3 Range Operators

Check if a value falls within a range.

| Operator | Description | Example |
|----------|-------------|---------|
| `BETWEEN ... AND ...` | Value within range (inclusive) | `WHERE age BETWEEN 18 AND 30` |
| `NOT BETWEEN ... AND ...` | Value outside range | `WHERE score NOT BETWEEN 50 AND 100` |

**Important:** `BETWEEN` is **inclusive** on both ends.

```sql
-- These are equivalent:
WHERE age BETWEEN 18 AND 30
WHERE age >= 18 AND age <= 30
```

**Example:**
```sql
SELECT *
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

---

### 5.4 Membership Operators

Check if a value exists in a list.

| Operator | Description | Example |
|----------|-------------|---------|
| `IN (...)` | Value matches any in list | `WHERE country IN ('US', 'UK', 'CA')` |
| `NOT IN (...)` | Value not in list | `WHERE status NOT IN ('inactive', 'banned')` |

**Performance Tip:** `IN` is faster than multiple `OR` conditions.

```sql
-- ❌ Slower:
WHERE country = 'US' OR country = 'UK' OR country = 'CA'

-- ✅ Faster:
WHERE country IN ('US', 'UK', 'CA')
```

---

### 5.5 Pattern Matching

Search for patterns in text using wildcards.

| Operator | Description | Example |
|----------|-------------|---------|
| `LIKE` | Matches a pattern | `WHERE name LIKE 'J%'` |
| `NOT LIKE` | Does not match pattern | `WHERE email NOT LIKE '%@gmail.com'` |
| `%` | Wildcard: any number of characters | `'Jo%'` → John, Joseph, Joe |
| `_` | Wildcard: exactly one character | `'J_n'` → Jon, Jan, Jen |

**Examples:**
```sql
-- Names starting with 'A'
WHERE name LIKE 'A%'

-- Names ending with 'son'
WHERE name LIKE '%son'

-- Names containing 'and'
WHERE name LIKE '%and%'

-- Exactly 3 characters
WHERE code LIKE '___'

-- Second letter is 'a'
WHERE name LIKE '_a%'
```

> **💡 Tip:** Use `ILIKE` in PostgreSQL for case-insensitive matching.

---

### 5.6 Combining Filters

Complex filtering with multiple operators:

```sql
SELECT *
FROM customers
WHERE age >= 18
    AND status = 'active'
    AND country IN ('US', 'UK', 'CA')
    AND registration_date BETWEEN '2024-01-01' AND '2024-12-31'
    AND email LIKE '%@company.com'
ORDER BY last_name;
```

---

## 6. Combining Data

### 6.1 JOIN Operations

Combine rows from two or more tables based on related columns.

---

#### INNER JOIN

Returns only rows with matches in **both** tables.

**Syntax:**
```sql
SELECT columns
FROM table1
INNER JOIN table2
    ON table1.key = table2.key;
```

**Example:**
```sql
SELECT 
    c.customer_id,
    c.first_name,
    o.order_id,
    o.sales
FROM customers AS c
INNER JOIN orders AS o
    ON c.customer_id = o.customer_id;
```

**Result:** Only customers who have placed orders.

> **💡 Tip:** Always use table aliases (`AS c`, `AS o`) for readability in joins.

---

#### LEFT JOIN (LEFT OUTER JOIN)

Returns **all rows from the left table**, plus matches from the right table.

**Syntax:**
```sql
SELECT columns
FROM table1
LEFT JOIN table2
    ON table1.key = table2.key;
```

**Example:**
```sql
SELECT 
    c.customer_id,
    c.first_name,
    o.order_id,
    o.sales
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customer_id = o.customer_id;
```

**Result:** All customers, including those without orders (NULLs for order columns).

---

#### RIGHT JOIN (RIGHT OUTER JOIN)

Returns **all rows from the right table**, plus matches from the left table.

**Syntax:**
```sql
SELECT columns
FROM table1
RIGHT JOIN table2
    ON table1.key = table2.key;
```

> **💡 Note:** `RIGHT JOIN` is less common. You can achieve the same result by swapping tables and using `LEFT JOIN`.

---

#### FULL JOIN (FULL OUTER JOIN)

Returns **all rows from both tables**, with NULLs where no match exists.

**Syntax:**
```sql
SELECT columns
FROM table1
FULL JOIN table2
    ON table1.key = table2.key;
```

**Example:**
```sql
SELECT *
FROM customers
FULL JOIN orders
    ON customers.customer_id = orders.customer_id;
```

**Result:** All customers and all orders, with NULLs where no match exists.

---

#### CROSS JOIN

Returns the **Cartesian product** of both tables (all possible combinations).

**Syntax:**
```sql
SELECT *
FROM table1
CROSS JOIN table2;
```

**Example:**
```sql
SELECT 
    p.product_name,
    s.store_name
FROM products AS p
CROSS JOIN stores AS s;
```

**Result:** Every product paired with every store.

> **⚠️ Warning:** Can produce massive result sets. Use with caution.

---

#### Anti-Joins

Find rows in one table that don't have matches in another.

**LEFT ANTI JOIN** (rows in left table with no match in right):
```sql
SELECT c.*
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

**FULL ANTI JOIN** (rows in either table with no match):
```sql
SELECT *
FROM customers AS c
FULL JOIN orders AS o
    ON c.customer_id = o.customer_id
WHERE c.customer_id IS NULL 
    OR o.customer_id IS NULL;
```

---

### 6.2 JOIN Decision Tree

**How to choose the correct JOIN:**

```
Need only matching rows?
└─→ INNER JOIN

Need all rows from one table?
├─→ LEFT JOIN (all from left + matches from right)
└─→ RIGHT JOIN (all from right + matches from left)

Need all rows from both tables?
└─→ FULL JOIN

Need only non-matching rows?
├─→ LEFT ANTI JOIN (left without matches)
└─→ FULL ANTI JOIN (either without matches)

Need all possible combinations?
└─→ CROSS JOIN
```

---

### 6.3 Multiple JOINs

Combine data from 3+ tables:

```sql
SELECT
    o.order_id,
    (c.first_name + ' ' + c.last_name) AS customer_name,
    p.product_name,
    o.sales AS order_total,
    p.price AS unit_price,
    (e.first_name + ' ' + e.last_name) AS salesperson
FROM orders AS o
LEFT JOIN customers AS c
    ON c.customer_id = o.customer_id
LEFT JOIN employees AS e
    ON e.employee_id = o.salesperson_id
LEFT JOIN products AS p
    ON p.product_id = o.product_id;
```

---

### 6.4 Set Operators

Combine results from multiple queries vertically (adding rows).

| Operator | Description |
|----------|-------------|
| `UNION` | Combine results, remove duplicates |
| `UNION ALL` | Combine results, keep duplicates (faster) |
| `INTERSECT` | Return only rows common to both queries |
| `EXCEPT` | Return rows in first query but not in second |

---

#### UNION

Combine results from multiple queries, removing duplicates.

```sql
SELECT customer_id, first_name
FROM customers_usa

UNION

SELECT customer_id, first_name
FROM customers_europe;
```

---

#### UNION ALL

Combine results, keeping all duplicates (faster than `UNION`).

```sql
SELECT customer_id
FROM orders_2023

UNION ALL

SELECT customer_id
FROM orders_2024;
```

> **💡 Performance:** Use `UNION ALL` when you know there are no duplicates or duplicates are acceptable.

---

#### INTERSECT

Return only rows that appear in **both** queries.

```sql
SELECT customer_id
FROM orders_2023

INTERSECT

SELECT customer_id
FROM orders_2024;
```

**Result:** Customers who ordered in both years.

---

#### EXCEPT (MINUS in Oracle)

Return rows in first query that are **not** in second query.

```sql
SELECT customer_id
FROM customers

EXCEPT

SELECT customer_id
FROM orders;
```

**Result:** Customers who have never placed an order.

---

### Set Operator Rules

1. ✅ `ORDER BY` can only be used **once** at the end
2. ✅ Queries must have the **same number of columns**
3. ✅ Columns must have **compatible data types**
4. ✅ Columns must be in the **same order**
5. ✅ First query controls column **aliases**
6. ✅ Map columns **correctly** between queries

**Example:**
```sql
SELECT customer_id, first_name, 'USA' AS region
FROM customers_usa

UNION

SELECT customer_id, first_name, 'Europe' AS region
FROM customers_europe

ORDER BY first_name;  -- Only one ORDER BY at the end
```

---

## 7. Row-Level Functions

Functions that operate on individual rows.

### 7.1 String Functions

The 7 most commonly used string functions in SQL.

---

#### 1. CONCAT() - Combine Strings

Joins multiple strings into one.

**Syntax:**
```sql
CONCAT(string1, string2, string3, ...)
```

**Examples:**
```sql
-- Full name
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM customers;

-- Address formatting
SELECT CONCAT(street, ', ', city, ', ', state, ' ', zip_code) AS full_address
FROM addresses;

-- With separators
SELECT CONCAT('Order #', order_id, ' - ', product_name) AS order_description
FROM orders;
```

> **💡 Tip:** Use `CONCAT_WS()` (Concat With Separator) to add separators automatically.

---

#### 2. UPPER() / LOWER() - Change Case

Convert text to uppercase or lowercase.

**Syntax:**
```sql
UPPER(string)   -- Convert to UPPERCASE
LOWER(string)   -- Convert to lowercase
```

**Examples:**
```sql
-- Standardize country names
SELECT 
    customer_id,
    UPPER(country) AS country_standardized
FROM customers;

-- Clean email addresses (lowercase)
SELECT 
    LOWER(email) AS email_clean
FROM users;

-- Case-insensitive comparison
SELECT *
FROM products
WHERE LOWER(product_name) = 'laptop';
```

**Use Cases:**
- Data standardization
- Case-insensitive searches
- Report formatting

---

#### 3. TRIM() - Remove Whitespace

Removes leading and trailing spaces (or specified characters).

**Syntax:**
```sql
TRIM(string)                    -- Remove spaces from both sides
LTRIM(string)                   -- Remove spaces from left
RTRIM(string)                   -- Remove spaces from right
```

**Examples:**
```sql
-- Clean user input
SELECT 
    customer_id,
    TRIM(first_name) AS first_name_clean,
    TRIM(last_name) AS last_name_clean
FROM customers;

-- Remove spaces before comparison
SELECT *
FROM products
WHERE TRIM(category) = 'Electronics';

-- Data quality check
SELECT 
    product_id,
    product_name,
    LEN(product_name) AS original_length,
    LEN(TRIM(product_name)) AS trimmed_length
FROM products
WHERE LEN(product_name) != LEN(TRIM(product_name));  -- Find records with extra spaces
```

---

#### 4. SUBSTRING() - Extract Part of String

Extracts a portion of a string.

**Syntax:**
```sql
SUBSTRING(string, start_position, length)
```

**Examples:**
```sql
-- Extract area code from phone number
SELECT 
    phone_number,
    SUBSTRING(phone_number, 1, 3) AS area_code
FROM contacts;

-- Get first 3 characters of product code
SELECT 
    product_code,
    SUBSTRING(product_code, 1, 3) AS category_code
FROM products;

-- Extract month from date string ('2024-03-15')
SELECT 
    order_date,
    SUBSTRING(CAST(order_date AS VARCHAR), 6, 2) AS month
FROM orders;
```

---

#### 5. LEFT() / RIGHT() - Extract from Edges

Extract characters from left or right side of string.

**Syntax:**
```sql
LEFT(string, number_of_characters)
RIGHT(string, number_of_characters)
```

**Examples:**
```sql
-- Extract first initial
SELECT 
    first_name,
    LEFT(first_name, 1) AS initial
FROM customers;

-- Extract file extension
SELECT 
    filename,
    RIGHT(filename, 4) AS file_extension
FROM documents;

-- Mask credit card (show last 4 digits)
SELECT 
    customer_id,
    CONCAT('****-****-****-', RIGHT(credit_card, 4)) AS masked_card
FROM payments;

-- Extract year from date string
SELECT 
    LEFT(order_date_string, 4) AS year
FROM orders;
```

---

#### 6. REPLACE() - Replace Substring

Replaces all occurrences of a substring with another string.

**Syntax:**
```sql
REPLACE(string, old_substring, new_substring)
```

**Examples:**
```sql
-- Clean phone numbers (remove dashes)
SELECT 
    phone_number,
    REPLACE(phone_number, '-', '') AS phone_clean
FROM contacts;

-- Standardize product names
SELECT 
    product_name,
    REPLACE(REPLACE(product_name, '&', 'and'), '/', ' or ') AS product_name_clean
FROM products;

-- Remove special characters
SELECT 
    email,
    REPLACE(REPLACE(email, '.', ''), '-', '') AS email_no_special
FROM users;

-- Mask sensitive data
SELECT 
    REPLACE(ssn, SUBSTRING(ssn, 1, 5), '*****') AS ssn_masked
FROM employees;
```

---

#### 7. LEN() / CHARINDEX() - Measure & Find

**LEN()** - Get string length  
**CHARINDEX()** - Find position of substring

**Syntax:**
```sql
LEN(string)                              -- Returns length
CHARINDEX(substring, string)             -- Returns position (0 if not found)
```

**Examples:**
```sql
-- Validate input length
SELECT 
    username,
    LEN(username) AS username_length
FROM users
WHERE LEN(username) < 3;  -- Find usernames too short

-- Extract email domain
SELECT 
    email,
    SUBSTRING(email, CHARINDEX('@', email) + 1, LEN(email)) AS email_domain
FROM users;

-- Find position of space (to split first/last name)
SELECT 
    full_name,
    CHARINDEX(' ', full_name) AS space_position,
    LEFT(full_name, CHARINDEX(' ', full_name) - 1) AS first_name,
    SUBSTRING(full_name, CHARINDEX(' ', full_name) + 1, LEN(full_name)) AS last_name
FROM contacts;

-- Check if string contains substring
SELECT 
    product_name,
    CASE 
        WHEN CHARINDEX('Pro', product_name) > 0 THEN 'Professional Edition'
        ELSE 'Standard Edition'
    END AS product_tier
FROM products;
```

---

#### String Functions - Quick Reference

| Function | Purpose | Common Use Case |
|----------|---------|-----------------|
| `CONCAT()` | Join strings | Full names, addresses |
| `UPPER()` / `LOWER()` | Change case | Standardization, searches |
| `TRIM()` | Remove spaces | Data cleaning |
| `SUBSTRING()` | Extract portion | Codes, IDs, patterns |
| `LEFT()` / `RIGHT()` | Extract edges | Initials, extensions |
| `REPLACE()` | Replace text | Cleaning, standardization |
| `LEN()` / `CHARINDEX()` | Measure & find | Validation, parsing |

---

### 7.2 Number Functions

| Function | Description | Example |
|----------|-------------|---------|
| `ROUND()` | Round to decimal places | `ROUND(123.456, 2)` → `123.46` |
| `ABS()` | Absolute value | `ABS(-10)` → `10` |
| `CEILING()` | Round up | `CEILING(4.2)` → `5` |
| `FLOOR()` | Round down | `FLOOR(4.8)` → `4` |

**Example:**
```sql
SELECT 
    product_name,
    price,
    ROUND(price * 1.08, 2) AS price_with_tax
FROM products;
```

---

### 7.3 Date and Time Functions

#### Date and Time Data Types

Understanding data types is crucial for proper table design.

| Data Type | Storage | Range | Precision | Use Case |
|-----------|---------|-------|-----------|----------|
| **DATE** | 3 bytes | 0001-01-01 to 9999-12-31 | Day | Birthdates, deadlines |
| **TIME** | 3-5 bytes | 00:00:00 to 23:59:59 | 100 nanoseconds | Opening hours, durations |
| **DATETIME** | 8 bytes | 1753-01-01 to 9999-12-31 | 3.33 milliseconds | Legacy systems |
| **DATETIME2** | 6-8 bytes | 0001-01-01 to 9999-12-31 | 100 nanoseconds | **Recommended** for new systems |
| **SMALLDATETIME** | 4 bytes | 1900-01-01 to 2079-06-06 | 1 minute | Compact storage |
| **DATETIMEOFFSET** | 10 bytes | Same as DATETIME2 | 100 nanoseconds | Time zones |

**Choosing Data Types:**

```
Need only date (no time)?
└─→ DATE

Need only time (no date)?
└─→ TIME

Need date + time?
├─→ Working with time zones? → DATETIMEOFFSET
├─→ Need high precision? → DATETIME2 (recommended)
├─→ Legacy compatibility? → DATETIME
└─→ Compact storage, low precision OK? → SMALLDATETIME
```

**Examples:**
```sql
-- Table design with proper types
CREATE TABLE events (
    event_id INT PRIMARY KEY,
    event_date DATE,                    -- Only need date
    start_time TIME,                    -- Only need time
    created_at DATETIME2,               -- Date + time with precision
    updated_at DATETIMEOFFSET           -- Include timezone
);
```

---

#### Date Function Decision Tree

**"Which date function should I use?"**

```
What do I need to extract?
├─→ Day, Month, or Year as NUMBER?
│   ├─→ Day? → DAY(date)
│   ├─→ Month? → MONTH(date)
│   └─→ Year? → YEAR(date)
│
├─→ Month or weekday as TEXT?
│   └─→ DATENAME(month, date)
│       Examples:
│       • DATENAME(month, '2024-03-15') → 'March'
│       • DATENAME(weekday, '2024-03-15') → 'Friday'
│
├─→ Other parts (quarter, week, day of year)?
│   └─→ DATEPART(part, date)
│       Examples:
│       • DATEPART(quarter, date) → 1, 2, 3, or 4
│       • DATEPART(week, date) → Week number
│       • DATEPART(dayofyear, date) → 1-365
│
├─→ Truncate to start of period?
│   └─→ DATETRUNC(part, date)
│       Examples:
│       • DATETRUNC(month, '2024-03-15') → '2024-03-01'
│       • DATETRUNC(year, '2024-03-15') → '2024-01-01'
│
└─→ Last day of month?
    └─→ EOMONTH(date)
        • EOMONTH('2024-02-15') → '2024-02-29'
```

**Visual Guide:**

| Need | Function | Returns | Example |
|------|----------|---------|---------|
| Day number | `DAY()` | Integer | `15` |
| Month number | `MONTH()` | Integer | `3` |
| Year | `YEAR()` | Integer | `2024` |
| Month name | `DATENAME(month, ...)` | String | `'March'` |
| Quarter | `DATEPART(quarter, ...)` | Integer | `1` |
| First of month | `DATETRUNC(month, ...)` | Date | `'2024-03-01'` |
| End of month | `EOMONTH()` | Date | `'2024-03-31'` |

---

#### Date Part Extraction

| Function | Returns | Example |
|----------|---------|---------|
| `DAY()` | Day (numeric) | `DAY('2024-03-15')` → `15` |
| `MONTH()` | Month (numeric) | `MONTH('2024-03-15')` → `3` |
| `YEAR()` | Year | `YEAR('2024-03-15')` → `2024` |
| `DATENAME()` | Part name (string) | `DATENAME(month, '2024-03-15')` → `'March'` |
| `DATEPART()` | Part value (int) | `DATEPART(quarter, '2024-03-15')` → `1` |
| `DATETRUNC()` | Truncate to precision | `DATETRUNC(month, '2024-03-15')` → `'2024-03-01'` |
| `EOMONTH()` | End of month | `EOMONTH('2024-03-15')` → `'2024-03-31'` |

---

#### Format and Casting

| Function | Purpose | Example |
|----------|---------|---------|
| `FORMAT()` | Format as string | `FORMAT(order_date, 'yyyy-MM-dd')` |
| `CONVERT()` | Type conversion with format | `CONVERT(VARCHAR, order_date, 101)` |
| `CAST()` | Type conversion (ANSI) | `CAST(sales AS INT)` |

**When to use:**
- `FORMAT()` → Convert to string with specific format
- `CONVERT()` → Type conversion (SQL Server specific)
- `CAST()` → Type conversion (ANSI standard, preferred for portability)

---

#### Date Calculations

| Function | Description | Example |
|----------|-------------|---------|
| `DATEADD()` | Add interval | `DATEADD(day, 7, order_date)` → Add 7 days |
| `DATEDIFF()` | Calculate difference | `DATEDIFF(day, start_date, end_date)` → Days between |

**Examples:**
```sql
-- Orders from last 30 days
SELECT *
FROM orders
WHERE order_date >= DATEADD(day, -30, GETDATE());

-- Customer age
SELECT 
    customer_id,
    first_name,
    DATEDIFF(year, birth_date, GETDATE()) AS age
FROM customers;

-- Month name and year
SELECT 
    order_id,
    DATENAME(month, order_date) AS month_name,
    YEAR(order_date) AS year
FROM orders;
```

---

#### Date Validation

| Function | Returns | Example |
|----------|---------|---------|
| `ISDATE()` | 1 if valid date, 0 if not | `ISDATE('2024-03-15')` → `1` |

**Use Case:** Data quality checks

```sql
-- Find invalid dates
SELECT *
FROM staging_table
WHERE ISDATE(date_column) = 0;
```

---

### 7.4 NULL Functions

Handle NULL values in queries.

| Function | Purpose | Example |
|----------|---------|---------|
| `ISNULL()` | Replace NULL (2 values) | `ISNULL(phone, 'N/A')` |
| `COALESCE()` | Return first non-NULL (multiple values) | `COALESCE(phone, mobile, 'N/A')` |
| `NULLIF()` | Return NULL if values match | `NULLIF(quantity, 0)` |
| `IS NULL` | Check if NULL | `WHERE phone IS NULL` |
| `IS NOT NULL` | Check if not NULL | `WHERE email IS NOT NULL` |

---

#### ISNULL vs COALESCE

**ISNULL (SQL Server specific):**
```sql
SELECT 
    customer_id,
    ISNULL(shipping_address, billing_address) AS address
FROM customers;
```

**COALESCE (ANSI standard, preferred):**
```sql
SELECT 
    customer_id,
    COALESCE(shipping_address, billing_address, 'Unknown') AS address
FROM customers;
```

| Feature | ISNULL | COALESCE |
|---------|--------|----------|
| **Parameters** | 2 values only | Unlimited values |
| **Performance** | Faster | Slightly slower |
| **Portability** | SQL Server only | ANSI standard |

> **💡 Recommendation:** Use `COALESCE` for better portability.

---

#### Common NULL Handling Scenarios

**1. Aggregations**
```sql
-- NULL values ignored in AVG
SELECT 
    AVG(COALESCE(score, 0)) AS avg_score
FROM customers;
```

**2. String Concatenation**
```sql
-- NULL breaks concatenation
SELECT 
    first_name + ' ' + COALESCE(last_name, '') AS full_name
FROM customers;
```

**3. Joins with Nullable Keys**
```sql
SELECT *
FROM table1 AS a
LEFT JOIN table2 AS b
    ON a.year = b.year
    AND ISNULL(a.type, '') = ISNULL(b.type, '');
```

**4. Division by Zero**
```sql
SELECT 
    sales / NULLIF(quantity, 0) AS unit_price
FROM orders;
```

---

#### NULL Handling Policies

**Policy 1:** Only NULLs and empty strings
```sql
TRIM(column_name)
```

**Policy 2:** Only NULLs (convert empty to NULL)
```sql
NULLIF(TRIM(column_name), '')
```

**Policy 3:** Use default values
```sql
COALESCE(NULLIF(TRIM(column_name), ''), 'Unknown')
```

> **💡 Recommendation:**  
> - Policy 2 for data cleaning (storage efficiency)
> - Policy 3 for reporting (clarity for end users)

---

### 7.5 CASE Statement

Conditional logic within queries (similar to IF-THEN-ELSE).

**Syntax:**
```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE default_result
END
```

---

#### Simple CASE Example

```sql
SELECT 
    order_id,
    sales,
    CASE
        WHEN sales > 50 THEN 'High'
        WHEN sales > 20 THEN 'Medium'
        ELSE 'Low'
    END AS sales_category
FROM orders;
```

---

#### CASE with Aggregation

```sql
SELECT
    CASE
        WHEN sales > 50 THEN 'High'
        WHEN sales > 20 THEN 'Medium'
        ELSE 'Low'
    END AS category,
    COUNT(*) AS order_count,
    SUM(sales) AS total_sales
FROM orders
GROUP BY 
    CASE
        WHEN sales > 50 THEN 'High'
        WHEN sales > 20 THEN 'Medium'
        ELSE 'Low'
    END;
```

---

#### Compact CASE Syntax

When checking equality on a single column:

**Full Form:**
```sql
CASE
    WHEN country = 'Germany' THEN 'DE'
    WHEN country = 'India' THEN 'IN'
    WHEN country = 'Mexico' THEN 'MX'
    ELSE 'Other'
END
```

**Compact Form:**
```sql
CASE country
    WHEN 'Germany' THEN 'DE'
    WHEN 'India' THEN 'IN'
    WHEN 'Mexico' THEN 'MX'
    ELSE 'Other'
END
```

---

#### CASE for NULL Handling

```sql
SELECT
    customer_id,
    last_name,
    score,
    CASE
        WHEN score IS NULL THEN 0
        ELSE score
    END AS score_clean,
    AVG(CASE
            WHEN score IS NULL THEN 0
            ELSE score
        END) OVER() AS avg_score_clean
FROM customers;
```

---

## 8. Aggregation & Analytical Functions

### 8.1 Aggregate Functions

Aggregate functions collapse multiple rows into a single summary value.

| Function | Description | Example |
|----------|-------------|---------|
| `COUNT()` | Count rows | `COUNT(*)` |
| `SUM()` | Sum values | `SUM(sales)` |
| `AVG()` | Average | `AVG(score)` |
| `MIN()` | Minimum value | `MIN(price)` |
| `MAX()` | Maximum value | `MAX(order_date)` |

---

#### COUNT()

**COUNT(*)** → Count all rows (including NULLs)
```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```

**COUNT(column)** → Count non-NULL values
```sql
SELECT COUNT(email) AS customers_with_email
FROM customers;
```

---

#### SUM() and AVG()

```sql
SELECT 
    COUNT(*) AS total_orders,
    SUM(sales) AS total_revenue,
    AVG(sales) AS average_order_value
FROM orders;
```

> **⚠️ Note:** `AVG()` ignores NULL values.

---

#### MIN() and MAX()

```sql
SELECT 
    MIN(order_date) AS first_order,
    MAX(order_date) AS last_order,
    MIN(sales) AS lowest_sale,
    MAX(sales) AS highest_sale
FROM orders;
```

---

#### GROUP BY

Combine rows into groups and apply aggregate functions.

```sql
SELECT 
    country,
    COUNT(*) AS customer_count,
    AVG(age) AS avg_age
FROM customers
GROUP BY country;
```

---

### 8.2 Window Functions

**Window Functions** perform calculations across rows **without collapsing them** (unlike `GROUP BY`).

---

#### GROUP BY vs Window Functions

| Feature | GROUP BY | Window Functions |
|---------|----------|------------------|
| **Output** | One row per group | All rows retained |
| **Detail Level** | Summary only | Detail + calculation |
| **Syntax** | Uses `GROUP BY` | Uses `OVER()` |
| **Use Cases** | Simple aggregations | Complex analytics, rankings |

**Example Comparison:**

**GROUP BY (collapses rows):**
```sql
SELECT 
    product_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY product_id;
```

**Window Function (keeps all rows):**
```sql
SELECT 
    order_id,
    product_id,
    order_date,
    sales,
    COUNT(*) OVER(PARTITION BY product_id) AS orders_per_product
FROM orders;
```

---

#### Window Function Syntax

```sql
WINDOW_FUNCTION(expression)
OVER (
    [PARTITION BY column]
    [ORDER BY column]
    [ROWS/RANGE frame_clause]
)
```

**Components:**
1. **Window Function** → `COUNT()`, `SUM()`, `ROW_NUMBER()`, etc.
2. **OVER()** → Defines the window (required)
3. **PARTITION BY** → Divide data into groups (optional)
4. **ORDER BY** → Order rows within each partition (optional)
5. **Frame Clause** → Define subset of rows (optional)

---

#### PARTITION BY

Divide data into groups (like `GROUP BY`, but keeps all rows).

```sql
SELECT 
    order_id,
    product_id,
    sales,
    SUM(sales) OVER() AS total_sales,  -- Overall total
    SUM(sales) OVER(PARTITION BY product_id) AS product_total  -- Per product
FROM orders;
```

---

#### ORDER BY

Order rows within each partition for cumulative calculations.

```sql
SELECT 
    order_id,
    order_date,
    sales,
    SUM(sales) OVER(ORDER BY order_date) AS running_total
FROM orders;
```

---

#### Frame Clause

Define a subset of rows within each partition.

**Syntax:**
```sql
ROWS BETWEEN [start] AND [end]
```

**Frame Boundaries:**
- `UNBOUNDED PRECEDING` → From start of partition
- `N PRECEDING` → N rows before current
- `CURRENT ROW` → Current row
- `N FOLLOWING` → N rows after current
- `UNBOUNDED FOLLOWING` → To end of partition

**Example:**
```sql
SELECT 
    order_id,
    order_date,
    sales,
    AVG(sales) OVER(
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3days
FROM orders;
```

---

#### Window Function Categories

| Category | Functions | Use Case |
|----------|-----------|----------|
| **Aggregate** | `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` | Calculations with details |
| **Rank** | `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE()`, `CUME_DIST()`, `PERCENT_RANK()` | Ranking and percentiles |
| **Value** | `LEAD()`, `LAG()`, `FIRST_VALUE()`, `LAST_VALUE()` | Access other rows |

---

### 8.3 Window Aggregate Functions

#### COUNT()

```sql
-- Count orders per product (shows on every row)
SELECT 
    order_id,
    product_id,
    COUNT(*) OVER(PARTITION BY product_id) AS orders_per_product
FROM orders;
```

---

#### SUM()

```sql
-- Total sales and percentage per order
SELECT 
    order_id,
    product_id,
    sales,
    SUM(sales) OVER() AS total_sales,
    ROUND(CAST(sales AS FLOAT) * 100 / SUM(sales) OVER(), 2) AS percent_of_total
FROM orders;
```

---

#### AVG()

```sql
-- Compare each order to average
SELECT 
    order_id,
    product_id,
    sales,
    AVG(sales) OVER() AS overall_avg,
    AVG(sales) OVER(PARTITION BY product_id) AS product_avg,
    CASE 
        WHEN sales > AVG(sales) OVER() THEN 'Above Average'
        ELSE 'Below Average'
    END AS performance
FROM orders;
```

---

#### MIN() and MAX()

```sql
-- Compare to min/max
SELECT 
    order_id,
    sales,
    MIN(sales) OVER() AS lowest_sale,
    MAX(sales) OVER() AS highest_sale,
    sales - MIN(sales) OVER() AS diff_from_min
FROM orders;
```

---

#### Running and Rolling Totals

**Running Total (Cumulative):**
```sql
SELECT 
    order_date,
    sales,
    SUM(sales) OVER(ORDER BY order_date) AS running_total
FROM orders;
```

**Rolling Total (Fixed Window):**
```sql
SELECT 
    order_date,
    sales,
    SUM(sales) OVER(
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS rolling_3day_total
FROM orders;
```

---

### 8.4 Window Ranking Functions

#### Integer-Based Ranking

**ROW_NUMBER()**
- Assigns unique sequential numbers
- No ties
- No gaps

```sql
SELECT 
    order_id,
    product_id,
    sales,
    ROW_NUMBER() OVER(ORDER BY sales DESC) AS row_num
FROM orders;
```

---

**RANK()**
- Assigns ranks with ties
- Gaps after ties

```sql
SELECT 
    order_id,
    sales,
    RANK() OVER(ORDER BY sales DESC) AS rank_num
FROM orders;

-- Result:
-- sales | rank_num
-- 100   | 1
-- 90    | 2
-- 90    | 2  ← Tie
-- 80    | 4  ← Gap (skipped 3)
```

---

**DENSE_RANK()**
- Assigns ranks with ties
- No gaps

```sql
SELECT 
    order_id,
    sales,
    DENSE_RANK() OVER(ORDER BY sales DESC) AS dense_rank_num
FROM orders;

-- Result:
-- sales | dense_rank_num
-- 100   | 1
-- 90    | 2
-- 90    | 2  ← Tie
-- 80    | 3  ← No gap
```

---

**NTILE(n)**
- Divides rows into n buckets

```sql
SELECT 
    order_id,
    sales,
    NTILE(3) OVER(ORDER BY sales DESC) AS sales_bucket
FROM orders;

-- Result: Divides into 3 groups (high, medium, low)
```

**Use Cases:**
- **Data Segmentation:** Divide customers into tiers
- **Load Balancing:** Split data for parallel processing

---

#### Ranking Function Comparison

| Function | Ties | Gaps | Use Case |
|----------|------|------|----------|
| `ROW_NUMBER()` | No (always unique) | No | Unique IDs, pagination |
| `RANK()` | Yes | Yes | Competition-style ranking |
| `DENSE_RANK()` | Yes | No | Continuous rankings |
| `NTILE(n)` | Groups into buckets | N/A | Segmentation |

---

#### Percentage-Based Ranking

**CUME_DIST()**
- Cumulative distribution
- Formula: `Position / Total Rows`

```sql
SELECT 
    sales,
    CUME_DIST() OVER(ORDER BY sales DESC) AS cumulative_dist
FROM orders;

-- Interpretation: 
-- 0.2 = Top 20% of values
-- 0.8 = Top 80% of values
```

---

**PERCENT_RANK()**
- Relative rank as percentage
- Formula: `(Rank - 1) / (Total Rows - 1)`

```sql
SELECT 
    sales,
    PERCENT_RANK() OVER(ORDER BY sales DESC) AS percent_rank
FROM orders;

-- Interpretation:
-- 0.0  = Highest value
-- 0.5  = Median
-- 1.0  = Lowest value
```

---

### 8.5 Window Value Functions

Access values from other rows in the partition.

#### LEAD() and LAG()

**LAG()** → Access previous row
```sql
SELECT 
    order_date,
    sales,
    LAG(sales, 1, 0) OVER(ORDER BY order_date) AS previous_day_sales
FROM orders;
```

**LEAD()** → Access next row
```sql
SELECT 
    order_date,
    sales,
    LEAD(sales, 1, 0) OVER(ORDER BY order_date) AS next_day_sales
FROM orders;
```

**Parameters:**
- 1st: Column
- 2nd: Offset (default: 1)
- 3rd: Default value for NULLs (default: NULL)

---

#### Time Series Analysis

**Month-over-Month Growth:**
```sql
SELECT 
    MONTH(order_date) AS month,
    SUM(sales) AS current_month,
    LAG(SUM(sales)) OVER(ORDER BY MONTH(order_date)) AS previous_month,
    SUM(sales) - LAG(SUM(sales)) OVER(ORDER BY MONTH(order_date)) AS change,
    CAST((SUM(sales) - LAG(SUM(sales)) OVER(ORDER BY MONTH(order_date))) AS FLOAT) 
        / LAG(SUM(sales)) OVER(ORDER BY MONTH(order_date)) * 100 AS percent_change
FROM orders
GROUP BY MONTH(order_date);
```

---

#### FIRST_VALUE() and LAST_VALUE()

**FIRST_VALUE()** → First value in partition
```sql
SELECT 
    order_id,
    order_date,
    sales,
    FIRST_VALUE(sales) OVER(ORDER BY order_date) AS first_order_amount
FROM orders;
```

**LAST_VALUE()** → Last value in partition

> **⚠️ Important:** Always specify frame for `LAST_VALUE()`

```sql
SELECT 
    order_id,
    order_date,
    sales,
    LAST_VALUE(sales) OVER(
        ORDER BY order_date
        ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
    ) AS last_order_amount
FROM orders;
```

---

### 8.6 Window Function Rules

1. ✅ Window functions only in `SELECT` and `ORDER BY`
2. ❌ Cannot nest window functions
3. ✅ Executed after `WHERE` (use subquery to filter window results)
4. ✅ Can combine with `GROUP BY` if columns match

**Example with GROUP BY:**
```sql
SELECT 
    o.customer_id,
    c.first_name,
    SUM(sales) AS total_sales,
    RANK() OVER(ORDER BY SUM(sales) DESC) AS customer_rank
FROM orders AS o
LEFT JOIN customers AS c
    ON o.customer_id = c.customer_id
GROUP BY o.customer_id, c.first_name;
```

---

## 🎯 Key Takeaways

### Filtering
- ✅ Use comparison, logical, range, and membership operators
- ✅ Pattern matching with `LIKE`, `%`, `_`
- ✅ Combine operators for complex filters

### Joins
- ✅ `INNER JOIN` → Only matches
- ✅ `LEFT/RIGHT JOIN` → All from one side
- ✅ `FULL JOIN` → All from both sides
- ✅ Anti-joins → Find non-matches

### Functions
- ✅ String functions for text manipulation
- ✅ Date functions for temporal analysis
- ✅ Handle NULLs with `COALESCE()` and `NULLIF()`
- ✅ Conditional logic with `CASE`

### Window Functions
- ✅ Keep row-level detail while calculating aggregates
- ✅ Use `PARTITION BY` to group without collapsing
- ✅ Use ranking functions for top-N analysis
- ✅ Use value functions for time series analysis

---

> **Next:** Continue to [ADVANCED_09-12.md](./ADVANCED_09-12.md) for advanced techniques, optimization, and projects.
