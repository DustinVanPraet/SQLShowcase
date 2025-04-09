
```sql
-- Alter COLUMN to correct spelling
EXEC sp_rename 'GroceryInv.Catagory', 'Category', 'COLUMN'

--Remove "-" in Product_ID and Supplier_ID to convert from NVARCHAR to INT
--	Check for rows that don't follow the same format '__-___-____'
SELECT	*
FROM	GroceryInv
WHERE	Product_ID NOT LIKE '__-___-____'

SELECT	*
FROM	GroceryInv
WHERE	Supplier_ID NOT LIKE '__-___-____'

--	Run UPDATEs to remove '-' 
UPDATE	GroceryInv
SET		Product_ID = REPLACE(Product_ID, '-', '')

UPDATE	GroceryInv
SET		Supplier_ID = REPLACE(Supplier_ID, '-', '')

-- Run this to cycle through columns and TRIM them to mitigate inaccuracies 
DECLARE		@sql NVARCHAR(MAX) = '';
	
SELECT		@sql += 'UPDATE GroceryInv 
					SET ' + STRING_AGG(QUOTENAME(name) + ' = TRIM(' +
					QUOTENAME(name) + ')', ', ')
FROM		sys.all_columns
WHERE		object_id = OBJECT_ID('GroceryInv')
	AND		system_type_id IN (167, 231);

EXEC sp_executesql @sql;

-- Change data types to INT
ALTER TABLE		GroceryInv
ALTER COLUMN	Product_ID INT

ALTER TABLE		GroceryInv
ALTER COLUMN	Supplier_ID INT

-- Check for duplicate Product_ID. Zero Duplicates.
SELECT		Product_ID, COUNT(Product_ID) AS cnt
FROM		GroceryInv
GROUP BY	Product_ID
HAVING		COUNT(Product_ID) > 1
ORDER BY	cnt DESC

-- Check for duplicate Supplier_ID. Zero Duplicates.
SELECT		Supplier_ID, COUNT(Supplier_ID) AS cnt
FROM		GroceryInv
GROUP BY	Supplier_ID
HAVING		COUNT(Supplier_ID) > 1
ORDER BY	cnt DESC

-- Run this first SELECT query to create a SELECT query for finding NULLs
-- This could be done manually but this is more scalable for when 100s of columns possibly exist.
SELECT 'SELECT * FROM GroceryInv WHERE ' + 
		STRING_AGG(name + ' IS NULL', ' OR ')
FROM	sys.all_columns
WHERE	object_id = OBJECT_ID('GroceryInv')

-- Copy and paste the result from running the previous SELECT query. 
SELECT	* 
FROM GroceryInv 
WHERE Product_Name IS NULL 
OR Category IS NULL 
OR Supplier_Name IS NULL 
OR Warehouse_Location IS NULL 
OR Status IS NULL 
OR Product_ID IS NULL 
OR Supplier_ID IS NULL 
OR Date_Received IS NULL 
OR Last_Order_Date IS NULL 
OR Expiration_Date IS NULL 
OR Stock_Quantity IS NULL 
OR Reorder_Level IS NULL 
OR Reorder_Quantity IS NULL 
OR Unit_Price IS NULL 
OR Sales_Volume IS NULL 
OR Inventory_Turnover_Rate IS NULL 
OR percentage IS NULL

-- UPDATE single NULL value to fruits and vegetable catagory. 
UPDATE	GroceryInv
SET		Category = 'Fruits & Vegetables'
WHERE	Product_ID = 103789729

SELECT	*
FROM	GroceryInv
WHERE	Product_ID = 103789729

-- Change percentage column to a DECIMAL data type for aggregations
-- Check and compare before UPDATE
SELECT [percentage], CAST(REPLACE([percentage],'%', '') AS decimal(5,2)) / 100
from GroceryInv

UPDATE	GroceryInv
SET		[percentage] = CAST(REPLACE([percentage],'%', '') AS DECIMAL(5,2)) / 100
```
