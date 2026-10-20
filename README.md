# LoRaWAN Indoor Distance Estimation via Kalman-Filtered RSSI and Log-Distance Path Loss Modeling

## Overview

This repository implements a modular indoor LoRaWAN localization workflow for time-ordered data preparation, RSSI smoothing, path loss modeling, and distance estimation.

The pipeline combines a self-tuning one-dimensional [Kalman filter](https://ani.stat.fsu.edu/~jfrade/HOMEWORKS/STA5107/presentation/sta5107-present/Kalman%20Filter/papers/kalman.pdf) with log-distance multi-wall path loss modeling to improve distance estimation from noisy RSSI measurements.

## Data

The notebooks read `../all_data_files/cleaned_dataset_per_device.csv`, the v4 candidate of the cleaned dataset, not yet released on Zenodo (md5 `638d918ffc3a15bb10564c852df6e6b0`): 2,660,274 uplinks from 1 October 2024 to 30 September 2025, SF7 to SF12. It holds the rows of file 3 of Zenodo v3 ([10.5281/zenodo.23127026](https://doi.org/10.5281/zenodo.23127026)) and adds the column `tx_power`. The end devices leave LoRaWAN's ADR on and never set a power, so The Things Network chose the transmit power of each uplink (14 dBm for 66.9 % of the rows, 12 dBm for 32.5 %, 10 dBm or less for 0.65 %). The path loss `exp_pl` is computed from that power, whereas Zenodo v3 assumes 14 dBm for every uplink (0.68 dB higher on average). Notebooks 00, 02 and 05 refer the RSSI to 14 dBm (`rssi + 14 - tx_power`) before they use it.

## Core Formulations

The pipeline first smooths RSSI with an adaptive one-dimensional Kalman filter, then converts the filtered signal to path loss for modeling.

**Adaptive Kalman Filtering**

The Things Network's ADR set the transmit power $P_{TX,k}$ of each uplink (`tx_power`), so the filter's measurement is the RSSI referred to $P_{TX} = 14$ dBm, $z_k = RSSI_k + P_{TX} - P_{TX,k}$. For each device, it is modeled with a random-walk state update:

$$
\hat{x}_{k|k-1} = \hat{x}_{k-1|k-1}, \qquad P_{k|k-1} = P_{k-1|k-1} + Q
$$

$$
\nu_k = z_k - \hat{x}_{k|k-1}, \qquad S_k = P_{k|k-1} + R_k
$$

$$
K_k = \frac{P_{k|k-1}}{S_k}, \qquad
\hat{x}_{k|k} = \hat{x}_{k|k-1} + K_k \nu_k
$$

$$
P_{k|k} = (1 - K_k)P_{k|k-1}
$$

The measurement-noise term is updated from the normalized innovation:

$$
\alpha_k = \mathrm{clip}\left(\frac{\nu_k^2}{S_k}, \gamma_{\min}, \gamma_{\max}\right)
$$

$$
R_{k+1} = \mathrm{clip}\left(\lambda R_k + (1-\lambda)\alpha_k R_k,\; R_{\min},\; R_{\max}\right)
$$

Filtered RSSI is then converted to path loss through the system offset at $P_{TX} = 14$ dBm:

$$
PL_{\mathrm{filtered}} = P_{TX} - CL_{TX} + G_{TX} + G_{RX} - RSSI_{\mathrm{filtered}}
$$

Path loss is modeled using two formulations.

**MWM**

$$
PL(d) = PL(d_0) + 10n\log_{10}\left(\frac{d}{d_0}\right) + W_cL_c + W_wL_w + \epsilon
$$

**MWM-EP**

$$
PL(d) = PL(d_0) + 10n\log_{10}\left(\frac{d}{d_0}\right) + 20\log_{10}(f) + W_cL_c + W_wL_w + \sum_{j=1}^{5}\theta_jE_j + \epsilon
$$

Distance is then obtained by inversion of the fitted path loss models.

## Workflow

1. [01_Data Preparation.ipynb](01_Data%20Preparation.ipynb)  
   Reads the cleaned dataset, performs the time-aware train/test split, and writes `Data Files/train.csv` and `Data Files/test.csv` (with `tx_power`).

2. [02_Kalman_Filtering.ipynb](02_Kalman_Filtering.ipynb)  
   Refers the RSSI to 14 dBm, applies the adaptive Kalman filter per device and writes `Data Files/train_kf.csv` and `Data Files/test_kf.csv` (their `rssi` is the RSSI referred to 14 dBm).

3. [03_Path_Loss_Modeling.ipynb](03_Path_Loss_Modeling.ipynb)  
   Fits the MWM and MWM-EP models on raw and filtered path loss, then saves fitted parameters and summary metrics.

4. [04_Distance_Estimation.ipynb](04_Distance_Estimation.ipynb)  
   Loads the fitted path loss parameters, performs model inversion, and evaluates distance estimation performance.

5. [05_Robustness_Checks.ipynb](05_Robustness_Checks.ipynb)  
   Rebuilds the pipeline from the cleaned dataset (and checks it against the saved results of step 4), then tests the ranging results beyond the six fitted links: leave-one-link-out ranging, per-link calibration from the last N packets, removal of the SF and channel shifts, day-block bootstrap intervals, baselines (causal smoothing windows, a Kalman filter with tuned Q, a constant guess, LightGBM rangers) and a censoring check from frame-counter gaps. Needs `lightgbm` and writes `Figures/robustness_calibration_smoothing.png` and `.pdf`.

Optional exploratory analysis (STL decomposition, Granger causality tests and a VAR model of the hourly RSSI, referred to 14 dBm, against the five sensors and the SNR) is available in [00_Time_Series_Analysis.ipynb](00_Time_Series_Analysis.ipynb).

## Main Artifacts

- `Data Files/train.csv`, `Data Files/test.csv`
- `Data Files/train_kf.csv`, `Data Files/test_kf.csv`
- `Data Files/path_loss_params.npz`
- `Data Files/path_loss_metrics.csv`
- `Data Files/distance_metrics.csv`
- `Data Files/distance_predictions.csv`

## Notes

- The input dataset path (see Data) is set in [01_Data Preparation.ipynb](01_Data%20Preparation.ipynb), [05_Robustness_Checks.ipynb](05_Robustness_Checks.ipynb) and [00_Time_Series_Analysis.ipynb](00_Time_Series_Analysis.ipynb) as `../all_data_files/cleaned_dataset_per_device.csv`.
- The notebooks are intended to be run in order.
- Later notebooks depend on saved outputs from earlier stages rather than shared notebook state.
