# 🚀 E-Commerce Executive Sales Analytics (TheLook Dataset)

## 📌 Business Overview
This project targets large-scale transactional data within a global cloud warehouse to engineer an automated, interactive sales reporting system. It transforms over 70,000 raw transaction logs into executive-ready business insights.

## 🔧 Technologies Used
* **Cloud Platform:** Google Cloud Platform (BigQuery Sandbox)
* **Data Transformation Language:** Advanced SQL (CTEs, Window Functions, Case Statements)
* **Business Intelligence Tool:** Looker Studio (Custom High-Contrast Theme)

## 💡 Core Data Solutions Engineered
1. **Financial Framework:** Automated calculations to isolate gross revenues, unique customer footprints, and true margins.
2. **Trend Analytics Smoothing:** Implemented a 7-day rolling revenue window function to bypass mid-week operational spikes and expose core consumer cycles.
3. **Multi-Dimension Visualization Matrix:** Built comprehensive tracking layers evaluating metrics against customer demographics and transaction fulfillment states.

## 📊 Live Metrics & Project Links
* **GROSS REVENUE ANALYZED:** $4.41M
* **TOTAL REVENUE TRANSACTIONS:** 71,508
* **GROSS PROFIT RETAINED:** $2.29M

🔗 **[Click Here to Interact with My Live Dashboard](https://datastudio.google.com/reporting/413736a5-74b9-4841-b4e4-da1c182a5ac8)**

---

## 💻 Clean Production SQL Script
Below is the optimized script executed inside Google BigQuery to model our master dashboard layout matrix:

```sql
-- Step 1: Aggregate daily sales, order volumes, and customer acquisition metrics
WITH daily_metrics AS (
  SELECT
    EXTRACT(DATE FROM o.created_at) AS order_date,
    o.status AS order_status,
    p.category AS product_category,
    p.department AS customer_gender,
    COUNT(DISTINCT o.order_id) AS total_orders,
    COUNT(DISTINCT o.user_id) AS unique_customers,
    SUM(oi.sale_price) AS gross_revenue,
    SUM(p.cost) AS total_cost
  FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
  INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi 
    ON o.order_id = oi.order_id
  INNER JOIN `bigquery-public-data.thelook_ecommerce.products` AS p 
    ON oi.product_id = p.id
  WHERE o.created_at >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 365 DAY)
  GROUP BY 1, 2, 3, 4
),

financial_summary AS (
  SELECT
    order_date,
    order_status,
    product_category,
    customer_gender,
    total_orders,
    unique_customers,
    gross_revenue,
    CASE 
      WHEN order_status IN ('Shipped', 'Complete') THEN gross_revenue 
      ELSE 0 
    END AS net_revenue,
    (gross_revenue - total_cost) AS gross_profit
  FROM daily_metrics
)

SELECT
  order_date,
  DATE_TRUNC(order_date, MONTH) AS order_month,
  order_status,
  product_category,
  customer_gender,
  total_orders,
  unique_customers,
  ROUND(gross_revenue, 2) AS gross_revenue,
  ROUND(net_revenue, 2) AS net_revenue,
  ROUND(gross_profit, 2) AS gross_profit,
  ROUND((gross_profit / NULLIF(gross_revenue, 0)) * 100, 2) AS profit_margin_percent,
  ROUND(
    AVG(gross_revenue) OVER(
      PARTITION BY product_category 
      ORDER BY order_date 
      ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ), 2
  ) AS rolling_7day_avg_revenue
FROM financial_summary
ORDER BY order_date DESC;
```
