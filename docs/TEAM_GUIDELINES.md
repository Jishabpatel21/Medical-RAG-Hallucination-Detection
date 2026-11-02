# Team guidelines

## Engineering principles

1. **Evidence first.** Do not expand the medical scope beyond the approved source material without an explicit review.
2. **Preserve contracts.** Keep API schemas and frontend types synchronized.
3. **Make uncertainty visible.** Do not hide missing evidence, low confidence, or model failures.
4. **Prefer small changes.** Change one pipeline stage or UI surface at a time.
5. **Test behavior.** Include supported, unsupported, empty, and failure cases.
6. **Protect data.** Never use real patient information and never commit keys or local `.env` files.
7. **Document operational steps.** Any new dependency, environment variable, generated artifact, or endpoint belongs in the README.

## Branch and change workflow

Before work:

```powershell
git pull --rebase origin main
git status
```

After work:

```powershell
git diff --check
git add <changed-files>
git commit -m "Describe the change"
git pull --rebase origin main
git push origin <branch>
```

Do not force-push. Do not reset or discard another contributor's work. Stop and coordinate if a rebase conflict involves behavior or data.

## Python and backend rules

- Keep settings in `backend/app/config.py` and environment variables.
- Keep request/response contracts in `backend/app/schemas.py`.
- Keep retrieval, LLM, verification, and correction responsibilities in their services.
- Raise or return explicit errors; do not silently produce a success-shaped fallback.
- Keep model and index startup status truthful through `/health`.
- Rebuild the FAISS index after changing source text, chunking, or embedding model.

## Frontend rules

- Keep backend calls in `frontend/src/services/api.ts`.
- Keep API types in `frontend/src/types/api.ts`.
- Show loading, unavailable, error, evidence, and correction states explicitly.
- Preserve accessible labels and keyboard operation when changing components.
- Run `npm run build` and `npm run lint` before handing off frontend changes.

## Data and secrets

- Use only approved, documented medical sources.
- Do not commit `backend/.env`, `frontend/.env`, API keys, tokens, or patient data.
- Do not place secrets in notebooks, screenshots, issue descriptions, or logs.
- Treat generated vector artifacts as derived data; record how they were produced.

## Review checklist

- Does the change preserve the retrieval → generation → verification → correction flow?
- Are confidence and hallucination labels still bounded and explainable?
- Are source/page references preserved?
- Are failure paths visible to the user?
- Are tests and documentation updated?
- Was a clean or representative startup check performed?
