# -plsql-window-functions-Simon-Pierre-MUKIZA
## A modern livestock farm Project.
## Project Overview
This project implements PL/SQL window functions to analyze farm sales data. It models a modern livestock farm with customers, products, and transactions. The system provides insights such as top products, customer quartiles, sales growth, and moving averages.
### Problem Definition
1.	Business Context: A modern livestock farm that houses cattle, goats, chickens, ducks, and turkeys in various locations.
2.	Data Challenge: Daily sales reports don't show management how sales are changing across different regions, which products are the best sellers, or how each branch is performing.
3.	Expected Outcome: To increase revenue, use analytics to identify the best products, monitor sales patterns, and enhance farm operations.
### Success Criteria
1.	Use RANK() to rank products by total sales in each region every three months.
2.	Use SUM() OVER() to figure out the total sales for each product month by month.
3.	Use LAG() to compare each month's sales to those of the prior month.
4.	Use NTILE(4) to divide clients into four equal groups based on how much they bought.
5.	Use AVG() OVER() to find the average sales for each product over a rolling 3-month period.
## Database Schema
```sql
CREATE TABLE customers ( Customer_id INT PRIMARY KEY, Names VARCHAR(100) NOT NULL, Region VARCHAR(100) NOT NULL );

CREATE TABLE products ( Product_id  INT PRIMARY KEY, Name VARCHAR(100) NOT NULL, Category    VARCHAR(100)  NOT NULL );

CREATE TABLE transactions (Transaction_id INT PRIMARY KEY, Customer_id    INT NOT NULL, Product_id     INT NOT NULL, Sale_date      DATE   NOT NULL, Amount DECIMAL(12,2) NOT NULL, FOREIGN KEY (Customer_id) REFERENCES customers(Customer_id), FOREIGN KEY (Product_id) REFERENCES products(Product_id) );
```
### Insert into customers
```sql INSERT INTO customers (Customer_id, Names, Region) VALUES (01, 'Simon Pierre', 'Kigali');
INSERT INTO customers (Customer_id, Names, Region) VALUES (02, 'Alicia M', 'Musanze');
INSERT INTO customers (Customer_id, Names, Region) VALUES (03, 'Peter Patrick', 'Rwamagana');
INSERT INTO customers (Customer_id, Names, Region) VALUES (04, 'Grace N', 'Musanze');
```
### Insert into products
``` sql
INSERT INTO products (Product_id, Name, Category) VALUES (0001, 'Fresh Milk', 'Dairy');
INSERT INTO products (Product_id, Name, Category) VALUES (0002, 'Goat Meat', 'Meat');
INSERT INTO products (Product_id, Name, Category) VALUES (0003, 'Chicken Eggs', 'Poultry');
INSERT INTO products (Product_id, Name, Category) VALUES (0004, 'Duck Meat', 'Meat');
```
### Insert into transaction
``` sql
INSERT INTO transactions VALUES (3001, 01, 0001, DATE '2024-01-15', 25000);
INSERT INTO transactions VALUES (3002, 02, 0002, DATE '2024-01-18', 8000);
INSERT INTO transactions VALUES (3004, 03, 0003, DATE '2024-02-05', 22000);
INSERT INTO transactions VALUES (3005, 04, 0004, DATE '2024-02-10', 30000);
```
## Window Functions Implementation
### 1.	Ranking Functions:
```sql
ROW_NUMBER():-- Assigns a unique sequential number to each customer ordered by total revenue (highest first).
SELECT c.customer_id,c.names,SUM(t.amount) AS total_revenue,ROW_NUMBER() OVER(ORDER BY SUM(t.amount) DESC) AS row_number FROM customers c JOIN transactions t ON c.customer_id = t.customer_id GROUP BY c.customer_id, c.names;
```
```sql
RANK():--Customers with the same total revenue share the same rank.
SELECT  c.customer_id,c.names, SUM(t.amount) AS total_revenue,RANK() OVER(ORDER BY SUM(t.amount) DESC) AS rank_position FROM customers c JOIN transactions t ON c.customer_id = t.customer_id GROUP BY c.customer_id, c.names;
```
```sql
DENSE_RANK():--Customers with the same total revenue share the same rank.
SELECT c.customer_id, c.names,SUM(t.amount) AS total_revenue,DENSE_RANK() OVER(ORDER BY SUM(t.amount) DESC) AS dense_rank_position FROM customers c JOIN transactions t ON c.customer_id = t.customer_id GROUP BY c.customer_id, c.names;
```
```sql
PERCENT_RANK():--Shows each customer’s relative standing as a percentage (0 to 1).
SELECT c.customer_id,c.names,SUM(t.amount) AS total_revenue,PERCENT_RANK() OVER(ORDER BY SUM(t.amount) DESC) AS percent_rank_position FROM customers c JOIN transactions t ON c.customer_id = t.customer_id GROUP BY c.customer_id, c.names;
```
### 2.	Aggregate:
```sql
SUM():-- SUM() with ROWS: Running total based on number of rows up to current row

SELECT transaction_id,sale_date,amount,SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total_rows FROM transactions
ORDER BY sale_date;
```
```sql
AVG():-- AVG() with RANGE: Running average, considers value ranges.
SELECT transaction_id,sale_date,amount,AVG(amount) OVER (ORDER BY sale_date RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_avg_range FROM transactions
ORDER BY sale_date;
```
```sql
MIN():-- MIN() with ROWS: Minimum sale value up to current row
SELECT transaction_id,sale_date,amount,MIN(amount) OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS min_so_far FROM transactions ORDER BY sale_date;
```
```sql
MAX():-- MAX() with RANGE: Maximum sale value up to current point, groups equal ORDER BY values together
SELECT transaction_id,sale_date,amount,MAX(amount) OVER (ORDER BY sale_date RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS max_so_far FROM transactions ORDER BY sale_date;
```
### 3.	Navigation
```sql
LAG():-- LAG(): Shows the sales from the previous month for comparison.
SELECT TO_CHAR(t.sale_date, 'YYYY-MM') AS month,SUM(t.amount) AS monthly_sales,LAG(SUM(t.amount)) OVER (ORDER BY TO_CHAR(t.sale_date, 'YYYY-MM')) AS prev_month_sales
FROM transactions t GROUP BY TO_CHAR(t.sale_date, 'YYYY-MM') ORDER BY month;
```
```sql
LEAD():-- LEAD(): Shows the sales from the following month
SELECT TO_CHAR(t.sale_date, 'YYYY-MM') AS month,SUM(t.amount) AS monthly_sales,LEAD(SUM(t.amount)) OVER (ORDER BY TO_CHAR(t.sale_date, 'YYYY-MM')) AS next_month_sales
FROM transactions t GROUP BY TO_CHAR(t.sale_date, 'YYYY-MM') ORDER BY month;
```
### 4.	Distribution
```sql
NTILE(4): -- NTILE(4): Splits customers into 4 groups
SELECT c.customer_id,c.names,SUM(t.amount) AS total_revenue,NTILE(4) OVER (ORDER BY SUM(t.amount)) AS revenue_quartile FROM customers c JOIN transactions t ON c.customer_id = t.customer_id GROUP BY c.customer_id, c.names ORDER BY total_revenue;
```
```sql
CUME_DIST():-- CUME_DIST(): Shows the relative position of a customer in the distribution of revenues.
SELECT c.customer_id,c.names,SUM(t.amount) AS total_revenue,CUME_DIST() OVER (ORDER BY SUM(t.amount)) AS revenue_distribution FROM customers c JOIN transactions t ON c.customer_id = t.customer_id GROUP BY c.customer_id, c.names ORDER BY total_revenue;
```
## Results Analysis
1. Descriptive, What happened?
Sales grew overall, but some months dropped. A few customers brought in most of the revenue.
2. Diagnostic, Why?
The drops came from seasonal changes and supply issues. High sales came from loyal customers and busy regions.
3. Prescriptive, What next?
Keep strong ties with top buyers, prepare for high-demand seasons, and support small buyers to grow.

