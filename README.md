# bigquery_budiman


1.
```sql

   WITH brand_gmv AS (
  SELECT 
    t.sbu,
    td.brand_name,
    SUM(td.gmv) AS total_gmv
  FROM `Trx` t
  JOIN `Trx_detail` td 
    ON t.transaction_id = td.transaction_id 
    AND t.sbu = td.sbu
  WHERE t.ordertime <= '2024-12-31'
  GROUP BY 1,2
),
ranked_brands AS (
  SELECT 
    sbu,
    brand_name,
    total_gmv,
    RANK() OVER (PARTITION BY sbu ORDER BY total_gmv DESC) AS rank
  FROM brand_gmv
)
SELECT 
  sbu,
  brand_name,
  total_gmv
FROM ranked_brands
WHERE rank <= 5;
```

2.
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

3.
```sql
WITH ordered_transactions AS (
  SELECT 
    customer_email,
    ordertime,
    LAG(ordertime) OVER (PARTITION BY customer_email ORDER BY ordertime) AS prev_ordertime,
    ROW_NUMBER() OVER (PARTITION BY customer_email ORDER BY ordertime) AS rn
  FROM `Trx`
),
second_purchases AS (
  SELECT 
    customer_email,
    DATE_DIFF(ordertime, prev_ordertime, DAY) AS days_to_second_purchase
  FROM ordered_transactions
  WHERE rn = 2
)
SELECT 
  AVG(days_to_second_purchase) AS avg_days_to_second_purchase
FROM second_purchases;
```

4.
```sql
WITH customer_gmv AS (
  SELECT 
    t.customer_email,
    SUM(td.gmv) AS total_gmv
  FROM `Trx` t
  JOIN `Trx_detail` td 
    ON t.transaction_id = td.transaction_id 
    AND t.sbu = td.sbu
  WHERE EXTRACT(YEAR FROM t.ordertime) = 2024
  GROUP BY 1
),
top_customers AS (
  SELECT 
    customer_email,
    total_gmv
  FROM customer_gmv
  ORDER BY total_gmv DESC
  LIMIT 10
),
customer_brands AS (
  SELECT 
    t.customer_email,
    ARRAY_AGG(DISTINCT td.brand_name) AS brands_purchased
  FROM `Trx` t
  JOIN `Trx_detail` td 
    ON t.transaction_id = td.transaction_id 
    AND t.sbu = td.sbu
  WHERE t.customer_email IN (SELECT customer_email FROM top_customers)
  GROUP BY 1
)
SELECT 
  tc.customer_email,
  tc.total_gmv,
  cb.brands_purchased
FROM top_customers tc
JOIN customer_brands cb 
  ON tc.customer_email = cb.customer_email
ORDER BY tc.total_gmv DESC;
```
   
