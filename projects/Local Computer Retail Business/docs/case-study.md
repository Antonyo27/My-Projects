# Local Computer Retail Business — Case Study

> **POS, Inventory, and Service-Log System for a Computer Retail & Repair Shop**
> *From paper logs to a typed, audited digital platform.*

---

> **Confidentiality Notice.** Specific client identity, branding, internal business rules, pricing structures, and proprietary workflows have been intentionally omitted. This case study is published with the client's understanding that no confidential information is disclosed.

---

## The Challenge

A local computer retail and repair business was operating with **paper-based records and disconnected spreadsheets** for everything that mattered:

- **Inventory** was counted manually, and stock levels were never the same in two places
- **Service tickets** (repairs, diagnostics, consultations) lived in handwritten logbooks — searching the history of a returning customer was a manual archaeology dig
- **Sales** were tallied at end-of-day from receipt stubs, with frequent reconciliation errors
- **Reporting** to the owner required hours of weekly compilation, and even then the numbers were stale by the time they were reviewed
- **No accountability** — staff actions were untraceable, and recurring discrepancies couldn't be attributed

The result was a business running on the experience of its long-tenured staff rather than on data — fine until something went wrong, but a serious risk to growth and continuity.

## The Solution

A custom-built **POS, inventory, and service-logging web application** that consolidated every operational concern into one platform.

### Point of Sale

A fast, focused checkout flow optimized for daily retail throughput. Receipts generate automatically, stock deducts in real time, and end-of-day reconciliation becomes a single screen rather than a stack of paper.

### Inventory Management

Real-time stock visibility with **product variations** (different SKUs, brands, models) and **low-inventory alerts**. The owner sees what's selling, what's stuck, and what needs to be reordered — without manually counting anything.

### Service Logging

A dedicated module for **repairs, technical consultations, troubleshooting sessions, and customer service tickets**. Every service has a status, a customer history, technician notes, and a clear lifecycle from intake to delivery. Returning customers are recognized instantly with their full prior service history.

### Reporting & Analytics

Sales, inventory, and service reports surface live in dashboards. **Excel exports** give the owner data to take to their accountant or to make purchasing decisions, without anyone manually compiling spreadsheets.

### Role-Based Access Control

Three roles (Admin, Cashier, Staff) with distinct capabilities. Cashiers can sell. Staff can log services. Only Admins can edit prices, adjust inventory, or void transactions. Every meaningful action is **logged with the responsible user attached**.

---

## Outcome

The system has been **live in production** at the client site, running daily POS, service intake, and end-of-day reporting workflows. Concrete operational improvements observed include:

- **Sales reconciliation** went from a multi-hour daily activity to a single end-of-day screen.
- **Service-ticket lookup** for returning customers is now instant — staff no longer dig through logbooks.
- **Stock visibility** is real-time, eliminating the "do we have this in stock?" pause that previously cost sales.
- **Audit trails** mean recurring discrepancies (cash, stock, voids) can finally be attributed and addressed.
- **Reports** that previously required weekly manual compilation are now available on demand.

---

## Architecture at a Glance

The system uses a **decoupled architecture** — React + TypeScript on the frontend, Express + TypeScript on the backend — connected through a strictly typed REST API. **Prisma** manages PostgreSQL access; **Zod** validates every incoming payload before it hits the database. **Sentry** captures production errors; **Pino** structures application logs; **PM2** keeps the Node process supervised in production.

See [`tech-stack.md`](../tech-stack.md) for the full breakdown.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · See [LICENSE](../../../LICENSE)</sub>
