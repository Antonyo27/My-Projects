# Local Computer Retail Business — System Architecture

> A high-level architectural overview of the POS, inventory, and service-log platform.
> Client identity, branding, and proprietary business details are intentionally omitted.
> Diagrams render natively on GitHub via Mermaid.

---

## 1. System Overview

This system uses a **decoupled architecture** — a React SPA frontend communicates with a strictly typed Express/Node.js REST API backend. TypeScript is used end-to-end on both the client and server, with **Zod schemas** as the single source of truth for request validation, inferring TypeScript types consumed by the frontend. Prisma manages the database schema and generates type-safe query clients.

```mermaid
graph TB
    subgraph Clients
        OWNER[Owner / Superadmin<br/>Desktop & Mobile Browser]
        STAFF[Shop Assistant<br/>Desktop Browser]
    end

    subgraph Infra["Hosted Infrastructure (VPS)"]
        CF[Cloudflare DNS / TLS]
        NGINX[Nginx Reverse Proxy]

        subgraph Frontend["React SPA (Vite)"]
            UI[React 18 + TypeScript<br/>Zustand State · React Router]
        end

        subgraph Backend["Express API (Node.js)"]
            API[Express.js + TypeScript<br/>Zod Validation · JWT Auth]
            PM2[PM2 Cluster Mode<br/>Process Supervision]
        end

        DB[(PostgreSQL<br/>Production Database)]
    end

    OWNER & STAFF --> CF
    CF --> NGINX
    NGINX -->|Static Files| Frontend
    NGINX -->|/api/*| Backend
    API <--> DB
    PM2 -.-|supervises| API
```

---

## 2. Authentication & Session Architecture

The system implements a **dual-mode authentication** strategy: standard email/password login for first access, and a **4-digit PIN with a rolling 72-hour session** for fast repeat access — optimized for the owner's on-the-go mobile usage.

```mermaid
flowchart TD
    START([User Opens App])

    START --> CHECK{Valid PIN<br/>Session?}

    CHECK -- Yes --> PIN_SCREEN[PIN Entry Screen<br/>Touch-Optimized Numpad]
    CHECK -- No --> LOGIN[Email + Password<br/>Login Screen]

    PIN_SCREEN --> PIN_VALID{PIN Correct<br/>& Not Expired?}
    PIN_VALID -- Yes --> DASHBOARD[Dashboard]
    PIN_VALID -- No / Expired --> LOGIN

    LOGIN --> AUTH{Credentials<br/>Valid?}
    AUTH -- No --> LOGIN
    AUTH -- Yes --> SETUP{PIN<br/>Session Set Up?}

    SETUP -- No --> PROMPT[Prompt: Set Up PIN]
    SETUP -- Yes --> DASHBOARD
    PROMPT --> DASHBOARD

    DASHBOARD --> ACTIVITY[Any Interaction]
    ACTIVITY --> REFRESH[Rolling 72hr<br/>Expiry Refreshed]
    REFRESH --> ACTIVITY
```

---

## 3. Module Structure & Role Access

```mermaid
graph LR
    subgraph Roles
        SUPERADMIN[Superadmin<br/>Business Owner]
        ASSISTANT[Assistant<br/>Shop Staff]
    end

    subgraph Modules
        DASH_SA[Dashboard<br/>Business-Wide Metrics]
        DASH_AS[Dashboard<br/>Own Activity Only]
        INV[Inventory<br/>Folder-Explorer · CRUD · Import/Export]
        POS[POS<br/>Service Sales · Item Sales]
        REPORTS[Reports Module<br/>Superadmin Only]
    end

    SUPERADMIN --> DASH_SA & INV & POS & REPORTS
    ASSISTANT --> DASH_AS & INV & POS
```

---

## 4. Core Data Flows

### 4a. Item Sale Flow

```mermaid
sequenceDiagram
    participant Cashier as Staff / Owner
    participant Frontend as React Frontend
    participant API as Express API
    participant DB as PostgreSQL

    Cashier->>Frontend: Select Item from Inventory
    Frontend->>API: GET /api/inventory/:id
    API->>DB: Fetch item (condition=New/Used, qty > 0)
    DB-->>API: Item with current price snapshot
    API-->>Frontend: Item details
    Cashier->>Frontend: Enter Quantity & Confirm Sale
    Frontend->>API: POST /api/sales (item type)
    API->>DB: BEGIN TRANSACTION
    API->>DB: INSERT sale record (price snapshot)
    API->>DB: UPDATE item quantity - sold_qty
    API->>DB: INSERT inventory_log (stock-out)
    API->>DB: COMMIT
    DB-->>API: Success
    API-->>Frontend: Sale confirmed
    Frontend-->>Cashier: Dashboard metrics updated
```

### 4b. Historical Inventory Calculation

```mermaid
flowchart LR
    A[Select Past Date on Dashboard]
    B[Query inventory_logs<br/>up to that date]
    C[Replay stock-in and stock-out<br/>events forward from item creation]
    D[Compute stock level<br/>at that point in time]
    E[Display: Inventory Count<br/>as of selected date]

    A --> B --> C --> D --> E
```

---

## 5. Entity Relationship (High-Level)

```mermaid
erDiagram
    USER {
        uuid id PK
        string email
        string password_hash
        enum role
        string pin_hash
        timestamp pin_expires_at
    }
    CATEGORY {
        uuid id PK
        uuid parent_id FK
        string name
        uuid created_by FK
    }
    ITEM {
        uuid id PK
        uuid category_id FK
        string name
        int quantity
        decimal selling_price
        enum condition
        uuid created_by FK
    }
    SALE {
        uuid id PK
        enum type
        uuid item_id FK
        int quantity_sold
        decimal unit_price_at_sale
        decimal total_amount
        string service_description
        boolean is_consultation
        uuid recorded_by FK
        date sale_date
    }
    INVENTORY_LOG {
        uuid id PK
        uuid item_id FK
        enum change_type
        int quantity_changed
        int quantity_before
        int quantity_after
        uuid performed_by FK
        timestamp created_at
    }
    ACTIVITY_LOG {
        uuid id PK
        uuid user_id FK
        string action_type
        string entity_type
        uuid entity_id
        timestamp created_at
    }

    USER ||--o{ CATEGORY : "creates"
    CATEGORY ||--o{ CATEGORY : "parent of (2-level max)"
    CATEGORY ||--o{ ITEM : "contains"
    ITEM ||--o{ SALE : "sold via"
    ITEM ||--o{ INVENTORY_LOG : "tracked by"
    USER ||--o{ SALE : "records"
    USER ||--o{ ACTIVITY_LOG : "generates"
```

---

## 6. End-to-End Type Safety

A key engineering decision is using **Zod as the single source of type truth** across the entire stack. This eliminates an entire class of runtime type mismatch bugs between the frontend and backend.

```mermaid
graph LR
    ZOD[Zod Schema<br/>e.g. CreateSaleSchema]
    BE_VALID[Express Route<br/>Validates Request Body]
    BE_TYPES[TypeScript Types<br/>Inferred on Server]
    FE_TYPES[TypeScript Types<br/>Inferred on Client]
    PRISMA[Prisma ORM<br/>Type-Safe DB Queries]

    ZOD --> BE_VALID
    ZOD --> BE_TYPES
    ZOD --> FE_TYPES
    BE_TYPES --> PRISMA
```

---

## 7. Key Architectural Decisions

| Decision | Rationale |
|---|---|
| **Decoupled React SPA + REST API** | The owner accesses the system primarily from a mobile browser on-the-go. A separate frontend allows the React layer to be fully optimized for responsive, touch-first interactions independent of server rendering. |
| **End-to-end TypeScript + Zod** | Zod schemas validate incoming requests and simultaneously infer TypeScript types used on both server and client — one definition, zero duplication. |
| **Ownership-based permissions** | Every record is stamped with `created_by`. Assistants can only edit/delete their own records. The UI hides action controls for records the user doesn't own before the API enforces the same rule server-side. |
| **Price snapshot on sale** | At the moment of sale, the item's current `selling_price` is captured into the sale record. Future price changes don't alter historical sales data. |
| **Inventory log replay for history** | Rather than storing daily snapshots, all stock changes are logged. Historical inventory counts are derived by replaying these logs forward to any target date. |
| **PM2 cluster mode** | Node.js is single-threaded; PM2 spawns multiple worker processes to utilize all available CPU cores and enables zero-downtime restarts. |
| **Rolling 72-hour PIN session** | The owner frequently checks sales from a mobile device. A PIN removes the friction of re-entering credentials while the rolling window ensures the session expires after genuine inactivity. |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · Provided for portfolio demonstration only. No confidential client information is disclosed.</sub>
