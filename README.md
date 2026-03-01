# Apple_1M_Sales_Project


# ![Apple Logo](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/Apple_Changsha_RetailTeamMembers_09012021_big.jpg.slideshow-xlarge_2x.jpg) Apple Retail Sales SQL Project - Analyzing Millions of Sales Rows


## Project Overview

In this project, I analyzed over one million rows of Apple retail sales data, including products, stores, sales transactions, and warranty claims across global locations.

I designed and executed a series of analytical queries ranging from foundational data exploration to advanced business-driven problem solving. Using structured SQL techniques, I answered complex questions related to sales performance, regional trends, product lifecycle behavior, and warranty risk patterns.

This project demonstrates my ability to work with large-scale datasets, apply advanced SQL concepts, and translate business questions into data-driven insights.

## Entity Relationship Diagram (ERD)

![ERD](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/erd.png)

---

## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

## Objectives

The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium (10 Questions)

1. Find the number of stores in each country.
```sql
SELECT COUNT(store_id) AS total_stores,
 		country
FROM stores
GROUP BY country
ORDER BY 1 DESC; 
```
2. Calculate the total number of units sold by each store.
```sql
SELECT s.store_id,
		st.store_name,
		SUM(s.quantity) AS total_unit_sold
FROM sales as s
JOIN
stores as st
ON st.store_id = s.store_id
GROUP BY 1, 2
ORDER BY 3 DESC;
```
3. Identify how many sales occurred in December 2023.
```sql
SELECT COUNT(sale_id) AS total_sales_in_December
FROM sales
WHERE TO_CHAR(sale_date, 'MM-YYYY') = '12-2023';
```
4. Determine how many stores have never had a warranty claim filed.
```sql
SELECT COUNT(*) FROM stores
WHERE store_id NOT IN (
					SELECT
					DISTINCT store_id
					FROM sales AS s
					RIGHT JOIN warranty as w
					on s.sale_id = w.sale_id
					);
```
5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql
no claim that as wv/total claim * 100

SELECT 
		ROUND(COUNT(claim_id)/
						(SELECT COUNT(*) FROM warranty) ::numeric
				* 100,2) AS warranty_voic_percentage
FROM warranty 
WHERE repair_status = 'Warranty Void'
```
6. Identify which store had the highest total units sold in the last year.
```sql
SELECT s.store_id,
		st.store_name,
		SUM(quantity)
FROM sales AS s
JOIN stores AS st
ON s.store_id = st.store_id
WHERE sale_date >= (CURRENT_DATE - INTERVAL '1 Year')
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 1;
```
7. Count the number of unique products sold in the last year.
```sql
SELECT COUNT(DISTINCT product_id)
FROM sales
WHERE sale_date >= (CURRENT_DATE - INTERVAL '1 Year');
```
8. Find the average price of products in each category.
```sql
SELECT p.category_id,
		c.category_name,
		AVG(p.price) AS avg_price
FROM products AS p
JOIN category AS c
ON p.category_id = c.category_id
GROUP BY 1,2
ORDER BY 3 DESC;
```
9. How many warranty claims were filed in 2020?
```sql
SELECT COUNT(*) AS warranty_cliam
FROM warranty 
WHERE EXTRACT (YEAR FROM claim_date) = 2020
```
10. For each store, identify the best-selling day based on highest quantity sold.
```sql
SELECT * 
FROM
	(SELECT 
		store_id,
		TO_CHAR(sale_date, 'Day') AS day_name,
		SUM(quantity) AS total_unit_sold,
		RANK() OVER(PARTITION BY store_id ORDER BY SUM(quantity) DESC) AS RANK
	FROM sales
	GROUP BY 1, 2
	) AS t1
WHERE rank = 1;
```
### Medium to Hard (5 Questions)

11. Identify the least selling product in each country for each year based on total units sold.
```sql
WITH product_rank
AS (
SELECT 
	st.country,
	p.product_name,
	SUM(s.quantity) AS total_qty_sold,
	RANK() OVER(PARTITION BY st.country ORDER BY SUM(s.quantity)) AS rank
FROM sales AS s
JOIN
stores AS st
ON s.store_id = st.store_id
JOIN 
products AS p
ON s.product_id = p.product_id
GROUP BY 1,2
)
SELECT * 
FROM product_rank
WHERE rank = 1;
```
12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
SELECT
	COUNT(*)
FROM warranty AS w
LEFT JOIN
sales AS s
ON s.sale_id = w.sale_id
WHERE w.claim_date - sale_date <= 180
```
13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
SELECT
	p.product_name,
	COUNT(w.claim_id) AS no_claim,
	COUNT(s.sale_id)
FROM warranty AS w
RIGHT JOIN
sales as s
ON s.sale_id = w.sale_id
JOIN products as p
ON p.product_id = s.product_id
WHERE p.launch_date >= CURRENT_DATE - INTERVAL '2 Year'
GROUP BY 1
HAVING COUNT(w.claim_id) > 0;
```
14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
SELECT
	TO_CHAR(sale_date, 'MM-YYYY') AS month,
	SUM(s.quantity) AS total_unit_sold
FROM sales AS s
JOIN
stores AS st
ON s.store_id = st.store_id
WHERE 
	st.country = 'USA'
	AND
	s.sale_date >= CURRENT_DATE - INTERVAL '3 Year'
GROUP BY 1
HAVING SUM(s.quantity) > 5000;
```
15. Identify the product category with the most warranty claims filed in the last three years.
```sql
SELECT 
	c.category_name,
	COUNT(w.claim_id) AS total_claims
FROM warranty AS w
LEFT JOIN
sales AS s
ON w.sale_id = s.sale_id
JOIN products AS p
ON p.product_id = s.product_id
JOIN
category AS c
ON c.category_id = p.category_id
WHERE
	w.claim_date >= CURRENT_DATE - INTERVAL '3 year'
GROUP BY 1;
```
### Complex (5 Questions)

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT 
	country,
	total_unit_sold,
	total_claim,
	COALESCE(total_claim:: numeric/total_unit_sold:: numeric * 100, 0) AS risk
FROM 
(SELECT 
	st.country,
	SUM(s.quantity) AS total_unit_sold,
	COUNT(w.claim_id) AS total_claim
FROM sales AS s
JOIN stores AS st
ON s.store_id = st.store_id
LEFT JOIN
warranty as w
ON w.sale_id = s.sale_id
GROUP BY 1
) AS t1
ORDER BY 4 DESC;
```
17. Analyze the year-by-year growth ratio for each store.
```sql
WITH yearly_sales
AS
(

		SELECT
			s.store_id,
			st.store_name,
			EXTRACT (YEAR FROM sale_date) AS year,
			SUM(s.quantity * p.price) AS total_sale
		FROM sales AS s
		JOIN 
		products AS p
		ON s.product_id = p.product_id
		JOIN stores AS st
		ON st.store_id = s.store_id
		GROUP BY 1, 2, 3	
		ORDER BY 2, 3
),
growth_ratio
AS
 (

SELECT 
	store_name,
	year,
	LAG(total_sale, 1) OVER(PARTITION BY store_name ORDER BY year) AS last_year_sale,
	total_sale AS current_year_sale
FROM yearly_sales
)

SELECT 
	store_name,
	year,
	last_year_sale,
	current_year_sale,
	ROUND(
			(current_year_sale - last_year_sale)::numeric /
			last_year_sale ::numeric * 100, 
			3) AS growth_raito
FROM growth_ratio
WHERE 
	last_year_sale IS NOT NULL
	AND
	YEAR <> EXTRACT (YEAR FROM CURRENT_DATE);
```
18. Calculate the correlation between product price and warranty claims for products sold in the last six years, segmented by price range.
```sql
SELECT 
	CASE
		WHEN p.price < 500 THEN 'Less Expensive Product'
		WHEN p.price BETWEEN 500 AND 1000 THEN 'Mid Range Product'
		ELSE 'EXpensive Product'
	END AS price_segment,
	COUNT(w.claim_id) AS total_claim
FROM warranty AS w
LEFT JOIN
sales AS s
ON w.sale_id = s.sale_id
JOIN
products AS p
ON p.product_id = s.product_id
WHERE claim_date >= CURRENT_DATE - INTERVAL '6 Year'
GROUP BY price_segment;
```
19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
WITH paid_repair
AS
(
SELECT
	s.store_id,
	COUNT(w.claim_id) AS paid_repaired
FROM sales AS s
RIGHT JOIN warranty AS w
ON w.sale_id = s.sale_id
WHERE w.repair_status = 'Paid Repaired'
GROUP BY 1
),

total_repaired
AS
(SELECT
	s.store_id,
	COUNT(w.claim_id) AS total_rapaired
FROM sales AS s
RIGHT JOIN warranty AS w
ON w.sale_id = s.sale_id
GROUP BY 1)

SELECT 
	tr.store_id,
	st.store_name,
	pr.paid_repaired,
	tr.total_rapaired,
	ROUND(pr.paid_repaired ::numeric/ tr.total_rapaired ::numeric * 100
	   ,2) AS percentage_paid_repaired
FROM paid_repair AS pr
JOIN
total_repaired tr
ON pr.store_id = tr.store_id
JOIN stores AS st
ON tr.store_id = st.store_id;
```
20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
WITH monthly_sales
AS
(
SELECT 
	store_id,
	EXTRACT (YEAR FROM sale_date) AS year,
	EXTRACT (MONTH FROM sale_date) AS month,
	SUM(p.price * s.quantity) AS total_revenue
FROM sales AS s
JOIN products AS p
ON s.product_id = p.product_id
GROUP BY 1, 2, 3
ORDER BY 1, 2, 3
)

SELECT 
	store_id,
	month,
	year,
	total_revenue,
	SUM(total_revenue) OVER(PARTITION BY store_id ORDER BY year, month) AS running_total
FROM monthly_sales;
```
### Bonus Question

- Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
SELECT 
	p.product_name,
    CASE
        WHEN s.sale_date BETWEEN p.launch_date 
             AND p.launch_date + INTERVAL '6 month'
            THEN '0-6 month'

        WHEN s.sale_date BETWEEN p.launch_date + INTERVAL '6 month'
             AND p.launch_date + INTERVAL '12 month'
            THEN '6-12 month'

        WHEN s.sale_date BETWEEN p.launch_date + INTERVAL '12 month'
             AND p.launch_date + INTERVAL '18 month'
            THEN '12-18 month'

        ELSE '18+ month'
    END AS PLC,
	SUM(s.quantity) AS total_qty_sale

FROM sales AS s
JOIN products AS p
ON s.product_id = p.product_id
GROUP BY 1, 2;
```

## Project Focus

In this project, I applied advanced SQL techniques to analyze large-scale retail sales data and solve real business problems.

The analysis involved working with complex joins and aggregations to integrate data from multiple tables, using window functions to perform trend and growth analysis, and segmenting data across different time periods to evaluate product lifecycle performance.

Additionally, I explored relationships between key business variables, such as product pricing and warranty claims, and answered practical business questions related to revenue drivers, regional performance, and operational risk patterns.

## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

## Key Insights

- A small number of flagship products contributed to the majority of total revenue, highlighting product concentration.
- Sales performance varied significantly across regions, indicating opportunities for targeted pricing and marketing strategies.
- Warranty claims were more frequent during the early lifecycle stage of products, emphasizing the importance of quality monitoring after launch.
- Sales showed strong lifecycle behavior, with peak performance occurring within the first 6–12 months after product release.
- Store-level performance differences revealed opportunities to replicate best practices from high-performing locations.

## Tools Used

- PostgreSQL
- SQL (Joins, Aggregations, Window Functions, CTEs, Date Functions)
- Data Modeling (ERD Design)
- Git & GitHub (Version Control and Documentation)

## Conclusion

Completing this project allowed me to fully leverage and deepen my SQL knowledge by working with a large-scale, real-world dataset. It challenged me to design efficient queries, implement advanced analytical techniques, and solve complex business problems using structured data.

Through this experience, I strengthened my expertise in data manipulation, performance optimization, window functions, CTEs, and time-based analysis. Most importantly, the project improved my ability to think like a data analyst — transforming raw data into actionable insights that drive business value.
This project demonstrates my strong SQL proficiency, analytical mindset, and readiness to contribute effectively in professional data analyst environments.

## Key Learnings

Working with a dataset of over one million records significantly enhanced my technical and analytical capabilities. This project strengthened my ability to:

- Write efficient and optimized SQL queries for large datasets
- Use advanced SQL concepts such as window functions, CTEs, aggregations, and complex joins
- Perform time-based and lifecycle analysis using date intervals
- Translate business questions into structured analytical queries
- Identify meaningful patterns and transform raw data into actionable insights

Most importantly, this project improved my analytical thinking — moving beyond writing queries to interpreting results and understanding their business impact.

Through this experience, I developed greater confidence in handling real-world datasets and delivering data-driven insights in a professional environment.

## Author - Mostafa Ramadan Ahmed

This project is part of my portfolio, showcasing the SQL skills for data analyst roles.

- **LinkedIn**: (https://www.linkedin.com/in/mostafa-ti-ahmed/)
- **Email Address**: (mustafatifa000034@gmail.com)

---
