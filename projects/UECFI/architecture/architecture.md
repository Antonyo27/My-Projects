# UECFI — System Architecture

> A high-level architectural overview of the UECFI mobile hymnal application.
> Diagrams are written in Mermaid and render natively on GitHub.

---

## 1. System Topology

UECFI is an offline-first mobile application. The master song database resides in **Firebase Firestore**, while the active catalog is loaded and queried from local **SQLite** databases on the user's mobile device.

```mermaid
graph TB
    subgraph MobileDevice["User Mobile Device (Android / iOS / Web)"]
        subgraph App["React Native Expo Application"]
            UI[User Interface<br/>Zustand State]
            TRANS[Transposition Parser<br/>In-Memory Shift]
            SYNC[Sync Manager<br/>NetInfo status]
        end

        SQL[(SQLite Database<br/>Local Cache)]
    end

    subgraph FirebaseCloud["Firebase Cloud Infrastructure"]
        AUTH[Firebase Authentication]
        FS[(Firestore Database<br/>Master Song Catalog)]
    end

    UI <--> SQL
    TRANS <--> UI
    SYNC <--> SQL
    SYNC <-->|HTTPS Sync| FS
    UI -->|Admin Sign In| AUTH
```

---

## 2. Catalog Synchronization Flow

This diagram illustrates how catalog updates are synced from the cloud, and how modifications flow from administrators back up to Firestore:

```mermaid
sequenceDiagram
    participant Admin as Admin Client
    participant Queue as SQLite Sync Queue
    participant Cloud as Firebase Firestore
    participant User as User Client

    %% Admin editing a song
    Note over Admin, Queue: Admin Session (Offline)
    Admin->>Queue: Edit Song / Write Lyrics
    Queue->>Queue: Stash transaction in sync_queue

    Note over Admin, Cloud: Connection Restored (Online)
    Admin->>Cloud: Process sync_queue (authenticated write)
    Cloud->>Cloud: Write to collections & increment version

    %% User syncing
    Note over Cloud, User: User Launch
    User->>Cloud: Fetch latest pack versions
    alt Local version < Cloud version
        User->>Cloud: Pull pack document differences
        User->>User: Update SQLite songs
        User->>User: Set local pack version = cloud version
    end
```

---

## 3. Entity Relationship Diagram (SQLite Database)

The database schema manages local files, preferences, favorites, and the offline queue:

```mermaid
erDiagram
    PACKS {
        integer id PK
        string firestore_id
        string name
        string description
        string category
        integer song_count
        string icon
        integer is_installed
        integer is_default
        string installed_at
        integer version
    }
    SONGS {
        integer id PK
        string firestore_id
        string title
        string author
        string lyrics_json
        string lyrics_snapshot
        string category
        string tags
        string language
        integer pack_id FK
        string created_at
        string updated_at
    }
    FAVORITES {
        integer id PK
        integer song_id FK
        string created_at
    }
    SETTINGS {
        string key PK
        string value
    }
    CHORD_OVERRIDES {
        integer id PK
        integer song_id FK
        string key_name
        string override_json
    }
    SYNC_QUEUE {
        integer id PK
        string action_type
        string payload_json
        string created_at
    }

    PACKS ||--o{ SONGS : "contains"
    SONGS ||--o{ FAVORITES : "marked as"
    SONGS ||--o{ CHORD_OVERRIDES : "has overrides"
```

---

## 4. Key Architectural Decisions

| Decision | Rationale |
|---|---|
| **WASM SQL Fallback for Web** | Enables database migrations and seeds to run in browser tests (Playwright) without stubbing database hooks. |
| **Separation of Lyrics and Chords** | Lyrics are stored in a structured JSON layout, separating musical guides (brackets) from display stanzas. |
| **Session-Only Dev Mode** | Admin mode credentials and access are not persisted on disk. Leaving dev mode clears credentials and destroys temp queues. |
| **SQLite Indexing on Write** | Triggers and indices ensure quick catalog searches on devices with limited CPU performance. |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · For portfolio demonstration only.</sub>
