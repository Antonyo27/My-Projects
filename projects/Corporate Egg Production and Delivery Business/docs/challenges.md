# Corporate Egg Production and Delivery Business — Engineering Challenges

A selection of the harder problems encountered while building this system and how they were solved.

---

## 1. Designing Blind Receiving as a First-Class Workflow

**The Problem.** Conceptually, blind receiving is simple: don't show the receiver what was dispatched until after they've counted. Operationally, it's tricky — the system needs to *know* the dispatched quantity (to compute the discrepancy) but must reliably *hide* it until the receiver commits.

**The Solution.** Receiving was modeled as a **two-phase commit**. Phase one: the receiver enters their physical count and categorization. The dispatch quantity is server-side only and never serialized to the receiver's view. Phase two: only after submission does the system render a side-by-side reconciliation showing dispatch vs. receive, with the discrepancy highlighted and persisted as part of the warehouse activity log. The receiver's count is **immutable** once committed — fixing a "miscount" requires an explicit, audited adjustment, not a silent edit.

---

## 2. Multi-Location Dispatch and Receive State

**The Problem.** Inventory in motion (dispatched but not yet received) is a real, accountable state — not a placeholder. If a dispatch leaves Farm A but never arrives at Warehouse B, that loss has to be visible and attributable. Naively decrementing farm inventory and waiting for the warehouse to increment its inventory loses the in-transit state entirely.

**The Solution.** Dispatch and receive are modeled as **paired event records** with explicit `dispatched`, `in_transit`, and `received` states. Inventory in transit is its own balance, attributable to the dispatching farm until the warehouse commits the receive. A dispatch that's been in-transit beyond a configurable threshold is surfaced on the management dashboard as an investigable anomaly.

---

## 3. PDF Dispatch Receipts That Tie the Whole Chain Together

**The Problem.** A printed dispatch receipt is the physical artifact that travels with the goods. It needs to be unambiguously linked to the dispatch event in the system, hard to forge, and easy for warehouse staff to scan-and-verify on arrival.

**The Solution.** Dispatch receipts are generated server-side via **`barryvdh/laravel-dompdf`** with an embedded **QR code** (via `simplesoftwareio/simple-qrcode`) encoding the dispatch event ID. Warehouse staff can verify the receipt during receiving by scanning the code, and the system can immediately confirm the dispatch is genuine and unreceived. The PDF is a re-renderable artifact, not a stored binary, so it can be regenerated on demand without sync drift.

---

## 4. Hierarchical RBAC Across Farm, Warehouse, and Management

**The Problem.** A role-based system that's too coarse leaves staff with access they shouldn't have. Too fine, and the role list explodes into unmaintainable permission combinations.

**The Solution.** A **hierarchical role architecture** mapped directly to the organizational structure: Owner / Superadmin at the top, with Farm Operations and Warehouse Management as separate functional tiers, each scoped to their physical location. Staff in the farm module cannot see warehouse internals; warehouse staff cannot retroactively edit dispatch records. Cross-cutting concerns (reports, dashboards) are gated to management roles. The role definitions live in code and migrate with the rest of the schema, so role changes are versioned, not ad-hoc.

---

## 5. Reports That Owners Will Actually Use

**The Problem.** Owners don't read 40-page PDFs. They want to know, at a glance, which farm is over- or under-performing, where the wastage is, and whether anything actionable changed since yesterday.

**The Solution.** Two-tier reporting:

- **Executive dashboard** — live, interactive, **Chart.js**-powered visualizations comparing farms, surfacing mortality and breakage trends, and flagging anomalies. This is the daily-use surface.
- **Formal reports** — PDF and Excel exports for compliance, accounting, and external stakeholders. Excel exports use **`openspout/openspout`** for fast, memory-efficient streaming so large historical exports don't OOM the worker.

The dashboard is the artifact owners actually consume; the formal reports back it up when needed.

---

## 6. Importing Legacy Data

**The Problem.** Going live with no historical data would have left the dashboards empty for weeks before trends emerged. But importing years of paper-based logs cleanly is its own problem — the data is messy, inconsistent, and partially missing.

**The Solution.** A **staged import pipeline** with explicit validation rules: rows that pass cleanly are imported; rows with recoverable issues are flagged for review with a suggested correction; rows that can't be reconciled are quarantined for manual decision rather than silently dropped or guessed. The import is **idempotent** so it can be re-run as data quality improves.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
