# Corporate Egg Production and Delivery Business — Tech Stack

A complete breakdown of the technologies powering this system, with rationale for each choice.

---

## Backend

| Layer | Technology | Why |
|-------|------------|-----|
| **Language** | PHP 8.3 | Modern PHP with strict types, enums, and attributes — production-grade language semantics |
| **Framework** | Laravel 13 | Secure, mature framework with robust session management, validation pipelines, and routing |
| **Database** | PostgreSQL | Enterprise relational database with transactional guarantees for inventory reconciliation writes |

## Frontend

| Layer | Technology | Why |
|-------|------------|-----|
| **Framework** | Inertia.js | Builds single-page apps without client-side API routing. Keeps the controller as the single source of truth |
| **View Layer** | Vue 3 | Component-driven reactive UI. Perfect for dense forms, status checks, and data management |
| **Styling** | Tailwind CSS (v4.x) | Utility-first CSS, compiling into high-performance styles with instant page loading |
| **Build Tool** | Vite | Ultra-fast Hot Module Replacement (HMR) and optimized, hashed production assets |
| **Data Visualization** | Chart.js | Renders responsive metrics, mortality logs, and trend analyses directly on the dashboard |

## Key Libraries & Integrations

| Layer | Technology | Why |
|-------|------------|-----|
| **Route Sharing** | `tightenco/ziggy` | Shares Laravel backend routes directly with Vue components for clean, standard API communication |

## Architecture Highlights

- **Single-Tenant Application** — Deployed as a single tenant where authenticated logins represent trusted operators with equal system credentials.
- **Two Reconciliation Checkpoints** — Silently performs dual audit loops:
  - **Checkpoint A (Upstream)**: Checks if Daily Receivings match warehouse categorizations.
  - **Checkpoint B (Downstream)**: Focuses on tracking flock health, mortality rates, and overall yield balances.
- **Rural-Internet Resilience (Save-State Form Pattern)** — Reusable frontend forms with explicit `Saving... / Saved / Failed - Retry` feedback loops to guarantee no data loss on unstable rural connections.
- **Snapshotted Calculations** — To prevent historical data drift when pricing or conversion configurations change, all settings constants (like `eggs_per_tray`) are copied directly onto operational logs during creation.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](./README.md)</sub>
