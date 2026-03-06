# ERCOT Zone-Level Electricity Demand Forecasting

This repository contains a time-series analysis project focused on predicting hourly electricity demand (MW) across the eight weather zones managed by the **Electric Reliability Council of Texas (ERCOT)**. The model generates 8,760 hourly predictions per zone for the year 2025.

## Project Overview

ERCOT manages approximately 90% of Texas's electricity, serving over 26 million customers. Accurate forecasting is critical for grid stability; under-forecasting risks blackouts, while over-forecasting leads to unnecessary generation costs.

---

## Methodology

The project employs a two-stage modeling pipeline to handle unknown future weather and load variables.

### 1. Data Processing & Feature Engineering

* 
**Data Sources:** Combined hourly load data (2018–2024) with hourly weather data (Temperature, Humidity, Wind, Solar) via the Open-Meteo API.


* 
**Timestamp Repair:** Handled Daylight Savings Time (DST) through interpolation and averaging, and corrected non-standard "24:00" timestamps to "00:00".


* **Feature Engineering:**
* 
**Cyclic Encoding:** Sin/Cos transforms for hours and months to preserve periodic continuity.


* 
**Lag Features:** 24h, 48h, and 168h lags to capture autocorrelation.


* 
**Weather Indicators:** Developed Heating Degree Days (HDD), Cooling Degree Days (CDD), and Heat Index (Temp x Humidity) to capture non-linear demand responses to extreme weather.





### 2. Modeling Pipeline

The forecasting is executed in two primary stages:

#### Stage A: Weather Forecasting

Because actual weather for 2025 is unknown at the time of forecast, a univariate **STL + ARIMA** model was built to generate the 2025 weather inputs.

* 
**Decomposition:** Used `msts` for dual seasonal periods (weekly and annual).


* 
**Hourly Reconstruction:** Historical hourly deviation profiles were added back to daily mean forecasts to restore intraday shapes.



#### Stage B: Load Forecasting

Three parallel modeling architectures were evaluated:

* 
**SARIMAX:** Automatically selected optimal $(p,d,q)$ orders per zone, utilizing 33 external regressors (xreg) including weather and calendar flags.


* 
**XGBoost:** A direct rolling forecast strategy (572 models) re-issuing fresh 72h forecasts every 24h across the test year.


* 
**MSTL + LightGBM (Hybrid):** The preferred architecture.


* 
**Layer 01 (MSTL):** Decomposes historical demand into Trend, Daily, and Weekly components to extract seasonal shapes as numeric features.


* 
**Layer 02 (LightGBM):** A Gradient Boosted Decision Tree model that receives MSTL shape features plus weather and calendar inputs to predict the final MW load level.





---

## Core Assumptions

1. 
**Non-Linearity:** Raw temperature cannot capture the U-shaped relationship where both extreme heat and cold drive demand.


2. 
**Structural Growth:** Rapid growth in zones like **FWEST** and **NORTH** (possibly due to data centers) requires models to be retrained on the most recent data to capture shifting baselines.


3. 
**Binding Constraints:** Errors in the weather forecast propagate directly into demand errors, placing a hard ceiling on achievable accuracy.



---

## Technical Stack

* 
**Languages:** Python (Pandas, Scikit-Learn, XGBoost, LightGBM) and R (forecast, seasonal packages).


* 
**Statistical Tests:** Augmented Dickey-Fuller (ADF) and KPSS tests were used to assess stationarity.



## Team

* [Faith Lucy Kirabo](https://www.linkedin.com/in/faithlucykirabo/) 


* Beste 


* Olivia Yang 
