# EggFlow — Engineering Challenges

A selection of the harder problems encountered while building EggFlow and how they were solved.

---

## 1. Reconciling Inventory Across Simultaneous Locations

**The Problem.** Stock was being moved between farm, warehouse, and retail simultaneously by different staff. Naive inventory updates produced **race conditions**: a sale could complete against stock that had just been transferred out, or two cashiers could oversell the last carton in a VIP reservation.

**The Solution.** Inventory writes are **transactional and row-locked at the location-SKU level** during the critical section. Reservations are first-class entities — when a VIP places a hold, available stock is decremented immediately into a reserved bucket, and the cashier UI exposes only *unreserved* available stock at the POS. Transfers between locations are modeled as **dispatch + receive event pairs**, never as a single mutation, so a discrepancy at receive time is logged rather than silently overwriting source-of-truth.

---

## 2. Shift-End Discrepancy Calculation

**The Problem.** Manual shift reconciliation was error-prone and delayed. By the time discrepancies surfaced (often days later), the staff member had moved on and accountability was impossible.

**The Solution.** Every shift opens with a **system-recorded baseline** of cash drawer + stock on hand. Every transaction during the shift is attributed to the on-duty staff. At shift close, the system computes the **expected end state** from the baseline plus the day's transactions and compares it to the **counted end state** entered by the staff member. Discrepancies are flagged immediately, attributed automatically, and surfaced on the manager dashboard before the staff member leaves the premises.

---

## 3. High-Value Expense Approval Without Becoming a Bureaucracy

**The Problem.** Adding approval workflows tends to create friction that staff route around — petty cash gets logged, but the actually-significant expenses end up on a manager's desk as a paper receipt. Solving for accountability without crippling daily operations was tricky.

**The Solution.** **Threshold-based routing**. Operational costs below a configurable threshold post immediately and surface in routine reports. Costs above the threshold enter an **approval queue** with full context (vendor, category, justification, supporting documents) and notify approvers instantly. Approvers can act from a phone in seconds. The threshold is per-shop and per-category so feed costs don't fight for attention with administrative supplies.

---

## 4. Yield Categorization at the Edge

**The Problem.** Yields are recorded by farm staff who may have intermittent connectivity and who absolutely cannot afford friction during peak collection windows. A slow form means missed entries, which means missing data forever.

**The Solution.** The yield entry interface is **mobile-first, large-tap-target, and resilient to slow connections**. Categories are pre-configured per farm, so staff tap-tap-submit rather than typing. Submissions are queued client-side if connectivity drops and synced automatically when restored, with conflict resolution favoring the most recent server timestamp.

---

## 5. Real-Time Customer Chat Without a Third-Party Service

**The Problem.** Retail customers wanted instant support but the business didn't want to depend on (or pay for) a third-party live chat SaaS — and didn't want customer conversations leaving the system.

**The Solution.** A **Reverb-backed live chat widget** running on the same WebSocket infrastructure as the rest of the platform. Conversations are persisted alongside customer records, fully searchable by staff, and subject to the same RBAC as the rest of the system. No external dependencies, no per-seat pricing, no data leaving the platform.

---

## 6. Executive Dashboards That Don't Lie

**The Problem.** Aggregating data across farms, warehouses, and POS locations produced dashboards that were *technically correct* but *operationally misleading* — averaging across farms with very different scales, or summing inventory at locations using different unit costs.

**The Solution.** Every aggregation in the executive layer is **explicitly weighted and unit-aware**. Production comparisons normalize by flock size or capacity. Inventory valuations carry their cost basis through the aggregation, so the rolled-up number reflects actual financial value rather than naive count × latest-price. The dashboard surfaces the underlying scale alongside the headline number, so a manager can never mistake a small farm's spike for a multi-farm trend.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
