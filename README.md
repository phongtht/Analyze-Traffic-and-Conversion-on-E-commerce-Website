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
I. [Introduction](#-Introduction)  
II. [Data Access](#-Data-Access)  
III. [Exploring Dataset](#-Exploring-Dataset)
IV. [Conclusion](#-Conclusion)

---

## I. Introduction 
My project involves SQL exploration of an eCommerce dataset (from the Google Analytics public data) within Google BigQuery. The data represents an eCommerce website.

## II. Data Access
1. Log in to Google Cloud Platform (GCP) and create a new project.

2. Navigate to the BigQuery console and ensure your new project is selected.

3. In the left navigation panel, click "Add Data" and then select "Search by project name" (or "Search a project" in older UIs).

4. Enter the public project ID: bigquery-public-data and press Enter.

5. In the navigation panel, expand the project bigquery-public-data, then expand the dataset google_analytics_sample.

6. Click on the ga_sessions_* table to view its schema and data.

## III. Exploring Dataset
In this project, I will write 08 query in Bigquery base on Google Analytics dataset
### Task 1: Querying all visits, pageviews, transactions and revenue of website in first quarter of 2017
* SQL code
  
  <img width="648" height="224" alt="query1" src="https://github.com/user-attachments/assets/de5a2023-821e-41cd-9a71-527bebd123d4" />

* Query results
  
  <img width="690" height="120" alt="kq1" src="https://github.com/user-attachments/assets/bbd6e125-0da1-4aa2-b085-48289f132172" />

### Task 2: Bounce rate per traffic source in July 2017
* SQL code
  
  <img width="601" height="144" alt="query2" src="https://github.com/user-attachments/assets/abcc0bef-1b9a-42f2-b107-424af31787bd" />

* Query results
  
  <img width="688" height="499" alt="kq2" src="https://github.com/user-attachments/assets/07a72646-a19e-4ceb-b5a0-93cf454b5940" />

### Task 3:  Revenue by traffic source by week, by month in June 2017
* SQL code
  
  <img width="525" height="639" alt="query3" src="https://github.com/user-attachments/assets/a4d927c4-727d-4763-9d3b-0ea22aff5a84" />

* Query results
  
  <img width="855" height="502" alt="kq3" src="https://github.com/user-attachments/assets/9e13272a-3b75-4440-935a-d90899e3fad4" />

### Task 4: Average number of product pageviews by purchaser type (purchasers vs non-purchasers) in June & July 2017
* SQL code
  
  <img width="730" height="634" alt="query4" src="https://github.com/user-attachments/assets/fecdec07-b4a8-40d2-b49d-d45f613bc97d" />

* Query results
  
  <img width="558" height="83" alt="kq4" src="https://github.com/user-attachments/assets/867f53d0-a15b-46c0-8b48-43f1e9223f07" />

### Task 5:  Average number of transactions per user that made a purchase in July 2017
* SQL code

  <img width="786" height="300" alt="query5" src="https://github.com/user-attachments/assets/9d057c4a-076f-411c-8357-2c935dd7eca1" />


* Query results

  <img width="502" height="54" alt="kq5" src="https://github.com/user-attachments/assets/e8fb34f2-58e7-4265-a854-98332af7fc5b" />


### Task 6: Average amount of money spent per session. Only include purchaser data in July 2017
* SQL code

  <img width="768" height="165" alt="query6" src="https://github.com/user-attachments/assets/1dff6628-b350-49a7-a26d-c41f73be5efb" />


* Query results

  <img width="502" height="51" alt="kq6" src="https://github.com/user-attachments/assets/b9c1c504-0b49-44d5-ae93-6feb9d261c80" />


### Task 7: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.
* SQL code

  <img width="639" height="303" alt="query7" src="https://github.com/user-attachments/assets/3ee9cf70-464c-44ca-95fd-c30509dadfdb" />


* Query results

  <img width="422" height="500" alt="kq7" src="https://github.com/user-attachments/assets/90274667-1048-4bf9-a682-086cf5e20156" />


### Task 8: Calculate cohort map from pageview to addtocart to purchase in last 3 month.
* SQL code
  
~~~
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


IV. Conclusion

* This project concluded with a successful SQL exploration of the public Google Analytics eCommerce dataset hosted on Google BigQuery. The resulting data provided actionable business intelligence by quantifying key performance indicators (KPIs) like total visits, pageviews, transactions, bounce rate, and traffic-source-specific revenue. The subsequent step will be to visualize these insights using tools like Power BI or Tableau to highlight key trends for stakeholders. This work clearly demonstrated how big data tools and SQL enable effective analysis and informed decision-making.
