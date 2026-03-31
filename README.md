# lectr

A privacy-first reading notebook app for iOS and Android. One-time purchase, no subscriptions, no user accounts.

**[lectr.io](https://lectr.io)**

## Philosophy

Lectr is built around a hard constraint: **the server never sees your data.** An architectural decision enforced at every layer.

- No authentication system (there are no user accounts to breach)
- No server-side storage of user data
- On-device NLP generates a "Reading Fingerprint" — only anonymised statistical vectors are ever sent to the API
- All personal reading data stays on-device, synced via CloudKit/platform-native sync

## Architecture

The project is split into four independent repositories, each deployed and versioned independently:

| Component | Tech | Purpose |
|---|---|---|
| **iOS** | Swift, SwiftData, CloudKit | Main app — shipping |
| **Android** | Kotlin | Reaching feature parity with iOS |
| **API** | Cloudflare Workers | Serverless recommendations engine |
| **Website** | Static HTML/CSS/JS | [lectr.io](https://lectr.io) |

## Security model

API requests are gated by platform cryptographic verification:

- **iOS:** Apple App Attest
- **Android:** Play Integrity

The threat model focuses on economic deterrence rather than theoretical nation-state hardening.

## Release process

All components share unified semantic versioning (`vMAJOR.MINOR`). iOS, API, and website ship simultaneously to ensure consistent snapshots. The website maintains a separate `production` branch to prevent feature leaks before app releases.

## Project structure

```
lectr/
├── iOS/          → Swift app (separate repo)
├── Android/      → Kotlin app (separate repo)
├── API/          → Cloudflare Workers (separate repo)
├── website/      → lectr.io (separate repo)
└── CLAUDE.md     → Full project brief and architecture docs
```
