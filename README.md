# Pneumonia Detection Using Chest X-Ray with DenseNet121

An educational and research web application for binary chest X-ray classification. A React interface sends JPG, JPEG, or PNG images to a FastAPI backend, which preprocesses an image, runs a DenseNet121 checkpoint, and returns a class prediction, probabilities, uncertainty status, and Grad-CAM visualization when the checkpoint is installed.

> This project is intended for educational and research purposes and is not a substitute for professional medical diagnosis. It is not a clinical diagnostic tool. Model outputs and Grad-CAM visualizations are not medical advice or proof of pathology.

## Overview

The project demonstrates a binary image-classification workflow and a local web application around it. The backend provides upload validation, temperature scaling, threshold-based classification, confidence-based review status, Grad-CAM overlays, SQLite analysis history, and generated PDF reports. The frontend provides dashboard, analysis, result, history, and model reliability views.

### Key features

- DenseNet121 with a two-class linear classifier.
- 224×224 image preprocessing and ImageNet normalization.
- Temperature-scaled probabilities and a configured pneumonia threshold.
- Confidence-based `ACCEPTED` or `UNCERTAIN / REVIEW` status.
- DenseBlock4 Grad-CAM overlay generation.
- Local SQLite history and PDF report generation.
- Standby behavior if the model checkpoint is not installed.

## Technology and architecture

- **Frontend:** React 18, Vite, Tailwind CSS, Axios, lucide-react.
- **Backend:** FastAPI, Pydantic Settings, SQLAlchemy, SQLite, PyTorch, torchvision, Pillow, Matplotlib, ReportLab.
- **Experiments:** Jupyter notebooks and scikit-learn metrics.

```text
React UI -> Axios -> FastAPI -> image validation -> preprocessing
  -> DenseNet121 -> temperature scaling -> threshold and uncertainty
  -> Grad-CAM -> SQLite -> JSON response -> result/history UI
```

## Dataset and methodology

The training notebook records the Kaggle `reflex7/cxr-data-set` archive (reported as 16.9 GB), selecting Normal and Pneumonia classes. It reports 36,284 images before exact MD5 duplicate removal and 30,283 retained images afterward. Its image-level split is stratified 70/15/15: 21,198 training, 4,542 validation, and 4,543 test images. It reports no corrupt images in its scan and no exact cross-class duplicate groups. Patient-independent splitting is not documented; exact-hash checking does not rule out near duplicates or patient overlap.

The notebooks are retained as research records. The package copies omit embedded chest X-ray and Grad-CAM image outputs to avoid redistributing medical images; notebook code, text outputs, metrics, and non-medical plots are retained. The training and external notebooks contain Colab/Google Drive paths that require adapting to the user's own environment before rerunning.

Training notebook preprocessing converts images to RGB, resizes to 224×224, converts to tensors, and applies ImageNet normalization. Training augmentation includes horizontal flips, mild rotation, and affine translation/scale. The notebook describes ImageNet-pretrained DenseNet121, a frozen-feature classifier stage, then fine-tuning DenseBlock4 and `norm5`, weighted cross-entropy, AdamW, and early stopping. The notebook includes multiple resumed experiments, and `backend/artifacts/preprocessing/training_configuration.csv` records a different summary. See [methodology](docs/methodology.md) for the limits of reconstructing a single final recipe from these records.

Backend inference in `backend/app/model/preprocessing.py` converts inputs to three-channel grayscale before ImageNet normalization. The training notebook's datasets are converted to RGB without a grayscale step. This is a documented preprocessing difference; this package does not alter it or assert that it is compatible with the checkpoint. Investigate before interpreting deployed predictions as reproducing notebook metrics.

## Evaluation results

The training notebook's final internal evaluation reports the following on 5,439 images:

| Metric | Reported value |
| --- | ---: |
| Accuracy | 0.9638 |
| Precision | 0.9478 |
| Sensitivity | 0.9816 |
| Specificity | 0.9460 |
| F1 | 0.9644 |
| ROC-AUC | 0.9960 |
| Confusion matrix | `[[2573, 147], [50, 2669]]` |

The notebook states this test set was not used for training or threshold selection. It does not document a patient-independent split. These are reported notebook results; the checkpoint needed to reproduce them is not distributed in this package.

The final frozen-threshold (0.16) Kermany result in `notebooks/Untitled3_(1).ipynb` is reported on 624 images: accuracy 0.9535, precision 0.9308, sensitivity 1.0000, specificity 0.8761, F1 0.9642, and ROC-AUC 0.9998. Its confusion matrix is `[[205, 29], [0, 390]]`. That notebook also reports a distinct 0.50-threshold operating point. The other external-testing notebook contains a 622-image construction and multiple checkpoint attempts; do not merge those experiments. Neither result is clinical validation.

## Model, calibration, and Grad-CAM

The backend defines DenseNet121 with block configuration `(6, 12, 24, 16)` and a linear 1024-to-2 classifier. Inference applies temperature `1.0726476907730103`, loaded from `backend/artifacts/calibration/temperature_scaling.json`, and labels pneumonia when its probability is at least `0.16`. Confidence is the predicted class probability; uncertainty is `1 - confidence`; uncertainty greater than `0.10` yields `UNCERTAIN / REVIEW`. Grad-CAM targets `features.denseblock4` and creates an overlay. These are implementation settings, not clinical guarantees or causal explanations.

## Backend and API

The application entry point is `backend/app/main.py`. Routes are served both at the root and under `/api`.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/health` | Application and model readiness |
| GET | `/model-info` | Model settings and recorded metrics |
| POST | `/predict` | Multipart upload using form field `file` |
| GET | `/history?page=1&page_size=20` | Paginated SQLite history |
| GET | `/history/{analysis_id}` | Analysis details |
| GET | `/report/{analysis_id}` | Generate/download a PDF report |
| GET | `/static/uploads/{filename}` | Uploaded image asset |
| GET | `/static/gradcam/{filename}` | Grad-CAM asset |

The backend accepts `.jpg`, `.jpeg`, and `.png` uploads up to 10 MB. It stores uploads, Grad-CAM images, reports, and SQLite data locally. These may contain sensitive medical information; do not publish runtime data.

## Frontend

The Vite/React single-page interface contains Dashboard, Analyze, Results, History, and Model Reliability views. API requests are implemented in `frontend/src/api.js`. The API base is `VITE_API_BASE_URL`, defaulting to `http://localhost:8000`; configure it in a local `frontend/.env` based on `frontend/.env.example`. There is no production deployment configuration in this repository.

## Project layout

```text
backend/app/       API, model, database, report generation
backend/artifacts/ Calibration and deployment metadata
backend/tests/     Pytest tests
frontend/src/      React application
notebooks/         Training and external-test experiment records
docs/              Architecture, methodology, model, evaluation, API, setup
```

## Model checkpoint setup

The checkpoint is intentionally excluded from this GitHub-ready package. It was present in the original project at 28,450,679 bytes (about 27.1 MiB), but this environment could not load it with PyTorch to verify its state-dict keys, classifier dimensions, or provenance. Notebook records reference several checkpoints, so the file's identity and redistribution rights are not established here. No download URL is available.

To enable inference, obtain the intended compatible checkpoint through an authorized source and copy it without renaming to:

```text
backend/artifacts/model/best_robustness_finetuned_densenet121.pth
```

The application expects a state dict compatible with the architecture in `backend/app/model/densenet.py`; loading uses strict key matching. If missing, the backend starts in standby and `/predict` returns HTTP 503. See `backend/artifacts/model/README.md`.

## Installation and running

Requirements: Python with packages supported by `backend/requirements.txt`, and Node.js/npm compatible with Vite 5. Dependencies are not pinned to exact versions.

### Backend

```powershell
cd backend
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```

API documentation: <http://127.0.0.1:8000/docs>.

### Frontend

```powershell
cd frontend
npm install
Copy-Item .env.example .env
npm run dev
```

The development UI is served at <http://localhost:5173>. Use `npm run build` to create the production bundle; generated `dist/` is ignored.

### Example API request

```bash
curl -X POST http://localhost:8000/predict -F "file=@chest-xray.png"
```

## Tests

Backend tests cover endpoint metadata, missing-checkpoint upload behavior, preprocessing shape, calibration/threshold behavior, history, and PDF creation. They do not test a real checkpoint inference. Run from `backend/`:

```bash
python -m pytest tests/ -v
```

There is no frontend test suite or test command in `frontend/package.json`.

## Limitations and future improvements

- Confirm checkpoint provenance and compatibility in an environment with PyTorch.
- Reconcile training versus inference preprocessing and document the exact final training configuration.
- Establish patient-level separation and evaluate duplicate/patient overlap where data permits.
- Reproduce metrics with the exact final checkpoint and frozen protocol.
- Define retention and access control for locally stored medical images and reports.
- Add a project license only after selecting appropriate terms.

**License: To be determined.** Dataset licenses are separate from this project's code license. Do not redistribute datasets or patient images without permission.
