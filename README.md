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
  GROUP BY t.sbu, td.brand_name
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
   
