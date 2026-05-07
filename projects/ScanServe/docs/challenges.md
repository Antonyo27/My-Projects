# ScanServe — Engineering Challenges

A selection of the harder problems encountered while building ScanServe and how they were solved.

---

## 1. Real-Time Synchronization Across Heterogeneous Clients

**The Problem.** A single queue event (e.g., "ticket REG-007 called") needed to propagate **simultaneously** to:

- The customer's mobile browser (live ticket page)
- One or more TV display boards in the branch
- The staff admin console
- Unit Head rescue dashboards

Traditional polling would have produced visible lag and put unnecessary load on the database.

**The Solution.** A WebSocket broadcast layer powered by **Laravel Reverb** with **Laravel Echo** on the client. Each branch subscribes to a private channel scoped by tenant + branch ID, so events only reach authorized clients. Livewire components rebroadcast their own state changes via Echo, making real-time sync feel "automatic" from the developer's perspective.

**Trade-off.** Reverb requires a long-lived process — handled with a dedicated Supervisor-managed worker on the deployment host (see [`deployment.md`](./deployment.md)).

---

## 2. Secure QR Code Routing Without User Accounts

**The Problem.** Customers should be able to join a queue **without creating an account or installing an app** — but anonymous access can't mean exploitable. A naive public URL would let anyone spam queue creation, scrape tenant data, or hijack tickets.

**The Solution.** A two-tier QR strategy:

- **Static QRs** (printed, anchored at service points) carry a tenant + service-point identifier and route to a hardened intake form rate-limited per IP.
- **Dynamic QRs** (broadcast on TV boards) embed **short-lived signed URLs** generated server-side. They expire within minutes, preventing screen-capture replay attacks.

Tickets themselves are signed tokens stored in the customer's browser, so even without auth, a stolen ticket URL can't be reassigned to a different customer.

---

## 3. Multi-Tenant Data Isolation Without Per-Tenant Databases

**The Problem.** Spinning up a separate database per organization is operationally expensive and complicates backups, migrations, and analytics. But shared-DB multi-tenancy risks **cross-tenant data leaks** if a single missing scope check escapes review.

**The Solution.** A single shared database with **enforced tenant scoping at the Eloquent layer**. Every tenant-aware model uses a global scope that injects `WHERE organization_id = ?` automatically, derived from the resolved subdomain (`{org_slug}.scanserve.app`). Subdomain resolution happens in middleware before the controller stack runs, so by the time queries execute, the tenant context is already locked in. Bypassing the scope requires explicit, auditable opt-in via a named macro.

---

## 4. "Zombie" Tickets and Queue Drift

**The Problem.** Real-world queues drift. Customers leave without telling anyone. Staff forget to mark a transaction complete. Tickets get stuck in `serving` for hours, blocking the window's call-next button and corrupting wait-time analytics.

**The Solution.** A **rescue dashboard** for Unit Heads showing every ticket whose state has been stale beyond a configured threshold. Heuristics flag candidates: "called > X minutes ago with no serve event," "serving > Y minutes with no completion," etc. Heads can bulk-resolve as no-show, redirect, or escalate. A scheduled job sweeps unresolved zombies after a longer timeout to keep analytics clean.

---

## 5. Priority Return Pass — Trustless Re-Entry

**The Problem.** Customers often need to leave the office mid-process (lunch, retrieve a missing document, return another day for pickup) and return without losing their priority. Issuing a paper slip is fragile; relying on staff memory is worse.

**The Solution.** A **signed return pass token** issued at the moment of pause. The token encodes the original ticket lineage, the priority class, and an expiration. On return, the customer scans the pass; the system verifies the signature, checks expiration, and re-injects them at the correct priority slot — without requiring staff to remember anything.

---

## 6. Feedback That Actually Gets Submitted

**The Problem.** Most feedback systems email customers a survey hours after their visit. Response rates are abysmal because the experience is no longer fresh.

**The Solution.** Feedback is delivered **in-flow**, on the same device the customer used to track their ticket, the moment their transaction is marked complete. The feedback form is contextually pre-bound to the specific transaction, processing window, and staff member — no manual selection required. Surveys can also be opened post-visit via a **static feedback QR** at the service point, scoped to the most recent transaction in that customer's session.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
