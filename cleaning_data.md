What issues will you address by cleaning the data?

- Removing irrelevatant data
- Removing duplicated data by creating CTE's
- Creating one or two unique key column for CTE
- Changing datatypes to fit the coloumn constraint
- Changing any unavailable or null data into N/A
- Changing values to make coloumns consistent

Queries:

CTE's were used in every query. One or multiple CTE's were used depending on the question.

```
WITH transactions AS
          (SELECT   visitid,
			              LPAD(fullvisitorid::varchar, 19, '0') as visitorid,
			              totaltransactionrevenue/1000000 as revenue,
			 	            CASE 
			              WHEN city LIKE '%not available%' THEN 'N/A'
			              ELSE city
			              END,
			              country,
			              CAST(date::varchar as DATE)
		    	FROM      all_sessions
		    	WHERE     totaltransactionrevenue IS NOT NULL
			    GROUP BY  fullvisitorid, visitid, totaltransactionrevenue, city, country,
                    CAST(date::varchar as DATE)
			    ORDER BY  visitid)
```

```
WITH transaction_details AS -- Productsku per Transaction
          (SELECT    visitid,
                     RPAD(productsku, 14, '0') as productsku
            FROM     all_sessions
						WHERE    totaltransactionrevenue > 0)
```

```
WITH Products AS  
          (SELECT   RPAD(productsku, 14, '0') as productsku,
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
			              END
            FROM  (  -- Subquery to Remove Duplicate Categories
                      SELECT    productsku,
                                v2productname as name,
                                v2productcategory as category,
                                DENSE_RANK() OVER (PARTITION BY productsku
                                ORDER BY v2productcategory DESC) as dups -- ranking duplicates to remove
                      FROM      all_sessions
                      WHERE     totaltransactionrevenue > 0
                      GROUP BY  productsku, v2productname, v2productcategory)
            WHERE dups = 1 -- removing duplicate productsku
                    )
```

```
WITH visitors AS
          (SELECT   fullvisitorid,
                    city,
                    country
          FROM  (
                  SELECT   fullvisitorid, city, country,
                            DENSE_RANK() OVER (PARTITION BY fullvisitorid ORDER BY date DESC, time DESC) as rank
                  FROM      all_sessions
                  GROUP BY  fullvisitorid, city, country, date, time)
          WHERE    rank = 1)
```











