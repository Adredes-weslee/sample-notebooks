# Predictive Maintenance — XGBoost + SHAP (Notebook + Streamlit)

End-to-end notebook that:
- Trains an XGBoost classifier on an Industrial IoT fault dataset (with offline fallbacks).
- Performs V&V (PR-AUC, ROC-AUC, confusion matrix, calibration/ECE, threshold tuning).
- Generates global and local SHAP explanations.
- Builds a Streamlit diagnostic app with sliders and a live SHAP waterfall.
- Exports artifacts (metrics, plots, model card, app).

## Quickstart (Windows + VS Code)

1) **Open this folder in VS Code and run the notebook:**
   - File: PdM_XGB_SHAP_Notebook.ipynb
   - The notebook includes a cell to create a Python 3.11 venv and register a kernel.
     - If needed, install Python 3.11:
       ```powershell
       winget install -e --id Python.Python.3.11
       ```
   - Run the “Install Packages” cell to install dependencies into the kernel.

2) **Provide data (any one works)**
   - Preferred: place a CSV at `data/iot_faults.csv`. If present, Kaggle is skipped and this file is loaded.
   - Reuse: any previously downloaded CSV under `data/kaggle/**/*.csv` will be auto‑picked up.
   - Kaggle fallback (online only): attempts download via `opendatasets` from  
     https://www.kaggle.com/datasets/ziya07/industrial-iot-fault-detection-dataset (saved under `data/kaggle/`).
   - Mirror fallback (online only): pulls a public sample CSV (`diabetes.csv`) from Plotly as a placeholder.
   - Last resort: generates a small synthetic dataset (Temperature, Vibration, Pressure, Fault) in‑notebook.

    **Post‑load normalization:**
    - Columns are normalized (strip, spaces→underscore, Title‑case).
    - If using the mirror CSV, `Outcome` is renamed to `Fault`.
    - If `Fault` is missing, it is derived from the top quartile of the first numeric feature.
    - A reproducible copy is saved to `data/working.csv`.
    - If `OFFLINE` is detected, Kaggle and mirror steps are skipped automatically.

      Note on column names: the notebook normalizes headers to TitleCase with underscores (e.g., “Vibration (mm/s)” → “Vibration_(Mm/S)”, “Fault Label” → “Fault_Label”). Labels are unified to `Fault`.

3) **Run the notebook**
   - File: PdM_XGB_SHAP_Notebook.ipynb
   - Execute cells top to bottom. This produces:
     - models/xgb.pkl, models/scaler.pkl
     - models/features_meta.json, models/shap_bg.npy
     - artifacts/metrics.json, artifacts/ece.json, artifacts/threshold.json
     - artifacts/plots/*.png
     - artifacts/vnv_report.md, artifacts/model_card.md

4) **Launch the Streamlit app**
      - The “Streamlit App (file writer)” cell writes app/app.py in this folder.
      ```powershell
      python -m streamlit run app/app.py --server.address localhost --server.port 8501
      ```
      - Then open http://localhost:8501.

## App usage
- Adjust key sensor sliders on the left and choose a decision threshold.
- Click “Predict Status” to compute probability and show a SHAP waterfall for that setting.
- “Download V&V Report” saves artifacts/vnv_report.md.

## Data handling
- Local file: data/iot_faults.csv takes precedence (Kaggle download is skipped).
- Kaggle reuse: if a CSV exists in data/kaggle/, it is reused without re-downloading.
- Offline: auto-detected; Kaggle is skipped when offline.

## Troubleshooting
- Flat/near-zero SHAP or cluttered x-axis ticks:
  - Sliders start at feature means; if the background is also near the mean, f(x) ≈ E[f(X)] and SHAP ≈ 0. Move sliders away from means.
  - Ensure models/shap_bg.npy exists (generated in preprocessing). The app prefers this background.
  - The app formats the SHAP x-axis to compact values; if still cluttered, zoom or reduce displayed features.
- “Fault_Label” shows up as a feature:
  - The preprocessing removes label-like columns. If you trained before this fix, delete models/features_meta.json and retrain (run preprocessing + training cells).
- Kaggle auth/permission issues:
  - Provide a local CSV at data/iot_faults.csv or run fully offline; the code falls back to a mirrored CSV and, if needed, a tiny synthetic dataset.

## Outputs
- Model: models/xgb.pkl
- Scaler + meta: models/scaler.pkl, models/features_meta.json
- SHAP background: models/shap_bg.npy
- Metrics & plots: artifacts/metrics.json, artifacts/plots/*
- Calibration & threshold: artifacts/ece.json, artifacts/threshold.json
- Reports: artifacts/vnv_report.md, artifacts/model_card.md

## Notes
- Random seeds fixed (SEED=42) for reproducibility.
- XGBoost uses scale_pos_weight for class imbalance.
- SHAP uses TreeExplainer with interventional perturbation and probability output.

Educational/experimental use with public/open sample data.