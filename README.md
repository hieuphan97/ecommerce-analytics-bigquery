# 📊 E-commerce Performance Analysis – Google Analytics 2017 | BigQuery SQL

*Analyze website traffic, revenue performance & user behavior using GA Sessions 2017 dataset on BigQuery*

---

**+ Business question:**
How is website traffic converting into revenue, and which traffic sources & user behaviors drive business performance?

**+ Domain:**
E-commerce / Digital Marketing Analytics

---

**Author:** [Phan Trung Hiếu]
**Date:** 12/09/2025
**Tools Used:** Google BigQuery (Standard SQL)

---

## 📑 Table of Contents

1. [📌 Background & Overview](#-background--overview)
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
3. [⚒️ Main Process](#-main-process)
4. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

# 📌 Background & Overview

## 🎯 Objective

This project analyzes the public dataset `ga_sessions_2017` from BigQuery to:

* Evaluate website traffic performance (visits, pageviews, transactions)
* Measure marketing channel effectiveness (bounce rate, revenue by source)
* Analyze purchasing behavior differences (purchasers vs non-purchasers)
* Understand conversion funnel performance (view → add to cart → purchase)
* Identify cross-selling opportunities

---

## 📖 Business Questions

1. How did traffic and transactions perform in Q1 2017?
2. Which traffic sources had the highest bounce rate?
3. How much revenue did each traffic source generate (weekly & monthly)?
4. Do purchasers behave differently from non-purchasers?
5. What is the average transaction frequency per user?
6. What is the average revenue per session?
7. What products are frequently co-purchased?
8. What is the conversion rate from product view → add to cart → purchase?

---

## 👤 Who is this project for?

✔️ Digital Marketing Analysts
✔️ E-commerce Managers
✔️ Growth & Performance Teams
✔️ BI & Data Analysts

---

# 📂 Dataset Description & Data Structure

## 📌 Data Source

* **Source:** Public dataset on Google BigQuery
* **Dataset:** `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
* **Type:** Partitioned tables (daily session-level data)
* **Format:** Nested & Repeated fields (STRUCT + ARRAY)

---

## 📊 Data Structure & Relationships

### 1️⃣ Tables Used

Only 1 main logical dataset:

* `ga_sessions_2017*` (daily sharded tables)

However, it contains nested structures:

* `totals`
* `trafficSource`
* `hits`
* `hits.product`
* `eCommerceAction`

This requires `UNNEST()` to flatten product-level data.

---

### 2️⃣ Key Fields Used

| Column                      | Description                 |
| --------------------------- | --------------------------- |
| date                        | Session date                |
| fullVisitorId               | Unique user identifier      |
| totals.visits               | Number of visits            |
| totals.pageviews            | Number of pageviews         |
| totals.transactions         | Number of transactions      |
| productRevenue              | Revenue (micro-units)       |
| trafficSource.source        | Traffic acquisition channel |
| eCommerceAction.action_type | Funnel stage indicator      |

---

# ⚒️ Main Process

---

## 1️⃣ Traffic Overview (Q1 2017)

### Purpose

Evaluate traffic trend and transaction volume in Jan–Mar 2017.

### What was done

* Aggregated visits, pageviews, transactions
* Grouped by month
* Ordered chronologically

### Insight Direction

* Identify growth/decline trend
* Compare traffic vs transaction alignment

---

## 2️⃣ Bounce Rate by Traffic Source (July 2017)

### Definition

Bounce Rate =
(Number of Bounces / Total Visits) × 100

A high bounce rate indicates poor landing page engagement or low traffic quality.

### What was done

* Aggregated visits per source
* Counted bounces
* Calculated bounce rate
* Ranked by total visits

### Insight Direction

* Identify high-volume but low-quality traffic
* Evaluate marketing channel effectiveness

---

## 3️⃣ Revenue by Source (Week & Month – June 2017)

### What was done

* Used `UNNEST(hits)` and `UNNEST(hits.product)`
* Aggregated productRevenue
* Calculated revenue per:

  * Month
  * ISO Week

### Insight Direction

* Detect revenue concentration by channel
* Identify short-term revenue spikes

---

## 4️⃣ Purchasers vs Non-Purchasers Behavior

### Metric

Average Pageviews per User

### Logic

* Purchasers: `transactions >= 1`
* Non-purchasers: `transactions IS NULL`
* Calculated:

  * Total pageviews
  * Unique users
  * Average per user

### Insight Direction

* Measure engagement gap
* Validate hypothesis:

  > Purchasers view more pages before converting

---

## 5️⃣ Avg Transactions per Purchasing User (July 2017)

### Purpose

Measure purchase frequency per buyer.

### Metric

Avg Transactions =
Total Transactions / Distinct Purchasing Users

### Insight Direction

* Understand repeat purchase intensity
* Evaluate customer value concentration

---

## 6️⃣ Average Revenue per Session (Purchasers Only)

### Purpose

Measure monetization efficiency per visit.

### Metric

Avg Revenue per Visit =
Total Revenue / Total Visits

### Insight Direction

* Evaluate traffic monetization quality
* Compare with industry benchmarks

---

## 7️⃣ Cross-Selling Analysis

### Business Question

Customers who purchased:
**"YouTube Men's Vintage Henley"**

→ What other products did they buy?

### Approach

* Identify users who bought the target product
* Join back to purchase table
* Aggregate other products by quantity

### Insight Direction

* Identify bundling opportunities
* Support recommendation system logic

---

## 8️⃣ Conversion Funnel Analysis (Q1 2017)

### Funnel Stages

| Action Type | Meaning      |
| ----------- | ------------ |
| 2           | Product View |
| 3           | Add to Cart  |
| 6           | Purchase     |

### Metrics

* Add to Cart Rate = AddToCart / Product View
* Purchase Rate = Purchase / Product View

### Insight Direction

* Detect drop-off points
* Identify UX / pricing / trust friction

---

# 🔎 Final Conclusion & Recommendations

## 🔍 Key Observations

1. Traffic volume does not always correlate with transaction volume.
2. Certain traffic sources generate high visits but high bounce rates.
3. Purchasers show significantly higher engagement (pageviews).
4. Conversion funnel shows major drop-off at Add-to-Cart stage.
5. Cross-selling opportunity exists around hero products.

---

## 📈 Business Recommendations

### 🎯 For Marketing Team

* Reduce budget allocation for high-bounce sources
* Scale high-revenue traffic channels

### 🛒 For E-commerce Team

* Optimize product detail pages (reduce drop-off)
* Improve Add-to-Cart UX
* Test checkout flow (A/B testing)

### 📦 For Growth Team

* Implement product bundling strategy
* Deploy recommendation engine logic
* Target high-engagement non-purchasers via remarketing

---

# 📌 Project Value

This project demonstrates:

* Advanced SQL (CTE, UNNEST, SAFE_DIVIDE, Window functions)
* Funnel analysis
* Behavioral segmentation
* Marketing performance analytics
* Business-driven analytical thinking

---

Nếu bạn muốn, tôi có thể:

* Viết thêm **phần Technical Architecture (BigQuery cost optimization, partition strategy, query performance)**
* Hoặc giúp bạn nâng cấp README lên level “Senior-ready portfolio” với ERD diagram + KPI framework 🚀
