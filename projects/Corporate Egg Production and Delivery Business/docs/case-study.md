# Corporate Egg Production and Delivery Business — Case Study

> **Poultry Farm Operations Monitoring & Reconciliation System**
> *From manual spreadsheets to an audited validation platform.*

---

> **Confidentiality Notice.** Specific client identity, branding, internal business rules, pricing structures, and proprietary workflows have been intentionally omitted. This case study is published with the client's understanding that no confidential information is disclosed. All screenshots in this study have been sanitized and blurred to protect sensitive data.

---

## The Challenge

A corporate-scale egg poultry operator was running their agribusiness operations — tracking daily collections, sorting batches, and monitoring flock health — on **manual logbooks and disconnected spreadsheets**.

The pain points compounded daily:
- **Egg collection** was logged on paper at the farm and re-keyed (often incorrectly) into spreadsheets later, leading to mismatches.
- **Flock management** records lived in different places for different staff — health metrics and mortality rates rarely lined up.
- **Warehouse sorting** suffered from an inventory integrity problem: there was no validation mechanism to compare the total trays collected at the farm against the final trays sorted by size at the warehouse.
- **Wastage and breakages** were hidden under loose reporting, making it impossible to attribute missing inventory.
- **Owners and management** had no real-time view across their business dates — comparison was a weekly compilation exercise, by which time the data was already stale.

The business needed a system that not only digitized these workflows but also **enforced ledger balancing and anomaly checking** in real time.

## The Solution

A **centralized web application** acting as an owner-side reconciliation dashboard, covering the full workflow from farm collection to daily ledger reports.

### Reconciliation Dashboard & Workflow Checklist
An operational hub showing daily progress (Receiving, Categorization, Balance checks, Mortality logging) and active high-severity flags for any selected business date.

### Daily Receivings & Categorizations
A streamlined data entry system that allows the owner to log total trays received from the farm and categorize them by grade (Pewee, Pullets, Small, Medium, Large, Extra Large, Jumbo) plus Cracked and Disposed trays. Forms use a **Save-State pattern** designed to prevent data loss on weak rural connections by displaying explicit `Saving... / Saved / Failed - Retry` states.

### Flag Engine & Checkpoint A Validation
The core audit checkpoint of the system:
- **Checkpoint A Validation** automatically converts physical trays to accounting eggs (using snapshotted constants) and verifies that delivered eggs match categorized + cracked + disposed eggs.
- Any discrepancy instantly triggers a **Delivery Count Variance Flag** (detecting missing or excess eggs) or an **Excessive Disposal Rate Flag** (when unsellable eggs exceed the configurable threshold).
- Flags freeze their calculation state to protect historical audit records from future configuration changes.

### Daily Reconciliation Report
A print-friendly daily summary breaking down total yields, category shares, mortality logs, and active flags.

---

## Outcome

The system has been **adopted into active production operations** across the client's agribusiness. Concrete operational improvements observed include:
- **End-to-end traceability** from farm collection through warehouse receiving, sorting, and disposal tracking.
- **Ledger integrity** — the Checkpoint A validation immediately highlights count mismatches, turning previously invisible errors into investigable events.
- **Real-time owner visibility** — what used to be a weekly manual report is now a live daily dashboard.
- **Reduced administrative overhead** — data operators enter logs directly, and the system handles all conversion math and validation rules.
- **Audit-ready history** — every operational day has a clean, frozen record of yields, mortality, and resolved or open flags.

---

## Architecture at a Glance

The platform is built with **Laravel 13** for the backend, **Inertia.js** for the API-free view layer, **Vue 3** for the reactive frontend, and **Tailwind CSS (v4)** for the styling system. **Chart.js** powers the dashboard visualizations. The system runs on a centralized Linux server and is fully containerized.

See [`tech-stack.md`](../tech-stack.md) for the full breakdown.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · See [LICENSE](../../../LICENSE)</sub>
