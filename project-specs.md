# Datagroom — Project Specs (Agent Orientation)

A single-page map of the system so an agent can orient quickly before planning a task. Pair this with `LESSONS.md` (memory) and `.cursor/rules/` (enforced rules).

## What Datagroom is

A collaborative dataset-management tool — think Google Sheets backed by MongoDB — with real-time multi-user editing, per-cell locking, JIRA integration, and an LLM/RAG layer for natural-language querying of datasets.

## Subprojects

### 1. `datagroom-gateway/` — Backend (git submodule)
Express REST + Socket.io WebSocket server. Owns: dataset CRUD/filter/pagination/bulk-edit, auth (JWT + optional LDAP/AD), real-time cell locking, JIRA integration, RAG controller.
- Stack: Node ≥22, Express 4, MongoDB driver v6, Socket.io v2, Pino, CommonJS.
- Key files: `server.js`, `dbAbstraction.js` (singleton), `acl.js`, `perRowAccessCheck.js`, `routes/dsReadApi.js`, `routes/mongoFilters.js`.
- Data model: **one MongoDB database per dataset** with collections `data`, `metaData`, `editlog`, `attachments`.
- Run: `node server.js disableAD=true` (port 8887). Tests: `npm test`.

### 2. `datagroom-ui/` — Legacy frontend (git submodule)
React 16 + CRA 3.1.1 SPA, Redux (thunks), class components, React Router 5, forked react-tabulator. Still served in production. Run: `npm start` (port 3000); build: `npm run build`.

### 3. `datagroom-ui-2026/` — Current frontend
React 19 + Vite 7 + React Router 7, TanStack Query, functional components, ESM. API layer in `app/api/`, data hooks in `app/hooks/`, pages in `app/pages/`, auth in `app/auth/`. **Default target for new UI work.** Dev: `npm run dev`; build: `npm run build`.

### 4. `datagroom-mcp-server/` — Python MCP server
Exposes Datagroom datasets to external LLMs (Cursor) via Personal Access Tokens. Async `httpx`, pydantic, MCP SDK. Source in `src/datagroom_mcp/`. Run via `datagroom-mcp` entry point; tests: `pytest`.

## Cross-cutting subsystems

- **Auth:** JWT cookies for browser sessions; PAT Bearer tokens for the MCP server. See `PAT_IMPLEMENTATION_AND_AUTH.md`.
- **Access control:** dataset-level ACL + row-level checks, enforced in the gateway before every operation. Everything (including MCP) goes through this.
- **RAG / NL querying:** hybrid retrieval (MongoDB aggregation + vector search + LLM) routed in the gateway. See `HYBRID_RAG_ARCHITECTURE.md`.
- **Real-time:** Socket.io v2 end-to-end (cell locking, live updates).

## How a feature usually flows

1. Define/extend the gateway API (route + ACL + DbAbstraction op + editlog).
2. Add validation/tests in `datagroom-gateway/tests/`.
3. Wire the UI in `datagroom-ui-2026` (api helper → TanStack Query hook → page/component).
4. If external/LLM access is needed, expose a tool in `datagroom-mcp-server`.
5. Validate each touched subproject's gate, self-critique, record lessons.

## Reference docs
- `HYBRID_RAG_ARCHITECTURE.md` — RAG design.
- `PAT_IMPLEMENTATION_AND_AUTH.md` — PAT + auth flow across UI/gateway/MCP.
- `datagroom-gateway/AGENTS.md`, `datagroom-ui/AGENTS.md` — per-submodule deep guides.
- `LESSONS.md` — accumulated lessons; `.cursor/rules/` — enforced rules.
