# ScanServe — Deployment

A high-level overview of the deployment topology and operational practices behind ScanServe. Specific server addresses, credentials, and provider account details are intentionally omitted.

---

## Hosting Topology

ScanServe runs on a **Linux VPS (Ubuntu LTS)** behind **Nginx** as a reverse proxy. The application is containerized with **Docker** and orchestrated via Docker Compose, allowing reproducible builds across local, staging, and production environments.

```
                ┌──────────────────────────────────────────┐
                │            Cloudflare (DNS + CDN)        │
                └───────────────────┬──────────────────────┘
                                    │ HTTPS
                                    ▼
                ┌──────────────────────────────────────────┐
                │            Nginx (Reverse Proxy)         │
                │   • Wildcard subdomain routing            │
                │   • TLS via Let's Encrypt (autorenew)     │
                │   • WebSocket upgrade for /reverb         │
                └───────────────────┬──────────────────────┘
                                    │
            ┌───────────────────────┼────────────────────────┐
            ▼                       ▼                        ▼
      ┌───────────┐          ┌───────────┐            ┌───────────┐
      │  Laravel  │          │  Reverb   │            │ Scheduler │
      │  (PHP-FPM)│          │ WebSocket │            │  + Queue  │
      │           │          │  Server   │            │  Workers  │
      └─────┬─────┘          └─────┬─────┘            └─────┬─────┘
            │                      │                        │
            └──────────┬───────────┴────────────────────────┘
                       ▼
                ┌───────────────┐
                │   Database    │
                │  (PostgreSQL  │
                │   or MySQL)   │
                └───────────────┘
```

---

## Domain & Subdomain Strategy

- **Apex** (`scanserve.app`) — public marketing site, organization directory, global search
- **`admin.*`** — Super Admin console for platform-level administration
- **`{org_slug}.*`** — isolated per-organization portals (the "digital lobby")

Wildcard DNS plus a **wildcard TLS certificate** issued by Let's Encrypt covers every tenant subdomain without per-tenant cert provisioning. Cloudflare sits in front for DDoS protection, edge caching of static assets, and DNS management.

---

## Process Supervision

Three categories of long-lived processes run alongside PHP-FPM:

| Process | Purpose | Supervisor |
|---------|---------|------------|
| **Reverb WebSocket server** | Real-time event broadcasting to TV boards, customer devices, staff dashboards | systemd / Supervisor |
| **Queue workers** | Async jobs: PDF report generation, Excel exports, audit log persistence, notification dispatch | Supervisor (multi-worker) |
| **Scheduler** | Cron-driven sweeps: zombie ticket reaper, daily analytics rollups, log pruning | Laravel scheduler invoked from cron |

All three are supervised so they restart automatically on crash or deploy.

---

## CI/CD

A **GitHub Actions** pipeline handles every push to `main`:

1. Lint (Pint, ESLint) + run Pest test suite
2. Build production frontend assets via Vite
3. Build the Docker image and push to the registry
4. SSH into the VPS, pull the new image, run `php artisan migrate --force`, restart workers

Database migrations are gated behind a manual approval step in protected environments. Rollback is a single command: `docker compose pull <prev-tag> && docker compose up -d`.

---

## Backups & Disaster Recovery

- **Database** — nightly logical backups (`pg_dump` / `mysqldump`) shipped to off-site object storage with a 30-day retention window.
- **Media uploads** — synced to off-site object storage on write.
- **Restore drills** — periodic restore tests against a staging environment to verify backup integrity end-to-end.

---

## Observability

- **Application logs** — structured JSON logs written to stdout, captured by the host journal, and shipped to a log aggregator.
- **Audit logs** — `spatie/laravel-activitylog` writes user-facing actions (call-next, serve, no-show, RBAC changes) to a dedicated table for compliance review.
- **Health checks** — `/up` endpoint probed by an external uptime monitor; alerts route to email + chat.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
