# Credit Card Tracker App

- [Term of Use](https://arctansin.github.io/Grits-Payment-Tracking/Term-of-Use/)
- [Data Disclosure](https://arctansin.github.io/Grits-Payment-Tracking/Data-Disclosure/)
- [Privacy Policy](https://arctansin.github.io/Grits-Payment-Tracking/Privacy-Policy/)

## Product Requirements Document (PRD)

## 1. Overview

Credit Card Tracker App is a native iOS app (SwiftUI + SwiftData) that helps users track credit card balances, due dates, minimum payments, and payoff progress.

The app supports:

- Firebase authentication (email/password + Google Sign-In)
- Per-user local data isolation
- Optional Firebase cloud backup/sync
- AI-powered statement extraction (image/PDF)
- AI assistant chat with wallet context
- Balance history tracking by change date

The app uses a hybrid model:

- Local-first for speed and offline usability
- Optional cloud sync for backup/restore across devices

---

## 2. Goals

### Primary goals

- Let users capture credit card data quickly from statement images/PDFs.
- Keep card data accurate with manual editing and safe update flows.
- Give users a clear home dashboard for total balance and next due date.
- Provide secure sign-in and per-user data separation.

### Secondary goals

- Reduce missed due dates through local reminders.
- Allow optional cloud backup/restore via Firebase.
- Provide AI help that can answer both general and wallet-specific questions.

### Near-term expansion goal

- Add Plaid integration for automated credit account import and balance refresh.

---

## 3. Target Users

- Users with multiple credit cards who want a simple debt/payment organizer.
- Users comfortable uploading statement screenshots or PDFs.
- Users who want optional cloud backup without mandatory cloud dependency.

---

## 4. Current Feature Scope (Implemented)

## 4.1 Authentication and Account Management

- Email/password registration and sign-in
- Google Sign-In
- Email verification + resend verification email
- Password reset
- Email-first auth UX (existing vs new user routing)
- First-login profile setup for:
  - `firestonesync` (cloud backup preference)
  - `personalkey` (personal AI key preference)

## 4.2 User Profile and Access Control

- Firestore user profile under `users/{uid}`
- `tier` currently fixed to `"free"`
- App access gated by authenticated + verified state (or Google provider)

## 4.3 Data Storage and Sync

- Local persistence with SwiftData
- Per-user ownership field (`ownerUID`) on local entities
- Optional Firebase sync toggle in Settings
- Firestore backup collections under `users/{uid}`:
  - `cards`
  - `statements`
  - `balanceSnapshots` (new)

## 4.4 Statement Upload and AI Extraction

- Import sources:
  - Camera photo
  - Photo Library image
  - File importer (PDF/JPEG/PNG/HEIC)
- PDF pages rendered to JPEG (up to 5 pages)
- AI extraction with editable review before save
- Duplicate detection to avoid re-saving identical statements

## 4.5 Credit Card Management

- Create/update cards from statement extraction
- Edit card details manually
- Delete card and related records
- Mark payment as made (adjust reminders / due cycle behavior)

## 4.6 Balance History (Change-Only Timeline)

- Track balance snapshots for each card only when balance changes
- No daily noise entries
- Display balance history by date in card detail screen
- Backfill logic from existing statements for legacy users

## 4.7 Dashboard and Reminders

- Total balance summary
- Card count
- Next payment due
- Local notification reminders (with permission flow in Settings)

## 4.8 AI Assistant

- Dedicated Chat tab
- Includes wallet context (cards + aggregates) in prompts
- Two AI service modes:
  - Personal key mode (local keychain key)
  - Shared backend mode via Firebase Cloud Functions
- User-controlled toggle in Settings: **Personal AI Service**

---

## 5. Plaid Integration Plan (Next Phase)

## 5.1 Product Objective

Allow users to import credit card account data directly from Plaid so balances and limits can be populated/updated without manual statement uploads.

## 5.2 Architecture Decision

Plaid integration will use:

- Native iOS Plaid Link SDK for connection UI
- Firebase Cloud Functions as secure backend

Plaid secrets (`client_id`, `secret`) will never be stored in iOS app code.

## 5.3 Planned Backend Functions

- `plaidCreateLinkToken`
  - Auth required
  - Returns Plaid `link_token`
- `plaidExchangePublicToken`
  - Auth required
  - Exchanges `public_token` for `access_token`
  - Stores token server-side only
- `plaidGetCreditAccounts`
  - Auth required
  - Returns safe account fields (name, mask, current balance, limit, etc.)

## 5.4 Planned iOS Flow

1. User taps "Connect via Plaid"
2. App requests `link_token` from backend
3. App opens Plaid Link
4. On success, app sends `public_token` to backend
5. App fetches credit accounts and maps to local card records
6. Existing cards update by match logic (`name + lastFour` preferred)
7. Balance snapshot is created only when balance changes

## 5.5 Security Requirements for Plaid

- Never send/store Plaid access tokens on device
- Server-side token storage only
- Firestore rules enforce `uid` ownership for Plaid-linked data
- Audit logging for token exchange and account fetch failures

---

## 6. Non-Functional Requirements

### Performance

- Dashboard load target: under 2 seconds for typical user data size
- Statement extraction request timeout safety in app flow

### Reliability

- Local-first behavior should work even if network is unavailable
- Cloud sync should not block local writes
- AI/Plaid failures should return user-friendly error messages

### Security and Privacy

- Auth-required backend callable functions
- HTTPS-only communication
- Secrets in Firebase Secret Manager
- Financial data and AI usage disclosures in app policy/docs

---

## 7. Data Model (Current)

## CreditCard

- `id`
- `ownerUID`
- `cardName`
- `lastFourDigits`
- `balance`
- `minimumPayment`
- `paymentDueDate`
- `statementDate`
- `creditLimit`
- `interestRate`
- promo fields (`promoTypeRaw`, `promoEndDate`, `promoAPR`, `deferredBackChargeAPR`)
- `updatedAt`

## StatementRecord

- `id`
- `ownerUID`
- `imagePath`
- `createdAt`
- flattened extracted statement fields
- optional link to `CreditCard`

## BalanceSnapshot

- `id`
- `ownerUID`
- `balance`
- `recordedAt`
- `source`
- optional link to `CreditCard`

## User Profile (Firestore)

- `displayName`
- `email`
- `tier` (`free`)
- `firestonesync` (Bool)
- `personalkey` (Bool)
- usage counters (`dailyAiCount`, etc.)

---

## 8. App Store and Compliance Notes

- Required for release:
  - Apple Developer account
  - Privacy policy + terms
  - Accurate data usage disclosures
  - Secure handling of auth/AI/Plaid secrets
- For App Review:
  - Ensure AI shared-key mode works when user has no personal key
  - Avoid hardcoded API keys in app binary
  - Keep local + cloud behaviors clearly explained in Settings

---

## 9. Out of Scope (Current)

- Investment/tax/legal advisory guarantees
- Full transaction categorization and spending analytics
- Multi-tenant admin tooling
- Web client

---

## 10. Open Items

- Finalize Firestore rules for new `balanceSnapshots` path and verify upload visibility.
- Implement production Plaid functions and iOS Link flow.
- Add optional manual sync status UI (last sync timestamp, error surface).

---

## 11. Plaid Environment Switching (Sandbox <-> Production)

This app uses Firebase Functions secrets to control Plaid environment and credentials.

### Active backend configuration keys

- `PLAID_ENV` (`sandbox`, `development`, or `production`)
- `PLAID_CLIENT_ID`
- `PLAID_SECRET`
- `PLAID_REDIRECT_URI`

### Switch to Plaid Production

1. In Plaid Dashboard (Production app), verify:
  - production `client_id` and `secret`
  - Allowed redirect URIs include the exact URI used by backend
2. Update Firebase secrets:
  - `PLAID_ENV=production`
  - `PLAID_CLIENT_ID=<prod client id>`
  - `PLAID_SECRET=<prod secret>`
  - `PLAID_REDIRECT_URI=<exact allowed redirect URI>`
3. Redeploy functions.
4. Relaunch app and run smoke test:
  - Add bank account from Plaid
  - Exchange token succeeds
  - Debt accounts import/update succeeds

### Switch back to Sandbox (rollback)

1. Update secrets:
  - `PLAID_ENV=sandbox`
  - `PLAID_CLIENT_ID=<sandbox client id>`
  - `PLAID_SECRET=<sandbox secret>`
  - `PLAID_REDIRECT_URI=<sandbox allowed redirect URI>`
2. Redeploy functions.
3. Re-test with sandbox institution credentials.

### Deployment notes

- If prompted about stale secret versions, choose to redeploy so functions use the latest secret version.
- If prompted to delete unrelated legacy functions during deploy, selecting `No` will still continue deployment.
- No iOS binary change is required for environment switch if backend secret-driven config is used.