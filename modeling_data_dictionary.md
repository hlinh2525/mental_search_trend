# Modeling Data Dictionary

## Project title

**Modeling COVID-Period Dynamics in Anxiety Search Behavior: An ARIMA, SARIMA, and SARIMAX Intervention Analysis Across Six Anglophone Countries**

This dictionary documents the modeling-ready dataset used in the final ARIMA, SARIMA, SARIMAX, forecast-evaluation, intervention, counterfactual, and cross-national comparison notebooks.

The main dependent variable is **anxiety**. Other Google Trends mental-health topics are retained in the cleaned dataset for transparency and possible robustness analysis, but the final empirical report focuses on anxiety search behavior.

---

## Notebook alignment

| Notebook | Role |
|---|---|
| `01_Preprocessing_Final_Complete.ipynb` | Cleans and combines Google Trends country-level data; creates modeling-ready datasets and COVID intervention variables |
| `02_EDA_Final_Complete.ipynb` | Descriptive analysis, stationarity checks, seasonality checks, ACF/PACF, and phase comparisons |
| `03_Modeling_Enhanced_Final_9point.ipynb` | Fits ARIMA, SARIMA, and SARIMAX candidates; produces raw model-selection, forecast, intervention, diagnostic, and counterfactual outputs |
| `04_Forecast_Evaluation_Final_9point.ipynb` | Evaluates ARIMA/SARIMA/SARIMAX against naive and seasonal-naive benchmarks |
| `05_SARIMAX_Intervention_Counterfactual_Final_9point.ipynb` | Interprets SARIMAX intervention coefficients and no-COVID SARIMA counterfactuals |
| `06_Crossnational_Comparison_Final_9point.ipynb` | Builds cross-country scorecards and final country-level interpretation tables |

---

## Core data files

| File | Format | Purpose |
|---|---|---|
| `input.csv` | wide country-panel | Cleaned EDA-ready dataset; preserves transparent preprocessing information |
| `input_modeling.csv` | wide country-panel | Complete modeling-ready dataset used for ARIMA, SARIMA, SARIMAX, and counterfactual analysis |
| `input_modeling_long.csv` | long format | Long-format dataset used for visualization, cross-country comparison, and variable-level summaries |
| `modeling_data_dictionary.md` | markdown | Documentation of dataset structure, variables, imputation policy, COVID phase design, and model alignment |

If the project is run with online data links instead of local folders, these files should be loaded from GitHub raw URLs or another stable direct-download source. The notebook logic should not depend on absolute local paths such as `data/raw` or `data/processed`.

---

## Sample coverage

| Item | Value |
|---|---|
| Countries | Australia, Canada, Ireland, New_Zealand, UK, US |
| Frequency | Monthly |
| Start | 2004-01 |
| End | 2025-12 |
| Months per country | 264 |
| Rows in `input_modeling.csv` | 1,584 |
| Rows in `input_modeling_long.csv` | 7,920 |
| Main dependent variable | `anxiety` |
| Supplementary variables retained | `depression`, `stress`, `insomnia`, `mental_disorder` |

---

## COVID-period definition

| Phase | Period | Months per country | Modeling interpretation |
|---|---:|---:|---|
| Pre-COVID | 2004-01 to 2019-12 | 192 | Baseline period used to estimate no-COVID dynamics |
| COVID period | 2020-01 to 2023-05 | 41 | Pandemic intervention period |
| Post-COVID | 2023-06 to 2025-12 | 31 | Post-intervention adjustment and normalization period |

The post-COVID phase starts in June 2023, immediately after the WHO ended the COVID-19 public health emergency in May 2023. This phase definition must remain identical across preprocessing, EDA, modeling, evaluation, intervention analysis, counterfactual analysis, and the final report.

---

## Variable definitions

| Variable          | Definition                                                                                                                                                                                                                                  |
|:------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| date              | Monthly timestamp stored as the first day of each month. Example: 2020-01-01 represents January 2020.                                                                                                                                       |
| country           | Country identifier: Australia, Canada, Ireland, New_Zealand, UK, or US.                                                                                                                                                                     |
| anxiety           | Google Trends topic-level relative search interest index for Anxiety. This is the main dependent variable used in ARIMA, SARIMA, SARIMAX, forecast evaluation, intervention analysis, and counterfactual analysis.                          |
| depression        | Google Trends topic-level relative search interest index for Major depressive disorder. Retained in the cleaned dataset for transparency and possible robustness analysis, but not used as the main dependent variable in the final report. |
| stress            | Google Trends topic-level relative search interest index for Psychological stress. Retained for robustness or supplementary analysis.                                                                                                       |
| insomnia          | Google Trends topic-level relative search interest index for Insomnia. Retained for robustness or supplementary analysis.                                                                                                                   |
| mental_disorder   | Google Trends topic-level relative search interest index for Mental disorder. Retained for robustness or supplementary analysis.                                                                                                            |
| *_was_imputed     | Variable-specific flag equal to 1 if the corresponding Google Trends value was originally zero or missing and was filled during preprocessing/modeling preparation; 0 otherwise.                                                            |
| year              | Calendar year extracted from date.                                                                                                                                                                                                          |
| month             | Calendar month extracted from date, from 1 to 12.                                                                                                                                                                                           |
| time_index        | Country-specific monthly time index starting at 0 in 2004-01.                                                                                                                                                                               |
| analysis_phase    | Categorical COVID phase: pre_covid, covid_period, or post_covid.                                                                                                                                                                            |
| early_covid_shock | Dummy equal to 1 from 2020-03 to 2020-06, otherwise 0. Used to capture the short-run early-pandemic shock separately from the full COVID period.                                                                                            |
| covid_period      | Dummy equal to 1 from 2020-01 to 2023-05, otherwise 0. Used as the COVID-period level intervention variable.                                                                                                                                |
| covid_trend       | Monthly counter within the COVID period. It equals 1 in 2020-01, 2 in 2020-02, ..., 41 in 2023-05, and 0 outside the COVID period. Used as the COVID-period slope intervention variable.                                                    |
| post_covid_period | Dummy equal to 1 from 2023-06 to 2025-12, otherwise 0. Used as the post-COVID level intervention variable.                                                                                                                                  |
| post_covid_trend  | Monthly counter within the post-COVID period. It equals 1 in 2023-06, 2 in 2023-07, ..., 31 in 2025-12, and 0 before the post-COVID period. Used as the post-COVID slope intervention variable.                                             |

---

## Google Trends interpretation

Google Trends values are normalized relative search interest indices, not absolute search counts and not clinical mental-health prevalence

A value of 100 represents the peak relative search interest for a topic within the selected country and time range. A lower value represents search interest relative to that peak. Therefore, the analysis focuses on time-series dynamics, persistence, seasonality, intervention effects, and within-country changes rather than absolute cross-country search volumes

---

## Imputation policy

Google Trends zeros are treated as potentially low-volume or insufficient-search-volume observations rather than literal absence of search interest

The preprocessing sequence is:

1. Standardize country files and monthly date format.
2. Rename Google Trends topic variables into compact snake-case names.
3. Treat selected zero values in Google Trends variables as missing where appropriate.
4. Apply short-gap linear interpolation using `limit=3` and `limit_direction="both"`.
5. Save the transparent cleaned EDA dataset as `input.csv`.
6. Fill any remaining edge missing values within country using forward/backward fill for `input_modeling.csv`.
7. Create variable-specific imputation flags before final modeling fill.
8. Reshape the wide modeling dataset into `input_modeling_long.csv` for visualization and cross-country comparison.

Interpretation of imputation flags:

| Flag value | Meaning |
|---:|---|
| 0 | The value was observed directly as a nonzero Google Trends value |
| 1 | The value was originally zero or missing and was filled during preprocessing/modeling preparation |

### Imputed observation counts by country

| country     |   anxiety_was_imputed |   depression_was_imputed |   stress_was_imputed |   insomnia_was_imputed |   mental_disorder_was_imputed |
|:------------|----------------------:|-------------------------:|---------------------:|-----------------------:|------------------------------:|
| Australia   |                     0 |                        0 |                    0 |                      0 |                             0 |
| Canada      |                     0 |                        0 |                    0 |                      0 |                             0 |
| Ireland     |                     4 |                        0 |                    3 |                     26 |                             4 |
| New_Zealand |                     0 |                        0 |                    3 |                     22 |                             0 |
| UK          |                     0 |                        0 |                    2 |                      0 |                             0 |
| US          |                     0 |                        0 |                    0 |                      0 |                             0 |

---

## SARIMAX intervention variables

The final SARIMAX intervention specification uses five pandemic-phase variables.

| Variable | Role in SARIMAX | Interpretation |
|---|---|---|
| `early_covid_shock` | short-run shock intervention | Captures the early pandemic shock from March 2020 to June 2020 |
| `covid_period` | COVID-period level intervention | Measures the average level shift during the COVID period |
| `covid_trend` | COVID-period slope intervention | Measures month-by-month increase, persistence, or decline during the COVID period |
| `post_covid_period` | post-COVID level intervention | Measures the average level shift after the COVID period |
| `post_covid_trend` | post-COVID slope intervention | Measures post-COVID normalization, recovery, decline, or continued growth |

The intervention model is interpreted as an explanatory model, not as the best pure forecasting model. Its purpose is to estimate whether anxiety search behavior changed during and after COVID after controlling for autoregressive and seasonal time-series structure.

---

## Forecasting and intervention roles

The final project separates two modeling roles.

| Role | Models | Purpose |
|---|---|---|
| Forecast evaluation | ARIMA, SARIMA, SARIMAX, Naive, Seasonal Naive | Tests whether Box-Jenkins models forecast the post-COVID test period better than simple persistence and seasonal benchmarks |
| Intervention explanation | SARIMAX | Estimates COVID and post-COVID level/trend effects after controlling for time-series dynamics |
| Counterfactual analysis | Pre-COVID SARIMA | Projects no-COVID dynamics into 2020-2025 and compares observed anxiety searches against the projected baseline |

The final results show that naive and seasonal-naive benchmarks dominate the full post-COVID test horizon, while SARIMA performs best for short horizons. SARIMAX is retained because it provides interpretable intervention evidence, not because it is the best long-horizon forecasting model.

---

## Modeling split

| Split | Period | Purpose |
|---|---|---|
| Train | 2004-01 to 2021-12 | Estimate candidate ARIMA/SARIMA/SARIMAX parameters |
| Validation | 2022-01 to 2023-05 | Select model specifications before the final test period |
| Test | 2023-06 to 2025-12 | Final out-of-sample post-COVID forecast evaluation |
| Counterfactual pre-COVID fit | 2004-01 to 2019-12 | Estimate no-COVID SARIMA baseline |
| Counterfactual forecast | 2020-01 to 2025-12 | Compare observed anxiety searches with no-COVID projected dynamics |

---

## Candidate-order logic from EDA

The EDA ACF/PACF and stationarity analysis motivates the ARIMA/SARIMA/SARIMAX candidate sets.

| EDA finding | Modeling implication |
|---|---|
| Anxiety search series are generally more stable after first differencing than in levels | Main ARIMA/SARIMA candidates use non-seasonal differencing `d = 1` |
| Strong short-run dependence appears around lag 1 | Include ARIMA-type candidates such as `(0,1,1)` |
| Some countries show mixed AR and MA short-lag behavior | Include candidates such as `(1,1,1)`, `(2,1,1)`, and `(1,1,2)` |
| Monthly ACF/PACF patterns show seasonal dependence around lags 12, 24, and 36 | Use seasonal period `s = 12` |
| Seasonal differencing is plausible but should not be imposed mechanically | Compare seasonal `D = 0` and `D = 1` candidates |
| Seasonal AR and MA structures are both plausible | Test seasonal AR and seasonal MA components |

Final model selection is not based on ACF/PACF alone. Candidate models are selected using validation forecast performance, checked using test-period forecast accuracy, and interpreted with residual diagnostics.

---

## Evaluation outputs

The forecast-evaluation notebook produces:

| Output file | Purpose |
|---|---|
| `04_best_test_forecast_model_by_country.csv` | Best model in the post-COVID test period for each country |
| `04_average_test_performance_by_model_class.csv` | Average RMSE and sMAPE by model class |
| `04_diebold_mariano_interpreted.csv` | Diebold-Mariano test results against the best benchmark |
| `04_diebold_mariano_summary.csv` | Summary of benchmark dominance by model class |
| `04_horizon_specific_average_errors.csv` | Forecast accuracy by horizon group |
| `04_rolling_origin_average_performance.csv` | Rolling-origin robustness evaluation |
| `04_forecast_evaluation_summary.csv` | Compact summary for report writing |

---

## Intervention and counterfactual outputs

The SARIMAX intervention and counterfactual notebook produces:

| Output file | Purpose |
|---|---|
| `05_sarimax_intervention_terms_long.csv` | Long-format SARIMAX intervention coefficient table |
| `05_significant_sarimax_intervention_terms.csv` | Statistically significant intervention terms at the 5% level |
| `05_full_sample_sarimax_residual_diagnostics.csv` | Ljung-Box, ARCH-LM, and Jarque-Bera diagnostics |
| `05_country_intervention_interpretation.csv` | Country-level interpretation of SARIMAX intervention results |
| `05_counterfactual_version_summary.csv` | Raw, capped, and logit counterfactual comparison |
| `05_counterfactual_by_country_phase.csv` | Counterfactual gap by country and phase |
| `05_logit_counterfactual_by_country_phase.csv` | Preferred bounded-scale counterfactual gap table |
| `05_sarimax_intervention_counterfactual_summary.csv` | Compact summary for report writing |

---

## Cross-national comparison outputs

The cross-national comparison notebook produces:

| Output file | Purpose |
|---|---|
| `06_crossnational_country_scorecard.csv` | Country-level scorecard combining forecast, intervention, diagnostics, and counterfactual evidence |
| `06_crossnational_country_scorecard_labeled.csv` | Report-ready labeled version of the country scorecard |
| `06_crossnational_summary_for_report.csv` | Compact cross-national summary table |
| `06_final_findings_for_report.csv` | Final findings used in the discussion and conclusion |

---

## Final interpretation rule

The project should not claim that ARIMA, SARIMA, or SARIMAX generally outperform simple benchmarks over the full post-COVID test period

The correct interpretation is:

- Naive and seasonal-naive benchmarks dominate the full post-COVID long-horizon forecast evaluation
- SARIMA is still useful for short-horizon dynamics and seasonal structure
- SARIMAX is central for estimating COVID and post-COVID intervention effects
- The evidence does not support a uniform positive COVID-period increase in anxiety searches
- The strongest conclusion is cross-national heterogeneity, persistence, seasonality, and post-COVID normalization or decline

