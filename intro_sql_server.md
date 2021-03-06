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

--Adding new column 
ALTER TABLE table_name
ADD column_name data_type

-- Add the book_id foreign key
ALTER TABLE fact_booksales(--table you want to add) ADD CONSTRAINT sales_book
    FOREIGN KEY (book_id) REFERENCES dim_book_star(--table where it came from) (book_id);
```

Calendar Tables

is a helpful utility table which you can use to perform date range calculations quickly and efficiently.

```sql
--Building a calendar 
--creating columns
CREATE TABLE dbo.Calendar
( DateKey INT NOT NULL,
	[Date] DATE NOT NULL,
	DayOfWeek TINYINT NOT NULL,
	...)
--creating days
SELECT 
	CAST(D.DateKey AS INT) AS DateKey,
	D.[DATE] AS [DATE],
	CAST(d.[dayofweek] AS TINYINT) AS [DayOfWeek],
	....

---- Find Tuesdays in December for calendar years 2008-2010
SELECT
	c.Date
FROM dbo.calendar c
WHERE
	c.MonthName = 'December'
	AND c.DayName = 'Tuesday'
	AND c.CalendarYear BETWEEN 2008 AND 2010
ORDER BY
	c.Date;
```

JOINING CALENDAR

```sql
SELECT
    t.Column1,
    t.Column2
FROM dbo.Table t
    INNER JOIN dbo.Calendar c
        ON t.Date = c.Date;
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

ORDER BY
	CASE WHEN DATENAME(WEEKDAY, StartDate) = 'Sunday' THEN 1
END
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

```sql

--returns expression based on data_type
CAST(expression AS data_type [(length)])

```

Convert a value to an int datatype

```sql
-- Calculate the net amount as amount + fee
SELECT transaction_date, amount + CAST(fee AS integer) AS net_amount 
FROM transactions;
```

Rounding Functions

CEILING(numeric_expression)

rounds up to the nearest int

returns the smallest INT greater than or equal to the expression

```sql
CEILING(-50.46) AS A
--output -50
CEILING(70.73) AS B
--output 71
```

CHARINDEX()

looks for a character expression in a given string. Returns its starting position 

```sql
CHARINDEX(expression_to_find, expression_to_search [, start_location])

SELECT 
	first_name,
	last_name,
	email 
FROM voters
-- Look for the "dan" expression in the first_name
WHERE CHARINDEX('dan', first_name) > 0 
    -- Look for last_names that do not contain the letter "z"
	AND CHARINDEX('z', last_name) = 0;
```

Cleaning Data

Data imputation(replacing 0 for example)

-mean

it can increase correlation among other columns

```sql
CREATE PROCEDURE dbo.ImputDurMean
AS 
BEGIN
DECLARE @AvgTripDuration AS float

SELECT @AvgTripDuration = AVG(Duration)
FROM CapitalBikeShare
WHERE Duration > 0

UPDATE CapitalBikeShare
SET Duration = @AvgTripDuration
WHERE Duration = 0
END; 
```

-hot Deck (picking it a random value from the data set)

```sql
CREATE FUNCTION dbo.GetDurHotDeck()
RETURNS decimal (18,4)
AS BEGIN
RETURN(SELECT TOP 1 Duration
FROM CapitalBikeShare
TABLESAMPLE (1000 rows)
WHERE Duration > 0)
END
--creating a function every time a 0 is found
SELECT
	StartDate,
	'TripDuration' = CASE WHEN Duration >0 THEN Duration
		ELSE dbo.GetDurHotDeck() END
FROM CaptialBikShare;
```

-omission

Things to consider

-data size

-end goal 

-data distribution 

```sql
SELECT
	DATENAME(weekday, StartDate) AS DayofWeek,
	AVG(Duration) AS 'AvgDuration'
FROM CapitalBikeShare
WHERE Duration>0
GROUP BY DATENAME(weekday,StartDate)
ORDER BY AVG(Duration)DESC
```

CREATE TABLE

unique table name

(column name, data type, size)

```sql
CREATE TABLE test_table(
	Test_date date,
	test_name text,
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

MAX: high, latest

MIN: low, previous

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
CONCAT(string1, string2 [,stringN])

-creating surrogate and primary keys
ALTER TABLE 
ADD COlUMN column_c VARCHAR (250)

UPDATE TABLE
SET column_c = CONCAT(column_a, column_b) --Surrogate key
ALTER TABLE table_name
ADD CONSTRAINT pk PRIMARY KEY (column_name)

```

CONCAT_WS()

```sql
CONCAT_WS(seperator, string1, string2 [,stringN])
CONCAT_WS(' ', 'Apples', 'and', 'oranges') AS result
--automatically adds the spaces in the string
```

Data

Downsampling

roll up to the hour instead of day

```sql
--Starting from the beginning until an occurance, then datediff returns an int of #of hours a rounding total
SELECT
	DATEADD(HOUR,DATEDIFF(HOUR,0,SomeDate),0)AS SomeDate
FROM...

--first day of the current week
SELECT DATEADD(week, DATEDIFF(week, 0, GETDATE()),0)
```

Upsampling

CREATE ROLE

```sql
-empty role
CREATE ROLE data_analyst;

-role for temp
CREATE ROLE intern WITH PASSWORD 'PasswordForIntern' VALID UNTIL '2020-01-01'

--access to create
CREATE ROLE admin CREATEDB

-- Create an admin role with two attributes 
CREATE ROLE admin WITH CREATEDB CREATEROLE;

--changing roles
ALTER ROLE admin CREATEROLE;
--continue
-- Add Marta to the data scientist group
GRANT data_scientist TO marta;
-- Celebrate! You hired data scientists.
-- Remove Marta from the data scientist group
REVOKE data_scientist FROM marta;
```

Date Functions

Higher Precision

- `SYSDATETIME()`
- `SYSUTCDATETIME()`
- `SYSDATETIMEOFFSET()`

Lower Precision

- `GETDATE()`
- `GETUTCDATE()`
- `CURRENT_TIMESTAMP`

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
--DATENAME syntax
DATENAME(datepart, date)
--returns nvarchar
--returns a string value
SELECT DATENAME(MONTH,@dt) AS theMonth;
```

DATEFROMPARTS

```sql
DATEFROMPARTS(2019,3,5) as new_date;

-- Create a date as the start of the month of the first vote
	DATEFROMPARTS(YEAR(first_vote_date), MONTH(first_vote_date), 1) AS first_vote_starting_month
```

Database views

is the result set of a stored query on the data, which the database users can query just as they would in a persistent database collection object

```sql
CREATE VIEW view_name AS
SELECT col1, col2
FROM table_name
WHERE condition;
```

viewing views in the database

```sql
--includes system views
SELECT* FROM INFORMATION_SCHEMA.views;

--excludes
SELECT* FROM INFORMATION_SCHEMA.views
WHERE table_schema NOT IN ('pg_catalog', 'information_schema');
```

DATEPART

```sql
DATEPART(datepart, date)
--returns int
```

determine what part of date you want to calculate

DD: day

wk, ww: week

MM: month

YY: year

HH: hour

DY, Y : dayofyear

dw, w: weekday

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
--returns an int; cant use datepart weekday value
--obtain the difference bwt two datetime values, always returns a number, and rounds up
DATEDIFF(datepart, startdate, endate
SELECT DATEDIFF(DD, '2020-05-22', '2020-06-21')
--output will. be 30
```

SQL Server does not have an intuitive way to round down to the month, hour, or minute. You can, however, combine the??`DATEADD()`??and??`DATEDIFF()`??functions to perform this rounding.

To round the date 1914-08-16 down to the year, we would call??`DATEADD(YEAR, DATEDIFF(YEAR, 0, '1914-08-16'), 0)`. To round that date down to the month, we would call??`DATEADD(MONTH, DATEDIFF(MONTH, 0, '1914-08-16'), 0)`

Building dates from parts

```sql
DATEFROMPARTS(year, month, day)
--take int values and returns a date type
TIMEFROMPARTS(hour, min, sec, fraction, precision)
DATETIMEFROMPARTS(year, month, day, hour, min, sec,ms)
--for more percision
DATETIME2FROMPARTS(year, month, day, hour, min, sec, fraction, precision)
DATETIMEOFFSETFROMPARTS(year, month, day, hour, min, sec, fraction, hour_offset, min_offset, precision)
	--Changing offsets
	DECLARE @someDate DATETIMEOFFSET=
		'2019-04-10 12:59:02.2345 -04:00';
	SELECT SWITCHOFFSET(@someDate, '-07:00') AS LATime;
	--converting to DATETIMEOFFSET
	DECLARE @someDate DATETIME2(3)=
		'2019-04-10 12:59:02.234';
	SELECT TODATETIMEOFFSET(@someDate, '-04.00') AS EDT;
	--Timezone swaps with TODATETIMEOFFSET
	DECLARE @someDate DATETIME2(3)=
		'2019-04-10 12:59:02.234';
	SELECT TODATETIMEOFFSET(
		DATEADD(Hour, 7, @someDate),
		'+02:00')AS BonnTime;

--example
DATETIMEFROMPARTS(1918, 11, 11, 05, 45, 17, 995) AT TIME ZONE 'UTC'AS DT

--ex2
SELECT TOP(10)
	c.CalendarQuarterName,
	c.MonthName,
	c.CalendarDayOfYear
FROM dbo.Calendar c
WHERE
	-- Create dates from component parts
	DATEFROMPARTS(c.CalendarYear, c.CalendarMonth, c.Day) >= '2018-06-01'
	AND c.DayName = 'Tuesday'
ORDER BY
	c.FiscalYear,
	c.FiscalDayOfYear ASC;
```

SWITCHOFFSET()

function to change the time zone of a??`DATETIME`
,??`DATETIME2`
, or??`DATETIMEOFFSET`
??typed date or a valid date string

takes two parameters: the date or string as input and the time zone offset. It returns the time in that new time zone,

Discovering time zones

```sql
SELECT
	tzi.name,
	tzi.current_utc_offset,
	tzi.is_currently_dst
FROM sys.time_zone_info tzi
	WHERE tzi.name LIKE '%Time Zone%';
```

Duplicate Row Managment 

```sql
--Distinct
SELECT DISTINCT (PlayerName)
FROM table;

SELECT PlayerName
FROM table
GROUP BY PlayerName;

SELECT PlayerName,
	COUNT(TEAM) AS TeamsPlayedFor
FROM table
GROUP BY PlayerName;

--Remove Duplicate with UNION
```

Error-handling 

-safe versions

-doest the job with a NULL

TRY_PARSE

TRY_CONVERT

TRY_CAST

```sql
--enclose statements within the try block
BEGIN TRY
	{sq_statement| statement_block}
END TRY
--place error handling code within the catch block
BEGIN CATCH
	[{sql_statement | statement_block}]
END CATCH
[;]

--ex
BEGIN TRY
    INSERT INTO products (product_name, stock, price)
        VALUES ('Trek Powerfly 5 - 2018', 10, 3499.99);
    SELECT 'Product inserted correctly!';
END TRY
BEGIN CATCH
     SELECT 'An error occurred! You are in the CATCH block';   
END CATCH

```

Error Functions

ERROR_NUMBER(): returns  # of error

ERROR_SEVERITY(): returns error severity (11-19)

ERROR_STATE(): returns state of the error

ERROR_LINE(): # of line of the error

ERROR_PROCEDURE(): returns name of stored procedure or trigger

ERROR_MESSAGE(): returns the text message

ways of customizing 

concatenating strings

```sql
CONCAT()
```

FORMATMESSAGE function

```sql
FORMATMESSAGE({'msg_string'|msg_number}, [param_value [,...n]]) 

--example
DECLARE @product_name AS NVARCHAR(10) = 'Trek CrossRip+ - 2018';
-- Set the number of sold bikes
DECLARE @sold_bikes AS INT = 100;
DECLARE @current_stock INT;

SELECT @current_stock = stock FROM products WHERE product_name = @product_name;

DECLARE @my_message NVARCHAR(500) =
	-- Customize the error message
	FORMATMESSAGE('There are not enough %s bikes. You have %d in stock.', @product_name, @current_stock);

IF (@current_stock - @sold_bikes < 0)
	-- Throw the error
	THROW 50000, @my_message, 1;

--ouput: if nvarchar changed to 50
error message: ('42000', '[42000] [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]There are not enough Trek CrossRip+ - 2018 bikes. You have 12 in stock. (50000) (SQLExecDirectW)')
```

 

```sql
BEGIN CATCH
	SELECT ERROR_NUMBER() AS ERROR_NUMBER
END CATCH
```

Formatting Functions

CAST()

Dates

```sql
--casting strings
SELECT 
	CAST('09/19/99' AS DATE) AS USDate;
```

CONVERT()

```sql
CONVERT(data_type [(length)], expression [,style])

-- Group by the date portion of StartDate (a column)
GROUP BY CONVERT(DATE, StartDate)
-- Sort the results by the date portion of StartDate
ORDER BY CONVERT(DATE, StartDate);
```

Used to change data types to another

like CAST but there is more control over formatting from dates to strings with using an optional style(its the third parameter)

Dates

```sql
SELECT 
	CONVERT(DATETIME2(3),
		'April 4, 2019 11:52:29.998 PM') AS April4
```

FORMAT()

more flexible then the two above but slower (around 50,000 rows)

FORMAT w/ CAST

```sql
--3/9/2019
SELECT TOP 5
FORMAT(CAST(StartDate as Date), 'd', 'en-us')
AS 'US Date',
--1,444,777.00
FORMAT(SUM(DURATION), 'n', 'en-us)
--or
FORMAT(SUM(DURATION), '#,0.00')
```

PARSE()

take non traditional date formate into date types

to parse a string as a date type using a spe

VERY SLOW

```sql
SELECT
	PARSE('25 Dezember 2014' AS DATE
		USING 'de-de') AS Weihnachten;
```

DECLARE

creating variable to avoid repeatability 

```sql
DECLARE @

--integer
DECLARE @test_int INT

--Varchar
DECLARE @my_artist VARCHAR(100)

--after the declare has been put, next step is SET to assigning the variable 
DECLARE @test_int INT
SET @test_int = 5

-- Declare @Shifts as a TABLE
DECLARE @Shifts table(
    -- Create StartDateTime column
	StartDateTime datetime,
    -- Create EndDateTime column
	EndDateTime datetime)
-- Populate @Shifts
INSERT INTO @Shifts (StartDateTime, EndDateTime)
	SELECT '3/1/2018 8:00 AM', '3/1/2018 4:00 PM'
`
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

EXISTS

analytic function

FIRST_VALUE()

returns the first value in an ordered set

```sql
FIRST_VALUE(numeric_expression)
	OVER ([[PARTITION BY column] ORDER BY column ROW_or_RANGE frame)
```

OVER clause components

PARTITION : optional - divides the result set into partitions

ORDER BY: mandatory - order set result

ROW_or_RANGE: optional - sets partition limits

Partition Limits

```sql

--this will take all rows from the table 
RANGE BETWEEN start_boundary AND end_boundary

ROW BETWEEN start_boundary AND end_boundary

```

Boundary

UNBOUNDED PRECEDING: first row in partition

UNBOUNDED FOLLOWING: last row in partition

CURRENT ROW

PRECEDING: previous row

FOLLOWING: next row

Rounding Functions

FLOOR(numeric_expression)

rounds down tot he nearest int

returns the largest INT less than or equal to the expression

```sql
FLOOR(-50.46) AS A
--output -51
FLOOR(70.73) AS B
--output 70
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

AGgregating by group. Use WHERE for individual rows and HAVING for numeric filter on grouped rows

Dont use HAVING to filter individual or ungrouped rows ???inefficient 

HAVING can only be applied to a numeric column in an aggregate function filter

Horizontal Partitioning

splitting tables into subcategories by splitting the rows.

Vertical partitioning- moves specific columns to seperate tabes 

```sql
--creating quarter partitions (table: id, product_id,amount, total_price, timestamp)
CREATE TABLE sales(
	...
	timestamp DATA NOT NULL
)
PARTITION BY RANGE (timestamp);
CREATE TABLE sales_2019_q1 PARTITION OF sales
	FOR VALUES FROM ('2019-01-01') TO ('2019-03-31');
CREATE TABLE sales_2019_q4 PARTITION OF sales
	FOR VALUES FROM ('2019-09-01') TO ('2019-12-31');
CREATE INDEX ON sales ('timestamp');

-- Create a new table called film_partitioned
CREATE TABLE film_partitioned (
  film_id INT,
  title TEXT NOT NULL,
  release_year TEXT
)
PARTITION BY LIST (release_year);

-- Create the partitions for 2019, 2018, and 2017
CREATE TABLE film_2019
	PARTITION OF film_partitioned FOR VALUES IN ('2019');
    
CREATE TABLE film_2018
	PARTITION OF film_partitioned FOR VALUES IN('2018');
    
CREATE TABLE film_2017
	PARTITION OF film_partitioned FOR VALUES IN ('2017');
```

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

Indexes

Clustered index

nonclustered

![0A0B1885-4A46-4529-A034-834572EBAD33_4_5005_c.jpeg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/28742515-6529-4c3f-b1bb-d90971443cfb/0A0B1885-4A46-4529-A034-834572EBAD33_4_5005_c.jpeg)

INTERSECT

???THis example, includes how many clients ordered somthing

```sql
--ordered products
SELECT customer
FROM customers
INTERSECT
SELECT customer
FROM orders

--didnot order products
SELECT customer
FROM customers
EXCEPT
SELECT customer
FROM orders
```
```

ISDATE(expression)

Determine whether an expression is a valid data type

ISNULL

substituting missing data with specific values- can replace with only one value 

```sql
SELECT column1, column2
ISNULL(column2, "Unknown") AS NewCountry
FROM...

--can substitute values from one column for another 
```

GROUPING SETS

The GROUPING SETS option??**gives you the ability to combine multiple GROUP BY clauses into one GROUP BY clause.** The results are the equivalent of UNION ALL of the specified groups.

```sql
--example
SELECT
	c.CalendarYear,
	c.CalendarMonth,
	c.DayOfWeek,
	c.IsWeekend,
	SUM(ir.NumberOfIncidents) AS NumberOfIncidents
FROM dbo.IncidentRollup ir
	INNER JOIN dbo.Calendar c
		ON ir.IncidentDate = c.Date
GROUP BY GROUPING SETS
(
    -- Each non-aggregated column from above should appear once
  	-- Calendar year and month
	(c.CalendarYear, c.CalendarMonth),
  	-- Day of week
	(c.DayOfWeek),
  	-- Is weekend or not
	(c.IsWeekend),
    -- This remains empty; it gives us the grand total
	()
)
ORDER BY
	c.CalendarYear,
	c.CalendarMonth,
	c.DayOfWeek,
	c.IsWeekend;
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

analytic function

LAST_VALUE()

returns the last value in an ordered set

```sql
LAST_VALUE(numeric_expression)
	OVER ([[PARTITION BY column] ORDER BY column ROW_or_RANGE frame)
```

OVER clause components

PARTITION : optional - divides the result set into partitions

ORDER BY: mandatory - order set result

ROW_or_RANGE: optional - sets partition limits

NOT EXISTS

```sql
--check to see if there is a staff memeber
IF NOT EXISTS(
	SELECT * 
	FROM staff 
	WHERE staff_id = 15)
		RAISERROR('No staff member...'))
```

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

PATINDEX

Similar to CHARINDEX, but more powerful. Returns starting position of a pattern in an expression

```sql
PATINDEX('%pattern%', expression, [location ])
--ex
-- Look for first names that contain "rr" in the middle
WHERE PATINDEX('%rr%', first_name) > 0;

-- Look for first names that start with C and the 3rd letter is r
WHERE PATINDEX('C_r%', first_name) > 0;

-- Look for first names that have an "a" followed by 0 or more letters and then have a "w"
WHERE PATINDEX('%a%w%', first_name) > 0;

-- Look for first names that contain one of the letters: "x", "w", "q"
WHERE PATINDEX('%[xwq]%', first_name) > 0;

  -- Select only voters with a first name less than 5 characters
WHERE LEN(first_name) < 5
   -- LopaTINok for the desired pattern in the email address
	AND PATINDEX('j_a%yahoo.com', email) > 0;

--wild chards
-- % : match any string of any length
-- _ : match on a single character
-- [] : match any character in brackets

```

Processing Order

1. From
2. On
3. Join
4. Where
5. Group By
6. Having
7. Select
8. Distinct
9. Order By
10. Top

RAISERROR 

it is recommended to use THROW instead

```sql
RAISERROR({msg_str|msg_id|@local_variable_message},
	severity,
	state,
	[argument[,...n]])
	[WITH option[,...n]]
```

WITH ROLLUP

comes after GROUP BY 

used for non-measure attribute hierarchal data, by taking the combo of each column follow by each matching value 

REVERT

Switches the execution context back to the caller of the last EXECUTE AS statement.

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

17.85??? 17           17.85??? 18

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

ABS():  absolute, returns positive numeric values, even if it is a negative

LOG(): returns the natural logarithm

```sql
LOG(number [,Base])
```

SIGN(numeric_expression): returns the sign of an expression, as an INT

SQRT(): finding the square root

Running Totals 

cumulative value 

The sum function needs to be a window function in order to get the detailed records and aggregated total. Using partition to get the totals per team. Use RANGE BETWEEN, use unbound preceding to start the calculation of winner from the beginning. The window following expression, CURRENT ROW, so the sum occur at the present row 

```sql
--example
total number of wins per team per games
SELECT
	s.team,
	s.game,
	s.runsScored,
	SUM(s.runsScored) OVER(
		PARTITION BY s.team
		ORDER BY s.Game ASC
		RANGE BETWEEN
			UNBOUNDED PRECEDING
			AND CURRENT ROW) AS TotalRuns
```

SavePoints

markers within a transaction

allows to rollback to the savepoints

```sql
SAVE {TRAN|TRANSACTION} {savepoint_name|@savepoint_variable} [;]

--example
BEGIN TRAN
	SAVE TRAN savepoint1;
	INSERT INTO customers VALUES ('m', 'd', 'md', '434343');
	ROLLBACK TRAN savepoint1;
	SELECT @@TRANCOUNT AS 'value';
COMMIT TRAN;
--output will be 0 due to the rollback
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

SELECT TOP 25 PERCENT 
-- Limit rows to the upper quartile

--Return unique rows (if there are mulitple of one, it will only return one)
SELECT DISTINCT column 
--example
-- Select the unique date values of StartDate and EndDate
SELECT DISTINCT
    -- Cast StartDate as date
	CAST(StartDate as date),
    -- Cast EndDate as date
	CAST(EndDate as date)

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

COUNT()

COUNT_BIG()

returns a 64bit INT

```sql
--number of rows
COUNT(*) or COUNT(1) or COUNT(1/0)
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
	SUBSTRING(column name/character_expression, 12(#of character to start from), 12(#of character to extract) AS new_name

SELECT
    -- Concatenate the first and last name
	CONCAT('***' , first_name, ' ', UPPER(last_name), '***') AS name,
    -- Mask the last two digits of the year
    REPLACE(birthdate, Substring(CAST(birthdate AS varchar), 3, 2), 'XX') AS birthdate,
	email,
	country

--find a specific character within a string
SELECT
	CHARINDEX('What want to find', column name) AS new_name

--replacing characters
SELECT 
	TOP(5) REPLACE(column name, 'character', 'replaced character') AS new_name
--removing
-- Remove the text '(Valrhona)' from the name
	REPLACE(company, '(Valrhona)', '') AS new_company,
```

STRING_AGG()

Concatenates the values of string expressions and places separator values between them. 

```sql
STRING_AGG(expression, seperator) [<order_clause>]
[WITHIN GROUP (ORDER BY expression)

SELECT 
	STRING_AGG(column_name, ',') AS column_name
--output is the column values with commas

--create a column with first, last name and the (first vote date)
SELECT
	STING_AGG(CONCAT(first_name, ' ', last_name, ' (', first_vote_date, ')'), CHAR(13)) AS list_of_names

--creating a list from a column from specific rows
SELECT
	-- Create a list with all bean origins, delimited by comma
	string_agg(bean_origin, ',') AS bean_origins
FROM ratings
WHERE company IN ('Bar Au Chocolat', 'Chocolate Con Amor', 'East Van Roasters');
```

STRING_SPLIT()

Divides a string into smaller pieces, based on a separator. Return a single column

```sql
STRING_SPLIT(string, seperator)

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

SET DATEFORMAT

sets the order of the dataparts for interpreting stings as dates

```sql
SET DATEFORMAT {format}

--it will not consider the date as a valid date only 30-12-2019
DECLARE @date1 NVARCHAR(20) = '12-30-2019'
SET DATEFORMAT dmy;
SELECT ISDATE(@date1) AS invalid_dmy;
```

Sub-Query

```sql
--used with FROM
SELECT col1, col2, col3
--this subquery calculated a new column 
FROM
	(SELECT*,
	DATEDIFF(DAY,col4,col5) AS Name
	FROM table) AS o
WHERE col3 = 35;

--used with WHERE
SELECT..
FROM..
WHERE CustomerID
	IN(SELECT CustomerID
		FROM Orders
		WHERE col3>800);

--used with SELECT
SELECT CustomerID,
	Company,
	(SELECT AVG(Freight)
	FROM Orders o
	WHERE c.CustomerID = o.CustomerID) AS AvgFreight
FROM..
```

Temporary Tables

using a # to create a temp table 

THROW

return an error caught by catch block

A THROW statement w/o parameters re-throws the OG error and SHOULD be placed within the CATCH block

```sql
BEGIN CATCH or TRY
	THROW[error_number, message, state][;]
	SELECT ...
END CATCH or TRY
```

syntax -place the throw before a select statement, if not use ; at the end of both

Transaction Statements 

```sql
BEGIN {TRAN|TRANSACTION}
	[{transaction_name|@tran_name_variable}
		[WITH MARK['description']]
	]
[;]

--the end of transaction
COMMIT[{TRAN|TRANSACTION}[transaction_name|tran_name_variable]]
	[WITH (DELAYED_DURABILITY = {OFF|ON})] [;]

--reversing back a transaction, buy using it in a try and catch block, if there is a mistake, then it will go back to og state
ROLLBACK {TRAN|TRANSACTION}
	[transaction_name|@tran_name_variable|
		savepoint_name|@savepoint_variable]
[;]

--example
Account 1 = $24k
Account 5 = $35k
BEGIN TRAN;
	UPDATE accounts SET current_balance=current_balance-100 WHERE account_id=1;
	INSERT INTO transactions VALUES (1, -100, GETDATE());
	UPDATE accounts SET current_balance=current_balance+100 WHERE account_id=5;
	INSERT INTO transactions VALUES (5, 100, GETDATE());
COMMIT TRAN;
```

Transaction isolation levels

doing one trans at a time

```sql
SET TRANSACTION ISOLATION LEVEL
	{READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE | SNAPSHORT
```

comparisons cons

Read uncommitted: dirty reads, non-repeatable reads, phantom reads

read committed: non-repeatable reads, phantom reads

repeatable read: phantom reads

serializable: none, you can get blocked and will need to wait for a new trans

snapshots: none, modifications are stored in tempDB

```sql
ALTER DATABASE myDatabaseName SET ALLOW_SNAPSHOT_ISOLATION ON;
```

WITH (NOLOCK)

similar to READ UNCOMMITTED. used to read uncommitted data

applies to a specific table

```sql
SELECT *
	-- Avoid being blocked
	FROM transactions WITH (NOLOCK)
WHERE account_id = 1
```

Triggers

type of stored procedure that activated when an event occurs in the database server

types

data manipulation language triggers

INSERT, UPDATE, DELETE

used with AFTER or INSTEAD OF

attached to tables or view

inserted and deleted special tables

Data definition language triggers

CREATE, ALTER, DROP

only used with AFTER

attached to databases or servers

no special tables

Logon triggers

LOGON events

```sql
--create a trigger
CREATE TRIGGER NameofTrigger
--linking it to an existing object/table
ON TableName
--Trigger behavior type
--this example is of insert
AFTER INSERT
-- or the following to prevent deletions
INSTEAD OF DELETE
--the begin of trigger workflow
AS
--the action executed by trigger
PRINT('Message.');

--using trigger as a computed column
CREATE TRIGGER [CalcTotalAmount]
ON [SalesWithoutPrice]
AFTER INSERT
AS
	UPDATE [sp]
	SET [sp].[TotalAmount]=[sp].[Quantity]*[p].[Price]
	FROM [SalesWithoutPrice] AS [sp]
	INNER JOIN [Products] AS [p] ON [sp].Product = [p].[Product]
	WHERE [sp].[TotalAmount] IS NULL;
```

```sql
--DDL trigger
CREATE TRIGGER TrackDatabaseTable
ON Database
--FOR is a synonym of AFTER and performs the trigger's set of actions after the triggering event finishes.
FOR Create_Table,
		ALTER_TABLE,
		DROP_TABLE
		
AS 
	INSERT INTO TablesChangeLog(EventData, ChangedBy)
	VALUES (EVENTDATA(), USER);

--DDL tigger capabilities
--database level
CREATE_TABLE, ALTER_TABLE,DROP_TABLE
CREATE_VIEW,ALTER_VIEW,DROP_VIEW
CRATE_INDEX,....
ADD_ROLE_MEMBER,DROP_ROLE_MEMBER
CREATE_STATISTICS,DROP_STATISTICS

--server level
CREATE_DATABASE, ALTER_DATABASE, DROP_DATABASE
GRANT_SERVER, DENY_SERVER, REVOKE_SERVER
CREATE_CREDENTIAL, ALTER_CREDENTIAL, DROP_CREDENTIAL
```

```sql
--logon triggers
CREATE TRIGGER LogonAudit
--attached at the server level
--sa account for security purposes
ON ALL SERVER WITH EXECUTE AS 'sa'
FOR LOGON
AS
	INSERT INTO ServerLogonLog
							(loginName, loginDate, SessionID, SourceIPAddress)
	SELECT ORIGINAL_LOGIN(), GETDATE(), @SPID, client_net_address
	FROM SYS.DM_EXEC_CONNECTIONS
	WHERE session_id = @SPID;
```

Deleting and Altering triggers

```sql
--deleting a trigger
DROP TRIGGER TriggerName;
--database
DROP TRIGGER TriggerName;
ON Database;
--server level
DROP TRIGGER TriggerName;
ON ALL SERVER;

--disabling 
DISABLE TRIGGER triggerName
ON Table;
--enabling 
ENABLE TRIGGER triggerName
ON Table;

--updating triggers
ALTER TRIGGER
--or
DROP TRIGGER triggerName
CREATE TRIGGER--rewrite trigger again
```

Finding server-level triggers

SQL Server provides information about the execution of any triggers that are currently stored in memory in the??`sys.dm_exec_trigger_stats`

```sql
--Finding server-level triggers
SELECT*FROM sys.server_triggers;

--finding database and table triggers
SELECT*FROM sys.triggers;

```

Viewing trigger definition

```sql
SELECT definition
FROM sys.sql_modules
WHERE object_id = OBJECT_ID('PreventOrdersUpdate');
--output you get the create trigger query 
--or
SELECT OBJECT_DEFINITION (OBJECT_ID('PreventOrdersUpdate'));
--or
EXECUTE sp_helptext @objname='PreventOrdersUpdate';
```

TIPS for best practice

-well documented database design

-simple logic in trigger design

-avoid overusing triggers

real case example

adding a modify date when there is a change to customers table

```sql
	CREATE TRIGGER CopyCustomersToHistory
--select the table where info is from, where the change occured
ON Customers
--the change is inserted into CopyCustomersToHistory
AFTER INSERT, UPDATE
AS
	INSERT INTO CustomerHistory (Customer, ContractID, Address, PhoneNo)
	SELECT Customer, ContractID, Address, PhoneNo, GETDATE()
	FROM inserted;

```

creating notification triggers

```sql
CREATE TRIGGER NewOrderNotification
ON Orders
AFTER INSERT
AS
	EXECUTE SendNotifiation @RecipientEmail = "email address"
						,@EmailSubject = "Subject"
						, @EmailBody = "Message.";
```

WHERE

return rows that met a certain criteria

can filter ???text???, numbers, ???dates??? (year-month-day)

Calculations in WHERE filter condition, but can increase query time

```sql
SELECT col1,
	col2,
	(col3 + col4) AS Total
FROM table
WHERE (col3 + col4) >= 1000
```

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

When creating window functions there are two options 

RANGE

-Specifies a range of results

-Duplicates processed all at once, so if theres more than one, it sum both of them for both records 

-Supports UNBOUNDED and CURRENT ROW 

ROWS

-specifies a # of rows 

-Duplicates processed a row at a time, the second duplicate will have the total of the previous 

-Supports UNBOUNDED and CURRENT ROW and # of rows

```sql
SELECT
	ir.IncidentDate,
	ir.IncidentTypeID,
	ir.NumberOfIncidents,
    -- Fill in the correct window function
	AVG(ir.NumberOfIncidents) OVER (
		PARTITION BY ir.IncidentTypeID
		ORDER BY ir.IncidentDate
      	-- Fill in the three parts of the window frame
		ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
	) AS MeanNumberOfIncidents
FROM dbo.IncidentRollup ir
	INNER JOIN dbo.Calendar c
		ON ir.IncidentDate = c.Date
WHERE
	c.CalendarYear = 2019
	AND c.CalendarMonth IN (7, 8)
	AND ir.IncidentTypeID = 1
ORDER BY
	ir.IncidentTypeID,
	ir.IncidentDate;
```

DENSE_RANK()

Ascending INT value start from 1. Can have ties, not skip #

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

The first row in a window where the??`LAG()` function is used is NULL.

```sql
SELECT
	dsr.Customer,
	dsr.MonthStart
	LAG(dsr.NumberVisits) OVER (PARTITION BY dsr.Customer ORDER BY dsr.MonthStart) AS Prior
	dsr.NumberVisits
```

Creating filters and CTE

```sql

--Getting prior values after filtering out previous information
WITH records AS (
	SELECT 
		Date,
		LAG(Val, 1) OVER (ORDER BY Date) AS PriorVal
		Val
FROM t
)
SELECT 
	r.Date,
	r.PriorVal,
	r.Val
FROM records r
WHERE
	r.Date > '2020-01-02'
```

Using DATEDIFF and LAG/LEAD

```sql
SELECT
	ir.IncidentDate,
	ir.IncidentTypeID,
    -- Fill in the days since last incident
	DATEDIFF(DAY, LAG(ir.IncidentDate, 1) OVER (
		PARTITION BY ir.IncidentTypeID
		ORDER BY ir.IncidentDate
	), ir.IncidentDate) AS DaysSinceLastIncident,
    -- Fill in the days until next incident
	DATEDIFF(DAY, ir.IncidentDate, LEAD(ir.IncidentDate, 1) OVER (
		PARTITION BY ir.IncidentTypeID
		ORDER BY ir.IncidentDate
	)) AS DaysUntilNextIncident
FROM dbo.IncidentRollup ir
WHERE
	ir.IncidentDate >= '2019-07-02'
	AND ir.IncidentDate <= '2019-07-31'
	AND ir.IncidentTypeID IN (1, 2)
ORDER BY
	ir.IncidentTypeID,
	ir.IncidentDate;
```

ROW_NUMBER(): creating an index, adds up rows, 

```sql
SELECT sales, year, current
	ROW_NUMBER()
	OVER (PARTITION BY year ORDER BY date) AS Next,
FROM...
```

RANK()

Ascending INT value starting from 1. Can have ties, can skip # (not unique)

Statistics

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
??keyword after the??`column_name`??that should be unique. This, of course, only works for??*new* tables

```sql
CREATE TABLE table_name (
 column_name UNIQUE
);
```

add a unique constraint to an??*existing*??table, you do it like that:

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

User defined functions

routines that accept input parameters, perform action, return results

Helps with execution time, network traffic and modular programming 

```sql
--Scaler function with no input parameters
CREATE FUNCTION GetTomorrow()
	RETURNS date AS BEGIN
RETURN(SELECT DATEADD(day, 1, GETDATE()))
END

--executing a function
SELECT dbo.GetTommorrow()
--output a date
-- dbo. is the schema where the function exist
--a schema must be specified for execution

--0r
FROM dbo.SumLocationStats('00/00/0000')
ORDER BY RideCount DESC
--0utput three column of that yr
--order by is not allow within the function itself

--EXEC & Store results in variable
DECLARE @TotalRideHrs AS numeric
EXEC @TotalRideHrs = dbo.GetRideHrsOneDay @DateFarm = '00/00/0000'
SELECT
	'Total Ride Hours for 00/00/0000:',
	@TotalRideHrs

--declare parameter variable
--set to oldest data
--pass to function with select
DECLARE @DateParm as date = 
(SELECT TOP 1 CONVERT(date, PickupDate)
	FROM YellowTripData
	ORDER BY PickupDate DESC)
SELECT @DateParm, dbo.GetRideHrsOneDay(@DateParm)
--output 0000-00-00 75987(riding hrs)

--example
-- Create SumRideHrsSingleDay
CREATE FUNCTION SumRideHrsSingleDay (@DateParm date)
-- Specify return data type
RETURNS numeric
AS
-- Begin
BEGIN
RETURN
-- Add the difference between StartDate and EndDate
(SELECT SUM(DATEDIFF(second, StartDate, EndDate))/3600
FROM CapitalBikeShare
 -- Only include transactions where StartDate = @DateParm
WHERE CAST(StartDate AS date) = @DateParm)
-- End
END

--execute TVF into a variable
-- Create @StationStats
DECLARE @StationStats TABLE(
	StartStation nvarchar(100), 
	RideCount int, 
	TotalDuration numeric)
-- Populate @StationStats with the results of the function
INSERT INTO @StationStats
SELECT TOP 10 *
-- Execute SumStationStats with 3/15/2018
FROM dbo.SumStationStats('3/15/2018') 
ORDER BY RideCount DESC
-- Select all the records from @StationStats
SELECT * 
FROM @StationStats
```

Conversion UDFs

```sql

--Function example 1
CREATE FUNCTION dbo.ConvertMileToMeter(@miles numeric)
RETURNS numeric
AS
BEGIN
RETURN (SELECT @miles * 1609.34)
END

--Function example 2
CREATE FUNCTION dbo.ConvertCurrency(@Currency numeric, @Exchange numeric)
RETURNS numeric
AS
BEGIN
RETURN (SELECT @ExhangeRate * @Currency)
END

--change function to not round
ALTER FUNCTION dbo.ConvertMileToMeter(
	@miles numeric(18,2)
) RETURNS numeric(18,2) as BEGIN RETURN(
	SELECT @miles * 1609)
) END;

--using Function 1, 2
SELECT TripDistance as 'MileDistance',
dbo.ConvertMileToMeter(TripDistance) as MeterDistance
FareAmount a
--output 4columns( FareUSD FareGBP)

--Creating a Shift function
CREATE FUNCTION dbo.GetShift(@Hour int)
RETURNS int
AS
BEGIN
RETURN(CASE
	WHEN @Hour >= 0 AND @Hour <9 THEN 1
	WHEN @Hour >= 9 AND @Hour <18 THEN 2
	WHEN @Hour >= 18 AND @Hour <24 THEN 3
END)
END;
```

CRUD

```sql
--C for create
CREATE PROCEDURE dbo.abcCreate(
	@word as date,
	@word2 as numeric( 18,0)
) AS BEGIN INSERT INTO dbo.TripSummary(Date, TripHours)
VALUES
	(@tripDate, @tripHours)
SELECT Date, TripHours
FROM dbo.TripSummary
WHERE date= @tripDate END

--R for read
CREATE PROCEDURE abcRead
	(@tripDate as date)
AS
BEgin
SELECT Date, TripHours
FROM TripSummary
WHERE Date = @tripDate
END;

--U for Update
CREATE PROCEDURE dbo.abcUpdate
	(@tripDate as date,
	@TripHours as numeric(18, 0))
AS 
BEGIN
UPDATE dbo.TripSummary
SET Date = @TripDate,
	TripHours = @TripHours
WHERE date = @tripDate
END;

--D for Delete
CREATE PROCEDURE dbo.abcDelete
	(@tripDate as date,
	@TripHours int OUTPUT)
AS 
BEGIN
DELETE 
FROM
WHERE date = @tripDate
SET @TripHours= @...
END;

```

Executing

no output parameter or return value

```sql
EXEC dbo.cusp_TripSummaryUpdate
	@TripDate = '00/00/0000'
	@TripHours = '300'
```

output parameter

```sql
--local variable is created to store the output vallue
Declare @RideHrs as numeric (18,0)
EXEC dbo.cuspSumRideHrsOneDay
	@TripDate = '00/00/0000',
	@TripHoursOut = @TripHours OUTPUT
```

return value

```sql
Declare @RideHrs as numeric (18,0)
EXEC @RideHrs = 
	dbo.cusp_tripSummaryUpdate
	@TripDate = '00/00/0000',
	@TripHoursOut = 300
SELECT @RideHrs as RideHours

--return value and output
Declare @RideHrs as int
Declare @RowCount as int

EXEC @RideHrs = 
	dbo.cusp_tripSummaryUpdate
	@TripDate = '00/00/0000',
	@TripHoursOut = @RowCount OUTPUT

SELECT @RideHrs as RideHours,
	@RowCount as RowCount

```

EXEC & store result set 

```sql
DECLARE @tripSummaryResultSet as Table(
	TripDate date,
	TripHours numeric(18,0))
INSERT INTO @tripSummaryResultSet
EXEC cusp_TripSummaryRead @TripDate = '00/00/0000'
```

Error Handling

```sql
--example
--lets try to change the data type to nvarchar(30)
ALTER PROCEDURE dbo.cusp_TripSummaryCreate
	@TripDate nvarchar(30),
	@RideHurs numeric,
--output parameter
@ErrorMsg nvarchar(max)=null OUTPUT
AS 
BEGIN
	BEGIN TRY
--where there should be an error
		INSERT INTO TripSummary (Date, TripHours)
		Values (@TripDate, @RideHrs)
END TRY
BEGIN CATCH
	SET @ErrorMsg = 'Error_Num: ' +
	CAST (ERROR_NUMBER() AS varchar) +
	'Error_Sev:' +
	CAST(ERROR_SEVERITY() AS varchar)+
	'Error_Sev:' + ERROR_MESSAGE()
END CATCH
END
```

Inline table valued functions

```sql
CREATE FUNCTION name(
	@startDate AS datetime = '0/0/0000'
)RETURNS TABLE AS RETURN 
SELECT
	column1 AS name,
	COUNT(column2) AS name,
	SUM(column3) as name
FROM table
WHERE CAST(column4 AS date) = @startDate
GROUP BY column1
```

MSTVF(multiple statement)

```sql
CREATE FUNCTION name(
	@startDate AS datetime = '0/0/0000'
	@endDate ...
)RETURNS @TripCountAvgFare TABLE (
	DropOffDate date, TripCount int, AvgFare numeric
) AS BEGIN INSERT INTO @TripCountAvgFare
SELECT 
	CAST(DropOffDate as date),
	COUNT(ID),
	AVG(fareAmount) AS name
FROM table
WHERE 
	DATEPART(month, DROPOFFDATE) = @month
	AND DATEPART(year, DROPOFFDATE) = @year
GROUP BY CAST(DropoffDate as date)
RETURN END;
```

Maintaining /changing functions

```sql
ALTER FUNCTION 
--cannot change inline to MSTVF
--in order to do that, need to 
DROP FUNCTION dbo.functionName
--then after use 
CREATE FUNCTION 

CREATE OR ALTER FUNCTION
```

Schemabinding

specifies schema is bound to database objects that it references

prevents changes to the schema if schema bound objects are referencing it

```sql
CREATE OR ALTER FUNCTION dbo.GetRideHrsOneDay(@DateParm date)
RETURNS numeric WITH SCHEMABINDING
AS
BEGIN
RETURN...
```

Stored Procedure

```sql
--creating procedure with OUTPUT parameter
--first four lines of code
--SP name must be unique
CREATE PROCEDURE dbo.cuspGetRideHrsOneDay
	--declare the input parameter	
	@DateParm date,
	--declare output parameter
	@RideHrsOut numeric OUTPUT
AS
-- Don't send the row count 
SET NOCOUNT ON
BEGIN
-- Assign the query result to @RideHrsOut
SELECT
	@RideHrsOut = SUM(DATEDIFF(second, StartDate, EndDate))/3600
FROM CapitalBikeShare
-- Cast StartDate as date and compare with @DateParm
WHERE CAST(StartDate AS date) = @DateParm
RETURN
END

--creating a table and using the stored proceudre 
-- Create SPResults
DECLARE @SPResults TABLE(
  	-- Create Weekday
	Weekday 		nvarchar(30),
    -- Create Borough
	Borough 		nvarchar(30),
    -- Create AvgFarePerKM
	AvgFarePerKM 	nvarchar(30),
    -- Create RideCount
	RideCount		nvarchar(30),
    -- Create TotalRideMin
	TotalRideMin	nvarchar(30))

-- Insert the results into @SPResults
INSERT INTO @SPResults
-- Execute the SP
EXEC dbo.cuspBoroughRideStats

-- Select all the records from @SPresults 
SELECT * 
FROM @SPResults;
```

View

a temporary table that has no memory. a view is??**a virtual table based on the result-set of an SQL statement**. A view contains rows and columns, just like a real table. The fields in a view are fields from one or more real tables in the database

modifying the underlying query that makes the view

```sql
CREATE OR REPLACE VIEW view_name AS new_query
```

replacing views

- the new query must generate same column names, order and data types as the old query
    - new columns can be added at the end.

granting and revoking access to a view

```sql
GRANT priviledge(s) or REVOKE priviledge(s)
ON object
TO role or FROM role
--privileges: SELECT, INSERT, UPDATE, DELETE

-examples
GRANT UPDATE ON ratings TO PUBLIC;
REVOKE INSERT ON fils FROM ub_users;
-- Revoke everyone's update and insert privileges
REVOKE UPDATE, INSERT ON long_reviews FROM PUBLIC; 

-- Grant editor update and insert privileges 
GRANT UPDATE, INSERT ON long_reviews TO editor;

```

Dropping views

```sql
DROP VIEW view_name[CASCADE| RESTRICT];
```

-RESTRICT (default) returns an error if there are objects that depend on the view

-CASCADE: drops view and any object that depends on that view

Materialized views

physically stores  results, not the query 

great for long execution time, but not great for data being updated often

useful for data warehouses 

One key difference is that we can refresh materialized views, while no such concept exists for non-materialized views.

```sql
CREATE MATERIALIZED VIEW my_mv AS 
SELECT*
FROM existing_table'

REFRESH MATERIALIZED VIEW my_mv;
```

XACT_ABORT

specifies whether the current transaction will be automatically rolled back when an error occurs

can be used with throw and raiserror

```sql
SET XACT_ABORT {ON|OFF}

--on = if there is an error, rollback the trans and aborts the execution
--off = (default) error, there can be an open trans
```

XACT_STATE

checks if there is an open transaction

0 ??? no open transaction

1 ??? open and committable trans 

-1 ??? open, uncomitt trans 

```sql
XACT_STATE()
--takes no parameters
--cant committ
--cant rollback to a savepoint
--can rollback the full transaction
```