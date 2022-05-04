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
```

Upsampling

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
--returns a string value
SELECT DATENAME(MONTH,@dt) AS theMonth;
```

DATEFROMPARTS

```sql
DATEFROMPARTS(2019,3,5) as new_date;

-- Create a date as the start of the month of the first vote
	DATEFROMPARTS(YEAR(first_vote_date), MONTH(first_vote_date), 1) AS first_vote_starting_month
```

DATEPART

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
--obtain the difference bwt two datetime values, always returns a number, and rounds up
DATEDIFF(datepart, startdate, endate
SELECT DATEDIFF(DD, '2020-05-22', '2020-06-21')
--output will. be 30
```

SQL Server does not have an intuitive way to round down to the month, hour, or minute. You can, however, combine the `DATEADD()` and `DATEDIFF()` functions to perform this rounding.

To round the date 1914-08-16 down to the year, we would call `DATEADD(YEAR, DATEDIFF(YEAR, 0, '1914-08-16'), 0)`. To round that date down to the month, we would call `DATEADD(MONTH, DATEDIFF(MONTH, 0, '1914-08-16'), 0)`

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

function to change the time zone of a `DATETIME`
, `DATETIME2`
, or `DATETIMEOFFSET`
 typed date or a valid date string

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

Error-handling 

-safe versions

-doest the job with a NULL

TRY_PARSE

TRY_CONVERT

TRY_CAST

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
```

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

The GROUPING SETS option **gives you the ability to combine multiple GROUP BY clauses into one GROUP BY clause.** The results are the equivalent of UNION ALL of the specified groups.

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

WITH ROLLUP

comes after GROUP BY 

used for non-measure attribute hierarchal data, by taking the combo of each column follow by each matching value 

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

Temporary Tables

using a # to create a temp table 


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

The first row in a window where the `LAG()` function is used is NULL.

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