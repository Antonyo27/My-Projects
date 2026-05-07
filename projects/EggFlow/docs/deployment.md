# EggFlow — Deployment

A high-level overview of the deployment topology and operational practices behind EggFlow. Specific server addresses, credentials, and provider account details are intentionally omitted.

---

## Hosting Topology

EggFlow runs on a **Linux VPS (Ubuntu LTS)** with **Nginx** terminating TLS and reverse-proxying to the application. The full stack is containerized via **Docker Compose**, ensuring identical environments across local development, staging, and production.

```
                ┌──────────────────────────────────────────┐
                │            Cloudflare (DNS + CDN)        │
                └───────────────────┬──────────────────────┘
                                    │ HTTPS
                                    ▼
                ┌──────────────────────────────────────────┐
                │            Nginx (Reverse Proxy)         │
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
          ┌─────────────────────────┐
          │   PostgreSQL (primary)  │
          │   Redis (cache + queue) │
          │   SendGrid (email API)  │
          └─────────────────────────┘
```

---

## Process Supervision

EggFlow's runtime is composed of multiple long-lived processes managed by Supervisor / systemd:

| Process | Purpose |
|---------|---------|
| **PHP-FPM** | Serves HTTP requests for the Laravel application |
| **Reverb WebSocket server** | Powers the live chat widget and real-time POS updates |
| **Queue workers** | Async jobs: email dispatch, PDF generation, dashboard cache rebuilds, audit log persistence |
| **Scheduler** | Cron-driven sweeps: daily yield rollups, shift reconciliation reports, low-stock alerts |

All processes are supervised so they restart automatically on crash or deploy.

---

## Caching & Queue Backing

**Redis** serves a dual role:

- **Cache** — dashboard aggregations, frequently-read configuration, RBAC permission lookups
- **Queue** — async job backing for the Laravel queue system

The caching layer is critical for executive dashboards: aggregations across farms, warehouses, and POS locations are expensive enough that a cold dashboard would block the manager view for several seconds. Cached aggregates with short TTLs (and explicit invalidation on inventory writes) keep response times under perceptual thresholds.

---

## Email & Notifications

Transactional email (shift reports, expense approval requests, low-stock alerts, reservation confirmations) is dispatched via **SendGrid**, integrated through Laravel's mail transport. SendGrid handles deliverability, bounce tracking, and webhook callbacks for delivery state.

---

## CI/CD

A **GitHub Actions** pipeline triggers on every push to `main`:

1. Lint (Pint, ESLint) + run Pest test suite
2. Build production frontend assets via Vite
3. Build the Docker image and push to the registry
4. SSH into the VPS, pull the new image, run `php artisan migrate --force`, restart workers

Database migrations are gated behind a manual approval step in protected environments. Rollback is a single command swapping back to the previous image tag.

---

## Backups & Disaster Recovery

- **PostgreSQL** — nightly logical backups (`pg_dump`) shipped to off-site object storage with a 30-day retention window.
- **Media uploads** (receipt scans, expense supporting documents) — synced to off-site object storage on write.
- **Redis** — treated as ephemeral; persisted state is always reconstructable from PostgreSQL.
- **Restore drills** — periodic restore tests against a staging environment to verify backup integrity end-to-end.

---

## Observability

- **Application logs** — structured JSON logs written to stdout, captured by the host journal, shipped to a log aggregator.
- **Audit logs** — written to a dedicated table via `OwenIt Laravel Auditing`, capturing every state-changing action with the responsible actor, before/after values, and timestamp.
- **Health checks** — `/up` endpoint probed by an external uptime monitor; alerts route to email + chat.
- **Error tracking** — exceptions reported with full stack and request context for rapid root-cause analysis.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
