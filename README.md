# AI Demos (VS Code + Windows)
Two self-contained projects:
- RAGLLM_notebook — Automated SITREP Generator from an incident PDF (RAG + Streamlit)
- SHAP_notebook — Predictive Maintenance with XGBoost + SHAP (diagnostic app)

Each project:
- Runs fully in a notebook with offline fallbacks.
- Exports artifacts and a Streamlit app under its subfolder.

## Prerequisites
- Windows 10/11, VS Code
- Python 3.11 recommended for RAGLLM; Python 3.10+ works for SHAP
- Internet for first-run model/data downloads; both notebooks have offline fallbacks

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

## Folder layout
- RAGLLM_notebook/
  - Automated_SITREP_RAGLLM.ipynb
  - app/app.py (generated)
  - data/, models/, artifacts/ (generated)
- SHAP_notebook/
  - PdM_XGB_SHAP_Notebook.ipynb
  - app/app.py (generated)
  - data/, models/, artifacts/ (generated)

## Tips
- First run downloads models; subsequent runs are fast due to caching.
- If FAISS/chunks mismatch warnings appear in the SITREP app, re-run the retrieval cell to regenerate data/chunks.json and models/faiss.index together.
- Both apps write run summaries to artifacts/.

## License and data use
Educational/experimental use with public/open sample data. Review any external data licenses before use in production.