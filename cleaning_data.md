## What issues will you address by cleaning the data?

- Removing irrelevatant data
- Removing duplicated data by creating CTE's
- Creating one or two unique key columns for CTE
- Changing datatypes to fit the coloumn constraint
- Changing any unavailable or null data into N/A
- Changing values to make coloumns consistent

## Queries:

CTE's were used in each query. One or multiple CTE's were used depending on the question.

```
WITH transactions AS -- information on transactions
	(
	SELECT	   	visitid,
			LPAD(fullvisitorid::VARCHAR, 19, '0') AS visitorid,
			totaltransactionrevenue/1000000 AS revenue,
			CASE 
			WHEN city LIKE '%not available%' THEN 'N/A'
			ELSE city
			END,
			country,
			CAST(date::VARCHAR as DATE)
	FROM  		all_sessions
	WHERE   	totaltransactionrevenue IS NOT NULL -- only visits that have transaction
	GROUP BY	visitid, fullvisitorid, totaltransactionrevenue, city, country,
                	CAST(date::VARCHAR as DATE) -- only unique visits
	)
```

```
WITH transaction_details AS -- productsku per transaction
	(
	SELECT	visitid,
		RPAD(productsku, 14, '0') AS productsku
	FROM   	all_sessions
	WHERE	totaltransactionrevenue > 0 -- only transactions
	)
```

```
WITH productspertrans AS  -- product information that have transactions
	(
	SELECT	RPAD(productsku, 14, '0') AS productsku,
		name,
		CASE
		WHEN category LIKE '%Shop by Brand%' THEN 'Shop by Brand'
		WHEN category LIKE '%Accessories%' THEN 'Accesories'
		WHEN category LIKE '%Apparel%' THEN 'Apparel'
		WHEN category LIKE '%Nest%' THEN 'Nest'
		WHEN category LIKE '%Bags%' THEN 'Bags'
		WHEN category LIKE '%Drinkware%' THEN 'Drinkware'
		WHEN category LIKE '%Office%' THEN 'Office'
		ELSE 'Apparel' -- Any others are both similar in productname to other products of category
		END AS category
	FROM
		(  -- Subquery to Remove Duplicate Categories
		SELECT		productsku,
				v2productname AS name,
				v2productcategory AS category,
				DENSE_RANK() OVER (PARTITION BY productsku
				ORDER BY v2productcategory DESC) as dups -- ranking duplicates to remove
		FROM    	all_sessions
		WHERE   	totaltransactionrevenue > 0
		GROUP BY	productsku, v2productname, v2productcategory -- removing duplicate products
		)
	WHERE	dups = 1
	)
```

```
WITH visitors AS -- visitor information 
	(
	SELECT	visitorid,
		city,
		country
	FROM
		( -- subquery to remove duplicate visitors
		SELECT		LPAD(fullvisitorid::VARCHAR, 19, '0') AS visitorid,
				CASE 
				WHEN city LIKE '%not available%' OR city LIKE '%not set%' THEN 'N/A'
				ELSE city
				END,
				CASE
				WHEN country LIKE '%not set%' OR country LIKE '%not available%' THEN 'N/A'
				ELSE country
				END AS country,
				DENSE_RANK() OVER (PARTITION BY fullvisitorid
				ORDER BY date DESC, time DESC) as rank -- chooses country/city based on most recent visit
		FROM		all_sessions
		GROUP BY	fullvisitorid, city, country, date, time
		)
	WHERE	rank = 1
	)
```











