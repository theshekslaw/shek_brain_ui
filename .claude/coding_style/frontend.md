# Frontend style — shek_brain_ui

Canonical style lives in `shek_ui_common_utility`:

<https://github.com/theshekslaw/shek_ui_common_utility/blob/main/.claude/coding_style/frontend.md>

## shek_brain_ui-specific rules

- **Routes** live under `src/routes/` following TanStack Router file-based routing.
- **All API traffic goes through `@shek/ui-common` fetchers** — never call `fetch` directly.
- **Graph rendering** uses `react-force-graph-2d`; wrapped in `components/GraphView.tsx`. Do not import the library outside that wrapper.
- **Chat streaming** uses the `useSSE` hook from `@shek/ui-common`.
- **Auth token** stored in localStorage; access via `useAuthToken()` hook — never `localStorage.getItem` directly.
