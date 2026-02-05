# plsql-window-functions-KWIZERA Prince
Milk Delivery Company
# Milk Delivery Company – Window Functions Project  
**Student:** KWIZERA Prince 
**ID:** 28426  
**Course:Database Development with PL/SQL   – AUCA  
**Instructor:** Mr. Eric Maniraguha  

---

##  1. Problem Definition
**Business Context:**  
A milk delivery company distributes fresh and processed milk to households, shops, and schools across multiple regions.  

**Data Challenge:**  
The company struggles to track liters sold each month by region and customer type. It is difficult to identify top customers and to monitor changes in sales over time. This creates risks such as poor demand forecasting, stock shortages, and missed opportunities to reward loyal customers.  

**Expected Outcome:**  
Reports that highlight top 5 customers per region, monthly running totals, month-over-month growth, and customer quartile segmentation for targeted loyalty programs.

---

##  2. Success Criteria
The project will be successful if it achieves these **five measurable goals**:  
1. Identify **Top 5 customers per region by liters delivered** using `RANK()` or `ROW_NUMBER()`.  
2. Compute **running monthly totals** of liters sold using `SUM() OVER (ORDER BY ...)`.  
3. Calculate **month-over-month growth** in liters with `LAG()`.  
4. Segment customers into **quartiles** with `NTILE(4)`.  
5. Calculate a **3-month moving average** of liters delivered using `AVG() OVER (...)`.  

---

##  3. Database Schema

### Tables
**Customers**
```sql
CREATE TABLE customers (
  customer_id  NUMBER PRIMARY KEY,
  name         VARCHAR2(100),
  region       VARCHAR2(50)
);
```


Products
```sql
CREATE TABLE products (
  product_id   NUMBER PRIMARY KEY,
  name         VARCHAR2(100),
  category     VARCHAR2(50)
);
```
DELIVERIES
```sql
CREATE TABLE deliveries (
  delivery_id    NUMBER PRIMARY KEY,
  transaction_id NUMBER REFERENCES transactions(transaction_id),
  delivery_date  DATE,
  status         VARCHAR2(20)
);
```
Sample data

customers
```sql
INSERT INTO customers VALUES (1, 'Alice Uwase', 'Kigali');
INSERT INTO customers VALUES (2, 'John Niyonsenga', 'Musanze');
INSERT INTO customers VALUES (3, 'Grace Mukamana', 'Huye');
INSERT INTO customers VALUES (4, 'Eric Habimana', 'Rubavu');
```
![customers](screenshots/Customers.png)
products
```sql
INSERT INTO products VALUES (101, 'Coffee Beans', 'Beverage');
INSERT INTO products VALUES (102, 'Tea Pack', 'Beverage');
INSERT INTO products VALUES (103, 'Milk Carton', 'Dairy');
INSERT INTO products VALUES (104, 'Chocolate Bar', 'Snack');
```
![customers](https://github.com/kprince43/plsql_window_functions_28426_prince1/blob/main/images/Customers.png)
deliveries
```sql
INSERT INTO deliveries VALUES (301, 1, 101, DATE '2025-09-02', 5.0, 12500.00);

INSERT INTO deliveries VALUES (302, 2, 103, DATE '2025-09-06', 10.0, 42000.00);

INSERT INTO deliveries VALUES (303, 3, 104, DATE '2025-09-11', 2.0, 3000.00);

INSERT INTO deliveries VALUES (304, 4, 102, DATE '2025-09-16', 8.0, 24800.00);
```
![deliveries](https://github.com/kprince43/plsql_window_functions_28426_prince1/blob/main/images/deliveries.png)
 4. Window Functions Implementation
1) Ranking Functions
```sql
SELECT customer_id,
       SUM(amount) AS total_amount,
       ROW_NUMBER() OVER (ORDER BY SUM(amount) DESC) AS row_num,
       RANK() OVER (ORDER BY SUM(amount) DESC) AS rank_num,
       DENSE_RANK() OVER (ORDER BY SUM(amount) DESC) AS dense_rank_num
FROM deliveries
GROUP BY customer_id
ORDER BY total_amount DESC;
--Interpretation:

--ROW_NUMBER gives a strict order (no ties).

--RANK leaves gaps if there are ties.

--DENSE_RANK doesn’t leave gaps.
```
![Ranking Query](screenshots/rank.png)
2) Aggregate Functions
Running totals
```sql
--sum
SELECT delivery_date,
       SUM(amount) AS daily_amount,
       SUM(SUM(amount)) OVER (
           ORDER BY delivery_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM deliveries
GROUP BY delivery_date
ORDER BY delivery_date;

--average
SELECT delivery_date,
       SUM(liters) AS daily_liters,
       AVG(SUM(liters)) OVER (
           ORDER BY delivery_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg_liters
FROM deliveries
GROUP BY delivery_date
ORDER BY delivery_date;

--Running totals show cumulative revenue, while moving averages smooth out daily fluctuations.

```

3) Navigation Functions
```sql
SELECT delivery_date,
       SUM(liters) AS daily_liters,
       LAG(SUM(liters)) OVER (ORDER BY delivery_date) AS prev_day_liters,
       SUM(liters) - LAG(SUM(liters)) OVER (ORDER BY delivery_date) AS liters_diff
FROM deliveries
GROUP BY delivery_date
ORDER BY delivery_date;

--Shows how deliveries increased or decreased compared to the previous date.
```
4) Distribution Functions
```sql
SELECT customer_id,
       SUM(amount) AS total_amount,
       NTILE(4) OVER (ORDER BY SUM(amount) DESC) AS quartile,
       CUME_DIST() OVER (ORDER BY SUM(amount) DESC) AS cum_dist
FROM deliveries
GROUP BY customer_id
ORDER BY total_amount DESC;
--Interpretation:
--Divides customers into 4 groups (quartiles) and shows the cumulative distribution of revenue.
```
5. Results Analysis

Descriptive (What happened?):
Deliveries were made across regions, with Kigali leading in demand.

Diagnostic (Why?):
Customers in urban regions tend to purchase more liters. Product type and seasonality influence sales volume.

Prescriptive (What next?):

Reward top quartile customers with loyalty programs.

Improve forecasting by monitoring moving averages.

Target underperforming regions with promotional campaigns.

References
Oracle SQL Documentation – Window Functions

AUCA Lecture Notes

W3Schools SQL Tutorial

Tutorialspoint SQL Analytical Functions

GeeksforGeeks SQL Window Functions

DB-Diagram tool for ERD

6.Academic Integrity Statement

I declare that this project is my own work. All sources are cited, and the queries, interpretations, and analysis were developed by me without unauthorized assistance
