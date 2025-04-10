```sql
--List all active products
SELECT		*
FROM		  GroceryInv
WHERE		  Status = 'Active'
```
![image](https://github.com/user-attachments/assets/fedfa60a-618e-43e6-8ec3-dad02b64e463)

```sql
-- All products below the reorder level
SELECT	Product_ID,
        Product_Name,
        Status,
        Stock_Quantity,
        Reorder_Level
FROM		GroceryInv
WHERE		Stock_Quantity < Reorder_Level
AND			[Status] != 'Discontinued'
```
![image](https://github.com/user-attachments/assets/bd903757-4cfc-4918-9b58-6b1b3ce737c7)
```sql
--Show products that expired in last 30 days
SELECT		*
FROM		  GroceryInv
WHERE		  Expiration_Date BETWEEN GETDATE() AND GETDATE() + 30
```
```sql
--Calculate how many days between Date_Received and Expiration_Date
SELECT		Product_ID, 
    			Product_Name,
    			Status,
    			Date_Received, 
    			Last_Order_Date,
    			Expiration_Date,
    			DATEDIFF(DAY, Date_Received, Expiration_Date) AS 'Days Between'
FROM		  GroceryInv
WHERE		  [Status] != 'Discontinued'
ORDER BY	[Days Between] ASC
```
![image](https://github.com/user-attachments/assets/0470020d-ef34-445f-a847-a71de5641946)
```sql
--Find AVG inventory_turnover_rate by category
SELECT		Category,
    			COUNT(*) AS 'Number In Category',
    			SUM(Inventory_Turnover_Rate) AS 'Total Turnover By Category',
    			AVG(Inventory_Turnover_Rate) AS 'Average Turnover Rate'
FROM		  GroceryInv
GROUP BY	Category
ORDER BY	[Average Turnover Rate] ASC
```
![image](https://github.com/user-attachments/assets/8d80b34b-3c5f-4c29-b4ef-e4854632561e)
```sql
--Overall Sales by category
SELECT		Category, 
			    SUM(Unit_Price*Sales_Volume) AS 'Total Sales'
FROM		  GroceryInv
GROUP BY	Category
ORDER BY	[Total Sales] DESC
```
![image](https://github.com/user-attachments/assets/a5e70993-7807-476a-8519-baed6e1bb1a5)
```sql
--Which Products had highest and lowest turnover rates
SELECT TOP(5)	Product_Name, 
  				    Inventory_Turnover_Rate, 
  				    Stock_Quantity
FROM			    GroceryInv
ORDER BY		  Inventory_Turnover_Rate DESC

SELECT TOP(5)	Product_Name, 
      				Inventory_Turnover_Rate, 
      				Stock_Quantity
FROM			    GroceryInv
ORDER BY		  Inventory_Turnover_Rate ASC
```
![image](https://github.com/user-attachments/assets/e3c3ec0b-d05d-4a6e-87d7-9f3f1b90e010)
```sql
--Find products where Reorder_Quantity is less than current Stock_Quantity
--Potential product that is overstocked
SELECT		Product_ID,
  				Product_Name,
  				Category,
  				Stock_Quantity,
  				Reorder_Quantity
FROM			GroceryInv
WHERE			Reorder_Quantity < Stock_Quantity
AND				[Status] != 'Discontinued'
```
![image](https://github.com/user-attachments/assets/0ccb2aad-f149-48e1-930d-2fef5fee8d61)
```sql
--Find product that may be understocked
SELECT			Product_ID,
    				Product_Name,
    				Category,
    				Reorder_Level,
    				Stock_Quantity,
    				SUM(Reorder_Level - Stock_Quantity) AS 'Amount Below Reorder Level',
    				Reorder_Quantity
FROM			  GroceryInv
WHERE			  Stock_Quantity < Reorder_Level
AND				  (Stock_Quantity + Reorder_Quantity) < (Reorder_Level - Stock_Quantity)
GROUP BY		Product_ID,
    				Product_Name,
    				Category,
    				Stock_Quantity,
    				Reorder_Level,
    				Reorder_Quantity
ORDER BY		[Amount Below Reorder Level] DESC
```
![image](https://github.com/user-attachments/assets/91fc5f73-efb5-47d0-bd3e-4f4744ec5844)
```sql
--Average Unit Price by Category
SELECT			Category, 
				    CAST(AVG(Unit_Price) AS DECIMAL(5,2)) AS 'Average Unit Price'
FROM			  GroceryInv
WHERE			  [Status] != 'Discontinued'
GROUP BY		Category
ORDER BY		[Average Unit Price] DESC
```
![image](https://github.com/user-attachments/assets/78f3bb25-f946-4d00-bc6b-e3d9e66bd5e8)
```sql
--Average Sales Volume and Turnorver for stronger comparison
WITH cte
AS
(
SELECT		AVG(Sales_Volume) AS 'Average Volume',
				  AVG(Inventory_Turnover_Rate) AS 'Average Turnover'
FROM			GroceryInv
WHERE			[Status] != 'Discontinued'
)

--Finding the performerance across product
SELECT	Product_Name,
				Sales_Volume,
				Inventory_Turnover_Rate,
				CASE
					WHEN Sales_Volume >= [Average Volume] AND Inventory_Turnover_Rate >= [Average Turnover] 
						THEN 'Strong Performer'
					WHEN Sales_Volume >= [Average Volume] AND Inventory_Turnover_Rate < [Average Turnover] 
						THEN 'Average Performer'
					WHEN Sales_Volume < [Average Volume] AND Inventory_Turnover_Rate >= [Average Turnover] 
						THEN 'Average Performer'
					WHEN Sales_Volume < [Average Volume] AND Inventory_Turnover_Rate < [Average Turnover] 
						THEN 'Under Performing'
				END AS 'Performance'
FROM			  GroceryInv
	CROSS JOIN	cte 
WHERE			  [Status] != 'Discontinued'
ORDER BY		(Sales_Volume + Inventory_Turnover_Rate) DESC
```
![image](https://github.com/user-attachments/assets/5f9f75aa-29bb-4a7e-af12-daccdbe88afd)
```sql
--Finding underpriced/overpriced products based on category
WITH cte
AS
(
SELECT			Category,
				    AVG(GroceryInv.Unit_Price) AS "Category's Average Price"
FROM			  GroceryInv
GROUP BY		Category
)

SELECT		Product_Name,
  				AVG(Unit_Price) AS 'Unit Price',
  				[Category's Average Price],
  				GroceryInv.Category,
  				CASE
  					WHEN Unit_Price < ([Category's Average Price]*.25) 
  						THEN 'Severely Under Category Average'
  					WHEN Unit_Price BETWEEN ([Category's Average Price]*.25) AND ([Category's Average Price]*1.25) 
  						THEN 'Within Range of Category Average'
  					ELSE 'Above Category Average'
  				END AS 'Category Price Point'
FROM			GroceryInv
	JOIN		cte ON GroceryInv.Category = cte.Category
--WHERE		Unit_Price < ([Category's Average Price]*.5)
AND				[Status] != 'Discontinued'
GROUP BY	GroceryInv.Product_Name, [Category's Average Price], GroceryInv.Category, GroceryInv.Unit_Price
```
![image](https://github.com/user-attachments/assets/5ec7e448-5a61-48cd-9549-a0547a713d58)

