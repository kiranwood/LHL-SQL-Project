# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals

This project is focusing on extracting, transforming, cleaning, loading and analyzing data. As well as developing and running a QA process.

The goals of this project is to:
- Load data into a database
- Clean data
- Answer questions based on the data
- Create a QA process

## Process

### Step 1

Download the CSV files and create a new database. Then create tables for each file and import file. Make sure all of the columns are filled in correctly and writing queries using the table is working.

### Step 2

Check out each of the questions and keep track of which columns are important to answering the questions. Get a general idea of how to solve the questions using queries.

### Step 3

Start making CTE's that can later be turned into View's or Temporary Tables that include the columns needed to answer the questions. Make sure these CTE's are normalized, remove duplicates and make sure there are unique keys.

### Step 4

Using the CTE's make queries to answer each of the questions. Come up with 5 new questions to be able to answer using the data. Then write queries to be able to find the answer for those.

### Step 5

Clean CTE's further. Check each column for potiental data issues and transform data to fix these issues. Then format and organize CTE's to be easier to read and understand. Add comments where neccessary. After place the CTE's used into the cleaning_data.md file.

### Step 6

Using the cleaned CTE's, make sure each query for answering the question works and is also formated and organized for readability. After write down the answers to the questions by using the queries. Add the queries/answers of the original 5 questions into the starting_with_quesitions.md file. Then add the answers/queries of new 5 questions into the starting_with_date.md.

### Step 7

Identify and describe risk areas. Then make a QA process to address them. Include the queries and explanation in the QA.md file.

### Step 8

Connect the original tables together by keys and relationships. Then download the ERD as a file named schema.png.

## Results

The totaltransactionrevenue, date, city, country, visitid, productsku, v2productname, v2productcategory and fullvisitorid columns from the all_sessions table can be used to able to queries to answer the questions. By organizing and cleaning these columns into separate views it is easier to write the queries and understand the data.

By finding the best selling products/categories from countries, months, could help a business to potentially push certain products to visitors in order to gain sales. Similarly the opposite, its possible to gain insights on what products are not worth being pushed.
Understanding the sales by year could help analyze potential trends to predict future sales.
While finding the countries/cities with the most sales could help a business focus on which location or demographic they want to focus their business on.
As well finding the countries/cities from all the visitors and comparing to the countries/cities from transactions could show which places have frequent visitors but not so many sales.
All this data could help give insights that could lead to future decisions regarding sales and the business.

## Challenges 

One of the biggest challenges was being able to understand the data. Without any information on what columns mean, only the column name it was hard to make judgements and find what data was needed. The original data is extremely messy and overwhelming to figure out how to tackle the project.

## Future Goals

With more time being spent on the project, I could focus deeper on cleaning and understanding the data on a deeper level. I could compare data on transactions with all sessions data to potentially see what countries, products, etc to focus on to increase potential sales. I could also compare to see what products are gaining the least attraction.


