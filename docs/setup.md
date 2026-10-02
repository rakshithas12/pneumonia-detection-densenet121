# Setup

Use Python with the dependencies in `backend/requirements.txt` and Node.js/npm compatible with Vite 5. Exact dependency versions and a Python version are not pinned.

## Backend (PowerShell)

```powershell
cd backend
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```

Install the compatible checkpoint at `backend/artifacts/model/best_robustness_finetuned_densenet121.pth` through an authorized source. The package intentionally omits it. Without it, health reports standby and prediction is unavailable.

## Frontend (PowerShell)

```powershell
cd frontend
npm install
Copy-Item .env.example .env
npm run dev
```

The frontend defaults to `http://localhost:8000`; override `VITE_API_BASE_URL` in the local `.env` if needed. Run `npm run build` to produce `dist/`.

## Tests

From `backend/`, run `python -m pytest tests/ -v`. No frontend test command is defined.
