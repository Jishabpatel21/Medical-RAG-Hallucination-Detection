# Medical RAG Hallucination Detection

An evidence-grounded medical question-answering application. The system retrieves passages from a trusted diabetes guideline, generates an answer with Google Gemini, verifies answer claims with semantic search and natural-language inference, scores confidence, and optionally self-corrects unsupported answers.

> **Important:** This is a research/demo application, not a medical diagnostic tool. It does not replace a qualified healthcare professional.

## What the application does

```text
Question
   ↓
FAISS retrieval from the trusted PDF
   ↓
Grounded Gemini answer
   ↓
Claim extraction and evidence verification
   ↓
Confidence scoring and hallucination decision
   ↓
Self-correction when evidence is insufficient
   ↓
Answer, evidence, scores, and pipeline trace
```

The current knowledge base contains `dataset/raw/niddk_guiding_principles_diabetes.pdf`, an 83-page NIDDK diabetes guideline. The ingestion script extracts 268 chunks using a 1,000-character size and 200-character overlap, then stores a 384-dimensional `all-MiniLM-L6-v2` FAISS index.

## Repository layout

```text
backend/
  app/
    main.py                    FastAPI application and API routes
    config.py                  Environment-backed settings
    schemas.py                 Request and response models
    services/
      retrieval.py             Embeddings, FAISS search, reference filtering
      llm.py                   Gemini generation and retry/fallback logic
      verification.py          Claim verification, confidence, decisions
      correction.py            Evidence-grounded self-correction
    evaluation/                Benchmark, metrics, and report export
  requirements.txt
frontend/
  src/
    App.tsx                    Main UI and backend health polling
    services/api.ts            Backend client
    components/                Answer, evidence, evaluation, and KB views
  package.json
dataset/raw/                   Trusted source PDF
scripts/build_index.py         Build vector_db artifacts from the PDF
tests/                         API, evaluation, retrieval, and verification tests
vector_db/                     Generated FAISS index and metadata
docs/                          Project, roadmap, team, and handoff documentation
```

## Prerequisites

- Python 3.10 or newer (Python 3.13 works in the current environment)
- Node.js 20 or newer
- npm
- A Google Gemini API key for answer generation
- Internet access on first run to download Hugging Face model weights

## Local setup

### 1. Install backend dependencies

From the repository root:

```powershell
python -m pip install -r backend\requirements.txt
```

### 2. Configure the backend

Copy `backend/.env.example` to `backend/.env` and set your own key:

```text
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL_NAME=gemini-3-flash-preview
CORS_ORIGINS=http://127.0.0.1:5173,http://localhost:5173
```

Never commit `backend/.env`. It is ignored by Git.

### 3. Build the local knowledge index

The generated files are intentionally not required to be committed. Build them locally:

```powershell
python scripts\build_index.py
```

This creates:

```text
vector_db/index.faiss
vector_db/metadata.json
vector_db/embeddings.npy
```

### 4. Install frontend dependencies

```powershell
Set-Location frontend
npm install
```

### 5. Start the backend

Open a terminal at the repository root:

```powershell
Set-Location backend
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

The first startup loads the embedding and NLI models and may take several minutes.

### 6. Start the frontend

Open another terminal:

```powershell
Set-Location frontend
npm run dev -- --host 127.0.0.1
```

Open <http://127.0.0.1:5173>. The frontend uses `http://127.0.0.1:8000` by default. To override it, create `frontend/.env`:

```text
VITE_API_BASE_URL=http://127.0.0.1:8000
```

## Running the application

Check backend readiness:

```powershell
Invoke-WebRequest http://127.0.0.1:8000/health
```

A ready response reports `embedding`, `faiss_index`, `nli`, and `gemini` as `true`.

Useful URLs:

| Resource | URL |
|---|---|
| Frontend | <http://127.0.0.1:5173> |
| API docs | <http://127.0.0.1:8000/docs> |
| Health | <http://127.0.0.1:8000/health> |

## API surface

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/health` | Report model and index readiness |
| `POST` | `/api/ask` | Run retrieval, generation, verification, scoring, and correction |
| `GET` | `/api/evaluation/benchmark` | Read benchmark metadata and items |
| `POST` | `/api/evaluation/run` | Run the evaluation benchmark |
| `POST` | `/api/evaluation/export/markdown` | Export evaluation results as Markdown |
| `POST` | `/api/evaluation/export/json` | Export evaluation results as JSON |
| `GET` | `/api/knowledge-base/overview` | View knowledge-base statistics |
| `GET` | `/api/knowledge-base/chunks` | Search and paginate indexed chunks |

Example request:

```powershell
Invoke-RestMethod `
  -Uri http://127.0.0.1:8000/api/ask `
  -Method Post `
  -ContentType "application/json" `
  -Body '{"query":"What are the risk factors for type 2 diabetes?"}'
```

## Testing and quality checks

Frontend:

```powershell
Set-Location frontend
npm run build
npm run lint
```

Backend tests:

```powershell
python -m unittest discover -s tests -p "test_*.py" -v
```

The live API tests need a configured backend, generated vector index, downloaded models, and a working Gemini key. Tests that call Gemini may incur API usage.

## Rebuilding the knowledge base

When the trusted PDF changes:

1. Replace or add the source document under `dataset/raw`.
2. Update `PDF_FILENAME` in `backend/app/config.py` if the filename changes.
3. Run `python scripts\build_index.py`.
4. Restart the backend.
5. Run retrieval and API tests.

The index, metadata, and cached embeddings must be generated from the same source and embedding model. Do not mix artifacts from different documents or model versions.

## Design and safety notes

- Gemini is prompted to use only retrieved context and to state when context is insufficient.
- Retrieval prioritizes clinical chunks and filters bibliographic reference blocks.
- NLI and semantic similarity are complementary signals; neither is a clinical safety guarantee.
- Confidence and hallucination labels are model-derived indicators, not medical truth.
- Do not send protected health information or other sensitive data to an external model.
- Keep API keys in local environment files or a secret manager, never in source code or notebooks.

## Documentation

- [Project plan](docs/PROJECT_PLAN.md)
- [Roadmap](docs/ROADMAP.md)
- [Team guidelines](docs/TEAM_GUIDELINES.md)
- [Team handoff](docs/TEAM_HANDOFF.md)

## License

See [LICENSE](LICENSE).
