# IoT Energy Project Goals

*ISTM 6217 · 7 November 2026*

## In one sentence
Use a year of hourly smart-meter readings from about 1,450 buildings, plus weather and building details, to learn how much energy each building *should* use every hour. Then use the gap between expected and actual use to find faults, waste and savings.

## Why it matters
- Buildings account for 32% of global energy use and 34% of CO₂ emissions (UNEP, 2025).
- US commercial buildings spent about $142 billion on energy in 2018, or $1.47 per sq ft (EIA, 2022).
- ASHRAE ran the competition to measure retrofit savings, which needs a credible estimate of what a building would have used without the upgrade (Miller et al., 2020).

It works like a plant energy balance: once you know what a process should consume, any gap points to a leak, a fault or a real improvement.

## The data

| Table | Contents | Joins on |
|---|---|---|
| train.csv | Hourly reading per building and meter, all of 2016 (~20 million rows) | building_id |
| building_metadata.csv | Site, primary use, floor area, year built, floors | building_id, site_id |
| weather_train.csv | Hourly weather per site | site_id + timestamp |

Meter types: 0 electricity, 1 chilled water, 2 steam, 3 hot water. 16 sites, mostly university campuses.

## Objectives
1. **Pipeline:** load the three files into bronze (raw) tables, then clean and join them into silver (clean, joined) and gold (model-ready) tables. Think raw feed → treated stream → product on spec.
2. **Patterns:** daily, weekly and seasonal cycles, and how much temperature, floor area and primary use drive usage.
3. **Prediction:** a linear regression baseline, a random forest and a gradient boosting model, trained on January–September and tested on October–December. The best model must beat the baseline on RMSLE; MAE and R² are also reported.
4. **Anomalies:** flag readings far from the prediction and check likely causes: meter outages, sudden spikes, after-hours use.
5. **Recommendations:** practical actions for facility managers, building owners and finance teams.

## Research questions
1. How do hour, weekday, season and outdoor temperature affect energy use, and does it differ by meter type?
2. Which primary building uses consume the most energy per square foot?
3. How accurately can October–December be predicted, and which features matter most?
4. What do the largest prediction errors reveal about meter faults, equipment problems or waste?

## Key ideas in plain terms
- **RMSLE:** error measured on a log scale, so it behaves like a percentage error. A 10% miss counts the same for a small lab and a large hospital.
- **Time-based split:** train on the past, test on the future, like validating a process model on next quarter's data instead of data it has already seen.
- **Anomaly detection on residuals:** a control chart on the gap between measured and predicted use. Points beyond 3 standard deviations get investigated.
- **Feature importance:** which inputs the model leaned on most.

## Who uses the results

| Stakeholder | Decision | What the project gives them |
|---|---|---|
| Facility managers | Which equipment to inspect, and when | Weekly anomaly list: after-hours use, spikes, dead meters |
| Owners and portfolio managers | Where to spend upgrade money | Buildings ranked by actual vs expected use; before/after checks on retrofits |
| Finance teams | How much to budget for energy | Forecasts by building and meter; early warning of overruns |

## Where the notebook stands (v2)

| Proposal item | Status |
|---|---|
| Load, clean and join the three tables | Done (Colab, Spark) |
| Fix time zones, outages, missing weather, outliers | Done |
| Time, weather and building features | Done |
| Patterns and temperature effects (RQ1) | Done: 5.1–5.4, 5.7 |
| Energy per sq ft by use and size (RQ2) | Added: 5.6 |
| Three models and metrics (RQ3) | Added: Section 6 |
| Anomaly detection and building ranking (RQ4) | Added: Section 7 |
| Findings and recommendations | Template in Section 8; fill in after the run |
| Databricks bronze/silver/gold Delta tables, MLflow | Not started |
| Report, slides, code export | Not started |

## Limitations to state in the report
- Only 2016, so each season appears once.
- Buildings are anonymized, and there is no occupancy, equipment or price data, so dollar and emissions figures need assumed rates.
- Mostly university buildings, so results may not transfer to offices or retail.
- Steam and hot water are harder to predict, so treat their anomalies with more caution.
- Time-zone offsets come from community analysis of the anonymized sites.
- 53% of buildings lack a build year and 75% lack a floor count.

## References
ASHRAE. (2019). *ASHRAE – Great Energy Predictor III* [Data set]. Kaggle. https://www.kaggle.com/competitions/ashrae-energy-prediction

Miller, C., Arjunan, P., Kathirgamanathan, A., Fu, C., Roth, J., Park, J. Y., Balbach, C., Gowri, K., Nagy, Z., Fontanini, A., & Haberl, J. (2020). The ASHRAE Great Energy Predictor III competition: Overview and results. *Science and Technology for the Built Environment*. https://doi.org/10.1080/23744731.2020.1795514

U.S. Energy Information Administration. (2022, September 28). *U.S. commercial buildings were consuming less energy per square foot as of 2018, according to latest EIA data* [Press release]. https://www.eia.gov/pressroom/releases/press517.php

> Note: the proposal cites UNEP (2025) in the text but leaves it out of its reference list. Add it before submitting.
