# UECFI — Deployment

A high-level overview of the deployment pipeline and security configurations behind UECFI. Specific database credentials, project IDs, and release signing certificates are intentionally omitted.

---

## Deployment Pipeline

UECFI utilizes a split native and web build distribution topology, ensuring congregation members can install the application from app stores while allowing automated browser testing on the web.

```
                         ┌────────────────────────────────────────┐
                         │              Git Commit                │
                         └───────────────────┬────────────────────┘
                                             │
                      ┌──────────────────────┴──────────────────────┐
                      ▼                                             ▼
         ┌────────────────────────┐                    ┌────────────────────────┐
         │     EAS Build (iOS)    │                    │   EAS Build (Android)  │
         │  • Compiles native .ipa│                    │  • Compiles native .aab│
         └────────────┬───────────┘                    └────────────┬───────────┘
                      │                                             │
                      ▼                                             ▼
         ┌────────────────────────┐                    ┌────────────────────────┐
         │     TestFlight / iOS   │                    │     Google Play Store  │
         │     App Store Connect  │                    │     Internal Testing   │
         └────────────────────────┘                    └────────────────────────┘
```

---

## Build System (Expo EAS)

For mobile platforms, native binaries are compiled using **Expo Application Services (EAS)**:

- **EAS Build** — Configured via `eas.json`. Compiles native code on remote build agents, managing release signing keys and Keystores securely.
- **EAS Submit** — Automated deployment scripts upload compilation outputs directly to App Store Connect (TestFlight) and Google Play Console.
- **Expo Updates (OTA)** — Dynamic Over-The-Air (OTA) updates are pushed to active user devices for quick patches, bypassing full app store review processes.

---

## Firestore Security Rules

To protect the master song catalog from unauthorized changes, Firestore database access is governed by granular rules configured in `firestore.rules`.

Read operations are open to all users (offline client download), while modifications are restricted solely to accounts matching administrator emails:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Helper function to check if the authenticated user is an administrator
    function isAdmin() {
      return request.auth != null && 
             request.auth.token.email.endsWith('@uecfi.org');
    }

    // Song packs catalog
    match /packs/{packId} {
      allow read: if true;
      allow write: if isAdmin();
      
      // Nested songs collection within each pack
      match /songs/{songId} {
        allow read: if true;
        allow write: if isAdmin();
      }
    }
  }
}
```

---

## Firebase Authentication

Administrative authentication is managed using **Firebase Authentication**. When admins login through the Settings Easter Egg portal, their session is checked against email and password credentials. The resulting JSON Web Token (JWT) is passed along with requests, satisfying the Firestore write rules above.

---

<sub>© 2026 Mark Anthony Resoso · [Back to project](../README.md)</sub>
