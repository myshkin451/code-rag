# AGENTS.md

This file gives persistent guidance to coding agents working in this repository.

## Mission

Help evolve this project from a personal MVP into a maintainable Code RAG product without losing the parts that already work.

Primary goals:

- keep the repo usable and honest
- preserve the existing end-to-end indexing/search flow
- improve engineering quality incrementally
- prefer small, verifiable changes over big rewrites

## Project Snapshot

This repository is a local-first Code RAG system with two main parts:

- a FastAPI backend for indexing, search, explain, and agent routes
- a VS Code extension that builds indexes and consumes backend APIs

Current architecture:

- `api/`: FastAPI entrypoints and request handling
- `ai/`: LLM/provider integration and agent logic
- `indexer/`: zip -> chunk -> ingest pipeline
- `retriever/`: hybrid search over Chroma
- `clients/vscode/`: VS Code extension UI and API client
- `eval/`: lightweight evaluation scripts

## Current Truth

Agents should optimize for the repository's real behavior, not aspirational docs.

- Official indexing support is currently JavaScript/TypeScript only.
- The VS Code extension currently uploads more file types than the backend can truly chunk.
- `explain` has a newer provider/config path than `agent`, and these should be kept aligned over time.
- This repo is in MVP shape: core flow exists, but tests, config hygiene, and dependency hygiene are incomplete.

## Working Style

- Prefer targeted improvements over broad rewrites.
- Keep changes easy to review and easy to revert.
- Update docs when behavior changes.
- If code and docs disagree, either fix the mismatch or clearly note it.
- Preserve workspace isolation behavior unless explicitly redesigning it.
- Avoid introducing new infrastructure unless it solves a clear existing pain.

## High-Value Priorities

When choosing what to do next, bias toward these:

1. Align product claims with actual behavior.
2. Improve developer trust in the repo.
3. Reduce inconsistency between backend and extension.
4. Strengthen the core indexing/search/explain loop before adding new features.

Current top priorities:

1. Align allowed extension upload types with backend chunker support.
2. Unify LLM configuration flow between `/explain` and `/agent/explain`.
3. Make search result previews actually useful.
4. Clean up dependencies and remove obviously unused or placeholder files.
5. Add lightweight smoke tests before deeper feature work.

## Repo Guardrails

- Do not claim multi-language support unless chunking and retrieval truly support it.
- Do not commit `.DS_Store`, local caches, `data/`, `logs/`, or dependency directories.
- Do not silently widen scope. This repo benefits from tighter scope, not broader ambition.
- Do not replace working core flows just because they look simple.
- Do not leave placeholder config or empty files without either filling or removing them.

## Change Rules By Area

### Backend API

- Keep route behavior explicit and easy to trace.
- Prefer moving logic into helper/services if `api/main.py` gets larger.
- Keep response shapes stable unless there is a strong reason to change them.

### Indexing Pipeline

- Any change to upload, file filtering, or chunking must be checked against the VS Code extension behavior.
- Prefer deterministic outputs and stable IDs.
- Do not expand language claims without parser-level support.

### Retrieval

- Favor simple, explainable ranking over clever but opaque heuristics.
- If ranking changes, note expected impact in the PR/summary.

### LLM / Agent

- Keep provider resolution consistent across explain and agent paths.
- Prefer evidence-grounded behavior over more autonomous behavior.
- Avoid changes that increase hallucination risk without improving grounding.

### VS Code Extension

- Keep commands and settings consistent with backend behavior.
- If the extension exposes a capability, verify the backend actually supports it.
- Preserve a simple onboarding path for local development.

## Verification Expectations

Choose the lightest checks that meaningfully validate the change.

Useful checks in this repo:

- Python syntax check:
  `python3 -m compileall api ai indexer retriever eval`
- Backend health check after startup:
  `curl http://127.0.0.1:8000/ping`
- Docker stack:
  `docker compose up -d --build`
- VS Code extension build, after installing dependencies in `clients/vscode/`:
  `npm run build`
- Eval script when backend is running:
  `python3 eval/run_eval.py`

If a check cannot be run, say so clearly and explain why.

## Definition Of Done

For most changes, done means:

- the relevant code path is updated
- the change matches actual product behavior
- basic verification was run, or the inability to run it is stated
- related docs/config were updated if needed
- no unrelated local files were committed

## Good Agent Outputs

Good work in this repo usually includes:

- a concise explanation of what changed
- any important tradeoffs or assumptions
- what was verified
- what remains risky or intentionally deferred

## Suggested Near-Term Roadmap

If no higher-priority user direction is given, the safest sequence is:

1. repo hygiene and documentation cleanup
2. backend/extension behavior alignment
3. provider and agent consistency cleanup
4. lightweight test and smoke-check coverage
5. only then broader feature expansion

