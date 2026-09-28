# Beijing Air Quality Forecasting

This project uses hourly measurements from Beijing's multi-site air-quality network to forecast **PM2.5 one hour ahead**. It covers data cleaning, exploratory analysis, time-based feature engineering, chronological model evaluation, and interpretation.

## Project files

```text
beijing_air_quality/
├── README.md
├── Beijing_Air_Quality_Forecasting.ipynb
└── data/
    └── beijing_air_quality.parquet
```

The notebook expects the data at the relative path `data/beijing_air_quality.parquet`. Keep the notebook and `data/` folder together when moving or publishing the project.

## Dataset

The dataset is OpenML dataset [42933: Beijing Multi-Site Air Quality](https://www.openml.org/d/42933). It contains **420,768 hourly observations** from 12 monitoring stations, from March 1, 2013 through February 28, 2017. Variables include PM2.5, PM10, SO2, NO2, CO, O3, temperature, pressure, dew point, rainfall, wind speed, and wind direction.

The original dataset is described in Zhang et al. (2017), “Cautionary Tales on Air-Quality Improvement in Beijing,” *Proceedings of the Royal Society A*, 473(2205), 20170457.

## Forecast design

For each station and hour, the notebook predicts the **next hour's PM2.5**. Inputs include current-hour pollutant and weather observations, station identity, calendar features, and PM2.5 history (1-hour and 3-hour lags, 24-hour lag, and a trailing 24-hour mean). The rolling mean is shifted so it does not include the target hour.

Observations are split by timestamp: the first 80% of time is used for training and the final 20% for testing. This tests performance on a later period at stations present in the training data. It does not measure performance at a completely unseen station.

## Data preparation

- The row-number field is removed, and timestamp fields are combined into a datetime.
- Duplicate station-hour rows are removed if present.
- Missing PM2.5 targets and rows without the required lag history are excluded because their outcomes or forecast history are unavailable.
- Zero readings are retained; zero rainfall and some zero pollutant readings can be physically valid.
- IQR outliers are counted and inspected, not automatically removed. High pollution values can represent real episodes that matter to forecasting.
- Missing predictors are filled using medians calculated from the training set only, then the same medians are applied to test data.
- Wind direction is represented with sine/cosine features to respect its circular nature. Calendar features are also encoded cyclically.

## Models and evaluation

The notebook compares a persistence baseline (next-hour forecast equals the current PM2.5 reading) with Linear Regression, Ridge Regression, Random Forest, and XGBoost. RMSE is the primary model-selection metric because it penalizes large errors; MAE and R² are also reported. A chronological holdout is used rather than a random split to avoid training on observations later than those being tested.

In the executed notebook, **Random Forest** had the lowest test RMSE:

| Metric | Result |
|---|---:|
| RMSE | 18.02 µg/m³ |
| MAE | 9.39 µg/m³ |
| R² | 0.954 |

These results apply to this split and this one-hour forecast setup. Error analysis shows larger misses at higher PM2.5 concentrations, so the model is less precise during high-pollution episodes. Permutation/feature importance indicates predictive associations, not causal effects.

## Run the notebook

Use Python 3.10 or later. From this project directory, install the libraries used by the notebook and launch Jupyter:

```bash
python -m pip install pandas numpy matplotlib seaborn scikit-learn xgboost pyarrow jupyter
jupyter notebook
```

Open `Beijing_Air_Quality_Forecasting.ipynb` and run the cells from top to bottom. The parquet reader requires `pyarrow`. The model cells can take some time because the dataset is large.

## Limitations and next steps

This is a one-hour-ahead forecast using current-hour pollutant and weather readings. It is not a multi-day forecast. In a real deployment, confirm those current-hour measurements are available when the forecast is issued. Results are for stations represented in training, and the final time block is reserved for testing. Time-aware rolling validation and a separately defined forecast horizon would be useful next steps.

## Sources and attribution

- Dataset: [OpenML dataset 42933](https://www.openml.org/d/42933).
- Original dataset paper: Zhang et al. (2017), citation above.
- Assignment requirements: the project brief provided for this work.
- The notebook documents the forecasting formulation and modeling workflow. Add any course-required disclosure of external assistance or code before submission.