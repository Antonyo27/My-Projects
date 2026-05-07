# Corporate Egg Production and Delivery Business — Deployment

A high-level, sanitized overview of how the system runs in production. Specific server addresses, credentials, client identity, and provider account details are intentionally omitted.

---

## Topology

A single-region deployment serving multiple physical locations (farms, warehouses, management offices) as users of a centralized web application.

```
                ┌────────────────────────┐
                │    DNS / Edge Layer    │
                └───────────┬────────────┘
                            │ HTTPS
                            ▼
                ┌────────────────────────┐
                │  Nginx (Reverse Proxy) │
                │  • TLS terminator       │
                │  • Static asset serve   │
                └───────────┬────────────┘
                            │
                  ┌─────────▼─────────┐
                  │  Laravel App      │
                  │  (PHP-FPM)        │
                  │  + Queue workers  │
                  │  + Scheduler      │
                  └─────────┬─────────┘
                            │
                  ┌─────────▼─────────┐
                  │   PostgreSQL      │
                  └───────────────────┘
```

Every operational role — farm staff, warehouse staff, management — connects to the same centralized application from their respective locations. Multi-location separation is enforced **at the application layer** via RBAC and location scoping, not by deploying per-location instances.

---

## Process Supervision

| Process | Purpose |
|---------|---------|
| **PHP-FPM** | Serves HTTP requests for the Laravel application |
| **Queue workers** | Async jobs: PDF dispatch receipt generation, Excel export streaming, audit log persistence, scheduled rollups |
| **Scheduler** | Daily yield rollups, in-transit anomaly sweeps, mortality trend recalculation |

All processes are supervised so they restart automatically on crash or deploy.

---

## Containerization

The application is packaged as a **Docker image** and deployed via Docker Compose. This keeps local development, staging, and production environments identical, eliminating "works on my machine" friction across the engineering workflow.

Dependencies (PHP version, Composer packages, Node + Vite for asset builds) are pinned in the image so deployments are deterministic.

---

## Build & Deploy

The deployment flow:

1. Lint + run tests
2. Build production frontend assets via Vite
3. Build the Docker image and tag it
4. Deploy to the host: pull the new image, run `php artisan migrate --force`, restart workers, swap traffic
5. Smoke-test the executive dashboard and a representative dispatch + receive flow before considering the deploy successful

Migrations are gated behind a manual approval step in protected environments. Rollback is an image-tag swap.

---

## Document Generation Path

PDF dispatch receipts and operational reports are generated **server-side on demand** via:

- **`barryvdh/laravel-dompdf`** — for PDF dispatch receipts and report exports
- **`openspout/openspout`** — for fast, memory-efficient streaming Excel exports (critical for multi-year historical data exports that would otherwise OOM)
- **`simplesoftwareio/simple-qrcode`** — for embedding QR codes in dispatch receipts

These run inside dedicated queue workers so a long export never blocks the request thread.

---

## Backups & Disaster Recovery

- **PostgreSQL** — nightly logical backups (`pg_dump`) shipped off-host with a retention window appropriate for the client's compliance and operational needs.
- **Generated documents** — PDFs and Excel files are re-renderable from source data; only the source data needs to be backed up.
- **Restore drills** — periodic restore tests against a staging environment to verify backup integrity end-to-end.

---

## Observability

- **Application logs** — structured JSON logs written to stdout, captured by the host journal.
- **Audit logs** — every consequential action (dispatch, receive, breakage, RBAC change, configuration change) is written to a dedicated audit table with actor, timestamp, and before/after state.
- **Health checks** — a lightweight health endpoint probed by external monitoring; alerts route to email + chat.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
