# Shop Sales and Profit Dashboard

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black)
![Data](https://img.shields.io/badge/Data-CSV-informational)
![Period](https://img.shields.io/badge/Period-2018-blue)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

An interactive Power BI dashboard that joins order and order-line data from a multi-category retail shop to show where sales turn into profit, and where they do not.

![Sales Dashboard](images/dashboard.png)

_Dashboard shown with the Qtr 1 slicer selected._

---

## Table of contents

- [Overview](#overview)
- [Key results](#key-results)
- [Key insights](#key-insights)
- [Dashboard features](#dashboard-features)
- [Dataset](#dataset)
- [Data model](#data-model)
- [Metrics](#metrics)
- [Data preparation](#data-preparation)
- [Repository structure](#repository-structure)
- [How to run](#how-to-run)
- [Limitations](#limitations)
- [Tools used](#tools-used)
- [Author](#author)

---

## Overview

The shop's order headers (dates, customers, places) and order details (amounts, profit, quantity, products) sit in two separate files. On their own, neither shows whether strong sales actually produce profit.

This project loads both files into Power BI, relates them by `Order ID`, and builds a single dashboard for sales, profit and quantity by product, payment mode, customer and time.

**Business question:** which products, periods and payment modes earn their sales, and which lose money?

## Key results

Full year 2018, all 19 states, no slicer applied.

| Metric              | Value       |
| ------------------- | ----------- |
| Total Sales         | ₹437,771    |
| Total Profit        | ₹36,963     |
| Total Quantity      | 5,615 units |
| Profit Margin       | 8.44%       |
| Total Orders        | 500         |
| Average Order Value | ₹875.54     |

## Key insights

- **Over a third of order lines lose money.** 529 of 1,500 lines (35%) made a loss, totalling ₹38,079.
- **High sales do not mean high profit.** Electronics has the highest sales (₹166,267, 38%), but Clothing earned slightly more profit (₹13,325 vs ₹13,162) thanks to a better margin (9.2% vs 7.9%).
- **Five sub-categories lose money overall:** Furnishings, Electronic Games, Kurti, Skirt and Leggings. Electronic Games brought in ₹39,168 in sales at a net loss of ₹644.
- **Top profit sources:** Printers (₹8,606), Bookcases (₹6,516) and Saree (₹4,057).
- **Profit swings sharply by quarter:** ₹25,942 in Q1 (16.1% margin), ₹882 in Q2, −₹1,469 in Q3 and ₹11,608 in Q4 (9.9%).
- **Two states dominate sales.** Maharashtra and Madhya Pradesh together provide about 43% of sales. Rajasthan and Andhra Pradesh both ran at a small loss.
- **Payment mode and margin:** cash on delivery is the largest mode (35.4% of sales). Credit card has the best margin (14.5%) and UPI the lowest (4.8%).

These are observations from one year of data, not proof of cause. See [Limitations](#limitations).

## Dashboard features

| Visual                           | What it shows                       |
| -------------------------------- | ----------------------------------- |
| Quarter buttons (Qtr 1 to Qtr 4) | Filter the whole page by quarter    |
| KPI cards                        | Sum of Quantity, Profit and Amount  |
| Bar chart                        | Sum of Amount by Sub-Category       |
| Bar chart                        | Sum of Profit by Sub-Category       |
| Donut chart                      | Quantity and Amount by Payment Mode |
| Donut chart                      | Quantity and Profit by Category     |
| Column chart                     | Profit by Customer Name             |
| Line chart                       | Quantity by Month                   |

## Dataset

Two CSV files covering 1 January to 31 December 2018.

**`Orders.csv`** (500 rows, one per order)

| Column       | Description              |
| ------------ | ------------------------ |
| Order ID     | Unique order identifier  |
| Order Date   | Order date, `dd-mm-yyyy` |
| CustomerName | Customer name            |
| State        | Customer state           |
| City         | Customer city            |

**`Details.csv`** (1,500 rows, one per product line)

| Column       | Description                              |
| ------------ | ---------------------------------------- |
| Order ID     | Links to Orders                          |
| Amount       | Line sales value                         |
| Profit       | Line profit (negative for a loss)        |
| Quantity     | Units sold                               |
| Category     | Clothing, Electronics or Furniture       |
| Sub-Category | 17 sub-categories                        |
| PaymentMode  | COD, Credit Card, Debit Card, EMI or UPI |

The files do not state a currency. Amounts are shown in rupees (₹) because all customers are in India.

## Data model

```
Orders (1)  ───────────<  Details (many)
 Order ID                   Order ID
```

One order has many detail lines. Filters flow from Orders to Details.

Source checks: 500 unique Order IDs in Orders, 1,500 rows in Details covering the same 500 IDs, no unmatched IDs in either direction, and no blank values.

## Metrics

| Metric              | Definition                 |
| ------------------- | -------------------------- |
| Total Sales         | Sum of Amount              |
| Total Profit        | Sum of Profit              |
| Total Quantity      | Sum of Quantity            |
| Profit Margin       | Total Profit / Total Sales |
| Total Orders        | Distinct count of Order ID |
| Average Order Value | Total Sales / Total Orders |

The dashboard cards use implicit sums. Profit margin, order count and average order value are calculated from the same data and can be added as DAX measures:

```DAX
Profit Margin = DIVIDE ( SUM ( Details[Profit] ), SUM ( Details[Amount] ) )

Total Orders = DISTINCTCOUNT ( Details[Order ID] )

Average Order Value = DIVIDE ( SUM ( Details[Amount] ), DISTINCTCOUNT ( Details[Order ID] ) )
```

## Data preparation

1. Load `Orders.csv` and `Details.csv` with Power Query.
2. Set text columns (Order ID, CustomerName, State, City, Category, Sub-Category, PaymentMode) to **Text**.
3. Set `Order Date` to **Date using locale** and choose **English (India)** or **English (United Kingdom)**. The dates are day-month-year, and a US locale will misread any day above 12.
4. Set Amount, Profit and Quantity to **Whole Number**.
5. Keep negative Profit values. They are genuine losses, not errors.
6. Create the one-to-many relationship on `Order ID`.

## Repository structure

```
.
├── README.md
├── Sales-Dashboard-shop.pbix        Power BI report
├── data/
│   ├── Orders.csv
│   └── Details.csv
├── images/
│   └── dashboard.png                Dashboard screenshot
└── report/
    └── Shop_Sales_Project_Report.docx   Full project report
```

## How to run

1. Clone or download this repository.
2. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (Windows).
3. Open `Sales-Dashboard-shop.pbix`.
4. If Power BI cannot find the data, go to **Home > Transform data > Data source settings**, choose **Change Source** and point both sources to the files in the `data/` folder.
5. Use the quarter buttons to filter the dashboard.

## Limitations

- **One year of data.** Quarterly and monthly patterns cannot be called seasonality without more years.
- **Profit is as supplied.** The files have no cost or discount fields, so profit cannot be reconciled or explained.
- **Customers are identified by name only.** The file has 336 distinct names across 500 orders, and two people with the same name would be counted as one.
- **Currency is assumed** to be rupees.
- **Tax and returns are unknown.** The data does not show whether Amount includes tax or is net of returns.

## Tools used

- Power BI Desktop
- Power Query
- DAX
- CSV data files
