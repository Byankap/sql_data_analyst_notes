# SQL Server Syntax

Created by Microsoft

ALTER TABLE

```sql
--used for renaming columns
ALTER TABLE table_name
RENAME COLUMN old_name TO new_name

--Droping a column
ALTER TABLE table_name
DROP COLUMN column_name

--charging data types
ALTER TABLE table_name
ALTER COLUMN column_name
	type VARCHAR(128)
```

CASE WHEN

```sql
CASE 
	WHEN boolean_expression THEN result_expression[...n]
	[ELSE else_result_expression]
END

	CASE WHEN continent = 'Europe' or Continent = 'Asia' THEN 'Eurasia'
		ElSE Continuent
		END AS NewContinent --column name

SELECT 
    SUM(CASE WHEN name = 'unnamed' THEN 1 else 0 END) AS NAME_COUNT,
```

Binning into groups

```sql
CASE WHEN x <30 THEN 1
		 WHEN x >30 AND x< 40 THEN 2
			.......
			ELSE 5
			END AS column_name

```

CAST

Convert a value to an int datatype

```sql
-- Calculate the net amount as amount + fee
SELECT transaction_date, amount + CAST(fee AS integer) AS net_amount 
FROM transactions;
```

CREATE TABLE

unique table name

(column name, data type, size)

```sql
CREATE TABLE test_table(
	Test_date date,
	test_name varchar (20),
	test_int int);
```

Data types

Dates

date (yyyy-mm-dd), datetime (yyyy-mm-dd hh:mm:ss)

time

Numeric

integer, decimal, float

bit(1=TRUE, 0=FALSE, also NULL values)

Strings

char: a fixed length string of n characters

varchar: a max number of characters

nvarchar

COALESCE

returns the first non-missing value

```sql
SELECT column1, column2
COALESCE(column1, column2, 'NA') AS new_column
FROM ....
--if value1 is null and value2 is not null, return value2
--if both, returns NA
--if both have values, it returns the first value 

```

Common Table Expressions

similar as Derived tables  but one can use multiple times in a query

```sql
--start with WITH and column names
WITH CTEName(Col1, Col2)
AS
--Define the CTE query using the two columns from above
(SELECT Col1, Col2
FROM TableName)

--example
--create a CTE to get Max BloodPressure by age
WITH BloodPressureAge(Age, MaxBloodPressure)
AS
(SELECT Age, MAX(BloodPressure) AS MaxBloodPressure 
FROM Kidney
GROUP BY Age)
--create query to use CTE
SELECT a.Age, MIN(a.BloodPressure), b.MAXBloodPressure 
FROM Kidney a
--Join CTE with Table
JOIN BloodPressureAge b 
ON a.Age=b.Age
GROUP BY a.Age, b.MaxBloodPressure   
```

CONCAT

glues together two columns

```sql
-creating surrogate and primary keys
ALTER TABLE 
ADD COlUMN column_c VARCHAR (250)

UPDATE TABLE
SET column_c = CONCAT(column_a, column_b) --Surrogate key
ALTER TABLE table_name
ADD CONSTRAINT pk PRIMARY KEY (column_name)

```

Date Functions

```sql
SELECT
--returns dates as a datetime datatype local or UTC
	GETDATE()
	GETTUTCDATE()
--returns the CURRENT TIME 
	SYSDATETIME()
	SYSUTCDATETIME()
```

Breaking down a date into parts

```sql
DECLARE
	@SomeDate DATETIME2(3) = '2019-03-01...'
SELECT YEAR(@SomeDate)
SELECT MONTH()
SELECT DAY()

--parsing dates with date parts 
DATEPART()
--returns the numeric value of the part wanted
SELECT DATEPART(YEAR,@dt) AS theYear;
DATENAME()
--returns a string value
SELECT DATENAME(MONTH,@dt) AS theMonth;
```

DATEPART

determine what part of date you want to calculate

DD: day

MM: month

YY: year

HH: hour

```sql
DATEADD()
--add or subtract datetime values, always returns a date
DATEADD(DATEPART, number, date)
--date thats 30 days from 6/21/2020
SELECT DATEADD(DD, 30, '2020-06-21')
--use negative numbers to back to the past
--Can also use columns that have dates
	-- Return the DeliveryDate as 5 days after the ShipDate
	SELECT OrderDate, 
       DATEADD (DD, 5, ShipDate) AS DeliveryDate
	FROM Shipments

--Can subtract 4days and 3 hours
SELECT 
	DATEADD(HOUR, -3, DATEADD(DAY, -4, @SomeTime)) AS ...

DATEDIFF()
--obtain the difference bwt two datetime values, always returns a number, and rounds up
DATEDIFF(datepart, startdate, endate
SELECT DATEDIFF(DD, '2020-05-22', '2020-06-21')
--output will. be 30
```

SQL Server does not have an intuitive way to round down to the month, hour, or minute. You can, however, combine the `DATEADD()` and `DATEDIFF()` functions to perform this rounding.

To round the date 1914-08-16 down to the year, we would call `DATEADD(YEAR, DATEDIFF(YEAR, 0, '1914-08-16'), 0)`. To round that date down to the month, we would call `DATEADD(MONTH, DATEDIFF(MONTH, 0, '1914-08-16'), 0)`

Formatting Functions

CAST()

![Screen Shot 2022-04-15 at 12.59.06 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ee96f08c-ea9c-4fe7-894d-6cea6b6960d4/Screen_Shot_2022-04-15_at_12.59.06_PM.png)

CONVERT()

like CAST but there is more control over formatting from dates to strings with using an optional style(its the third parameter)

![Screen Shot 2022-04-15 at 1.04.28 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/be2a5363-22bf-4617-b439-095b2e497212/Screen_Shot_2022-04-15_at_1.04.28_PM.png)

FORMAT()

more flexible then the two above but slower (around 50,000 rows)

![Screen Shot 2022-04-15 at 1.07.45 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1727d5ff-fa62-486b-a1fd-076105208ea0/Screen_Shot_2022-04-15_at_1.07.45_PM.png)

DECLARE

creating variable to avoid repeatability 

![Screen Shot 2022-03-29 at 12.51.17 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/30468c39-401e-4fc3-a464-d8830ca9222d/Screen_Shot_2022-03-29_at_12.51.17_PM.png)

```sql
DECLARE @

--integer
DECLARE @test_int INT

--Varchar
DECLARE @my_artist VARCHAR(100)

--after the declare has been put, next step is SET to assigning the variable 
DECLARE @test_int INT
SET @test_int = 5
```

![Screen Shot 2022-03-29 at 1.08.54 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/34aff8ae-68df-4477-a466-e26bfb543ea6/Screen_Shot_2022-03-29_at_1.08.54_PM.png)

DELETE

tip: there will not be a confirmation, it will happen immediately 

```sql
--deletes specific items
DELETE
FROM table
WHERE 
	--conditions

--Clearing entire table as once
TRUNCATE TABLE
```

Derived Tables

breaking down complex query into smaller steps

queries treated like a temp table 

contained within main query

specified in the FROM clause

```sql
SELECT a.* 
FROM kidney a
--dervied table computes average age joined to the actual table
JOIN(SELECT AVG(Age) AS AverageAge
FROM kidney) b
ON a.Age = b.AverageAge
```

FROM

the source of the location of tables

GROUP BY

occurs after WHERE, but a WHERE clause in not necessary

CANNOT use WHERE with GROUP BY

split data into groups/comb of one or more values/columns 

avoids having to individual queries for filtering in WHERE, instead it will be a sum of all unique values for specific columns at once. 

```sql
--calculate the maximum value for each state
SELECT State, MAX(DurationSeconds)
FROM Incidents
GROUP BY State
```

HAVING

appear after the GROUP BY clause and filters on groups or aggregates

INSERT INTO

when adding data in the order of the column

```sql
INSERT INTO table_name

--adding columns and values (specify the destination of columns and values
INSERT INTO table_name (col1, col2, col3)
VALUES
	('value1', 'value2')

--INSERT SELECT
INSERT INTO table_name
SELECT
	col1, 
	col2, 
	col3
FROM...
```

tip: dont use SELECT *, be specific  in case table structure changes 

ISNULL

substituting missing data with specific values- can replace with only one value 

```sql
SELECT column1, column2
ISNULL(column2, "Unknown") AS NewCountry
FROM...

--can substitute values from one column for another 
```

JOIN

primary keys: uniquely identify each row

Foreign keys: key found in other tables

```sql
--template
SELECT table_a.column1, table_a.column2, table_b.column1, ... 
FROM table_a
JOIN table_b 
ON table_a.column = table_b.columng
WHERE ...
```

INNER JOIN- only returns matching rows

```sql
--example
SELECT
	table_A.columnX,
	table_A.columnY,
	table_B.columnZ
FROM Table_A
INNER JOIN table_B ON TableA.foreign_key = table_B.primary_key;

SELECT
	album_id,
	title,
	album.artist_id,
	name
FROM album
INNER JOIN artist on artist.artist_id = album.artist_id;

--multiple joins
SELECT
	table_A.columnX,
	table_A.columnY,
	table_B.columnZ, table_C columnD
FROM Table_A
INNER JOIN table_B ON Table_B.foreign_key = table_A.primary_key;
INNER JOIN table_C ON Table_C.foreign_key = table_B.primary_key;
```

LEFT & RIGHT

-All rows from the main table plus matches from the joining table. if a data point is missing from the main tables and merging table a NULL

LEFT JOIN

```sql
SELECT
	Admitted.Patient_ID,
	Admitted,
	Discharged
FROM Admitted
LEFT JOIN Discharged ON Discharged.Patient_ID = Admitted.Patient_ID;
```

![Screen Shot 2022-03-28 at 8.36.15 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5959b80b-0485-4498-ab98-5d644cff793c/Screen_Shot_2022-03-28_at_8.36.15_PM.png)

RIGHT JOIN

![Screen Shot 2022-03-28 at 8.42.21 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/391b83c9-95f0-49b7-a350-4f89ef440cd7/Screen_Shot_2022-03-28_at_8.42.21_PM.png)

NOT NULL

prevents from obtaining null values

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/13ef8933-3a96-43a9-b8eb-15e6fe083f13/Untitled.png)

BlANK VALUES

are different from null values to exclude them 

```sql
WHERE len(column_name) > 0
```

ORDER BY

```sql
-- Order in ascending order (the default)
ORDER BY column

--Order in descending 
--ASC is smallest to biggest (default)
--DESC big to small 
SELECT TOP (10) column1, column2
FROM table
ORDER By column2 DESC, column1;
```

ROUND

either side of the decimal 

ROUND(number, length [,function])

```sql

--right side of the decimal
SELECT DurationSeconds,
ROUND(DurationSeconds, 0) AS RoundToZero,
ROUND(DurationSeconds, 1) AS RoundToOne
FROM....
----output
DurationSeconds=121.6480
RoundToZero=122.0000
RoundToOne=121.6000

--rounding on the left side
SELECT DurationSeconds,
ROUND(DurationSeconds, -1) AS RoundToTen,
ROUND(DurationSeconds, -2) AS RoundToHundred
FROM....
----output
DurationSeconds=121.6480
RoundToTen=120.0000
RoundToHundred=100.0000
```

TRUNCATE

ROUND use truncate values when specifying third argument

truncate      vs      round

17.85→ 17           17.85→ 18

```sql
SELECT DurationSeconds,
ROUND(DurationSeconds, 0) AS RoundToWhole,
ROUND(DurationSeconds, 0, 1) AS Truncating
FROM....
----output
DurationSeconds=15.6100
RoundToWhole=16.000
Truncating=15.000

```

MORE MATH FUNCTIONS

ABS():  absolute, returns positive numeric values 

SQRT(): finding the square root

LOG(): returns the natural logarithm

```sql
LOG(number [,Base])
```

SELECT

key term for retrieving data

use a comma after every column and a  ;  at the end of query

### commands

```sql
--Return 5 rows
SELECT TOP (5) column

--Return top 5% of rows
SELECT TOP (5) PERCENT column

--Return unique rows (if there are mulitple of one, it will only return one)
SELECT DISTINCT column 

--Return all rows (not suitable for large tables)
SELECT *
SELECT TOP (5) PERCENT *

--Aliasing column names with AS
SELECT column(old) AS column(new_name)

```

### Aggregating Data

```sql
--calculate total amount
SELECT
	SUM(column) AS new_name (without an alias with will be a no column name)

--calucate the frequecny of
SELECT
	COUNT(column) AS new_name (without an alias with will be a no column name)
SELECT
	COUNT(DISTINCT column) AS new_name (without an alias with will be a no column name)

-- Count the number of distinct rows with columns make, model
SELECT COUNT(DISTINCT(make, model)) 
FROM cars;

--min and max (column value)
SELECT
	MIN(column) AS new_name (without an alias with will be a no column name)

--average value
SELECT
	AVG(column) AS new_name (without an alias with will be a no column name)

```

### Strings

```sql
--length of characters
SELECT 
	column1,
	LEN(column1) AS new_name

--how to select x number of characters from the left of the string
SELECT 
	column1,
	LEFT(column1, #) AS new_name
--or
	RIGHT(column1, #) AS new_name (works backwards)
--Selecting characters from the middle of the text
SELECT 
	SUBSTRING(column name, 12(#of character to start from), 12(#of character to extract) AS new_name

--find a specific character within a string
SELECT
	CHARINDEX('What want to find', column name) AS new_name

--replacing characters
SELECT 
	TOP(5) REPLACE(column name, 'character', 'replaced character') AS new_name
```

SUBSTRING

The SUBSTRING() function extracts some characters from a string.

```sql
--Extract 3 characters from a string, starting in position 1:

SELECT SUBSTRING('SQL Tutorial', 1, 3) AS ExtractString;

--another example
ALTER TABLE professors 
ALTER COLUMN firstname 
TYPE varchar(16)
USING SUBSTRING (firstname FROM 1 for 16)
```

Temporary Tables

using a # to create a temp table 

![Screen Shot 2022-03-29 at 1.18.50 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1bf5af8c-0f6a-46f7-9766-043df6ff7b2e/Screen_Shot_2022-03-29_at_1.18.50_PM.png)

WHERE

return rows that met a certain criteria

can filter ‘text’, numbers, ‘dates’ (year-month-day)

```sql
--first three customers with invoice of >15
SELECT column1, column2
FROM table
WHERE column2 > 15;

--non-equality
WHERE total <> 10;

--return values in a range
WHERE total BETWEEN 20 AND 30;
--or
WHERE total NOT BETWEEN 20 AND 30;

--finding NULL (A NULL value could mean 'zero' - if something doesn't happen, it can't be logged in a table. However, NULL can also mean 'unknown' or 'missing'.)
WHERE column IS NULL;
--or
WHERE column IS NOT NULL;

--Using AND (it works like a comma)
WHERE column = 1999
	AND column = 'Artist'
	AND column = 'blah';

--Using OR
WHERE 
	column1 = 1999
	OR column1 > 2002;

--OR and AND together, without () it will assume column2 is not related to column1
WHERE 
	column1 = 'artist'
	AND (
		column2 = 1994
		OR column2 > 2002
	);

--Using IN
WHERE 
	column1 IN ('artist1', 'artist2')

--Using LIKE (retrieve info from a specific letter type)
WHERE song LIKE 'a%';
--returns all songs with letter A

```

WHILE loop

```sql
DECLARE @ctr INT
SET @crt = 1
WHILE @crt<10
	BEGIN
		SET @crt = @crt +1
		--creating a break
		if @crt = 4
			BREAK
	END
--view result
SELECT @crt
```

Window Functions

creates groups of data for analyzing 

creates a window with OVER clause

PARTITION BY creates a frame, if not, the frame is the entire table

arrange results with ORDER BY

allows aggregations

```sql
OVER (PARTITION BY SalesYear ORDER BY SalesYear)
```

FIRST_VALUE(column_name): returns the first value in the window

LAST_VALUE(column_name): returns the last value in the window

LEAD(): getting the next value (adjacent row) from the next row

requires ORDER BY to order rows

```sql
SELECT sales, year, current
--create a window function to get values from the next row
	LEAD(current)
	OVER (PARTITION BY year ORDER BY date) AS Next,
	date AS ModDate
FROM...
```

LAG(): getting the previous value/row

The first row in a window where the `LAG()` function is used is NULL.

ROW_NUMBER(): creating an index, adds up rows, 

```sql
SELECT sales, year, current
	ROW_NUMBER()
	OVER (PARTITION BY year ORDER BY date) AS Next,
FROM...
```

Satistics

STDEV: standard deviation 

```sql
SELECT sales, year, current
	STDEV(current)
	OVER () AS StandardDev,
	--or
	OVER (PARTITION BY year ORDER BY year) AS StDev
FROM...
```

Mode

create a CTE containing ordered count of values using ROW_NUMBER

Then, write a query using the CTE to pick values with highest #

```sql
WITH QuotaCount AS (
SELECT Person, SalesYear, CurrentQuota,
	ROW_NUMBER()
	OVER (PARTITION BY CurrentQuota ORDER BY CurrentQuota) AS QuotaList 
FROM SaleGoal)

SELECT CurrentQuota, QuotaList AS Mode
FROM QuotaCount
WHERE QuotaList IN (SELECT MAX(QuotaList) FROM QuotaCount)
```

UNION & UNION ALL

combining same or different tables with the same info/structure and same data types

UNION - Duplicated rows are excluded

UNIQUE

`UNIQUE`
 keyword after the `column_name` that should be unique. This, of course, only works for *new* tables

```sql
CREATE TABLE table_name (
 column_name UNIQUE
);
```

add a unique constraint to an *existing* table, you do it like that:

```sql
ALTER TABLE table_name
ADD CONSTRAINT some_name UNIQUE(column_name);
```

UPDATE

```sql
UPDATE table
SET column = value
WHERE
	--conditions

--update columns of a table based on values in another table:
UPDATE table_a
SET column_to_update = table_b.column_to_update_from
FROM table_b
WHERE condition1 AND condition2 AND ...;

```

tip: dont forget the WHERE clause, otherwise it will update all the columns