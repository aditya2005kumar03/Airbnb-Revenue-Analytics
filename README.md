# 🏠 Airbnb Revenue Analytics

An end-to-end Power BI dashboard analyzing 100,000+ simulated Airbnb booking records across New York City, covering revenue performance, occupancy, host quality, and neighbourhood trends. Built as a full data engineering + BI workflow, from raw CSV to a polished, dark-themed two-page report.

---

## 📊 Dashboard Preview

**Page 1 : Executive Overview**

| Total Revenue | Avg Nightly Rate | Occupancy Rate | Avg Review Score | Cancellation Rate |
|---|---|---|---|---|
| $176.08M (▲ +47.4% YoY) | $235.56 (▼ -0.3% YoY) | 48.8% (▼ -0.1 pts YoY) | 3.92 (flat YoY) | 20.0% (▼ +0.8 pts YoY) |

- Monthly revenue trend shown across 2023–2026, with a consistent seasonal dip in Feb and a peak around May and year-end
- **Bronx** leads bookings by neighbourhood (12.6K), followed by Crown Heights, Harlem, and Chelsea (~7.5K each)
- **Entire home/apt** dominates bookings at 56.16% (56K), Private room at 31.22% (31K), Hotel room at 6.39% (6K)
- Occupancy rate holds steady in the 47-49% range all year
- RevPAR and Avg Nightly Rate move inversely in early months, converging mid-year

**Page 2 : Host & Property**

- **Superhosts drive the business**: 71.07% of bookings (71K) and 69.3% of revenue ($122.02M) come from superhosts vs. regular hosts
- **Hostel** is the top property type by bookings (16.2K), followed by Loft (13.7K) and House (12.6K)
- Avg Nightly Rate vs. Avg Review Score bubble chart shows Loft and Studio at the highest price points (~$300), while ratings stay tightly clustered between 3.91–3.93 across all property types
- Host Performance table ranks 100+ hosts by revenue, bookings, and avg revenue per booking, e.g. Andre Moreau leads with $83.5L revenue across 3,772 bookings
- Decomposition tree lets users drill Revenue → Neighbourhood → Room Type → Property Type interactively (e.g. Bronx → Entire home/apt → Loft)

---

## 📌 Project Overview

This project simulates a production-grade Airbnb data extract for NYC, spanning **January 2023 - December 2024**, across:

- **100,200** booking records (fact table)
- **80** property listings
- **50** hosts
- **20** NYC neighbourhoods

The goal was to answer key business questions around:
- Revenue performance across neighbourhoods
- Occupancy patterns and availability trends
- Host quality : regular vs. superhost performance
- Month-over-month and year-over-year booking trends

---

## 🎯 Objectives

**Primary**
- Monitor revenue performance across NYC neighbourhoods
- Identify high-performing listings and superhosts using occupancy rate, RevPAR, and review scores
- Track booking trends over time (MoM / YoY)
- Demonstrate a complete Power BI workflow from raw CSV to published dashboard

**Secondary**
- Showcase Power Query M code for real-world data cleaning
- Apply DAX best practices (VAR patterns, CALCULATE, time intelligence)
- Illustrate star schema modeling principles
- Provide a reusable dataset/report template for Power BI learners

---

## 🗂️ Data Sources

Five CSV files simulate the Airbnb data extract, loaded via Power Query into a star schema:

| File | Table | Rows | Columns | Description |
|------|-------|------|---------|-------------|
| `fact_bookings.csv` | `fact_bookings` | 100,200 | 13 | Central booking transactions — revenue, reviews, pricing |
| `dim_host.csv` | `dim_host` | 50 | 6 | Host profiles : names, join date, response rate, superhost flag |
| `dim_property.csv` | `dim_property` | 80 | 11 | Listing details : room type, property type, amenities, base price |
| `dim_location.csv` | `dim_location` | 20 | 8 | NYC neighbourhood lookup with lat/lon and zip code |
| `dim_date.csv` | `dim_date` | 1,461 | 11 | Date dimension covering Jan 2023 - Dec 2024 |

---

## 🧹 Data Cleaning (Power Query)

The raw dataset ships with **intentional, documented data quality issues**, each resolved with a targeted M code fix:

| Table | Column | Issue | Fix |
|-------|--------|-------|-----|
| `fact_bookings` | `booking_id` | 200 exact duplicate rows | `Table.Distinct()` |
| `fact_bookings` | `price` | ~3,000 rows stored as text (`"$1,234.00"`) | `Text.Remove` + `Number.From` |
| `fact_bookings` | `review_score` | ~1,500 rows > 5.0 (invalid range) | `Table.SelectRows` — keep ≤ 5.0 |
| `fact_bookings` | `minimum_nights` | ~800 rows with 0 or negative values | `Table.SelectRows` — keep ≥ 1 |
| `fact_bookings` | `availability_365` | ~600 rows = 999 (sentinel error) | `Table.ReplaceValue` → null |
| `dim_host` | `host_name` | 3 empty/null rows | Replace null → `"Unknown Host"` |
| `dim_host` | `host_response_rate` | 3 rows stored as decimal (0.85) instead of `85%` | Detect type → ×100 → format as % |
| `dim_property` | `bedrooms` | 5 null values (studio-type units) | Replace null → `0` |
| `dim_location` | `latitude` | 2 values with leading/trailing spaces | `Text.Trim` → `Number.From` |
| `dim_date` | `month_name` | Mixed casing (`'january'`, `'February'`) | `Text.Proper` |

All fixes are applied before `Close & Apply`, and all data quality checks pass acceptance criteria.

---

## ⭐ Data Model — Star Schema

A classic star schema with **1 fact table** and **4 dimension tables**, all relationships one-to-many with single-direction filtering (dimension → fact):

| From (Dim) | Key | To (Fact) | Cardinality |
|------------|-----|-----------|-------------|
| `dim_date` | `date_key` | `fact_bookings[date_key]` | 1:* |
| `dim_host` | `host_id` | `fact_bookings[host_id]` | 1:* |
| `dim_property` | `listing_id` | `fact_bookings[listing_id]` | 1:* |
| `dim_location` | `location_id` | `fact_bookings[location_id]` | 1:* |

**Modeling rules applied:**
- All foreign key columns in `fact_bookings` hidden from Report View
- `dim_date` marked as the official Date Table (`full_date` column)
- All dimension tables set to Import mode
- Auto date/time disabled
- All 24 measures organized in a dedicated `📊 Measures` table

---

## 📐 DAX Measures

24 measures across 5 display folders:

- **💰 Core Revenue KPIs** : Total Revenue, Total Bookings, Avg Nightly Rate, RevPAR, Avg Revenue Per Booking
- **🏠 Occupancy & Availability** : Occupancy Rate, Cancellation Rate, Avg Nights Per Booking
- **🗓️ Time Intelligence** : Revenue PM/PY/YTD, MoM & YoY Revenue Growth %
- **⭐ Review & Quality** : Avg Review Score, Total Reviews, High Rating %
- **🏆 Host Performance** : Superhost Revenue, Superhost Revenue %, Superhost Avg Review, Total Active Hosts
- **🎨 KPI Formatting** : MoM/YoY Arrow Labels, Revenue Label (compact `$1.2M` format)

Example patterns used: `CALCULATE`, `DATEADD`, `SAMEPERIODLASTYEAR`, `DATESYTD`, `DIVIDE`, and `VAR`-based conditional formatting for KPI cards.

---

## 📊 Report Pages

**Page 1 : Executive Overview**
- 4 KPI cards: Total Revenue, Avg Nightly Rate, Occupancy Rate, Avg Review Score
- Monthly revenue trend (line chart, years overlaid)
- Top 10 neighbourhoods by revenue (clustered bar)
- Room type revenue split (donut chart)
- Slicers: Year, Room Type

**Page 2 : Host & Property Drill**
- Avg Nightly Rate vs. Avg Review Score bubble chart (bubble size = Revenue)
- Superhost vs. Regular host comparison (Revenue, Occupancy, Reviews)
- Host performance matrix with data bar conditional formatting
- Decomposition tree: Revenue by Neighbourhood → Room Type → Superhost flag
- Page navigation back to Page 1

Design: 1280×720 canvas, dark theme (`#0F172A` background), native Power BI visuals only, no custom visual imports.

---

## 🛠️ Tools Used

- Power BI Desktop
- Power Query (M)
- DAX
- CSV source data

---

## 🚀 How to Use

1. Clone this repo
2. Place all 5 CSV files in a single local folder
3. Open the '.pbix' file in Power BI Desktop and update the data source folder path
4. Refresh the data. Power Query will apply all cleaning steps automatically
5. Explore the report on Page 1 (Executive Overview) and Page 2 (Host & Property Drill)

---
