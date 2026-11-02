# Team handoff

## Current state

The repository contains a runnable local full-stack application:

- React/Vite frontend at `http://127.0.0.1:5173`
- FastAPI backend at `http://127.0.0.1:8000`
- Local FAISS index generated from the NIDDK diabetes guideline
- Sentence Transformer embedding model loaded at startup
- NLI cross-encoder loaded at startup
- Gemini client enabled through a local environment variable

The frontend polls `/health` and uses the backend for all live answers, evaluation, and knowledge-base views.

## Start from a clean checkout

```powershell
python -m pip install -r backend\requirements.txt
python scripts\build_index.py
Set-Location frontend
npm install
```

Create `backend/.env` from `backend/.env.example` and add a personal Gemini key. Then use two terminals:

Terminal 1:

```powershell
Set-Location backend
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Terminal 2:

```powershell
Set-Location frontend
npm run dev -- --host 127.0.0.1
```

Wait for model loading to complete, then verify:

```powershell
Invoke-RestMethod http://127.0.0.1:8000/health
```

Expected:

```json
{
  "status": "ok",
  "ready": true,
  "models_loaded": {
    "embedding": true,
    "faiss_index": true,
    "nli": true,
    "gemini": true
  }
}
```

## What to verify manually

1. Open the frontend and confirm the system status becomes ready.
2. Ask: `What are the risk factors for type 2 diabetes?`
3. Confirm the result includes an answer, source chunks, claim rows, confidence, decision, correction state, and stage timings.
4. Ask an unrelated question such as `How do I repair a car engine?`.
5. Confirm the response exposes weak retrieval or insufficient evidence rather than presenting the answer as authoritative.
6. Open the evaluation lab and knowledge-base explorer.

## Test commands

```powershell
Set-Location frontend
npm run build
npm run lint

Set-Location ..
python -m unittest discover -s tests -p "test_*.py" -v
```

Tests that call Gemini need network access and may consume API quota. If the key is absent, configuration and failure-path tests can still be run, but live generation tests will not pass.

## Important files

| File | Responsibility |
|---|---|
| `backend/app/main.py` | FastAPI lifespan, routes, pipeline orchestration |
| `backend/app/config.py` | Environment-backed settings and paths |
| `backend/app/schemas.py` | API contracts |
| `backend/app/services/retrieval.py` | FAISS loading and search |
| `backend/app/services/llm.py` | Gemini client and generation |
| `backend/app/services/verification.py` | Claims, confidence, and decisions |
| `backend/app/services/correction.py` | Self-correction |
| `scripts/build_index.py` | PDF ingestion and vector artifact generation |
| `frontend/src/services/api.ts` | Frontend/backend integration |
| `frontend/src/App.tsx` | UI state and health polling |
| `tests/` | Automated behavior coverage |

## Handoff rules

- Do not commit `.env` files or generated secrets.
- Do not replace the approved source document without recording provenance.
- Rebuild `vector_db` after ingestion changes.
- Keep API schemas and TypeScript types aligned.
- Update the README and relevant docs for endpoint, configuration, or workflow changes.
- Report changed files, validation commands, results, warnings, and remaining work.

## Known issues and follow-up

- First startup downloads large Hugging Face models and can be slow.
- Gemini availability depends on key validity, quota, network, and model availability.
- The current source coverage is limited to diabetes guidance.
- Generated vector artifacts may be absent on a fresh checkout and must be rebuilt.
- Production deployment still needs authentication, rate limiting, secret management, monitoring, and a formal clinical review process.
