# ScamShield — Roadmap, Distribution & Open-Source Docs

## 1. Build roadmap

### Phase 0 — Foundation (Weekend 1, first half)
- [ ] Monorepo with pnpm workspaces, TypeScript strict, ESLint/Prettier, Vitest
- [ ] `packages/engine` skeleton + types
- [ ] `packages/rules` with schema + validator script
- [ ] CI workflow (typecheck, lint, validate rules, tests)

### Phase 1 — Engine MVP (Weekend 1, second half)
- [ ] Normalization + extraction (URLs, phones, UPI IDs)
- [ ] Rule matcher with spans
- [ ] Scoring with diminishing returns, combos, overrides, dampeners
- [ ] 30–50 starter rules with examples
- [ ] Corpus (≥ 40 scam, ≥ 40 benign) + `eval` script

### Phase 2 — Mobile app (Weekend 2)
- [ ] Expo app with design tokens, light/dark theme
- [ ] Home, Result, Report screens
- [ ] Share-target integration (dev build)
- [ ] Highlighted message + signal cards + disclaimer
- [ ] English strings via i18n

### Phase 3 — Polish & release (Weekend 3)
- [ ] Link analyzer + `LinkCard`
- [ ] Hindi translation, Scam Library
- [ ] Onboarding, haptics, animations, accessibility pass
- [ ] Release workflow → signed APK on GitHub Releases
- [ ] README with demo GIF, screenshots, "How to add a rule"

### Phase 4 — Community & reach
- [ ] PWA build (Web Share Target) hosted on GitHub Pages
- [ ] F-Droid submission (fastlane metadata)
- [ ] Obtainium instructions
- [ ] More languages (Assamese, Bengali, Tamil, Telugu, Marathi)
- [ ] Good-first-issue labels, contributor docs, rule bounty list
- [ ] Optional on-device ML classifier as an assistive signal

## 2. Distribution without Play Console

| Channel | Notes |
|---|---|
| **GitHub Releases** | Signed APK + checksums; primary channel |
| **Obtainium** | Users add repo URL; auto-updates from Releases |
| **F-Droid** | Needs fully FOSS dependencies, no Google services; submit metadata via fdroiddata merge request |
| **PWA** | Install from browser, works offline via service worker; Web Share Target works on Android Chrome for installed PWAs |
| **IzzyOnDroid** | F-Droid-compatible repo that accepts GitHub-hosted APKs faster |

Include in README: "How to install an APK safely" (enable install from browser once, verify checksum).

## 3. README.md template

```markdown
<p align="center"><img src="docs/assets/logo.png" width="96" /></p>
<h1 align="center">ScamShield</h1>
<p align="center"><b>Check before you tap.</b><br/>Offline, open-source scam checker for India.</p>

<p align="center">
  <a href="releases"><img alt="release" src="https://img.shields.io/github/v/release/OWNER/scamshield"/></a>
  <img alt="license" src="https://img.shields.io/badge/license-GPL--3.0-blue"/>
  <img alt="no internet" src="https://img.shields.io/badge/permissions-no%20internet-brightgreen"/>
</p>

## Why
(2-3 sentences on the problem and the privacy promise)

## Features
- Paste or share a message → verdict + plain-language reasons
- Link analyzer (lookalike domains, shorteners, punycode)
- What-to-do-next guide (1930, cybercrime.gov.in)
- 100% offline • no accounts • no ads • open rules

## Screenshots / Demo GIF

## Install
- GitHub Releases (APK) • Obtainium • F-Droid (soon) • Web (PWA)
- Verify: `sha256sum scamshield-vX.Y.Z-arm64.apk` and compare with the release notes.
- Signing certificate SHA-256: `XX:XX:...`

## How it works
(Short diagram: input → normalize → rules → score → verdict)

## Add a scam rule in 5 minutes
(Link to CONTRIBUTING.md, show a 15-line JSON example)

## Roadmap
## Limitations (be honest)
ScamShield shows risk indicators, not guarantees. It can miss scams and can flag genuine messages. When in doubt, call your bank on the number printed on your card.

## Report a scam
1930 • cybercrime.gov.in

## License
App: GPL-3.0 • Rules data: CC0-1.0
```

## 4. CONTRIBUTING.md outline

1. Ways to help (rules, translations, bug reports, UI, docs)
2. Dev setup (`pnpm i`, `pnpm test`, `pnpm mobile:dev`)
3. **Adding a rule** (step-by-step with example and checklist)
4. **Adding a translation** (locale file, how to test in app)
5. Anonymization rule for examples
6. Commit style (Conventional Commits) and PR checklist
7. Code of Conduct reference and review expectations (response within ~7 days)

## 5. PR template (`.github/pull_request_template.md`)

```markdown
## What does this PR do?
- [ ] New rule  - [ ] Rule fix  - [ ] Translation  - [ ] Code  - [ ] Docs

## Evidence (for rules)
Public source or description of how this scam appears in the wild:

## Checklist
- [ ] `pnpm rules:validate` passes
- [ ] `pnpm eval` shows no regression
- [ ] Examples are anonymized (no real names/numbers/accounts)
- [ ] Explanation is in plain language (Class 8 reading level)
- [ ] Includes at least one negative example to avoid false positives
```

## 6. Issue templates
- **Bug report** (device, Android version, steps, expected/actual)
- **Missed scam** ("this scam wasn't detected" — paste anonymized text)
- **False positive** ("genuine message flagged" — paste anonymized text)
- **Translation request / contribution**

## 7. Licensing recommendation

- **Code: GPL-3.0** (keeps forks open; F-Droid friendly). Alternative: Apache-2.0 if you want max adoption.
- **Rules data: CC0-1.0** so anyone (including other tools) can reuse the knowledge.
- Add `REUSE`-style headers or a license note per directory.

## 8. Labels & governance
- Labels: `good first issue`, `rule`, `translation`, `bug`, `false-positive`, `missed-scam`, `needs-evidence`, `help wanted`.
- `CODEOWNERS`: rules need maintainer review; `benign.json` needs two reviewers.
- Release cadence: rules pack monthly; app releases as needed.
- Maintain `CHANGELOG.md` (Keep a Changelog format).

## 9. Launch checklist
- [ ] Demo GIF (paste → verdict → advice) under 10 MB
- [ ] README top-fold clear in 5 seconds
- [ ] First release v0.1.0 with APK + checksums
- [ ] 10 "good first issue" tickets (e.g., "add 5 Hindi explanations")
- [ ] Post to r/developersIndia, r/opensource, Hacker News "Show HN", LinkedIn, X, relevant Telegram/Discord communities, college cyber clubs
- [ ] Write a short blog/LinkedIn post: "I built an offline scam checker for India — here's how it works"
- [ ] Submit to awesome-lists (awesome-android, awesome-privacy, FOSS India lists)
