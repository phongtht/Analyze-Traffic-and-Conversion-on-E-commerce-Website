# Analyze-Traffic-and-Conversion-on-E-commerce-Website

---
<img width="1263" height="612" alt="image" src="https://github.com/user-attachments/assets/662d14ec-d4e6-462e-a3af-530a7a76ecfb" />

👉🏻Change Icon emoji 🔥🔍📘🚩 to your likings by clicking "Start" + "."

# 📊 Project: Analyze-Traffic-and-Conversion-on-E-commerce-Website  
Author: Tran Huu Tran Phong 
Date: YYYY-MM-DD  
Tools Used: SQL on Google BigQuery 

---

## 📑 Table of Contents  
I. [📌 Background & Overview](#-background--overview)  
II. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
III. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---
## 📌 Background & Overview  

### Objective:
### 📖 What is this project about? What Business Question will it solve?

🎯 Main Business Question

How can the business optimize website performance and user conversion by analyzing visitor behavior, traffic sources, and product-level purchase patterns?

📘 Project Overview

  - Data Source: A comprehensive dataset of web analytics (visits, pageviews, bounce rates) and e-commerce transactions from 2017.

  - Methodology: Advanced SQL querying to analyze performance at the session, user, and product levels.

  - Goal: To diagnose traffic quality, revenue drivers, and funnel efficiency.

💡 Business Questions this project answers

- ✔️ How does website performance change across months (visits, pageviews, and transactions)?
- ✔️ Which traffic sources bring the most valuable visitors and have the lowest bounce rates?
- ✔️ How does revenue vary by traffic source and over time (weekly and monthly trends)?
- ✔️ How do purchasers behave differently from non-purchasers in terms of engagement (pageviews, transactions, and spending)?
- ✔️ What is the average value per session and per customer, helping to measure marketing ROI?
- ✔️ Which products are commonly purchased together, revealing potential cross-sell opportunities?
- ✔️ What is the conversion funnel from product view → add-to-cart → purchase, and how efficient is it for each product?

### 👤 Who is this project for?  

- ✔️ Digital Marketing Teams – to evaluate traffic sources, campaign performance, and bounce rate.
- ✔️ E-commerce Managers – to monitor key metrics like transactions, revenue, and conversion rates.
- ✔️ Product Managers – to identify top-performing and underperforming products and potential bundling opportunities.
- ✔️ Data Analysts – to build dashboards or performance reports using SQL and analytics data.
- ✔️ Business Decision Makers / Executives – to make data-driven decisions about marketing spend, website optimization, and sales strategy.

## II. Data Access
1. Log in to Google Cloud Platform (GCP) and create a new project.

2. Navigate to the BigQuery console and ensure your new project is selected.

3. In the left navigation panel, click "Add Data" and then select "Search by project name" (or "Search a project" in older UIs).

4. Enter the public project ID: bigquery-public-data and press Enter.

5. In the navigation panel, expand the project bigquery-public-data, then expand the dataset google_analytics_sample.

6. Click on the ga_sessions_* table to view its schema and data.

## III. Exploring Dataset
In this project, I will write 08 query in Bigquery base on Google Analytics dataset
### Task 1: Querying traffic trends of website in first quarter of 2017 
* SQL code
~~~sql
SELECT
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
  ,SUM(totals.visits) as visits
  ,SUM(totals.pageviews) as pageviews
  ,SUM(totals.transactions) as transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
~~~
* Query results
  
  <img width="690" height="120" alt="kq1" src="https://github.com/user-attachments/assets/bbd6e125-0da1-4aa2-b085-48289f132172" />

| month  | visits | pageviews | transactions |
|--------|--------|-----------|--------------|
| 201701 | 64694  | 257708    | 713          |
| 201702 | 62192  | 233373    | 733          |
| 201703 | 69931  | 259522    | 993          |

**Insights:**


### Task 2: Top traffic sources in July 2017
* SQL code
~~~sql
SELECT 
  trafficSource.source as source
  ,SUM(totals.visits) as total_visits
  ,SUM(totals.bounces) as total_no_of_bounces
  ,SUM(totals.bounces)/SUM(totals.visits) as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY source
ORDER BY total_visits DESC
LIMIT 10;
~~~

* Query results
  
  <img width="688" height="499" alt="kq2" src="https://github.com/user-attachments/assets/07a72646-a19e-4ceb-b5a0-93cf454b5940" />

  **Insights:**

  - Google: It’s a primary engine, driving ~60% of total traffic (38k visits). It’s working, keep fueling it.

  - Direct: This is a highest-quality traffic. They have the lowest bounce rate (43.3%), meaning these users know your brand and actually stick around.

  - YouTube: This is a weakest link. It has the lowest volume (6k) and the highest bounce rate (66.7%). Two out of three people leave immediately, perhaps suggesting       your video content doesn't match what's on the website.

### Task 3:  Revenue by Traffic Sources in June 2017
* SQL code
~~~sql
WITH month_summary AS (
    SELECT
        FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS time,
        trafficSource.source AS source,
        SUM(product.productRevenue) / 1000000 AS revenue
    FROM
        `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
        UNNEST(hits)hits,
        UNNEST(hits.product)product
        WHERE product.productRevenue is not null
    GROUP BY
        time,
        source
),

week_summary AS (
    SELECT
        FORMAT_DATE('%Y%W', PARSE_DATE('%Y%m%d', date)) AS time,
        trafficSource.source AS source,
        SUM(product.productRevenue) / 1000000 AS revenue
    FROM
        `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
        UNNEST(hits)hits,
        UNNEST(hits.product)product
        WHERE product.productRevenue is not null
    GROUP BY
        time,
        source
)

SELECT
    'Month' as time_type,
    month_summary.time,
    source,
    revenue
FROM
    month_summary
UNION ALL 
SELECT 
    'Week' as time_type,
    week_summary.time,
    source,
    revenue
FROM week_summary
ORDER BY revenue DESC
LIMIT 10;

~~~

* Query results
  
  <img width="855" height="502" alt="kq3" src="https://github.com/user-attachments/assets/9e13272a-3b75-4440-935a-d90899e3fad4" />

**Insights:**

### Task 4: Average number of product pageviews by purchaser type (purchasers vs non-purchasers) in June & July 2017
* SQL code
~~~sql
WITH purchasers as
(
SELECT
 FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) month
 ,SUM(totals.pageviews)/COUNT(DISTINCT fullVisitorId) as avg_pageviews_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) hits,
UNNEST (hits.product)product
WHERE _table_suffix BETWEEN '0601' AND '0731'
AND  totals.transactions >=1
AND productRevenue is not null
GROUP BY month
),
non_purchasers as
(
SELECT
 FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) month
 ,SUM(totals.pageviews)/COUNT(DISTINCT fullVisitorId) as avg_pageviews_non_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) hits,
UNNEST (hits.product)product
WHERE _table_suffix BETWEEN '0601' AND '0731'
AND  totals.transactions IS NULL
AND  product.productRevenue is null
GROUP BY month
)

SELECT 
  p1.month
  ,p1.avg_pageviews_purchase
  ,p2.avg_pageviews_non_purchase
FROM purchasers as p1
LEFT JOIN non_purchasers as p2
USING(month)
ORDER BY 1;
~~~

* Query results
  
  <img width="558" height="83" alt="kq4" src="https://github.com/user-attachments/assets/867f53d0-a15b-46c0-8b48-43f1e9223f07" />
**Insights:**
### Task 5:  Average number of transactions per user that made a purchase in July 2017
* SQL code
~~~sql
SELECT
 FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) month
 ,SUM(totals.transactions)/COUNT(DISTINCT fullVisitorId) as avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST (hits) hits,
UNNEST (hits.product)product
WHERE  totals.transactions >=1
AND productRevenue is not null
GROUP BY month;
~~~

* Query results

  <img width="502" height="54" alt="kq5" src="https://github.com/user-attachments/assets/e8fb34f2-58e7-4265-a854-98332af7fc5b" />
**Insights:**

### Task 6: Average amount of money spent per session. Only include purchaser data in July 2017
* SQL code
~~~sql
SELECT  
  FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) month
  , SUM(product.productRevenue)/SUM(totals.visits)/1000000 as avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST (hits) hits,
UNNEST (hits.product)product
WHERE totals.transactions IS NOT NULL
AND product.productRevenue IS NOT NULL
GROUP BY month;
~~~

* Query results

  <img width="502" height="51" alt="kq6" src="https://github.com/user-attachments/assets/b9c1c504-0b49-44d5-ae93-6feb9d261c80" />
**Insights:**

### Task 7: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.
* SQL code
~~~sql
SELECT  
  DISTINCT v2ProductName AS other_purchased_products
  , SUM(productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST (hits) hits,
UNNEST (hits.product)product
WHERE product.productRevenue IS NOT NULL
AND v2ProductName != "YouTube Men's Vintage Henley"
AND fullVisitorId IN
  (SELECT DISTINCT fullVisitorId
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST (hits) hits,
  UNNEST (hits.product)product
  WHERE product.productRevenue IS NOT NULL
  AND v2ProductName = "YouTube Men's Vintage Henley")
GROUP BY v2ProductName
ORDER BY 2 DESC
LIMIT 10;
~~~

* Query results

  <img width="422" height="500" alt="kq7" src="https://github.com/user-attachments/assets/90274667-1048-4bf9-a682-086cf5e20156" />
**Insights:**

### Task 8: Calculate cohort map from pageview to addtocart to purchase in last 3 month.
* SQL code
  
~~~ sql

with
product_view as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_product_view
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '2'
  GROUP BY 1
),

add_to_cart as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_addtocart
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '3'
  GROUP BY 1
),

purchase as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '6'
  and product.productRevenue is not null 
  group by 1
)

select
    pv.*,
    num_addtocart,
    num_purchase,
    round(num_addtocart*100/num_product_view,2) as add_to_cart_rate,
    round(num_purchase*100/num_product_view,2) as purchase_rate
from product_view pv
left join add_to_cart a on pv.month = a.month
left join purchase p on pv.month = p.month
order by pv.month;

~~~
* Query results
  
  <img width="879" height="103" alt="kq8" src="https://github.com/user-attachments/assets/96f26514-0a88-4349-a390-3d7a96499eaa" />
**Insights:**

IV. Conclusion

* This project concluded with a successful SQL exploration of the public Google Analytics eCommerce dataset hosted on Google BigQuery. The resulting data provided actionable business intelligence by quantifying key performance indicators (KPIs) like total visits, pageviews, transactions, bounce rate, and traffic-source-specific revenue. The subsequent step will be to visualize these insights using tools like Power BI or Tableau to highlight key trends for stakeholders. This work clearly demonstrated how big data tools and SQL enable effective analysis and informed decision-making.
