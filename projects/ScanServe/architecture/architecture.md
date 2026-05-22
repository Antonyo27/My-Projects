# ScanServe — System Architecture

> A high-level architectural overview of the ScanServe queue management and feedback platform.
> Diagrams are written in Mermaid and render natively on GitHub.

---

## 1. System Overview

ScanServe follows a **multi-tenant monolithic architecture** built on the TALL stack (Tailwind, Alpine, Laravel, Livewire). Each onboarded organization receives an isolated subdomain (`{org}.domain.com`). A wildcard TLS certificate covers all tenant subdomains, so onboarding requires only DNS + database configuration — no per-tenant certificate provisioning.

Real-time features (TV queue boards, live ticket state) are powered by **Laravel Reverb**, a native WebSocket server, removing any dependency on third-party services like Pusher.

```mermaid
graph TB
    subgraph Clients
        C1[Customer<br/>Mobile Browser]
        C2[Staff<br/>Desktop/Tablet]
        C3[TV Queue Board<br/>Large Display]
        C4[Admin<br/>Desktop Browser]
        C5[Super Admin<br/>admin.domain.com]
    end

    subgraph Cloudflare
        CF[Cloudflare CDN / DNS<br/>DDoS Protection · Wildcard TLS]
    end

    subgraph VPS["Linux VPS (Ubuntu LTS)"]
        NGINX[Nginx Reverse Proxy<br/>Subdomain Routing]

        subgraph App["Laravel 12.x Application"]
            WEB[Web Routes<br/>Livewire Components]
            WS[Laravel Reverb<br/>WebSocket Server]
            Q[Queue Worker<br/>PDF · Excel · Notifications]
            SCHED[Laravel Scheduler<br/>Daily Stats · Cache Warm]
        end

        DB[(PostgreSQL)]
        CACHE[(Redis<br/>Cache · Queue Backing)]
    end

    C1 & C2 & C3 & C4 & C5 --> CF
    CF --> NGINX
    NGINX -->|HTTP/HTTPS| WEB
    NGINX -->|WSS| WS
    WEB <--> DB
    WEB <--> CACHE
    Q <--> DB
    WS <--> CACHE
```

---

## 2. Subdomain Routing & Tenant Isolation

```mermaid
graph LR
    A[domain.com] -->|Marketing & Org Directory| MARKETING[Public Site]
    B[admin.domain.com] -->|Super Admin Console| SUPERADMIN[Super Admin Panel]
    C[org-slug.domain.com] -->|Tenant App| TENANT[Organization Portal]

    TENANT --> B1[Customer QR Ticket Flow]
    TENANT --> B2[Staff Queue Console]
    TENANT --> B3[TV Queue Board]
    TENANT --> B4[Branch Admin Dashboard]
    TENANT --> B5[Feedback Submission]
```

---

## 3. Core User Roles & Operational Flow

```mermaid
flowchart TD
    subgraph Customer Journey
        QR[Customer Scans QR Code<br/>on TV Display or Kiosk]
        TICKET[Receives Digital Ticket<br/>on Mobile Browser]
        WAIT[Waits — Monitors Queue<br/>via Live TV Board]
        CALLED[Ticket Called<br/>TV Board Updates Instantly]
        SERVE[Proceeds to Service Window]
        FEEDBACK[Submits Feedback Survey<br/>via Static QR or SMS Link]
    end

    subgraph Staff Console
        CALL[Staff Calls Next Ticket]
        REDIRECT[Redirect / Transfer]
        RETURN[Issue Return Pass<br/>for Document Pickup]
        CLOSE[Mark Transaction Complete]
    end

    subgraph Admin Layer
        RESCUE[Unit Head Rescue Dashboard<br/>Missed · Zombie · Idle Slots]
        ANALYTICS[Analytics & Reporting<br/>Wait Times · Satisfaction · Staff KPIs]
        EXPORT[Export Impact Logs<br/>PDF · Excel]
    end

    QR --> TICKET --> WAIT --> CALLED --> SERVE
    SERVE --> FEEDBACK
    SERVE --> RETURN

    CALL --> CALLED
    SERVE --> CLOSE
    CLOSE --> ANALYTICS
    REDIRECT --> CALL

    RESCUE --> CALL
    ANALYTICS --> EXPORT
```

---

## 4. Real-Time Event Architecture

```mermaid
sequenceDiagram
    participant Staff as Staff Console
    participant Laravel as Laravel App
    participant Reverb as Laravel Reverb<br/>(WebSocket)
    participant Board as TV Queue Board
    participant Mobile as Customer Mobile

    Staff->>Laravel: Call Next Ticket (HTTP POST)
    Laravel->>Laravel: Update Transaction State
    Laravel->>Reverb: Broadcast TicketCalled Event
    Reverb-->>Board: Push — Board Re-renders Instantly
    Reverb-->>Mobile: Push — Customer Notified
    Board-->>Board: Display updates without page reload
```

---

## 5. Entity Relationship (High-Level)

```mermaid
erDiagram
    ORGANIZATION {
        uuid id PK
        string slug
        string name
        string subdomain
    }
    BRANCH {
        uuid id PK
        uuid organization_id FK
        string name
    }
    SERVICE_POINT {
        uuid id PK
        uuid branch_id FK
        string name
    }
    WINDOW {
        uuid id PK
        uuid service_point_id FK
        string label
    }
    TRANSACTION {
        uuid id PK
        uuid service_point_id FK
        uuid window_id FK
        string ticket_number
        enum status
        timestamp queued_at
        timestamp served_at
    }
    RETURN_PASS {
        uuid id PK
        uuid transaction_id FK
        string token
        timestamp expires_at
    }
    FEEDBACK {
        uuid id PK
        uuid transaction_id FK
        json responses
        int rating
    }
    FORM_TEMPLATE {
        uuid id PK
        uuid organization_id FK
        json schema
    }
    QR_CODE {
        uuid id PK
        uuid branch_id FK
        enum type
        string signed_url
    }
    USER {
        uuid id PK
        uuid organization_id FK
        string email
        enum role
    }

    ORGANIZATION ||--o{ BRANCH : "has"
    BRANCH ||--o{ SERVICE_POINT : "has"
    SERVICE_POINT ||--o{ WINDOW : "has"
    SERVICE_POINT ||--o{ TRANSACTION : "receives"
    TRANSACTION ||--o| WINDOW : "served at"
    TRANSACTION ||--o| RETURN_PASS : "may issue"
    TRANSACTION ||--o| FEEDBACK : "generates"
    ORGANIZATION ||--o{ FORM_TEMPLATE : "owns"
    BRANCH ||--o{ QR_CODE : "has"
    ORGANIZATION ||--o{ USER : "has"
```

---

## 6. Key Architectural Decisions

| Decision | Rationale |
|---|---|
| **TALL Stack (no decoupled SPA)** | Queue state is server-driven — Livewire keeps the hot path on the server where the source of truth lives. A separate REST API would add an unnecessary serialization layer. |
| **Subdomain-per-tenant** | Hard tenant isolation without separate deployments. Wildcard TLS handles all subdomains automatically. |
| **Laravel Reverb (self-hosted WebSockets)** | Eliminates SaaS dependency on Pusher. Real-time TV board updates require low-latency, persistent connections. |
| **Signed Dynamic QR URLs** | Short-lived signed URLs prevent ticket spoofing. TV displays rotate QR codes at configured intervals. |
| **Append-only Transaction Log** | Transactions are never deleted — soft states (cancelled, abandoned, zombie) are tracked for compliance and audit exports. |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · For portfolio demonstration only.</sub>
