# UECFI — Case Study

> **A Mobile worship companion for modern congregations.**
> *Replacing printed binders with an offline-first, transposable digital hymnal.*

---

## The Challenge

Worship leaders and musicians at UECFI previously relied on printed binders, photocopying sheets, and scrolling through offline PDF documents during service runs. This analog layout created significant operational pain points:

- **Key Transposition Friction** — Vocalists often request a key change right before or during a practice session. Transposing chords in your head or manually rewriting them on sheet music is slow, error-prone, and causes delays.
- **Sanctuary Connectivity Drops** — Sanctuary structures are often heavily insulated, creating severe cellular dead zones. Standard cloud-hosted web apps or database APIs fail completely under these conditions, rendering digital songbooks inaccessible.
- **Fragmented Catalog Management** — Updates to lyrics (correcting typos, adding stanzas) required printing new versions or asking members to re-download files, causing version fragmentation across the team.

---

## The Solution

The **UECFI Songbook Companion** is a React Native mobile application built on an offline-first architecture. It features local caching, custom text parsing, and cloud-synchronized content management.

### In-Memory Chord Transposition

We engineered a lightweight parser that identifies bracketed chord markers (`[G]`, `[Em7]`) within raw lyrics text. When a user selects a semitone adjustment on the screen, the transposition engine shifts the pitch in-memory. The UI recalculates alignments instantly, enabling musicians to switch song keys on-the-fly without modifying the underlying database record.

### Offline-First SQLite Data Layer

Rather than querying a remote API on page load, UECFI pre-seeds a local SQLite database with bundled hymnal packs. All user preferences (favorites list, transposition offsets, custom font sizes) are stored locally. The application loads instantly and remains 100% functional inside steel-reinforced sanctuaries with no network connection.

### Hidden Dev Portal & Sync Queue

For administrative content managers, a hidden dev portal is accessible via an easter-egg tap gesture in the Settings tab. Once authenticated via Firebase Auth, administrators can add new hymns or modify existing stanzas directly from their phones. Local changes are staged in an offline SQLite sync queue and automatically pushed to Firestore when network connectivity is re-established.

---

## Impact & Results

Measurable outcomes that demonstrate the value delivered by the UECFI system:

<table>
<tr>
<td align="center" width="33%">

### **100%**
**Offline Independence**

All seeded hymns, custom favorites, and settings run entirely locally, eliminating network failure points.

</td>
<td align="center" width="33%">

### **0ms**
**Key Shifting Latency**

Dynamic pitch changes are calculated in-memory, updating chord layouts instantly with zero render lag.

</td>
<td align="center" width="33%">

### **<2s**
**Synchronized Updates**

Admin corrections sync to Firebase and download automatically to other users' caches upon app launch.

</td>
</tr>
</table>

---

## Architecture at a Glance

The mobile app relies on **React Native (Expo v54)** for clean native views across mobile platforms. Local state management is backed by **Zustand** with persistent storage handled via **SQLite (expo-sqlite)**. Cloud backups and real-time administrative syncing are powered by **Firebase Firestore**.

See [`architecture.md`](../architecture/architecture.md) for system design diagrams and [`tech-stack.md`](../tech-stack.md) for the full technology breakdown.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md) · See [LICENSE](../../../LICENSE)</sub>
