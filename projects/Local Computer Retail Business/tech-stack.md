# Local Computer Retail Business — Tech Stack

A complete breakdown of the technologies powering this system, with rationale for each choice.

---

## Frontend

| Layer | Technology | Why |
|-------|------------|-----|
| **Framework** | React 18 with Vite | Modern, mature, fast HMR for tight iteration |
| **Language** | TypeScript | Compile-time safety across UI components |
| **Styling** | Tailwind CSS | Utility-first, design-system-friendly, tiny production bundles |
| **State Management** | Zustand | Minimal boilerplate, excellent for the modest amount of client state this app needs |
| **Routing** | React Router v6 | Standard, lazy-loadable route bundles |
| **Icons & Toasts** | Lucide React, Sonner | Lightweight, accessible UI primitives |
| **Error Tracking** | Sentry | Production error visibility for a small-business client without an ops team |

## Backend

| Layer | Technology | Why |
|-------|------------|-----|
| **Runtime** | Node.js | Single-language stack with the frontend simplifies the codebase |
| **Framework** | Express.js | Minimal, predictable, extensively documented |
| **Language** | TypeScript | Compile-time safety on the server, shared types with the frontend |
| **ORM** | Prisma | Generated types, migrations, and prepared queries — eliminates entire classes of SQL bugs |
| **Database** | PostgreSQL (production) / SQLite (development) | Real concurrency in prod; zero-friction local dev |
| **Authentication** | JSON Web Tokens (JWT) + bcryptjs | Standard, audited primitives — short-lived access tokens with refresh rotation |
| **Validation** | Zod | Single source of truth for request shape; types inferred for both backend and frontend |
| **Logging** | Pino | High-performance structured JSON logging |
| **File Handling & Export** | Multer, ExcelJS | Receipt uploads + streaming Excel exports |
| **Security** | Helmet, Express Rate Limit, CORS | Defence-in-depth against common HTTP vulnerabilities |

## Infrastructure & DevOps

| Layer | Technology |
|-------|------------|
| **Process Supervisor** | PM2 (cluster mode) |
| **Reverse Proxy** | Nginx |
| **TLS** | Let's Encrypt |
| **DNS** | Cloudflare |

## Architecture Highlights

- **Decoupled frontend / backend** with a strictly typed REST API.
- **End-to-end type safety**: Zod schemas drive backend validation and infer TypeScript types consumed by the frontend.
- **Prisma-managed schema** with migration history kept in version control.
- **PM2 cluster mode** in production for resilience and zero-downtime reload.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](./README.md)</sub>
