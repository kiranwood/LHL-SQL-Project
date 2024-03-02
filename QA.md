## What are your risk areas? Identify and describe them.

- Lack of understanding the data
  - Data is very confusing, lots of missing data and not a lot of info on what each column means.
  - Can cause missing info due to the lack of knowledge on the columns
  - Required making assumptions based on the data that may not be entirely accurate
- Duplicate data
- Null Data 
- Valid data values

## QA Process:

First start with turning all the CTE's into views.

### Step 1 Check for duplicate data.

```
SELECT visitorid, COUNT(visitorid)
FROM transactions
GROUP BY visitorid
HAVING COUNT(visitorid) > 1
```

```
SELECT visitid, productsku, COUNT(visitid)
FROM transaction_details
GROUP BY visitid, productsku
HAVING COUNT(visitid) > 1
```

```
SELECT productsku, COUNT(productsku)
FROM productspertrans
GROUP BY productsku
HAVING COUNT(productsku) > 1
```

```
SELECT visitorid, COUNT(visitorid)
FROM visitors
GROUP BY visitorid
HAVING COUNT(visitorid) > 1
```

