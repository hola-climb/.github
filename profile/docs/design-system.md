# Hola Design System

> Climbing analysis, route logging, and gym exploration — for mobile.

Hola is a mobile-first app combining **AI climbing analysis**, **route logging**, and **climbing gym exploration**. The system feels calm and athletic — neutral off-white canvas, vivid "climbing-hold" accents (pink, lime, orange, cyan), and a single primary action per screen. The aesthetic reference is *Apple Fitness × Linear × modern climbing-gym branding*.

This folder is the source of truth for visual identity, foundational tokens, components, and UI kits used to design and prototype the product.

---

## Sources & provenance

| Source | Path | Notes |
| --- | --- | --- |
| Brand brief | pasted into project init | Core copy doc; defines colors, type, components, motion, AI experience |
| Asset library reference | `uploads/assets_v2.png` | "20 types" sheet of holds, volumes, markers, icons. **Note:** the sheet is titled "Olla." but the brief calls the product "Hola." We've used **Hola** throughout; ask the user to reconcile. |
| Concept board | `uploads/hola_concept.png` | Mood / type / palette / chips / buttons / phone frame mock |
| Codebase | _not provided_ | Brief mentions Vue 3 + Ionic + Capacitor + PWA. No repo attached — UI kit recreations are rebuilt in React from the brief & concept board. If you have the repo, attach it via the Import menu so we can pixel-match. |

---

## Index

| File | Purpose |
| --- | --- |
| `README.md` | This file — context, content & visual foundations, iconography, index |
| `SKILL.md` | Agent-Skill manifest for use in Claude Code |
| `colors_and_type.css` | All CSS variables — colors, fonts, type scale, semantic tokens |
| `fonts/` | Outfit + Noto Sans KR (Google Fonts — see typography note) |
| `assets/` | Logos, hold imagery, illustrations, icon set |
| `preview/` | Design-system preview cards (palette, type, chips, buttons, etc.) |
| `ui_kits/hola_app/` | React recreation of the mobile app — 5 core screens & components |

---

## Content fundamentals

**Voice.** Calm, motivating, human. Hola talks to a climber the way a friend who climbs would — confident but never coach-y, never gamified.

**Casing.**
- Headlines & titles → Title Case or Sentence case, no period
- Section labels, chips, tabs → `UPPERCASE`, tracked +0.02em (e.g. `SESSION`, `RECORDS`, `V4 HARD`)
- Body / inline → Sentence case
- Stat callouts → numeric + lowercase unit (e.g. `12 sends`, `42 min`)

**Person.** Mostly *you* / *your* (`Your last session`, `Ready when you are`). System narration is third-person and brief (`AI is analyzing your climb`).

**Density.** Short. One idea per line. A screen has **one** primary verb (Upload, Analyze, Log, Explore). Body copy rarely exceeds two lines per card.

**Examples — good:**
- `Ready when you are.`
- `Upload a clip — we'll find the beta.`
- `Nice send. V4 logged.`
- `Analyzing your movement…`
- `No routes yet. Tap the + to log one.`

**Examples — avoid:**
- ❌ `Welcome back, climber! 🧗‍♂️ Let's crush some new heights today!!` *(over-gamified, emoji-heavy)*
- ❌ `Your performance analytics dashboard` *(enterprise tone)*
- ❌ `INITIATING NEURAL ANALYSIS…` *(sci-fi, intimidating)*

**Emoji.** Almost never. The brand's visual punctuation is hold-shaped color, not emoji. Exception: a single 🧗 or ✨ might appear in a marketing surface — never in product.

**AI voice.** Soft, present-tense, low-confidence-friendly. `We think this looks like a V4.` Not `We have determined…`. Never robotic, never mystical.

**Korean parity.** All product copy must be writable in Korean (Noto Sans KR). Keep sentences short — they tend to expand in translation.

---

## Visual foundations

### Palette
A warm off-white canvas with five vivid "climbing-hold" accents and one near-black. Only **one** strong accent per section; the rest of the surface stays neutral. Soft, blurred ambient color blobs sit behind hero cards for warmth.

| Token | Hex | Role |
| --- | --- | --- |
| `--bg` | `#F7F7F5` | App canvas — warm off-white |
| `--surface` | `#FFFFFF` | Cards |
| `--surface-soft` | `#F0F1F4` | Sunken / muted card |
| `--fg` | `#151515` | Primary text & Dark Nordic accent |
| `--fg-muted` | `#8D96A8` | Secondary text, micro-labels |
| `--border` | `#E7EAF0` | Dividers, card outlines |
| `--hold-pink` | `#FF4D94` | Tape direction, hard route, urgent |
| `--hold-lime` | `#C8FF00` | Start, success, accent CTA |
| `--hold-orange` | `#FF9800` | Mid-grade tag, warm highlight |
| `--hold-cyan` | `#22D3EE` | Top marker, AI scan, info |
| `--hold-dark` | `#151515` | Primary button, dark accent |

Tints (used for chip backgrounds, glow halos) are the hold color at ~12–18% opacity over white.

### Typography
**Outfit** — geometric sans, three weights (500 Medium, 600 SemiBold, 800 ExtraBold). **Noto Sans KR** for Korean. Tight tracking, compact line-height on headlines; generous line-height (1.45) on body.

- `display` — Outfit ExtraBold, 40–56, tracking -0.02em, line-height 1.0
- `h1` — Outfit ExtraBold, 28, tracking -0.015em
- `h2` — Outfit SemiBold, 22
- `h3` — Outfit SemiBold, 17
- `body` — Outfit Medium, 15, line-height 1.45
- `caption` — Outfit Medium, 13, color `--fg-muted`
- `micro-label` — Outfit SemiBold, 11, UPPERCASE, tracking +0.08em, color `--fg-muted`

Avoid serifs, condensed cuts, decorative type.

### Spacing & radii
Scale: **4 / 8 / 12 / 16 / 24 / 32 / 48**. Cards breathe; never stack two ≤8 gaps in a row.

| Element | Radius |
| --- | --- |
| Cards | **24px** |
| Buttons | **16px** |
| Inputs | **16px** |
| Chips / pills | **999px** |
| Modals / sheets | 28px top corners |

### Backgrounds
- App canvas is flat `--bg`. No textures, no patterns, no full-bleed imagery in product chrome.
- Hero cards may sit on a soft **ambient blob** — a 600px Gaussian-blurred radial of one hold color at 14% opacity, positioned off-screen behind a card. Extremely subtle — should read as ambient lighting, not a gradient.
- Photography (gym walls, routes) is warm, naturally-lit, never desaturated. Slightly grainy is fine; cold/cyan grading is not.

### Shadows / elevation
Soft, low, neutral — never colored, never deep.

- `--shadow-card` → `0 1px 2px rgba(20, 22, 28, 0.04), 0 8px 24px rgba(20, 22, 28, 0.06)`
- `--shadow-float` → `0 2px 6px rgba(20, 22, 28, 0.05), 0 20px 48px rgba(20, 22, 28, 0.10)` (FAB, sheets)
- `--shadow-pressed` → `0 1px 0 rgba(20, 22, 28, 0.06) inset`

Inset shadows are rare — only on pressed primary buttons.

### Borders
- Hairline `1px solid var(--border)` on neutral cards
- Accent borders are **never** decorative — only as the active-state ring on chips/tabs (1.5px in the chip's hold color)

### Cards
White surface, **24px radius**, 1px border `--border`, `--shadow-card`. Inner padding 20–24px. Section-header cards sometimes drop the border and rely on shadow alone.

### Motion
Spring-based, short, restrained.

- **Easing**: `cubic-bezier(0.22, 1, 0.36, 1)` (soft-out) for entrances; `cubic-bezier(0.4, 0, 0.2, 1)` for state changes.
- **Durations**: 150ms (hover/press), 220ms (state), 320ms (sheet/route)
- **Patterns**: fade + 8px translateY for entrances; fade + scale 0.98→1 for cards; no bounce, no rotation.
- AI states animate via a slow **breathing dot** (1400ms ease-in-out, opacity 0.3↔1), not a spinner.

### Hover & press states
- **Hover** (web/desktop preview only): background tint shifts by 4%, no scale.
- **Press** (mobile, primary): `transform: scale(0.97)`, 120ms.
- **Press** (chip / secondary): background darkens by 6%, no scale.
- **Disabled**: opacity 0.4, no other treatment.

### Transparency & blur
- Used sparingly: only the **bottom nav bar** uses a `backdrop-filter: blur(20px) saturate(140%)` over a 78%-white tint, so content scrolls subtly beneath it.
- No frosted modals, no card-on-card blur.

### Layout rules
- Mobile design width: **390px** (iPhone 14/15 standard)
- Safe area top: 56px (status bar + breathing room)
- Bottom nav: 84px including safe-area inset
- Horizontal page padding: **20px**
- Section vertical rhythm: **32px between sections**, 16px between cards within a section
- One primary action per screen — typically the FAB (Upload) or a full-width button at the bottom.

### Imagery
Warm, natural light. Photos of holds, walls, climbers — never stock-y, never desaturated, never with heavy filters. When imagery is unavailable, use a **placeholder card** in `--surface-soft` with a centered hold-shape silhouette in 8% opacity.

---

## Iconography

**System.** Thin, line-based icons, **1.5px stroke**, rounded caps & joins, ~24px viewBox. Athletic, lightweight, not clinical.

**Set used.** **Lucide** (via CDN) is the substitute set — closest match for the brief's "thin modern line icons, slightly rounded stroke edges." Loaded from `https://unpkg.com/lucide-static`. Flagged: if the actual product ships a custom icon font, drop the SVGs into `assets/icons/` and we'll swap.

**Brand icons.** Five "hold" shapes (pink / lime / orange / cyan / dark nordic) and the climbing-specific markers (start `S`, top `T`, tape arrow, route tag `V4 HARD`) live in `assets/holds/` and `assets/markers/` as SVGs. These are **brand**, not utility — use them for empty states, hero illustrations, and feature ornament. Never substitute Lucide for these.

**Emoji.** Effectively never. The hold-color system is the brand's visual punctuation. The only acceptable exception is marketing copy outside the app shell.

**Unicode glyphs.** Used only for typographic content (×, →, ✓ in body copy). Never as UI controls — those use SVG.

**Usage.**
- Tab bar icons: 24px, stroke 1.75, color `--fg-muted` (inactive) / `--fg` (active)
- Inline icons in chips: 14px, stroke 2
- Hero / empty-state illustrations: brand hold imagery, 96–160px

---

## Caveats & open questions

1. **Brand name conflict.** Asset sheet says "Olla."; brief says "Hola." We went with **Hola**. Confirm.
2. **No codebase attached.** Brief mentions Vue 3 + Ionic + Capacitor, but no repo was imported. Components in `ui_kits/hola_app/` are rebuilt from the brief — they may not pixel-match the production app. Attach the repo if you want a 1:1 recreation.
3. **No real product screens.** Aside from the single phone wireframe in `hola_concept.png`, we have no live screens. The five sample screens in the UI kit (Feed, Records, Upload, Explore, My Page) are designed from the brief.
4. **Fonts.** Outfit + Noto Sans KR are loaded via Google Fonts. If you have licensed/weight-specific files, drop them into `fonts/` and we'll switch from the CDN.
5. **Iconography substitute.** Using Lucide CDN. Confirm or swap.
