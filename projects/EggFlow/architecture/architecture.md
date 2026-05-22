# EggFlow — System Architecture

> A high-level architectural overview of the EggFlow agribusiness management platform.
> Diagrams are written in Mermaid and render natively on GitHub.

---

## 1. System Overview

EggFlow is a **multi-tenant TALL stack application** (Tailwind, Alpine, Laravel, Livewire) purpose-built for agribusinesses managing egg production end-to-end — from daily farm collection through warehouse processing to retail sale. Each shop operates in strict data isolation, while upper management can aggregate across locations.

Real-time features (live chat, POS inventory sync) run through **Laravel Reverb** (self-hosted WebSockets). **Redis** backs both the cache layer (dashboard aggregates) and the async job queue (document generation, email delivery).

```mermaid
graph TB
    subgraph Users
        FARM[Farm Staff<br/>Mobile / Tablet]
        WH[Warehouse Staff<br/>Desktop]
        CASHIER[Cashier<br/>POS Terminal]
        MGR[Manager<br/>Dashboard]
        ADMIN[Admin / Owner<br/>Full Access]
        CUST[Customer<br/>Live Chat Widget]
    end

    subgraph Cloudflare
        CF[Cloudflare CDN / DNS<br/>DDoS Protection · TLS]
    end

    subgraph VPS["Linux VPS (Ubuntu LTS)"]
        NGINX[Nginx Reverse Proxy]

        subgraph App["Laravel 12.x Application"]
            WEB[Web Routes<br/>Livewire Components]
            WS[Laravel Reverb<br/>WebSocket Server]
            Q[Queue Workers<br/>PDF · Email · Reports]
            SCHED[Laravel Scheduler<br/>EOD Checks · Cache Warm]
        end

        DB[(PostgreSQL<br/>Primary Store)]
        CACHE[(Redis<br/>Cache + Queue)]
        MAIL[SendGrid<br/>Transactional Email]
    end

    FARM & WH & CASHIER & MGR & ADMIN & CUST --> CF
    CF --> NGINX
    NGINX -->|HTTPS| WEB
    NGINX -->|WSS| WS
    WEB <--> DB
    WEB <--> CACHE
    Q --> MAIL
    Q <--> DB
    WS <--> CACHE
```

---

## 2. Multi-Tenant Shop Isolation

```mermaid
graph LR
    PLATFORM[EggFlow Platform]

    PLATFORM --> SHOP1[Shop A<br/>Retail Location]
    PLATFORM --> SHOP2[Shop B<br/>Retail Location]
    PLATFORM --> FARM1[Farm Operations]
    PLATFORM --> WH1[Warehouse]

    SHOP1 --> S1DATA[(Shop A Data<br/>Isolated)]
    SHOP2 --> S2DATA[(Shop B Data<br/>Isolated)]
    FARM1 --> FARMDATA[(Farm Data)]
    WH1 --> WHDATA[(Warehouse Data)]

    MGR[Upper Management] -->|Aggregated View| PLATFORM
```

---

## 3. Core Operational Flow

```mermaid
flowchart TD
    subgraph Farm
        FC[Log Daily Egg Gathering<br/>Categorize by Size]
        FD[Create Delivery to Warehouse<br/>Log Breakage & Damage]
    end

    subgraph Warehouse
        WR[Receive Delivery<br/>Verify Quantities]
        WD[Log Discrepancies]
        WC[Update Inventory Stock]
    end

    subgraph POS
        PS[Process Sale<br/>Retail or Wholesale]
        PR[Check VIP Reservation<br/>Prevent Double-Booking]
        PD[Deduct from Live Inventory]
    end

    subgraph Shift
        SS[Staff Clock-In]
        SR[Shift Reconciliation<br/>Cash & Stock Variance]
        SE[EOD Report Generation]
    end

    subgraph Expenses
        EA[Submit Expense]
        EAP[Management Approval<br/>Workflow for High-Value]
        EF[Finalize & Log]
    end

    FC --> FD --> WR --> WD --> WC
    WC --> PS
    PR --> PS --> PD
    SS --> SR --> SE
    EA --> EAP --> EF
```

---

## 4. Real-Time Communications

```mermaid
sequenceDiagram
    participant Customer as Customer Browser
    participant Laravel as Laravel App
    participant Reverb as Laravel Reverb<br/>(WebSocket)
    participant Staff as Shop Staff Console

    Customer->>Reverb: Connect to Chat Channel
    Staff->>Reverb: Connect to Chat Channel
    Customer->>Laravel: Send Message (HTTP)
    Laravel->>Reverb: Broadcast MessageSent Event
    Reverb-->>Staff: Push — New Message Appears
    Staff->>Laravel: Reply (HTTP)
    Laravel->>Reverb: Broadcast MessageSent Event
    Reverb-->>Customer: Push — Reply Appears
```

---

## 5. Entity Relationship (High-Level)

```mermaid
erDiagram
    SHOP {
        uuid id PK
        string name
    }
    FARM {
        uuid id PK
        string name
    }
    DAILY_COLLECTION {
        uuid id PK
        uuid farm_id FK
        date collection_date
        int total_pieces
    }
    DELIVERY {
        uuid id PK
        uuid farm_id FK
        uuid shop_id FK
        enum status
    }
    INVENTORY {
        uuid id PK
        uuid shop_id FK
        uuid egg_category_id FK
        int quantity_pieces
    }
    SALE {
        uuid id PK
        uuid shop_id FK
        uuid shift_id FK
        decimal total_amount
    }
    SALE_ITEM {
        uuid id PK
        uuid sale_id FK
        uuid inventory_id FK
        int quantity
    }
    SHIFT {
        uuid id PK
        uuid shop_id FK
        uuid user_id FK
        timestamp started_at
        timestamp ended_at
    }
    RESERVATION {
        uuid id PK
        uuid customer_id FK
        uuid shop_id FK
        enum status
    }
    EXPENSE {
        uuid id PK
        uuid shop_id FK
        enum status
        decimal amount
    }
    WASTAGE_LOG {
        uuid id PK
        uuid shop_id FK
        int quantity_pieces
        enum stage
    }
    CUSTOMER {
        uuid id PK
        string name
        string contact
    }
    USER {
        uuid id PK
        uuid shop_id FK
        string email
        enum role
    }

    SHOP ||--o{ INVENTORY : "holds"
    SHOP ||--o{ SALE : "records"
    SHOP ||--o{ SHIFT : "runs"
    SHOP ||--o{ EXPENSE : "logs"
    SHOP ||--o{ WASTAGE_LOG : "tracks"
    FARM ||--o{ DAILY_COLLECTION : "logs"
    FARM ||--o{ DELIVERY : "dispatches"
    DELIVERY ||--o{ INVENTORY : "replenishes"
    SALE ||--o{ SALE_ITEM : "contains"
    SALE_ITEM }o--|| INVENTORY : "deducts"
    SHIFT ||--o{ SALE : "encompasses"
    RESERVATION }o--|| CUSTOMER : "belongs to"
    RESERVATION }o--|| SHOP : "at"
    USER }o--|| SHOP : "assigned to"
```

---

## 6. Key Architectural Decisions

| Decision | Rationale |
|---|---|
| **TALL Stack (no decoupled SPA)** | All critical state (inventory levels, shift state, approval status) lives server-side. Livewire eliminates a serialization boundary without sacrificing reactivity. |
| **Redis for Cache + Queue** | Dashboard aggregates are expensive to compute; Redis caches them between invalidations. The same Redis instance backs async job queues for PDF and email generation. |
| **Expense Approval Workflow** | High-value operational costs route through management before finalization, creating an auditable approval trail. |
| **Shift Reconciliation** | Staff clock-in establishes a baseline; on close, actual cash and stock are compared to expected values. Discrepancies are flagged automatically. |
| **VIP Reservation Sync** | Reservations lock inventory at a record level, preventing POS from double-booking reserved stock in real time. |
| **Audit Logging (OwenIt)** | Every state-changing action captures before/after values with actor attribution — essential for discrepancy investigation. |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · For portfolio demonstration only.</sub>
