# Privacy Policy — Credit Card Tracker

**Effective date:** 2026-04-15
**Contact:** grits.0119@gmail.com

This policy describes how the Credit Card Tracker mobile app (“App”) handles information when you use it on your device.

## Summary

- **Card and statement data** you save in the App is stored **on your device** using Apple’s on-device storage (SwiftData).
- **OpenAI:** If you choose to use AI features (statement image analysis or the in-app assistant), **images and/or text are sent from your device to OpenAI** using an API key **you provide** (stored in the iOS Keychain on your device, or supplied via a developer environment variable in debug builds). OpenAI’s handling of that data is governed by **OpenAI’s policies**, not this document alone.
- The App **does not include our own user accounts** in the MVP described here; we do not operate a separate login server for this app.

## Information the App processes

1. **Financial details you enter or capture**  
   Examples: card name, balances, minimum payments, due dates, statement dates, credit limits, APRs, and statement images you upload.

2. **API key**  
   Your OpenAI API key is stored in the **iOS Keychain** on the device when you save it in Settings.

3. **Local notifications**  
   If you allow notifications, the App may schedule **local** reminders about payment due dates. Those notifications are generated on-device; we do not send your due dates to our servers (there are no app developer servers in this MVP).

## How information is used

- To display your balances, due dates, and history inside the App.
- To **send statement images and chat messages to OpenAI** when you use those features, so the model can return extracted fields or answers.
- To schedule **local** payment reminders if you opt in.

## Data retention and deletion

- Data stored by the App on your device remains until you **delete cards**, **remove the app**, or erase device content.
- Removing your API key in Settings stops new outbound OpenAI requests from using that key; it does not by itself delete data already sent to OpenAI while the key was active (see OpenAI’s documentation for their retention practices).

## Children

The App is not intended for children. Do not use it if you are under the age required in your region to consent to processing of personal data.

## Changes

We may update this policy. Post the new version at the same URL you provide to App Store Connect and update the “Effective date.”

## Contact

For privacy questions, contact: **grits.0119@gmail.com**

---

*Replace bracketed placeholders, set the effective date, host this file at a **public HTTPS URL**, and enter that URL in App Store Connect (App Privacy and optionally the App Description / Support URL).*
