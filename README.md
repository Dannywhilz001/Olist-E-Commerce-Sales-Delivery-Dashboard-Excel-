# Olist E-Commerce Sales & Delivery Dashboard (Excel)

An end-to-end business analysis of the Olist Brazilian E-Commerce dataset, built entirely in **Excel**. Seven raw CSV files were cleaned and merged with **Power Query**, analysed with **PivotTables**, and presented in an interactive, non-technical **dashboard** with KPI cards and slicers.

![Dashboard preview](./dashboard.png)

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Business Questions](#business-questions)
3. [Dataset](#dataset)
4. [Tools Used](#tools-used)
5. [Data Preparation](#data-preparation)
6. [Final Data Model](#final-data-model)
7. [Dashboard Features](#dashboard-features)
8. [Key Findings](#key-findings)
9. [Challenges and Lessons Learned](#challenges-and-lessons-learned)
10. [Limitations](#limitations)
11. [How to Use This Project](#how-to-use-this-project)
12. [Repository Structure](#repository-structure)
13. [About the Author](#about-the-author)

---

## Project Overview

Olist is a Brazilian marketplace that connects small businesses to customers. The public dataset covers about **112.7K order items** across 2016 to 2018, with details on orders, customers, products, payments, delivery times, and reviews.

The goal of this project was to turn that raw, multi-table data into a clear dashboard that a non-technical manager could use to understand sales performance, delivery reliability, and customer satisfaction.

## Business Questions

- How has revenue changed over time?
- Which product categories generate the most revenue?
- Which states drive the most sales and orders?
- How reliable is delivery, and where do late deliveries happen most?
- How satisfied are customers, and which categories are rated best and worst?

## Dataset

- **Source:** [Olist Brazilian E-Commerce Public Dataset on Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Files used (7 of 9):**
  - `olist_orders_dataset.csv`
  - `olist_order_items_dataset.csv`
  - `olist_customers_dataset.csv`
  - `olist_products_dataset.csv`
  - `olist_order_payments_dataset.csv`
  - `olist_order_reviews_dataset.csv`
  - `product_category_name_translation.csv`
- **Not used:** `olist_geolocation_dataset.csv` (about 1 million rows, not needed for this analysis) and `olist_sellers_dataset.csv`

## Tools Used

| Tool | Purpose |
|---|---|
| Excel 2021 | Analysis and dashboard |
| Power Query | Importing, cleaning, and merging tables |
| PivotTables / PivotCharts | Aggregation and charts |
| Data Model (Distinct Count) | Counting unique orders |
| Slicers | Interactive filtering |

## Data Preparation

All preparation was done in Power Query, so the process is repeatable and the raw files are never edited.

1. **Imported** the seven CSV files as separate queries.
2. **Prepared one-to-many tables before merging**
   - Payments were grouped by `order_id` to get one payment total per order.
   - Reviews were reduced to `order_id` and `review_score`, with duplicates removed by `order_id`.
   - Without this step, merging would have duplicated rows and inflated revenue.
3. **Merged everything into one table** (`Sales_Data`) using Left Outer joins:
   - Order items + Orders (on `order_id`)
   - + Customers (on `customer_id`)
   - + Products (on `product_id`)
   - + Category translation (on `product_category_name`), so category names are in English
   - + Grouped payments and Reviews (on `order_id`)
4. **Created calculated columns**

   | Column | Logic |
   |---|---|
   | Revenue | Price + Freight |
   | Order Month | Start of the month of the purchase date |
   | Delivery Days | Delivered date minus purchase date |
   | Delay Days | Delivered date minus estimated delivery date |
   | Late Delivery | 1 if Delay Days is greater than 0, otherwise 0 |

5. **Cleaned the data**
   - Replaced underscores in category names and applied proper case
   - Set correct data types (dates, decimals, whole numbers)
   - Renamed columns to readable names and removed columns that weren't needed

## Final Data Model

One flat table, `Sales_Data`, with about 112.7K rows (one row per order item).

| Column | Description |
|---|---|
| Order ID | Unique order identifier |
| Item | Item number within the order |
| Price / Freight | Item price and shipping cost |
| Order Status | Delivered, shipped, canceled, and so on |
| Purchase Date / Delivered Date / Estimated Delivery Date | Order timeline |
| City / State | Customer location |
| Category | Product category (English) |
| Review Score | Customer rating from 1 to 5 |
| Revenue | Price + Freight |
| Order Month | Month of purchase |
| Delivery Days / Delay Days | Delivery performance measures |
| Late Delivery | 1 if delivered after the estimated date |
| Payment Total | Total payment for the whole order (repeats on every item row, so it is not summed) |

## Dashboard Features

- **KPI cards:** Total Revenue, Orders, Average Order Value, Average Review Score, Late Delivery %
- **Revenue trend** over time
- **Top 10 product categories** by revenue
- **Revenue and orders by state**
- **Late delivery rate by state**
- **Review score distribution**
- **Interactive slicers** for State, Year, and Category, connected to every chart and KPI

## Key Findings

| Metric | Value |
|---|---|
| Total orders | about 112.7K |
| Total revenue | €15.8M |
| Average order value | €140.64 |
| Average review score | 4.03 / 5 |
| Late delivery rate | 6.59% |

- **São Paulo (SP)** is the largest state by revenue by a wide margin, followed by RJ and MG.
- **Health Beauty** is the top-earning product category, followed by Watches Gifts and Bed Bath Table.
- Most customers give **5-star reviews**, though a visible share give 1 star, which is worth investigating alongside late deliveries.
- Revenue grew through 2017 and into 2018. The drop at the end of the trend line should be read with caution, because the final months of the dataset are incomplete.

## Challenges and Lessons Learned

- **Avoiding double counting:** merging payments and reviews directly created duplicate rows. Grouping and de-duplicating before the merge fixed it.
- **Counting orders correctly:** one order can contain several items, so orders are counted with **Distinct Count of Order ID**, not a row count.
- **Choosing the right revenue field:** `Payment Total` repeats on every item row, so summing it would overcount. `Revenue` (price + freight) is used for all totals.
- **Data types matter:** a rounding step earlier in the query turned revenue into whole numbers. Rebuilding the column as a decimal fixed it.
- **Readable categories:** merging the translation table replaced Portuguese category names with English ones.
- **Dashboard readability:** with 27 states, charts were cluttered, so they were filtered to the top states or formatted as horizontal bars.

## Limitations

- Review score is averaged per item row, not per order, so multi-item orders weigh slightly more.
- Late Delivery % is also calculated per item row.
- Early and late months in the dataset are incomplete and may distort the trend.
- Currency is Brazilian real (R$), and no conversion was applied.

## How to Use This Project

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and place the CSV files in a `data/` folder.
2. Open `Olist_Dashboard.xlsx` in Excel 2021 or later.
3. Go to **Data > Refresh All** if you want to reload the queries (you may need to update the file paths under **Data > Get Data > Data Source Settings**).
4. Use the **State**, **Year**, and **Category** slicers on the Dashboard sheet to explore the data.

## Repository Structure

```
olist-excel-dashboard/
├── README.md
├── Olist_Dashboard.xlsx
├── images/
│   └── dashboard.png
└── data/
    └── (raw CSV files, or a link to Kaggle)
```

## About the Author

**[Oladotun Olawale Daniel]**
Aspiring Business / Data Analyst based in Lagos, Nigeria

- ![LinkedIn](https://www.linkedin.com/in/oladotun-olawale)
- Email: oladotunolawale29@yahoo.com
- ![Portfolio](https://github.com/Dannywhilz001)

If you found this project useful, please give it a star. ⭐
