# 📊 Explore E-commerce Performance of Google Merchandise Store | Web Analytics + SQL

**Business Question:** How do users browse, engage, and convert on an e-commerce platform — and what drives revenue?

**Domain:** E-commerce / Digital Marketing Analytics

**Tools Used:** SQL (Google BigQuery)

> 📌 This project applies real-world web analytics queries to the **Google Analytics Sample Dataset** — a publicly available BigQuery dataset (`bigquery-public-data.google_analytics_sample.ga_sessions_2017`) — to extract actionable insights about user behavior, traffic sources, and purchase funnel performance for the Google Merchandise Store.

**Author:** [Your Name]

**Date:** 2025

---

## 📑 Table of Contents

1. [📌 Background & Overview](#-background--overview)
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
3. [⚒️ Main Process](#️-main-process)
4. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview

### Objective

### 📖 What is this project about? What Business Question will it solve?

This project uses **SQL on Google BigQuery** to analyze web session data from the Google Merchandise Store (2017) to:

✔️ Measure traffic volume, pageviews, and transactions over time to understand growth trends.

✔️ Identify which traffic sources generate the most visits and highest bounce rates.

✔️ Compare revenue contribution by traffic source across different time granularities (weekly vs. monthly).

✔️ Understand the behavioral difference between purchasers and non-purchasers through pageview patterns.

✔️ Analyze average transaction value and revenue per session to evaluate monetization efficiency.

✔️ Discover cross-selling opportunities based on co-purchase patterns.

✔️ Map the product conversion funnel (view → add-to-cart → purchase) to identify drop-off points.

### 👤 Who is this project for?

✔️ **E-commerce managers & digital marketers** — to evaluate campaign effectiveness and traffic quality.

✔️ **Data analysts & business analysts** — to understand how to query and derive insights from GA session-level data.

✔️ **Product & growth teams** — to identify funnel bottlenecks and cross-sell opportunities.

---

## 📂 Dataset Description & Data Structure

### 📌 Data Source

- **Source:** Google BigQuery Public Dataset — `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
- **Size:** Multiple partitioned tables by date (daily), covering the full year of 2017
- **Format:** BigQuery native tables (queried via SQL); nested & repeated fields (ARRAY / STRUCT)
- **Access:** Free via Google BigQuery sandbox — no download required

### 📊 Data Structure & Relationships

#### 1️⃣ Tables Used:

This project uses the **`ga_sessions_2017*`** wildcard table, which is a single denormalized table containing session-level data with nested fields. Key nested structures used:

- `hits` — repeated record containing page hits and e-commerce actions per session
- `hits.product` — nested within hits; contains product-level revenue, quantity, and name
- `totals` — session-level aggregated metrics (visits, pageviews, transactions, bounces)
- `trafficSource` — information about where the session originated

#### 2️⃣ Table Schema & Data Snapshot

**Main Table: `ga_sessions`**

👉🏻 *Insert a screenshot of the BigQuery table schema here*

| Column Name | Data Type | Description |
|---|---|---|
| `date` | STRING | Session date in `YYYYMMDD` format |
| `fullVisitorId` | STRING | Unique identifier for each visitor |
| `totals.visits` | INTEGER | Number of sessions |
| `totals.pageviews` | INTEGER | Total pageviews in the session |
| `totals.transactions` | INTEGER | Number of transactions completed |
| `totals.bounces` | INTEGER | 1 if session bounced, NULL otherwise |
| `trafficSource.source` | STRING | Traffic source (e.g., google, direct) |
| `hits.eCommerceAction.action_type` | STRING | E-commerce action (2=view, 3=add to cart, 6=purchase) |
| `hits.product.v2ProductName` | STRING | Product name |
| `hits.product.productRevenue` | INTEGER | Product revenue (in micros, divide by 1,000,000) |
| `hits.product.productQuantity` | INTEGER | Quantity of product ordered |

---

## ⚒️ Main Process

### 1️⃣ Data Cleaning & Preprocessing

- Used `FORMAT_DATE` and `PARSE_DATE` to standardize the `date` STRING field into readable month/week formats.
- Used `UNNEST(hits)` and `UNNEST(hits.product)` to flatten the nested arrays, enabling product-level and hit-level analysis.
- Filtered out sessions with `NULL` revenue or transactions where needed to isolate purchaser behavior.
- Applied `SAFE_DIVIDE` to prevent division-by-zero errors when computing conversion rates.

---

### 2️⃣ SQL Analysis

---

#### 🔍 Query 01: Total Visits, Pageviews & Transactions (Jan–Mar 2017)

This query tracks overall site activity month-over-month for Q1 2017 — a foundational health check to understand how the store's traffic and transaction volume evolved early in the year.

```sql
SELECT
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', CAST(date AS STRING))) AS month
  ,SUM(totals.visits) AS visits
  ,SUM(totals.pageviews) AS pageviews
  ,SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _TABLE_SUFFIX BETWEEN '0101' AND '0331'
GROUP BY 1
ORDER BY 1;
```

👉🏻 *Insert screenshot of query result here*

**📌 Observations:** *(Add your findings — e.g., "Pageviews grew by X% from January to March...")*

---

#### 🔍 Query 02: Bounce Rate per Traffic Source (July 2017)

Bounce rate measures the percentage of sessions where a user visits only one page and leaves without further interaction. A high bounce rate from a specific source may indicate poor landing page relevance or low-intent traffic.

```sql
WITH raw_data AS (
  SELECT
    trafficSource.source AS source
    ,SUM(totals.visits) AS total_visits
    ,COUNT(totals.bounces) AS total_no_of_bounces
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  GROUP BY trafficSource.source
  ORDER BY total_visits DESC
)
SELECT
  source
  ,total_visits
  ,total_no_of_bounces
  ,ROUND(total_no_of_bounces / total_visits * 100, 3) AS bounce_rate
FROM raw_data;
```

👉🏻 *Insert screenshot of query result here*

**📌 Observations:** *(e.g., "Direct traffic had the highest volume but also a significant bounce rate...")*

---

#### 🔍 Query 03: Revenue by Traffic Source — Week vs. Month (June 2017)

By comparing revenue across weekly and monthly granularities, we can identify short-term spikes that a monthly view might obscure — helping marketers evaluate campaign timing.

```sql
WITH month_data AS (
  SELECT
    'Month' AS time_type
    ,FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS time
    ,trafficSource.source AS source
    ,ROUND(SUM(productRevenue / 1000000), 4) AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    UNNEST(hits) hits,
    UNNEST(hits.product) product
  WHERE product.productRevenue IS NOT NULL
  GROUP BY 1, 2, 3
),
week_data AS (
  SELECT
    'Week' AS time_type
    ,FORMAT_DATE("%Y%W", PARSE_DATE("%Y%m%d", date)) AS time
    ,trafficSource.source AS source
    ,ROUND(SUM(productRevenue / 1000000), 4) AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    UNNEST(hits) hits,
    UNNEST(hits.product) product
  WHERE product.productRevenue IS NOT NULL
  GROUP BY 1, 2, 3
)
SELECT * FROM month_data
UNION ALL
SELECT * FROM week_data
ORDER BY time_type;
```

👉🏻 *Insert screenshot of query result here*

**📌 Observations:** *(e.g., "Google (organic search) was the dominant revenue source in June 2017...")*

---

#### 🔍 Query 04: Average Pageviews — Purchasers vs. Non-Purchasers (Jun–Jul 2017)

Comparing browsing depth between purchasers and non-purchasers reveals engagement signals. If purchasers view significantly more pages, content richness and product discovery play a key role in conversion.

```sql
WITH purchasers AS (
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
    ,SUM(totals.pageviews) AS total_pageviews_purchase
    ,COUNT(DISTINCT fullVisitorId) AS unique_users_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST(hits.product) product
  WHERE _TABLE_SUFFIX BETWEEN '0601' AND '0731'
    AND totals.transactions >= 1
    AND productRevenue IS NOT NULL
  GROUP BY month
),
non_purchasers AS (
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
    ,SUM(totals.pageviews) AS total_pageviews_non_purchase
    ,COUNT(DISTINCT fullVisitorId) AS unique_users_non_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST(hits.product) product
  WHERE _TABLE_SUFFIX BETWEEN '0601' AND '0731'
    AND totals.transactions IS NULL
    AND productRevenue IS NULL
  GROUP BY month
)
SELECT
  COALESCE(p.month, np.month) AS month
  ,ROUND(p.total_pageviews_purchase / p.unique_users_purchase, 7) AS avg_pageviews_purchase
  ,ROUND(np.total_pageviews_non_purchase / np.unique_users_non_purchase, 7) AS avg_pageviews_non_purchase
FROM purchasers p
FULL JOIN non_purchasers np ON p.month = np.month
ORDER BY month;
```

👉🏻 *Insert screenshot of query result here*

**📌 Observations:** *(e.g., "Purchasers viewed on average X more pages than non-purchasers...")*

---

#### 🔍 Query 05: Average Transactions per Purchasing User (July 2017)

This metric evaluates customer purchasing frequency. A value above 1 indicates repeat purchases within the same month — a sign of high-value customers or successful upselling.

```sql
SELECT
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
  ,SUM(totals.transactions) / COUNT(DISTINCT fullVisitorId) AS avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
WHERE totals.transactions >= 1
  AND productRevenue IS NOT NULL
GROUP BY month;
```

👉🏻 *Insert screenshot of query result here*

**📌 Observations:** *(e.g., "The average purchasing user completed approximately X transactions in July 2017...")*

---

#### 🔍 Query 06: Average Revenue per Session — Purchasers Only (July 2017)

Revenue per session is a critical monetization metric. By restricting to sessions that resulted in a purchase, we isolate the true spending power of converting visitors — useful for estimating ROI of acquisition campaigns.

```sql
WITH revenue_per_user AS (
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
    ,SUM(productRevenue) / 1000000 AS total_revenue
    ,SUM(totals.visits) AS totals_visits
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST(hits) hits,
    UNNEST(hits.product) product
  WHERE totals.transactions IS NOT NULL
    AND productRevenue IS NOT NULL
  GROUP BY month
)
SELECT
  month
  ,ROUND(total_revenue / totals_visits, 2) AS avg_revenue_by_user_per_visit
FROM revenue_per_user;
```

👉🏻 *Insert screenshot of query result here*

**📌 Observations:** *(e.g., "On average, each purchasing session generated approximately $X in revenue in July 2017...")*

---

#### 🔍 Query 07: Cross-Sell Analysis — Co-purchased Products with "YouTube Men's Vintage Henley" (July 2017)

Understanding what products customers buy alongside a popular item enables data-driven bundling and recommendation strategies. This query identifies the most frequently co-purchased items.

```sql
WITH raw_data AS (
  SELECT
    fullVisitorId
    ,v2ProductName AS product_name
    ,productQuantity AS quantity
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE totals.transactions >= 1
    AND product.productRevenue IS NOT NULL
),
purchased_orders AS (
  SELECT DISTINCT fullVisitorId
  FROM raw_data
  WHERE product_name = "YouTube Men's Vintage Henley"
),
other_purchases AS (
  SELECT
    r.product_name AS other_purchased_products
    ,SUM(r.quantity) AS quantity
  FROM raw_data r
  JOIN purchased_orders p ON r.fullVisitorId = p.fullVisitorId
  WHERE r.product_name != "YouTube Men's Vintage Henley"
  GROUP BY other_purchased_products
)
SELECT *
FROM other_purchases
ORDER BY quantity DESC;
```

👉🏻 *Insert screenshot of query result here*

**📌 Observations:** *(e.g., "The top co-purchased products were X, Y, Z — suggesting strong cross-sell potential...")*

---

#### 🔍 Query 08: Product Conversion Funnel — View → Add to Cart → Purchase (Jan–Mar 2017)

This cohort map tracks how many products were viewed, added to cart, and ultimately purchased each month. It quantifies drop-off at each stage — a key metric for conversion rate optimization (CRO).

| `action_type` value | Meaning |
|---|---|
| `'2'` | Product view |
| `'3'` | Add to cart |
| `'6'` | Purchase |

```sql
WITH cte_view AS (
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
    ,COUNT(product.v2ProductName) AS num_product_view
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE _table_suffix BETWEEN '0101' AND '0331'
    AND eCommerceAction.action_type = '2'
    AND product.v2ProductName IS NOT NULL
  GROUP BY month
),
cte_addtocart AS (
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
    ,COUNT(product.v2ProductName) AS num_add_to_card
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE _table_suffix BETWEEN '0101' AND '0331'
    AND eCommerceAction.action_type = '3'
    AND product.v2ProductName IS NOT NULL
  GROUP BY month
),
cte_purchase AS (
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
    ,COUNT(product.v2ProductName) AS num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE _table_suffix BETWEEN '0101' AND '0331'
    AND eCommerceAction.action_type = '6'
    AND product.productRevenue IS NOT NULL
  GROUP BY month
)
SELECT
  COALESCE(v.month, a.month, p.month) AS month
  ,SUM(v.num_product_view) AS num_product_view
  ,SUM(a.num_add_to_card) AS num_addtocart
  ,SUM(p.num_purchase) AS num_purchase
  ,ROUND(SAFE_DIVIDE(SUM(a.num_add_to_card), SUM(v.num_product_view)) * 100, 2) AS add_to_cart_rate
  ,ROUND(SAFE_DIVIDE(SUM(p.num_purchase), SUM(v.num_product_view)) * 100, 2) AS purchase_rate
FROM cte_view v
LEFT JOIN cte_addtocart a ON v.month = a.month
LEFT JOIN cte_purchase p ON v.month = p.month
GROUP BY month
ORDER BY month;
```

👉🏻 *Insert screenshot of query result here*

**📌 Observations:** *(e.g., "The add-to-cart rate averaged ~X% across Q1 2017, while purchase rate was only ~Y%, indicating a significant drop-off at checkout...")*

---

## 🔎 Final Conclusion & Recommendations

Based on the insights and findings above, we would recommend the **e-commerce & marketing team** to consider the following:

✔️ **Traffic Quality over Quantity** — Certain high-volume sources have disproportionately high bounce rates. Prioritize content optimization for these channels rather than increasing ad spend.

✔️ **Improve Cart-to-Purchase Conversion** — The funnel analysis (Query 08) reveals significant drop-off between add-to-cart and purchase. Consider A/B testing checkout flows, urgency signals, or free shipping thresholds.

✔️ **Leverage Cross-Sell Opportunities** — Products frequently co-purchased with popular items (Query 07) should be surfaced via "frequently bought together" recommendations to increase average order value.

✔️ **Engage High-Pageview Non-Purchasers** — Non-purchasers who browse extensively are high-intent visitors who did not convert. Retargeting campaigns or personalized email nudges could recover this segment.

✔️ **Monitor Revenue per Session as a Core KPI** — Average revenue per purchasing session (Query 06) is a strong proxy for customer value. Track this monthly alongside traffic source to prioritize high-ROI acquisition channels.
