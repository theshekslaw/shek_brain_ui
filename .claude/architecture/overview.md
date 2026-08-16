# Architecture — shek_brain_ui

## Purpose

Personal dashboard for the brain. Read + explore + chat + share. Runs against `shek_brain` (data) and `model_engine` (chat streaming). Not a public product.

## Non-purposes

- Not multi-user. One user (you). Bearer token in localStorage is enough.
- Not offline-capable. Requires Tailscale + brain reachable.
- Not a full CRUD UI. Ingest happens via workflows (Temporal) or MCP; UI mostly reads.

## Stack

- **Vite 6** + **React 19** + **TypeScript strict**.
- **TanStack Router** (file-based routes).
- **TanStack Query** for server state.
- **Zustand** for tiny UI state (open panels, active graph focus).
- **Tailwind CSS** via **`uiCommonPreset`** from `@shek/ui-common`.
- **`react-force-graph-2d`** for the Obsidian-look knowledge graph.
- **`@shek/ui-common`** for primitives, hooks, fetcher, types.

## Routes (v0.1)

```
/                    dashboard: recent items, workflow status, engine health
/graph               Obsidian-style force-directed graph over shek_brain graph_view
/items               list + filter + semantic search
/items/$id           item detail: metadata + mindmap + neighborhood subgraph + blob viewer
/chat                streaming chat with model_engine (qwen2.5:3b warm)
/kinds               registered kinds + their JSON schemas
```

## API boundary

All backend traffic through TanStack Query + `@shek/ui-common` fetcher. Two clients:

- `lib/api/brain.ts` → `shek_brain` endpoints.
- `lib/api/model_engine.ts` → `model_engine` endpoints (chat streaming via `useSSE`).

Query keys via `keys` factory from `@shek/ui-common`. No inline arrays.

## Auth

`useAuthToken()` — token stored in localStorage. `fetcher` injects `Authorization: Bearer <token>` on every call. Login screen v0.1 is a paste-your-token form (no OAuth).

## Graph rendering

`components/GraphView.tsx` wraps `react-force-graph-2d`. Consumers of the UI only see typed props (`{ nodes, edges, onFocus }`). Everything about the library — canvas, physics, click handling — is contained here. Zoom + drag + focus supported; details drawer opens on click.

Node color per kind, edge color per relationship type — pulled from Tailwind CSS variables via `getComputedStyle`. Consistent with the preset.

## Design decisions

1. **File-based routing.** TanStack Router generates the route tree; no manual registration.
2. **All server state via TanStack Query.** No `useEffect`-based fetches. No global loading spinner.
3. **Optimistic mutations only where rollback is trivial** (renames, tags). Never for ingest.
4. **Stream chat via SSE.** `useSSE` from `@shek/ui-common`; renders tokens as they arrive.
5. **Graph paint budget.** Cap force-graph node count at 500 in-view. If brain returns more, use `depth=1` and let the user expand.

## Files (target for v0.1)

```
src/
├─ main.tsx
├─ App.tsx
├─ routes/
│  ├─ __root.tsx
│  ├─ index.tsx
│  ├─ graph.tsx
│  ├─ items.index.tsx
│  ├─ items.$id.tsx
│  ├─ chat.tsx
│  └─ kinds.tsx
├─ components/
│  ├─ GraphView.tsx
│  ├─ ItemCard.tsx
│  ├─ ChatWindow.tsx
│  ├─ TokenPasteForm.tsx
│  └─ ui/                       thin re-exports of @shek/ui-common where we override
├─ lib/
│  ├─ api/
│  │  ├─ brain.ts
│  │  ├─ model_engine.ts
│  │  └─ index.ts
│  └─ query.ts                  QueryClient config
├─ stores/
│  ├─ auth.ts                   (just wraps @shek/ui-common's useAuthToken; may drop)
│  └─ graph.ts                  focus id, depth
├─ styles/
│  ├─ globals.css               token CSS variables (light + dark)
│  └─ theme.css                 palette definitions
└─ types/
   └─ index.ts                  re-export from @shek/ui-common/types
```

## Not-yet-decided

- **Dark mode toggle in v0.1?** Tokens support it; toggle UX is trivial. Ship on if the palette looks good.
- **Item detail: which blob viewer for PDFs?** `<embed src>` is the fastest; upgrade to react-pdf if we need annotations.
- **Rendering figures inline in item detail.** Would fetch signed URLs from `shek_brain`. Add if MinIO signed URL support ships in brain v0.1.
