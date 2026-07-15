# SAP BDC Sales 360 — Public (anonymous) build

A static, no-login public build of the SAP BDC Sales 360 dashboard, deployed to
GitHub Pages. Dashboards render from a **point-in-time data snapshot** (no Snowflake
connection, no credentials in the browser). The optional "Cortex Agent" chat page calls
a separate serverless agent when configured.

## How it works

- **Dashboards:** built with `VITE_STATIC=1`. The client reads pre-baked JSON in
  `public/data/*.json` instead of a live `/api`.
- **Customer filter:** Sales 360 filters on include/exclude across ~1,400 customers — a
  pre-baked power set is infeasible, so the public build **bakes the all-customers view**
  and disables the customer filter (Sidebar `showFilters` gate). Forecast is baked per
  version (PLAN / REVISED / STRETCH).
- **Data refresh:** re-run the exporter against the live dashboard server, commit the
  updated `public/data/*.json`, and push — Actions redeploys.
  ```bash
  # from the sales_360_react monorepo, with the server running:
  EXPORT_BASE=http://localhost:3003 node scripts/export-static.mjs
  ```
- **Live agent (optional):** set the repo variable `AGENT_URL` to a deployed Cortex
  agent Worker URL. If unset, the Chat page shows a "not available" notice but the
  dashboards work fully.

## Local build

```bash
npm ci
VITE_STATIC=1 npx vite build      # outputs dist/
python3 -m http.server -d dist    # preview
```

## Deploy

Push to `main` → `.github/workflows/deploy.yml` builds with
`BASE_PATH=/sales-360-public/` and publishes `dist/` to GitHub Pages.

No secrets are stored in this repo. It contains only synthetic SAP demo data.
