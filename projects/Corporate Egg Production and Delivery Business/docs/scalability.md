# Corporate Egg Production and Delivery Business — Scalability

How the system is designed to scale within and beyond the realistic envelope of a corporate-scale poultry operator, and the trade-offs taken along the way.

---

## Scaling Dimensions

| Axis | Example | Approach |
|---|---|---|
| **Daily throughput** | High-volume egg collections, daily receivings, categorizations, and mortality logs | Indexed PostgreSQL queries with bounded date windows |
| **Historical data** | Years of daily reconciliation records and flags | Database indexing on date columns and snapshot fields |

---

## Centralized Architecture, Distributed Users

The deliberate design choice was to deploy **one centralized application** that the business owner and operators connect to. The trade-offs:

- **+** Single source of truth across the entire business — no inter-device sync layer needed
- **+** Cross-date comparisons on the dashboard are trivial (it's all one database)
- **+** Deployments and migrations happen once, not on local devices
- **−** A network outage at the host affects all operators. Mitigated via offsite backups, supervised process restart, and a deployment surface designed for fast restore

---

## Database Strategy

- **Hot transactional path** — daily receivings, categorizations, and mortality logs go straight to PostgreSQL with proper indexing on the lookup keys (date, user_id, flag type).
- **Date-Window query bounding** — since the system operates day-by-day, all dashboard and report queries are strictly bounded by date. This prevents performance degradation as the database grows, as queries never perform full table scans.
- **Settings Snapshotting** — snapshotting constants like `eggs_per_tray` onto operational records removes the need to join or look up historic configuration values during report renders, speeding up query execution times.
- **Fast Integer Math** — all egg quantities are stored as integers (pieces), and money as integer cents/centavos. This avoids floating-point operations at the database layer and enables fast, precise arithmetic.

---

## What Would Change at 10× Volume

If the operator expanded to multiple separate poultry businesses, added wholesale distributions, or scaled transaction volume:

1. **Multi-tenant scoping** — add a tenant boundary above the existing user boundary (e.g., introducing an `organizations` table) to support multiple corporate operators on the same codebase.
2. **Read replicas** — redirect analytical report queries and historical exports to a read replica to keep write operations on the primary database completely unaffected.
3. **Redis caching** — cache static data like egg sizes and system settings, and use Redis to store today's active dashboard summaries with write-through invalidation.
4. **Horizontal application servers** — scale PHP-FPM and Vite across multiple load-balanced web servers; the application is stateless except for standard session data.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
