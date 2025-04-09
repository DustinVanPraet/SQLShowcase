#### In a new database, import FirstNames.txt and LastNames.txt as flat files. After importing files, running the script below will generate a customer table. 
```sql
--Run this after importing flat table files to auto generate a customers table.
--Change the @recordsToMake variable to desired number of customers to generate.
DECLARE @recordsToMake INT = 1000;

--BEGIN	TRANSACTION; --Still have to add closing TRANS & TRY
--BEGIN 	TRY

-- Fix Gender descriptions
update FirstNames
set		sex = 'M'
where	sex = 'boy';

update	FirstNames
set		sex = 'F'
where	sex = 'girl';

--Clean data by converting percent column cells that are displayed as Scientific Notation
--Filtered by where LIKE '%E%' since its distinctive to all scientific notiation
UPDATE FirstNames
SET [percent] = CAST(TRY_CAST([percent] AS FLOAT) * 100 AS DECIMAL(20,20))
WHERE [percent] LIKE '%E%'

--Create a new table removing duplicate names and re-calculating AVG percent over the years for a name. 
CREATE TABLE newFirstNames
(
	name NVARCHAR(50), 
	sex NVARCHAR(50),
	[percent] DECIMAL(20,20)
);
-- Create cte with new data that will be going into new table. 
WITH cte 
AS 
(
	SELECT 
		name, 
		sex, 
		AVG([percent]) AS [percent]
	FROM FirstNames
	GROUP BY 
		name, 
		sex
)
--Insert cte into new Table 
	INSERT INTO newFirstNames (name, sex, [percent])
	SELECT	cte.name,
			cte.sex,
			cte.[percent]
	FROM	cte;
--Drop old table
	DROP TABLE FirstNames;
--Rename new table to reflect old table. 
	EXEC sp_rename 'newFirstNames', 'FirstNames';

--Minimize LastNames into a smaller table 
SELECT TOP(10000)
	LastNames.name
INTO LastNamesOnly
FROM LastNames

--Delete LastNames Table
DELETE LastNames;
DROP TABLE LastNames;

--Rename Table
EXEC sp_rename 'LastNamesOnly', 'LastNames';

--Update lastNames for capitalization
update LastNames
set		name = UPPER(LEFT(name,1)) + LOWER(SUBSTRING(name, 2, len(name)))

--Create Customers table
CREATE TABLE Customers
(
	CustomerID INT IDENTITY(1000,1) 
		PRIMARY KEY,
	FirstName VARCHAR(30) 
		NOT NULL,
	MiddleInitial CHAR(1),
	LastName VARCHAR(30) 
		NOT NULL,
	Age TINYINT CHECK  (Age > 0 AND Age <= 100)
		NOT NULL,
	Gender CHAR(1) CHECK (Gender IN ('M','F')) 
		NOT NULL,
	Email VARCHAR(50) CHECK (Email LIKE '%@%.%')	
		NOT NULL
)

--Insert 100k rows of random data into Customers table
DECLARE @counter INT = 0

WHILE @counter < @recordsToMake

	BEGIN

		INSERT INTO Customers (
					FirstName, 
					Gender, 
					LastName, 
					MiddleInitial, 
					Age
					,Email
					)
		SELECT
			FirstNames.name,
			FirstNames.sex,
			(SELECT TOP 1 name FROM LastNames ORDER BY NEWID()) AS LastName,
			(SELECT TOP 1 LEFT(name,1) FROM FirstNames ORDER BY NEWID()) AS MiddleInitial,
			(SELECT FLOOR(18 + RAND() * (75 - 18 + 1))),
			CONCAT(FirstNames.name, FLOOR(RAND() * 10), FLOOR(RAND() * 10), FLOOR(RAND() * 10),
				CASE 	WHEN RAND() > .5 THEN '@gmail.com'
				ELSE '@yahoo.com'
				END)
		FROM 
			(SELECT TOP 1 name, sex FROM FirstNames ORDER BY NEWID()) AS FirstNames;

		SET @counter += 1

	END

SELECT *
FROM	Customers;```
