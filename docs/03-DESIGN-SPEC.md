# ScamShield — Design Specification

## 1. Design principles

1. **Calm authority.** This is a safety app used in a moment of fear. Never panicky, never gimmicky.
2. **One glance, one answer.** The verdict is the hero; reasons come second.
3. **Plain language.** Class 8 reading level; no jargon ("phishing" is always explained as "fake message that tricks you").
4. **Show, don't just tell.** Highlight the exact words and the real domain.
5. **Accessible by default.** Big touch targets, large text, high contrast, color + icon + word for every state.

## 2. Brand

- **Name:** ScamShield
- **Tagline:** "Check before you tap."
- **Logo:** Rounded shield with a subtle magnifier/check cut-out. Flat, works at 48px and as a monochrome adaptive icon.
- **Voice:** Warm, direct, reassuring. "This looks like a scam. Here's why." Never blame the user.

## 3. Color system (light / dark)

| Token | Light | Dark | Use |
|---|---|---|---|
| `bg` | `#F7F8FA` | `#0E1116` | App background |
| `surface` | `#FFFFFF` | `#161B22` | Cards |
| `surfaceAlt` | `#EEF1F5` | `#1E252E` | Input, chips |
| `text` | `#0F172A` | `#E6EDF3` | Primary text |
| `textMuted` | `#5B6475` | `#9AA7B5` | Secondary |
| `brand` | `#2563EB` | `#4C8DFF` | Primary actions |
| `safe` | `#16A34A` | `#3FB968` | Safe verdict |
| `safeBg` | `#E7F6EC` | `#10261A` | |
| `warn` | `#D97706` | `#F0A53A` | Suspicious |
| `warnBg` | `#FEF3E2` | `#2B1F0C` | |
| `danger` | `#DC2626` | `#FF5C5C` | Scam |
| `dangerBg` | `#FDECEC` | `#2E1213` | |
| `highlight` | `#FFE08A` (40% alpha) | `#FFD25A` (30% alpha) | Matched text spans |

Verdict states always pair color with an icon and a word (✓ Looks Safe / ! Be Careful / ✕ Likely Scam) for color-blind users.

## 4. Typography

- **Font:** Inter (UI) — or system font fallback; **Noto Sans Devanagari** for Hindi. Bundle fonts locally (no network).
- Scale: Display 32/40 · Title 22/28 · Body 17/24 (default; user-scalable) · Caption 13/18.
- Minimum body size 16sp; respect system font scale up to 200%.
- Monospace (JetBrains Mono or system mono) only for URLs/domains.

## 5. Spacing, shape, elevation

- 4pt grid; common spacing 8 / 12 / 16 / 24.
- Corner radius: cards 20, inputs 16, chips 999.
- Elevation: soft shadows in light mode; subtle borders (1px, 8% opacity) in dark mode.
- Touch targets ≥ 48dp.

## 6. Motion

- Verdict reveal: 600–800 ms, shield scales in, score ring animates from 0 to value, haptic (light for safe, medium for suspicious, heavy for scam).
- Signal cards stagger in (40 ms apart).
- Respect "Reduce motion": replace animations with fades.

## 7. Screens

### 7.1 Home
- Top: small logo + settings icon.
- Headline: "Got a suspicious message?"
- Large multiline input card with placeholder "Paste the message here…"; **Paste** button (reads clipboard on tap only), **Clear**.
- Primary button: **Check now** (full width, brand color).
- Secondary row: "Check a link" · "Scam Library".
- Below: a gentle tip card ("Tip: Share a message straight from WhatsApp → ScamShield").
- Footer chip: "🔒 Works offline. Nothing leaves your phone."

### 7.2 Result
- **VerdictHero** (top ~35% of screen): tinted background by verdict, large shield icon, verdict title, score ring, one-line summary ("This message pretends to be your bank and asks you to share your OTP.").
- **Why we think so**: list of `SignalCard`s ordered by severity. Card = severity chip + title + one-sentence explanation + "Learn more" chevron.
- **Your message**: original text with highlighted spans; tapping a highlight scrolls to its signal.
- **Links found**: `LinkCard` with the real domain in bold mono, flags as chips, "Don't open this link" note.
- **What to do now**: numbered checklist; sticky bottom bar: **Call 1930** (primary for scam verdict) · **Report online**.
- **Share warning** button: creates a text summary (and optional image card) for family groups: "⚠️ ScamShield flagged this message as a likely scam: …"
- Disclaimer banner at the bottom.

### 7.3 Report ("What to do now")
- Sections: "If you haven't clicked/paid" · "If you shared an OTP/PIN" · "If you've lost money".
- Each with ordered steps; money lost → "Call 1930 immediately; the sooner, the better. Inform your bank and block the card."
- Buttons: Call 1930, Open cybercrime.gov.in, Open Sanchar Saathi.

### 7.4 Scam Library
- Grid/list of scam types with icon, name, and one-line description. Detail screen: "What it looks like" (example message), "Red flags", "What to do", "Real-life tip".

### 7.5 Settings
- Language, Theme (System/Light/Dark), Text size hint, History (off by default) + Clear all, About, Open-source licenses, "Contribute a rule" (opens GitHub), Privacy promise.

### 7.6 Onboarding (3 cards, skippable)
1. "Check any suspicious message in a second."
2. "Share from WhatsApp or SMS — no copy-paste needed."
3. "100% offline. No accounts. No ads. Open source."

## 8. Component states

- Input: empty, focused, filled, error (too long).
- Check button: disabled when empty; loading shimmer while analyzing.
- Empty states: friendly illustration + one line.
- Errors: never technical; "Something went wrong. Your message is safe — nothing was sent anywhere."

## 9. Copy guidelines (microcopy)

| Situation | Copy |
|---|---|
| Safe | "No scam signs found. Stay careful — if anything feels off, call the company on the number printed on your card." |
| Suspicious | "A few warning signs. Don't click or pay until you verify." |
| Scam | "This looks like a scam. Don't click, don't reply, don't pay." |
| Disclaimer | "ScamShield shows risk indicators, not guarantees." |
| Privacy | "Works offline. Nothing leaves your phone." |

Never say "Safe!" without a caveat; use "Looks safe".

## 10. Accessibility checklist

- All interactive elements labeled for TalkBack; verdict announced on arrival (`accessibilityLiveRegion`).
- Contrast ≥ 4.5:1 text, ≥ 3:1 UI components.
- Focus order logical; no gesture-only actions.
- Test at 200% font scale and with Hindi script.

## 11. Assets to generate
- App icon (adaptive, foreground/background/monochrome), splash, in-app shield illustrations (3 verdict states), scam-type icons (lucide), Play-style feature graphic for GitHub README, social preview image (1280×640), demo GIF.
