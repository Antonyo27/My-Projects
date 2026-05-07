# Local Computer Retail Business — Engineering Challenges

A selection of the harder problems encountered while building this system and how they were solved.

---

## 1. End-to-End Type Safety Across the Network Boundary

**The Problem.** A decoupled architecture (React frontend, Express backend) introduces a serialization boundary where types can drift silently. A backend response shape change can break the frontend in production with no compile-time warning.

**The Solution.** **Zod schemas as the single source of truth** for every request and response. The backend validates incoming payloads with Zod and infers TypeScript types from those same schemas. The frontend imports the inferred types so a schema change ripples through the entire codebase as a compile error rather than a runtime crash. Combined with Prisma's generated types for the database layer, the system has unbroken type-safety from the database through the API into the UI.

---

## 2. Inventory Deductions Without Race Conditions

**The Problem.** Two cashiers ringing up the same low-stock item at the same time could oversell. Naive `SELECT` then `UPDATE` is racy.

**The Solution.** Inventory writes happen inside **Prisma transactions** with row-level locking, so the second checkout either reads the already-decremented value or is rolled back if stock is exhausted. The UI surfaces the conflict immediately rather than silently overselling.

---

## 3. Service Tickets That Span Days or Weeks

**The Problem.** Computer repairs aren't ten-minute transactions — a service ticket may be opened today, parts ordered tomorrow, technician work done a few days later, and pickup happen a week out. Modeling that as a sale was wrong.

**The Solution.** Service tickets are first-class entities with a **state machine**: `intake → diagnosed → awaiting-parts → in-repair → ready-for-pickup → closed`. Each transition is timestamped and attributed, so the customer's full history is visible at a glance and the technician's actions are accountable.

---

## 4. Authentication Hardening Without UX Friction

**The Problem.** A staff-facing app needs strong auth — but cashiers can't be locked out by overly-aggressive rules during a customer transaction.

**The Solution.** **JWT** with short-lived access tokens and refresh rotation, plus **bcrypt** password hashing. Helmet sets security headers; rate limits guard the login endpoint specifically (not the entire app). Sessions are revocable from the admin side if a staff member leaves.

---

## 5. Searchable, Exportable Reports Without Killing the Database

**The Problem.** Owner-facing reports could span months of sales × multiple categories × multiple staff — naive queries would lock the OLTP path during business hours.

**The Solution.** Reports run against **indexed projection queries** with date windowing and pagination. Excel exports run through **ExcelJS** as streamed responses so a multi-thousand-row report doesn't buffer in memory. Heavy report runs are scheduled outside business hours when possible.

---

## 6. Production Error Visibility

**The Problem.** A small business client doesn't have a dedicated ops team — when something breaks, you find out from a phone call, not a metrics dashboard. That's too slow.

**The Solution.** **Sentry** captures unhandled exceptions on both frontend and backend with full request context. **Pino** writes structured JSON logs that can be tailed during incident response. **PM2** keeps the Node process supervised and restarts on crash. Errors surface to me before the client even notices.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
