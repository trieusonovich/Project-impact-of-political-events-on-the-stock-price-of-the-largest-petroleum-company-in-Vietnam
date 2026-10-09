# Impact of Global Political Events on Vietnamese Petroleum Stock Prices

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-Random%20Forest-F7931E?logo=scikitlearn&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-data%20analysis-150458?logo=pandas&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-t--test-8CAAE6?logo=scipy&logoColor=white)

A financial econometrics project that applies **event study methodology** combined with a **machine learning model (Random Forest)** to assess the impact of global geopolitical events on the stock price of Petrolimex (PLX) — the largest petroleum company in Vietnam. The workflow covers data preparation, time-stratified train/test splitting, model training on "normal" periods, abnormal return (AR) and cumulative abnormal return (CAR) computation, and statistical significance testing via t-test.

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Key Results](#key-results)
- [Statistical Tests](#statistical-tests)
- [Limitations](#limitations)
- [Author](#author)

---

## Overview

Geopolitical events such as armed conflicts, OPEC+ production decisions, trade wars, and regional escalations can indirectly affect Vietnam's economy — especially its energy sector — through global oil prices, exchange rates, and market sentiment. This project answers a focused question: **do selected geopolitical events have a statistically significant impact on PLX stock prices, and can we quantify it using a machine learning counterfactual?**

**Objectives**

1. Collect and prepare daily PLX stock data with relevant macro variables
2. Build a "normal-period" training set for each event (no event influence)
3. Train a Random Forest model to predict PLX price under normal conditions
4. Compute Abnormal Return (AR) and Cumulative Abnormal Return (CAR)
5. Test statistical significance using one-sample t-tests

## Dataset

| Property | Value |
|---|---|
| Frequency | Daily (trading days only) |
| Period | 2021 – 2026 |
| File | `data PLX.xlsx` |
| Target | `PLX` — closing price of Petrolimex stock |
| Features | `USD/VND` — exchange rate |
| | `Brent` — Brent crude oil price (USD/barrel) |
| | `VNIndex` — Vietnam stock market index |

## Methodology

**1. Data preparation**
Load daily data, remove weekends (Vietnam stock market is closed Saturday/Sunday), standardize column names.

**2. Event definition**
Four geopolitical events are analyzed:

| # | Date | Event |
|---|---|---|
| 1 | 2022-02-24 | Russia–Ukraine conflict begins |
| 2 | 2023-04-03 | OPEC+ announces voluntary production cuts |
| 3 | 2025-04-02 | Trump administration raises import tariffs |
| 4 | 2026-03-02 | Iran conflict escalation (started 28/02/2026) |

**3. Time-stratified train/test split**
Unlike random splitting (which risks data leakage), the training set is built **only from "normal" days** unaffected by the event:

- **Train:** 100 observations from day −110 to day −10 relative to the event
- **Buffer:** 10 days before the event excluded
- **Test:** 10 observations — 2 days before + 7 days after the event

**4. Model: Random Forest**
A `RandomForestRegressor` is trained independently for each event on the normal-period data. Each tree is built on a bootstrap subsample with a random feature subset at each split. The final prediction is the average across trees.

Hyperparameter tuning via **Grid Search**:

- `n_estimators` = [50, 100, 200]
- `max_depth` = [5, 10, 20, None]
- `min_samples_split` = [2, 5, 10]

Best combination selected by minimum MSE on a validation fold.

**5. Abnormal Return (AR) and Cumulative Abnormal Return (CAR)**

AR_t = P_t^actual − P_t^predicted (1)
CAR_total = Σ AR_t, t = 1..10 (2)

- `AR_t > 0` → market reacted more positively than expected
- `AR_t < 0` → market fell more than expected
- `AR_t ≈ 0` → no event effect

**6. Statistical testing**
One-sample t-test with H₀: mean AR = 0, H₁: mean AR ≠ 0, α = 0.05, df = n − 1 = 9.

## Key Results

### Model quality on training set

| Event | Date | R² (train) |
|---|---|---|
| 1. Russia–Ukraine | 2022-02-24 | 0.935 |
| 2. OPEC+ cuts | 2023-04-03 | 0.917 |
| 3. Trump tariffs | 2025-04-02 | 0.914 |
| 4. Iran escalation | 2026-03-02 | 0.988 |

All models exceed R² = 0.91, confirming they capture "normal" market behavior well.

### Cumulative Abnormal Return (CAR_total)

| Event | CAR_total | Interpretation |
|---|---|---|
| Russia–Ukraine | +40.62 | Strong positive effect |
| OPEC+ | +2.49 | Weak positive effect |
| Trump tariffs | −63.74 | Strong negative effect |
| Iran | +51.16 | Strong positive effect |

**Key insights:**

- Events linked to **rising oil prices** (Russia–Ukraine, Iran) produced **positive** and statistically significant effects on PLX — Vietnam is a net oil exporter, so higher prices improve Petrolimex's financials.
- The **OPEC+ cut** had only a weak effect, likely because the market had already priced it in.
- The **Trump tariff** event — not directly oil-related — caused a strong **negative** effect, confirming that global trade conflicts hurt emerging markets through demand and uncertainty channels.

## Statistical Tests

One-sample t-test on AR and CAR (α = 0.05):

| Event | p-value (AR) | p-value (CAR) | Significant? |
|---|---|---|---|
| Russia–Ukraine | 5.86 × 10⁻⁸ | 2.45 × 10⁻⁴ | Yes (AR and CAR) |
| OPEC+ | 0.260 | 6.75 × 10⁻⁶ | No (AR), Yes (CAR) |
| Trump tariffs | 6.20 × 10⁻⁵ | 2.22 × 10⁻³ | Yes (AR and CAR) |
| Iran | 0.018 | 2.54 × 10⁻³ | Yes (AR and CAR) |

**Note on OPEC+:** individual daily AR values are not significant on average, but their cumulative sum (CAR) is highly significant — positive ARs in early days outweigh negative ones later. This is a classic case of a short-term cumulative effect that a daily test cannot capture.

## Limitations

- **Small test window.** Only 10 observations per event — results should be interpreted with caution.
- **Univariate target, limited features.** The model uses only three exogenous variables; news sentiment, earnings, and other macro factors are ignored.
- **Model dependence.** AR/CAR values depend on the Random Forest's ability to generalize; a different model may yield different magnitudes.

**Possible next steps:** expand the event set, include more petroleum companies, compare Random Forest with XGBoost and neural networks, and apply the methodology to other sectors.

## Author
Nguyen Dinh Trieu
Economics (Analytical Economics and Econometrics)

trieu31072004@gmail.com
