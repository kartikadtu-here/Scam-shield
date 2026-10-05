# ScamShield — Technical Specification

## 1. Architecture overview

Monorepo (pnpm workspaces). The detection engine is a **pure TypeScript package with zero platform dependencies**, so it runs identically in the mobile app, a PWA, Node CLI, and CI tests.

```
scamshield/
├─ apps/
│  ├─ mobile/            # Expo (React Native) Android app
│  └─ web/               # Vite + React PWA (phase 3)
├─ packages/
│  ├─ engine/            # pure TS: analyze(text, rules, options) -> Result
│  ├─ rules/             # JSON rule packs + JSON Schema + validator script
│  ├─ i18n/              # locale JSON (en, hi, ...) shared by apps
│  └─ ui-tokens/         # design tokens (colors, spacing, type) shared
├─ tools/
│  ├─ validate-rules.ts  # CI rule validation
│  └─ eval-corpus.ts     # runs engine on labelled corpus, prints metrics
├─ corpus/
│  ├─ scam/*.txt         # labelled scam samples (synthetic/anonymized)
│  └─ benign/*.txt       # real-looking legit messages (OTP, bank alerts)
├─ docs/                 # these documents
├─ .github/workflows/    # ci.yml, release.yml
├─ AGENTS.md
└─ README.md, LICENSE, CONTRIBUTING.md, SECURITY.md, CODE_OF_CONDUCT.md
```

## 2. Tech stack

| Layer | Choice | Reason |
|---|---|---|
| Language | TypeScript (strict) | Shared engine, type-safe rules |
| Mobile | Expo SDK (latest stable) + React Native + Expo Router | Fast, matches web skills, EAS/GitHub Actions builds |
| Share target | `expo-share-intent` (config plugin; requires a **development build**, not Expo Go) | Receives Android `ACTION_SEND` text |
| Styling | NativeWind or StyleSheet with shared tokens | Consistent theming, dark mode |
| Animation | `react-native-reanimated` | Smooth verdict reveal |
| Icons | `lucide-react-native` | Clean, consistent |
| i18n | `i18next` + `react-i18next` | Locale JSON, fallback chain |
| Storage | `expo-secure-store` / encrypted MMKV (history opt-in only) | Local-only |
| Tests | Vitest (engine), Jest + RNTL (UI), Maestro (e2e, optional) | |
| Lint/format | ESLint, Prettier, TypeScript `strict` | |
| CI | GitHub Actions | Validate rules, run tests, build APK |
| Web (later) | Vite + React + Workbox PWA with Web Share Target | Install-from-link distribution |

> Note: the INTERNET permission must be **absent** from the release AndroidManifest. In Expo, verify via a prebuild check in CI (`grep` the generated manifest). Dev builds may need it for Metro; the release workflow must strip it and fail the build if present.

## 3. Engine design

### 3.1 Public API

```ts
export type Verdict = 'safe' | 'suspicious' | 'scam';
export type Severity = 'low' | 'medium' | 'high' | 'critical';

export interface Signal {
  ruleId: string;            // e.g. "kyc-urgency-001"
  category: ScamCategory;    // e.g. "bank_kyc"
  severity: Severity;
  weight: number;            // contribution to score
  titleKey: string;          // i18n key
  explanationKey: string;    // i18n key
  spans: Array<[number, number]>; // char offsets matched in the input
}

export interface LinkReport {
  raw: string;
  host: string;
  registrableDomain: string;
  flags: LinkFlag[];         // 'shortener' | 'punycode' | 'lookalike:<brand>' | 'ip_host' | 'bad_tld' | 'no_https' | 'at_trick' | 'many_subdomains' | 'keyword_in_host'
  lookalikeOf?: string;
}

export interface AnalysisResult {
  verdict: Verdict;
  score: number;             // 0..100
  primaryCategory?: ScamCategory;
  signals: Signal[];
  links: LinkReport[];
  adviceKeys: string[];      // ordered, deduped
  engineVersion: string;
  rulesVersion: string;
}

export function analyze(input: string, opts?: AnalyzeOptions): AnalysisResult;
```

### 3.2 Pipeline

1. **Normalize**: Unicode NFKC, lowercase copy for matching (keep original for span offsets), collapse whitespace, map common obfuscations (`0`→`o`, `1`→`l`, zero-width chars removed, Devanagari digits → ASCII where relevant).
2. **Extract**: URLs (robust regex + `URL` parsing guarded in try/catch), phone numbers, UPI handles (`name@bank`), amounts, OTP-like 4–8 digit codes.
3. **Rule matching**: for each rule, test `patterns` (regex), `keywords_all` / `keywords_any` (token sets), and optional `requires` / `excludes` conditions. Record spans.
4. **Link analysis**: per URL, run heuristics; look up `brands.json` for lookalike detection (edit distance ≤ 2 on the registrable label, or brand token embedded in a non-official domain).
5. **Scoring**: see 3.3.
6. **Benign dampeners**: legitimate patterns (e.g., OTP message containing "do not share" from a known sender ID format, with no link) reduce score via negative-weight rules.
7. **Verdict + advice**: map score to verdict; collect advice keys by category with priority; dedupe.

### 3.3 Scoring model

- Each matched rule contributes `weight` (1–40). Same-category signals are combined with diminishing returns to prevent keyword-stuffing from dominating:
  `categoryScore = Σ weight_i × 0.7^(i)` (sorted descending, i from 0).
- `score = min(100, Σ categoryScore + comboBonus − dampeners)`.
- **Combo bonuses**: urgency + authority + link = +15; request for PIN/OTP/CVV = critical override (floor at 75); "pay a fee to receive money/prize/loan" = floor at 70.
- Hard overrides ("never legitimate" rules): any message asking to **share OTP/PIN/CVV/password**, or **install remote-access apps** (AnyDesk/TeamViewer/QuickSupport) → floor 80.
- Thresholds in `engine.config.ts`: Safe < 30, Suspicious 30–64, Scam ≥ 65.
- All weights live in rule JSON, never hard-coded in engine logic.

### 3.4 Link heuristics (details)

| Flag | Test | Weight hint |
|---|---|---|
| shortener | host in `shorteners.json` (bit.ly, tinyurl, cutt.ly, t.co, etc.) | +10 |
| punycode | host contains `xn--` | +20 |
| ip_host | host is IPv4/IPv6 literal | +25 |
| bad_tld | TLD in curated risky list (`.xyz`, `.top`, `.click`, `.icu`, `.live`, `.cfd`, …) — soft signal | +8 |
| lookalike | label within edit distance 2 of a brand, or brand token + extra words (`sbi-kyc-update.xyz`) on non-official registrable domain | +35 |
| many_subdomains | > 3 labels before registrable domain | +8 |
| at_trick | `@` in authority part | +25 |
| no_https | scheme http | +5 |
| keyword_in_host | `kyc`, `verify`, `update`, `reward`, `claim`, `refund`, `secure`, `login` in host | +10 |

Use a bundled **public suffix list subset** (or `tldts`) to compute the registrable domain correctly (e.g., `co.in`).

### 3.4.1 Caveat on TLD signals
TLD reputation is a soft hint only; never produce a "Scam" verdict from TLD alone.

## 4. Data model (rules)

See `04-RULES-SCHEMA.md`. Key files in `packages/rules/data/`:

- `rules/*.json` — one file per category (bank_kyc.json, utility_cut.json, parcel_customs.json, digital_arrest.json, upi_collect.json, job_task.json, loan_app.json, lottery_prize.json, tech_support.json, customer_care_fake.json, investment_crypto.json, romance_sextortion.json, otp_phishing.json, benign.json)
- `brands.json` — brand names + official registrable domains + sender-ID prefixes
- `shorteners.json`, `risky_tlds.json`
- `advice.json` — advice keys → structure per category
- `meta.json` — `rulesVersion`, `updatedAt`

## 5. Mobile app architecture

### 5.1 Navigation (Expo Router)
```
app/
├─ _layout.tsx            # theme provider, i18n init, share-intent listener
├─ index.tsx              # Home (paste/check)
├─ result.tsx             # Result screen (param: text or id)
├─ library/index.tsx      # Scam Library list
├─ library/[id].tsx       # Scam type detail
├─ report.tsx             # "What to do now" + reporting links
├─ history.tsx            # opt-in history
├─ settings.tsx           # language, theme, history toggle, about, licenses
└─ about.tsx              # privacy promise, open-source, how to contribute
```

### 5.2 State
- Local React state + lightweight store (Zustand) for settings/history. No server state.

### 5.3 Share intent flow
1. `useShareIntent()` in root layout detects `shareIntent.text`.
2. Navigate to `/result` with the text; clear the intent after consuming.
3. Handle cold start and warm start.

### 5.4 Key components
- `VerdictHero` (animated shield + verdict + score ring)
- `SignalCard` (severity chip, title, explanation, expand for details)
- `HighlightedMessage` (renders original text with matched spans highlighted)
- `LinkCard` (real domain emphasized, flags as chips, "Do not open" note)
- `AdviceList` + `ReportActions` (tap-to-call 1930, open cybercrime.gov.in)
- `ScamTypeCard` (library)
- `DisclaimerBanner`

## 6. Build and release

- **CI (`ci.yml`)**: install → typecheck → lint → validate rules → engine tests → eval corpus (fail if recall/FPR thresholds regress) → mobile unit tests.
- **Release (`release.yml`)** on tag `v*`: build release APK (arm64-v8a + universal) via Gradle/EAS local build on the runner, sign with keystore from GitHub Secrets, verify **no INTERNET permission**, compute SHA-256, attach to GitHub Release with changelog.
- Document F-Droid metadata (fastlane structure) for later submission; keep build free of proprietary blobs (no Google Play Services dependency, no Firebase).
- Obtainium compatibility: stable release asset naming `scamshield-vX.Y.Z-arm64.apk`.

## 7. Testing strategy

- **Rules**: schema validation + each rule must include `examples.positive` (must match) and `examples.negative` (must not match); CI executes them.
- **Engine unit tests**: normalization, extraction, scoring math, overrides, dampeners.
- **Golden corpus**: `corpus/scam` and `corpus/benign` with expected verdicts in a manifest; track recall and false-positive rate.
- **UI**: snapshot + interaction tests for Home → Result; accessibility checks (labels, contrast).
- **Manual**: share from WhatsApp, SMS, Chrome; cold/warm start; airplane mode; Hindi; large font.

## 8. Performance budget

- Precompile regexes once at startup; cache per rules version.
- Avoid catastrophic backtracking: lint rule regexes with `safe-regex2`/`recheck` in CI; enforce max pattern length.
- Run analysis off the UI thread if > 50 ms (InteractionManager / worklet-free simple deferral).

## 9. Security considerations (technical)

- Treat input as untrusted; never `eval`, never render as HTML.
- Do not auto-open links; opening reporting URLs only via explicit user tap, to a hard-coded allowlist.
- Dependencies: minimal, pinned, `pnpm audit` + Dependabot.
- Release signing key stored only in GitHub Secrets; publish checksums.
