# EU ETS Extension to Maritime Dry Bulk Shipping

### Project for the Environmental Economics course — University of Lausanne, Spring 2026

This repository contains the full replication package for our paper studying the behavioral effects of the EU Emissions Trading System (EU ETS) on dry bulk shipping vessels following its extension to maritime transport in January 2024.


## Project Structure

```
Project/
│
├── README.md
├── Report.pdf                             ← Full written report
├── Environmental_Economics_Presentation.mp4 ← Presentation video
├── Presentation_Slides.pdf                ← Beamer slides for the final presentation
│
├── Data/
│   ├── combin-all.csv                     ← Raw input dataset
│   └── prepared_panel_data.csv            ← Cleaned, balanced panel dataset
│
├── Results/
│   ├── Descriptive_stats/                 ← Tables and figures from script 2
│   │   ├── Global_Summary_Statistics.csv
│   │   ├── Global_Pre_vs_Post_Comparison.csv
│   │   ├── Cluster_Feature_Means.csv
│   │   ├── Cluster_Variable_Means.csv
│   │   ├── Cluster_Population_Counts.csv
│   │   ├── Pretrend_Cluster_MonthlyMeans_2023.csv
│   │   ├── Boxplot_*.png                  ← Distribution plots by cluster (6 variables)
│   │   ├── TimeSeries_Quarterly_*.png     ← Quarterly trends by cluster (2013–2025)
│   │   └── Pretrend_Cluster_*.png         ← Monthly pre-trend plots for 2023
│   │
│   └── Results_regressions/               ← Full regression output files from script 3
│       ├── GLOBAL_{outcome}_Baseline.txt  ← Global DiD, no trend control (one file per outcome)
│       ├── GLOBAL_{outcome}_Trend.txt     ← Global DiD, with trend control — main spec (one file per outcome)
│       ├── CLUSTER_{cluster}_{outcome}.txt ← Cluster-level regressions (one file per cluster × outcome)
│       └── ROBUST_PureBulk_{outcome}.txt  ← Robustness check: pure bulk subsample
│
└── Scripts/
    ├── data_preparation.ipynb             ← Script 1: cleaning, variable construction, clustering
    ├── descriptive_statistics.ipynb       ← Script 2: summary tables and visualizations
    └── regression_analysis.ipynb          ← Script 3: TWFE DiD regressions and robustness checks
```

---

## Scripts Overview

### Script 1 — `data_preparation.ipynb`

Loads the raw voyage-level dataset and produces the balanced panel used by scripts 1 and 2.

**Main steps:**
- Loads `combin-all.csv` and keeps the relevant columns (vessel name, dates, commodity, fuel consumption, speed, distance, CO₂, etc.)
- Constructs the key econometric variables:
  - `ETS_Rate`: treatment intensity (1.0 for intra-EU voyages, 0.5 for EU-adjacent, 0.0 for international)
  - `Post2024`: binary indicator for the treatment period (voyages departing on or after January 1, 2024)
  - `Interaction`: the DiD term (`ETS_Rate × Post2024`)
  - `log_CO2_per_nm`: log CO₂ per nautical mile (the main environmental outcome)
  - `MGO_Share`, `HSFO_Share`, `VLSFO_Share`: fuel-type shares (fuel switching outcomes)
  - `TimeIdx`: continuous monthly time index for time fixed effects and pre-trend controls
- Performs hierarchical clustering on departure-country profiles using 2023 data only (to avoid post-policy contamination). Seven country level features are standardized, reduced via PCA, and clustered with Ward's linkage into three trade profiles: High EU, Med EU, and Low EU
- Aggregates voyages to vessel-month observations (required for TWFE) and winsorizes continuous outcomes at the 1st/99th percentiles
- Retains only the balanced panel: vessels observed in both the pre-2024 and post-2024 periods (15,781 vessels)
- Tags each vessel with its primary commodity (mode across its voyages) for robustness checks
- Output: `Data/prepared_panel_data.csv` (1,755,789 vessel-month observations)

---

### Script 2 — `descriptive_statistics.ipynb`

Produces all summary tables and figures reported in the paper.

**Main steps:**
- Global summary statistics (mean, std, min, max, median) for the six outcome variables
- Raw pre/post comparison (unconditional means before and after 2024, with percentage change)
- Cluster-level breakdowns: population counts, mean outcomes per trade profile (High/Med/Low EU)
- Cluster feature profiles: mean values of the seven clustering variables by group (Table 3 in the report)
- **Distribution boxplots** for each clustering variable, separated by trade profile
- **Quarterly time-series plots** spanning the full 2013–2025 window, with a vertical line marking the January 2024 policy start
- **Monthly pre-trend plots** for 2023 only, used to visually assess the parallel trends assumption
- **Output:** all CSV tables and PNG figures saved to `Results/Descriptive_stats/`

---

### Script 3 — `regression_analysis.ipynb`

Runs all econometric estimations using a two-way fixed effects (TWFE) continuous-treatment DiD design.

**Design:** The treatment variable is continuous (`Interaction = ETS_Rate × Post2024`) rather than binary. This exploits variation in EU exposure intensity across voyage types. All models include vessel fixed effects (absorbing time-invariant ship characteristics) and time fixed effects (absorbing global monthly shocks), with standard errors clustered at the vessel level.

**Main steps:**

1. **Global comparative analysis** — runs two model specifications for each of the six outcomes:
   - *Baseline* (Model A): standard DiD with vessel and time FE only
   - *Main specification* (Model B): adds `ETS_x_Time` (`ETS_Rate × TimeIdx`) to control for pre-existing linear trends in EU-exposed vessels, isolating the discontinuous jump caused by the 2024 policy

2. **Cluster-level regressions** — re-runs the main specification separately for Low EU, Med EU, and High EU vessels to recover heterogeneous treatment effects across trade profiles

3. **Robustness check I** — excludes General Cargo, Project Cargo, and Unknown commodity vessels to confirm results hold for the core specialized bulk fleet (15,509 vessels)

**Output:** full regression summaries (coefficients, standard errors, R², F-statistics) saved as `.txt` files to `Results/Results_regressions/`

---

## Data

The raw input (`combin-all.csv`) covers dry bulk shipping voyages from 2022 to 2024. The prepared panel (`prepared_panel_data.csv`) contains **1,755,789 vessel-month observations** across **15,781 unique vessels**, after cleaning, clustering, aggregation, and balancing.

The six outcome variables studied are:

| Variable | Description |
|---|---|
| `log_CO2_per_nm` | Log CO₂ emissions per nautical mile (environmental intensity) |
| `MGO_Share` | Share of Marine Gas Oil in total fuel consumption (clean fuel) |
| `HSFO_Share` | Share of High-Sulphur Fuel Oil in total fuel consumption (dirty fuel) |
| `VLSFO_Share` | Share of Very Low-Sulphur Fuel Oil in total fuel consumption (standard fuel) |
| `Avg Speed (kn)` | Average voyage speed in knots (operational response) |
| `Distance (nm)` | Voyage distance in nautical miles (routing/evasion response) |

---

## Dependencies

All scripts run in Python 3. The main packages required are:

```
pandas
numpy
scipy
scikit-learn
matplotlib
seaborn
linearmodels
```
