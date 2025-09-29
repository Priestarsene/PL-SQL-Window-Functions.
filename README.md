# PL-SQL-Window-Functions.
# NSHUTI MUGABO Arsene (27668)
## 📌 Problem Definition

Business Context:
A retail company operating in Rwanda wants to better understand customer purchasing behavior. The company sells a variety of products including beverages, bakery, dairy, and grocery items.
Data Challenge:
Management wants to identify top products per region, track monthly sales, measure growth, and segment customers for targeted marketing campaigns.
Expected Outcome:
Generate actionable insights such as top-performing products by region and quarter, customer spending segments, and sales trends using PL/SQL window functions.

## 🎯 Success Criteria

The analysis will demonstrate the following goals using PL/SQL analytic functions:
1.	Top 5 products per region/quarter → RANK()
2.	Running monthly sales totals → SUM() OVER()
3.	Month-over-month growth → LAG() / LEAD()
4.	Customer quartiles → NTILE(4)
5.	3-month moving averages → AVG() OVER()

## 🛠 Database Schema
Three relational tables were created:
•	customers: stores customer details
•	products: stores product catalog
•	transactions: records all sales
```sql
CREATE TABLE customers (
    customer_id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    region VARCHAR2(50)
);
```
```sql
CREATE TABLE products (
    product_id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    category VARCHAR2(50)
);
```
```sql
CREATE TABLE transactions (
    transaction_id NUMBER PRIMARY KEY,
    customer_id NUMBER REFERENCES customers(customer_id),
    product_id NUMBER REFERENCES products(product_id),
    sale_date DATE,
    amount NUMBER(12,2)
);
```
<img width="600" height="141" alt="s1" src="https://github.com/user-attachments/assets/b0799780-11d1-4e35-8c97-5ef47845c90a" />
<img width="576" height="167" alt="cTable" src="https://github.com/user-attachments/assets/5cd160c3-a6c3-4db4-be30-15c2c09591b9" />

<img width="457" height="109" alt="s11" src="https://github.com/user-attachments/assets/e66531bb-eb07-41ed-9241-203c5027a9e3" />
<img width="631" height="188" alt="pTable" src="https://github.com/user-attachments/assets/2508a2a2-560b-4313-a37f-3597f7957a9c" />

<img width="718" height="247" alt="s12" src="https://github.com/user-attachments/assets/3ffd2c9a-ceb4-4b2b-a24a-074654c4d8d4" />
<img width="570" height="203" alt="tTable" src="https://github.com/user-attachments/assets/b4a3f06f-7dc4-470e-8c4e-81cb091ced14" />

## 💻 Window Function Queries
1. Top 5 Products per Region/Quarter → RANK()
   
```sql
SELECT region, quarter, product_name, total_amount, rn
FROM (
    SELECT c.region,
           TO_CHAR(t.sale_date, 'YYYY-"Q"Q') AS quarter,
           p.name AS product_name,
           SUM(t.amount) AS total_amount,
           RANK() OVER (
               PARTITION BY c.region, TO_CHAR(t.sale_date, 'YYYY-"Q"Q')
               ORDER BY SUM(t.amount) DESC
           ) AS rn
    FROM transactions t
    JOIN customers c ON t.customer_id = c.customer_id
    JOIN products p ON t.product_id = p.product_id
    GROUP BY c.region, TO_CHAR(t.sale_date, 'YYYY-"Q"Q'), p.name
) ranked_sales
WHERE rn <= 5
ORDER BY region, quarter, rn;
```
<img width="626" height="229" alt="Srank1" src="https://github.com/user-attachments/assets/b8aa2006-1e62-4d67-ac39-0070fda0ea2b" />
<img width="557" height="65" alt="rankt" src="https://github.com/user-attachments/assets/6f1819cb-4610-44bf-b948-f55a82e3397c" />

2. Running Monthly Sales Totals → SUM() OVER()

```sql
SELECT month, monthly_sales,
       SUM(monthly_sales) OVER (ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM (
    SELECT TO_CHAR(sale_date, 'YYYY-MM') AS month,
           SUM(amount) AS monthly_sales
    FROM transactions
    GROUP BY TO_CHAR(sale_date, 'YYYY-MM')
)
ORDER BY month;
```
Insight: Shows cumulative sales growth month by month.

<img width="536" height="225" alt="SUM" src="https://github.com/user-attachments/assets/a9cd8a33-8bb9-49f4-b0da-8f0343719fb3" />


<img width="354" height="65" alt="monthly" src="https://github.com/user-attachments/assets/758f98d2-f837-417c-8b3f-2c107f7e4871" />

3. Month-over-Month Growth → LAG()

```sql
SELECT month,
       monthly_sales,
       LAG(monthly_sales) OVER (ORDER BY month) AS prev_month,
       ROUND(((monthly_sales - LAG(monthly_sales) OVER (ORDER BY month)) 
              / LAG(monthly_sales) OVER (ORDER BY month)) * 100, 2) AS growth_percent
FROM (
    SELECT TO_CHAR(sale_date, 'YYYY-MM') AS month,
           SUM(amount) AS monthly_sales
    FROM transactions
    GROUP BY TO_CHAR(sale_date, 'YYYY-MM')
)
ORDER BY month;
```
Insight: Measures percentage growth compared to the previous month.

<img width="662" height="232" alt="LAG" src="https://github.com/user-attachments/assets/05e65891-1897-4929-9e8d-bfc4bafa3df9" />
<img width="476" height="68" alt="lag1" src="https://github.com/user-attachments/assets/c978b12b-6018-4cf5-a264-296ad05b277d" />

4. Customer Quartiles → NTILE(4)
```sql
SELECT customer_id,
       total_spent,
       NTILE(4) OVER (ORDER BY total_spent DESC) AS spending_quartile
FROM (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM transactions
    GROUP BY customer_id
)
ORDER BY spending_quartile, total_spent DESC;
```
Insight: Groups customers into 4 segments (top spenders vs low spenders).

<img width="536" height="180" alt="NTILE" src="https://github.com/user-attachments/assets/783801dd-3d49-488f-a371-cf3c4ee8a8b6" />

<img width="374" height="67" alt="ntile1" src="https://github.com/user-attachments/assets/bc99a0df-94de-4a76-ace9-885a01a5953e" />

5. 3-Month Moving Average → AVG() OVER()
```sql
SELECT month, monthly_sales,
       ROUND(AVG(monthly_sales) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS three_month_avg
FROM (
    SELECT TO_CHAR(sale_date, 'YYYY-MM') AS month,
           SUM(amount) AS monthly_sales
    FROM transactions
    GROUP BY TO_CHAR(sale_date, 'YYYY-MM')
)
ORDER BY month;
```
Insight: Smooths out sales fluctuations by calculating rolling 3-month averages.

<img width="503" height="394" alt="AVG" src="https://github.com/user-attachments/assets/301b73c8-9788-45e0-94d5-422e2f67e7e3" />

<img width="375" height="65" alt="3mon" src="https://github.com/user-attachments/assets/c0db5a21-f09b-4d27-9b42-d0f8a0452f30" />

## 📊 Results Analysis

Descriptive:
•	Kigali consistently leads in sales, with Coffee Beans as a top product.
•	Sales increased steadily from January to May.
Diagnostic:
•	Promotions in March likely boosted bread and dairy sales.
•	Kigali region’s dominance is due to higher customer density.
Prescriptive:
•	Expand marketing in Musanze and Huye to balance revenue streams.
•	Increase stock of top-performing beverages in Kigali during Q2.

## 📚 References
1.	Oracle Docs – Analytic Functions
2.	Oracle 10g SQL Reference
3.	TutorialsPoint – PL/SQL Window Functions
4.	GeeksforGeeks – SQL Analytics
5.	AUCA Lecture Notes
6.	W3Schools SQL Window Functions
7.	Mode Analytics SQL Window Tutorial
8.	IBM SQL Analytic Functions Guide
9.	TowardsDataScience – SQL for Data Analysis
10.	PostgreSQL/Oracle comparison articles


✅ Integrity Statement
All sources were properly cited. Implementations and analysis represent original work. No AI-generated content was copied without attribution or adaptation.
