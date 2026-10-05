# ScamShield — Antigravity Build Prompts

## How to use this (read first)

1. Create an empty folder `scamshield/`, open it in Antigravity.
2. Create `docs/` and copy all the documents from this pack into it (`01-PRD.md` … `06-ROADMAP-AND-OSS-DOCS.md`).
3. Save the **AGENTS.md** block (section A below) as `AGENTS.md` in the repo root. Antigravity reads project-level rule files; if your version uses a different location for rules (for example a `.agent/rules` folder or `GEMINI.md`), put the same content there. Check Antigravity's current docs for the exact name.
4. Paste the **Master Prompt** (section B) into the Agent Manager.
5. Then feed the **Stage Prompts** (section C) one at a time. Review and test between stages; don't let the agent do everything in one go.
6. Use the **Review Prompts** (section D) to polish.

Tip: choose the strongest available model for Stages 1–2 (engine logic) and any model for UI tweaks. Ask the agent to produce an implementation plan first and approve it before it writes code.

---

## A. AGENTS.md (project rules)

```markdown
# AGENTS.md — ScamShield

## Mission
Build ScamShield: a free, open-source, fully offline Android app (Expo/React Native) and shared TypeScript engine that checks messages/links for scam signals, tailored to India. Privacy is the core feature.

## Source of truth
Read these before coding, and follow them over assumptions:
- docs/01-PRD.md (what and why)
- docs/02-TECH-SPEC.md (architecture, engine, scoring)
- docs/03-DESIGN-SPEC.md (UI, tokens, copy)
- docs/04-RULES-SCHEMA-AND-STARTER-RULES.md (rule format, taxonomy, starter rules)
- docs/05-THREAT-MODEL-AND-PRIVACY.md (security constraints)
- docs/06-ROADMAP-AND-OSS-DOCS.md (roadmap, OSS docs)

## Hard constraints (never violate)
1. No network access in the app. Do NOT add the INTERNET permission to the release build. No analytics, ads, crash reporters, Firebase, or Google Play Services.
2. All analysis happens on-device in packages/engine (pure TypeScript, no platform deps).
3. Never log or persist user message text unless the user enabled History.
4. Never auto-open links. Outbound actions only on explicit user tap, to an allowlist (tel:1930, cybercrime.gov.in, sancharsaathi.gov.in).
5. Results are "risk indicators", never guarantees. Always show the disclaimer. Never claim "Safe" without a caveat.
6. Rule weights and text live in JSON, not hard-coded in engine logic.
7. Every rule needs positive and negative examples; CI must run them.
8. TypeScript strict mode. No `any` without a comment explaining why.

## Conventions
- pnpm workspaces monorepo as in docs/02-TECH-SPEC.md.
- Conventional Commits. Small, reviewable commits.
- Tests alongside code (Vitest for engine). Engine coverage target ≥ 90%.
- UI strings go through i18n keys (en + hi). No hard-coded user-facing strings.
- Follow design tokens from docs/03-DESIGN-SPEC.md; support light and dark mode; respect reduce-motion and font scaling.

## Working style
- Before each stage: write a short implementation plan and list files to create.
- After each stage: run typecheck, lint, tests; summarize what changed and what to verify manually.
- If a requirement is ambiguous, choose the option in the docs; otherwise pick the simplest and note it in docs/DECISIONS.md.
- Don't invent facts about Indian institutions, helplines, or brand domains. Use only those in the docs; mark anything uncertain with TODO(verify).
```

---

## B. MASTER PROMPT (paste into Antigravity Agent Manager)

```text
You are the lead engineer building ScamShield, an open-source, fully offline scam-checking app for India. The repository contains AGENTS.md and a docs/ folder with a complete PRD, technical spec, design spec, rules schema, threat model, and roadmap. Read ALL of them first and treat them as the source of truth.

GOAL
Deliver a polished, production-quality v0.1 that:
- lets a user paste text or share it from WhatsApp/SMS/Chrome (Android share target) and instantly see a verdict (Safe / Suspicious / Scam), a 0–100 score, plain-language reasons with the matched text highlighted, link analysis, and clear next steps (call 1930, cybercrime.gov.in);
- runs 100% on-device with NO INTERNET permission in the release build;
- uses a pure-TypeScript engine in packages/engine driven by JSON rules in packages/rules, with 40+ starter rules each carrying positive/negative examples;
- looks beautiful, calm, and trustworthy per docs/03-DESIGN-SPEC.md (light + dark mode, smooth verdict animation, accessible, English + Hindi);
- ships with CI (typecheck, lint, rule validation, tests, corpus evaluation) and a release workflow that builds a signed APK, verifies the manifest has no INTERNET permission, and attaches checksums to a GitHub Release.

PROCESS (important)
1. Do NOT write code yet. First produce an implementation plan: monorepo layout, dependency list with justification (keep minimal), milestones mapped to the stages below, risks, and any questions. Save it to docs/IMPLEMENTATION_PLAN.md and wait for my approval.
2. After approval, work stage by stage (I will send each stage prompt). At the end of each stage, run typecheck/lint/tests, and give me a summary, a manual test checklist, and screenshots or recordings of the app where applicable using the browser/emulator tools.
3. Prefer simple, well-tested code over cleverness. Keep dependencies minimal and pinned. No telemetry of any kind.

QUALITY BAR
- Engine: deterministic, fast (<150 ms for 5,000 chars), ReDoS-safe, thoroughly unit-tested.
- UI: feels like a premium safety product — generous spacing, large readable type, strong visual hierarchy, subtle motion, haptics, zero clutter.
- Copy: warm, plain language (Class 8 reading level), never blames the user.
- Everything verifiable: tests, CI, reproducible build instructions.

Begin with step 1 (the implementation plan) now.
```

---

## C. STAGE PROMPTS (send one at a time after approving the plan)

### Stage 1 — Monorepo, rules package, CI

```text
Stage 1: Foundation.
- Scaffold the pnpm monorepo exactly as in docs/02-TECH-SPEC.md (apps/mobile placeholder, packages/engine, packages/rules, packages/i18n, packages/ui-tokens, tools/, corpus/).
- Configure TypeScript strict, ESLint, Prettier, Vitest.
- In packages/rules: add the JSON Schema from docs/04, a validator script (`pnpm rules:validate`) that checks schema, unique ids, mandatory positive/negative examples, regex length limits, and ReDoS safety (use a safe-regex checker), and that negative weights appear only in benign.json.
- Create the data folder with empty-but-valid category files, brands.json, shorteners.json, risky_tlds.json, advice.json, meta.json.
- Add .github/workflows/ci.yml running typecheck, lint, rules:validate, and tests.
- Add LICENSE (GPL-3.0 for code, note CC0 for rules data), README skeleton, CONTRIBUTING, SECURITY, CODE_OF_CONDUCT, PRIVACY, PR/issue templates from docs/06 and docs/05.
Deliver: passing CI locally, summary, and checklist.
```

### Stage 2 — Detection engine + rules

```text
Stage 2: Engine and rules.
- Implement packages/engine per docs/02-TECH-SPEC.md: normalization (NFKC, zero-width removal, common obfuscation mapping), extraction (URLs, phone numbers, UPI IDs, amounts, OTP-like codes), rule matching with character spans, link heuristics (shorteners, punycode, IP host, risky TLD as soft signal, lookalike brand detection via edit distance and brand-token logic, many subdomains, @ trick, no-https, keyword-in-host) using a public-suffix helper for registrable domains, scoring with diminishing returns per category, combo bonuses, hard-override floors, benign dampeners, verdict thresholds from a config file, and advice-key collection.
- Export the typed `analyze()` API exactly as specified.
- Author ALL 48 starter rules listed in docs/04 section 4 as JSON in the category files, each with en + hi title/explanation (use accurate, simple Hindi), 1+ positive and 1+ negative example, and sources where you are confident; otherwise TODO(verify).
- Seed brands.json with major Indian banks, UPI apps, couriers, telecoms, and government bodies. Do NOT guess official domains: only include ones you are certain of, and mark the rest TODO(verify).
- Build a labelled corpus (≥ 60 synthetic scam messages across categories and ≥ 60 benign messages: bank alerts, OTPs, delivery updates, personal chats). Write tools/eval-corpus.ts that prints recall, false-positive rate, and per-category confusion, and fails CI if recall < 90% or FPR > 5%.
- Write unit tests (≥ 90% coverage for the engine).
Deliver: test + eval results, list of any rules you weakened or struggled with, and suggestions for improvement.
```

### Stage 3 — Mobile app UI

```text
Stage 3: Mobile app.
- Create apps/mobile with Expo + Expo Router + TypeScript, consuming packages/engine, packages/rules, packages/i18n, packages/ui-tokens.
- Implement the design system from docs/03-DESIGN-SPEC.md: tokens, light/dark themes, Inter + Noto Sans Devanagari bundled locally, 48dp touch targets, reduce-motion support.
- Build screens: Onboarding (3 cards), Home, Result, Report ("What to do now"), Scam Library (list + detail, 12 scam types), Settings, About.
- Build components: VerdictHero (animated shield + score ring + haptics), SignalCard, HighlightedMessage (tap a highlight to jump to its signal), LinkCard (real domain emphasized), AdviceList, ReportActions (tel:1930, cybercrime.gov.in, sancharsaathi.gov.in only on tap), DisclaimerBanner, ShareWarning (generates a shareable text summary).
- Integrate Android share target with expo-share-intent (config plugin), handling cold and warm starts; shared text lands directly on Result.
- Clipboard is read only when the user taps Paste.
- i18n for English and Hindi with a language switcher; follow device locale by default.
- Opt-in encrypted History (off by default) with Clear all.
- Android config: allowBackup=false, no INTERNET permission in release, minSdk 26, adaptive icon (foreground/background/monochrome), splash.
- Make it genuinely beautiful: spacious layout, clear hierarchy, soft elevation, tinted verdict backgrounds, smooth staggered card animation, and polished empty/error states.
Deliver: running dev build instructions, screenshots of every screen in light/dark and Hindi, and an accessibility self-audit (TalkBack labels, contrast, 200% font).
```

### Stage 4 — Release pipeline and docs

```text
Stage 4: Release and docs.
- Add .github/workflows/release.yml: on tag v*, build the release APK (arm64 + universal), sign with a keystore from GitHub Secrets, FAIL the build if the merged manifest contains android.permission.INTERNET, generate SHA-256 checksums and a CycloneDX SBOM, and attach everything to the GitHub Release with auto-generated notes. Document how to create the keystore and add secrets.
- Add fastlane/metadata/android/ structure (title, short/full description, changelogs, screenshots placeholders) for F-Droid/IzzyOnDroid.
- Write the final README (using the template in docs/06) with badges, feature list, install + verify-checksum instructions, "Add a rule in 5 minutes", limitations, and report links. Generate docs/assets (logo SVG, feature graphic, social preview).
- Write CHANGELOG.md and tag-ready v0.1.0 notes.
- Add a minimal web demo page (apps/web, Vite) that uses the same engine, with PWA manifest, service worker, and Web Share Target, for GitHub Pages hosting.
Deliver: dry-run output of the release workflow, and a launch checklist status.
```

---

## D. REVIEW / POLISH PROMPTS (use after the stages)

**Security review**
```text
Act as a mobile security reviewer. Audit the repo against docs/05-THREAT-MODEL-AND-PRIVACY.md. Verify: no INTERNET permission, no network libraries or telemetry in dependencies, no logging of user text, backup disabled, share-intent can't trigger unwanted actions, outbound URLs are allowlisted, regexes are ReDoS-safe, CI actions are pinned by SHA. Output a findings table (severity, file, fix) and apply the fixes.
```

**False-positive hunt**
```text
Generate 150 new realistic benign Indian messages (bank credit/debit alerts, genuine OTPs, courier updates, telecom recharge, utility bills, EMI reminders, college/work notices, family chats in English/Hinglish/Hindi) and 100 new scam variants (obfuscated spelling, Hinglish, spacing tricks). Run them through the engine, report misclassifications, and propose rule or weight changes. Apply only changes that don't reduce overall recall.
```

**UI polish**
```text
Review every screen as a senior product designer. Check spacing consistency on the 4pt grid, typographic hierarchy, verdict color + icon + word pairing, motion timing, dark-mode contrast, Hindi text wrapping, 200% font scale, small-screen (360dp) layout, and empty/error states. List 15 improvements ranked by impact and implement the top 10.
```

**Hindi/translation QA**
```text
Review every Hindi string for correctness, tone, and simplicity (everyday Hindi, no heavy Sanskritized words, Class 8 reading level). Provide a table of original, suggested fix, and reason, and apply fixes. Flag anything needing native-speaker review.
```

---

## E. Things to verify yourself (don't skip)

- **Helplines and portals**: confirm 1930, cybercrime.gov.in, and sancharsaathi.gov.in on the official sites before release.
- **Brand official domains** in `brands.json`: verify each against the brand's own website.
- **Hindi copy**: have a native speaker skim it.
- **expo-share-intent** setup: it needs a development build (not Expo Go); follow its current docs.
- **Antigravity rule-file location**: check current docs for where project rules go.
- **Manifest check**: install the release APK and confirm in Android app info that no network permission is listed.
