# LESSONS.md — Datagroom Agent Long-Term Memory

This file is the durable memory for AI agents working in this repo. **Read it before starting a task** and **append to it after any significant iteration** (see `.cursor/rules/70-self-improvement.mdc`).

Format for new entries (newest at the top of each section):

```
- (YYYY-MM-DD) [subproject] Insight — what happened and the rule to follow next time.
```

Subproject tags: `gateway`, `ui` (legacy), `ui-2026`, `mcp`, `repo` (cross-cutting).

---

## Architecture & Conventions

- (2026-06-07) [repo] Monorepo with 4 subprojects: `datagroom-gateway` (Node/Express backend), `datagroom-ui` (legacy React 16/CRA), `datagroom-ui-2026` (current React 19/Vite), `datagroom-mcp-server` (Python MCP). New UI work defaults to `datagroom-ui-2026`.
- (2026-06-07) [repo] `datagroom-gateway` and `datagroom-ui` are **git submodules** — edits there are commits to separate repos. Their canonical guidance is each repo's own `AGENTS.md`.
- (2026-06-07) [gateway] DB model is **one MongoDB database per dataset** (not per collection); each has `data`, `metaData`, `editlog`, `attachments`. Always go through the `DbAbstraction` singleton.
- (2026-06-07) [gateway] Route contract is ACL → per-row access → business logic → editlog → respond. This order is load-bearing for security.
- (2026-06-07) [ui-2026] Uses **2-space indent / single quotes**; the legacy `datagroom-ui` uses **4-space / double quotes**. Match the subproject you're in, not a global default.
- (2026-06-07) [ui-2026] Server state is managed with TanStack Query hooks under `app/hooks/`; API calls go through `app/api/client.js`, never raw `fetch` in components.

## Pitfalls & Gotchas

- (2026-06-07) [gateway] Pre-existing failing test: `dbAbstraction.test.js › should delete documents correctly` expects `removeOne(...).n === 1`, but the mongodb v6 driver returns `{ deletedCount }` (not `{ n }`). Unrelated to feature work; fix `removeOne`/the test together when touching `dbAbstraction.js`.
- (2026-06-07) [gateway] `editlog` docs store `date` as a JS `Date()` **string** (not a Date/ISO) — not lexicographically sortable. To get the most recent edit, sort by `_id` desc (ObjectIds are monotonic), `limit: 1`. `dbAbstraction.find(db, table, query, { sort, limit })` passes options straight to the native driver, so `sort`/`limit` work.
- (2026-06-07) [ui-2026] On Windows/PowerShell, `npm run build` prints the Vite chunk-size warning to stderr, which surfaces as a `node.exe RemoteException`; the build still succeeds (exit 0, `✓ built`). Don't mistake that warning for a failure.
- (2026-06-07) [repo] The submodule `AGENTS.md` files contain **stale version numbers** (e.g. claim Node 12 / `mongodb` v3.5.9 / Express 4.17). Reality per `package.json`: Node ≥22, `mongodb` ^6.21, Express ^4.21. **Always trust `package.json` / `pyproject.toml` over prose.**
- (2026-06-07) [gateway] Socket.io is **v2** on both server and client (incl. ui-2026). Do not upgrade one side independently.
- (2026-06-07) [gateway] body-parser limit is **200 MB** to support large uploads — don't lower it.
- (2026-06-07) [ui] `DsView.js` is ~2300 lines and the most fragile component; extensive changes need human review.

## Reusable Patterns & Commands

- (2026-06-07) [repo] Cross-cutting read-only feature pattern (e.g. "last modified by"): derive from the existing `editlog` instead of adding mutable state to the hot edit path — zero schema change, no write-path risk, always consistent. Gateway: new ACL-checked GET route → UI: helper in `app/api/ds.js` → TanStack Query hook in `app/hooks/` → render in component.
- (2026-06-07) [gateway] Validate with `npm test` (Jest + `mongodb-memory-server`; first run downloads a Mongo binary — slow, not a failure).
- (2026-06-07) [ui-2026] Validate with `npm run build` (no test runner configured yet).
- (2026-06-07) [mcp] Validate with `pytest` inside the venv; entry point is `datagroom_mcp.server:main`.
- (2026-06-07) [repo] Environment is Windows / PowerShell — Unix tools like `head`/`grep`/`cat` are not available in the shell; use the editor's read/search tools instead.

## Doc/Rule Corrections Log

- (2026-06-07) [repo] Logged the stale-version discrepancy above; the canonical fix belongs in each submodule's `AGENTS.md` when next editing those repos.
