# Project plan

## 1. Mission

Provide a transparent medical RAG experience in which every generated answer is accompanied by retrieved evidence, claim-level verification, confidence signals, and a visible decision about support or contradiction.

The application is intended for research, education, and engineering demonstration. It is not a diagnostic or treatment recommendation system.

## 2. Current baseline

| Area | Current implementation |
|---|---|
| Source | 83-page NIDDK diabetes guideline PDF |
| Chunking | Recursive splitter, 1,000 characters, 200-character overlap |
| Knowledge base | 268 chunks |
| Embeddings | Sentence Transformers `all-MiniLM-L6-v2`, 384 dimensions |
| Retrieval | FAISS `IndexFlatL2`, clinical/reference filtering |
| Generation | Google Gemini through `google-genai` |
| Verification | Semantic similarity plus `cross-encoder/nli-deberta-v3-base` |
| Decision | Supported, potential hallucination, or contradicted |
| Correction | Optional evidence-grounded regeneration and re-verification |
| Backend | FastAPI on port 8000 |
| Frontend | React, TypeScript, Vite on port 5173 |

## 3. Product requirements

### R1 — Evidence-grounded answers

The system must retrieve relevant source chunks and instruct the LLM to answer only from that context. Answers should include source/page evidence where available.

### R2 — Claim verification

The system must break an answer into claims and compare claims against retrieved evidence. Verification results must expose supported claims, total claims, consistency, and evidence details.

### R3 — Confidence and decisions

The API must return a bounded confidence score, confidence level, retrieval quality, semantic similarity, NLI entailment, and claim consistency. It must also expose a clear hallucination decision.

### R4 — Self-correction

When the answer is unreliable, the system should generate a constrained correction using the evidence and re-verify the corrected answer before returning it.

### R5 — Transparency

The UI must make sources, claims, evidence, scores, pipeline stages, timings, evaluation, and knowledge-base content inspectable.

### R6 — Operational readiness

The project must provide repeatable setup, index generation, health reporting, tests, and safe secret handling.

## 4. Architecture

```text
React/Vite frontend
        │ HTTP JSON
        ▼
FastAPI backend
        ├── RetrievalService
        │     ├── SentenceTransformer
        │     └── FAISS + metadata + cached embeddings
        ├── LLMService → Google Gemini
        ├── VerificationService
        │     └── NLI cross-encoder + similarity metrics
        ├── CorrectionService
        └── Evaluation and knowledge-base routes
```

The backend loads models and the local index during FastAPI lifespan startup. `/health` reports whether all four runtime dependencies are ready.

## 5. Delivery workflow

1. Maintain the source document and ingestion process.
2. Keep retrieval and verification deterministic where practical.
3. Change one pipeline stage at a time.
4. Add or update tests for behavior changes.
5. Run frontend build/lint and targeted backend tests.
6. Rebuild the index when the source document or embedding model changes.
7. Review API and UI behavior with both supported and unrelated questions.

## 6. Acceptance checks

- A clean setup can install backend and frontend dependencies.
- `scripts/build_index.py` creates all three vector artifacts.
- `/health` returns ready only when embedding, FAISS, NLI, and Gemini are initialized.
- Valid supported questions return sources, claims, confidence, decision, correction state, and pipeline trace.
- Empty questions return HTTP 422.
- Missing index or Gemini configuration is reported as HTTP 503.
- Frontend build succeeds and the UI displays backend readiness.
- Secrets are absent from tracked documentation and source.

## 7. Known constraints

- The current knowledge base is limited to one diabetes guideline.
- Model downloads require network access and local disk space.
- Gemini calls require an API key and can incur usage.
- NLI startup is resource-intensive on CPU.
- Generated vector artifacts are environment outputs and must be rebuilt when inputs change.
