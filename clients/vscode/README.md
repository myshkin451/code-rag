# CodeRAG Agent

> Local-first, AST-aware code search and explain for JavaScript/TypeScript workspaces, with optional agent assistance.

CodeRAG Agent is a VS Code extension backed by a local FastAPI + Chroma indexing service. Its core job is simple:

1. build an index for your current JS/TS workspace
2. search and explain code with evidence
3. optionally let an agent search before answering

## Stable Scope

This extension currently supports:

- JavaScript / TypeScript workspaces only
- local-first indexing through your own backend
- `Build Index`, `Search`, `Explain`, and optional `Agent` flows

This extension does not try to replace a full hosted coding agent.

## Features

- AST-aware chunking for JS/TS code
- Workspace-isolated Chroma collections
- One-click index build from the active workspace
- Hybrid semantic + symbol-aware search
- Evidence-grounded explanations
- Optional agent assistance using the same workspace-aware backend

## Install And Configure

1. Install **CodeRAG Agent** from the VS Code Marketplace.
2. Start the backend from the repository root:

```bash
cp .env.example .env
docker compose up -d --build
```

3. In VS Code settings, configure:

- `rag.apiBase`
- `rag.apiKey` if needed
- `rag.providerOverride` if you want to force a provider
- `rag.topK`
- `rag.symbolBoost`
- `rag.maxTokens`
- `rag.maxCtxChars`

## Commands

- `RAG: Build Index (Current Workspace)`
- `RAG: Search Code`
- `RAG: Explain Selection`
- `RAG: Ask Code Agent`

## How It Works

- The extension zips supported source files from the current workspace
- The backend unpacks the archive and chunks JS/TS code with Tree-sitter
- Chunks are embedded into a workspace-specific Chroma collection
- Search and Explain use `x-workspace-id` to stay isolated per workspace
- Agent mode can search the same workspace before answering

## Supported Files

Index upload is intentionally limited to:

- `.js`
- `.jsx`
- `.ts`
- `.tsx`
- `.mjs`
- `.cjs`

If your workspace does not contain supported files, the extension will stop early with a clear message.

## Provider Notes

- `openai` is the best-tested provider path for Agent mode
- `local` works through an OpenAI-compatible local endpoint
- `qwen_api` follows the same request-scoped provider selection path as Explain

## Privacy

- Indexing happens inside your own Docker-backed backend
- Your code is sent to your configured backend, not to a hosted CodeRAG service
- Any external model traffic depends on the provider you configure

## Screenshots

![RAG Search View](images/search-view.png)
![Explain Panel](images/explain-panel.png)
![Build Index](images/build-index.png)

## Known Limitations

- JS/TS only
- Agent is an optional enhancement, not the primary workflow
- Very large workspaces may need zip and retrieval tuning

## Suggested Workflow

For the best experience:

1. build the index
2. use Search to find symbols and entrypoints
3. use Explain for grounded summaries
4. use Agent only when you want a broader guided answer
