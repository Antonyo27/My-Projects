# Corporate Egg Production and Delivery Business — Deployment

A high-level, sanitized overview of how the system runs in production. Specific server addresses, credentials, client identity, and provider account details are intentionally omitted.

---

## Topology

A single-region deployment serving owners and administrative operators of the poultry business.

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
                  │  + Scheduler      │
                  └─────────┬─────────┘
                            │
                  ┌─────────▼─────────┐
                  │   PostgreSQL      │
                  └───────────────────┘
```

---

## Process Supervision

| Process | Purpose |
|---------|---------|
| **PHP-FPM** | Serves HTTP requests for the Laravel + Inertia application |
| **Scheduler** | Performs daily operational date resets, background flag evaluation, and cleanup tasks |

All processes are supervised so they restart automatically on crash or deploy.

---

## Containerization

The application is packaged as a **Docker image** and deployed via Docker Compose. This keeps local development, staging, and production environments identical, eliminating "works on my machine" friction across the engineering workflow.

Dependencies (PHP 8.3 version, Composer packages, Node + Vite for Inertia/Vue asset builds) are pinned in the image so deployments are deterministic.

---

## Build & Deploy

The deployment flow:

1. Run backend tests (PHPUnit) and frontend linter
2. Build production frontend assets via Vite (`npm run build`)
3. Build the Docker image and tag it
4. Deploy to the host: pull the new image, run `php artisan migrate --force`, restart PHP-FPM and Vite, swap traffic
5. Smoke-test the single-screen dashboard and a daily reconciliation cycle before considering the deploy successful

Migrations are gated behind a manual approval step in protected environments. Rollback is a simple image-tag swap.

---

## Backups & Disaster Recovery

- **PostgreSQL** — nightly logical backups (`pg_dump`) shipped off-host to secure storage with a weekly retention window.
- **System States** — since all operational logs (receivings, categorizations, mortality, settings) are transaction-backed, the database holds the entire system state. No file uploads or document binaries need backing up.
- **Restore drills** — periodic restore tests against a staging environment to verify backup integrity end-to-end.

---

## Observability

- **Application logs** — structured Laravel logs written to storage, captured by the host system log viewer.
- **Error monitoring** — lightweight error capturing on state-saving forms to track connection dropouts or validation exceptions.
- **Health checks** — a lightweight health endpoint probed by external monitoring; alerts route to owner email.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
