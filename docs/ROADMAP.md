# Roadmap

The project has moved beyond the original notebook-only prototype. Items below describe the current product and the next engineering increments.

## Completed

- Trusted diabetes guideline added under `dataset/raw`.
- PDF extraction and text normalization implemented.
- Chunking and Sentence Transformer embedding pipeline implemented.
- FAISS index generation implemented in `scripts/build_index.py`.
- FastAPI backend assembled around retrieval, generation, verification, scoring, and correction.
- Gemini retry and model fallback behavior added.
- NLI-based claim verification and confidence scoring added.
- Evaluation benchmark and Markdown/JSON export routes added.
- Knowledge-base overview and chunk explorer routes added.
- React/Vite UI added for answers, evidence, telemetry, evaluation, and knowledge-base exploration.
- API tests and component-level backend tests added.
- Local startup validated with generated index and loaded models.

## Near term

1. Add a reproducible environment lock strategy for Python dependencies.
2. Add a documented local smoke-test script that checks both ports and `/health`.
3. Improve error reporting for model download and Gemini quota failures.
4. Add tests for API response compatibility and knowledge-base pagination.
5. Record benchmark results over time rather than only exporting the latest run.
6. Add a source-document manifest with provenance, version, and checksum.

## Medium term

1. Support multiple approved documents with document-level provenance.
2. Add configurable retrieval strategies and compare them in evaluation.
3. Add answer citation validation and stronger page/chunk alignment checks.
4. Add authentication, rate limiting, and request tracing before any shared deployment.
5. Move model and vector storage behind configurable services for production environments.
6. Add automated frontend browser smoke tests.

## Long term

1. Expand the curated medical knowledge base through a documented review process.
2. Add clinician-reviewed evaluation sets and calibration analysis.
3. Measure robustness to adversarial, ambiguous, and out-of-domain questions.
4. Add deployment automation with secret management, observability, and rollback.
5. Establish governance for source updates, model upgrades, and safety incidents.

## Release gates

No release should proceed without:

- passing frontend build and lint;
- passing backend tests relevant to the changed area;
- a successful clean index build when ingestion changes;
- a healthy `/health` response in the target environment;
- no secrets in tracked files;
- updated documentation for changed configuration or behavior.
