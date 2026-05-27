# Modeling Data Dictionary

## Project

**Explaining the COVID-19 Effect on Mental Health Search Behavior: A Comparative SARIMA/SARIMAX Intervention Analysis Across Six Anglophone Countries.**

This dictionary describes the modeling-ready data created by `01_Preprocessing.ipynb` and used by `03_Modeling_COVID_Period_Final.ipynb`.

---

## Files created

| File | Format | Purpose |
|---|---|---|
| `input.csv` | wide country-panel | transparent cleaned EDA dataset; may retain long leading missing values |
| `input_modeling.csv` | wide country-panel | complete modeling dataset for ARIMA, SARIMA, and SARIMAX |
| `input_modeling_long.csv` | long format | plotting, cross-country/variable comparisons, robustness summaries |
| `modeling_data_dictionary.md` | markdown | documentation of variables, imputation, COVID intervention design, and model alignment |

---

## Sample coverage

| Item | Value |
|---|---|
| Countries | Australia, Canada, Ireland, New_Zealand, UK, US |
| Frequency | Monthly |
| Start | 2004-01 |
| End | 2025-12 |
| Months per country | 264 |
| Total rows in `input_modeling.csv` | 1,584 |
| Total rows in `input_modeling_long.csv` | 7,920 |

---

## COVID-period definition

| Phase | Period | Months per country | Modeling interpretation |
|---|---:|---:|---|
| Pre-COVID | 2004-01 to 2019-12 | 192 | baseline period |
| COVID period | 2020-01 to 2023-05 | 41 | disruption/intervention period |
| Post-COVID | 2023-06 to 2025-12 | 31 | post-intervention adjustment period |

This phase definition must remain identical across preprocessing, EDA, and modeling.

---

## Variable definitions

| Variable          | Definition                                                                                                                |
|:------------------|:--------------------------------------------------------------------------------------------------------------------------|
| date              | Monthly timestamp stored as first day of month. Example: 2020-01-01 represents January 2020.                              |
| country           | Country identifier: Australia, Canada, Ireland, New_Zealand, UK, or US.                                                   |
| anxiety           | Google Trends topic-level index for Anxiety.                                                                              |
| depression        | Google Trends topic-level index for Major depressive disorder.                                                            |
| stress            | Google Trends topic-level index for Psychological Stress.                                                                 |
| insomnia          | Google Trends topic-level index for Insomnia.                                                                             |
| mental_disorder   | Google Trends topic-level index for Mental disorder.                                                                      |
| *_was_imputed     | Flag equal to 1 if the corresponding raw value was zero/missing and was filled during preprocessing/modeling preparation. |
| year              | Calendar year extracted from date.                                                                                        |
| month             | Calendar month extracted from date.                                                                                       |
| time_index        | Monthly time index within each country, starting at 0 in 2004-01.                                                         |
| analysis_phase    | Categorical phase: pre_covid, covid_period, or post_covid.                                                                |
| covid_period      | Dummy equal to 1 from 2020-01 to 2023-05, otherwise 0.                                                                    |
| covid_trend       | Monthly counter inside COVID period: 1, 2, ..., 41; 0 outside COVID period.                                               |
| post_covid_period | Dummy equal to 1 from 2023-06 onward, otherwise 0.                                                                        |
| post_covid_trend  | Monthly counter inside post-COVID period: 1, 2, ..., 31; 0 before post-COVID period.                                      |

---

## Imputation policy

Google Trends zeros are treated as potentially low-volume or insufficient-search-volume observations rather than literal absence of search interest.

Preprocessing uses this sequence:

1. Convert zeros in Google Trends variables to missing values.
2. Apply linear interpolation to short gaps using `limit=3` and `limit_direction='both'`.
3. Save the transparent EDA dataset as `input.csv`.
4. Fill any remaining edge missing values within each country using forward/backward fill for `input_modeling.csv`.
5. Create variable-specific imputation flags before final modeling fill.

Interpretation of imputation flags:

- `0`: value was directly observed as a nonzero Google Trends value.
- `1`: value was originally zero/missing and was filled during preprocessing/modeling preparation.

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

The modeling notebook uses the following exogenous variables:

| Variable | Role in SARIMAX | Interpretation |
|---|---|---|
| `covid_period` | level intervention | average level shift during COVID period |
| `covid_trend` | slope intervention | month-by-month persistence or fading during COVID period |
| `post_covid_period` | level intervention | average level shift after COVID period ends |
| `post_covid_trend` | slope intervention | post-COVID recovery, normalization, or continued increase |

The trend counters use monthly units. For example, `covid_trend = 12` means the 12th month of the COVID period.

---

## Modeling alignment

`03_Modeling.ipynb` uses:

| Split | Period | Purpose |
|---|---|---|
| Train | 2004-01 to 2021-12 | estimate candidate model parameters |
| Validation | 2022-01 to 2023-05 | select ARIMA/SARIMA/SARIMAX specification |
| Test | 2023-06 to 2025-12 | final out-of-sample forecast evaluation |
| Counterfactual pre-COVID fit | 2004-01 to 2019-12 | estimate no-COVID SARIMA baseline |
| Counterfactual forecast | 2020-01 to 2025-12 | compare observed data with no-COVID forecast |

---

## Candidate-order logic from EDA

The EDA ACF/PACF analysis motivates the model candidate set used in `03_Modeling_COVID_Period_Final.ipynb`:

| EDA finding | Modeling implication |
|---|---|
| First-differenced anxiety is more stationary than levels | set non-seasonal `d = 1` |
| ACF has strong lag-1 dependence | include `ARIMA(0,1,1)` |
| ACF and PACF both show short-lag signals | include `ARIMA(1,1,1)` |
| Some countries show lag 1–2 dependence/noisier short-run structure | include `ARIMA(2,1,1)` and `ARIMA(1,1,2)` |
| ACF/PACF show seasonal spikes at 12, 24, 36 | use monthly seasonal period `s = 12` |
| Seasonal differencing is plausible but not automatic | compare seasonal `D = 0` and `D = 1` |
| Seasonal AR/MA behavior appears in ACF/PACF | test seasonal AR and MA terms |

Final model selection is not based on visual ACF/PACF alone. It is based on validation forecast errors, test forecast errors, AIC/BIC, and residual diagnostics.
