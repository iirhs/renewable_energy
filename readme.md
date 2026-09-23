# ⚡ Renewable Energy Grid Analysis — Germany 2023

An end-to-end data analysis project examining hourly solar and wind generation vs. grid demand across Germany for the full year 2023 — identifying curtailment hours, negative-price risk periods, and estimating the battery storage capacity needed to reduce energy waste.

---

## 📌 Project Overview

As Germany pushes toward 80% renewable energy by 2030, understanding when and why renewable generation exceeds demand becomes critical. This project analyses 8,760 hourly observations from Germany's official electricity market data platform (SMARD) to answer three questions:

1. When does renewable generation exceed demand — and by how much?
2. Which hours carry negative-price risk (too much renewable supply)?
3. How large does a battery need to be to absorb the worst-case surplus?

---

## 🔑 Key Findings

| Metric | Value |
|--------|-------|
| Solar generated (2023) | 55.5 TWh |
| Wind generated (2023) | 142.6 TWh |
| Total grid demand (2023) | 458.5 TWh |
| Wind + Solar share of demand | 43.2% |
| Curtailment hours (renewable > demand) | 51 hours |
| Total curtailed energy | 109.5 GWh |
| Negative-price risk hours (renewable ≥ 90% of demand) | 239 hours |
| Battery capacity needed (worst-case hour) | 5.6 GWh |
| Battery capacity needed (P95 sizing) | 5.2 GWh |

---

## 💡 Key Insights

- **July** had the most curtailment hours (18) — driven by peak solar production on long summer days
- **December** had the second highest (14) — driven by strong winter wind, not solar
- **Solar and wind are inversely correlated** — when solar peaks in summer, wind is lowest; when wind peaks in winter, solar is minimal. They naturally balance each other across the year
- **51 curtailment hours in 2023 is low** — but as renewable capacity grows toward 80%, these hours will multiply rapidly, making battery storage planning critical now

---

## 🛠️ Tools & Technologies

- **Python** — core analysis language
- **Pandas** — data loading, cleaning, and feature engineering
- **NumPy** — numerical operations and clipping
- **SQL (sqlite3)** — aggregations and curtailment queries on hourly data
- **Matplotlib** — visualisation (6-chart dashboard)
- **Data source** — [SMARD / Bundesnetzagentur](https://www.smard.de) (CC BY 4.0)

---

## 📂 Project Structure

```
renewable-energy-analysis/
│
├── analysis.py                        # Full analysis pipeline
├── smard_generation_2023.csv          # Raw generation data (SMARD)
├── smard_demand_2023.csv              # Raw demand/consumption data (SMARD)
├── outputs/
│   ├── germany_grid_hourly_2023.csv   # Cleaned hourly dataset (8,760 rows)
│   ├── monthly_summary.csv            # Monthly aggregations
│   └── renewable_grid_dashboard.png   # 6-chart visualisation dashboard
└── README.md
```

---

## 🔧 Data Pipeline

### 1. Load & Clean
- Loaded two SMARD CSV files: hourly generation and hourly consumption for 2023
- Separator: semicolon (`;`) — SMARD format
- Stripped thousand-separator commas from numeric columns and converted to float
- Parsed timestamps for time-based feature extraction

### 2. Feature Engineering
- Combined wind onshore + wind offshore → `wind_total_mwh`
- Combined solar + wind total → `renewable_mwh`
- Calculated `surplus_mwh` = renewable − demand, clipped at zero
- Created `curtailment_flag` (True when surplus > 0)
- Created `renewable_share` = renewable ÷ demand
- Created `neg_price_risk_flag` (True when renewable share ≥ 90%)

### 3. SQL Aggregations (sqlite3)
```sql
-- Key findings in one query
SELECT 
    COUNT(*) as curtailment_hours,
    SUM(surplus) as curtailed_energy_mwh,
    MAX(surplus) as battery_sizing_mwh
FROM grid_data
WHERE curtailment_flag = 1;

-- Monthly breakdown
SELECT month, SUM(Solar) as solar_mwh, SUM(`Wind Total [MWh]`) as wind_mwh,
       SUM(surplus) as curtailed_mwh, SUM(curtailment_flag) as curtailment_hours
FROM grid_data
GROUP BY month;
```

### 4. Battery Sizing Logic
The battery is sized to absorb the worst-case single hour of surplus:
- **Max surplus (worst case):** 5.6 GWh → minimum battery size
- **P95 surplus (practical):** 5.2 GWh → handles 95% of all curtailment events

---

## 📊 Dashboard

The 6-chart dashboard covers:

1. **Monthly Generation vs. Demand** — solar and wind bars vs. demand line
2. **Curtailment & Negative-Price Risk Hours** — by month, dual axis
3. **Average Hourly Generation Profile** — stacked area chart, hour 0–23
4. **Distribution of Hourly Renewable Share** — histogram with neg-price threshold
5. **Battery Sizing: Surplus Duration Curve** — ranked surplus with battery size scenarios
6. **Renewable Share Heatmap** — day of week × hour of day (green = high, red = low)

---

## 🚀 How to Run

**Install dependencies:**
```bash
pip install pandas numpy matplotlib
```

**Add your data files** (download from [smard.de](https://www.smard.de/en/downloadcenter/download-market-data)):
- `smard_generation_2023.csv` — Electricity generation, hourly, Germany, 2023
- `smard_demand_2023.csv` — Electricity consumption, hourly, Germany, 2023

**Run the analysis:**
```bash
python analysis.py
```

Outputs are saved to the `outputs/` folder automatically.

---

## 👩‍💻 Author

**Shrinithi Sellam VP**  
MSc Data Science — Hochschule Fulda, Germany  
[LinkedIn](https://www.linkedin.com/in/shrinithi-sellam) | [GitHub](https://github.com/iirhs)
