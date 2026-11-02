# Medical RAG frontend

This directory contains the React 19 + TypeScript + Vite client for the Medical RAG Hallucination Detection application.

## Development

From this directory:

```powershell
npm install
npm run dev -- --host 127.0.0.1
```

The development server runs at <http://127.0.0.1:5173>. It expects the FastAPI backend at `http://127.0.0.1:8000`.

To override the backend URL, create `.env`:

```text
VITE_API_BASE_URL=http://127.0.0.1:8000
```

## Scripts

```powershell
npm run dev
npm run build
npm run lint
npm run preview
```

## Frontend responsibilities

- Poll backend readiness through `GET /health`.
- Submit medical questions to `POST /api/ask`.
- Render the answer, source chunks, claim verification, confidence, decision, correction state, and pipeline telemetry.
- Provide evaluation and knowledge-base explorer views through the corresponding backend routes.

Backend contracts are represented in `src/types/api.ts`, and HTTP integration is centralized in `src/services/api.ts`.

See the repository [README](../README.md) for full-stack setup and [team guidelines](../docs/TEAM_GUIDELINES.md) for contribution rules.
