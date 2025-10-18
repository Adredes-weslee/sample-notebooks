# Automated SITREP Generator (RAG + Streamlit)

End-to-end notebook that:
- Extracts text from a public MOT Marine accident investigation PDF (with offline synthetic fallback).
- Splits text into chunks and indexes with FAISS + sentence-transformers.
- Generates a guarded SITREP using an open LLM (fallback to extractive if transformers unavailable).
- Builds a Streamlit app with citations and startup timing diagnostics.
- Saves governance artifacts (SPEC-DRIVE, groundedness, V&V report, model card).

## Quickstart (Windows + VS Code)

1) **Open this folder in VS Code and run the notebook:**
   - File: Automated_SITREP_RAGLLM.ipynb
   - The notebook includes a cell to create a Python 3.11 venv and register a kernel:
     - If needed, install Python 3.11:
       powershell> winget install -e --id Python.Python.3.11
   - Run the “Install Packages” cell to install dependencies into the kernel.

2) **Provide data (any one works)**
      - Preferred: place your PDF at `data/report.pdf`. If present (and non-empty), the pipeline uses it and skips any download.
      - Online download (default): fetches the public MOT report to `data/report.pdf`:
        https://www.mot.gov.sg/docs/default-source/about-mot/missing-of-fitter-from-rtm-zheng-he-at-sea-on-26-december-2024.pdf?sfvrsn=b4c661aa_1
      - Offline/failed-download fallback: generates a tiny synthetic PDF with ReportLab and saves it to `data/report.pdf`.
      - If none of the above succeed, the notebook raises an error prompting you to upload `data/report.pdf`.

      **Post‑ingestion:**
      - Extracts page‑anchored text with PyMuPDF (fitz) and writes `data/report_chunks.json`.

  
3) **Execute the pipeline cells in order:**
   - Data ingestion and PDF extraction (saves data/report.pdf and data/report_chunks.json).
   - Retrieval pipeline:
     - Writes data/chunks.json (the exact chunk list used for FAISS)
     - Builds models/faiss.index and data/chunk_map.json
   - Generation and V&V (optional probes + groundedness).

4) **Generate and run the Streamlit app:**
   - The “Streamlit App (file writer)” cell writes app/app.py in this folder.
   - Launch from the notebook cell or terminal:
     powershell> python -m streamlit run app/app.py --server.address localhost --server.port 8501
   - Open http://localhost:8501

## App usage
- Left pane shows PDF pages (from data/report_chunks.json).
- Right pane:
  - Sample prompts dropdown.
  - “Or write your own prompt” overrides the sample if non-empty after stripping. The app shows “Using prompt: '...’” so you know which ran.
  - TOP_K selects how many chunks to retrieve.
  - Optional PII masking (spaCy) redacts PERSON/ORG in the output.
- Output includes a SITREP and a Citations section listing retrieved chunks with page numbers and previews.

## Guardrails and behavior
- Instruction enforces “context-only” generation; if info is missing, the model should respond: NOT FOUND IN SOURCE.
- If transformers or models cannot load, the app falls back to a fast extractive summary of retrieved chunks.
- The loader displays per-stage timings:
  - IO, Embeddings model load, FAISS index load, and LLM load, plus total time.

## Outputs
- data/
  - report.pdf, report_chunks.json, chunks.json, chunk_map.json
- models/
  - faiss.index
- artifacts/
  - groundedness.json, vnv_report.md, model_card.md, spec_drive.md
  - run_summary.json (written by the app)

## Troubleshooting
- IndexError: list index out of range (during generation)
  - Cause: FAISS ntotal does not match len(chunks). Fix by re-running the retrieval cell to regenerate data/chunks.json and models/faiss.index together, then reload the app.
- “FAISS index not available”
  - Ensure the retrieval cell ran successfully and models/faiss.index exists.
- “Transformers not available” or slow LLM download
  - The app will use extractive fallback. For faster starts, keep the HF cache (hf_cache/) and avoid cleaning it.
- “data/chunks.json missing; using page texts”
  - The app can run but retrieval may be misaligned. Re-run the retrieval cell to write data/chunks.json.

## Performance notes (typical)
- Warm start (all cached): ~2–7 s to load.
- Cold start with downloads: ~1–5 min on 50–100 Mbps; slower on poor links.

## License and data use
Educational/experimental use with public/open sample data. Verify external report licenses before operational use.