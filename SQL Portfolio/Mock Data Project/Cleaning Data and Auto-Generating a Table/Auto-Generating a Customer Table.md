
```sql
TRUNCATE TABLE customers
SET NOCOUNT OFF

DECLARE @counter INT = 0

WHILE @counter < 100

	BEGIN

		INSERT INTO Customers (
						FirstName, 
						Gender, 
						LastName, 
						MiddleInitial, 
						Age,
						Email	)
		SELECT
			FirstNames.name,
			FirstNames.sex,
			(SELECT TOP 1 name FROM LastNames ORDER BY NEWID()) AS LastName,
			(SELECT TOP 1 LEFT(name,1) FROM FirstNames ORDER BY NEWID()) AS MiddleInitial,
			(SELECT FLOOR(18 + RAND() * (75 - 18 + 1))),
			CONCAT(FirstNames.name, FLOOR(RAND() * 10), FLOOR(RAND() * 10), FLOOR(RAND() * 10), CASE 
																									WHEN RAND() > .5 THEN '@gmail.com'
																									ELSE '@yahoo.com'
																								END)
		FROM 
			(SELECT TOP 1 name, sex FROM FirstNames ORDER BY NEWID()) AS FirstNames;

		SET @counter += 1

	END

SELECT TOP(@counter)*
FROM	Customers
ORDER BY CustomerID DESC;
```
