### All queries in these questions have this CTE above (more details in cleaning_data.md):



```
WITH transactions AS -- information on transactions
	(
	SELECT	   	visitid,
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
	SELECT	fullvisitorid AS visitorid,
		city,
		country
	FROM
		( -- subquery to remove duplicate visitors
		SELECT		fullvisitorid,
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



    
## **Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


### SQL Queries: 

```
SELECT		city,
            	SUM(revenue) as totalrevenue
FROM        	transactions
WHERE		city != 'N/A' -- removing invalid cities
GROUP BY    	city
ORDER BY	SUM(revenue) DESC
LIMIT 3
```

```
SELECT		country,
		SUM(revenue) as totalrevenue
FROM 		transactions
GROUP BY 	country
ORDER BY	SUM(revenue) DESC
LIMIT 3
```

### Answer:

The top 3 countries that have the highest level of revenue are The United States at $12,999, Israel at $602 and Austrailia at $358.

While top 3 cities excluding unidentified cities is San Francisco at $1,561, Sunnyvale at $991 and Atlanta at $853.




## **Question 2: What is the average number of products ordered from visitors in each city and country?**


### SQL Queries:

```

SELECT 		country,
		AVG(productcount) AS avgproduct
FROM
	( -- subquery to count products per visitor
	SELECT		visitorid,
			city,
			country,
			COUNT(productsku) AS productcount
	FROM		transaction_details
	LEFT JOIN 	transactions
	USING 		(visitid)
	GROUP BY	visitorid, city, country
	)
GROUP BY	country
ORDER BY	avgproduct DESC
```

```
SELECT		city,
		AVG(productcount) AS avgproduct
FROM
	( -- subquery to count products per visitor
	SELECT		visitorid,
			city,
			country,
			COUNT(productsku) AS productcount
	FROM 		transaction_details
	LEFT JOIN 	transactions
	USING		(visitid)
	GROUP BY	visitorid, city, country
	)
GROUP BY	city
ORDER BY	avgproduct DESC
```



### Answer:

Due to the original data being extremely messy and large amounts of null values, I was unable to find a way of getting the number of products ordered per transaction without it being inaccurate or making huge assumptions on the data. Instead I was able to find the average number of distinct products being brought for each country and city.

Using this almost all countries have the average of 1, except the United States which has the average of 1.03.
With the cities almost every city has the average of 1 except the non specified cities that has a 1.0416 average.





## **Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


### SQL Queries:

```
SELECT		country,
		category,
		COUNT(category) AS productamount
FROM		transaction_details
JOIN		transactions
USING		(visitid)
JOIN		products
USING		(productsku)
GROUP BY	country, category
ORDER BY	productamount DESC
```

```
SELECT		city,
		category,
		COUNT(category) AS productamount
FROM 		transaction_details
JOIN 		transactions
USING		(visitid)
JOIN 		products
USING		(productsku)
GROUP BY 	city, category
ORDER BY 	productamount DESC
```


### Answer:

Most if not all visitors in the United States, Canada and Switzerland have brought Apparel. Where as the top category of products in Israel and Australia has been Nest. The United States has brought the most amount of Apparel with 31 separate sales. The United States has also brought the most amount of Nest products at 26 separate sales.

In cities excluding the unidentified cities, the highest product category has been Apparel. Notably Mountain View and New York both having 5 separate sales buying the producs. The second highest category has been Nest. Notably including San Francisco with 4 separate sales and Palo Alto with 3 separate sales.

Overall Apparel and Nest products have to be the most sold category of products.





## **Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


### SQL Queries:

```
SELECT	country,
	name,
	productcount
FROM
	( -- Subquery to get only top-selling product
	SELECT		country,
			name,
			count(name) AS productcount,
			DENSE_RANK() OVER (PARTITION BY country ORDER BY COUNT(name) DESC)
	FROM 		transaction_details
	JOIN		transactions
	USING		(visitid)
	JOIN 		products
	USING		(productsku)
	GROUP BY	country, name
	ORDER BY	productcount DESC
	)
WHERE	dense_rank = 1
```

```
SELECT	city,
	name,
	productcount
FROM
	(  -- Subquery to get only top-selling product
	SELECT 		city,
			name,
			count(name) AS productcount,
			DENSE_RANK() OVER (PARTITION BY city ORDER BY COUNT(name) DESC)
	FROM 		transaction_details
	JOIN 		transactions
	USING		(visitid)
	JOIN 		products
	USING		(productsku)
	GROUP BY	city, name
	ORDER BY	productcount DESC
	)
WHERE	dense_rank = 1
```


### Answer:

The top selling product in the United States is the Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel with 7 separate sales. The rest only have one product with one sale. Austrailia's top product is the Nest® Cam Indoor Security Camera - USA. Canada has a tie with the Google Men's 3/4 Sleeve Raglan Henley Grey and Google Men's  Zip Hoodie. Israel top product is the Nest® Protect Smoke + CO Black Wired Alarm-USA and Switzerland's is the YouTube Men's 3/4 Sleeve Henley.

Most cities only have 1 sale per product. The only products that have mutiple sales are the Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel and the Nest® Protect Smoke + CO White Wired Alarm-USA with 2 sales in cities that are unidentified.

Overall the product that has the most sales between counties and cities is the Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel.



## **Question 5: Can we summarize the impact of revenue generated from each city/country?**

### SQL Queries:

```
SELECT		country,
		city,
		SUM(revenue) AS totalrevenue
FROM 		transactions
GROUP BY 	country, city
HAVING		SUM(revenue) IS NOT NULL
ORDER BY	totalrevenue DESC, country, city
```

### Answer:

In summary the most revenue comes from unidenfied cities from the United States at $5,968. San Fransico also in the United States is second with $1,561 generated. Where as the least amount of revenue generated is Zurich in Switzerland with $16 generated. Columbus in the United States is second last with only $21 generated.

Overall the United States make the most revenue with San Fransico being the known best selling city. Where on the opposite scale Switzerland and Columbus make the least.







