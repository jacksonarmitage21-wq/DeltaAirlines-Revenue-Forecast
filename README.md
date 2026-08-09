# Delta Air Lines Quarterly Revenue Forecast

An OLS time-series model that forecasts Delta Air Lines (NYSE: DAL) quarterly total revenue, built in Python with `pandas` and `statsmodels`. It fits a linear time trend on revenue levels with COVID and Q1-seasonality dummies (plus interaction terms), validates on a 25% out-of-sample hold-out, benchmarks the result against Delta's actual reported revenue, and projects the next four quarters (2026 Q3 – 2027 Q2).

## Repository layout

```
DeltaAirlines-Revenue-Forecast/
├── code/
│   └── DeltaAirlines_Revenue_Forecast.ipynb
├── data/
│   └── DELTA_Q_Revenue.csv
└── README.md
```

## Data

`DELTA_Q_Revenue.csv` — 40 quarters of DAL total revenue, FQ3 2016 through FQ2 2026. Two columns:

| Column | Description |
|--------|-------------|
| `Dates` | Fiscal quarter label (e.g. `FQ3 2016`), converted to calendar quarter-end dates |
| `delta_revenue` | Total revenue in billions (parsed from strings like `10.48b`) |

The series shows the COVID trough clearly — revenue collapses from ~$12.5B (FQ2 2019) to $1.47B (FQ2 2020) before recovering.

## Methodology

A single OLS specification on revenue **levels** (in $B):

```
delta_revenue = β₀ + β₁·time + β₂·covid_dv + β₃·(covid_dv × time)
                   + β₄·q1_dv + β₅·(q1_dv × time) + ε
```

Regressors:
- **`time`** — sequential period index (1…40), the linear trend
- **`covid_dv`** — 1 for quarters between 2020-03-11 and 2023-05-05, else 0
- **`covid_dv_interaction`** — `time × covid_dv`, allowing a different trend slope during COVID
- **`q1_dv`** — 1 for calendar-Q1 quarters (captures Delta's seasonal Q1 softness)
- **`q1_dv_interaction`** — `time × q1_dv`

Data is split 75/25 (30 train / 10 test). The model is fit on the training set, scored on the hold-out using the **full feature set**, and used to forecast four forward quarters.

## Model results (training set, n = 30)

| Term | Coefficient | p-value |
|------|-------------|---------|
| const | 9.49 | <0.001 |
| time | 0.191 | <0.001 |
| covid_dv | −20.88 | <0.001 |
| covid_dv_interaction | 0.764 | <0.001 |
| q1_dv | 0.174 | 0.914 |
| q1_dv_interaction | −0.039 | 0.690 |

**R² = 0.80, Adjusted R² = 0.76.**

The trend and both COVID terms are highly significant. The base trend adds ~$0.19B/quarter; the COVID dummy drops the intercept sharply while its interaction partially offsets via a steeper recovery slope. Both Q1 seasonality terms are statistically insignificant (p ≈ 0.91 and 0.69) — Delta's Q1 softness isn't cleanly separated by a dummy over this sample. The terms are retained for the forecast but carry little weight, and the "Limitations" section notes the simpler specification they point toward.

## Validation (hold-out, 10 quarters)

**RMSE = 1.01 ($B), MAPE = 4.32%.**

Errors are modest across most of the hold-out. The largest miss is FQ2 2026 (actual $19.76B vs. predicted $17.14B, +$2.62B), where the model under-forecasts a strong quarter — consistent with a linear trend not keeping pace with recent revenue acceleration.

## Forecast (2026 Q3 – 2027 Q2)

| Quarter | Predicted revenue ($B) |
|---------|------------------------|
| 2026 Q3 | 17.33 |
| 2026 Q4 | 17.53 |
| 2027 Q1 | 16.23 |
| 2027 Q2 | 17.91 |

The Q1 dip reflects the (statistically weak) seasonal dummy.

## Running it

```bash
pip install pandas numpy matplotlib statsmodels
```

The notebook reads `../data/DELTA_Q_Revenue.csv`, so run it from the `code/` folder with the CSV in `data/`. Locally: open `code/DeltaAirlines_Revenue_Forecast.ipynb` in Jupyter and run top to bottom. In Colab, open it from the GitHub tab and add a cell at the top:

```python
!git clone https://github.com/jacksonarmitage21-wq/DeltaAirlines-Revenue-Forecast.git
%cd DeltaAirlines-Revenue-Forecast/code
```

then Runtime → Run all.

## Notes

Coursework/portfolio project (AFM 244, University of Waterloo). Not investment advice. 

Sources: 
https://ir.delta.com/home/default.aspx
https://ca.finance.yahoo.com/quote/DAL/history/
