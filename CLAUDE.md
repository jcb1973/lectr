# Lectr — Project Root

## What is Lectr?
Lectr (lectr.io) is a privacy-first reading notebook app for iOS and Android.
One-time purchase ($7.99), no subscriptions, no user accounts.

---

## Repository Structure
This root directory is a navigation convenience and Claude Code workspace.
Each subdirectory is a symlink to an independent git repo.

| Symlink    | Actual path                                             | Language / Stack                  |
|------------|---------------------------------------------------------|-----------------------------------|
| `iOS/`     | `~/XCJCB/SimpleReadingTracker/SimpleReadingTracker`     | Swift, Xcode, SwiftData, CloudKit |
| `Android/` | `~/AndroidStudioProjects/Marginalia`                    | Kotlin, Android Studio            |
| `API/`     | `~/src/lectr-recommendations`                           | Cloudflare Workers                |
| `website/` | `~/src/lectr-support`                                   | Static HTML/CSS/JS                |

> ⚠️ Legacy repo names: iOS repo is `SimpleReadingTracker`, Android repo is `Marginalia`,
> API repo is `lectr-recommendations`. These are old names — the product is Lectr.

### Working across repos
- Each component has its own git repo — commit and push from within the symlinked subdirectory
- Do not `git add` the symlinks from this root
- When making cross-cutting changes (e.g. new API endpoint + iOS consumer), work in each
  subdirectory independently and note the dependency in commit messages

---

## Component Responsibilities

### iOS (`iOS/`)
- Primary platform, Swift / SwiftData
- CloudKit sync for user library
- On-device NaturalLanguage processing (Reading Fingerprint feature)
- Calls the API for book recommendations and ISBN lookups
- Uses Apple App Attest for API request verification

### Android (`Android/`)
- Kotlin
- Calls the API for book recommendations and ISBN lookups
- Uses Play Integrity for API request verification
- Currently pre-release — in active development toward iOS feature parity

### API (`API/`)
- Cloudflare Workers (serverless, stateless)
- Verifies requests via Apple App Attest (iOS) and Play Integrity (Android) before serving
- Proxies book recommendations using Claude (primary) with Gemini as fallback
- Proxies ISBN lookups to prevent direct API abuse

### Website (`website/`)
- Static HTML/CSS/JS, hosted at lectr.io
- Support documentation and privacy policy

---

## Privacy Principles (Non-Negotiable)

Lectr is architecturally privacy-first. These are hard constraints, not preferences.

- **No server-side user data** — we never store anything about a user or their library
- **No user accounts** — no sign-up, no login, no user identity on any backend
- **No content transmission** — raw user data (book titles, notes, highlights) never leaves the device
- **Statistical representation only** — when the API needs user context (e.g. recommendations),
  it receives anonymised statistical representations (e.g. genre vectors, reading patterns),
  never actual content
- **No analytics that identify users** — no tracking, no fingerprinting, no behavioural telemetry

### Implications for development
When suggesting features or architectural changes, always respect these constraints:

- Do not suggest server-side storage solutions for user data
- Do not suggest authentication or account systems
- Do not suggest passing book titles, notes, or library contents to any external API
- Recommendation prompts must be derived from on-device statistical representations only
- Any new API endpoint must be stateless and receive no user-identifiable information

---

## Current Focus

### Android
- Achieving feature parity with iOS — this is the primary Android goal before release
- Do not suggest new features for Android that don't already exist in iOS

### iOS Marketing
- Active marketing push is underway for the current iOS release
- Website and App Store presence should be considered live and user-facing

---

## Cross-Cutting Concerns

### Privacy by Design
Privacy is not a feature — it is an architectural constraint. See **Privacy Principles** above.
Every component is designed so that privacy-preserving behaviour is the default, not an option.

### API Security
- Platform cryptographic verification is the security foundation:
  - iOS requests are verified via **Apple App Attest**
  - Android requests are verified via **Play Integrity**
- The threat model is **economic deterrence**, not nation-state hardening
  — the goal is to make attacking the API more expensive than it's worth,
  not to achieve perfect security
- The Claude proxy endpoint is the most sensitive endpoint and should receive the
  most scrutiny in any security-related changes
- When suggesting API changes, always consider whether the change weakens the
  App Attest / Play Integrity verification gate

### Future: Add-On Credits
- There is a potential future mechanism to allow end users to purchase additional
  API credits (e.g. for higher recommendation volume)
- This is not yet designed or implemented
- Any API or billing-adjacent work should be done with this possibility in mind —
  avoid architectural decisions that would make it hard to add later
- Any such mechanism must remain consistent with the no-accounts, privacy-first principles

---

## Release Process

### Branching
- All development happens on `main` in each repo
- **Website only**: a `production` branch exists to prevent feature leaks
  - Merge `main` → `production` only when the corresponding app release goes live

### Versioning
- Releases are tagged `vMAJOR.MINOR` (e.g. `v4.0`, `v4.1`)
- iOS, API, and website are tagged in sync at the same version number, even if
  not all three changed — this ensures any live state can be reconstructed from tags
- A release does not require all four components to ship simultaneously — for example,
  the API or website may be tagged and deployed independently of an iOS submission

### Android release
- Android is pre-release and not yet on the release tagging cycle
- Current goal: reach feature parity with iOS
- When Android is ready to ship, it joins the unified tag at the then-current iOS version
  (e.g. if iOS is at `v4.2`, Android's first release is tagged `v4.2`)
- This preserves a coherent cross-platform snapshot from day one of Android being live

### Running an iOS, website and API release
- The full 12-step release process (including benchmark gating and server tagging)
  is defined as a Claude Code slash command in the iOS repo
- To run: open Claude Code from `iOS/`, then invoke `/release`
- Source file: `iOS/.claude/commands/release.md`
- Do not invoke `/release` from the root workspace — it must be run from the iOS repo context

