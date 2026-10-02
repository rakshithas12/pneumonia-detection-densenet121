# Architecture

## Runtime flow

The React app calls the FastAPI service through the Axios client in `frontend/src/api.js`. The backend validates the uploaded extension, size, and image readability, preprocesses it, runs the loaded DenseNet121, applies temperature scaling and threshold/uncertainty rules, generates a Grad-CAM overlay, saves an analysis record in SQLite, and returns JSON URLs and values. The frontend renders the returned result and can request history details and PDF reports.

## Components

- `frontend/src/App.jsx`: state-based view selection for dashboard, analyze, results, history, and model reliability.
- `backend/app/main.py`: FastAPI setup, startup lifecycle, CORS, static mounts, root and `/api` route mounts.
- `backend/app/api/routes.py`: health, model info, prediction, history, and PDF report endpoints.
- `backend/app/model/`: model architecture/loading, preprocessing, inference, calibration, uncertainty, and Grad-CAM.
- `backend/app/database/`: SQLAlchemy record model and SQLite session setup.
- `backend/app/services/report_service.py`: PDF report generation.

## Local persistence

Uploaded images, Grad-CAM outputs, reports, and the SQLite database are stored below `backend/storage/`. This directory is runtime state and excluded from the GitHub-ready package.
