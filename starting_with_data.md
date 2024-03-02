### All queries in these questions have this CTE above (more details in cleaning_data.md):



```
WITH transactions AS -- information on transactions
	(
	SELECT	    visitid,
			    LPAD(fullvisitorid::varchar, 19, '0') AS visitorid,
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
	),

    transaction_details AS -- productsku per transaction
	(
	SELECT	visitid,
		    RPAD(productsku, 14, '0') AS productsku
	FROM   	all_sessions
	WHERE	totaltransactionrevenue > 0 -- only transactions
	),

    products AS  -- product information that have transactions
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
	),

    visitors AS -- visitor information 
	(
	SELECT	fullvisitorid,
		    city,
		    country
	FROM
		( -- subquery to remove duplicate visitors
		SELECT		fullvisitorid, city, country,
				    DENSE_RANK() OVER (PARTITION BY fullvisitorid
				    ORDER BY date DESC, time DESC) as rank -- chooses country/city based on most recent visit
		FROM		all_sessions
		GROUP BY	fullvisitorid, city, country, date, time
		)
	WHERE	rank = 1
	)
 ```


## Question 1: What countries and cities are most visitors from?

### SQL Queries:

```
SELECT country, COUNT(fullvisitorid)
FROM (SELECT  fullvisitorid,
		 city, country,
 		DENSE_RANK() OVER (partition by fullvisitorid order by  date DESC, time DESC) as rank
FROM all_sessions
GROUP BY fullvisitorid, city, country, date, time)
WHERE rank = 1
GROUP BY country
ORDER BY COUNT(fullvisitorid) DESC
```

### Answer: 

The top 3 



## Question 2: Which month has the highest revenue? 

### SQL Queries:

### Answer:



## Question 3: 

### SQL Queries:

### Answer:



## Question 4: 

### SQL Queries:

### Answer:



## Question 5: 

### SQL Queries:

### Answer:
