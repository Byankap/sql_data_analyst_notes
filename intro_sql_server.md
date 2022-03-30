Created by Microsoft

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

char, varchar, nvarchar

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

FROM

the source of the location of tables

GROUP BY

occurs after WHERE, but a WHERE clause in not necessary

split data into groups/comb of one or more values/columns 

avoids having to individual queries for filtering in WHERE, instead it will be a sum of all unique values for specific columns at once. 

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

JOIN

primary keys: uniquely identify each row

Foreign keys: key found in other tables

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

ORDER BY

```sql
-- Order in ascending order (the default)
ORDER BY column

--Order in descending 
SELECT TOP (10) column1, column2
FROM table
ORDER By column2 DESC, column1;
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

UNION & UNION ALL

combining same or different tables with the same info/structure and same data types

UNION - Duplicated rows are excluded

UPDATE

```sql
UPDATE table
SET column = value
WHERE
	--conditions

```

tip: dont forget the WHERE clause, otherwise it will update all the columns