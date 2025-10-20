# AI Demos (VS Code + Windows)
Three self-contained projects:
- RAGLLM_notebook — Automated SITREP Generator from an incident PDF (RAG + Streamlit)
- SHAP_notebook — Predictive Maintenance with XGBoost + SHAP (diagnostic app)
- YOLO_notebook — All-Weather Maritime Asset Identification with YOLOv8 (drift quantification + Streamlit)

Each project:
- Runs fully in a notebook with offline fallbacks.
- Exports artifacts and a Streamlit app under its subfolder.

## Prerequisites
- Windows 10/11, VS Code
- Python 3.11 recommended for RAGLLM and YOLO; Python 3.10+ works for SHAP
- Internet for first-run model/data downloads; all notebooks have offline fallbacks

## Quickstart
Open this folder in VS Code, then choose a project:

### 1) RAGLLM_notebook (SITREP)
- Open RAGLLM_notebook/Automated_SITREP_RAGLLM.ipynb
- Run cells top → bottom:
  - The notebook can create a Python 3.11 venv and install packages.
  - It ingests a public MOT report (or creates a tiny synthetic PDF if offline).
  - It builds embeddings + FAISS and writes a Streamlit app to app/app.py.
- Launch app (from the notebook cell or terminal in RAGLLM_notebook):
  - Terminal (PowerShell):
    powershell> python -m streamlit run app/app.py --server.address localhost --server.port 8501
  - Open http://localhost:8501

### 2) SHAP_notebook (Predictive Maintenance)
- Open SHAP_notebook/PdM_XGB_SHAP_Notebook.ipynb
- Run cells top → bottom:
  - Trains XGBoost, computes metrics/calibration, generates SHAP plots.
  - Writes a Streamlit diagnostic app to app/app.py.
- Launch app (from the notebook cell or terminal in SHAP_notebook):
  - Terminal (PowerShell):
    powershell> python -m streamlit run app/app.py --server.address localhost --server.port 8501
  - Open http://localhost:8501

### 3) YOLO_notebook (All-Weather Maritime)
- Open YOLO_notebook/All_Weather_Maritime_YOLOv8.ipynb
- Run cells top → bottom:
  - Create a Python 3.11 venv and install packages (registers kernel “Python 3.11 (sitrep)”).
  - Data pipeline: attempts Kaggle download of the Singapore Maritime dataset; if offline/fails, generates synthetic clear/adverse images + labels. Extracts frames from .avi, pseudo‑labels with COCO YOLO if needed, writes data_clear.yaml/data_adverse.yaml.
  - Optional training: Blue Lane (clear-only, default 5 epochs) and Red Lane (mixed‑weather + stronger aug + larger imgsz). Evaluates clear vs adverse and writes artifacts (see below).
- Launch app (from the notebook cell or terminal in YOLO_notebook):
  - Terminal (PowerShell):
    powershell> python -m streamlit run app/app.py --server.address localhost --server.port 8501
  - Open http://localhost:8501

## Folder layout
- RAGLLM_notebook/
  - Automated_SITREP_RAGLLM.ipynb
  - app/app.py (generated)
  - data/, models/, artifacts/ (generated)
- SHAP_notebook/
  - PdM_XGB_SHAP_Notebook.ipynb
  - app/app.py (generated)
  - data/, models/, artifacts/ (generated)
- YOLO_notebook/
  - All_Weather_Maritime_YOLOv8.ipynb
  - app/app.py (generated)
  - data/, models/, artifacts/, runs/ (generated)
  - data_clear.yaml, data_adverse.yaml (generated)
  - data_mixed.yaml, data/maritime/lists/mixed_train.txt (generated if Red Lane runs)

## Tips
- First run downloads models; subsequent runs are faster due to caching.
- SITREP app: if FAISS/chunks mismatch warnings appear, re-run the retrieval cell to regenerate data/chunks.json and models/faiss.index together.
- YOLO data/training:
  - Force fresh Kaggle download: powershell> $env:FORCE_KAGGLE="1"
  - Force re‑extract raw videos: powershell> $env:FORCE_REEXTRACT="1"
  - Skip YOLO training: powershell> $env:SKIP_HEAVY_TRAINING="True"
  - Red Lane (optional):
    - powershell> $env:RUN_RED_LANE="1"
    - powershell> $env:MIX_ADVERSE_RATIO="0.3"
    - powershell> $env:RED_EPOCHS="30"
    - powershell> $env:RED_IMGSZ="960"
- All apps write run summaries to artifacts/.
- Change Streamlit host/port:
  - powershell> $env:PORT="8501"; $env:BIND_ADDR="localhost"
  - powershell> python -m streamlit run app/app.py --server.address $env:BIND_ADDR --server.port $env:PORT

## Key YOLO artifacts
- Blue Lane: artifacts/perf_clear.json, artifacts/perf_adverse.json, artifacts/degradation.csv, artifacts/vnv_report.md, models/yolov8n_clear.pt
- Red Lane: artifacts/perf_mixed_clear.json, artifacts/perf_mixed_adverse.json, artifacts/degradation_mixed.csv, artifacts/redlane_comparison.csv, artifacts/vnv_report_mixed.md, models/yolov8n_mixed.pt
- Model card and spec: artifacts/model_card.md, artifacts/spec_drive.md

## License and data use
Educational/experimental use with public/open sample data. Review any external data licenses before use in production.