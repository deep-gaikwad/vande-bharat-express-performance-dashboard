# Methodology & Build Notes

This document explains how the dataset was structured and how the report was built, so the project can be reproduced or extended.

## 1. Data Source

The dataset is a **synthetic booking log** for the Indian Railways Vande Bharat Express network, built to resemble a realistic operational dataset.

- **164 trains** across major Indian corridors (`Train_Master`)
- **Approximately 100,000 booking records** spanning **January–May 2026** (`Fact_Bookings`)
- **23 states** grouped into 5 regions (`State_Master`)

The dataset was generated for portfolio and learning purposes. It is not sourced from official Indian Railways systems and should not be interpreted as official operational data.

## 2. Data Preparation (Power Query)

The following preparation steps were applied in Power Query before loading the data into the model:

1. **Type enforcement** — Explicit data types for dates (`Journey_Date`, `Booking_Date`), times (`Scheduled_Departure`, `Actual_Departure`), decimal values (`Revenue`, `Ticket_Price`, `Occupancy_%`, `Speed_kmph`), and Boolean values (`On_Time`).
2. **Referential integrity checks** — Verified that every `Train_No` in `Fact_Bookings` exists in `Train_Master`, and every `State` exists in `State_Master`.
3. **Month sort order** — Added a numeric `Month Sort` helper, or used `Year` and `Month`, to ensure chronological sorting rather than alphabetical sorting.
4. **No personally identifiable information (PII)** — The dataset intentionally avoids real passenger-identifying information. `Age`, `Gender`, and `Passenger_Type` are aggregate demographic fields only.

## 3. Data Model

The model follows a **star schema**:

- `Fact_Bookings` → many-to-one relationship with `Train_Master` on `Train_No`
- `Fact_Bookings` → many-to-one relationship with `State_Master` on `State`
- Power BI's automatic date hierarchy is used on `Journey_Date` for month, quarter, and year drill-down.

Single-direction filtering is used throughout, with dimension tables filtering the fact table. This helps keep the model predictable and avoids ambiguous filter paths.

## 4. Measures

All KPIs and chart values are driven by explicit DAX measures rather than implicit column aggregations. This helps maintain consistent formatting, filter context, and totals behavior across visuals.

See [`DAX_measures.md`](DAX_measures.md) for the documented measures.

## 5. Report Design

- **Theme:** Dark navy background with white and orange accents, inspired by Vande Bharat branding, creating a high-contrast operations-dashboard appearance.
- **Layout:** KPI strip across the top, trend and distribution charts in the middle, and a detailed performance table with leaderboard callouts at the bottom. The layout follows a summary-to-detail reading flow.
- **Filters:** A persistent left-hand slicer panel includes State, Route, Train_Name, Train_No, Coach_Type, Class, Journey_Date, Month, Passenger_Type, and Gender.
- **Leaderboard cards:** Automatically calculated highlights for highest revenue route, busiest train, best occupancy, highest speed, and least average delay provide a quick performance overview.

## 6. Known Limitations

- **Map visual:** The state-wise passenger distribution map uses a Power BI map visual identified for retirement. A future version could migrate to Azure Maps or ArcGIS, subject to availability and compatibility.
- **Class and coach fields:** `Class` and `Coach_Type` are currently duplicate fields. A future version could distinguish them to enable richer class-mix analysis.
- **On-time threshold:** The delay tolerance used to determine whether a record is on time is fixed in the source `On_Time` flag rather than controlled by a configurable parameter.

## 7. Suggested Next Steps

- **Region-level analysis:** Add a regional rollup page using `State_Master[Region]`.
- **Year-over-year comparison:** Add YoY analysis when additional months or years of data are available.
- **Configurable punctuality target:** Introduce a Power BI What-If parameter to explore different delay thresholds.
- **Row-level security (RLS):** Implement regional access controls if the dashboard is adapted for zonal railway managers.

---

## Dataset Disclaimer

This project uses synthetic data for portfolio and educational purposes. The figures and insights represent the constructed dataset and do not describe verified official Indian Railways performance.
