# Corporate Egg Production and Delivery Business — Tech Stack

A complete breakdown of the technologies powering this system, with rationale for each choice.

---

## Backend

| Layer | Technology | Why |
|-------|------------|-----|
| **Language** | PHP 8.3 | Modern PHP with strict types, enums, and attributes — production-grade language semantics |
| **Framework** | Laravel | Mature ecosystem with first-class queue, scheduler, and PDF/export integrations |
| **Database** | PostgreSQL | Strong transactional guarantees for inventory writes; rich query capabilities for analytics |

## Frontend

| Layer | Technology | Why |
|-------|------------|-----|
| **Reactive UI** | Livewire (v4.x) | Server-driven components — keeps the source of truth on the server where it belongs |
| **Styling** | Tailwind CSS (v4.x) | Utility-first, design-system-friendly, tiny production bundles |
| **Build Tool** | Vite | Fast HMR in development, hashed production builds |
| **Data Visualization** | Chart.js | Interactive executive dashboards — production comparisons, mortality trends, breakage analytics |

## Key Libraries & Integrations

| Layer | Technology | Why |
|-------|------------|-----|
| **PDF Generation** | `barryvdh/laravel-dompdf` | Server-side PDF for **dispatch receipts** and **operational reports** |
| **Excel Import / Export** | `openspout/openspout` | **Memory-efficient streaming** — critical for multi-year historical exports that would otherwise OOM |
| **QR Code Generation** | `simplesoftwareio/simple-qrcode` | Embedded codes on dispatch receipts for **scan-and-verify** at warehouse receiving |

## Infrastructure & DevOps

| Layer | Technology |
|-------|------------|
| **Containerization** | Docker + Docker Compose |
| **Reverse Proxy** | Nginx |
| **TLS** | Let's Encrypt |

## Architecture Highlights

- **Centralized application** with RBAC + location scoping enforced at the application layer (not per-location deployments).
- **Server-side document generation** via Livewire-triggered queue jobs — no client-side document assembly.
- **Paired dispatch / receive event model** with explicit `dispatched`, `in_transit`, and `received` states for full inventory traceability.
- **Blind-receiving discipline** — server-side data hiding prevents receivers from seeing dispatch quantities until after they commit their physical count.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](./README.md)</sub>
