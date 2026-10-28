# Glucose Forecasting with an MLP on CGM Data

Time-series forecasting of blood glucose levels from continuous glucose monitoring (CGM) data of 12 patients with type 1 diabetes. The project is part of the Examination Project 2025 (Signal Processing Task).

## Overview

- **Task:** single-step forecasting of future glucose values (mg/dL) from a window of past measurements
- **Data:** about 8 weeks of raw CGM data per patient, sampled every 5 minutes
- **Model:** Multi-Layer Perceptron (MLP)
- **Validation:** Leave-One-Patient-Out cross-validation (12 folds)
- **Metrics:** RMSE, MAE, MAPE (mean ± standard deviation across folds)

## Data Preprocessing

1. **Cleaning:** values outside the physiological range 40–400 mg/dL are set to NaN.
2. **Gap filling:** gaps of up to 12 consecutive samples (60 min) are filled by linear interpolation.
3. **Window filtering:** windows still containing NaN after interpolation are discarded.
4. **Sliding window:** the series is converted to a supervised problem (step = 1).
   - Input: the last `WIN_SIZE` samples
   - Target: the value `PREDICTION_HORIZON` samples after the end of the window
5. **Normalization:** Min-Max scaling, with min/max computed on the training set only and applied to the test patient, to avoid data leakage. Predictions are denormalized before computing the metrics.

## Model

| Component | Details |
|---|---|
| Architecture | Dense 128 → 64 → 32 → 16 → 1 |
| Activation | ReLU |
| Regularization | BatchNorm (first two layers), Dropout 0.3 / 0.2 / 0.1 |
| Optimizer | Adam |
| Loss | MSE |
| Early stopping | patience = 10 (restores best weights) |
| LR scheduler | ReduceLROnPlateau (factor 0.5, patience = 5) |

### Training parameters

| Parameter | Value |
|---|---|
| Window size | 12 samples (60 min), tuned on the validation set |
| Reference prediction horizon | 6 samples (30 min) |
| Epochs | 100 (max) |
| Batch size | 32 |
| Learning rate | 0.001 |

## Validation Strategy

- **Outer loop:** Leave-One-Patient-Out (12 folds). Each patient is used once as the test set, while the other 11 are used for training, to evaluate generalization across patients.
- **Inner validation:** chronological split of the training data (last 15% used for validation), which preserves temporal order.
- **Hyperparameter tuned:** window size, selected on the validation set.

## Results

Performance at a prediction horizon of 30 minutes (mean ± std across the 12 folds):

| Metric | Value |
|---|---|
| MAE | 16.78 ± 2.40 mg/dL |
| RMSE | 23.76 ± 3.63 mg/dL |
| MAPE | 12.20 ± 2.35 % |

**Naive baseline** (persistence): RMSE = 26.65 mg/dL. The MLP improves on the baseline by about 10.8% in RMSE.

## Repository Structure

```
.
├── notebook.ipynb          # Full pipeline (Google Colab)
├── glucose_level.npy       # Training data (12 patients)
├── glucose_level_test.npy  # Test data
└── README.md
```



