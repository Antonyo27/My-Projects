# ScanServe — Tech Stack

A complete breakdown of the technologies powering ScanServe, with rationale for each choice.

---

## Stack Family: TALL

ScanServe is built on the **TALL Stack** (Tailwind, Alpine, Laravel, Livewire), giving it a **server-rendered, single-page-application feel** without the overhead of maintaining a decoupled frontend.

> **Why TALL?** A decoupled SPA + REST API would have doubled the codebase, doubled the deployment surface, and added an unnecessary serialization boundary — all for a system whose UX is dominated by *server-driven state* (tickets called, queues advanced, feedback submitted). Livewire keeps the hot path on the server, where the source of truth already lives.

---

## Backend Infrastructure

| Layer | Technology | Why |
|-------|------------|-----|
| **Core Framework** | Laravel 12.x (PHP 8.2+) | Mature ecosystem, first-class real-time + queue + scheduling primitives |
| **Real-Time** | Laravel Reverb | Native WebSocket server, no external service dependency |
| **Database** | PostgreSQL / MySQL | Database-agnostic via Eloquent ORM — clients can choose |
| **PDF Generation** | `barryvdh/laravel-dompdf` | Server-side PDF for impact logs, compliance reports, audit exports |
| **Excel Exports** | `maatwebsite/excel` | High-performance spreadsheet generation for large analytics exports |
| **Audit Logging** | `spatie/laravel-activitylog` | Structured, queryable activity trails for compliance |
| **QR Code Generation** | `simplesoftwareio/simple-qrcode` | Static and dynamic signed QR rendering |

## Frontend Architecture

| Layer | Technology | Why |
|-------|------------|-----|
| **UI Framework** | Tailwind CSS 3.x | Utility-first, design-system-friendly, tiny production bundles |
| **Reactivity & Components** | Laravel Livewire 4.x | Server-driven components, no client-state duplication |
| **DOM Interactions** | Alpine.js 3.x | Lightweight client-side polish (modals, dropdowns, transitions) |
| **Event Broadcasting** | Laravel Echo + Pusher.js protocol | Standard client API for WebSocket subscriptions |
| **Drag & Drop** | SortableJS | Reordering windows, services, queue priorities |
| **Asset Bundler** | Vite | Fast HMR in development, hashed production builds |

## Network & Routing Design

| Layer | Approach | Why |
|-------|----------|-----|
| **Apex domain** | Marketing site, public org directory, global search | Single entry point for SEO and onboarding |
| **`admin.*` subdomain** | Super Admin console | Hard isolation from tenant traffic |
| **`{org_slug}.*` subdomains** | Per-organization portal (the "digital lobby") | Strict tenant separation with shared infrastructure |

A wildcard TLS certificate covers every tenant subdomain so onboarding requires only DNS + database work — no certificate provisioning per tenant.

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
| **Manual QA** | Multi-browser smoke tests on TV board, mobile customer view, and admin console before each release |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](./README.md)</sub>
