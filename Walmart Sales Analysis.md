# Walmart Sales Analysis 

This project is a replication of [Walmart Sales Analysis by Princekrampah](https://github.com/Princekrampah/WalmartSalesAnalysis/tree/master)  

The project aims to understand which products are performing well, which stores and which regions are good at sales, sales trend of different products, customer behavior. The purpose of this project is mainly to understand different factors that affect sales data. This would enable us to gain insights and improve and optimize our sales decision.  

This is just one way to explore data set and see where we can find relationships between variables. The whole purpose of this exercise is to explore dataset in SQL 

```SQL 

CREATE DATABASE walmartsales 

CREATE TABLE Sales (
		invoice_id VARCHAR(30) NOT NULL PRIMARY KEY,
		branch VARCHAR(5),
		city VARCHAR(30),
		customer_type VARCHAR(30),
		gender VARCHAR(30),
		product_line VARCHAR(100),
		unit_price DECIMAL(10,2),
		quantity INT,
		tax_pct numeric(6,4),
		total DECIMAL(12, 4),
        date date ,
        time TIME ,
        payment VARCHAR(15),
        cogs DECIMAL(10,2) ,
        gross_margin_pct numeric,
        gross_income DECIMAL(12, 4),
        rating numeric
);
```
We have created the database and imported csv file into the database. 
Just a caveat we are using postgreSQL to do data transformation and data analysis.
We have used pgadmin4 in interacting with postgreSQL. 

On to the analysis 

We will do three kinds of data analysis 

1) Product Analysis 
   We will be conducting analysis to understand different product lines, finding out which product lines are performing the best 

2) Sales Analysis   
   We will analyse the sales trend of the products. We can see which region and stores are performing well. This can result into finding the effectiveness of each stores' sale strategy 
   
3) Customer Analysis 
	This analysis aims to uncover the different customer segments, purchase trends, and the profitability of each customer segment. 


Firstly, we did some data wrangling and find out there are no missing values and the data is accurate and relevant for our analysis 

Secondly, we did some feature engineering or in another words, generated new variables that would help us in our analysis 

We will make the following new columns beffore we start our analysis 
1) time of the day
2) day name 
3) month name 

This is important to understand which part of the day has more sales and does weekdays or weekends have a different sales trend. Also, months shows the seasonality of certain products. This helps us in estimating the demand for different products in different seasons. 

Firstly, time of the day is generated 

``` SQL 

SELECT time,(
CASE WHEN time BETWEEN '00:00:00' AND '12:00:00' THEN 'Morning'
	 WHEN "time" BETWEEN '12:01:00' AND '16:00:00' THEN 'Afternoon'
	 WHEN "time" BETWEEN '16:01:00' AND '20:00:00' THEN 'Evening'
	 ELSE 'Night'
	 END) AS time_of_day
FROM sales;

ALTER TABLE sales
ADD COLUMN time_of_day VARCHAR (20) 

UPDATE sales 
SET time_of_day = (
CASE WHEN time BETWEEN '00:00:00' AND '12:00:00' THEN 'Morning'
	 WHEN "time" BETWEEN '12:01:00' AND '16:00:00' THEN 'Afternoon'
	 WHEN "time" BETWEEN '16:01:00' AND '20:00:00' THEN 'Evening'
	 ELSE 'Night'
	 END);

```
Secondly, day of the week is generated 


```SQL 
SELECT "date",to_char("date", 'Day') AS day_name
FROM sales;

ALTER TABLE sales
ADD COLUMN day_name VARCHAR(20);

SELECT * FROM sales

UPDATE sales 
SET day_name = to_char("date",'Day')

```
Thirdly, month of the year is generated 

``` SQL 

SELECT "date",to_char("date", 'Month') AS month_name
FROM sales;

ALTER TABLE sales
ADD COLUMN month_name VARCHAR(10);

SELECT * FROM sales

UPDATE sales 
SET month_name = to_char("date",'Month')

```

Now we will conduct exploratory data analysis to understand about the dataset and how walmart sales are performing. 


**Generic Questions**
- How many unique cities does the data have?

``` SQL 
SELECT DISTINCT city
FROM sales 
```
- In which city is each branch?
``` SQL 
SELECT DISTINCT city, branch 
FROM sales
```

These two questions tells us which city are we focusing on. So the dataset has three cities of Myanmar. The data we have is for one branch per city. 


**Product**
How many unique product lines does the data have?
```SQL
SELECT DISTINCT product_line
FROM sales 
```
What is the most common payment method?
``` SQL 
SELECT payment, COUNT(payment) AS payment_count
FROM sales
GROUP BY payment
ORDER BY payment_count DESC

```
| Payment Method | Count |
|-----------|---------|
|Ewallet| 345|
|Cash| 344|
|Credit Card|	311|

This shows that Ewallet and Cash are more common payment methods in Myanmar. 

What is the most selling product line?

```SQL 
SELECT SUM(quantity) AS Qty,
product_line
FROM sales 
GROUP BY product_line
ORDER BY qty DESC
```

Below shows the result of the query and it shows that electronic accessories are the most selling product line 

| Product Line | Quantity |
|-----------|---------|
|Electronic accessories| 971|
|Food and beverages| 952|
|Sports and travel|	920|
|Home and lifestyle| 911|
|Fashion accessories| 902|
|Health and beauty| 854|

What is the total revenue by month?
```SQL 
SELECT month_name AS Month,
ROUND(SUM(total)) AS total_revenue
FROM sales
GROUP BY month_name
ORDER BY total_revenue DESC
```
January saw the highest revenue but needs to cogs to really understand the profitability of the business 

| Month | Total Revenue |
|-----------|---------|
|January| 116292|
|March| 109456|
|February|	97219|


What month had the largest COGS?
``` SQL 
SELECT month_name AS Month,
ROUND(SUM(cogs)) AS total_COGS
FROM sales
GROUP BY month_name
ORDER BY total_COGS DESC
```
| Month | Cost of Goods Sold  |
|-----------|---------|
|January| 110754|
|March| 104243|
|February|	92590|

As a norm for retail industry, profit margins are low and it is the volume that matters. We can see that profits are low for the months we have data. 

Gross Profit is equal to Revenue minus cost of goods sold 

| Month | Profits  |
|-----------|---------|
|January| 5538|
|March| 5213|
|February|4629|


What product line had the largest revenue?
``` SQL 
SELECT
	product_line,
	ROUND(SUM(total)) as total_revenue
FROM sales
GROUP BY product_line
ORDER BY total_revenue DESC;
``` 
| Product Line | Total Revenue |
|-----------|---------|
|Food and beverages| 56145|
|Sports and travel| 55123|
|Electronic accessories |54338|
| Fashion accessories | 54306|
|Home and lifestyle| 53862|
|Health and beauty| 49194|

What product line had the largest COGS?
```SQL 
SELECT
	product_line,
	ROUND(SUM(cogs)) as total_cost
FROM sales
GROUP BY product_line
ORDER BY total_cost DESC;
``` 
| Product Line | Cost of good sold  |
|-----------|---------|
|Food and beverages| 53471|
|Sports and travel| 52498|
|Electronic accessories |51750|
| Fashion accessories | 51720|
|Home and lifestyle| 51297|
|Health and beauty| 46851|

So now we look at gross profitability with respect to product line
| Product Line | Profit  |
|-----------|---------|
|Food and beverages| 2674|
|Sports and travel| 2625|
|Electronic accessories |2588|
| Fashion accessories | 2586|
|Home and lifestyle| 2565|
|Health and beauty| 2343|

What is the city with the largest revenue?

```SQL
SELECT
	city,
	ROUND(SUM(total)) as total_revenue
FROM sales
GROUP BY city
ORDER BY total_revenue DESC;
```
| City| total revenue  |
|-----------|---------|
|Naypyitaw| 110569|
|Yangon| 106200|
|Mandalay|	106198|

What is the city with the largest cost of good sold?
```SQL 
SELECT
	city,
	ROUND(SUM(cogs)) as total_cogs
FROM sales
GROUP BY city
ORDER BY total_cogs DESC;
```
| City| total costs  |
|-----------|---------|
|Naypyitaw| 105304|
|Yangon| 101143|
|Mandalay|	101141|

We will also look into city-wise profits 

| City| Profits  |
|-----------|---------|
|Naypyitaw| 5265|
|Yangon| 5057|
|Mandalay|5057|

What product line had average VAT?
```SQL 
SELECT product_line, ROUND(AVG(tax_pct),2) as avg_tax
FROM sales
GROUP BY product_line
ORDER BY avg_tax DESC;
```
| Product_line| Average_tax  |
|-----------|---------|
|Home and lifestyle| 16.03|
|Sports and travel| 15.81|
|Health and beauty|15.41|
|Food and beverages|15.37|
|Electronic accessories|15.22|
|Fashion accessories|14.53|

What product line had largest VAT ? 
```SQL 
SELECT product_line, ROUND(AVG(tax_pct),2) as avg_tax_pct
FROM sales
GROUP BY product_line
ORDER BY avg_tax_pct DESC

```

| Product_line| Total_tax  |
|-----------|---------|
|Home and lifestyle|2674|
|Sports and travel| 2625|
|Electronic accessories|2588|
|Fashion accessories|2586|
|Home and lifestyle|2565|
|Health and beauty|2343|

Tax analysis shows that although avg tax for sports and travel is greater than food and beverages but food and beverages are consumed more hence their total tax is the highest 

What is the most common product line by gender?
|Gender| Product_line| Count  |
|-----------|---------|------|
|Female|Fashion accessories|96|
|Female|Food and beverages|	90|
|Female|Sports and travel|	88|
|Male|Health and beauty|	88|
|Male|Electronic accessories|	86|
|Male|	Food and beverages|	84|
|Female|Electronic accessories|	84|
|Male|Fashion accessories|	82|
|Male|	Home and lifestyle|	81|
|Female|Home and lifestyle|	79|
|Male|Sports and travel|	78|
|Female|Health and beauty|	64|

This shows that female use fashion accessories more than men and ELectronic accessories are used by men more than women but there is not much difference 


What is the average rating of each product line?

```SQL
SELECT product_line, ROUND (AVG (rating),2) AS average_rating
FROM sales
GROUP BY  product_line
ORDER BY average_rating DESC;
```

| Product_line| average_rating  |
|---------|---------|
|Food and beverages|	7.11|
|Fashion accessories|	7.03|
|Health and beauty|	7.00|
|Electronic accessories|	6.92|
|Sports and travel|	6.92|
|Home and lifestyle|	6.84|


**Sales**
What is the total product sold at every branch?
```SQL 
SELECT branch, SUM(quantity) AS qty
FROM sales
GROUP BY branch
ORDER BY qty DESC
```
|Branch|Qty|
|-----|-----|
|A|	1859|
|C|	1831|
|B|	1820|

Number of sales made in each time of the day per weekday?
```SQL 
SELECT
	time_of_day,
	COUNT(quantity) AS total_sales
FROM sales
WHERE REPLACE(day_name, ' ', '') NOT IN ('Saturday','Sunday') 
GROUP BY time_of_day 
ORDER BY total_sales DESC;
```
It is important to note here that somehow in my data, day_name column had spaces and hence it was not showing data but using replace command gave me the desired results

|time_of_day|Qty|
|-----|-----|
|Afternoon|	269|
|Evening|	241|
|Morning|	141|
|Night|		52|

Another thing that is important is that most of the sales are on weekdays rather than weekends



Which of the customer types brings the most revenue?
```SQL 
SELECT customer_type, ROUND(SUM(total)) as total_sales
FROM sales
GROUP BY customer_type 
ORDER BY total_sales DESC
```
|customer_type |total_sales|
|-----|-----|
|Member|	164223|
|Normal|	158743|

As memebers bring more revenue, we need to focus on targeting them and increasing members 

Which city has the largest tax percent/ VAT (Value Added Tax)?
```SQL
SELECT
	city,
    ROUND(AVG(tax_pct), 2) AS avg_tax_pct
FROM sales
GROUP BY city 
ORDER BY avg_tax_pct DESC;
```
|city |avg_tax_pct|
|-----|-----|
|Naypyitaw|	16.05|
|Mandalay|	15.23|
|Yangon|	14.87|

Which customer type pays the most in VAT?
```SQL 
SELECT customer_type, ROUND(AVG(tax_pct),2) as avg_tax_pct
FROM sales
GROUP BY customer_type 
ORDER BY avg_tax_pct DESC
```
|customer_type |avg_tax_pct|
|-----|-----|
|Member|	15.61|
|Normal|	15.15|

Members on average give more tax but it can be due to what product line they are buying more 

**Customer**
How many unique customer types does the data have?
```SQL 
SELECT DISTINCT customer_types
FROM sales 
```
|customer_type |
|-----|
|Member|	
|Normal|	


How many unique payment methods does the data have?
```SQL 
SELECT DISTINCT payment 
FROM sales  
```
|Payment methods |
|-----|
|Ewallet|	
|Credit Card|
|Cash|


What is the most common customer type?
```SQL
SELECT customer_type, COUNT(*) AS count
FROM sales  
GROUP BY customer_type
ORDER BY count DESC
```
|customer_type |Count|
|-----|-----|
|Member|	501|
|Normal|	499|

Which customer type buys the most?
```SQL
SELECT customer_type, SUM(quantity) AS count
FROM sales  
GROUP BY customer_type
ORDER BY count DESC
```
|customer_type |Quantity Bought|
|-----|-----|
|Member|	2785|
|Normal|	2725|

What is the gender of most of the customers?
```SQL 
SELECT gender, COUNT(*) AS count
FROM sales  
GROUP BY gender
ORDER BY count DESC
```
|customer_type |Count|
|-----|-----|
|Female|	501|
|Male|	499|

What is the gender distribution per branch?
```SQL 
SELECT gender, branch,  COUNT(*) AS count
FROM sales  
GROUP BY gender, branch 
ORDER BY count DESC
```
|Gender|Branch|Count|
|------|------|-----|
|Male|	A|	179|
|Female|	C	|178|
|Male|	B|	170|
|Female|	B|	162|
|Female|	A	|161|
|Male|	C	|150|


Which time of the day do customers give most ratings?

As mostly people bought in the afternoon or evening hence most of the rating came in afternoon or evening 
```SQL 
SELECT time_of_day,COUNT(rating) AS count_rating
FROM sales
GROUP BY time_of_day
ORDER BY count_rating DESC;
```
|Time of the day| Count|
|---------|----------|
|Afternoon|	377|
|Evening|	358|
|Morning|	191|
|Night|	74|



Which time of the day do customers give most ratings per branch?
```SQL
SELECT branch, time_of_day,COUNT(rating) AS count_rating
FROM sales
GROUP BY branch, time_of_day
ORDER BY count_rating DESC;
```
Branch|Time of the day| Count|
|-------|---------|----------|
|A|	Afternoon|	126|
|C|	Afternoon|	126|
|B| Afternoon|	125|
|B|	Evening|	123|
|A|	Evening|	119|
|C|	Evening|	116|
|A| Morning|	73|
|B|	Morning|	59|
|C|	Morning|	59|
|C|	Night|	27|

Which day of the week has the best avg ratings?

```SQL 
SELECT day_name,ROUND(AVG(rating),2) AS average_rating
FROM sales
GROUP BY day_name
ORDER BY average_rating DESC;
```
Time of the day| Average_Rating|
|---------|----------|
|Monday|	7.15|
|Friday|	7.08|
|Sunday|	7.01|
|Tuesday|	7.00|
|Saturday|	6.90|
|Thursday|	6.89|
|Wednesday|	6.81|

Which day of the week has the best average ratings per branch?

```SQL 
SELECT branch,day_name, ROUND(AVG(rating),2) AS average_rating
FROM sales
GROUP BY branch, day_name
ORDER BY average_rating DESC;
```
Branch|Time of the day| Average_rating|
|-------|---------|----------|
|B|	Monday|	7.34|
|A|	Friday|	7.31|
|C|	Friday|	7.28|
|C|	Saturday|	7.23|
|A|	Monday|	7.10|
|A|	Sunday|	7.08|
|C|	Wednesday| 7.06|
|A|	Tuesday|	7.06|
|C|	Monday|7.04|
|C|	Sunday|	7.03|
|B|	Tuesday|	7.00|
|A|	Thursday|	6.96|
|C|	Tuesday|	6.95|
|C|	Thursday|	6.95|
|A|	Wednesday|	6.92|
|B|	Sunday|	6.89|
|B|	Thursday|	6.75|
|A|	Saturday|	6.75|
|B|	Saturday|	6.74|
|B|	Friday|	6.69|
|B|	Wednesday|	6.45|
