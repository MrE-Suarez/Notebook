# INTERMEDIATE


## FIltering data

### 1 Comparison operators 

| Operator | Description | Example | Notes |
|----------|-------------|---------|-------|
| `=` | Equal to | `WHERE age = 30` | Exact match |
| `<>` | Not equal to | `WHERE age <> 30` | Standard SQL; equivalent to `!=` |
| `!=` | Not equal to | `WHERE age != 30` | Supported in most databases; `<>` preferred in standard SQL |
| `<` | Less than | `WHERE score < 100` | Numeric or date comparisons |
| `<=` | Less than or equal | `WHERE score <= 100` |  |
| `>` | Greater than | `WHERE score > 100` |  |
| `>=` | Greater than or equal | `WHERE score >= 100` |  |

### 2 Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `AND` | Combine conditions; both **must** be true | `WHERE age > 18 AND country = 'US'` |
| `OR` | Either condition can be true | `WHERE age < 18 OR status = 'active'` |
| `NOT` | Negates a condition | `WHERE NOT country = 'US'` |

> 🧠 Logical operators are **evaluated left-to-right**, parentheses `()` can clarify order.

### 3️ Range Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `BETWEEN ... AND ...` | Checks if a value is within a range | `WHERE age BETWEEN 18 AND 30` |
| `NOT BETWEEN ... AND ...` | Checks if a value is outside a range | `WHERE score NOT BETWEEN 50 AND 100` |

> ⚡ Inclusive: `BETWEEN` includes both boundary values.
---

### 4️ Membership Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `IN (...)` | Matches any value in a list | `WHERE country IN ('US','UK','CA')` |
| `NOT IN (...)` | Excludes values in a list | `WHERE status NOT IN ('inactive','banned')` |

> 🧠 Faster than multiple `OR` conditions; use for a fixed set of values.

---

### 5️ Pattern Matching / Search

| Operator | Description | Example |
|----------|-------------|---------|
| `LIKE` | Matches a pattern | `WHERE name LIKE 'A%'` → Names starting with A |
| `NOT LIKE` | Does not match a pattern | `WHERE email NOT LIKE '%@gmail.com'` |
| `%` | Wildcard — any number of characters | `LIKE 'Jo%'` → John, Joseph |
| `_` | Wildcard — exactly one character | `LIKE 'J_n'` → Jon, Jan |

> ⚡ Use `ILIKE` in PostgreSQL for case-insensitive matching.

---

### ✅ Tips

- Combine operators for complex filters:
```sql
WHERE age >= 18 AND status = 'active' AND country IN ('US','UK')
```

## Combining Data

### JOIN
Combining tables, add columns from another table.

#### Inner join > Only matching rows
```
sql
select 
	customers.id,
	customers.first_name,
	orders.order_id,
	orders.sales
from customers
inner join orders
on customer_id = id
```
To identify the column: table_name.column_name

Alias for readablity:

select 
	cx.id,
	cx.first_name,
	o.order_id,
	o.sales
from customers AS cx
inner join orders AS o
on o.customer_id = cx.id 

We can create the same inner join effect using left + where

select *
from customers AS cx
left join orders AS o
on cx.id = o.customer_id
WHERE o.customer_id IS NOT NULL;

#### Left join > All rows from left and only matching from right
The left table goes after FROM

```
sql
select 
	cx.id,
	cx.first_name,
	o.order_id,
	o.sales
from customers AS cx
left join orders AS o
on cx.id = o.customer_id
```


#### Right join > All rows from right and only matching from the left
The right table goes after FROM. 
Same as the left join but viceversa.
You would pull all the results in table B and matching results on table A.

Alternative: You can also use a left join and change the positions of the tables on FROM and LEFT JOIN.

#### Full Join > All the rows from both tables

SELECT *
FROM customers
FULL JOIN orders
ON customers.id = orders.customer_id;


#### LEFT ANTI JOIN > Only rows in left table withouth matching rows

```
select 
*
from customers AS cx
right join orders AS o
on cx.id = o.customer_id
where cx.id  IS NULL
```

"where cx.id  IS NULL" Will indicate to remove the matching rows
Right antijoin works the same just including all rows from right and removing the matching with the where usage

#### FULL ANTI JOIN > Returns only rows that don't match in either tables

select 
*
from customers AS cx
full join orders AS o
on cx.id = o.customer_id
where cx.id IS NULL OR o.customer_id IS NULL

#### CROSSING JOIN > Combines every row from left with every row from right, all possible combinations

SELECT *
FROM customers
cross JOIN orders

### How to choose the correct join
Decision tree
DEfine the values you are looking for:

Only matching
    -> Inner join
All rows
    -> Focus on one side
        -> Left join
    -> Focus on both sides 
        -> Full join
Only unmatching
    -> Focus on one side
        -> Left ANTI join
    -> Focus on both sides 
        -> Full ANTI join

SELECT
	o.OrderID,
	(cx.FirstName + ' ' + cx.LastName) as 'Customer name',
	p.Product,
	o.Sales as 'Order sales',
	p.Price as 'Sales Price',
	(e.FirstName+ ' ' + e.LastName) as 'Employee'	
FROM
	Sales.Orders as o
LEFT JOIN Sales.Customers as cx
	on cx.CustomerID  = o.CustomerID
LEFT JOIN Sales.Employees as e
	on e.EmployeeID = o.SalesPersonID
LEFT JOIN Sales.Products as p
	on p.ProductID = o.ProductID;


Requirement: Key column

Usage 
* Recombine Data > Complete a big picture (INNER, LEFT, FULL)
* Data Enrichment > Join the reference table and master (LEFT)
* Check for existence "filtering" > Confirm the existance (INNER, LEFT+WHERE, FULL+WHERE)

/* Starting here, I overwrote an outdated file and I lost the rest of the note until the next checkpoint. Anyway, the topics to discuss after these are that need to be in this note as well:


Set operators

Union
Union all
Except
Intersect

Rules of set operators
1- order by can be used only once
2- same number of columns
3- matching data types
4- same order of columns
5- first query controls aliases 
6- mapping correct columns

Adding rows is with SET operators.

There was an example of using one of these as a data engineer to compare the source and the created data warehouse. And other examples as well 
The usages

## row level functions

String funcs
Divided by:
	manipulation
		concat
		upper
		lower
		trim
		replace
	calculation
		len
	extract
		left
		right
		substring

Number funcs
	round
	abs
DAte and time funcs
	explaining the data type, date and time, only time, only date
	Part extraction
		day()
		month()
		year()
		datepart()
		datename()
		datetrunc()
		eomonth()
		summary of which func to use like....
			which part is needed?
			Day? Month?
				numeric? -> day(), month()
				full name -> datename
			year? -> year()
			other parts -> datepart()
	format and casting
		format
			casting anytype only to string / formats date-time and numbers
		convert
			casting anytype to anytype / formats only date-time
		cast
			casting any type to anytype / no formatting
		differences, usages, best practices, etc. Also the note of what data type returns each
	calculations
		dateadd()
		datediff()
	validation
		isdate() -> There was an example where they explained how this is good for data clean up
Null funcs -> Found below, I stayed here
Case statement -> Same, It's below since I was able to enter this note

/*All this was missing in my note when overwriting
Create this section and merge this with the following since I stayed in the very middle of it
Being said this, I will close the comment */


### NULL functions
Replace a NULL to a value
ISNULL
	ISNULL(value, replacement_value)
	Examples:
		ISNULL(shipping_Address, 'unknown')
		ISNULL(shipping_Address, billing_Address) --To take the value from shipping
COALESCE -> Return the first non-NULL value from a list
	COALESCE(value1, value2, value3, ...)
	Example:
		COALESCE(shipping_Address, billing_Address, 'N/A') -- To return N/A in case billing_Address is NULL
Comparison
ISNULL,	COALESCE
limited values, unlimited
fast, slow
different func along data bases, same in all data bases

Easier to stick to COALESCE to stick to the standard

Usage example:
1. If trying to retrieve an AVG, NULLs will not be considered. Therefore we can first clean the NULLs
AVG(COALESCE(Score,0)) OVER() AvgScore

2. Mathematical operations or String aggregations won't work. If there are NULLs, the result will be NULL
FIRSTnAME + ' ' + COALESCE(LastName, '') AS FullName

3. Joining tables when keys there are NULL means missing information
...
FROM table1 a
Join Table2 b
ON a.year= b.year 
AND ISNULL (a.type, '') = ISNULL(b.type, '')

Therefore the NULLS will be replaced to empty strings in any missing key

NULLIF 
Compares two values
if equal, returns NULL
else, returns first value
	NULLIF(value1, value2)

case: dividing by 0
...
Sales / NULLIF(Quantity,0) AS Price


IS NULL -> Returns TRUE if the value is NULL
IS NOT NULL -> Returns TRUE

Usage:
1. Searching for NULLs
...
WHERE Score IS NULL

2. Anti Joins
Left Join + IS NULL
Right Join + IS NULL
customers who have not place an order:
SELECT
c.*,
o.OrderID
FROM Sales.Customers c
LEFT JOIN Sales.Orders o
ON c.CustomerID = o.CustomerID
WHERE o.CustomerID IS NULL

Standards, Data policies
How data should be handled
Option 1: Only use NULLs and empty strings, avoid blank spaces
	TRIM(column_Name) Policy1
	This returns trimmed spaces data

Option 2: Only use NULLS and avoid empty strings and blank spaces
	NULLIF(TRIM(column_Name), '') Policy 2
	After trimmed to blank, if equal, it will make these NULL

Option 3: Use a default value 'unknown', avoid using NULLs, empty strings and blank spaces
	COALESCE(column_Name, 'unknown') --If NULL replace to unknown
	COALESCE(NULLIF(TRIM(column_Name), ''), 'unknown') -- This covers the 3 scenarios

Recommendation: Option 2 takes less space, Option 3 is easy to understand. Avoid option 1
Option 2 for cleaning, Option 3 prior to report

Summary of ussage
Handle NULLS for...
* data aggregation
* mathematical operations
* joining tables
* sorting data
* findig unmatvhed data - left anti join

### Case Statement

Evaluates a list of conditions and returns a value when the first condition is met

Case
	WHEN condition1 THEN result1
	WHEN condition2 THEN result2
	...
	ELSE result_if_conditions_were_falsa
END

Usage example:
Categorazing Data
Mapping values
Handling nulls
conditional aggregations,
some examples below:

Step 1- Label the data
SELECT
    OrderID,
    Sales,
    CASE 
      WHEN Sales > 50 THEN 'High'
      WHEN Sales > 20 THEN 'Medium'
      ELSE 'Low'
    END AS Category
  FROM Sales.Orders

Step 2- Categorize the data
SELECT
  Category,
  SUM(Sales) AS TotalSales
FROM (
  SELECT
    OrderID,
    Sales,
    CASE 
      WHEN Sales > 50 THEN 'High'
      WHEN Sales > 20 THEN 'Medium'
      ELSE 'Low'
    END AS Category
  FROM Sales.Orders
) AS t
GROUP BY Category;

Case forms full vs quick
Full form:
CASE 
	WHEN Country = 'Germany' THEN 'DE'
	WHEN Country = 'India' THEN 'IN'
	WHEN Country = 'Mexico' THEN 'US'
	WHEN Country = 'France' THEN 'FR'
	WHEN Country = 'Italy' THEN 'IT'
ELSE 'n/a'
END

Quick form:
CASE Country
	WHEN = 'Germany' THEN 'DE'
	WHEN = 'India' THEN 'IN'
	WHEN = 'Mexico' THEN 'US'
	WHEN = 'France' THEN 'FR'
	WHEN = 'Italy' THEN 'IT'
ELSE 'n/a'

Managing NULLs
SELECT
CustomerID,
LastName,
Score,
CASE
	WHEN Score IS NULL THEN 0
	ELSE Score
END ScoreCleaned,

AVG(CASE
		WHEN Score IS NULL THEN 0
		ELSE Score
	END) OVER() AVGscoreCleaned, 

AVG(Score) OVER() AvgCustomer
FROM Sales.Customers

### Agregation & Analytical Functions

#### Aggregate functions
These accept multiple rows and meant to reaturn usually a single value

Count()
Counting rows

SUM()
Find the total summarizing

AVG()
Summirize the values and divide it by the number of values, NULL WILL NOT BE CONSIDERED

MAX()
Search for the highest value

MIN()
Search for the lowest value

Examples:
-- Find total of orders
SELECT
COUNT(*) AS total_n_orders
FROM orders

--Include the total sales of all orders
SELECT
COUNT(*) AS total_n_orders,
SUM(sales) AS total_sales
FROM orders

...

If we add GROUP BY, we would break the big numbers by customer, date, country.
Providing more details based on the criteria it's grouped by:
SELECT
customer_ID,
COUNT(*) AS total_n_orders
FROM orders
GROUP BY customer_id

#### Window basics
Also analitical functions... Usefull for performinc calculations on subsets of data providing more details.


##### WINDOW vs GROUP BY
Window functions can perform calculations (aggregations) without losing the level of details of rows

Group By simply summarizes, squizes the results
	-> Returns a single row for each group
	-> Simple aggregations
	-> Only aggregation functions
	-> Simple analysis
	-> Exapmples
		Total sales
		Total sales for each product (adding GROUP BY)
		Providing DETAILS IS NOT viable: Total sales for each product + the date -- This doesn't match, it woldn't return the total sales on each product
Window doesn't summarize the whole row, it returns all the rows with the criteria you chose summarized 
	-> Returnd a result for each row
	-> Aggregation + Keep details
	-> Aggregations + Rank + Value functions
	-> Complex analysis
	-> Examples
		Providing DETAILS:Total sales for each product + the date. Example:

##### Window Syntax

1. Window Func
2. Parrtition Clause + Order clause + Frame Clause

1. 
###### Window func
	One of the following:
	Aggregate	-> Count, Sum, Avg...
	Rank		-> Row number, Rank, Dense Rank...
	Value		-> Lead, Lag, First Value... 

Then window expression
	Arguments you pass to the function
	This can be emptym column, number, multiple arguments or a conditional logic
	Depending on the function:
		Aggregate:	Mostly numeric, and all data type
		Rank: 		Mostly empty, and numeric
		Value:		All any data type

2. 
###### Parrtition Clause + Order clause + Frame Clause
OVER()
	-> Start the window. THIS IS A *MUST*

###### PARTITION BY - Can be skipped
	-> If skipped, It takes the whole data on the entire data
	-> When used, The calculation is based on the selected partition/group 
		Example: 
		-- Find the total sales across all orders
		SELECT
			OrderID,
			OrderDate,
			ProductID,
			SUM(Sales) OVER(PARTITION BY ProductID) TotalSalesByProducts-- PARTITION BY is similar to GROUP BY
		FROM Sales.Orders 	
		-- This shows all the rows, DIFFERENT orderID, OrderDate, ProductID, SAME TotalSalesByProduct based on SALES-PRODUCTID

####### ORDER BY - Optional for aggregate | MUST for rank | MUST for value
	-> Just as an ORDER BY but for the window section
	-> This can be skipped
	-> It's a musto for some funcs
		EXAMPLE:
		-- Rank each order based on their sales from lowest to highest
		SELECT
			OrderID,
			OrderDate,
			Sales,
			RANK() OVER( ORDER BY Sales ASC) RankSales
			-- This returns all orders but ranked + ordered

####### FRAME
Define a subset of rows within each window
Example query:
	AVG (Ssales) OVER (PARTITION BY Category ORDER BY OrderDate ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING)

Explanation:
	Frame Types
		ROWS
		RANGE
	Frame Boundary (lower value)
		CURRENT ROW
		N PRECEDING
		UNBOUNDED PRECEDING
	Frame Boundary (Higher value)
	
Rules
	- Frame clause MUST be used along with ORDER BY 
	- Lower value MUST be BEFORE the higher value

Example:
SELECT
	OrderID,
	OrderDate,
	OrderStatus,
	Sales,
	SUM (Sales) Over (PARTITION BY OrderStatus ORDER BY OrderDate
	ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING) TotalSales
FROM Sales.Orders

/* I did not understand this note in here...
Compact Frame 
Only with PRECEDING, *CURRENT ROW* can be *SKIPPED*
Normal	-> ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
Short	-> ROWS 2 PRECEEDING
...
SUM (Sales) Over (PARTITION BY OrderStatus ORDER BY OrderDate
	ROWS 2 PRECEDING) TotalSales */ 

Default Frame when using ORDER BY in a WINDOW
ROW BETWEEN UNBOUNDED PRECEDING AND CURRENT RORE
It will be stacking the result

###### Window Rules / Limitations
1. Used only in the SELECT and ORDER BY clauses - WINDOW are not for filtering data
2. Nesting WINDOW functions is NOT ALLOWED
3. SQL executes WINDOW funcs after WHERE - First filters then calculates
4. WINDOW Function can be used with GROUP BY ONLY if the same COLUMNS are USED 
	Example:

--Rank Customers based on their total sales

Only GROUPING BY won't show the actual position
SELECT
	o.CustomerID,
	c.FirstName,
	SUM (Sales) TotalSales --Group column
FROM Sales.Orders as o
LEFT JOIN sales.Customers as c
	ON o.CustomerID = c.CustomerID
GROUP BY o.CustomerID, c.FirstName
ORDER BY TotalSales DESC


This fulfills the task succesfully:
SELECT
	RANK() OVER(ORDER BY SUM (Sales)DESC) AS Rank_Position, --WINDOW
	o.CustomerID,
	c.FirstName,
	SUM (Sales) TotalSales -- GROUP BY: Anything that is inside the window, should be part of the group by as well
FROM Sales.Orders as o
LEFT JOIN sales.Customers as c
	ON o.CustomerID = c.CustomerID
GROUP BY o.CustomerID, c.FirstName

#### Window Aggregate func

##### COUNT
Returns the number of rows within a window

COUNT(*)		Returns ALL ROWS, even NULLs -- Also COUNT(1)
COUNT(Sales)	Returns ALL ROWS with no NULLs at Sales (or any specific column)

Usages
1. Overall Analysis
2. Category Analysis
3. Quality Checks: Identify NULLs
4. Cuality Checks: Identify Duplicates

Example:
Check for duplicates using a window as a subquery
SELECT *
FROM(
	SELECT
		OrderID,
		COUNT(*) OVER (PARTITION BY OrderID) CheckPK --Primary Key
	FROM Sales.Orders
)t WHERE CheckPK > 1

This would show only resuts where PK > 1

##### SUM
Returns the sum of values within a window
This allows only numbers

Usages
Find the totals by a column: Part-to-Whole Analysis
Example:
	SELECT
		OrderID,
		ProductID,
		Sales,
		SUM(sales) OVER() TotalSales,
		Round(CAST(sales as FLOAT)*100/ SUM(sales) OVER(),2) Percenta
	FROM sales.Orders
	ORDER BY sales ASC

##### AVG
Returns the AVG of values within a window

Example part of a total:
-- Average sales across all orders + AVG for each product
-- include orderid and date

SELECT
	OrderID,
	ProductID,
	OrderDate,
	Sales,
	AVG(sales) OVER() TotalAVG,
	AVG(sales) OVER(PARTITION BY ProductID) ProductAVG
FROM sales.Orders

Example handling nulls:
SELECT
	CustomerID,
	FirstName + ' ' + COALESCE(LastName, '') FullName,
	COALESCE(score, 0) individualScore,
	AVG(COALESCE(score, 0)) OVER() AVGTotalScore
FROM sales.Customers

Example subquery for filtering:
SELECT
*
FROM
(
	SELECT
		OrderID,
		ProductID,
		Sales,
		OrderDate,
		--COUNT(OrderID) OVER(OrderID)TotalOrders,
		AVG(sales) OVER () AVGsales
	FROM sales.orders
	)t where sales> AVGsales 

##### MIN and MAX
MIN() returns the lowest value within a window
MAX() returns the highest value within a window



Running and Rolling Total
They aggregate squence of members and aggregation is updated each time a new member is added

Tracking
	-> Track current sales with Target Sales | Aggregates all values from the beggining up to the ccurrent point without dropping off older data.
Trend Analysis
	-> Provide insights into historical patterns | Aggregates all values within a fixed Window (e.g. 30 days), as new data is added, the oldest data point will be dropped

Example with moving AVG:
SELECT
	OrderId,
	ProductID,
	OrderDate,
	Sales,
	AVG(Sales) OVER(PARTITION BY ProductID) AVGbyProduct,
	AVG(Sales) OVER(PARTITION BY ProductID ORDER BY OrderDate) MovingAvg, --Running Avg, unbounded by productID
	AVG(Sales) OVER(PARTITION BY ProductID ORDER BY OrderDate ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING) MovingAVG_NextOrder --Including only the next order
FROM Sales.Orders

/*I did not totally understood the usage if these two, and the example, you can explay practical scenarios. It was first explained using SUM() then AVG(). We can do running avg as well and rolling avg*/

Overall Total: Overview of entire data
SUM(Sales) OVER()

Total Per Groups: Compare Categories
SUM(Sales) OVER(PARTITION BY Product)

Running Total: Progress Over Time
SUM(Sales) OVER(ORDER BY OrderDate)

Rolling Total: Progress Over Time in Specific Fixed Window
SUM(Sales) OVER(order by OrderDate ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING)

== Summary
Rules
- Use a number
- Count accepts any data type

Cases
- Overall Analysis
- Total per groups analysis
- Part-to-whole analysis
- comparison analysis (avg or extreme highest/lowest)
- Identify duplicates
- Outlier detection
- Runnning total
- Rolling total
- Moving average

#### Window Ranking func

There are 2 ranking bases:
1. Integer-based Ranking	-> Discrete Values	
2. PErcentage-based Ranking	-> Continuous Values

1 is best Top/Bottom N Analysis "Find the top 3 Products" 
	ROW_NUMBER()
	RANK()
	DENSE_RANK()
	NTILE()

2 is best for Distribuition analysis "Find the top 20% products"
	CUME_DIST()
	PERCENT_RANK()	

##### Syntax
	ROW_NUMBER()	-> Expression optional | PARTITION BY optional | ORDER BY Required | FRAME not allowed
	RANK()			-> Expression optional | PARTITION BY optional | ORDER BY Required | FRAME not allowed
	DENSE_RANK()	-> Expression optional | PARTITION BY optional | ORDER BY Required | FRAME not allowed
	NTILE()		-> Expression NUMBER   | PARTITION BY optional | ORDER BY Required | FRAME not allowed
	CUME_DIST()		-> Expression optional | PARTITION BY optional | ORDER BY Required | FRAME not allowed
	PERCENT_RANK()	-> Expression optional | PARTITION BY optional | ORDER BY Required | FRAME not allowed



##### Inter-Based Ranking Functions
ROW_NUMBER()
Assign a unique number to each row, NO TIES
NO GAPS

Case:
Top-N Analysis -> Find the top highest

SELECT *
FROM (
SELECT 
	o.OrderID,
	p.Product,
	o.Sales,
	ROW_NUMBER() OVER(PARTITION BY o.ProductID ORDER BY Sales DESC) TOPSALES
FROM sales.Orders o
LEFT JOIN sales.Products p
	ON o.ProductID = p.ProductID
	)t WHERE TOPSALES = 1

Bottom-N Analysis -> Find the top lowest
This can be done a similar way

Assign Unique IDs
SELECT 
	ROW_NUMBER() OVER(ORDER BY OrderID) newID,
	*
FROM sales.OrdersArchive

Identify Duplicates and return cleaned data
SELECT *
FROM(
SELECT
	ROW_NUMBER() OVER(PARTITION BY OrderID ORDER BY CreationTime DESC) pre_clean,
	*
FROM sales.OrdersArchive
)t WHERE pre_clean = 1

The subquery counts the time an OrderID is repeated, and orders this from new to old on Creation time
Meaning the 1 would be the most updated data
Then WHERE would return the most updated data only
We could DROP the >1 Results

RANK()
Assign a rank to each rows, TIES are ACCEPTED and will repeat the same number
There can be gaps

DENSE_RANK()
Assign a rank to each row, TIES are ACCEPTED 
NO GAPS

NTILE()
Divides the rows into specified number of apprximately equal groups (Buckets)
Enter the number of buckets you'd lite to divide it as argument

Use cases

Data Analyst	-> Data segmentation
	Divide a dataset into distinct subsets based on certain criteria

	Example:
		SELECT
		OrderId,
		ProductID,
		sales,
		CASE TOPsales
		WHEN 1 THEN 'high'
		WHEN 2 THEN 'medium'
		WHEN 3 THEN 'low'
		END AS salesCategory
	FROM(
	SELECT 
		OrderId,
		ProductID,
		SalesPersonID,
		sales,
		NTILE(3) OVER (ORDER BY sales DESC) TOPsales
	FROM SALES.Orders
	)t

Data Engineer	-> Equalizing load processing
	Equalizing load
		When migrating one database to another, it may be better to divide the table using NTILE() in order to work to do it on parts

##### Percentage-Based Ranking

CUME_DIST()
Comulative distribution		-> Calculates the distribution of data points within a window
	Position Number / Number or Rows

	CUME_DIST() OVER (ORDER BY Sales DESC)
		Sales	|	DIST
		100		|	0.2			-> 1/5
		80		|	0.6			-> 3/5 -- If there's a tie, SQL will use the last row position in a tie  
		80		|	0.6			-> 3/5 
		50		|	0.8			-> 4/5
		30		|	1			-> 5/5

PERCENT_RANK()
Calculates the relative position of each Row
	Position Number - 1 / Number of Rows -1

	PERCENT_RANK() OVER (ORDER BY Sales DESC)
		Sales	|	DIST
		100		|	0			-> 1-1/5-1	-> 0/4
		80		|	0.25		-> 2-1/5-1	-> 1/4 
		80		|	0.25		-> 1/4 		-- When ties, SQL will take the first occurrance as the position number
		50		|	0.75		-> 3/4
		30		|	1			-> 5/5

#### Window Value Func
Acces a value from another row

##### Syntax

FUNC			Expression		PARTITION	ORDER BY	FRAME
LEAD()			all data type	optional	must		not allowed
LAG()			all data type	optional	must		not allowed
FIRST_VALUE()	all data type	optional	must		optional
LAST_VALUE()	all data type	optional	must		should be used

##### LEAD() and LAG()
LEAD
	Let's you access a value from the next row within a window

	LEAD(Sales, 2, 10) OVER (PARTITION BY  ProductID ORDER BY OrderDate)
	-- 2 specifies the number of tows backward. If empty the default is 1
	-- 10 is the defaultvalue. If empty the default is NULL

LAG
	Let's you access a value from the prev row within a window
	LAG(Sales, 2, 10) OVER (PARTITION BY  ProductID ORDER BY OrderDate)

Time Series Analysis
	Analazing the data to undertand patterns trnds and behaviours over the time
	Year-over-Year(YoY)		-> Analyze the overall growth or decline of the business' performance over time
	Month-over-Month(MoM)	-> Analyze short-term trends and discover patterns in seasonality

	Example:
		--Analyze month ove rmonth performance 
		-- find the percentage change in sale between the current and previous months

		SELECT
		*,
		CurrentMonth - PreviousMonthSales as Month_Change,

		CAST((CurrentMonth - PreviousMonthSales) AS FLOAT) / PreviousMonthSales *100 as MONTH_cahngePercentage
		FROM (
		SELECT 
			Month(OrderDate) OrderMonth,
			SUM(Sales) CurrentMonth,
			LAG(SUM(Sales)) OVER(ORDER BY MONTH(OrderDate)) PreviousMonthSales
		FROM Sales.Orders
		GROUP BY MONTH(OrderDate)
		)t 

Customer retention analysis
Measuring customer's behaviour and loyalty

-- Rank Customers based on the average days between their orders
-- Rank Customers based on the average days between their orders

SELECT
	CustomerID,
	AVG(daysDiff) AvgDifference, -- 3. AVG of days between orders
	RANK() OVER(ORDER BY COALESCE(AVG(daysDiff), 9999)) AvgRank --4. Rank them + handle nulls
FROM(
	SELECT
		OrderID,
		CustomerID,
		OrderDate CurrentOrder,
		LEAD (OrderDate) OVER(PARTITION BY CustomerID ORDER BY OrderDate) NextOrder, --1. Get next order in the same row 
		DATEDIFF(day,OrderDate,LEAD (OrderDate) OVER(PARTITION BY CustomerID ORDER BY OrderDate)) daysDiff --2. Get days between customers orders
	FROM sales.Orders
	)t 
	GROUP BY CustomerID


### FIRST_VALUE() and LAST_VALUE()

FIRST_VALUE()	-> default works good
Access a vlaue form the first row within a window

LAST_VALUE()	-> W e should include "ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
Access a vlaue form the last row within a window


/* More context for this note:
I had seen all of the previous. Date and time funcs seemed interesting, i hadn't used these much before... but when starting NULLs it all came new for me. Nulls were understandable as having programming foundations. Similarly with CASE statements. But then WINDOW Functions was totally new and unknown for me, it took me longer taking notes there... */

/*Note 2: This should be only helpful notes of my progress, howeve I will admit that sometimes I entered queries I did for practice and some may not be relevant and checked again*/


