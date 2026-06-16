# Forecasting USD/VND Exchange Rate Volatility with Global and Domestic Financial Shocks

This repository contains the full empirical workflow for a time-series research project on USD/VND exchange-rate volatility. The project studies whether global and domestic financial shocks improve the modelling and forecasting of USD/VND volatility, using GARCH-family models, EGARCH-X extensions, HAC regressions, and weekly recursive SVAR analysis.

The repository is designed as a reproducible academic research workflow. It includes raw and processed data, feature-engineering notebooks, statistical diagnostics, audited model-selection code, output tables, and the final research report.

## Research Objective

The main objective is to evaluate the volatility dynamics of the USD/VND exchange rate and examine whether external and domestic financial shocks contain useful information for forecasting and interpreting exchange-rate volatility.

The project addresses four empirical questions:

1. How persistent and heteroskedastic are daily USD/VND returns?
2. Which conditional-volatility model performs best under a chronological train-validation-test framework?
3. Do global and domestic financial shocks improve volatility forecasting relative to standard GARCH-family models?
4. How do weekly financial shocks transmit to USD/VND returns and conditional volatility in a recursive SVAR framework?

## Repository Structure

```text
final-timeseries/
│
├── data/
│   ├── raw/
│   │   ├── dataset.csv
│   │   ├── exchangerate.csv
│   │   ├── exchangerate2019.csv
│   │   └── VNIDEX.csv
│   │
│   └── processed/
│       ├── processed_data.csv
│       ├── model_data_train.csv
│       ├── model_data_valid.csv
│       └── model_data_test.csv
│
├── notebooks/
│   ├── EDA.ipynb
│   ├── FE.ipynb
│   ├── test.ipynb
│   └── model_test_audited.ipynb
│
├── outputs/
│   ├── table_00_data_audit.csv
│   ├── table_01_train_arma_selection.csv
│   ├── table_06_validation_accuracy.csv
│   ├── table_12_test_accuracy_locked_selection.csv
│   ├── table_17_hac_shock_volatility_coefficients.csv
│   ├── table_18_hac_shock_volatility_model_summary.csv
│   ├── table_19_weekly_svar_standardization.csv
│   ├── table_20_weekly_recursive_svar_summary.csv
│   ├── table_21_weekly_recursive_svar_granger_causality.csv
│   ├── table_22_weekly_recursive_svar_structural_irf.csv
│   └── tables/
│
├── reports/
│   └── _TS__PHAMKHANHLINH.pdf
│
└── README.md
```

## Data Description

The project uses daily financial and exchange-rate data from 2010 to 2025. The main dependent variable is the daily USD/VND return, constructed from the USD/VND exchange rate.

### Core Variables

| Variable             | Description                   | Role                                |
| -------------------- | ----------------------------- | ----------------------------------- |
| `usd_vnd`            | Nominal USD/VND exchange rate | Source variable for FX returns      |
| `fx_return`          | Daily log return of USD/VND   | Main dependent variable             |
| `realized_var_proxy` | Realized variance proxy       | Forecast-evaluation proxy           |
| `vix`                | CBOE Volatility Index         | Global risk proxy                   |
| `us_10y`             | U.S. 10-year Treasury yield   | Global interest-rate condition      |
| `dxy`                | U.S. Dollar Index             | Global USD strength                 |
| `wti_oil`            | WTI crude oil price           | Commodity-market shock              |
| `vnindex`            | Vietnam stock-market index    | Domestic financial-market condition |

### Shock Variables

The project constructs financial-shock variables from raw market series:

| Shock Variable   | Transformation                         | Interpretation               |
| ---------------- | -------------------------------------- | ---------------------------- |
| `vix_change`     | First difference of VIX                | Global risk shock            |
| `us10y_change`   | First difference of U.S. 10-year yield | U.S. rate shock              |
| `dxy_return`     | Log return of DXY                      | Global USD shock             |
| `oil_change`     | Change in WTI oil price                | Commodity shock              |
| `vnindex_return` | Log return of VNINDEX                  | Domestic equity-market shock |

For volatility forecasting, the shock variables are lagged by one trading day:

```text
vix_change_lag1
us10y_change_lag1
dxy_return_lag1
oil_change_lag1
vnindex_return_lag1
```

This lagging structure prevents look-ahead bias and ensures that only information available before the forecast date is used.

## Chronological Sample Split

The modelling workflow uses a strict chronological split:

| Split      |                   Period | Observations | Use                                  |
| ---------- | -----------------------: | -----------: | ------------------------------------ |
| Train      | 2010-01-06 to 2021-12-31 |        3,014 | ARMA and volatility-model estimation |
| Validation | 2022-01-03 to 2023-12-29 |          501 | Model selection                      |
| Test       | 2024-01-02 to 2025-12-31 |          504 | Final out-of-sample evaluation       |

The split chronology audit confirms:

* Dates are sorted in ascending order.
* There are no duplicate dates.
* There are no missing required values.
* There are no non-finite numeric values.
* Train, validation, and test periods do not overlap.

## Methodological Pipeline

The empirical pipeline follows this sequence:

```text
Raw market data
        ↓
Data cleaning and transformation
        ↓
Feature engineering with lagged shocks
        ↓
Preliminary time-series diagnostics
        ↓
ARMA mean-model selection on training data
        ↓
Volatility-model estimation on training data
        ↓
Validation-based model selection
        ↓
Locked-model test evaluation
        ↓
Robustness checks
        ↓
Supplementary HAC and weekly SVAR shock analysis
```

## Notebooks

### `EDA.ipynb`

Performs exploratory data analysis on the raw daily dataset.

Main tasks:

* Load and inspect raw data.
* Check date coverage.
* Check duplicate dates.
* Check missing values.
* Examine outliers.
* Visualize key market variables.
* Provide descriptive analysis for the raw exchange-rate and financial-market series.

### `FE.ipynb`

Constructs the leakage-safe modelling dataset.

Main tasks:

* Load processed daily data.
* Apply chronological train-validation-test split.
* Construct daily FX returns.
* Construct realized variance proxy.
* Construct financial-shock variables.
* Lag financial shocks by one trading day.
* Export final model-ready files:

  * `model_data_train.csv`
  * `model_data_valid.csv`
  * `model_data_test.csv`

### `test.ipynb`

Runs preliminary statistical diagnostics.

Main tests:

* ADF stationarity test.
* KPSS stationarity test.
* Ljung-Box serial-correlation test.
* ARCH-LM conditional-heteroskedasticity test.
* Jarque-Bera normality test.
* VIF multicollinearity diagnostics.
* ACF/PACF visual diagnostics.
* Distributional diagnostics for return and shock variables.

### `model_test_audited.ipynb`

Runs the main audited modelling pipeline.

Main tasks:

* Validate split-level data integrity.
* Select ARMA(p, q) model on training data.
* Estimate GARCH-family and EGARCH-X volatility models.
* Compare validation forecasts using QLIKE, MSE, RMSE, and MAE.
* Lock the validation-selected model.
* Evaluate all models on the test set without using the test set for selection.
* Run robustness checks.
* Run dynamic HAC regressions.
* Run weekly recursive SVAR analysis.

## Models Estimated

### Mean Model

The conditional mean of USD/VND returns is modelled using ARMA(p, q) candidates. The audited ARMA grid search evaluates p and q up to 5.

The selected train-sample mean model is:

```text
ARMA(4,5)
```

This specification passes the residual Ljung-Box diagnostic screen used in the model-selection pipeline.

### Volatility Models

The volatility-model comparison includes:

| Model                    | Description                                          |
| ------------------------ | ---------------------------------------------------- |
| `GARCH(1,1)-t`           | Standard GARCH with Student-t innovations            |
| `GJR-GARCH(1,1)-t`       | Asymmetric GARCH with Student-t innovations          |
| `EGARCH(1,1)-t`          | Exponential GARCH with Student-t innovations         |
| `Custom EGARCH no-X-t`   | Custom EGARCH likelihood without external regressors |
| `EGARCH-X signed lag1-t` | EGARCH-X using signed lagged financial shocks        |
| `EGARCH-X abs lag1-t`    | EGARCH-X using absolute lagged financial shocks      |
| `EWMA_0.94`              | Exponentially weighted moving average benchmark      |
| `HistoricalVariance`     | Historical-variance benchmark                        |
| `RollingVariance22`      | 22-day rolling-variance benchmark                    |

## Model Selection Design

The project uses a locked chronological evaluation design:

1. Train models on the training sample.
2. Select the best model using validation-sample QLIKE.
3. Lock the selected model.
4. Refit on the development sample where required.
5. Evaluate on the test sample.
6. Do not use test performance to select the final model.

This design reduces selection bias and makes the reported test evaluation closer to a genuine out-of-sample exercise.

## Main Forecasting Results

### Validation Selection

The validation sample selects:

```text
GJR-GARCH(1,1)-t
```

Validation performance by QLIKE:

| Rank | Model                  |   QLIKE |
| ---: | ---------------------- | ------: |
|    1 | GJR-GARCH(1,1)-t       | -2.6783 |
|    2 | EGARCH-X abs lag1-t    | -2.6684 |
|    3 | EGARCH-X signed lag1-t | -2.6046 |
|    4 | Custom EGARCH no-X-t   | -2.5695 |
|    5 | EGARCH(1,1)-t          | -2.5694 |

The final locked forecasting model is therefore:

```text
ARMA(4,5) – GJR-GARCH(1,1)-t
```

### Test Evaluation

On the test sample, all models are evaluated for reporting, but the test set is not used for model selection.

Test QLIKE ranking:

| Rank | Model                  |   QLIKE | Role                             |
| ---: | ---------------------- | ------: | -------------------------------- |
|    1 | EGARCH(1,1)-t          | -2.8511 | Reporting comparator             |
|    2 | Custom EGARCH no-X-t   | -2.8510 | Reporting comparator             |
|    3 | EGARCH-X abs lag1-t    | -2.8485 | Reporting comparator             |
|    4 | EGARCH-X signed lag1-t | -2.8482 | Reporting comparator             |
|    5 | GARCH(1,1)-t           | -2.8059 | Reporting comparator             |
|    6 | GJR-GARCH(1,1)-t       | -2.7751 | Locked validation-selected model |
|    7 | EWMA_0.94              | -2.7594 | Benchmark                        |
|    8 | RollingVariance22      | -2.6962 | Benchmark                        |
|    9 | HistoricalVariance     | -2.4523 | Benchmark                        |

The test-sample ranking shows that EGARCH-type models perform strongly ex post. However, the locked model remains GJR-GARCH(1,1)-t because it was selected using validation data only.

## Robustness Checks

The project includes robustness analysis using an alternative realized-volatility proxy based on squared returns.

Under this alternative proxy, EGARCH-X with absolute lagged shocks ranks first on the test sample:

```text
EGARCH-X abs lag1-t
```

This suggests that shock magnitude may contain useful information for volatility evaluation under some proxy definitions. However, the main model-selection rule remains based on the validation-sample QLIKE ranking.

## HAC Shock-Volatility Regressions

Dynamic HAC regressions are used to examine whether lagged standardized financial shocks explain selected conditional volatility.

The dependent variable is:

```text
log_selected_cond_vol
```

The main control is lagged conditional volatility:

```text
log_selected_cond_vol_lag1
```

The HAC regressions show:

* Strong volatility persistence.
* Lagged conditional volatility is highly significant.
* The signed-shock specification adds limited incremental explanatory power.
* The absolute-shock specification shows evidence that the magnitude of VNINDEX shocks is associated with future conditional volatility.
* The model R-squared is approximately 0.808 across dynamic specifications.

These results support the interpretation that USD/VND volatility is highly persistent, while external and domestic shocks provide supplementary information rather than replacing volatility persistence as the dominant driver.

## Weekly Recursive SVAR Analysis

The repository also includes weekly recursive SVAR analysis to examine dynamic shock transmission.

### Weekly SVAR Variables

The weekly SVAR uses the following recursive ordering:

```text
vix_change
→ us10y_change
→ dxy_return
→ oil_change
→ vnindex_return
→ target variable
```

Two target variables are considered:

1. `fx_return`
2. `log_selected_cond_vol`

### SVAR Design

The weekly SVAR is estimated using:

```text
VAR(1)
```

The weekly sample includes:

```text
729 observations
```

Both weekly SVAR systems are stable under the reported model summary.

### Granger-Causality Results

For weekly USD/VND returns, the following shocks are significant at the 5 percent level:

| Shock            | Target      | p-value | Significant |
| ---------------- | ----------- | ------: | ----------- |
| `us10y_change`   | `fx_return` |  0.0012 | Yes         |
| `dxy_return`     | `fx_return` |  0.0092 | Yes         |
| `vnindex_return` | `fx_return` |  0.0424 | Yes         |

For weekly selected conditional volatility, none of the tested shocks are significant at the 5 percent level in the reported Granger-causality table.

This suggests that weekly financial shocks have clearer predictive content for USD/VND returns than for the selected conditional-volatility series.

## Key Output Files

| File                                                   | Purpose                                             |
| ------------------------------------------------------ | --------------------------------------------------- |
| `table_00_data_audit.csv`                              | Split-level data integrity audit                    |
| `table_00b_split_chronology_audit.csv`                 | Chronology and overlap audit                        |
| `table_01_train_arma_selection.csv`                    | ARMA candidate comparison                           |
| `table_02_train_volatility_fit.csv`                    | Train-sample volatility-model fit                   |
| `table_02b_train_volatility_residual_diagnostics.csv`  | Train-sample residual diagnostics                   |
| `table_05_validation_daily_forecasts.csv`              | Daily validation forecasts                          |
| `table_06_validation_accuracy.csv`                     | Validation forecast accuracy and locked model       |
| `table_08_development_volatility_fit.csv`              | Development-sample volatility refit                 |
| `table_11_test_daily_forecasts.csv`                    | Daily test forecasts                                |
| `table_12_test_accuracy_locked_selection.csv`          | Test evaluation under locked model-selection design |
| `table_13_test_return_squared_robustness.csv`          | Test robustness under squared-return proxy          |
| `table_16_egarchx_development_coefficients.csv`        | EGARCH-X coefficient estimates                      |
| `table_17_hac_shock_volatility_coefficients.csv`       | HAC shock-volatility regression coefficients        |
| `table_18_hac_shock_volatility_model_summary.csv`      | HAC model-level summaries                           |
| `table_19_weekly_svar_standardization.csv`             | Weekly SVAR standardization parameters              |
| `table_20_weekly_recursive_svar_summary.csv`           | Weekly recursive SVAR model summary                 |
| `table_21_weekly_recursive_svar_granger_causality.csv` | Weekly SVAR Granger-causality tests                 |
| `table_22_weekly_recursive_svar_structural_irf.csv`    | Weekly SVAR structural impulse-response output      |

## Installation

This repository does not currently include a `requirements.txt` file. The following environment is sufficient for running the notebooks.

```bash
python -m venv .venv
source .venv/bin/activate

pip install pandas numpy scipy statsmodels arch matplotlib seaborn tqdm ipykernel jupyter
python -m ipykernel install --user --name final-timeseries --display-name "Python (final-timeseries)"
```

For Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1

pip install pandas numpy scipy statsmodels arch matplotlib seaborn tqdm ipykernel jupyter
python -m ipykernel install --user --name final-timeseries --display-name "Python (final-timeseries)"
```

## How to Reproduce the Results

Because the notebooks use relative paths such as `../data/processed`, run Jupyter from inside the `notebooks` directory:

```bash
cd notebooks
jupyter lab
```

Run the notebooks in this order:

```text
1. EDA.ipynb
2. FE.ipynb
3. test.ipynb
4. model_test_audited.ipynb
```

Expected input and output flow:

```text
data/raw/dataset.csv
        ↓
data/processed/processed_data.csv
        ↓
FE.ipynb
        ↓
data/processed/model_data_train.csv
data/processed/model_data_valid.csv
data/processed/model_data_test.csv
        ↓
test.ipynb and model_test_audited.ipynb
        ↓
outputs/*.csv
outputs/tables/*
reports/_TS__PHAMKHANHLINH.pdf
```

## Reproducibility Safeguards

The audited model notebook includes several safeguards:

* Strict chronological splitting.
* Split-level data audit.
* Duplicate-date checks.
* Missing-value checks.
* Non-finite-value checks.
* Positive-variance validation.
* ARMA convergence audit.
* Residual autocorrelation diagnostics.
* ARCH effect diagnostics.
* Standardized residual diagnostics.
* Validation-only model locking.
* Test-sample non-selection flag.
* Separate robustness checks.

These safeguards are important because volatility-forecasting models are sensitive to leakage, overlapping samples, unstable variance forecasts, and test-set selection bias.

## Interpretation Notes

The empirical results should be interpreted as evidence from a controlled historical forecasting experiment rather than a real-time trading system.

Important caveats:

* Conditional volatility is latent and must be evaluated using proxies.
* Results depend on the selected realized-variance proxy.
* Daily financial data may reflect market-calendar and liquidity differences.
* Test-sample rankings are reported for transparency but are not used to select the final locked model.
* The repository is structured for academic analysis, not for production deployment.

## Project Status

Current status:

```text
Academic research repository
Reproducible notebook-based workflow
Final report included
Output tables included
No production API
No package release
```

## Suggested Citation

```text
Pham Khanh Linh. Forecasting USD/VND Exchange Rate Volatility with Global and Domestic Financial Shocks: Evidence from GARCH-Family and EGARCH-X Models. Research report and reproducible code repository.
```

## License

No explicit license file is currently included in this repository. Add a license before public reuse, redistribution, or external collaboration.
