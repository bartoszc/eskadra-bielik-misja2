# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A serverless RAG (Retrieval-Augmented Generation) demo on Google Cloud, built as a tutorial/codelab ("Eskadra Bielik – Misja 2"). It answers Polish-language questions over a custom knowledge base. The reference dataset is a set of hotel rules (`vector_store/hotel_rules.csv`).

Most user-facing strings, comments, and prompts are in **Polish** — match that language when editing them.

## Architecture

Four independently deployed pieces, all running on **Cloud Run** in `europe-west1`:

1. **`llm/`** — the LLM. An Ollama container serving the Polish [Bielik](https://ollama.com/SpeakLeash/bielik-4.5b-v3.0-instruct) model (`SpeakLeash/bielik-4.5b-v3.0-instruct:Q8_0`). GPU-backed (nvidia-l4), deployed as service `bielik`. The model weights are baked into the image at build time (`RUN ollama pull` in the Dockerfile).
2. **`embedding_model/`** — the embedding model. Another Ollama container serving Google's `embeddinggemma`. CPU-only, deployed as service `embedding-gemma`.
3. **`vector_store/`** — BigQuery setup. `init_db.py` creates the `rag_dataset.hotel_rules` table with schema `id STRING, content STRING, embedding FLOAT64 REPEATED`. Retrieval uses BigQuery's native `VECTOR_SEARCH` (COSINE, top_k=3) — there is no separate vector DB.
4. **`orchestration/`** — the FastAPI app (`main.py`) that ties everything together and serves the Web UI (`static/index.html`). Deployed as service `orchestration-api`; the only public (`--allow-unauthenticated`) service.

**Request flow for `/ask` (RAG):** query → `embedding-gemma` (vector) → BigQuery `VECTOR_SEARCH` (top 3 docs) → build prompt with context → `bielik` → answer + `context_used`. The `/ask_direct` endpoint skips retrieval and hits Bielik directly, so the UI can show RAG vs. baseline side by side.

**Service-to-service auth:** the LLM and embedding services are private (`--no-allow-unauthenticated`). `main.py` calls them with a Google-signed **identity token** (`get_id_token`), fetched via `google.oauth2.id_token` with a `gcloud auth print-identity-token` fallback for local dev. The audience is the target service URL.

## Configuration (environment variables)

`setup_env.sh` (run with `source setup_env.sh`) sets the shared names: `PROJECT_ID`, `REGION=europe-west1`, `LLM_SERVICE=bielik`, `EMBEDDING_SERVICE=embedding-gemma`, `BIGQUERY_DATASET=rag_dataset`, `BIGQUERY_TABLE=hotel_rules`.

`orchestration/main.py` additionally reads `EMBEDDING_URL` and `LLM_URL` (the deployed Cloud Run URLs). These are resolved and injected automatically by `orchestration/cloud_run.sh` at deploy time via `gcloud run services describe`. Re-source `setup_env.sh` after any new shell.

## Common commands

There is **no build step, test suite, or linter** in this repo — it deploys source directly to Cloud Run (`gcloud run deploy --source .`, which builds via Cloud Build/Buildpacks). Order matters: models and BigQuery must exist before the orchestrator is deployed.

```bash
# 0. Load env vars (do this in every new shell)
source setup_env.sh

# 1. Deploy the models (each script reads $LLM_SERVICE / $EMBEDDING_SERVICE etc.)
cd llm && ./cloud_run.sh && cd ..
cd embedding_model && ./cloud_run.sh && cd ..

# 2. Create the BigQuery dataset + table
cd vector_store && python init_db.py && cd ..

# 3. Deploy the orchestrator (resolves EMBEDDING_URL/LLM_URL itself)
cd orchestration && ./cloud_run.sh && cd ..
export ORCHESTRATION_URL=$(gcloud run services describe orchestration-api --region $REGION --format="value(status.url)")

# Smoke-test the models directly
cd llm && ./llm_test1.sh                  # asks Bielik a question
cd embedding_model && ./embedding_test1.sh  # generates an embedding

# Load data and query the RAG endpoints
curl -X POST "$ORCHESTRATION_URL/ingest" -F "file=@vector_store/hotel_rules.csv"
curl -X POST "$ORCHESTRATION_URL/ask" -H "Content-Type: application/json" \
     -d '{"query": "O której godzinie jest podawane śniadanie?"}'
```

## Notes when editing

- **CSV ingest contract:** `/ingest` expects columns named `id` and `text`; rows missing either are silently skipped. The BigQuery column is `content`, not `text`.
- **`VECTOR_SEARCH` query** in `main.py` interpolates the embedding vector directly into the SQL string (`(SELECT {query_embedding} as embedding)`) — keep that in mind if changing the retrieval query.
- The Ollama model tags (`embeddinggemma`, `SpeakLeash/bielik-4.5b-v3.0-instruct:Q8_0`) are hardcoded in both the Dockerfiles and `main.py`; they must stay in sync.
- The system/RAG prompts live in `main.py` (`/ask` and `/ask_direct`); the README explicitly invites editing these to change the assistant's persona.
