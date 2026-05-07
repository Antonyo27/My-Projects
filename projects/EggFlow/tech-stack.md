# EggFlow — Tech Stack

A complete breakdown of the technologies powering EggFlow, with rationale for each choice.

---

## Stack Family: TALL

EggFlow is built on the **TALL Stack** (Tailwind, Alpine, Laravel, Livewire), giving it a **server-rendered, single-page-application feel** without maintaining a decoupled frontend.

> **Why TALL?** EggFlow's UX is dominated by *server-driven state* — inventory levels, shift state, approval workflow status, dashboard aggregates. Livewire keeps the hot path on the server where the source of truth already lives, eliminating an entire serialization boundary between frontend and backend.

---

## Backend Infrastructure

| Layer | Technology | Why |
|-------|------------|-----|
| **Core Framework** | Laravel 12.x (PHP 8.2+) | Mature ecosystem, first-class real-time + queue + scheduling primitives |
| **Database** | PostgreSQL | Strong transactional guarantees for inventory writes; rich query capabilities for analytics |
| **Cache & Queue Backing** | Redis | Low-latency cache for dashboards + reliable backing for async job queues |
| **Authentication** | Laravel Fortify & Laravel Sanctum | Standard, audited auth primitives — no custom rolled-our-own auth |
| **Authorization** | Spatie Laravel Permission | Battle-tested RBAC with role + permission abstractions |
| **Audit Logging** | OwenIt Laravel Auditing | Captures every state-changing action with before/after values and actor attribution |
| **API Documentation** | Dedoc Scramble | Auto-generated OpenAPI specs from existing route + validation code |
| **Document Generation** | `barryvdh/laravel-dompdf` | Server-side PDF for shift reports, approval docs, and exports |
| **Email Delivery** | SendGrid | Production-grade deliverability + webhook callbacks for delivery state |

## Frontend Architecture

| Layer | Technology | Why |
|-------|------------|-----|
| **UI & Server Reactivity** | Livewire 3.7+ | Server-driven components, no client-state duplication |
| **Client-Side Interactivity** | Alpine.js 3.15+ | Lightweight client polish (modals, dropdowns, transitions) |
| **CSS Framework** | Tailwind CSS 4.1 | Utility-first, design-system-friendly, tiny production bundles |
| **Data Visualization** | ApexCharts 5.3 | Interactive, responsive charts for executive dashboards |
| **Build Pipeline** | Vite 7.0 | Fast HMR in development, hashed production builds |

## Real-Time Communications

| Layer | Technology | Why |
|-------|------------|-----|
| **WebSocket Engine** | Laravel Reverb | Native WebSocket server, no third-party SaaS dependency |
| **Client Broadcasting** | Laravel Echo | Standard client API for WebSocket subscriptions |

The real-time layer powers the **floating live chat widget** connecting customers directly to shop staff and surfaces real-time inventory updates to the POS.

## Infrastructure & DevOps

| Layer | Technology |
|-------|------------|
| **Containerization** | Docker + Docker Compose |
| **Reverse Proxy** | Nginx |
| **TLS** | Let's Encrypt (auto-renewing) |
| **CDN / DNS** | Cloudflare |
| **CI/CD** | GitHub Actions |
| **Process Supervision** | systemd / Supervisor |
| **Hosting** | Linux VPS (Ubuntu LTS) |

## Testing & Quality

| Layer | Tool |
|-------|------|
| **PHP Test Runner** | Pest / PHPUnit |
| **PHP Linting** | Laravel Pint |
| **JS Linting** | ESLint |
| **Manual QA** | Cross-browser smoke tests on POS, dashboard, and chat widget before each release |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](./README.md)</sub>
