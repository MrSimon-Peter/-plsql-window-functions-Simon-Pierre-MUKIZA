# -plsql-window-functions-Simon-Pierre-MUKIZA
## A modern livestock farm Project.
### Project Overview
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
### Database Schema
CREATE TABLE customers ( Customer_id INT PRIMARY KEY, Names VARCHAR(100) NOT NULL, Region VARCHAR(100) NOT NULL );

CREATE TABLE products ( Product_id  INT PRIMARY KEY, Name VARCHAR(100) NOT NULL, Category    VARCHAR(100)  NOT NULL );

CREATE TABLE transactions (Transaction_id INT PRIMARY KEY, Customer_id    INT NOT NULL, Product_id     INT NOT NULL, Sale_date      DATE   NOT NULL, Amount DECIMAL(12,2) NOT NULL, FOREIGN KEY (Customer_id) REFERENCES customers(Customer_id), FOREIGN KEY (Product_id) REFERENCES products(Product_id) );
