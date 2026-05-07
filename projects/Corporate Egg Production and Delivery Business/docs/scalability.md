# Corporate Egg Production and Delivery Business — Scalability

How the system is designed to scale within and beyond the realistic envelope of a corporate-scale poultry operator, and the trade-offs taken along the way.

---

## Scaling Dimensions

| Axis | Example | Approach |
|------|---------|----------|
| **Locations** | Multiple farms, multiple warehouses, multiple dispatch points | Centralized application with RBAC + location scoping at the application layer |
| **Concurrent staff** | Farm + warehouse + management users active simultaneously | PHP-FPM concurrency tuned to host capacity; Livewire keeps round-trips small |
| **Daily throughput** | High-volume egg collection events, dispatches, receives, breakage logs | Indexed PostgreSQL queries with bounded date windows |
| **Historical data** | Years of yields, dispatches, audits | Reporting tables and date-windowed aggregation queries |

---

## Centralized Architecture, Distributed Users

The deliberate design choice was to deploy **one centralized application** that all locations connect to, rather than per-location deployments. The trade-offs:

- **+** Single source of truth across the entire business — no inter-location sync layer needed
- **+** Cross-farm comparisons in the executive dashboard are trivial (it's all one database)
- **+** Deployments and migrations happen once, not per-location
- **−** A network outage at the central host affects every location. Mitigated via offsite backups, supervised restart, and a deployment surface designed for fast restore

For a corporate operator with a stable network footprint, the centralized model is the right call. A truly multi-region operator with isolated network islands would require a different topology — documented as a known evolution path.

---

## Database Strategy

- **Hot transactional path** — egg collection, dispatches, receives, breakage logs go straight to PostgreSQL with proper indexing on the natural lookup keys (location, date, flock, dispatch event ID).
- **Reporting path** — executive dashboards query against **rolled-up tables** populated by scheduled jobs, not against raw transactional tables.
- **Excel exports** — large historical exports stream via `openspout/openspout` so they never load the full result set into memory.
- **Long-tail data** — historical records older than the active reporting window are still queryable but excluded from hot index maintenance.

---

## Document Generation at Scale

PDF dispatch receipts and Excel reports are CPU-bound work that, if run inline, would block the request thread. Mitigations:

- **Queue workers** handle all document generation asynchronously — the user sees a "preparing" state and is notified when ready.
- **Streamed Excel exports** via `openspout/openspout` keep memory bounded regardless of row count.
- **Re-renderable PDFs** are generated on demand rather than stored, so document storage doesn't grow unboundedly with operational activity.

---

## RBAC Performance

A hierarchical role system gets queried on **every** request. Naïvely loading roles + permissions on every page load would balloon DB traffic. Mitigations:

- Permission lookups are **cached per user session** with explicit invalidation on role changes.
- Eloquent eager-loading prevents N+1 explosions on RBAC-heavy admin pages.

---

## What Would Change at 10× Volume

If the operator added more farms, expanded into wholesale, or started serving multiple corporate operators on the same platform:

1. **Multi-tenant scoping** — add a tenant boundary above the existing location boundary (similar to ScanServe).
2. **Read replica** — push reporting queries to a replica to keep OLTP unaffected.
3. **Horizontal app servers** — scale PHP-FPM across multiple hosts behind a load balancer; the app is already stateless except for sessions.
4. **Cache layer** — introduce Redis for hot dashboard reads with explicit invalidation on consequential writes.

None of these would require rewriting the system — they're additive evolutions of the current architecture.

---

## What Would *Not* Change

- The **blind-receiving discipline** is a domain-modeling choice that scales without modification — it's about *what* the system enforces, not *how* it's deployed.
- The **paired dispatch / receive event model** scales linearly with throughput.
- The **role-based location scoping** model extends naturally with new locations.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
