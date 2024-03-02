### All queries in these questions have this CTE above (more details in cleaning_data.md):


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



## Question 1: What countries and cities are most visitors from?

### SQL Queries:

```
SELECT country, COUNT(visitorid) AS visitorcount
FROM visitors 
GROUP BY country
ORDER BY visitorcount DESC
```

```
SELECT city, COUNT(visitorid) AS visitorcount
FROM visitors 
GROUP BY city
ORDER BY visitorcount DESC
```

### Answer: 

The top 3 countries that visitors are from are the United States with 8,117 visitors, India with 693 visitors and the United Kingdom with 641 visitors. Canada, Germany, Japan, Austrailia and France all have over 200 visitors.

Excluding unidentify cities, the top 3 cities with most visitors are from Mountain View with 1,085 visitors, New York with 603 visitors and San Francisco with 432 visitors. Sunnyvale, San Jose and Los Angeles all have over 200 visitors.



## Question 2: What years and months have made the most revenue?

### SQL Queries:

```
SELECT EXTRACT(YEAR FROM date) AS year,
		SUM(revenue) AS sumrevenue
FROM transactions
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY sumrevenue DESC
```

```
SELECT EXTRACT(MONTH FROM date) as month,
		SUM(revenue) AS sumrevenue
FROM transactions
GROUP BY EXTRACT(MONTH FROM date)
ORDER BY sumrevenue DESC
```

### Answer:

2017 made the most revenue with $9,705, where as 2016 only made $4,419.

March has the most revenue with $2,860, following December with $2,551 and January with $2,241.

## Question 3: What months and years has the most sales?

### SQL Queries:

```
SELECT EXTRACT(YEAR FROM date) as year,
		COUNT(revenue) AS salesamount
FROM transactions
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY salesamount DESC
```

```
SELECT EXTRACT(MONTH FROM date) as month,
		COUNT(revenue) AS salesamount
FROM transactions
GROUP BY EXTRACT(MONTH FROM date)
ORDER BY salesamount DESC
```

### Answer:

2017 had the most sales with 55, having more than double than 2016 with 25 sales.

May had the most sales of 12 of any month. While March had 11 sales. Both January and December had 10 sales.


## Question 4: What months and years have the higher average of revenue per sale? 

### SQL Queries:

```
SELECT EXTRACT(YEAR FROM date) as year,
		AVG(revenue) AS avgrevenue
FROM transactions
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY avgrevenue DESC
```

```
SELECT EXTRACT(MONTH FROM date) as month,
		AVG(revenue) AS avgrevenue
FROM transactions
GROUP BY EXTRACT(MONTH FROM date)
ORDER BY avgrevenue DESC
```

### Answer:

2016 has the higher average with $176.76. While 2017 has an average of $176.45 per sale.

November has the highest average of $480.67. July comes in second with a $305.25 average per sale. While March comes in third with a $260 average per sale.



## Question 5: What is the best selling product/category by year/month?

### SQL Queries:

```
SELECT year, name, productcount
FROM( -- Subquery to get the best selling
SELECT EXTRACT(YEAR FROM date) as year,
		name,
		COUNT(name) AS productcount,
		DENSE_RANK() OVER (PARTITION BY EXTRACT(YEAR FROM date)  ORDER BY COUNT(name) DESC)
FROM transactions
JOIN transaction_details
USING(visitid)
JOIN products
USING (productsku)
GROUP BY EXTRACT(YEAR FROM date), name
ORDER BY productcount DESC)
WHERE dense_rank = 1
```

```
SELECT year, category, productcount
FROM( -- Subquery to get the best selling
SELECT EXTRACT(YEAR FROM date) as year,
		category,
		COUNT(category) AS productcount,
		DENSE_RANK() OVER (PARTITION BY EXTRACT(YEAR FROM date)  ORDER BY COUNT(category) DESC)
FROM transactions
JOIN transaction_details
USING(visitid)
JOIN products
USING (productsku)
GROUP BY EXTRACT(YEAR FROM date), category
ORDER BY productcount DESC)
WHERE dense_rank = 1
```

```
SELECT month, name, productcount
FROM( -- Subquery to get the best selling
SELECT EXTRACT(MONTH FROM date) as month,
		name,
		COUNT(name) AS productcount,
		DENSE_RANK() OVER (PARTITION BY EXTRACT(MONTH FROM date)  ORDER BY COUNT(name) DESC)
FROM transactions
JOIN transaction_details
USING(visitid)
JOIN products
USING (productsku)
GROUP BY EXTRACT(MONTH FROM date), name
ORDER BY productcount DESC)
WHERE dense_rank = 1
```

```
SELECT month, category, productcount
FROM( -- Subquery to get the best selling
SELECT EXTRACT(MONTH FROM date) as month,
		category,
		COUNT(category) AS productcount,
		DENSE_RANK() OVER (PARTITION BY EXTRACT(MONTH FROM date)  ORDER BY COUNT(category) DESC)
FROM transactions
JOIN transaction_details
USING(visitid)
JOIN products
USING (productsku)
GROUP BY EXTRACT(MONTH FROM date), category
ORDER BY productcount DESC)
WHERE dense_rank = 1
```

### Answer:

In 2017 the best selling category is NEST with 24 sales, while in 2016 it is Apparel with 12 sales. The best selling product in both 2016 and 2017 is the Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel. 2017 also shared 2 other best selling products which are the Nest® Cam Outdoor Security Camera - USA and the Nest® Cam Indoor Security Camera - USA.

The best selling product in both December and June is the Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel with 3 and 2 sales. The Nest® Cam Outdoor Security Camera - USA had the most sales in both January and March with 2 sales. January also shared the best selling product with the Nest® Protect Smoke + CO Black Wired Alarm-USA. March as well had 2 other best selling being the Nest® Cam Indoor Security Camera - USA and the Nest® Learning Thermostat 3rd Gen-USA. The rest of the months only had one sale per product.


