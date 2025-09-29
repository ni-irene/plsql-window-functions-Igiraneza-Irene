STEP 1: PROBLEM DEFINITION

Business Context:
The supermarket company operates multiple branches across different regions, serving a wide variety of customers on a daily basis. With competition in the retail industry increasing, management has realized the importance of data-driven decision-making. By analyzing customer transactions, they hope to understand purchasing behavior, identify their most valuable customers, and track sales performance across time.

Data Challenge:
Although sales transactions are recorded in the company’s database, managers struggle to extract actionable insights. It is not immediately clear which customers contribute the most revenue, how sales are trending over time, or how to segment customers into meaningful groups for marketing. Relying only on raw sales data makes it difficult to identify growth patterns or to allocate resources effectively.

Expected Outcome:
Through the use of SQL window functions, the company expects to:
Rank customers by total revenue, helping to identify top spenders.
Calculate running daily totals and 7-day moving averages to track overall sales performance while smoothing short-term fluctuations.
Measure month-over-month growth in customer spending to monitor business momentum.
Segment customers into quartiles based on total revenue, so that marketing campaigns can be tailored to different customer groups.

By achieving these outcomes, management will be able to focus on high-value customers, forecast sales trends more accurately, and design effective loyalty programs and targeted promotional strategies.

STEP 2: SUCCESS CRITERIA

The following five success criteria define how SQL window functions will be applied to solve the business problem.

1.	Top customers by revenue → using ROW_NUMBER(), RANK(), DENSE_RANK(), and PERCENT_RANK() to identify which customers spend the most.
2.	Running daily sales totals → using SUM() OVER() to calculate cumulative revenue day by day.
3.	7-day moving average & trend analysis → using AVG() OVER() with a 7-day rolling window (plus MIN/MAX to check extremes).
4.	Month-over-month growth → using LAG() and LEAD() to calculate monthly revenue change percentages.
5.	Customer quartiles → using NTILE(4) and CUME_DIST() to segment 
customers into four revenue groups.

STEP 3: DATABASE SCHEMA

Table	                   Key Columns	                                                                                Example Row

customers	          customer_id (PK), name, region	                                                             1001, Uwase Diana, Kigali

products	          product_id (PK), name, category                        	                                     2001, Coffee Beans, Beverages

transactions	      transaction_id (PK), customer_id (FK), product_id (FK), sale_date, amount	                   3001, 1001, 2001, 2025-07-15, 25000


Entity Relationship Diagram:

1.	transactions.customer_id is a Foreign Key referencing
customers.customer_id.

3.	transactions.product_id is a Foreign Key referencing
products.product_id.

Relationships:
Relationship	                                   Type	                             Explanation

Customer  -->Transaction           	One-to-Many (1:M)	                    One Customer can be involved in many Transactions. Each Transactions belongs to only one Customer.

Product  --> Transaction	          One-to-Many (1:M)                     One Product can be sold in many Transactions. Each Transaction involves only one Product. 

Detailed diagram Structure:

  customers {
        INT customer_id PK
        VARCHAR name
        VARCHAR region
    }
    
    products {
        INT product_id PK
        VARCHAR name
        VARCHAR category
    }
    
    transactions {
        INT transaction_id PK
        INT customer_id FK
        INT product_id FK
        DATE sale_date
        DECIMAL amount
  }

STEP 4: WINDOW FUNCTIONS IMPLEMENTATION

(A)	Ranking (Top N customers by revenue)

Query:

SELECT customer_id, SUM(amount) AS total_revenue, ROW_NUMBER() OVER (ORDER BY SUM(amount) DESC) AS rank_row_number, DENSE_RANK() OVER (ORDER BY SUM(amount) DESC) AS rank_dense, PERCENT_RANK() OVER (ORDER BY SUM(amount) DESC) AS rank_percent FROM transactions GROUP BY customer_id ORDER BY total_revenue DESC;

	Interpretation: The ranking analysis highlights which customers contribute the most revenue overall. ROW_NUMBER() gives each customer a unique ranking, while RANK() and DENSE_RANK() handle ties differently. PERCENT_RANK() shows each customer’s relative standing compared to others. This helps management identify and focus on high-value customers.

(B)	Aggregate (Running totals & trends)

Query:

SELECT sale_date, SUM(amount) AS daily_sales, SUM(SUM(amount)) OVER (ORDER BY sale_date) AS running_total, AVG(SUM(amount)) OVER (ORDER BY sale_date  ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS seven_day_moving_avg, MAX(SUM(amount)) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS max_sales_last_7_days, MIN(SUM(amount)) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS min_sales_last_7_days FROM transactions GROUP BY sale_date ORDER BY sale_date;
 
	Interpretation: The cumulative sales confirm overall business growth, while the 7-day moving average smooths out daily spikes and dips. Including MIN and MAX in the rolling window also highlights the best and worst-performing days in each 7-day period. This gives a clear picture of short-term performance and sales stability.

(C)	Navigation (Growth % using LAG)

Query:

WITH MonthlySales AS (SELECT  DATE_FORMAT(sale_date, '%Y-%m-01') AS sale_month,        SUM(amount) AS monthly_sales_amount    FROM transactions   GROUP BY 1) SELECT sale_month, monthly_sales_amount, LAG(monthly_sales_amount, 1) OVER (ORDER BY sale_month) AS previous_month_sales, LEAD(monthly_sales_amount, 1) OVER (ORDER BY sale_month) AS next_month_sales, (monthly_sales_amount - LAG(monthly_sales_amount, 1) OVER (ORDER BY sale_month))     / NULLIF(LAG(monthly_sales_amount, 1) OVER (ORDER BY sale_month), 0) AS growth_percent FROM  MonthlySales ORDER BY sale_month;

	Since I practiced my queries in MySQL 8.0, functions such as DATE_FORMAT are written in MySQL syntax. In Oracle PL/SQL, the equivalent would be TRUNC(sale_date, 'MM').
 
	Interpretation: The LAG() function captures the previous month’s sales, and LEAD() provides the next month for comparison. This allows management to track month-over-month growth trends. Positive growth rates confirm market expansion, while negative growth signals areas needing corrective action.

(D)	Distribution (Customer segmentation with NTILE)

Query:

WITH CustomerRevenue AS (SELECT  customer_id, SUM(amount) AS total_spent FROM  transactions    GROUP BY  customer_id) SELECT customer_id, total_spent, NTILE(4) OVER (ORDER BY total_spent DESC) AS customer_quartile, CUME_DIST() OVER (ORDER BY total_spent) AS percentile_rank_cume_dist FROM CustomerRevenue ORDER BY total_spent DESC;

	Interpretation: NTILE(4) divides customers into four equal groups (Q1 = top spenders, Q4 = lowest spenders). CUME_DIST() shows the cumulative share of customers at or below each spending level. This segmentation highlights which customer groups are most valuable, guiding marketing and loyalty strategies to focus on top quartiles.

STEP 6: RESULTS ANALYSIS

Descriptive:
Daily sales are steadily increasing. A small group of top customers contributes the highest revenue. Customers fall into four clear spending tiers.

Diagnostic:
Certain customers consistently spend more, creating revenue concentration. Sales spikes appear around certain days, but the 7-day average shows stable upward growth.

Prescriptive:
Strengthen relationships with top-spending customers through loyalty rewards. Monitor weak-spending groups with promotions to increase activity. Use daily patterns and rolling averages to plan inventory and staffing more effectively.

STEP 7: REFERENCES

•	Oracle Documentation: SQL Window Functions

•	SQLShack Tutorials on Window Functions

•	GeeksforGeeks SQL Window Functions

•	W3Schools SQL Analytics Functions

•	Academic Papers on Business Analytics with SQL

•	DataCamp SQL Window Functions Guide

•	Mode Analytics Window Functions Tutorial

•	PostgreSQL Docs (Window Functions section)

•	TutorialsPoint SQL Analytics

•	StackOverflow SQL Q&A (cited with examples)
