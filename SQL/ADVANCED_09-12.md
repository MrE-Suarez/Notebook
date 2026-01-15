# ADVANCED

## ADVANCED TECHNIQUES

The DBA (Data base administrator) needs to explain the data model so that other analysts, data engineer, data scientists, etc can start performing queries and analysis 

Challenges when maintaining a data warehouse:
Redundancy
Performance Issues
Complexity
Hard To Maintain
DB Stress
Security

5 Solutions:
Subqueries
CTE
Views
Temporary tables
CTAS

Data base architecture

Database engine     -> This is the brain of the database, executing several operations 
    Storing, retireving and managing data within the database 
Storage
    DISK storage    -> Long term memory where the data is stored permanently. This stores large amounts of data but this is slow to read and write
        USER    -> Main content of the database
        CATALOG -> This holds the Metadata. Internal storage for its own information. Blueprint of the database. Table name, column name, data type, lenght, is nullable. This is at the Information squema with built-in views: Select * FROM INFORMATION_SCHEMA.COLUMNS 
        TEMP    -> Temporary space used by the database for short-term tasks like processing queries, ince the taks is done the storage is cleared. This is at System Databases > tempdb > Temporary Tables 
    CACHE storage   -> Fast short-term memory
        This is fast to read and write but it can hold smaller amount amount of data

### Subqueries
A query inside another query

DB Tables -> Query -> Result -> Query from that result (subquery) -> Result

Why to use these?
You may plan Join, Filtering, Transformation, Aggregation and it is no mantainable or readable in a single query, so each step can be a different query for readability

### Subquery Categories

BY CORRELATION:
Non-correlated subquery -> Non depandant on the main query
Correlated subquery     -> Depandant on the main query

BY RESULT TYPES:
Scalar Subquery     -> Returns a single value   | Mainly aggregations
Row Subquery        -> Returns multiple rows    | Looking for a single column 
Table Subquery      -> Returns a table          | Multiple columns

BY LOCATION/CLAUSES: Where the subquery will be used within the main query

    SELECT  -> Aggregate data side by side with the main query's data.
        IMPORTANT RULE: The result of the subquery must be a scalar subquery (a single value result)
        SELECT column1, (Select column FROM table1 WHERE condition), ... 
        FROM FROM table 1

    FROM    -> most common, used as temp table for the main query
        SELECT column1, column2, ... 
        FROM (Select column FROM table1 WHERE condition) AS Alias

    JOIN    -> Prepare data (filtering or aggregation) before joining with other tables
        -- Show all customer details and find the total orders of each customer
        SELECT
        c.*,
        o.TotalOrders
        FROM Sales.Customers c
        LEFT JOIN (
            SELECT
            CustomerId, Count(*) TotalOrders
            FROM Sales.Orders
            GROUP BY CustomerID) o
        ON c.CustomerID = o.CustomerID

    WHERE   -> Complex filtering logic making query more flexible and dynamic
        IMPORTANT RULE: Only SCALAR subqueries are allowed
        *Comparison operators -> Comparing 2 values*
        SELECT column1, column2, ...
        FROM table1
        WHERE column = (Select column FROM table1 WHERE condition)
    
    USING LOGICAL OPERATORS

    IN -> Check wheter a value matches any value from a list
    -- Showing the details of orders made by customers in Germany
    SELECT *
    FROM Sales.Orders
    WHERE CustomerID IN(
        SELECT CustomerID
        FROM Sales.Customers
        WHERE Country = 'Germany'
        )

    ANY | ALL (ANY for at least "higher" than one result | ALL for "higher than all)
    SELECT column1, column2, ...
    FROM table1
    WHERE column < ANY (Select column FROM table1 WHERE condition)
    Example:
    -- Find the female employees whose salaries are greather than any male employees salaries 
    SELECT
    EmployeeID,
    FirstName,
    Gender,
    Salary
    FROM sales.Employees
    WHERE Gender ='F'
    AND Salary > ANY (SELECT Salary FROM sales.Employees WHERE Gender ='M')

All the previous were non-correlated subqueries (runing independantly from the main query)
At correlated subqueries, the main runs first, then the subquery row by row (excluding those with no result) as an iteration

Correlated query
    --Show all customer details and find the total orders of each customer
    SELECT
    *,
    (SELECT COUNT(*) FROM Sales.Orders o Where o.CustomerID = c.customerID) TotalSales
    FROM Sales.Customers c
    
    Detailed explanation at https://www.youtube.com/watch?v=SSKVgrwhzus min 14:13:44

    EXISTS -> Check if a subquery returns any rows 
    SELECT column1, column2, ...
    FROM Table2
    WHERE EXISTS( Select 1 FROM Table1 WHERE table1.ID = Table2.ID)
    
    Example:
    -- Show the details of orders made by customers in Germany
    SELECT
    *
    FROM Sales.Orders o
    WHERE EXISTS (
        SELECT 1
        FROM Sales.Customers c
        WHERE Country = 'Germany'
        AND o.CustomerID = c.CustomerID) --Subquery


### CTE - Common Table Expression
This is a temporary, named result set (virtual table) that can be used multiple times within your query to simplify and organize complex query
ORDER BY is not allowed on CTEs
Recommendation: Not to use +5 CTEs

#### None-Recursive CTE
CTE that is executed only once

##### Standalone CTE
Defined and used independently. Self-contained and independient. But dependent Main query
DB -> CTE -> Intermediate RESULT -> MAIN QUERY -> Final RESULT

    WITH CTE-Name AS    -- CTE Query
    (
        SELECT ...
        FROM ...
        WHERE ...
    )

    SELECT ...          -- Main query
    FROM CTE-Name
    WHERE

Multiple standlone CTEs.
We use only 1 WITH, then we separate with coma (,)

        -- Step1: Find the total sales per customer
    WITH CTE_Total_Sales AS
    (
    SELECT
        CustomerID,
        SUM(Sales) AS TotalSales
    FROM Sales.Orders
    GROUP BY CustomerID
    )

        --Step2: Find the lastt order date for each customer
    , CTE_Last_Order AS 
    (
    SELECT
    CustomerID,
    MAX(OrderDate) Last_Order--Last order
    FROM sales.Orders
    GROUP BY CustomerID
    )
        -- Main Query
    SELECT
    c.CustomerID,
    c.FirstName, 
    c.LastName,
    cts.TotalSales,
    ctl.Last_Order
    FROM Sales.Customers c
    LEFT JOIN CTE_Total_Sales cts
    ON cts.CustomerID = c.CustomerID
    LEFT JOIN CTE_Last_Order ctl
    ON ctl.CustomerID = c.CustomerID

##### Nested CTE
CTE inside another CTE. This is dependant from the result of the other CTE

    WITH CTE-Name1 AS   -- Standalone CTE 1
    (
        SELECT ...
        FROM ...
        WHERE ...
    )

    , CTE-Name2 AS      -- Nested CTE
    (
        SELECT ...
        FROM CTE-Name1      
        WHERE ...
    )

    SELECT ...          -- Main query
    FROM CTE-Name2
    WHERE

Practice:

-- Step1: Find the total sales per customer

WITH CTE_TotalSales AS
(
SELECT
	CustomerID,
	SUM(Sales) AS TotalSales
	FROM Sales.Orders
	GROUP BY CustomerID
)

-- Step2: Find the last order date for each customer (standalone CTE)
, CTE_LastOrder AS
(
SELECT
	CustomerID,
	MAX(OrderDate) AS LastOrder
FROM sales.orders
GROUP BY CustomerID
)

-- Step3: Rank Customers based on Total Sales per customer (Nested CTE)
, CTE_CustomerRank AS
(
SELECT
	CustomerID,
	TotalSales,
	Rank() OVER(Order by TotalSales DESC) CxRank
FROM CTE_TotalSales
)

-- Step4: Segment the customers based on Total Sales (Nested CTE)
, CTE_Segment AS
(
SELECT
    CustomerID,
    CASE
        WHEN TotalSales >= 100 THEN 'Great'
        WHEN TotalSales >= 90  THEN 'Good'
        WHEN TotalSales >= 50  THEN 'OK'
        ELSE 'Bad'
    END AS Segmentation
FROM CTE_TotalSales
)

SELECT
c.CustomerID,
c.FirstName,
c.LastName,
cts.TotalSales,
clo.LastOrder,
ccr.CxRank,
cs.Segmentation
FROM Sales.Customers c
LEFT JOIN CTE_TotalSales cts
ON cts.CustomerID = c.CustomerID
LEFT JOIN CTE_LastOrder clo
ON clo.CustomerID = c.CustomerID
LEFT JOIN CTE_CustomerRank ccr
ON ccr.CustomerID = c.CustomerID
LEFT JOIN CTE_Segment cs
ON c.CustomerID = cs.CustomerID

#### Recursive CTE
Self-referencing query that repeatedly processes data until a specific condition is met

##### Syntax
    WITH CTE-Name AS    -- CTE Query
    (
        SELECT ...          --Anchor query
        FROM ...
        WHERE ...
        UNION           -- or UNION ALL

        SELECT ...          --Recursive query
        FROM CTE-Name
        WHERE [Break condition]
    )


Practice example:

WITH Series AS(
			-- Anchor Query
	SELECT 
	1 AS Mynumber
	UNION ALL
			--Recursive query
	SELECT
	Mynumber + 1
	FROM Series
	WHERE Mynumber < 105 -- set iteration limit
)

    -- Main Query
SELECT *
FROM Series
OPTION (MAXRECURSION 150) -- > Control the number of iterations

Employee Hierarchy Example:

    WITH CTE_emp_hierarchy AS
    (
        --Anchor query
    SELECT 
        EmployeeID,
        FirstName,
        managerID,
        1 AS Level
    FROM sales.Employees
    WHERE ManagerID IS NULL

    UNION ALL

        -- Recursive query
    SELECT 
        e.EmployeeID,
        e.FirstName,
        e.ManagerID,
        Level +1
    FROM sales.EMPLOYEES e
    INNER JOIN CTE_emp_hierarchy ceh
    ON e.managerID = ceh.EmployeeID
    )

        -- Main query
    SELECT *
    FROM CTE_emp_hierarchy

### VIEWs
The VIEW is a virtual tale that shows data without storing it physically
Virtual table based on the result set of a query, WITHOUT sotring the data in the database


Database structure:
SQL SERVER
    > DATABASE
        > SCHEMA
            > TABLE
                > COLUMNS
                > KEY, etc.
            > VIEW
                > COLUMNS
            > etc.

We built and manage this structure using Data definition language (DDL), A set of commands that allows us to define and manage the structure of a database. CREATE, ALTER, DROP, etc.

#### 3 levels of Architecture | 3 abstractions

PHYSICAL layer (internal)
    Where the data is stored in a physical storage. Usually DBA can access this
    There are DATA FILES, PARTITIONS, LOGS, CATALOG, BLOCKS, CACHES, etc.
LOGICAL layer (conceptual)
    How to organize/structure your data. Used by Data engineers
    Creating TABLES, defining RELATIONSHIPS, find VIEWS. INDEXES, PROCEDURES, FUNCTIONS, etc.
VIEW layer (external)
    Different VIEWs created according to the users that will use the data. For Business Analysts, Power Bi, other users...


#### Usages

Use case 1: Central Complex Query Logic
Store central complex query logic in the database for access by multiplpe queries, reducing project complexityc -> In case 3 different roles are querying from a table, and they all do SUM and JOIN, it's best to have a VIEW a SUM and JOIN VIEW, so they don't have to query it everytime 

Use case 2: hide complexity by offerin friendly views to users: Easier readability and non repetitive query according to the roles

Use Case 3: Security, provide access to certain columns according to the privileges

Use case 4: Flexibility and dynamic

Use case 5: Multiple languages (for international team)

Use case 6: Data Marts in Data Warehouse System
##### CTE vs VIEW vs TABLE

CTE vs View vs Table — Comparison

| Feature / Concept        | CTE (Common Table Expression)                                      | View                                                   | Table                                              |
|---------------------------|--------------------------------------------------------------------|--------------------------------------------------------|----------------------------------------------------|
| **Definition**            | A temporary, named result set defined within a single query        | A saved SQL query stored in the database               | A physical object that stores data                 |
| **Persistence**           | Exists only during query execution                                 | Persistent                                            | Persistent                                         |
| **Storage**               | No physical storage (virtual)                                      | No physical storage unless materialized               | Physically stores data                             |
| **Lifetime**              | One query only                                                     | Permanent until dropped                               | Permanent until dropped                            |
| **Performance**           | Recomputed every time                                              | Recomputed unless indexed/materialized                | Fastest (data stored physically)                   |
| **Can be Indexed?**       | ❌ No                                                              | ❌ No (unless materialized view)                       | ✅ Yes                                              |
| **Use in Multiple Queries** | ❌ No (one query scope)                                           | ✅ Yes (reusable across queries)                       | ✅ Yes                                              |
| **Best Use Case**         | Breaking complex queries into readable parts                       | Reusing logic, simplifying queries                    | Storing real data                                  |
| **Dependencies**          | Limited (inside one statement)                                     | Stored in metadata, can depend on tables/other views  | Dependencies allow other objects to reference it   |
| **Works with Recursion**  | ✅ Yes (recursive CTEs)                                            | ❌ No                                                  | ❌ Not relevant                                     |
| **Security / Permissions**| None (inherits from query)                                         | Can restrict user access to underlying tables         | Full security options                              |
| **Can Be Updated?**       | ❌ Not directly                                                     | Sometimes (updatable views)                           | ✅ Always                                           |
| **Complex Query Support** | Great for hierarchical data, window functions                      | Good for abstraction                                   | N/A (data container)                               |
| **Example**               | `WITH CTE AS (...) SELECT ...`                                     | `CREATE VIEW v AS SELECT ...`                         | `CREATE TABLE t (...)`                             |

#### Syntax

##### CREATE
CREATE VIEW View-Name AS  -- DDL Command
(
    SELECT  ...
    FROM    ...
    WHERE   ...
)

In case the following CTE is commonly used, we can add this as a view to make queries easier

    WITH CTE_Monthly_Summary AS(
        SELECT
        DATETRUNC(month, OrderDate) OrderMonth,
        SUM(Sales) TotalSales
        FROM Sales.Orders
        GROUP BY DATETRUNC(month, OrderDate)
    )

    SELECT
    OrderMonth,
    TotalSales,
    SUM(TotalSales) OVER(ORDER BY OrderMonth) AS RuningTotal
    FROM CTE_Monthly_Summary

... In a different query:
    CREATE VIEW Sales.V_Monthly_Summary AS -- Schema name before the dot
    (
    SELECT
        DATETRUNC(month, OrderDate) OrderMonth,
        SUM(Sales) TotalSales,
        COUNT(OrderID) TotalOrders,
        SUM(Quantity) TotalQuantities
        FROM Sales.Orders
        GROUP BY DATETRUNC(month, OrderDate)
    )

##### UPDATE the logic (replace)

Valid on Postgrees:
    CREATE or REPLACE VIEW Sales.V_Monthly_Summary

Valid on SQL Server:
    DROP it, and then CREATE it again

    Also, T-SQL (Extension with programming features)
        IF OBJECT_ID ('Sales.V_Monthly_Summary', 'V') IS NOT NULL -- It exists
        DROP VIEW Sales.V_Monthly_Summary;
        GO
        CREATE VIEW Sales.V_Monthly_Summary AS
        (
            ...
        )

Therefore, if it's made already, this could just update the columns


### CTAS Table & Temp Tables - CREATE TABLE AS SELECT

#### TABLE types
##### Permanent Tables
These stay there unless you drop them
We can create these:
###### CREATE + INSERT
        Define the structure of the table, CREATE the table
        INSERT Data into the existing table, 
###### CTAS (CREATE TABLE AS SELECT)
        Create a new table based on the result of an SQL query  
Temporary Tables -> These are dropped once the session ends 

Syntax: CREATE + INSERT 
Create + Insert
CREATE TABLE Table-Name
(
    ID INT,
    Name Varchar (50)
)

INSERT INTO Table-Name
VALUES (1,'Frank')

CTAS --Most SQL
CREATE TABLE Name AS
(
SELECT ...
FROM ...
WHERE ...
)

CTAS -- SQL Server only <shortcut>
SELECT ...
INTO New-Table
FROM ...
WHERE ...

###### Usages

Use case 1: Optimize performance
    Used when the VIEW is making it lack on performance, we better use CTAs 
    Example:
        SELECT
            DATENAME(month,OrderDate) OrderMonth,
            Count(OrderID) TotalOrders
        INTO Sales.MonthlyOrders
        FROM Sales.Orders
        GROUP BY DATENAME(month, OrderDate)

Use case 2: Create a Snapshot
    Making analysis while updating the data at the same time

Use case 3: Physical Data Marts in DWH
    Using Data Marts (tables) instead of VIEWs to improve performance

###### Refresh - UPDATE the content of a CTAS
Option 1: Drop it and create it again
Option 2: T-SQL (extension fro programming)
    IF OBJECT_ID('Sales.MonthlyOrders', 'U') IS NOT NULL -- U for user defined table 
        DROP TABLE Sales.MonthlyOrders;
    GO
    SELECT  ...
    INTO    ...
    FROM    ...

##### Temporary Tables
These store intermediate results in temporary storage within the database during the runnign session

###### Syntax: Temp Table

SELECT  ...
INTO    #SCHEMA.New-Table      -- The hash makes it TEMP
FROM    ...
WHERE   ...

###### Usages

Use case 1: Intermediate Results
    Useful when extracting th data from the source Database, in order to transform the data and make it part of the DWH Database later as a table

### Stored Procedure
Stored Procedures are precompiled SQL programs stored inside the database. They allow multiple SQL statements to be executed together and support parameters, loops, conditions, and error handling.

Advantages:
- High performance (execution plan caching)
- High security (permission abstraction)
- Consistent business rules
- Less network overhead

Disadvantages:
- Harder to version control
- More difficult to test/debug
- Tightly coupled to the DB vendor
- Not ideal for complex business logic

#### Python vs Stored Procedures
- Use Stored Procedures for data manipulation, transformations inside the DB, and performance-critical operations.
- Use Python for ETL, orchestration, business logic, and flexible transformations.

... Skipped for now as it's tending to be outdated.
Video: 17:36:35 - 18:12:48 (36m)




### Triggers
Special stored procedure (set of statements) that automatically runs in response to a specific event on a table or view.

#### Trigger Types
DML
    INSERT
    UPDATE
    DELETE

    There are AFTER and INSTEAD OF
    AFTER > Runs after the event
    INSTEAD OF > Runs during the event

DDL
    CREATE
    ALTER
    DROP

LOGGON

#### Syntax: Triggers

CREATE TRIGGER TriggerName ON TableName
AFTER INSERT,UPDATE,DELETE                  -- When this should start
BEGIN
-- SQL STATEMNETS HERE                      -- What will happen
END

EXAMPLE:
CREATE TRIGGER trg_AfterInsertEmployee ON Sales.Employees
AFTER INSERT
AS
BEGIN
	INSERT INTO Sales.EmployeeLogs (EmployeeID, LogMessage,LogDate)
	SELECT
		EmployeeID,
		'New Employee Added =' + CAST(EmployeeID AS VARCHAR), 
		GETDATE()
	FROM INSERTED 
END

## PERFORMANCE OPTIMIZATION

### Indexes
Data structure that provides quick access to data, optimizing the speed of queries

-------- Index tables

#### Index Structure
##### Clustered Index
A clustered index defines the physical order of the rows in the table.
Internally, it is stored as a B-Tree (balanced tree) with:
- Root node
- Intermediate index nodes
- Leaf level → Data pages (actual rows)

Index nodes contain key values and pointers that guide the engine to a range.
Data pages at the leaf contain the real table rows, sorted by the clustered index key.

Structure:

                Root
    /         |         \
    Index     Index      Index
    |          |          |
    Data       Data       Data
    Page       Page       Page

Root → Intermediate → Leaf (Data Pages)
- Leaf = actual rows
- Rows are physically sorted
- Only one clustered index per table

##### Non-Clustered Index
A non-clustered index is a separate B-Tree from the table.
Its leaf level does not contain data rows.
Instead, it stores:
- RID (Row ID) → FileID : PageNumber : RowOffset
(for heaps — tables without clustered index)
OR...
- Clustered Key
(if the table already has a clustered index)

This leaf entry points to where the actual data is stored.
The data pages are not sorted based on the non-clustered index.

Structure:

                 Root
        /         |         \
     Index     Index      Index
       |          |          |
   Index Page  Index Page  Index Page   <-- points to data pages

Root → Intermediate → Leaf (Index Pages → Data Page Pointer)

- Leaf = pointers (RID or clustered key)
- Data pages are separate, not part of the index B-Tree
- You can have many non-clustered indexes

##### Clustered vs Non-Clustered Indexes
Clustered index =       table of contents
Non-Clustered index =   index at the end of the book


Clustered:
Root Node → Index Pages
Intermediate Nodes → Index Pages
Leaf Level → Data pages (physically sorted rows)

Non-Clustered
Root Node → Index Pages
Intermediate Nodes → Index Pages
Leaf Level → Index rows pointing to data pages
(Data pages are not part of the B-Tree and are not sorted)

There can be multiple Non-Clustered Indexes, but only one Clustered Index per table.

Best for…

Faster reads → Clustered Index

Faster writes → fewer indexes overall (not a specific type)


Storage efficiency → fewer indexes; heaps require the least storage

Clustered Index usage:
Unique or mostly unique columns
Not frequently modified columns
Improve range and ordering performance

Non-Clustered Index usage:
Columns used in search conditions and joins
Exact match lookups
Covering queries

##### Syntax: Index CLUSTERED / NONCLUSTERED

CREATE [CLUSTERED | NONCLUSTERED] INDEX index_name ON table_Name (column1,column2, ...)
    Example:
    CREATE CLUSTERED INDEX IX_Customers_ID ON Customers (ID)
    CREATE INDEX IX_Customers_Name ON Customes (LastName ASC, FirstName DESC) -- NONCLUSTERED default

    In case we query FirstName often on a table, we can index it to enhance the performance, this would be NONCLUSTERED since there are no unique values in the FirstName column:
        CREATE INDEX idx_DBCustomer_FirstName -- NONCLUSTERED default
        ON Sales.DBCustomers (FirstName)

Composite Index
This has multiple columns inside the same index
MUST: The order of the query must match the order of the index

    Frequent Query:
    SELECT *
    FROM SalesDBCustomers
    WHERE Country = 'USA' AND Score > 500

    Index creation:
    CREATE INDEX idx_DBCustomers_CountryScore
    ON SalesDBCustomers (Country,Score)

SQL can use this index even if we query by only one column, but this MUST be the left one.
Since Country is at the left of score...

- Works: WHERE Country condition
- Partially works: WHERE Country = ... AND City = ...
(The index helps with Country, but not with City because City is not part of the index key.)
- Doesn’t work: WHERE Score = ... (Score alone cannot use the index because it’s not the leftmost column.)



##### Index Storage

Columnstore Index

Stores the data column by column

Process:

1- Separete the data in Row Groups
2- Column segment
3- Compression -> Creates a dictionary replacing the values for smaller values like numbers
4- Store the result in data pages LOB
5- Definition CLUSTERED or NONCLUSTERED
    CLUSTERED -> Removes the original Rowstore HEAP table
    NONCLUSTERED -> keeps the original Roswtore HEAP table as companion 

Rowstore Index
    Stores the data row by row
    Just as usual

Columnstore vs Rowstore

Definition:
Rowstore    - Organizes the data row by row
Colunstore  - Organizes the data column by column

Storage:
Rowstore    - Less efficient
Columnstore - High efficient with compression

Read/Write:
Rowstore    - Fair speed for both 
Columnstore - Faster reading / Slow writing

Input Output efficiency:
Rowstore    - Lower, Retrieves all collumns
Columnstore - Higher, retrieves specific columns

Suitable system:
Rowstore    -  OLTP (Transactional), commerce, banking, order processing. 
Columnstore - OLAP (Analytical), Data Warehouse, Business intelligence, reporting, analytics

Use case:
Rowstore    - High-frequency transaction apps, quick access to complete records 
Columnstore - Big data analytics, fast aggregation

###### Syntax: Columnstore Index 

CREATE [CLUSTERED | NONCLUSTERED] [COLUMNSTORE]  INDEX index_name
ON table_Name                                                       -- (Do not specify columns)

    Example:
    CREATE CLUSTERED COLUMNSTORE INDEX idx_DBCustomers_Cs
    ON Sales.DGCustomers (CustomerID)

There can be only 1 clustered/nonclustered columnstore in SQL SERVER. Others allow multiple nonclustered indexes.
But rowstore can accept 1 clustered, several nonclustered

###### Syntax: Rowstore Index 
CREATE [CLUSTERED | NONCLUSTERED]  INDEX index_name     -- Rowstore is default
ON table_Name                                           -- (Do not specify columns)


##### Unique Index
This ensures no duplicate values in a specific column 
- Enforces uniqueness
- Slightly performance increase
It's Slower WRITING than non-unique
But Faster READING than non-unique 


###### Syntax: Unique index
CREATE [UNIQUE] [CLUSTERED | NONCLUSTERED] [COLUMNSTORE]  INDEX index_name
ON table_Name (column1, column2, ...)     

    Example:
    CREATE UNIQUE NONCLUSTERED INDEX idx_Products_Product
    ON Sales.Products (Product)


##### Filtered index
An index that includes only rows meeting the specified conditions
Targeted optimization 

###### Syntax: Filtered index
CREATE [UNIQUE] [CLUSTERED | NONCLUSTERED] [COLUMNSTORE]  INDEX index_name
ON table_Name (column1, column2, ...)     
WHERE [Condition]

Some servers such as SQL Server can be restricted:
- NO filtered index on a clustered index
- NO filtered indes oon a colunstore index

    Example:
    If frquently filtering WHERE Country = 'USA'

    CREATE NONCLUSTERED INDEX idx_Customers_Country
    ON Sales.Customers (Country)
    WHERE Country = 'USA'

#### Use the right index

HEAP(no index)  -> Staging tables
CLUSTERED       -> Primary Keys (PK) / OLTP systems
COLUMNSTORE     -> Analytical Queries / OLAP systems
NONCLUSTERED    -> For non-PK (Foreign Keys, Joins and Filters)
FILTERED        -> Target Subset of Data
UNIQUE          -> Enforce Uniqueness

#### Index Management and Monitoring
Indexes need maintenance

##### 1- Monitor index usage
Are indexes being used? Are indexes improving something?
Find the usage of the indexes

Get description of indexes:
sp_helpindex 'TableName'

Get metadata of indexes and tables:
SELECT * FROM sys.indexes
or
SELECT
    tbl.name AS TableName,
    idx.name AS IndexName,
    idx.type_desc AS IndexType,
    idx.is_primary_key AS IsPrimaryKey,
    idx.is_unique AS ISUnique,
    idx.is_disabled AS IsDisabled
FROM sys.indexes idx
JOIN sys.tables tbl             -- We need another table to call the table Name
ON idx.object_id = tbl.object_id
ORDER BY tbl.name, idx.name

Get the count of usage:
SELECT * FROM sys.dm_db_index_usage_stats
or
SELECT
    tbl.name AS TableName,
    idx.name AS IndexName,
    idx.type_desc AS IndexType,
    idx.is_primary_key AS IsPrimaryKey,
    idx.is_unique AS ISUnique,
    idx.is_disabled AS IsDisabled,
    s.user_seeks AS UserSeeks,
    s.user_scans AS UserScans,
    s.user_lookups AS UserLookups,
    s.user_updates AS UserUpdates ,
    COALESCE(s.last_user_seek,s.last_user_scan) LastUpdate 
FROM sys.indexes idx
JOIN sys.tables tbl             -- We need another table to call the table Name
ON idx.object_id = tbl.object_id
LEFT JOIN sys.dm_db_index_usage_stats s
ON s.object_id = idx.object_id
AND s.index_id = idx.index_id
ORDER BY tbl.name, idx.name

Tip:
Discuss with the team wether to DROP the unused indexes or keep these in order to
1- Save Storage
2- Improve Write Perofrmance

##### 2- Monitor missing indexes
You can get recommendations about missing indexes

SELECT * FROM sys.dm_db_missing_index_details

##### 3- Monitor duplicate indexes
Decide what type of indexes you'd like to keep if duplicated

SELECT
    tbl.name AS TableName,
    col.name AS IndexColumn,
    idx.name AS IndexName,
    idx.type_desc AS IndexType,
    COUNT(*) OVER(PARTITION BY tbl.name, col.name) ColumnCount --Flag to check repeated
FROM sys.indexes idx
JOIN sys.tables tbl ON idx.object_id = tbl.object_id
JOIN sys.index_columns ic ON idx.object_id = ic.object_id AND idx.index_id = ic.index_id
JOIN sys.columns col ON ic.object_id = col.object_id AND ic.column_id = col.column_id
ORDER BY tbl.name, col.name

##### 4- Update Statistics
Database engines use statistics to understand which index should be used. If this are not up to date, SQL will make wrong decisions

Get the lasat time statistics were updated:

SELECT
    SCHEMA_NAME(t.schema_id) AS SchemaName,
    t.name AS TableName,
    s.name AS StatisticsName,
    sp.last_updated AS LastUpdate,
    DATEDIFF(day, sp.last_updated, GETDATE()) AS LastUpdateDay,
    sp.rows AS 'Rows',
    sp.modification_counter AS ModificationSinceLastUpdate
FROM sys.stats s
JOIN sys.tables t
ON s.object_id = t.object_id
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
ORDER BY
sp.modification_counter DESC;

Choose the outdated and update this by their name

Get 1 statistic updated:
UPDATE STATISTICS Sales.Employees PK_employees

Get all the table's statistics updated:
UPDATE STATISTICS Sales.Employees

Get the whole database updated:     -- It may take longer
EXEC sp_updatestats

###### Recommendations - When to update stats
1- Weekly job to update stastics on weekends (depending on DB size)
2- After migrating data


##### 5- Monitor fragmentations
Over the time, indexes can become fragmented
    Fragmented: unused spaces in data pages or data pages out of order

Fragmentation Methods
    Reorganize:
        - Defragments leaf nodes to keep them sorted
        - "Light" Operation

    Rebuild
        - Recreates index from Scratch
        - "Heavy" Operation

Find out the fragmentation level:
SELECT *
FROM sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, 'LIMITED')

Find out the specific fragmented tables and level:      -- more useful

SELECT
    tbl.name AS TableName,
    idx.name AS IndexName,
    s.avg_fragmentation_in_percent,
    s.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') AS s
INNER JOIN sys.tables tbl
ON s.object_id = tbl.object_id
INNER JOIN sys.indexes AS idx
ON idx.object_id = s.object_id
AND idx.index_id = s.index_id
ORDER BY s.avg_fragmentation_in_percent DESC

0   means no fragmentation
100 means the index is completely fragmented 


###### Reorganize method
ALTER INDEX [indexName] ON [TableName] REORGANIZE

###### Rebuild method
Drop the whole index and create it from scratch
ALTER INDEX [indexName] ON [TableName] REBUILD


###### When to defragment? --Recomendation
<10%    No action needed
10-30%  Reorganize
>30%    Rebuild

#### Execution plan
Roadmap generated by a database on how it will execute your query step by step
Understand teh procedure in order to know where to optimize your plan 

Display Estimated Execution Plan    -- Before
    Predict the exectuion plan without running the query

Include Actual Execution Plan       -- After
    Shows the execution plan as it occurred after running the query

Include Live Query Statistics       -- Live 
    Shows real-time execution flow as the query runs

##### How to read
We read right to left
Check for CPU cost to confirm which consumes more CPU

Types of scan:

Table scan
Reads the entire table page by page and row by row - Everything

Index Scan
Scans all data in an index to find matching rows - CLUSTERED or NONCLUSTERED

Index Seek
Quickly locates specific rows in an index

Types of Join algorithms:

Nested Loops
Compares tables row by row; best for small tables

Hash Match
Matches rows using hash table; best for large tables

Merge Join
Merge two sorted tables; efficient when both are sorted

Estimated vs Actual Plan
    When these don't match, it may mean something in the index or statistics is wrong, leading poor performance

##### Comparing Execution plans
It is also possible to save any execution plan by right click and save and then compare it with another execution plan by right click on the plan and then compare execution plan

##### SQL Hints
Commands you can add to aquery to force the database to run it in a specific way for better performance

Hinting Hash JOIN:
SELECT 
    o.Sales
    c.Country
FROM Sales.Orders o
LEFT JOIN Sales.Customers c
ON o. CustomerID = c.CustomerID
OPTION (HASH JOIN)                              -- Hint

Forcing INDEX seek:
SELECT 
    o.Sales
    c.Country
FROM Sales.Orders o
LEFT JOIN Sales.Customers c WITH (INDEX([IndexName]))    -- Forcing index
ON o. CustomerID = c.CustomerID

Forcing INDEX seek:
SELECT 
    o.Sales
    c.Country
FROM Sales.Orders o
LEFT JOIN Sales.Customers c WITH (FORCESEEK)    -- Forcing
ON o. CustomerID = c.CustomerID

Recommendations:
1- Test hints in both DEB and PROD environment,a s performance may vary
2- Hints are quick fixes (workaround solutions). But it's still necessary to find the cause and fix it

#### Index strategy
AVOID OVER INDEXING - confuses the execution plan

##### 1- Initial strategy
What is the main goal?

Analytical:     OLAP -  Online Analytical Processing
GOAL: optimize READ performance
Columstore index

Transaction:    OLTP - Online Transaction Processing
GOAL: optimize WRITE performance
Clustered index pk

##### 2- Usage patterns indexing

1. Identify frequently used Tables & Columns
2. Choose right index according to the needs
3. Test once applied index

##### 3- Scenario-Based indexing

Analazy used queries 
1. Identify slow queries
2. Check execution plan for each
3. Choose the right index
4. Compare execution Plans - Testing

##### 4- Monitoring & Maintenance
1. Monitor the index USAGE - Frequency
2. Monitor MISSING inexes
3. Monitor DUPLICATE indexes
4. Updatre STATISTICS
5. Monitor FRAGMENTATIONS

### Partitions

Dividing a table into smaller partitions in order to improve performance (Writing/ Reading).

#### Create Partition function 

1. Partition Function 
Define th elogic on how to divide the data. Based on Partition Key (Date,Id, Region )
Syntax:
    CREATE PARTITION FUNCTION PartitionByYear (DATE)
    AS RANGE LEFT FOR VALUES ('2023-12-31', '2024-12-31', '2025-12-31')

Confirm its existance/creation:
    SELECT
        name,
        function_id,
        type,
        type_desc,
        boundary_value_on_right
    FROM sys.partition_functions

2. File Groups
Logical container of one or more data files. To help organize partitions

Creating empty FileGroups:
ALTER DATABASE SalesDB ADD FILEGROUP FG_2023;
ALTER DATABASE SalesDB ADD FILEGROUP FG_2024;
ALTER DATABASE SalesDB ADD FILEGROUP FG_2025;
ALTER DATABASE SalesDB ADD FILEGROUP FG_2026;

Drop a FileGroup:
ALTER DATABASE SalesDB REMOVE FILEGROUP FG_2026;

Confirm FileGroup existance/creation:
SELECT *
FROM sys.filegroups
WHERE type = 'FG'

3. Create Data Files (.ndf)
Files for the FileGroups

Add .ndf files into each FileGroup:

    ALTER DATABASE SalesDB ADD FILE
    (
        NAME = P_2023           -- Logical name of the file
        FILENAME = 'C:\Program Files\Microsoft SQL Server\MSSQL16.SQLEXPRESS\MSSQL\DATA\P_2023.ndf'                     -- Path, depending on the version-> MSSQL\DATA\dilename
    ) TO FILEGROUP FG_2023

Confirm its existance:
    SELECT
        fg.name AS FilegroupName,
        mf.name AS LogicalFileName,
        mf.physical_name AS PhysicalFilePath,
        mf.size / 128 AS SizeInMB
    FROM
        sys.filegroups fg
    JOIN
        sys.master_files mf ON fg.data_space_id = mf.data_space_id
    WHERE
        mf.database_id = DB_ ID('SalesDB');

4. Partition Scheme
Map each FileGroup to its proper partition

    CREATE PARTITION SCHEME SchemePartitionByYear
    AS PARTITION PartitionByYear
    TO (FG_2023, FG_2024, FG_2025, FG_2026)     -- Respect the order the FGs were created

Confirm its existance/creation:

    SELECT
        ps.name AS PartitionSchemeName,
        ps.name AS PartitionFunctionName,
        ds.destination_id AD PartitionNumber
        fg.name AS FilegroupName
    FROM sys.partition_schemes ps
    JOIN sys.partition_functions pf ON ps.function_id pf.function_id
    JOIN sys.destination_data_spaces ds ON ps.data_space_id = ds.partition_scheme_id
    JOIN sys.filegroups fg ON ds.data_space_id = fg.data_spacie_id

5. Create Partitioned Table

Create table:
    CREATE TABLE sales.Orders_PArtitioned
    (
        OrderID INT,
        OrderDate DATE,
        Sales INT
    ) ON SchemePartitionByYear (OrderDate)       -- Partition Scheme Name

Insert Data:
    INSERT INTO Sales.Orders_Partitioned VALUES (1, '2023-05-15', 100);

Confirm data has been isnterted
    SELECT
        p.partition_number AS PartitionNumber,
        f.name AS PartitionFilegroup
        p.rows AS NumberOfRows
    FROM sys.partitions p
    JOIN sys.destination_data_spaces dds ON p.partition_number = dds.destination_id
    JOIN sys.filegroups f ON dds.data_space_id = f.data_space_id
    WHERE OBJECT_NAME (p.object_id) = 'Orders_Partitioned';

### Performance Tips
For small / medium tables (hundreds of thousands of data), there won't be any performance difference due to the size.
This changers for big data sets.
Always check the execution plan to choose the best query, if there's no difference, pick the easier to read one.

#### FETCHING DATA TIPS 

##### 1- Select only what you need
Do not query SELECT *, list the columns you need only

##### 2- Avoid unnecessary DISTINCT & ORDER BY 
There may be not repeted data, and not necessary to sort it

##### 3- For exploration, limit rows
Best to use TOP

#### FILTERING DATA TIPS

##### 4- Create non clustered index on frequently used columns in WHERE Clause
CREARTE NONCLUSTERED INDEX Idx_tableName_Column ON tableName(Column)

##### 5- Avoid applying functions to columns in WHERE clause
If there's a func in the column, SQL will skip the index

##### 6- Avoid using leading wildcards, as they prevent index usage 
WHERE LastName LIKE '%Gold%'    - wildcard at the beginning skips the index
WHERE LastName LIKE 'Gold%'     - This is good

##### 7- Use IN instead of multiple OR conditions
WHERE CustomerID = 1 OR CustomerID = 2 OR CustomerID = 3        -- Bad Practice
WHERE CustomerId IN (1, 2, 3)       -- Good Practice

#### JOINING DATA BEST PRACTICES

##### 8- Understand the speed of joins
INNER JOIN          -- Best Performance
RIGHT/LEFT JOIN     -- Slightly slower performance
OUTER JOIN          -- Worst Performance

##### 9- Use Explicit Join (ANSI Join) Instead of implicit Join (non-ANSI Join)
Regular Join syntax is best

##### 10- Make Sure to Index the columns used in the ON clause
Ensure taht the columns used in the ON clase are indexed

##### 11- Filter before joining (specilally on big tables)

-- Filter After Join (WHERE)        Good enough for small/medium DB
SELECT c.FirstName, o.OrderID
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.customerID = o.CustomerID
WHERE o.OrderStatus = 'Delivered'

-- Filter during Join (AND)
SELECT c.FirstName, o.OrderID
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.customerID = o.CustomerID
AND o.OrderStatus = 'Delivered'

-- Filter Before Join (SUBQUERY)    BEST!! for Large DB
SELECT c.FirstName, o.OrderID
FROM Sales.Customers c
INNER JOIN (SELECT OrderID, CustomerID FROM Sales.Orders WHERE OrderStatus = 'Delivered') o
ON c.customerID = o.CustomerID

Or using a CTE and joining the result also means best performance

##### 12- Aggregate before joining (specilally on big tables)

-- Grouping and Joining         Good for small/medium DB
SELECT c.CustomerID, c.FirstName, COUNT(o.OrderID) AS OrderCount
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID, c.FirstName

-- Pre-aggregated Subquery      BEST! for Big DB
SELECT c.CustomerID, c.FirstName, o.OrderCount
FROM Sales.Customers c
INNER JOIN (
        SELECT CustomerID, count(OrderID) AS OrderCount
        FROM Sales.Orders
        GROUP BY CustomerID
) o
ON c.CustomerID = o.CustomerID

Prepare the data first, then join

-- Correlated Subquery          WORSE PERFORMANCE!
SELECT
c.customerID,
c.FirstName,
(SELECT COUNT(o.OrderID)
FROM Sales.Orders o
WHERE o.CustomerID = c.CustomerID) AS OrderCount
FROM Sales.Customers c

##### 13- Use UNION instead of OR joins

SELECT o.OrderID, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.CustomerID = o.CustomerID
OR c.CustomerID = o.SalesPersonID       -- OR is Performance killer

SELECT o.OrderID, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.CustomerID = o.CustomerID
UNION
SELECT o.OrderId, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.CustomerID = o.SalesPersonID

Even though it looks larger, it performs better. Specially for big tables

##### 14- Check for Nested loops and use SQL Hints
Check the execution plan when joining
If the Physical Operation is 'Nested Loops', specially for big tables
Then use SQL Hints -> Hash Join

SELECT o.OrderID, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.CustomerID = o.customerID
OPTION (HASH JOIN)                  -- hinting for Hash Match

#### UNION Best Practices

##### 15- Use UNION ALL instead of UNION | Duplicates are acceptable

SELECT CustomerID FROM Sales.Orders
UNION ALL
SELECCT CustomerID FROM Sales.OrdersArchive

##### 16- Use UNION ALL + Distinct instead of UNION | Duplicates are NOT acceptable

SELECT DISTINCT CustomerID
FROM(
    SELECT CustomerID FROM Sales.Orders
    UNION ALL
    SELECCT CustomerID FROM Sales.OrdersArchive
) AS CombinedData

#### Aggregating Data Best Practices

##### 17- Use COLUMNSTORE index for Aggregations on large table
SELECT CustoemrID, COUNT(OrderID) AS OrderCount
FROM Sales.Orders
GROUP BY CustomerID

CREATE CLUSTERED COLUMNSTORE INDEX Idx_Orders_Columnstore ON Sales.Orders

##### 18- Pre-Aggregate Data and store it in new table for Reporting

SELECT MONTH(OrderDate) OrderYearl, SUM(Sales) AS TotalSales
INTO Sales.SalesSummary
FROM Sales.Orders
GROUP BY MONTH(OrderDate)

SELECT OrderYear, TotalSales FROM Sales.SalesSummary

Creating a pre-set table, just remember to have it up to date

#### Subqueries Best Practices

##### 19- JOIN vs EXISTS vs IN

--JOIN (Best Practice: If the Performance equals to EXISTS)
SELECT o.OrderID, o.Sales
FROM Sales.Orders o.
INNER JOIN Sales.Customers c
ON o.CustomerID = c.CustomerID
WHERE c.Country = 'USA'

--EXISTS (Best Practice Overall)
SELECT o.OrderID, o.Sales
FROM Sales.Orders o
WHERE EXISTS (
    SELECT 1
    FROM Sales.Customers c
    WHERE c.CustomerID = o.CustomerID
    AND c.Country = 'USA'
)

--IN (Bad Practice -for large tables-)
SELECT o.OrderID, o.Sales
FROM Sales.ORders o
WHERE o.CustomerID IN (
    SELECT CustomerID
    FROM Sales.Customers
    FROM Sales.Customers
    WHERE Country = 'USA'
)

##### 20- Avoid Redundant Logic in Your Query

-- Bad Practice
SELECT EmployeeID, FirstName, 'Above Average' Status
FROM Sales.Employees
WHERE Salary > (SELECT AVG(Salary) FROM Sales.Employees)
UNION ALL
SELECT ERmployeeID, FirstName, 'Below Average' Status
FROM Sales.Employees
WHERE Salary < (SELECT AVG(Salary) FROM Sales.Employees)

Redundancies:
    - We are scanning sales.emplyees 4 times
    - AVG is being calculated twice

-- Good Practice
SELECT
    EmployeeID,
    firstName,
    CASE
        WHEN Salary > AVG(Salary) OVER () THEN 'Above Average'
        WHEN Salary < AVG(Salary) OVER () THEN 'Below Average
    END AS Status
FROM Sales.Employees

#### Creating Tables (DDL) Best Practices

##### 21- Avoid Data types VARCHAR & TEXT

-- Bad Practice
CREATE TABLE CustomersInfo(
    CustomerID INT,
    FirstName VARCHAR(MAX),
    LastName TEXT,
    Country VARCHAR(255),
    TotalPurchases FLOAT,
    Score VARCHAR(255),
    BirthDate VARCHAR(255),
    EmployeeID INT,
    CONSTRAINT FK_ CustomersInfo_EmployeeID FOREIGN KEY (EmployeeID)
        REFERENCES Sales.Employees(EmployeeID)
)

VARCHAR and TEXT (TEXT is worse) consume many resources and may cause issues with data fragmentations

-- BEST Practice avoiding TEXT and VARCHAR as much as possible
CREATE TABLE CustomersInfo(
    CustomerID INT,
    FirstName VARCHAR(MAX),
    LastName VARCHAR(50),       -- Changed
    Country VARCHAR(255),
    TotalPurchases FLOAT,
    Score INT,                  -- Changed
    BirthDate DATE,             -- Changed
    EmployeeID INT,
    CONSTRAINT FK_ CustomersInfo_EmployeeID FOREIGN KEY (EmployeeID)
        REFERENCES Sales.Employees(EmployeeID)
)

##### 22- Avoid (MAX) unnecessarily or large lenghts

CREATE TABLE CustomersInfo(
    CustomerID INT,
    FirstName VARCHAR(50),     -- Changed
    LastName VARCHAR(50),
    Country VARCHAR(50),       -- Changed
    TotalPurchases FLOAT,
    Score INT,
    BirthDate DATE,
    EmployeeID INT,
    CONSTRAINT FK_ CustomersInfo_EmployeeID FOREIGN KEY (EmployeeID)
        REFERENCES Sales.Employees(EmployeeID)
)

##### 23- Use the NOT NULL constraing where applicable

CREATE TABLE CustomersInfo(
    CustomerID INT,
    FirstName VARCHAR(50) NOT NULL,         -- Updated
    LastName VARCHAR(50) NOT NULL,          -- Updated
    Country VARCHAR(50) NOT NULL,           -- Updated
    TotalPurchases FLOAT,
    Score INT,
    BirthDate DATE,
    EmployeeID INT,
    CONSTRAINT FK_ CustomersInfo_EmployeeID FOREIGN KEY (EmployeeID)
        REFERENCES Sales.Employees(EmployeeID)
)

##### 24- Ensure all your tables have a Clustered Primary Key

CREATE TABLE CustomersInfo(
    CustomerID INT PRIMARY KEY CLUSTERED,       -- Updated
    FirstName VARCHAR(50) NOT NULL,         
    LastName VARCHAR(50) NOT NULL,
    Country VARCHAR(50) NOT NULL,
    TotalPurchases FLOAT,
    Score INT,
    BirthDate DATE,
    EmployeeID INT,
    CONSTRAINT FK_ CustomersInfo_EmployeeID FOREIGN KEY (EmployeeID)
        REFERENCES Sales.Employees(EmployeeID)
)

##### 25- Create a non-clustered index for foreign that are used frequently

CREATE TABLE CustomersInfo(
    CustomerID INT PRIMARY KEY CLUSTERED,       -- Updated
    FirstName VARCHAR(50) NOT NULL,         
    LastName VARCHAR(50) NOT NULL,
    Country VARCHAR(50) NOT NULL,
    TotalPurchases FLOAT,
    Score INT,
    BirthDate DATE,
    EmployeeID INT,
    CONSTRAINT FK_ CustomersInfo_EmployeeID FOREIGN KEY (EmployeeID)
        REFERENCES Sales.Employees(EmployeeID)
)

CREATE NONCLUSTERED INDEX IX_CustomersInfo_EmployeeID
ON CustomersInfo(EmployeeID)

#### INDEXING Best Practices

##### 26- Avoid Over Indexing
Too many indexes slow down insert, update, delete operations
This also confuses the execution plan, therefore the performance

##### 27- Drop unused Indexes
Monitor the usage of indexes to improve space and therefore performance

##### 28- Update Statistics (weekly)
Update Statistics regularly, weekly. This way we will have an optimal execution plan and therefore the best performance

##### 29- Rebuild & Organize indexes (weekly)
Prevent data fragmentations

##### 30- Partition Large Tables (Facts) to improve performance + Columnstore Index
Especially for very large tables, we can partition the table into smaller pieces
Then Apply a Columnstore Index for best results

## AI & SQL
Using copilot can be usefull to auto-complete code
ctrl+tab    -> accepts the suggestion
ctrl+right  -> accept part of the suggestion
ctrl+i      -> request something to be generated

ChatGPT should be used more as brainstorming and when we are stuck completely

### ChatGPT Prompts
MANDATORY:
1- Task *MUST*

IMPORTANT:
2- Context
3- Specifications-Details

NICE TO HAVE:
4- Role
5- Tone

Example
You are a sr SQL expert                     -- role
I am a Data analyst working
on an SQL project using SQL server          -- context
Explain the concept of
SQL Window Functions and do the following   -- task
- Explain each window func and show syntaxt
- Describe why they are important
and when to use them
- List the top 3 use cases                  -- Specifications

The tone shoud be conversational
and direct as if you're speaking
to me one-on-one                            -- tone

#### Prompt 1 - Solve an SQL Task prompt

In my SQL Server database, we have two tables:

The first table is `orders` with the following columns: order_id, sales, customer_id, product_id

The second table is `customers` with the following columns: customer_id, first_name, last_name, country

Do the following:
- Write a query to rank customers based on their sales.
- The result should include the customer's customer_id, full name, country, total sales, and their rank.
- Include comments but avoid commenting on obvious parts.
- Write three different versions of the query to achieve this task.
- Evaluate and explain which version is best in terms of readability and performance.

#### Prompt 2 - Improve readability prompt

The following SQL Server query is long and hard to understand.

Do the following:

- Improve its readability.
- Remove any redundancy in the query and consolidate it.
- Include comments but avoid commenting on obvious parts.
- Explain each improvement to understand the reasoning behind it.

[ SQL QUERY GOES HERE ]
...

#### Prompt 3 - Optimize the performance query

The following SQL Server query is slow.

Do the following:
- Propose optimizations to improve its performance.
- Provide the improved SQL query.
- Explain each improvement to understand the reasoning behind it.

[ SQL QUERY GOES HERE ]
...

#### Prompt 4 - Optimize Execution Plan

The image is the execution plan of SQL Server query

Do the following:
- Describe the execution plan steb by step.
- Identify performance bottlenecks and issues.
- Sugges ways to improve performance and optimize the execution plan.

[ IMAGE GOES HERE ]

[ SQL QUERY GOES HERE ]
...

#### Prompt 5 - Debugging

The following SQL Server Query is causing this error: [ ERROR MESSAGE GOES HERE ]

Do the following:
- Explain the error message.
- Find the root cause of the issue.
- Suggest how to fix it.

[ SQL QUERY GOES HERE ]
...

#### Prompt 6 - Explain the Result

I didn't understand the result of the following SQL Server query.

Do the following:
- Break down how SQL processes the following query step by step.
- Explaining each stage and how the serult is formed.

[ SQL QUERY GOES HERE ]
...

#### Prompt 7 - Styling and Formatting

The following SQL Server query is hard to understand.

Do the following:
- Restyle the code to make it easier to read.
- Align columns aliases.
- Keep it compact - do not introduce unnecessary new lines.
- Ensure the formatting follows best practices.

[ SQL QUERY GOES HERE ]
...

#### Prompt 8 - Documentations & Comments

The following SQL Server query lacks comments and documentation.

Do the following:
- Insert a leading commernt at the start of the query describing its overall purpose.
- Add comments only where clarification is necessary, avoid obvious statements.
- Create a separate document explaining the business rules implemented by the query.
- Create another separate document describing how the query works.

[ SQL QUERY GOES HERE ]
...

#### Prompt 9 - Improve Database DDL

The following SQL Server DDL Script has to be optimized.

Do the following:
- Naming: Check the consistency of table/column names, prefixes, standards.
- Dara Types: Ensure data types are appropriate and optimized.
- Integrity: Verify the integrity of primary keys and foreign keys.
- Indexies: Check that indexes are sufficient and avoid redundancy.
- Normalization: Ensure proper normalization and avoid redundancy.

[ SQL DDL SCRIPT GOES HERE ]
...

#### Prompt 10 - Generate Test Dataset

I need dataset for testing for the following SQL Server DDL

Do the following:
- Generate test dataset as Insert statements.
- Dataset should be realistic.
- Keep the dataset small.
- Ensure all primary/foreign key relationships are valid (use matching IDs).
- Don't introduce any NULL values.

[ SQL DDL SCRIPT GOES HERE ]
...


#### Prompt 11 - Create SQL Course | Studying promt
Create a comprehensive SQL course with a detailed roadmap and agenda.

Do the following
- Start with SQL fundamentals and advance to complex topics.
- Make it beginner-friendly.
- Include topics relevant to data analysts and data scientists.
- Focus on real-world data analytics use cases and scenarios.
[ ADDITIONAL CONTEXT IF NEEDED ]




#### Prompt 12 - Understand SQL Concept | Studying promt
I want detailed explanation about SQL Window Functions.

Do the following:
- Explain what Window Functions are.
- Give an analogy.
- Describe why we need them and when to use them.
- Explain the syntax.
- Provide simple examples.
- List the top 5 use cases.

#### Prompt 13 - Understand SQL Concept| Studying promt
I want to understand the differences between SQL Windows and GROUP BY.

Do the following:
- explain the key differences.
- Describe when to use each, include examples.
- Provide the pros and cons of each approach.
- Summarize the comparison in a clear side-by-side table. 

#### Prompt 14 - Practice SQL | Studying promt

Topic...

Do the folowing:
- Make it interactive practicing, you provide tasks and I provide solutions.
- Provide a sample dataset.
- Give SQL tasks that gradually increase in difficulty.
- Act as an SQL Server and show the resultys of my queries.
- Review my queries, provide feedback and suggest improvements.

#### Prompt 15 - Prepare for a SQL Interview | Studying promt

Act as a SQL interviewer and help me prepare a SQL interview.

Do the following:
- Ask common SQL interview questions.
- Make it interactive practicing, you provide questions and I provide answers.
- Gradually progress to advanced topics.
- Evaluate my answers and give me feedback on how to improve.

## SQL PROJECTS

### Data Warehousing Project
### Exploratory Data Analysis (EDA) Project
### Advanced Data Analytics Project

Goal per day 120 - 180m

Lunes   =>   30m
3:45pm - 4:15         -> 30m
22:45 - 23:04       -> 19m

Miercoles => 95m
7:10 - 7:15           -> 5m
23:04 - 23:11       -> 7m

8:40 - 8:55 -> 15m
23:11 - 23:21 -> 10m

9:38 - 10:30 -> 52m
23:21 - 23:21 -> 0m

11:10 - 11:33 -> 23m
23:21 - 23:21 -> 0m
