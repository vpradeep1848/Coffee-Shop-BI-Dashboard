# ☕ Coffee Shop Performance Dashboard

> A Power BI dashboard analyzing coffee shop sales transactions — surfacing revenue trends, peak sales hours, and customer buying patterns to support data-driven staffing and inventory decisions.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Data%20Modeling-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

---

## 📊 Overview

This dashboard is built on transactional coffee shop sales data and delivers a single-page view of:

- 💰 Total revenue & cups sold
- ⏰ Peak sales hour detection
- 📅 Weekday vs. weekend spending comparison
- 📈 Month-over-month spending trend
- 🌗 Time-of-day (Morning / Afternoon / Night) breakdown

---

## 🗃️ Data Model

**Table:** `coffee_data` — a single flat transactional table (one row per sale)

| 🏷️ Column | 🔤 Type | 📝 Description |
|---|---|---|
| `hour_of_day` | Number | Hour (24h) transaction occurred |
| `cash_type` | Text | Payment method |
| `money (Rs)` | Decimal | Transaction amount (₹) |
| `coffee_name` | Text | Product purchased |
| `Time_of_Day` | Text | Morning / Afternoon / Night |
| `Weekday` | Text | Day name (Mon–Sun) |
| `Month_name` | Text | Month name |
| `Weekdaysort` | Number | Ordering key for weekdays |
| `Monthsort` | Number | Ordering key for months |
| `Date` | Date | Transaction date |
| `Time` | Time | Transaction timestamp |

> 🔧 **Model type:** Single-table (flat) model — grouping and sorting handled via helper columns rather than separate dimension tables.

---

## 🧮 Key Calculations

| ⚙️ Metric | 💡 What it does |
|---|---|
| 💵 **Total Spend** | Sums revenue across all transactions |
| ☕ **Total Cups Sold** | Counts total transactions |
| 🔥 **Peak Hour Spend** | Finds the single hour with highest revenue |
| 🔥 **Peak Hour Cups** | Finds the single hour with highest cup volume |
| ⏰ **Peak Hour** | Returns *which* hour had the most cups sold |
| 📆 **Weekday Average Spend** | Average daily spend, Mon–Fri |
| 🛌 **Weekend Average Spend** | Average daily spend, Sat–Sun |
| 🖼️ **Coffee Images** | Dynamically maps each product to its image for the product tiles |

---

## 🖥️ Dashboard Visuals

| 📌 Visual | 🔗 Data Source |
|---|---|
| 💰 Total Spend / ☕ Total Cups (KPI cards) | Total Spend, Total Cups Sold |
| 🔥 Peak Hour Spend / Cups / Time | Peak Hour measures |
| 📈 Spending Trend Over Time (line chart) | Spend by Month |
| 📊 Total Spend by Day of Week (bar chart) | Spend by Weekday |
| 🍩 Total Spend by Time Period (donut chart) | Spend by Morning/Afternoon/Night |
| 🧋 Product tiles | Coffee Images + product slicer |

---

## 🔍 Key Insights

- 🧑‍💼 **Commuter-driven traffic:** Weekdays outspend weekends (₹4K vs ₹3K avg) — pointing to a routine, office-goer customer base rather than weekend social buyers.
- ⏰ **Morning rush is king:** 10 AM is the single busiest hour, generating ₹3.11K and 101 cups — the clearest signal for where to concentrate staffing and stock.
- 📅 **Tuesday leads the week** (₹4.3K), with a mid-week dip on Wed/Thu, before a Saturday rebound (₹3.7K) outshining Sunday.
- 🌗 **Balanced daily rhythm:** Morning (₹10.03K), Afternoon (₹7.34K), and Night (₹7.38K) spend are close — steady traffic all day rather than one dominant slot.
- 📉📈 **Volatile monthly trend:** Lowest spend in March 2024 (₹1.0K), highest in September 2024 (₹2.9K) — a swing worth investigating further (seasonality, promotions, or data quality).

---

## 🚀 Future Enhancements

- [ ] Convert to a proper star schema (separate Date & Product dimension tables)
- [ ] Add a dedicated calendar table with time-intelligence support
- [ ] Add YoY / MoM growth tracking
- [ ] Parameterize "peak hours" to support top-N hours, not just the single peak

---

## 🛠️ Tools Used

`Power BI` · `DAX` · `Data Modeling` · `Data Visualization`

---

## 📷 Dashboard Preview

*(Add a screenshot of your dashboard here — e.g. `![dashboard](./dashboard-preview.png)`)*