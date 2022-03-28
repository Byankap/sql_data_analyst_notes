
FROM

the source of the location of tables

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
--Caluclate total amount
SELECT
    SUM(column1)

```

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
