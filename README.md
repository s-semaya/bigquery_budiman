# bigquery_budiman


**1. What are the top 5 brands based on GMV for each SBU in PTD (Period to Date) (up to December 2024)?** 
```sql

SELECT 
  sbu,
  brand_name,
  total_gmv
FROM (
  SELECT 
    t.sbu,
    td.brand_name,
    SUM(td.gmv) AS total_gmv,
    RANK() OVER (PARTITION BY t.sbu ORDER BY SUM(td.gmv) DESC) AS rnk
  FROM `Trx` t
  JOIN `Trx_detail` td 
    ON t.transaction_id = td.transaction_id 
    AND t.sbu = td.sbu
  WHERE t.ordertime <= '2024-12-31'
  GROUP BY t.sbu, td.brand_name
)
WHERE rnk <= 5;

```
<br>

**2. What is the customer’s first transaction for each SBU, and when did they make the purchase?**
```sql

SELECT 
  customer_email,
  sbu,
  transaction_id,
  ordertime
FROM (
  SELECT 
    customer_email,
    sbu,
    transaction_id,
    ordertime,
    ROW_NUMBER() OVER (PARTITION BY customer_email, sbu ORDER BY ordertime) AS rn
  FROM `Trx`
)
WHERE rn = 1;

```
<br>

**3. How long does it take for a customer to make their second purchase?**
```sql

SELECT AVG(days_diff) AS avg_days_to_second_purchase
FROM (
  SELECT 
    customer_email,
    DATE_DIFF(ordertime, LAG(ordertime) OVER (PARTITION BY customer_email ORDER BY ordertime), DAY) AS days_diff,
    ROW_NUMBER() OVER (PARTITION BY customer_email ORDER BY ordertime) AS rn
  FROM `Trx`
)
WHERE rn = 2;

```
<br>

**4. Identify the top 10 customers with the highest GMV in 2024 and the brands they purchased.**
```sql

SELECT 
  customer_email,
  SUM(td.gmv) AS total_gmv,
  ARRAY_AGG(DISTINCT td.brand_name) AS brands_purchased
FROM `Trx` t
JOIN `Trx_detail` td 
  ON t.transaction_id = td.transaction_id 
  AND t.sbu = td.sbu
WHERE EXTRACT(YEAR FROM t.ordertime) = 2024
GROUP BY customer_email
ORDER BY total_gmv DESC
LIMIT 10;

```
   
