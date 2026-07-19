# UECFI — Tech Stack

A complete breakdown of the technologies powering UECFI, with rationale for each choice.

---

## Stack Family: React Native & Expo

UECFI is built on **React Native (Expo v54)**, utilizing a unified TypeScript codebase to deliver a smooth, responsive, native mobile application experience across Android, iOS, and Web targets.

> **Why Expo?** Expo 54 provides modern file-based routing (`expo-router`), excellent TypeScript support, and robust local storage systems (`expo-sqlite`). By compile-targeting the web as well, it allowed us to construct automated headless browser tests and screenshot pipelines using Playwright, without needing complex Android/iOS emulators.

---

## Core Client Architecture

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **Core Framework** | React Native 0.81 (Expo 54) | Fast, responsive, native rendering engine with unified asset management |
| **Language** | TypeScript 5.9+ | Static typing for lyrics JSON structures, database schema models, and chord transposition states |
| **Routing Engine** | `expo-router` v6 | File-based navigation providing modal presentation, deep-linking, and typed routes |
| **Local Database** | `expo-sqlite` v16 | Embedded SQLite database providing high-speed local SQL queries for song catalogs and favorites |
| **State Management** | Zustand 5.x | Lightweight, decoupled global state management for theme, font size, transposition, and admin credentials |

---

## Cloud Integration & Synchronization

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **Primary Data Sync** | Firebase Firestore | Real-time, document-oriented cloud database for remote song pack hosting and catalog adjustments |
| **Authentication** | Firebase Auth | Secure administrative account gatekeeper for dev console updates |
| **Offline Sync Queue** | SQLite + Custom Sync Manager | Queues local additions/edits during offline admin sessions, syncing to Firestore when connection is restored |
| **Network Detection** | NetInfo (`@react-native-community/netinfo`) | Tracks device network connectivity state to notify users and toggle real-time capabilities |

---

## UI, Styling & Aesthetics

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **Styling Primitive** | StyleSheet API | High-performance, native layout calculations conforming to flexbox standards |
| **Theme System** | Custom JS Theme Tokens | Dynamic Light/Dark variables covering semantic colors, typography configurations, and margins |
| **Visual Accents** | `expo-linear-gradient` | Smooth, high-performance visual transitions on card interfaces |
| **Vector Icons** | `@expo/vector-icons` (Ionicons) | Vector glyphs for navigation tabs, favorites buttons, and quick browse nodes |

---

## Testing & Quality Assurance

| Layer | Tool | Rationale |
|-------|------|-----------|
| **Developer Tools** | Expo Go & EAS CLI | Streamlined development builds and production releases |
| **Screenshot Automation** | Playwright (Python API) | Headless browser execution capturing hydrated web targets |
| **Image Processing** | Pillow (PIL) | Automated cropping, resizing, and styling for portfolio showcase assets |

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](./README.md)</sub>
