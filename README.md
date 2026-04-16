# Credit Card Tracker — Product Requirements Document

**Version:** 1.1 (Post-MVP)
**Last Updated:** April 2026
**Platform:** iOS (SwiftUI, iOS 17+)
**Author:** Mingming

---

## 1. Overview

Credit Card Tracker is a native iOS application that helps users manage multiple credit cards in one place. Users can upload images of their credit card statements, which are parsed by AI to extract key financial data. All data is stored locally on the user's device — no cloud, no authentication, no third-party financial API required.

The app also includes an AI chatbot tab powered by OpenAI, allowing users to ask questions about credit card concepts and app usage.

---

## 2. Goals

**Primary Goals**
- Allow users to upload credit card statement images and automatically extract key financial data using AI
- Display a consolidated dashboard of all credit card balances and upcoming payment due dates
- Persist all data locally on the user's device across app launches
- Provide an easy way to manually correct or update extracted data

**Out of Scope (Deferred)**
- Bank API integrations (e.g. Plaid) — requires a backend; not compatible with local-only architecture
- Cloud sync or iCloud backup
- Push notification reminders
- Transaction history tracking
- User authentication

---

## 3. Target Users

Individual credit card holders who:
- Own multiple credit cards and want a simple unified view
- Take screenshots of statements from banking apps
- Want to track balances and due dates without logging into each bank separately
- Prefer their financial data to stay on their own device

---

## 4. Technology Stack

| Layer | Choice |
|---|---|
| Language | Swift |
| UI Framework | SwiftUI |
| Persistence | SwiftData (local on-device) |
| AI Extraction | OpenAI Vision API (`gpt-4o`) |
| AI Chatbot | OpenAI Chat Completions API (`gpt-4.1-mini`) |
| Image Handling | PhotosUI + UIImagePickerController (camera) |
| Architecture | MVVM |
| Minimum iOS | 17.0 |

---

## 5. Architecture

```
Views (SwiftUI)
     ↓
ViewModels (@MainActor ObservableObject)
     ↓
Services (AIExtractionService, AIChatService, ImageStorageService)
     ↓
SwiftDataStorageService (ModelContext)
     ↓
SwiftData (on-device persistent store)
```

### Key Files

| File | Role |
|---|---|
| `CreditCardTrackerApp.swift` | App entry point, ModelContainer setup |
| `FinanceModels.swift` | `@Model` classes: `CreditCard`, `StatementRecord`; plain structs: `ExtractedStatementData`, `ChatMessage` |
| `SwiftDataStorageService.swift` | Owns ModelContext, all CRUD operations, seeds data on first launch |
| `LocalStorageService.swift` | Protocol defining the storage interface |
| `AppViewModels.swift` | `UploadViewModel`, `DashboardViewModel`, `ChatbotViewModel` |
| `RealAIExtractionService.swift` | Sends base64 image to OpenAI Vision API, parses JSON response |
| `AIChatService.swift` | Sends chat messages to OpenAI Chat Completions API |
| `RootTabView.swift` | Tab navigation: Home, Upload, Cards, Chat |

---

## 6. Data Models

### CreditCard (`@Model` class)
| Field | Type | Notes |
|---|---|---|
| `id` | UUID | Unique, indexed |
| `cardName` | String | Matched case-insensitively on upsert |
| `balance` | Double | Current balance |
| `minimumPayment` | Double | |
| `paymentDueDate` | Date | |
| `statementDate` | Date | |
| `creditLimit` | Double? | Optional |
| `updatedAt` | Date | Updated on every save |
| `statements` | [StatementRecord] | Cascade delete relationship |

### StatementRecord (`@Model` class)
| Field | Type | Notes |
|---|---|---|
| `id` | UUID | Unique |
| `imagePath` | String | Local file path of stored image |
| `createdAt` | Date | |
| `extractedCardName` | String | Flattened from ExtractedStatementData |
| `extractedBalance` | Double | |
| `extractedMinimumPayment` | Double | |
| `extractedPaymentDueDate` | Date | |
| `extractedStatementDate` | Date | |
| `extractedCreditLimit` | Double? | |
| `card` | CreditCard? | Inverse relationship |

### ExtractedStatementData (plain struct)
Transient — used only during the AI extraction flow before saving to SwiftData.

### ChatMessage (plain struct)
In-session only — chat history is not persisted across app launches.

---

## 7. Screens & Features

### 7.1 Home Dashboard (`DashboardView`)

Displays a financial summary at the top and a list of all tracked cards below.

**Summary card shows:**
- Total balance across all cards
- Number of cards tracked
- Next payment due date

**Card list shows per card:**
- Card name (with themed styling: Chase blue, Amex gold, neutral for others)
- Current balance
- Minimum payment
- Due date

**Actions:**
- Tap any card → navigate to Card Detail
- "Upload Statement" button → navigate to Upload tab

---

### 7.2 Upload Statement (`UploadStatementView`)

Allows users to import a statement image from their photo library or camera.

**Features:**
- Photo library picker via `PhotosUI`
- Camera capture via `UIImagePickerController`
- Full-screen processing overlay while AI extraction runs
- Duplicate detection — if all key fields match an existing record, shows "Already uploaded" and skips save
- On successful extraction → opens Review sheet

---

### 7.3 AI Extraction (`RealAIExtractionService`)

Sends the selected image to OpenAI Vision API (`gpt-4o`) for extraction.

**Extracted fields:**
- Card name
- Current balance
- Minimum payment
- Payment due date
- Statement date
- Credit limit (optional)

**Prompt instructs the model to:**
- Return valid JSON only, no markdown
- Return `null` for any field that cannot be found

**Error handling:**
- Invalid image → `AIExtractionError.invalidImage`
- API failure → `AIExtractionError.extractionFailed`
- Strips markdown fences from response before JSON parsing

---

### 7.4 Review Extracted Data (`ReviewExtractedDataView`)

Shown as a sheet after successful extraction. Lets the user confirm or correct what the AI found before saving.

**Editable fields:**
- Card name
- Current balance
- Minimum payment
- Payment due date

**Actions:**
- Toggle edit mode
- Save card → calls `SwiftDataStorageService.saveStatement()`
- Dismiss without saving

---

### 7.5 Cards List (`CardsListView`)

A dedicated tab listing all tracked cards. Tapping any card navigates to Card Detail.

---

### 7.6 Card Detail (`CardDetailView`)

Shows full details for a single card including statement history.

**Displays:**
- Current balance (large header)
- Minimum payment and due date
- Statement history list (month + balance per statement)

**Actions:**
- "Edit Record" → opens Edit sheet
- "Delete Card" → removes card and all associated statements

---

### 7.7 Edit Record (`EditCardView`)

Sheet for manually updating a card's information.

**Editable fields:**
- Card name
- Current balance
- Minimum payment
- Payment due date

Changes are saved immediately to SwiftData and reflected across all views.

---

### 7.8 AI Chatbot (`ChatbotView`)

A conversational assistant powered by OpenAI Chat Completions API.

**Features:**
- Chat bubble UI (user right-aligned, assistant left-aligned)
- In-session message history (not persisted across launches)
- Loading indicator while response is in flight
- Send button disabled while request is in-flight
- Error message shown on failure
- Disclaimer: "AI responses may be inaccurate. Verify important information."

**System prompt instructs the model to:**
- Be helpful and concise
- Not provide legal, tax, or investment advice

---

## 8. Persistence — SwiftData

`SwiftDataStorageService` owns the `ModelContext` and handles all reads and writes.

**Key behaviors:**
- `ModelContainer` is created at app launch in `CreditCardTrackerApp.swift`
- Context is injected via `@Environment(\.modelContext)` in `AppEntryPoint`
- On first launch (empty store), seed data is inserted: Chase Sapphire Preferred, Amex Gold, Capital One Venture with sample statements
- `saveStatement()` performs an upsert — if a card with the same name exists, it updates the existing record; otherwise it creates a new one
- `deleteCard()` cascades to delete all associated `StatementRecord` entries
- All mutations call `context.save()` and re-fetch to keep `@Published` arrays in sync

---

## 9. App Icon

**Design direction:** Blue background (`#0052CC`), white credit card with gold EMV chip, green checkmark badge.

**Key colors:**
| Role | Hex |
|---|---|
| Primary blue | `#0052CC` |
| Primary dark | `#003D99` |
| Check green | `#00C853` |
| Chip gold | `#FFD700` |
| Destructive red | `#DD3333` |

**Asset requirements:**
- 1024×1024 PNG, no transparency, no pre-rounded corners
- Place in `Assets.xcassets/AppIcon.appiconset/`
- Add `"filename"` key to `Contents.json` matching the PNG filename

---

## 10. Security & Privacy

- API key (`OPENAI_API_KEY`) stored as an Xcode scheme environment variable — never hardcoded in source
- All financial data stored locally on-device via SwiftData
- No user credentials stored
- All API communication over HTTPS
- Statement images processed by OpenAI Vision API — users should be informed via the Me/Privacy screen

---

## 11. Known Limitations (Current Version)

- Chat history is lost when the app is closed (in-memory only)
- Statement images are saved to the local Documents directory but the path is not currently displayed in the UI
- No push notifications for upcoming payment due dates yet
- Seed data reappears if the SwiftData store is deleted (e.g. app reinstall)
- No iCloud backup of SwiftData store

---

## 12. Future Enhancements

| Feature | Notes |
|---|---|
| Payment due reminders | Local push notifications via `UserNotifications`, no backend needed |
| Balance trend chart | Use statement history already stored in SwiftData |
| Spending insights | Categorize transactions extracted from statement images |
| iCloud sync | SwiftData supports CloudKit with minimal changes |
| App icon | Finalize 1024×1024 PNG and assign in Xcode |
| Persist chat history | Store `ChatMessage` as a SwiftData model |
| Plaid integration | Requires a backend — revisit when server infrastructure is ready |
