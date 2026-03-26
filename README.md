# CodeRAG Agent

> Local-first, AST-aware code search and explain for JavaScript/TypeScript workspaces, with optional agent assistance.

CodeRAG Agent is a VS Code extension plus a local backend that helps you index a JS/TS repository, search it semantically, explain implementation details, and optionally ask an agent grounded in your own codebase.

This project is intentionally scoped as a focused code understanding tool. It is not trying to compete with full general-purpose coding agents.

## Project Status

CodeRAG Agent is in a stable portfolio release state.

- Supported indexing targets: JavaScript / TypeScript workspaces only
- Primary workflow: `Build Index -> Search -> Explain`
- Agent support: available as an optional enhancement
- Deployment model: local-first backend with Docker

## What It Does Well

- AST-aware chunking instead of naive fixed-size text slicing
- Workspace-isolated indexes stored in Chroma
- One-click index building from the current VS Code workspace
- Search results that link back to source locations
- Evidence-grounded explanations over retrieved code
- Optional agent flow that can search before answering

## Current Scope

CodeRAG currently supports:

- `.js`, `.jsx`, `.ts`, `.tsx`, `.mjs`, `.cjs`
- local backend deployment through Docker Compose
- workspace-aware search, explain, and agent requests

CodeRAG does not currently promise:

- production-grade multi-language support
- autonomous coding workflows
- parity with large hosted coding agents

## Architecture

- VS Code extension
  - Commands: `Search`, `Explain`, `Agent`, `Build Index`
  - Uploads the current workspace as a zip archive
  - Sends `x-workspace-id` on all RAG requests

- Backend API
  - FastAPI routes for indexing, search, explain, streaming explain, and agent explain
  - Redis + RQ for background indexing jobs
  - Request-scoped LLM provider selection

- Indexing pipeline
  - Tree-sitter chunking for JS/TS code structure
  - Chroma persistent collections per workspace
  - Hybrid ranking with semantic similarity plus symbol-aware boosting

## Quick Start

### 1. Start the backend

```bash
git clone https://github.com/myshkin451/code-rag
cd code-rag
cp .env.example .env
docker compose up -d --build
```

Backend default:

- API: `http://127.0.0.1:8000`

### 2. Configure the extension

Install **CodeRAG Agent** from the VS Code Marketplace, then set:

- `rag.apiBase`
- `rag.apiKey` if you want to override the backend API key per client
- `rag.providerOverride` if you want to force `openai`, `local`, or `qwen_api`

### 3. Use the primary workflow

1. Run `RAG: Build Index (Current Workspace)`
2. Run `RAG: Search Code`
3. Run `RAG: Explain Selection`
4. Optionally run `RAG: Ask Code Agent`

## Commands

- `RAG: Build Index (Current Workspace)`
- `RAG: Search Code`
- `RAG: Explain Selection`
- `RAG: Ask Code Agent`

## Runtime Requirements

- Docker
- Git
- An OpenAI-compatible model endpoint for the best Explain / Agent experience

Provider notes:

- `openai`: best-tested path for agent behavior
- `local`: supported through an OpenAI-compatible local endpoint
- `qwen_api`: supported through the same request-scoped provider pipeline as Explain

## Repository Layout

- `api/`: FastAPI routes and request handling
- `ai/`: provider logic and agent behavior
- `indexer/`: chunking and Chroma ingest
- `retriever/`: hybrid retrieval logic
- `clients/vscode/`: extension source
- `eval/`: lightweight evaluation scripts

## Verification

Useful checks for this repository:

```bash
python3 -m compileall api ai indexer retriever eval
curl http://127.0.0.1:8000/ping
python3 eval/run_eval.py
```

For the VS Code extension:

```bash
cd clients/vscode
npm install
npm run build
```

## Known Limitations

- Indexing is intentionally limited to JS/TS workspaces
- Agent mode is a secondary capability, not the core product story
- Large monorepos may require tuning zip size, chunking, and retrieval parameters
- Config files under `configs/` document defaults, but not every value is dynamically loaded yet

## Screenshots

![Search view](clients/vscode/images/search-view.png)
![Explain panel](clients/vscode/images/explain-panel.png)
![Build index](clients/vscode/images/build-index.png)

## Chinese Summary

这是一个面向 JavaScript / TypeScript 仓库的本地优先代码理解工具，核心链路是：

- 构建索引
- 搜索代码
- 解释实现
- 可选使用 Agent 做进一步问答

它的重点是把代码检索和解释做好，而不是把自己包装成全能型 coding agent。
