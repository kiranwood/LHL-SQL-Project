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

### Step 1: Check for duplicate data.

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

None of the View's have any duplicates.

### Step 2: Check for Nulls

```
SELECT *
FROM transactions
WHERE visitid IS NULL
OR visitorid IS NULL
OR revenue IS NULL
OR city IS NULL
OR country IS NULL
OR date IS NULL
```

```
SELECT *
FROM transaction_details
WHERE visitid IS NULL
OR productsku IS NULL
```

```
SELECT *
FROM productspertrans
WHERE productsku IS NULL
OR name IS NULL
OR category IS NULL
```

```
SELECT *
FROM visitors
WHERE visitorid IS NULL
OR city IS NULL
or country IS NULL
```

None of the views have any Null values.

### Step 3: Check for valid data values
