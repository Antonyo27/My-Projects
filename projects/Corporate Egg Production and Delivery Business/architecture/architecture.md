# Corporate Egg Production & Delivery Business — System Architecture

> A high-level architectural overview of the poultry farm management and distribution system.
> Client identity, branding, and proprietary business rules are intentionally omitted.
> Diagrams render natively on GitHub via Mermaid.

---

## 1. System Overview

This system is a **centralized, multi-location web application** built on the TALL stack (Tailwind, Alpine, Laravel, Livewire). Rather than deploying separate instances per location, a single application instance serves all physical sites — farms, warehouse, and sales points — with access scoped strictly by user role and location assignment at the application layer.

```mermaid
graph TB
    subgraph Users
        FARM[Farm Staff<br/>Mobile / Tablet]
        WH[Warehouse Staff<br/>Desktop]
        MGR[Management<br/>Dashboard & Reports]
        OWNER[Owner / Superadmin<br/>Full System Access]
    end

    subgraph Infra["Hosted Infrastructure"]
        NGINX[Nginx Reverse Proxy<br/>TLS Termination]

        subgraph App["Laravel Application (PHP 8.3)"]
            WEB[Web Routes<br/>Livewire Components]
            Q[Queue Workers<br/>PDF · Excel Report Generation]
        end

        DB[(PostgreSQL<br/>Primary Datastore)]
    end

    FARM & WH & MGR & OWNER --> NGINX
    NGINX --> WEB
    WEB <--> DB
    Q <--> DB
```

---

## 2. Role-Based Access & Module Visibility

Each operational tier interacts only with the modules relevant to their responsibilities. Access is enforced at both the routing layer and the application policy layer.

```mermaid
graph LR
    subgraph Roles
        OWNER[Owner / Superadmin]
        ADMIN[Manager / Admin]
        FSTAFF[Farm Staff]
        WSTAFF[Warehouse Staff]
        CASHIER[Cashier]
    end

    subgraph Modules
        EXEC[Executive Dashboard<br/>& Analytics]
        USERMGMT[User Management]
        SETTINGS[System Configuration<br/>Pricing · Category Rules]
        REPORTS[Report Generation<br/>PDF · Excel]
        FLOCK[Flock & Batch Management]
        COLLECTION[Egg Collection Logging]
        DISPATCH[Dispatch Operations]
        RECEIVING[Warehouse Receiving]
        CATEGORIZE[Categorization & Sorting]
        BREAKAGE[Breakage Logging]
        POS[Point of Sale]
    end

    OWNER --> EXEC & USERMGMT & SETTINGS & REPORTS & FLOCK & COLLECTION & DISPATCH & RECEIVING & CATEGORIZE & BREAKAGE & POS
    ADMIN --> EXEC & REPORTS & FLOCK & COLLECTION & DISPATCH & RECEIVING & CATEGORIZE & BREAKAGE
    FSTAFF --> COLLECTION & DISPATCH
    WSTAFF --> RECEIVING & CATEGORIZE & BREAKAGE
    CASHIER --> POS
```

---

## 3. Core Operational Flow — Farm to Warehouse

The system enforces a **paired dispatch / receive model** with explicit inventory state transitions. A key control: warehouse staff perform **blind receiving** — they physically count and categorize an incoming delivery without prior knowledge of the dispatched quantity, ensuring maximum inventory accuracy.

```mermaid
flowchart TD
    subgraph Farm Operations
        A[Log Daily Egg Collection<br/>Per Flock / Batch]
        B[Record Flock Health<br/>& Mortality]
        C[Generate Dispatch Receipt<br/>QR Code Embedded]
        D[Mark Shipment In Transit]
    end

    subgraph Warehouse Operations
        E[Scan Dispatch Receipt QR]
        F[Blind Physical Count<br/>No Quantity Visible Yet]
        G[Commit Received Quantities<br/>System Reveals Dispatched Qty]
        H[Auto-Detect Discrepancies]
        I[Categorize by Grade/Size]
        J[Log Breakage]
        K[Inventory Updated]
    end

    subgraph Reporting
        L[Owner Views Dashboard<br/>Aggregated Multi-Location]
        M[Export PDF / Excel Reports<br/>Automated via Queue Jobs]
    end

    A --> C --> D --> E --> F --> G --> H --> I --> J --> K
    B --> A
    K --> L --> M
```

---

## 4. Document Generation Architecture

Dispatch receipts and operational reports are generated **server-side via queue jobs** — no client-side document assembly. This keeps generation off the request cycle and allows large multi-year exports without timeout risk.

```mermaid
sequenceDiagram
    participant User as Staff / Owner
    participant Livewire as Livewire Component
    participant Queue as Queue Worker
    participant PDF as dompdf / OpenSpout
    participant Storage as File Storage

    User->>Livewire: Request PDF/Excel Export
    Livewire->>Queue: Dispatch GenerateReport Job
    Livewire-->>User: "Report is being generated..."
    Queue->>PDF: Compile Template + Data
    PDF->>Storage: Save Generated File
    Queue-->>User: Notify — Download Ready
    User->>Storage: Download File
```

---

## 5. Entity Relationship (High-Level)

```mermaid
erDiagram
    BATCH {
        uuid id PK
        string batch_code
        enum status
        date started_at
    }
    BATCH_CATEGORY {
        uuid id PK
        uuid batch_id FK
        string category_label
        int quantity_pieces
    }
    INVENTORY_TRANSACTION {
        uuid id PK
        uuid batch_category_id FK
        enum type
        int quantity_pieces
        timestamp created_at
    }
    DISPATCH {
        uuid id PK
        uuid batch_id FK
        uuid farm_location_id FK
        enum status
        string qr_code_token
    }
    DISPATCH_ITEM {
        uuid id PK
        uuid dispatch_id FK
        string category_label
        int quantity_pieces
    }
    SALE {
        uuid id PK
        uuid location_id FK
        uuid user_id FK
        decimal total_amount
    }
    SALE_ITEM {
        uuid id PK
        uuid sale_id FK
        uuid batch_category_id FK
        int quantity_pieces
    }
    PRICE {
        uuid id PK
        string category_label
        int unit_price_centavos
        date effective_from
        date effective_until
    }
    FLAG {
        uuid id PK
        uuid transaction_id FK
        string details
        enum status
    }
    USER {
        uuid id PK
        string email
        enum role
        uuid location_id FK
    }

    BATCH ||--o{ BATCH_CATEGORY : "categorized into"
    BATCH_CATEGORY ||--o{ INVENTORY_TRANSACTION : "tracks via"
    BATCH ||--o{ DISPATCH : "dispatched as"
    DISPATCH ||--o{ DISPATCH_ITEM : "contains"
    SALE ||--o{ SALE_ITEM : "contains"
    SALE_ITEM }o--|| BATCH_CATEGORY : "draws from"
    INVENTORY_TRANSACTION ||--o| FLAG : "may flag"
    USER }o--|| SALE : "records"
```

---

## 6. Key Architectural Decisions

| Decision | Rationale |
|---|---|
| **Centralized single deployment** | All locations share one application instance with role + location scoping enforced at the application layer — no per-site deployments to maintain. |
| **Blind receiving discipline** | Server hides dispatched quantities until warehouse staff commit their physical count. Eliminates confirmation bias and ensures true independent verification. |
| **Append-only transactional tables** | Core transactional records (`BatchCategory`, `InventoryTransaction`, `SaleItem`) are immutable after creation. Corrections are made via new compensating records, not edits — preserving a complete audit trail. |
| **Integer centavos for money** | All monetary values stored as integers (centavos). No floating-point arithmetic near financial data. |
| **Integer pieces for quantities** | All egg quantities stored as integer pieces. Display layer converts to tray + piece format as needed. |
| **UUID primary keys** | All domain tables use UUID v4 PKs, preventing enumerable ID attacks and supporting future distributed scenarios. |
| **Queue-dispatched document generation** | PDF and Excel generation is always handled by background workers — never on the HTTP request cycle. |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · Provided for portfolio demonstration only. No confidential client information is disclosed.</sub>
