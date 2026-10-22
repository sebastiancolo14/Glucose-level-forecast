Modello: MLP (Multi-Layer Perceptron)
Architettura: 128 → 64 → 32 → 16 → 1 (ReLU, BatchNorm, Dropout 0.3/0.2/0.1)

Parametri Training:
  - Window Size: 12 campioni (60 minuti) [OTTIMIZZATO su validation]
  - Prediction Horizon di riferimento: 6 campioni (30 minuti)
  - Epochs: 100
  - Batch Size: 32
  - Learning Rate: 0.001
  - Early Stopping: patience=10, ReduceLROnPlateau: patience=5

Pulizia dati: valori fuori da 40-400 mg/dL -> NaN; interpolazione lineare buchi <= 12 campioni;
              finestre con NaN residui scartate
Validation Strategy: Leave-One-Patient-Out (12 fold), validation cronologica interna (ultimo 15%)

PERFORMANCE FINALI @ PH=30 min (Mean ± Std):
  - MAE:  16.78 ± 2.40 mg/dL
  - RMSE: 23.76 ± 3.63 mg/dL
  - MAPE: 12.20 ± 2.35 %
  - Baseline naive RMSE: 26.65 mg/dL
