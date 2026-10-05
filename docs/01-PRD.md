# ScamShield — Product Requirements Document (PRD)

**Version:** 1.0 · **Owner:** Kartik (Vortex) · **License:** GPL-3.0 (code) + CC0 (rules data) · **Status:** Draft for build

---

## 1. Summary

ScamShield is a free, open-source, **fully offline** app that tells a person whether a message, link, or UPI request is likely a scam, explains *why* in plain language, and shows *what to do next*. It is built for the Indian fraud landscape (fake KYC, electricity-cut texts, parcel/customs, "digital arrest", UPI collect requests, job/task scams, loan-app harassment, fake customer care numbers).

**One-line pitch:** "Paste or share a suspicious message. Get a verdict in a second. Nothing ever leaves your phone."

## 2. Problem

- Online fraud in India hits millions of people every year; the most vulnerable are first-time smartphone users, parents, and grandparents.
- Scams succeed through urgency, authority, and fear; victims rarely have anyone to ask in the moment.
- Existing tools are closed-source, upload messages to servers (a privacy problem in itself), show ads, or only block numbers.
- Google Play restricts SMS/notification-reading permissions, so "read all my SMS" apps are a poor fit; a **share-sheet + paste** model is both safer and distributable without Play Console.

## 3. Goals and non-goals

### Goals
1. Verdict in under 1 second, fully on-device, works in airplane mode.
2. Explanations a 60-year-old can understand, in English, Hindi, and later more Indian languages.
3. Community-contributed detection rules via tiny JSON pull requests (no coding needed).
4. Zero network permission in the base build (INTERNET permission absent) — a verifiable privacy claim.
5. Beautiful, calm, trustworthy UI that feels like a safety product, not a toy.

### Non-goals (v1)
- No SMS auto-reading, call blocking, or notification listening.
- No cloud backend, accounts, analytics, or ads.
- No guarantee of detection; results are **risk indicators**, never certainty.
- No automatic reporting on the user's behalf (we guide, user acts).

## 4. Target users and personas

| Persona | Description | Key need |
|---|---|---|
| **Asha (58)** | Retired teacher, WhatsApp-heavy, low tech confidence | Big text, simple verdict, Hindi, "what do I do now" |
| **Rohan (22)** | College student, gets job/task/loan-app scams | Fast check, link analysis, learn patterns |
| **Meera (35)** | Adult child who gets forwards from parents | Share a screenshot-free result back to the family group |
| **Contributor (any)** | Security enthusiast / student | Add a rule in 5 minutes via PR |

## 5. User stories

**Core**
- As a user, I paste text and tap **Check**, and see Safe / Suspicious / Scam with reasons.
- As a user, I tap **Share → ScamShield** from WhatsApp/SMS/Chrome and land directly on the result.
- As a user, I see which exact words or links triggered the verdict, highlighted in the message.
- As a user, I get clear next steps (don't click, don't pay, call 1930, report on cybercrime.gov.in).
- As a user, I can switch language to Hindi (and more later).

**Secondary**
- As a user, I can check a standalone link, phone number, or UPI ID/handle format.
- As a user, I can see a short "Scam Library" explaining common scam types with examples.
- As a user, I can share the result as a clean text/image card to warn family.
- As a user, I can keep a local history and clear it in one tap (history is opt-in).

**Contributor**
- As a contributor, I can add a rule by editing one JSON file and opening a PR; CI validates it.

## 6. Functional requirements

### FR-1 Message analysis
- Input: free text up to 5,000 characters.
- Output: `{ verdict, score (0–100), signals[], matchedSpans[], advice[] }`.
- Verdict thresholds: Safe 0–29, Suspicious 30–64, Scam 65–100 (tunable in config).
- Always show a disclaimer: "Risk indicator, not a guarantee."

### FR-2 Link analysis
- Extract URLs from text; analyze each: shorteners, punycode/homoglyphs, lookalike of known brands (Levenshtein/token match against `brands.json`), suspicious TLDs, IP-address hosts, excessive subdomains, `@` tricks, non-HTTPS, "kyc/update/verify/reward" tokens in host.
- Show the **real domain** prominently, with the registrable part highlighted.
- Never open or fetch the link. No network calls.

### FR-3 UPI / payment request analysis
- Detect "collect request" language, "enter PIN to receive", "refund via QR", wrong-direction payment tricks.
- Rule: *You never need a PIN to receive money.*

### FR-4 Phone number / sender-ID hints
- Flag patterns like personal numbers claiming to be a bank, +92/international prefixes claiming to be domestic institutions, and sender IDs that don't match the claimed brand. Keep as soft signals only.

### FR-5 Explanations and advice
- Each signal has a title, a one-sentence plain explanation, and severity.
- Advice is contextual by scam category (e.g., digital-arrest → "No agency arrests by video call. Hang up.").
- Reporting card: **1930** (National Cyber Crime Helpline), **cybercrime.gov.in**, **Sanchar Saathi / Chakshu** for suspicious calls/messages, and "tell your bank immediately" with the card-block step. Tap-to-call and tap-to-open only; the user initiates.

### FR-6 Share target (Android)
- App registers as a share target for `text/plain`; shared text opens the Result screen directly.
- Also handles shared links.

### FR-7 Localization
- i18n with English + Hindi at launch; strings and rule explanations keyed by locale; fallback to English.

### FR-8 Scam Library
- 10–15 scam type cards (what it looks like, red flags, what to do), offline.

### FR-9 History (opt-in)
- Local-only (encrypted storage), off by default, delete-all button.

### FR-10 Rules updates
- Rules ship inside the app bundle. Optional later: manual "Import rules pack" from a file. No silent network fetching in the base build.

## 7. Non-functional requirements

| Area | Requirement |
|---|---|
| Privacy | No INTERNET permission in the release manifest; no analytics; no crash reporters that phone home |
| Performance | Analysis < 150 ms for 5k chars on a mid-range phone; cold start < 2 s |
| Size | APK < 30 MB (arm64 split) |
| Accessibility | WCAG AA contrast, 200% font scaling, screen-reader labels, color never the only signal |
| Offline | 100% functional in airplane mode |
| Android | minSdk 26 (Android 8), target latest stable |
| Quality | ≥ 90% unit-test coverage on the engine; golden-file tests for rules |
| Security | Reproducible build instructions; signed releases with published checksums |

## 8. Success metrics (open-source appropriate)

- Engine accuracy on the labelled test corpus: ≥ 90% scam recall, ≤ 5% false-positive on benign messages (tracked in CI).
- 50+ rules at launch, 150+ within 3 months.
- GitHub stars, forks, community PRs merged, languages contributed.
- Qualitative: user feedback "this saved me / my parents".

## 9. Scope and release plan

| Release | Contents |
|---|---|
| **v0.1 (MVP)** | Paste checker, scoring, explanations, 30–50 rules, English, share target, GitHub APK |
| **v0.2** | Link analyzer, Hindi, Scam Library, result share card |
| **v0.3** | PWA build with Web Share Target, F-Droid submission, history (opt-in) |
| **v1.0** | 150+ rules, 4+ languages, accessibility audit, optional on-device ML classifier, reproducible builds |

## 10. Risks and mitigations

| Risk | Mitigation |
|---|---|
| False positives erode trust | Conservative thresholds, "Suspicious" middle tier, test corpus of benign bank/OTP messages |
| Scammers adapt | Community rules, fast PR flow, versioned rules pack |
| Users over-trust the verdict | Persistent disclaimer, "when in doubt, call your bank on the number on your card" |
| Legal/defamation of brands or numbers | No blocklists of named individuals; domains/patterns only, with sources in rule metadata |
| Maintainer burnout | Clear CONTRIBUTING, CI validation, rule-review checklist |

## 11. Open questions

1. Hindi-first or English-first default based on device locale? (Recommend: follow device locale.)
2. Include an opt-in, user-initiated "Report this pattern" that opens a prefilled GitHub issue in the browser (no in-app network)? (Recommend: yes, as an external link.)
3. Add on-device ML in v1.0 or keep rules-only? (Recommend: rules first, ML as an assistive second opinion.)
