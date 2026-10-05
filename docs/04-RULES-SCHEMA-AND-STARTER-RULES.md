# ScamShield — Rules Schema, Taxonomy & Starter Rules

## 1. Scam category taxonomy (`ScamCategory`)

| id | Description |
|---|---|
| `bank_kyc` | "Your account will be blocked, update KYC/PAN now" |
| `otp_phishing` | Asks for OTP/PIN/CVV/password |
| `upi_collect` | Collect request, "enter PIN to receive", refund QR |
| `utility_cut` | "Electricity/gas/water will be cut tonight" |
| `parcel_customs` | Courier/customs/FedEx/India Post fees, held parcel |
| `digital_arrest` | Fake police/CBI/ED/TRAI/customs on call/video, "you are under arrest" |
| `job_task` | Part-time job, like/review tasks, Telegram group, deposits |
| `loan_app` | Instant loan apps, harassment, upfront fees |
| `lottery_prize` | Prize, lucky draw, KBC, gift |
| `tech_support` | Remote-access apps, fake support, antivirus |
| `fake_customer_care` | Fake helpline numbers in search results/messages |
| `investment_crypto` | Guaranteed returns, stock/crypto groups, trading apps |
| `romance_sextortion` | Video-call blackmail, honeytrap |
| `government_impersonation` | Fake challan, tax refund, Aadhaar/PAN, subsidy |
| `generic_urgency` | Pressure language without a specific category |

## 2. Rule file format (JSON)

Each file in `packages/rules/data/rules/` is an array of rules.

```json
{
  "id": "kyc-urgency-001",
  "category": "bank_kyc",
  "severity": "high",
  "weight": 25,
  "title": { "en": "Threatens to block your account", "hi": "खाता बंद करने की धमकी" },
  "explanation": {
    "en": "Banks do not block accounts by SMS for pending KYC and ask you to click a link.",
    "hi": "बैंक KYC के नाम पर SMS से लिंक भेजकर खाता बंद करने की धमकी नहीं देते।"
  },
  "match": {
    "patterns": ["(account|a\\/c).{0,40}(block|suspend|freez|deactivat)"],
    "keywordsAny": ["kyc", "pan update", "verify now"],
    "keywordsAll": [],
    "requires": [],
    "excludes": []
  },
  "adviceKeys": ["dont_click_links", "call_bank_official"],
  "sources": ["https://www.rbi.org.in/ (public advisory on KYC fraud)"],
  "examples": {
    "positive": [
      "Dear customer, your SBI account will be blocked today. Update KYC now: http://sbi-kyc-update.xyz"
    ],
    "negative": [
      "Your account ending 1234 was credited with Rs 500. Avl bal Rs 5,230."
    ]
  },
  "tags": ["sms", "whatsapp"],
  "addedBy": "github-username",
  "addedOn": "2026-10-05"
}
```

### Field rules
- `id`: unique, `<topic>-<name>-<number>`, lowercase-kebab.
- `severity`: `low | medium | high | critical`.
- `weight`: integer 1–40 (critical ≤ 40; informational signals 1–10; negative weights allowed **only** in `benign.json`, −5 to −30).
- `match`: at least one of `patterns`, `keywordsAny`, `keywordsAll`. Logic: (any pattern OR any keywordsAny OR all keywordsAll) AND all `requires` AND none of `excludes`. `requires`/`excludes` are rule-local keyword/regex arrays.
- Regex: JS flavor, matched case-insensitive on normalized text, max 200 chars, must pass the ReDoS checker.
- `examples.positive` ≥ 1 and `examples.negative` ≥ 1 are **mandatory**; CI runs them.
- `sources`: where the pattern is publicly documented (RBI, I4C, CERT-In, news advisories). Optional but encouraged.
- Translations: `en` required; others optional (fallback to `en`).

## 3. JSON Schema (draft 2020-12) — `packages/rules/schema/rule.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://scamshield.dev/schema/rule.schema.json",
  "type": "object",
  "required": ["id", "category", "severity", "weight", "title", "explanation", "match", "examples"],
  "additionalProperties": false,
  "properties": {
    "id": { "type": "string", "pattern": "^[a-z0-9]+(-[a-z0-9]+)*-[0-9]{3}$" },
    "category": { "type": "string", "enum": ["bank_kyc","otp_phishing","upi_collect","utility_cut","parcel_customs","digital_arrest","job_task","loan_app","lottery_prize","tech_support","fake_customer_care","investment_crypto","romance_sextortion","government_impersonation","generic_urgency","benign"] },
    "severity": { "type": "string", "enum": ["low","medium","high","critical"] },
    "weight": { "type": "integer", "minimum": -30, "maximum": 40 },
    "title": { "$ref": "#/$defs/i18n" },
    "explanation": { "$ref": "#/$defs/i18n" },
    "match": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "patterns": { "type": "array", "items": { "type": "string", "maxLength": 200 } },
        "keywordsAny": { "type": "array", "items": { "type": "string" } },
        "keywordsAll": { "type": "array", "items": { "type": "string" } },
        "requires": { "type": "array", "items": { "type": "string" } },
        "excludes": { "type": "array", "items": { "type": "string" } }
      },
      "anyOf": [
        { "required": ["patterns"] },
        { "required": ["keywordsAny"] },
        { "required": ["keywordsAll"] }
      ]
    },
    "adviceKeys": { "type": "array", "items": { "type": "string" } },
    "sources": { "type": "array", "items": { "type": "string", "format": "uri" } },
    "examples": {
      "type": "object",
      "required": ["positive", "negative"],
      "properties": {
        "positive": { "type": "array", "minItems": 1, "items": { "type": "string" } },
        "negative": { "type": "array", "minItems": 1, "items": { "type": "string" } }
      }
    },
    "tags": { "type": "array", "items": { "type": "string" } },
    "addedBy": { "type": "string" },
    "addedOn": { "type": "string", "format": "date" }
  },
  "$defs": {
    "i18n": {
      "type": "object",
      "required": ["en"],
      "additionalProperties": { "type": "string" }
    }
  }
}
```

## 4. Starter rule set (target 40+; write these first)

Use these as the checklist the build agent must implement as JSON with examples. Weights in brackets are suggested.

### bank_kyc
1. Account will be blocked/suspended + KYC/PAN update [25, high]
2. "Update KYC/PAN/Aadhaar" + link [20, high]
3. "Your debit/credit card is deactivated, click to reactivate" [22, high]
4. Reward-points expiry + "redeem now" + link [18, medium]
5. Fake net-banking login prompt wording ("login to avoid suspension") [20, high]

### otp_phishing
6. Asks you to share/send OTP, PIN, CVV, password [40, critical, floor 80]
7. "I sent OTP by mistake, please share" [35, critical]
8. "Share the code you received to confirm delivery/refund" [35, critical]

### upi_collect
9. "Enter your UPI PIN to receive money/refund/cashback" [40, critical]
10. "Approve the collect request to get your prize/refund" [30, high]
11. Wrong-payment-sent, "please return the money" + QR/UPI link [20, medium]
12. Scan-QR-to-receive-payment (QR for receiving never needs scanning) [30, high]

### utility_cut
13. Electricity will be disconnected tonight + call/click [30, high]
14. Gas/water/DTH/SIM "will be disconnected" + KYC/bill update [25, high]
15. "Call this mobile number (not official helpline) to avoid cut" [18, medium]

### parcel_customs
16. Parcel held at customs/courier, pay fee/duty via link [28, high]
17. "Drugs/illegal items found in your parcel" (digital-arrest lead-in) [35, critical]
18. India Post/FedEx/DHL/Blue Dart impersonation + non-official link [25, high]
19. "Delivery failed, reschedule" + shortened link [15, medium]

### digital_arrest / government_impersonation
20. CBI/ED/Police/TRAI/Customs claims on call/video + "you are under investigation" [40, critical]
21. "Do not disconnect the call / stay on video" [30, high]
22. "Your SIM/Aadhaar used for illegal activity, number will be blocked in 2 hours" [30, high]
23. "Transfer money to a safe/RBI/verification account" [40, critical]
24. Fake traffic challan/e-challan with link [22, high]
25. Income-tax refund / PM scheme subsidy + link/KYC [20, high]

### job_task / investment_crypto
26. Part-time job, earn ₹X/day, WhatsApp/Telegram contact [18, medium]
27. Like/subscribe/rate tasks for money [25, high]
28. Pay deposit/registration fee to start job or unlock earnings [35, critical]
29. Guaranteed/daily returns, double your money [30, high]
30. Join stock tips/VIP trading group, "insider" tips [22, high]
31. Crypto/USDT investment platform with withdrawal fee [28, high]

### loan_app
32. Instant loan, no documents, pay processing fee first [30, high]
33. Threats to contact your phone contacts / shame you [35, critical]
34. "Download this APK to get the loan" (sideloaded APK) [30, high]

### lottery_prize / tech_support / misc
35. You have won a prize/lottery/KBC/gift [28, high]
36. Claim by paying tax/fee/shipping [35, critical]
37. Install AnyDesk/TeamViewer/QuickSupport/AirDroid [40, critical, floor 80]
38. Fake customer-care number for bank/e-commerce/UPI app [22, high]
39. Romance/video-call blackmail wording [35, critical]
40. Download APK from a link (e.g., "PM Kisan.apk", "RTO challan.apk") [30, high]

### generic_urgency (low weights, combine with others)
41. "Immediately/within 24 hours/today only/last warning" [6, low]
42. Threat language: legal action, FIR, arrest, penalty [10, medium]
43. Excessive punctuation/ALL CAPS urgency [3, low]
44. Poor templated language mixing scripts/odd spacing around brand names [4, low]

### benign (negative weights; protect against false positives)
45. Genuine bank debit/credit alert (no link, masked account, "Avl bal") [−20]
46. OTP message with "do not share with anyone", no link, from standard sender format [−25]
47. Delivery OTP from known courier, no link [−10]
48. Plain personal conversation without links/money asks [−10]

## 5. Brands file (`brands.json`) — structure

```json
[
  {
    "id": "sbi",
    "names": ["sbi", "state bank of india", "yono"],
    "officialDomains": ["sbi.co.in", "onlinesbi.sbi", "sbi.bank.in"],
    "senderIdHints": ["SBI", "SBIINB", "SBIPSG"],
    "category": "bank"
  }
]
```
Seed with major banks (SBI, HDFC, ICICI, Axis, PNB, BoB, Kotak), UPI apps (PhonePe, Google Pay, Paytm, BHIM), couriers (India Post, Delhi­very, Blue Dart, DHL, FedEx), telecom (Jio, Airtel, Vi, BSNL), government (Income Tax, UIDAI, EPFO, Parivahan), e-commerce (Amazon, Flipkart). **Official domains must be verified from each brand's own site before adding**, and each entry should cite its source in a comment field.

## 6. Advice keys (`advice.json`)

`dont_click_links`, `dont_share_otp_pin`, `dont_pay_fees`, `hang_up_digital_arrest`, `call_bank_official`, `block_card_now`, `call_1930`, `report_cybercrime_portal`, `report_chakshu`, `uninstall_remote_app`, `dont_install_apk`, `dont_scan_qr_to_receive`, `tell_family`, `verify_via_official_app`. Each has `title`, `body`, and optional `action` (`tel:1930`, allowlisted URL), localized.

## 7. Contribution flow (for CONTRIBUTING.md)

1. Fork → add or edit a rule in the right category file.
2. Include ≥ 1 positive and ≥ 1 negative example (anonymize personal data!).
3. Run `pnpm rules:validate` and `pnpm eval`.
4. Open a PR using the template; CI validates schema, ReDoS safety, examples, and corpus regression.
5. Maintainer review checklist: real-world evidence, low false-positive risk, clear plain-language explanation, translation quality.
