# EggFlow — Scalability

How EggFlow scales from a single-shop operator to multi-location agribusinesses, and the trade-offs taken along the way.

---

## Scaling Dimensions

EggFlow scales along four independent axes:

| Axis | Example | Bottleneck |
|------|---------|------------|
| **Tenants (Shops)** | More businesses onboarded onto the platform | Database connection pool, RBAC lookup cost |
| **Locations per tenant** | More farms, warehouses, retail outlets per business | Inventory join complexity, dashboard aggregation cost |
| **Transaction volume** | Higher daily POS throughput | Database write contention, queue depth |
| **Historical data** | Years of yields, sales, audits | Read query latency, report generation time |

Each axis has a distinct mitigation strategy.

---

## Multi-Tenancy at Scale

EggFlow uses a **shared-database, shared-schema** multi-tenant model with strict per-shop scoping enforced at the Eloquent layer. Staff members are scoped to their assigned shop context; admins and upper management can aggregate across shops via dedicated, audited query paths.

The trade-off:

- **+** Onboarding a new shop is an `INSERT` — no migrations, no per-tenant infrastructure.
- **+** Cross-tenant analytics for the platform owner are trivial.
- **−** A single missing scope check would risk cross-tenant leaks. Mitigated by **default-deny** scoping: tenant-aware models *cannot* be queried without a tenant context, so a missing scope produces an immediate runtime error rather than silent data leakage.

---

## Inventory Reads vs. Writes

POS reads (live availability) and inventory writes (sales, transfers, receives) have very different scaling profiles:

- **Writes** — modest absolute throughput, but with strict consistency requirements. Handled directly against PostgreSQL with row-level locking on the location-SKU during the critical section.
- **Reads** — *very* high frequency on the POS shopfloor (every cashier polling availability per scan). Fronted by a short-TTL Redis cache invalidated on write, so a cashier sees freshly-correct availability without overwhelming the primary database.

Reservation logic deliberately bypasses the cache — VIP holds always read-and-write against the source of truth to prevent double-booking.

---

## Dashboard Aggregation

Executive dashboards are the most expensive read path in the system:

- Production comparisons across farms join across daily yields, flock metadata, and time windows.
- Inventory valuations across locations carry per-SKU cost basis through aggregation.
- Sales trends roll up across multiple POS endpoints with date filtering and weekly comparison.

Mitigations:

1. **Materialized aggregates** — daily roll-up jobs run via the scheduler write pre-aggregated rows to dedicated reporting tables. The dashboard reads from those, not from the raw transactional tables.
2. **Cache layer** — dashboard responses are cached at short TTLs keyed by shop + filter parameters. Cache invalidates on relevant writes.
3. **Date-window guards** — the UI prevents unbounded "all time" queries by default; users must opt into broader windows, which routes to the reporting tables instead of OLTP.

---

## Queue Workers

Async work is split across **named queues** with independent worker pools so that, e.g., a slow PDF report doesn't starve transactional email:

| Queue | Workload |
|-------|----------|
| `default` | General async tasks |
| `notifications` | Transactional email via SendGrid (shift reports, approvals, alerts) |
| `exports` | PDF + Excel generation |
| `audit` | Activity log persistence |
| `aggregations` | Scheduled dashboard rollups |

Failed jobs are retained with full payload for replay; persistent failures alert on threshold breaches.

---

## Frontend Performance at Distributed Locations

EggFlow's POS runs at retail locations that may have **flaky internet**. Mitigations:

- **Optimistic UI** — POS interactions reflect immediately, then reconcile with server confirmation.
- **Resilient submissions** — form submissions retry transparently across network blips.
- **Server-rendered Livewire** — minimal client-side bundle keeps cold loads quick on modest hardware.

The live chat widget and dashboard pages are intentionally tolerant of brief reconnects via Echo's built-in reconnect with exponential backoff.

---

## Operational Limits Tested

| Scenario | Result |
|----------|--------|
| Multi-location inventory transfers + simultaneous POS sales | No double-spend, no oversold reservations under load testing |
| Daily executive dashboard for a multi-farm operator | Sub-second response from cached aggregates |
| Shift reconciliation across overlapping shifts and multiple staff | Discrepancies attributed correctly with no false positives in test scenarios |

Numbers reflect internal load testing on the reference deployment topology described in [`deployment.md`](./deployment.md).

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
