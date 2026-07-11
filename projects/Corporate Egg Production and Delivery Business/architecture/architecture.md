# Corporate Egg Production & Delivery Business — System Architecture

> A high-level architectural overview of the poultry farm operations monitoring and reconciliation system.
> Client identity, branding, and proprietary business rules are intentionally omitted.
> Diagrams render natively on GitHub via Mermaid.

---

## 1. System Overview

This system is a **centralized web application** built on the **Laravel + Inertia.js + Vue 3** stack. The application acts as a single-tenant administrative dashboard. All operators connect to the same central database, ensuring real-time balancing of farm deliveries and sorting metrics, with built-in resilience for weak rural internet connections.

```mermaid
graph TB
    subgraph Users
        OWNER[Owners / Operators<br/>Mobile · Tablet · Desktop]
    end

    subgraph Infra["Hosted Infrastructure"]
        NGINX[Nginx Reverse Proxy<br/>TLS Termination]

        subgraph App["Laravel Application (PHP 8.3)"]
            WEB[Web Routes<br/>Inertia.js + Vue 3 Pages]
            SCHED[Scheduler<br/>Active Flag Scans]
        end

        DB[(PostgreSQL<br/>Primary Datastore)]
    end

    OWNER --> NGINX
    NGINX --> WEB
    WEB <--> DB
```

---

## 2. Operational Modules

Since all authenticated accounts share administrative reconciliation access, the system modules are organized around the daily audit workflow:

```mermaid
graph LR
    subgraph Modules
        DASH[Dashboard<br/>Workflow & Flags]
        REC[Daily Receivings<br/>Farm Trays Input]
        CAT[Categorizations<br/>Warehouse Sizes sorting]
        MORT[Mortality Log<br/>Flock Health counts]
        FLAGS[Flag Engine<br/>Discrepancy validation]
        REP[Reports<br/>Daily Summary & CSV exports]
        SET[Settings<br/>Thresholds & Egg sizes]
    end

    DASH --> REC & CAT & MORT & FLAGS & REP & SET
```

---

## 3. Core Operational Flow — Farm to Reconciliation

The daily workflow uses the **Asia/Manila business date** as its primary scope. Data operators log daily farm collections and warehouse sorting counts. The system then automatically converts trays to eggs and reconciles the counts via **Checkpoint A**:

```mermaid
flowchart TD
    subgraph Farm Operations
        A[Farm Staff Report Daily Collections<br/>to Owner via Messenger]
    end

    subgraph Owner Actions
        B[Log Daily Receivings<br/>in Trays]
        C[Log Categorizations<br/>per Size/Grade]
        D[Log Daily Mortality<br/>counts]
    end

    subgraph Validation Engine
        E[Convert Trays to Eggs<br/>using Settings constant]
        F[Run Checkpoint A<br/>compare Receivings vs. Sorted]
        G{Discrepancy Found?}
        H[Trigger Count Variance Flag<br/>or Disposal Rate Flag]
        I[Clean Daily Status]
    end

    subgraph Analytics
        J[View Dashboard Summary]
        K[Export Monthly CSV Reports]
    end

    A --> B --> C --> D
    C --> E --> F --> G
    G -- Yes --> H --> J
    G -- No --> I --> J
    J --> K
```

---

## 4. Save-State Form Architecture

To prevent data loss on weak rural internet connections, all data-entry forms implement a **Save-State design pattern**. Operations are executed through Inertia.js with explicit visual states:

```mermaid
sequenceDiagram
    participant User as Operator
    participant Vue as SaveStateForm (Vue 3)
    participant Server as Laravel Controller
    participant DB as PostgreSQL

    User->>Vue: Type input & click Save
    Vue->>Vue: Set state: "Saving..."
    Vue->>Server: Send POST/PUT via Inertia
    alt Connection Success
        Server->>DB: Write/Update records
        DB-->>Server: Done
        Server-->>Vue: Return Redirect (Success)
        Vue->>Vue: Set state: "Saved" (Green badge)
    else Connection Loss / Database Error
        Server--xVue: Request Timeout / Failure
        Vue->>Vue: Set state: "Failed" (Red alert)
        Vue-->>User: Enable "Retry" button
    end
```

---

## 5. Entity Relationship Diagram (ERD)

The database schema is structured for strict validation and data integrity:

```mermaid
erDiagram
    users {
        bigint id PK
        string name
        string email
        string password
        timestamp email_verified_at
        timestamp created_at
        timestamp updated_at
    }
    egg_sizes {
        bigint id PK
        string slug
        string name
        int display_order
        boolean is_disposal
        timestamp created_at
        timestamp updated_at
    }
    settings {
        bigint id PK
        string key UNIQUE
        string value
        string label
        string group
        string type
        timestamp created_at
        timestamp updated_at
    }
    daily_receivings {
        bigint id PK
        date date UNIQUE
        int received_trays
        int eggs_per_tray_snapshot
        text source_notes
        bigint user_id FK
        timestamp deleted_at
        timestamp created_at
        timestamp updated_at
    }
    mortality_logs {
        bigint id PK
        date date UNIQUE
        int count
        text notes
        bigint user_id FK
        timestamp deleted_at
        timestamp created_at
        timestamp updated_at
    }
    categorizations {
        bigint id PK
        date date
        bigint egg_size_id FK
        int trays
        int eggs_per_tray_snapshot
        bigint user_id FK
        timestamp deleted_at
        timestamp created_at
        timestamp updated_at
    }
    flags {
        bigint id PK
        date date
        string type
        string severity
        string status
        string flagable_type
        bigint flagable_id
        text message
        decimal impact_quantity
        string impact_unit
        json calculation_snapshot
        json metadata
        text investigation_notes
        bigint resolved_by FK
        timestamp resolved_at
        timestamp superseded_at
        bigint user_id FK
        timestamp created_at
        timestamp updated_at
    }

    users ||--o{ daily_receivings : "creates"
    users ||--o{ mortality_logs : "creates"
    users ||--o{ categorizations : "creates"
    users ||--o{ flags : "creates/resolves"
    egg_sizes ||--o{ categorizations : "categorizes into"
```

---

## 6. Key Architectural Decisions

| Decision | Rationale |
|---|---|
| **Centralized single-tenant deployment** | Deployed as a single tenant serving trusted operators. Avoids tenant complexity while maintaining high performance. |
| **Asia/Manila business date** | All operational records use the Manila business calendar date, not `created_at`. This aligns the digital ledger with real-world farm collection periods. |
| **Unit conversion at the egg level** | The user inputs all counts in physical trays (whole units). The system converts trays → eggs internally using `eggs_per_tray` to allow fractional losses and disposal sorting. |
| **Settings snapshotting** | To ensure data integrity, settings parameters like `eggs_per_tray_snapshot` are copied directly to the `daily_receivings` and `categorizations` tables at the moment of saving. |
| **Frozen Flag Snapshots** | Active flags freeze the variables and thresholds used during calculation inside `flags.calculation_snapshot`. This prevents old flags from changing their values if settings thresholds are adjusted later. |
| **Check constraints** | PostgreSQL table constraints enforce that quantities like `received_trays`, `trays` (sorting), and `count` (mortality) are strictly non-negative. |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · Provided for portfolio demonstration only. No confidential client information is disclosed.</sub>
