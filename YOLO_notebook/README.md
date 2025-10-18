# All-Weather Maritime Asset Identification (YOLOv8)

End-to-end notebook that:
- Curates a maritime dataset into clear_weather vs adverse_weather (with synthetic fallback when needed).
- Fine-tunes YOLOv8n on clear-only and evaluates on clear vs adverse to quantify drift.
- Builds a Streamlit app for side‑by‑side detections and confidence behavior.
- Exports governance artifacts (metrics, plots, degradation report, model card) under `artifacts/`.

## Quickstart (Windows + VS Code)

1) Open `All_Weather_Maritime_YOLOv8.ipynb` in VS Code.
2) Run “Create venv with Python 3.11” (registers kernel “Python 3.11 (sitrep)”).
   - If Python 3.11 is missing:
     ```powershell
     winget install -e --id Python.Python.3.11
     ```
3) Switch the notebook kernel to “Python 3.11 (sitrep)”.
4) Run “Install Packages (graceful)”.
5) Run the remaining cells to prepare data, optionally train, evaluate, save artifacts, and write the Streamlit app.

Notes
- Kaggle download uses `opendatasets`. If offline or it fails, a synthetic dataset is auto‑generated.
- To keep runtime short: set `SKIP_HEAVY_TRAINING=True` in the first cell.

## Data pipeline

- Attempts to download Singapore Maritime dataset:
  - URL: https://www.kaggle.com/datasets/mmichelli/singapore-maritime-dataset
  - Saved under: `data/maritime/raw/singapore-maritime-dataset`
- Extracts frames from `.avi` videos into:
  - `data/maritime/clear_weather/images/{train,val}`
  - `data/maritime/adverse_weather/images/test`
- If frames/labels are missing:
  - Pseudo‑labels are created using a COCO‑pretrained YOLO (remapping “boat/ship” → class 0).
  - Synthetic images + YOLO labels are generated as a last resort.
- Auto‑writes YOLO data configs:
  - `data_clear.yaml` (clear train/val)
  - `data_adverse.yaml` (clear train/val + adverse test)

Env flags (PowerShell)
- Force Kaggle re‑download:
  ```powershell
  $env:FORCE_KAGGLE="1"
  ```
- Force re‑extract frames from raw videos:
  ```powershell
  $env:FORCE_REEXTRACT="1"
  ```
- Skip YOLO training:
  ```powershell
  $env:SKIP_HEAVY_TRAINING="True"
  ```

## Training and evaluation

- Model: YOLOv8n. Default 5 epochs on clear‑only (GPU if available, else CPU).
- Evaluation:
  - Clear validation metrics → `artifacts/perf_clear.json`
  - Adverse test metrics → `artifacts/perf_adverse.json`
  - Drift table/report → `artifacts/degradation.csv`, `artifacts/vnv_report.md`
- Model card is initialized and then appended with run details → `artifacts/model_card.md`
- Sample annotated predictions saved under `artifacts/plots/{clear,adverse}`

## Streamlit app

The notebook writes `app/app.py`. Launch from the notebook “Launch app” cell, or via terminal:

- PowerShell:
  ```powershell
  cd "<your project root>"
  python -m streamlit run app/app.py --server.address localhost --server.port 8501
  ```
- Optional env/ports:
  ```powershell
  $env:PORT="8501"; $env:BIND_ADDR="localhost"
  python -m streamlit run app/app.py --server.address $env:BIND_ADDR --server.port $env:PORT
  ```

App features
- Side‑by‑side inference on clear vs adverse samples.
- Sidebar controls: confidence threshold and inference size (640/800/960/1280).
- Image uploader with the same controls.
- Illustrative precision/recall proxy plot (for real V&V, use ground‑truth + IoU).
- Run summary written to `artifacts/run_summary.json`.

## Outputs (key paths)

- `data/maritime/...` — prepared dataset (or synthetic fallback)
- `models/yolov8n_clear.pt` — trained weights (if training ran)
- `artifacts/` — metrics, plots, degradation report, model card
- `artifacts/plots/{clear,adverse}/*.jpg` — sample annotated predictions
- `app/app.py` — Streamlit app
- `data_clear.yaml`, `data_adverse.yaml` — YOLO data configs
- `runs/` — Ultralytics training outputs

## Troubleshooting

- Python 3.11 not found: install with `winget`, then re‑run the venv cell and switch the kernel.
- “No .avi video files found”: pipeline will fall back to synthetic data; to re‑scan after placing videos, set `$env:FORCE_REEXTRACT="1"` and re‑run.
- Kaggle auth/permissions: ensure Kaggle is configured or skip it (offline mode/synthetic fallback).
- Torch/CUDA issues: pipeline auto‑falls back to CPU; training simply runs slower.
- “%%writefile not found”: the notebook already writes `app/app.py` via a portable file writer; just run that cell.

## License and data use

Educational/experimental use with public/open sample data. Verify external dataset licenses before operational use.