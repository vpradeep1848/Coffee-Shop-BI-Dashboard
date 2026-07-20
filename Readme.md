# Coffee Shop Performance Dashboard — Documentation

## 1. Project Overview

This Power BI dashboard analyzes transactional sales data from a coffee shop to surface key
performance metrics, spending trends, and customer buying patterns. It is built on a **single
flat fact table** (`coffee_data`) containing one row per transaction, enriched with derived
time-based columns and custom DAX measures.

**Core objectives:**
- Track overall sales performance (total spend, cups sold)
- Identify peak sales hours to support staffing/inventory decisions
- Compare weekday vs. weekend spending behavior
- Break down sales by time of day and day of week
- Visualize spending trend over a 12+ month period

---

## 2. Data Model

### Table: `coffee_data`

| Column | Type | Description |
|---|---|---|
| `hour_of_day` | Whole Number | Hour (24h) the transaction occurred |
| `cash_type` | Text | Payment method (e.g., card) |
| `money` (Rs) | Decimal | Transaction amount in ₹ |
| `coffee_name` | Text | Product purchased (Americano, Latte, etc.) |
| `Time_of_Day` | Text | Derived bucket: Morning / Afternoon / Night |
| `Weekday` | Text | Day name (Mon–Sun) |
| `Month_name` | Text | Month name (Jan–Dec) |
| `Weekdaysort` | Whole Number | Numeric key for correct weekday ordering in visuals |
| `Monthsort` | Whole Number | Numeric key for correct month ordering in visuals |
| `Date` | Date | Transaction date |
| `Time` | Text/Time | Transaction timestamp (minute:second) |

**Model type:** Single-table (flat) model — no separate dimension tables (e.g., no dedicated
Date table or Product table). All grouping/sorting is handled via helper columns
(`Weekdaysort`, `Monthsort`) already present in the source data.

> *Note for documentation completeness: a natural enhancement would be to split this into a
> star schema (separate Date, Product, and Payment dimension tables) — worth mentioning as a
> "future improvement" section if you want to show data modeling maturity.*

---

## 3. Measures (DAX)

### `Total_Spend`
```dax
Total_Spend = SUM(coffee_data[money])
```
Sums the `money` column across all transactions. Powers the "Total Spend" KPI card (₹24.75K).

---

### `Total_Cup_Sold`
```dax
Total_Cup_Sold = COUNT(coffee_data[coffee_name])
```
Counts total transactions (each row = one cup sold). Powers the "Total Cups Sold" KPI card (809).

---

### `Peak_Hour_Spend`
```dax
Peak_Hour_Spend =
VAR HoursSpend =
    SUMMARIZE(
        'coffee_data',
        'coffee_data'[hour_of_day],
        "Total_Spend", SUM(coffee_data[money])
    )
VAR TopHour =
    TOPN(1, HoursSpend, [Total_Spend], DESC)
RETURN
    MAXX(TopHour, [Total_Spend])
```
**Logic:** Groups all transactions by `hour_of_day` and computes total spend per hour using
`SUMMARIZE`. `TOPN(1, ..., DESC)` isolates the single highest-spending hour, and `MAXX`
extracts that hour's spend value. Powers the "Peak Hour Spend" KPI (₹3.11K).

---

### `Peak_Hour_Cups`
```dax
Peak_Hour_Cups =
VAR HourCups =
    SUMMARIZE(
        'coffee_data',
        'coffee_data'[hour_of_day],
        "TotalCups", COUNTROWS('coffee_data')
    )
VAR TopHour =
    TOPN(1, HourCups, [TotalCups], DESC)
RETURN
    MAXX(TopHour, [TotalCups])
```
**Logic:** Same pattern as `Peak_Hour_Spend`, but aggregates cup count (`COUNTROWS`) per hour
instead of spend, then extracts the top hour's cup count. Powers "Peak Hour Cups" KPI (101).

---

### `Peak_Hour`
```dax
Peak_Hour =
VAR HourTable =
    SUMMARIZE(
        'coffee_data',
        'coffee_data'[hour_of_day],
        "Cups", COUNT(coffee_data[coffee_name])
    )
VAR MAXCups =
    MAXX(HourTable, [Cups])
VAR PeakHour =
    MAXX(FILTER(HourTable, [Cups] = MAXCups), [hour_of_day])
RETURN
    PeakHour
```
**Logic:** Builds an hour-by-hour cup count table, finds the maximum cup count (`MAXCups`),
then filters back to find *which hour* achieved that max and returns the hour value itself
(rather than the count). Powers the "Peak Hour Time" label (10 AM).

> *Difference from `Peak_Hour_Cups`: that measure returns the count at the peak hour; this one
> returns the hour label itself — a subtle but important distinction worth calling out in an
> interview.*

---

### `Weekday_Average_Spend`
```dax
Weekday_Average_Spend =
AVERAGEX(
    FILTER(
        VALUES('coffee_data'[Weekday]),
        'coffee_data'[Weekday] IN {"Mon", "Tue", "Wed", "Thu", "Fri"}
    ),
    CALCULATE([Total_Spend])
)
```
**Logic:** `VALUES` returns distinct weekdays; `FILTER` restricts to Mon–Fri; `AVERAGEX`
iterates over those five weekday values and calculates `Total_Spend` for each (via
`CALCULATE` context transition), then averages the five daily totals. Powers "Weekday Average
Spend" (₹4K).

---

### `Weekend_Average_Spend`
```dax
Weekend_Average_Spend =
AVERAGEX(
    FILTER(
        VALUES(coffee_data[Weekday]),
        'coffee_data'[Weekday] IN {"Sat", "Sun"}
    ),
    CALCULATE([Total_Spend])
)
```
Same pattern as above, restricted to Sat/Sun. Powers "Weekend Average Spend" (₹3K).

---

### `Coffee_Images`
```dax
Coffee_Images =
SWITCH(
    MAX('coffee_data'[coffee_name]),
    "Americano", "<GitHub raw image URL>",
    "Americano with milk", "<GitHub raw image URL>",
    "Cappuccino", "<GitHub raw image URL>",
    "Cocoa", "<GitHub raw image URL>",
    "Cortado", "<GitHub raw image URL>",
    "Espresso", "<GitHub raw image URL>",
    "Hot Chocolate", "<GitHub raw image URL>",
    "Latte", "<GitHub raw image URL>"
)
```
**Logic:** Maps each `coffee_name` to a hosted product image URL (stored on GitHub), enabling
dynamic image display per selected product card — used to render the coffee product tiles at
the top of the dashboard and the "featured item" images.

---

## 4. Visual-to-Measure Mapping

| Dashboard Element | Measure(s) Used |
|---|---|
| Total Spend KPI card | `Total_Spend` |
| Total Cups Sold KPI card | `Total_Cup_Sold` |
| Peak Hour Spend KPI card | `Peak_Hour_Spend` |
| Peak Hour Cups KPI card | `Peak_Hour_Cups` |
| Peak Hour Time label | `Peak_Hour` |
| Weekday Average Spend | `Weekday_Average_Spend` |
| Weekend Average Spend | `Weekend_Average_Spend` |
| Product image tiles | `Coffee_Images` |
| Spending Trend Over Time (line chart) | `Total_Spend` by `Date`/`Month_name` (sorted via `Monthsort`) |
| Total Spend by Day of Week (bar chart) | `Total_Spend` by `Weekday` (sorted via `Weekdaysort`) |
| Total Spend by Time Period (donut chart) | `Total_Spend` by `Time_of_Day` |
| Product filter tiles (Americano, Latte, etc.) | Slicer on `coffee_name` |

---

## 5. Key Insights Surfaced by the Dashboard

- **Total revenue:** ₹24.75K from 809 cups sold over the analyzed period.
- **Peak trading hour:** 10 AM, generating ₹3.11K in spend and 101 cups — a clear signal for
  staffing and inventory prep ahead of the morning rush.
- **Weekday vs. weekend:** Weekdays outperform weekends (₹4K vs. ₹3K average spend),
  suggesting a commuter/office-driven customer base rather than leisure-weekend traffic.
- **Day-of-week pattern:** Tuesday leads (₹4.3K), with a mid-week dip on Wednesday/Thursday
  and a Saturday rebound (₹3.7K).
- **Time-of-day split:** Spend is fairly evenly distributed across Morning (₹10.03K),
  Afternoon (₹7.34K), and Night (₹7.38K), though mornings edge out slightly.
- **Trend over time:** Spending is volatile month-to-month, with a notable peak in Sep 2024
  (₹2.9K) and a low in Mar 2024 (₹1.0K) — worth investigating seasonality or promotions.

---

## 6. Possible Future Enhancements (optional section to mention in interviews)

- Convert to a proper star schema (separate Date, Product dimension tables) for scalability.
- Add a dedicated calendar/date table with `MONTH()`/`WEEKDAY()` DAX functions instead of
  relying on pre-computed sort columns in the source data.
- Add YoY / MoM growth measures using time-intelligence functions (`DATEADD`, `SAMEPERIODLASTYEAR`).
- Parameterize the peak-hour logic to support "peak N hours" instead of just the single top hour.