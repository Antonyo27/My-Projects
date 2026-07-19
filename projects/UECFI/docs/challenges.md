# UECFI — Engineering Challenges

A selection of the harder problems encountered while building UECFI and how they were solved.

---

## 1. Dynamic Chord Parsing and Pitch Transposition

**The Problem.** Lyrics are entered into the database as a single string containing inline chord markers (e.g., `[G] Jesus agyamancami [C] taraon...`). Regular expression replacements can match brackets, but transposing these notes dynamically (e.g., from `G` to `A` or `F#`) requires understanding musical scales, handling accidental formatting (sharps vs. flats), and preserving chord extensions (such as `m7`, `sus4`, `dim`).

**The Solution.** We built a custom parser and pitch transposition engine:
1. **Accidental Mapping**: We define the chromatic scale index map (0 to 11) for roots: `C`, `C#`/`Db`, `D`, `D#`/`Eb`, `E`, `F`, `F#`/`Gb`, `G`, `G#`/`Ab`, `A`, `A#`/`Bb`, `B`.
2. **Tokenization**: Chord strings are split into root notes (e.g., `C#`) and chord modifiers (e.g., `m7sus4`).
3. **Index Shifting**: The root note index is shifted by a transposition offset (semitones value) modulo 12:
   $$\text{new\_index} = (\text{old\_index} + \text{offset}) \bmod 12$$
4. **Reassembly**: The shifted root is mapped back to its note name and rejoined with the original modifiers. The UI performs this calculation during the rendering cycle of the `LyricsView` component, ensuring instant transposition with no layout shifts.

---

## 2. Offline-First SQLite Cache Synchronization

**The Problem.** UECFI must operate fully offline. However, administrators need the ability to add and update songs, which must sync up to Firestore. Normal users must also download these changes when they reconnect. Standard direct-to-cloud SDK queries cause the app to crash or hang indefinitely in sanctuaries with bad reception.

**The Solution.** We implemented a local staging and delta tracking layer:
- **Admin Sync Queue**: Admin writes are logged to a local SQLite `sync_queue` table containing transaction payloads. A sync manager listens to network status using NetInfo. When the device returns online, the manager executes queued actions against Firestore in order and deletes the local queue upon successful server confirmation.
- **Pack Versioning**: The cloud Firestore catalog holds a global version counter for each pack. On launch, the client checks if the local SQLite pack version matches the remote version. If the remote version is higher, the app fetches the delta list of modified songs, updates the local SQLite tables, and bumps the local version. This minimizes data usage and guarantees synchronization consistency.

---

## 3. Gating the Dev Portal Without Bloating the UI

**The Problem.** The application is used by a broad congregation of non-technical members. Adding a prominent "Admin Login" or "Developer Settings" button would clutter the interface and invite unauthorized attempts.

**The Solution.** We implemented a hidden **Easter Egg gesture**:
- In the settings panel, the app version text (e.g., `v1.0.0`) is wrapped in a touchable pressable.
- Tapping this label 7 times within 3 seconds triggers the `DevLoginModal` window.
- The administrator log-in executes via Firebase Authentication. 
- Access is strictly governed by Firebase Security Rules: writes to Firestore collections check if the authenticated user's email matches the verified administrator email list, preventing mock accounts from updating the remote master catalog.

---

## 4. Web Compat & WASM Bundling for Local Tests

**The Problem.** The core data access layers depend on `expo-sqlite`, which uses native Android/iOS C-bindings. Since we wanted to build a web-based portfolio deployment and automated screenshot extraction using Playwright, standard native builds would throw immediate dependency errors when compiled for web.

**The Solution.** We configured a dual Web-Assembly (WASM) fallback:
- We adjusted Metro's resolver config in `metro.config.js` to support `.wasm` extensions.
- When compiled for web, `expo-sqlite` automatically routes database connections to a web-worker (`worker.ts`) using an in-memory SQL.js script backed by WASM.
- Upon app launch, the web client fetches the bundled `starter-pack.json` asset, seeds the WASM SQL database inside the browser, and renders the layout. This enables full browser support and allows Playwright to run automated UI workflows.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
