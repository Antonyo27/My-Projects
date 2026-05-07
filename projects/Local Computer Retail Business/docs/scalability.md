# Local Computer Retail Business — Scalability

How the system is designed to scale within the realistic envelope of a single-shop retail and repair business, and the trade-offs taken along the way.

---

## Right-Sizing

This system was deliberately architected to fit a **single-shop operator** rather than a multi-location enterprise. Over-engineering for hypothetical scale would have meant longer delivery, higher hosting costs, and a more complex codebase the client doesn't need.

That said, the architectural choices keep the door open to grow.

---

## Scaling Dimensions

| Axis | Realistic Range | Approach |
|------|-----------------|----------|
| **Concurrent users** | A handful of cashiers + technicians + the owner | Single Node process under PM2 cluster mode is more than sufficient |
| **Daily transactions** | Hundreds, not thousands | Indexed Prisma queries handle this comfortably |
| **Inventory SKUs** | Low thousands | PostgreSQL indexes on category, brand, and model keep search snappy |
| **Service tickets** | Hundreds open at any time, thousands historically | Status-indexed plus customer-indexed queries make lookup constant-time |
| **Historical data** | Years of receipts and service logs | Reporting queries are date-windowed; old data partitions are queryable but excluded from hot index maintenance |

---

## Frontend Performance

- **Vite** produces hashed, long-cached production bundles served as static assets via Nginx.
- **Zustand** keeps client state lean — no heavyweight state library overhead.
- **React Router v6** lazy-loads route bundles so the POS view loads in seconds even on modest hardware.
- **Sonner** toasts replace blocking modals for non-critical feedback, keeping the cashier flow uninterrupted.

---

## Backend Performance

- **Prisma** generates type-safe, prepared queries — no string-concatenated SQL, no N+1 surprises beyond the ones you write yourself.
- **PostgreSQL indexes** are tuned to the actual hot paths: SKU lookup, customer service history, daily sales rollup.
- **PM2 cluster mode** scales the API horizontally across CPU cores on the host without code changes.

---

## What Would Change at 10×

If the client expanded to multiple branches or added a wholesale arm, the architectural pivots required would be:

1. **Multi-tenancy** — add a tenant scope to every model and route, similar to ScanServe's pattern.
2. **Cache layer** — introduce Redis for hot reads (SKU availability, dashboard aggregates).
3. **Read replica** — push reporting queries to a replica to keep OLTP unaffected.
4. **Background queue** — move heavy report generation and Excel exports off the request thread.

None of these would require rewriting the system; they're additive evolutions of the current architecture.

---

## What Would *Not* Change

- The **Zod-driven type contract** between frontend and backend scales with the team, not just the load.
- The **state-machine model** for service tickets remains the right abstraction at any volume.
- The **RBAC model** (Admin/Cashier/Staff) extends naturally with new roles for multi-branch managers.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
