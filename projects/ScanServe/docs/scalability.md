# ScanServe — Scalability

How ScanServe is designed to scale from a single small office to large multi-branch institutions, and the trade-offs taken along the way.

---

## Scaling Dimensions

ScanServe scales along three independent axes:

| Axis | Example | Bottleneck |
|------|---------|------------|
| **Tenants** | More organizations onboarded onto the platform | Database connection pool, subdomain provisioning |
| **Concurrency** | More simultaneous customers per branch (peak hours) | WebSocket fan-out, queue worker throughput |
| **Data volume** | Years of historical tickets, feedback, audit logs | Read query latency, report generation time |

Each axis has a distinct mitigation strategy so growth in one dimension doesn't cascade into another.

---

## Multi-Tenancy at Scale

ScanServe uses a **shared-database, shared-schema** multi-tenant model with strict tenant scoping at the ORM layer. The deliberate trade-off:

- **+** Onboarding a new tenant is an `INSERT` and a DNS update — no migrations, no cluster sprawl.
- **+** Cross-tenant analytics (for the Super Admin) are trivially queryable.
- **−** A noisy neighbor can theoretically impact others. Mitigated via per-tenant rate limits and async queueing of expensive operations.
- **−** Hard tenant isolation requirements (e.g., HIPAA-style) would require migrating to a database-per-tenant model. Documented as a known migration path.

For the largest tenants, a future evolution is **logical sharding**: keeping the shared schema but partitioning hot tables (`tickets`, `feedback_responses`) by `organization_id` to keep working sets cache-friendly.

---

## Real-Time Layer

The Reverb WebSocket server is the most concurrency-sensitive component. Mitigations:

1. **Channel scoping** — events broadcast on private channels keyed by `tenant + branch`, so a customer device only receives messages relevant to its branch. A 500-tenant deployment isn't a 500× fan-out problem; it's still a per-branch fan-out.
2. **Horizontal Reverb** — Reverb supports multiple worker processes behind a load balancer for sticky-session-free scale-out, sharing state via a Redis pub/sub bridge.
3. **Backpressure** — clients use Echo's built-in reconnect with exponential backoff so a brief Reverb restart doesn't stampede the server.

---

## Database Strategy

- **Read-heavy paths** (live boards, analytics dashboards) are pushed through a **query result cache** with short TTLs (1–5 seconds) to absorb burst load without sacrificing freshness.
- **Hot writes** (ticket state transitions) are deliberately *not* cached and go straight to the primary, with the cache invalidated by Eloquent model events.
- **Reporting** runs against a **read replica** when one is available, keeping heavy aggregations off the OLTP primary.
- **Long-tail data** — historical tickets older than the active reporting window can be partitioned or archived to cold storage. Archived tables remain queryable via the analytics layer but are excluded from hot index maintenance.

---

## Queue Workers

Async work is split across **named queues** with independent worker pools, so a slow report-export job can't starve real-time notification dispatch:

| Queue | Workload | Workers |
|-------|----------|---------|
| `default` | General async tasks | Auto-scaled |
| `notifications` | Email / SMS / push for ticket events | Dedicated, low-latency |
| `exports` | PDF + Excel generation | Dedicated, slow-and-fat |
| `audit` | Activity log persistence | Dedicated, durable |

Failed jobs are retained with full payload for replay; persistent failures alert on threshold breaches.

---

## Frontend Performance

- **Tailwind + Livewire + Alpine** keeps the hydration cost low. There is no SPA bootstrap — pages are server-rendered, then Livewire handles incremental updates over the wire.
- **Asset bundling** via Vite produces hashed, long-cached files served via Cloudflare's CDN.
- **TV board pages** are intentionally austere — minimal CSS, no images per ticket — so a low-power smart TV browser can run them indefinitely.

---

## Operational Limits Tested

| Scenario | Result |
|----------|--------|
| 50 simultaneous customer devices on one branch | Sub-second WebSocket update latency |
| 1,000 tickets / day per branch | Full feedback + analytics flow without degradation |
| Multi-branch hierarchy with 5 levels of RBAC | RBAC checks add < 5 ms per request via cached permission lookups |

Numbers above reflect internal load testing on the reference deployment topology described in [`deployment.md`](./deployment.md).

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
