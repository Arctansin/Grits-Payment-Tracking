# Data Transparency Disclosure

**App Name:** Grits Payment Tracker  
**Developer:** Mingming Li  
**Last Updated:** April 24, 2026  
**Contact:** [grits.0119@gmail.com](mailto:grits.0119@gmail.com)

---

## Before You Get Started

We believe you should know exactly how your data is used. Here is a plain-language summary of what this app collects, stores, and shares — and what it does not.

---

## What We Collect


| Data                                                                                         | Why                                                                  |
| -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Email address                                                                                | To create and manage your account                                    |
| Debt tracking info (account name, last 4 digits when provided, balance, due date, APR, etc.) | Core app functionality — tracking payments and balances              |
| AI chat messages and uploaded files *(if you use AI features)*                               | To answer your questions via the AI assistant                        |
| Debt account details from Plaid *(only if you connect via Plaid)*                            | To import eligible debt accounts (for example, credit/loan balances) |


**We never collect:** full card numbers, CVV codes, bank passwords, or Social Security Numbers.

---

## Where Your Data Lives

### Cloud sync is ON

Your card data is stored securely in Google Firestore (encrypted in transit and at rest). You can access it from any device when signed in.

### Cloud sync is OFF

Your data stays on **this device only**. We cannot see it, access it, or recover it. If you lose your phone, your data is gone permanently.

> You can change this setting anytime under **Settings → Sync data to Firebase**.

---

## Third-Party Services This App Uses

This app connects to outside services to function. Your data may pass through them.

**Firebase (Google)**
Handles your login and stores your cloud data. [Google's Privacy Policy →](https://policies.google.com/privacy)

**Plaid** *(only if you connect a bank account)*
Plaid connects to your bank directly. We never see your bank login or full account number — only the balance you authorize to share. [Plaid's Privacy Policy →](https://plaid.com/legal/#privacy-statement)

**OpenAI** *(only if you use the AI assistant)*
Your questions and any files you upload are sent to OpenAI for processing.

- If you use **your own OpenAI API key**: your data goes directly to OpenAI under your account. We see nothing.
- If you use **our built-in key**: your messages and files pass through our backend before reaching OpenAI. The developer can technically see this content, though we do not use it. For maximum privacy, use your own key in **Settings → Personal AI Service**.

[OpenAI's Privacy Policy →](https://openai.com/policies/privacy-policy)

---

## What We Do Not Do

- We do **not** sell your data
- We do **not** use your data for advertising
- We do **not** store your bank credentials or full card numbers
- We do **not** share your data with anyone beyond the services listed above

---

## Your Controls


| You want to…                | How to do it                                                        |
| --------------------------- | ------------------------------------------------------------------- |
| Keep data on device only    | Settings → Sync data to Firebase → Off                              |
| Use your own OpenAI key     | Settings → Personal AI Service → On, then enter API key             |
| Stop Plaid updates          | Do not use “Add bank account from Plaid” / “Refresh Plaid balances” |
| Request cloud data deletion | Email support with your account email                               |
| Ask us about your data      | Email: **[grits.0119@gmail.com](mailto:grits.0119@gmail.com)**      |


---

## This App Is Not Financial Advice

Grits Payment Tracker is a personal tracking tool. Nothing in this app — including AI responses — is financial advice. We are not a bank or licensed financial advisor. Always consult a qualified professional for financial decisions.

---

*Last updated: April 23, 2026*