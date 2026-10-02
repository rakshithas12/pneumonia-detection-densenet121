# API reference

The FastAPI router is exposed at the root and `/api` prefixes. Local default origin settings are defined in `backend/app/config.py`.

| Method | Route | Details |
| --- | --- | --- |
| GET | `/health` | Readiness and artifact status |
| GET | `/model-info` | Architecture, preprocessing, thresholds, and notebook-recorded metrics |
| POST | `/predict` | Multipart form field `file`; JPG/JPEG/PNG, maximum 10 MB |
| GET | `/history?page=1&page_size=20` | Paginated records |
| GET | `/history/{analysis_id}` | One record |
| GET | `/report/{analysis_id}` | Generate and stream a PDF |
| GET | `/static/uploads/{filename}` | Locally stored uploaded image |
| GET | `/static/gradcam/{filename}` | Locally stored Grad-CAM image |

Prediction response fields are defined in `backend/app/api/schemas.py`. If the model is unavailable, `/predict` returns HTTP 503. Example:

```bash
curl -X POST http://localhost:8000/predict -F "file=@chest-xray.png"
```
