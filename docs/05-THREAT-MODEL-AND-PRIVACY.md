# ScamShield — Threat Model & Privacy

## 1. Security and privacy posture

ScamShield's core promise: **your messages never leave your device.** Every design decision supports a promise that anyone can verify.

Verifiable claims:
1. The release APK declares **no INTERNET permission** (checked by CI and visible in the Android app info screen).
2. No analytics, ads, crash-reporting, or third-party SDKs that communicate externally.
3. Source code is public; releases are signed, with published SHA-256 checksums and build instructions.
4. History is **off by default**, stored locally, and deletable in one tap.

## 2. Assets

| Asset | Sensitivity |
|---|---|
| Pasted/shared message text (may contain names, account fragments, OTPs) | High |
| Link contents | Medium |
| Saved history (opt-in) | High |
| Rules data (public) | Low, but integrity matters |
| Release signing key | Critical |
| Maintainer GitHub account | Critical |

## 3. Actors

- **Scammers** (want the app to under-detect, or to fool users into false safety).
- **Malicious contributors** (poison rules to cause false negatives/positives, or ReDoS the app).
- **Malware on the device** (reads clipboard/history).
- **Supply-chain attackers** (compromised dependency or CI).
- **Impersonators** (fake "ScamShield" APKs).

## 4. STRIDE-style analysis

| Threat | Example | Mitigation |
|---|---|---|
| **Spoofing** | Fake ScamShield APK with malware | Publish official repo + signing cert SHA-256 fingerprint in README; verify instructions; distribute only via GitHub Releases/F-Droid/Obtainium |
| **Tampering (rules)** | PR adds a rule that whitelists a scam pattern via negative weight | Negative weights only in `benign.json`, CODEOWNERS review, corpus regression gate in CI, two-maintainer approval for benign rules |
| **Tampering (build)** | Compromised dependency | Pinned lockfile, Dependabot, `pnpm audit`, minimal deps, GitHub Actions pinned by commit SHA |
| **Repudiation** | N/A (no accounts) | — |
| **Information disclosure** | Message text leaks via logs, screenshots, backups | No logging of input text; `FLAG_SECURE` optional on Result screen; disable Android auto-backup of history; encrypted storage |
| **Information disclosure (clipboard)** | App reads clipboard silently | Read clipboard only on explicit **Paste** tap |
| **Denial of service** | Catastrophic-backtracking regex; 1 MB input | Input cap 5,000 chars; ReDoS linting in CI; per-rule max length; analysis timeout guard |
| **Elevation of privilege** | Malicious deep link or share intent triggers unwanted actions | Share intent only populates text; never auto-opens URLs; outbound actions only to an allowlist via explicit tap |
| **False sense of safety** | User trusts "Looks safe" | Never "Safe!" without caveat; disclaimer always visible; advice to verify via official channels |

## 5. Abuse and misuse considerations

- **Evasion by scammers** (character substitution, spacing, images instead of text): normalization layer, and honest limits stated in the README. Screenshots/OCR not in v1.
- **Defamation risk**: do not publish blocklists naming individuals or small businesses. Domain/pattern rules must cite a public source; takedown/appeal process via GitHub issue.
- **Contributor data hygiene**: examples must be anonymized (replace real numbers/names with placeholders). PR template checkbox enforces this.

## 6. Android permissions (release)

| Permission | Status |
|---|---|
| INTERNET | **Absent** |
| READ_SMS / RECEIVE_SMS / READ_CALL_LOG / contacts | **Absent** |
| POST_NOTIFICATIONS | Absent |
| VIBRATE | Allowed (haptics) |
| Queries / intents | `ACTION_SEND text/plain` intent filter; `tel:` and `https:` via explicit user taps |

`allowBackup="false"`; `usesCleartextTraffic="false"`.

## 7. Privacy policy (draft — ship as `PRIVACY.md`)

> **ScamShield Privacy Policy**
>
> ScamShield analyzes text entirely on your device. We do not collect, transmit, store on any server, sell, or share your messages, links, or any personal data. The app has no account system, no analytics, no advertising, and does not request internet access.
>
> If you enable the optional History feature, results are stored only on your device in encrypted local storage. You can delete them any time from Settings, and uninstalling the app removes them.
>
> When you tap links such as "Report on cybercrime.gov.in" or call 1930, your phone opens the browser or dialer. Those actions are governed by those services' policies.
>
> ScamShield provides risk indicators for educational help and is not a substitute for official advice. Source code is public so you can verify these claims.
>
> Contact: open an issue at the project's GitHub repository.

## 8. Vulnerability disclosure (`SECURITY.md` draft)

- Report privately via GitHub Security Advisories ("Report a vulnerability") or the maintainer email listed in the repo.
- Acknowledge within 72 hours; aim to fix critical issues within 14 days.
- Safe-harbor language for good-faith researchers.
- Supported versions: latest release only.

## 9. Release integrity checklist

- [ ] Keystore stored only in GitHub Secrets; backup in an offline encrypted location
- [ ] Release workflow runs only on protected tags
- [ ] Manifest check: no INTERNET permission
- [ ] SHA-256 of each APK published in release notes
- [ ] Signing cert fingerprint documented in README
- [ ] SBOM generated (CycloneDX) and attached
- [ ] 2FA on maintainer GitHub account
