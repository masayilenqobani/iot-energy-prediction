# Notebook v2 Fixes

*7 November 2026 · Nqobani Masayile*

## Summary
The original notebook ran, but several of its charts were driven by bad data rather than building behaviour. Version 2 (`ashrae_iot_analysis_v2.ipynb`) cleans the meter data, lines up the weather and meter clocks, compares meter types fairly, and adds the analysis the proposal promises but the notebook lacked. The original file is kept unchanged. **v2 has not been run yet**: run it top to bottom in Colab or Databricks before quoting any numbers.

## Fixes at a glance

| # | Problem | Section | Fix |
|---|---|---|---|
| 1 | Building 1099's steam meter reports impossible values | 4.2 | Removed that meter |
| 2 | Site 0 electricity reads zero until 20 May 2016 | 4.2 | Removed those readings |
| 3 | Zero electricity readings | 4.2 | Removed (zeros kept on heating and cooling meters) |
| 4 | Site 0 electricity recorded in kBTU | 4.2 | Converted to kWh (× 0.2931) |
| 5 | Weather clock is UTC, meter clock is local time | 4.3 | Shifted weather to each site's local time |
| 6 | Charts averaged all four meter types together | 5.1, 5.2 | Split by meter, shown as an index |
| 7 | Two different weekday numbering systems | 5.2 | Uses the engineered Mon = 0 feature |
| 8 | "kWh per sq ft" included non-kWh meters | 5.5 | Electricity only |
| 9 | Missing build years all filled with the median | 4.1, 5.5 | Flagged; shown as an "Unknown" age group |
| 10 | Spark's 1 GB default memory | 3 | Raised to 6 GB |
| 11 | Redundant cells, placeholder team names | 4.1, 4.2, header | Removed; names added |

## Why each fix matters
1. **Building 1099 steam.** Its readings are far larger than any other steam meter. In the original output, Spring steam totalled 2.5 × 10¹⁰, 3.6 to 16 times the other seasons. That spike was one faulty meter, not a season effect, like one bad flow transmitter swamping a plant-wide total.
2. **Site 0 before 20 May 2016.** The meters were not online yet, so missing data was recorded as zero, which dragged every average down.
3. **Zero electricity.** An occupied building always draws some power, so a zero is a sensor or logging fault. Zeros on chilled water, steam and hot water can be real (no cooling in winter), so they stay.
4. **Site 0 units.** Converting kBTU to kWh puts site 0 on the same scale as the other sites.
5. **Time zones.** Without the shift, the join attaches afternoon weather to a morning reading at a US East Coast site, 5 hours off. That blurs every temperature result (5.3–5.5) and the models. Offsets (−5 to −8 hours for US sites, 0 for the European sites) come from Kaggle community analysis, since the sites are anonymized.
6. **Mixed meter types.** Steam readings run about 100 times larger than electricity (in the 5.4 output, steam averages 2,400–19,000 against 150–250 for electricity). Averaging them together meant the daily and weekly curves mostly showed steam. v2 groups by meter and scales each to its own average (100 = average hour or day), like normalizing reactor temperature and pressure trends to their setpoints so both fit one chart. 5.2 also prints the weekend drop in % per meter.
7. **Weekday numbering.** Spark's `DAYOFWEEK` counts Sunday = 1, while the engineered `day_of_week` counts Monday = 0. v2 uses the engineered features throughout, so Sections 4 and 5 agree.
8. **kWh per sq ft.** Steam and water meters are not measured in kWh, so 5.5 now uses electricity only and the unit label is true.
9. **Missing build years.** 774 of 1,449 buildings (53%) have no `year_built`. Filling them all with the median put them in one age group, so 5.5 was mostly comparing filled values. v2 flags missing values and shows those buildings as "Unknown". Missing floor counts (75% of buildings, filled with 1) are flagged too, and both flags feed the models.
10. **Memory.** Spark's 1 GB default can crash on 20-million-row joins and model training in Colab.

## New analysis added

| Section | What it adds | Proposal link |
|---|---|---|
| 5.6 | Electricity use per sq ft per year by primary use and by size quartile | RQ2; uses `building_size`, which the original built but never used |
| 5.7 | Temperature vs usage scatter per meter, plus within-building correlation | RQ1, weather correlation |
| 6.1 | Gold feature table saved as Parquet (Delta on Databricks) | Objective 1 |
| 6.2 | Train on Jan–Sep, test on Oct–Dec | Validation strategy |
| 6.3 | Linear regression, random forest and gradient boosting, scored on RMSLE, MAE and R² | Objective 3, RQ3 |
| 6.4 | Accuracy per meter type and feature importance | RQ3 |
| 7.1–7.2 | Flags readings far from the model (z-score of 3 or more), likely-cause rules, weekly report, review sample, example chart | Objective 4, RQ4 |
| 7.3 | Buildings using more electricity than expected | Recommendations for owners |
| 8 | Key-findings template to fill in after the run | Report |

**Design choices**
- Models train on a 5% sample of January–September (under 1 million rows) because Colab has 2 CPU cores. Raise `TRAIN_FRACTION` on Databricks.
- Gradient boosting uses Spark's built-in `GBTRegressor`: the same method as XGBoost and LightGBM, with no extra installs.
- `month` and `us_season` are left out of the models because the test months never appear in training.
- Cause labels are rule-based first guesses to guide inspection, not confirmed causes.

## Still to do
- Run v2 top to bottom in Colab (expect 30–60 minutes, mostly the models). It has not been run yet.
- Fill in Section 8 with the results.
- Quote the number of rows removed in 4.2, which the notebook prints.
- Proposal items not in the notebook yet: Databricks bronze/silver/gold Delta tables and MLflow tracking.
