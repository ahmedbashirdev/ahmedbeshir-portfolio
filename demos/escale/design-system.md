# Escalé — Design System

> Source of truth for visual language across all surfaces.
> When the Figma file diverges from this document, the document wins.

---

## 1. Brand Foundation

### Aesthetic Direction
**Editorial Hospitality with North African Soul.**

Picture the lobby of a thoughtfully restored Cairene townhouse, or a quiet Siwa retreat at golden hour. Calm, warm, considered. Editorial but never cold. Sophisticated but never intimidating. The design language must scale from a 3-star city hotel in Cairo to a luxury resort in Sharm without losing identity.

### Brand Pillars
- **Calm sophistication** — restraint over ornament
- **Warm hospitality** — earth tones, generous space
- **Bilingual by design** — AR and EN feel equally native, never bolted on
- **Editorial layouts** — magazine-grade typography and composition
- **Subtle Egyptian/North African motifs** — the three triangles from the logo as a recurring geometric signature

### What to AVOID
- Generic SaaS purple gradients
- Inter / Roboto / system-font fatigue
- Soulless "fintech blue" palettes
- Overused Material-style elevation
- Kitschy "Arabian Nights" decoration
- Boutique hotel cliché (too dark, too moody)

---

## 2. Color System

All colors as CSS variables. Tailwind config maps to these.

### Primary

| Token | Hex | Use |
|-------|-----|-----|
| `--brand-aubergine-900` | `#3A1234` | Display text, deep accents |
| `--brand-aubergine-700` | `#4A1942` | Primary buttons, navigation |
| `--brand-aubergine-500` | `#5B2C5F` | Default brand surface |
| `--brand-aubergine-300` | `#8E5A8E` | Secondary actions |
| `--brand-aubergine-100` | `#EFE3EF` | Tinted surfaces, badges |
| `--brand-magenta-500` | `#B8228C` | Accent CTA, focus states |
| `--brand-magenta-300` | `#D86CB0` | Hover states |

### Warm Accents

| Token | Hex | Use |
|-------|-----|-----|
| `--terracotta-700` | `#A05A3A` | Secondary buttons, illustrations |
| `--terracotta-500` | `#C9956B` | Highlight elements, icons |
| `--terracotta-300` | `#E2BC9C` | Tinted backgrounds |
| `--terracotta-100` | `#F5E8DC` | Soft surfaces |

### Neutrals (Warm)

| Token | Hex | Use |
|-------|-----|-----|
| `--ink-900` | `#1C1410` | Primary text |
| `--ink-700` | `#3D332D` | Body text |
| `--ink-500` | `#6B5D54` | Secondary text |
| `--ink-300` | `#A89A8E` | Tertiary, placeholders |
| `--sand-100` | `#F0E9DF` | Borders, dividers |
| `--sand-50` | `#F7F2E9` | Soft surfaces |
| `--cream` | `#FAF7F2` | Default page background |
| `--paper` | `#FDFBF7` | Cards on cream |

### Semantic

| Token | Hex | Use |
|-------|-----|-----|
| `--success` | `#5A8C5F` | Confirmations |
| `--warning` | `#C9956B` | Caution (reuses terracotta) |
| `--danger` | `#B53F3F` | Errors, destructive |
| `--info` | `#5B7BA8` | Informational |

### Dark mode (Phase 2)
Not in MVP. Will use a warm-charcoal foundation, not pure black.

---

## 3. Typography

### Type Stack

```css
--font-display: 'Fraunces', 'Newsreader', Georgia, serif;
--font-body: 'Geist', 'Söhne', -apple-system, sans-serif;
--font-arabic: 'Tajawal', 'IBM Plex Sans Arabic', sans-serif;
--font-mono: 'Geist Mono', 'IBM Plex Mono', monospace;
```

### Why these
- **Fraunces** — variable serif with warmth and editorial gravity. Optical sizes from soft (small) to confident (display). Distinctly *not* generic.
- **Geist** — Vercel's open-source sans. Modern, geometric, refined. Avoids Inter fatigue.
- **Tajawal** — Arabic sans that pairs cleanly with geometric Latin fonts. Excellent multi-weight support. More distinctive than Cairo or IBM Plex Sans Arabic.
- **Geist Mono** — for prices, codes, booking references.

### Type Scale

Modular scale with 1.250 ratio (Major Third).

| Token | Latin | Arabic | Use |
|-------|-------|--------|-----|
| `--text-display-2xl` | Fraunces 72/76 — opsz 144 | Tajawal 64/72 700 | Hero |
| `--text-display-xl` | Fraunces 56/62 — opsz 144 | Tajawal 52/60 700 | Section headers |
| `--text-display-lg` | Fraunces 44/52 — opsz 96 | Tajawal 40/48 700 | Page titles |
| `--text-display-md` | Fraunces 32/40 — opsz 72 | Tajawal 30/38 700 | Card titles |
| `--text-display-sm` | Fraunces 24/32 — opsz 48 | Tajawal 22/30 600 | Sub-headers |
| `--text-body-lg` | Geist 18/28 400 | Tajawal 17/28 400 | Lead paragraphs |
| `--text-body-md` | Geist 16/26 400 | Tajawal 15/26 400 | Default body |
| `--text-body-sm` | Geist 14/22 400 | Tajawal 13/22 400 | Secondary |
| `--text-caption` | Geist 12/18 500 — uppercase tracked | Tajawal 12/18 500 | Labels |
| `--text-mono` | Geist Mono 14/20 500 | — | Prices, codes |

### Type Rules
- **Display sizes use Fraunces optical sizing (`opsz`) for proper rendering at scale.**
- Display text in Latin always tracks tight: `letter-spacing: -0.02em`.
- Display text in Arabic stays default tracking.
- Caption labels are uppercase, Latin only, tracked `+0.08em`.
- Arabic body text uses slightly larger size than Latin equivalent for readability.
- Line-height ratios mirror the Latin equivalent (don't compress Arabic).
- Never combine display weight for Arabic + heavy weight for Latin in the same heading — keep weight parity.

---

## 4. Spacing & Layout

### Spacing Scale (4px base)
```
1   = 4px      6  = 24px     14 = 56px
2   = 8px      7  = 28px     16 = 64px
3   = 12px     8  = 32px     20 = 80px
4   = 16px     10 = 40px     24 = 96px
5   = 20px     12 = 48px     32 = 128px
```

### Container widths
- Mobile: full width, 16px padding
- Tablet: max 720px, 24px padding
- Desktop: max 1200px, 32px padding
- Wide (dashboards): max 1440px

### Layout principles
- **Asymmetry over symmetry** — never default to centered hero unless intentional
- **Editorial gutters** — generous outer margins
- **Vertical rhythm** — section spacing follows the type scale
- **Logical properties only** — `ps-*` `pe-*` `start-*` `end-*` (RTL-safe)

---

## 5. Border Radius

| Token | Value | Use |
|-------|-------|-----|
| `--radius-sm` | 4px | Tags, small chips |
| `--radius-md` | 8px | Inputs, secondary buttons |
| `--radius-lg` | 12px | Cards, primary buttons |
| `--radius-xl` | 20px | Modals, sheets |
| `--radius-2xl` | 32px | Hero cards, feature blocks |
| `--radius-full` | 9999px | Avatars, pills |

**Rule:** Photography cards and hero blocks use larger radius (`--radius-2xl`). Functional UI uses `--radius-lg` or `--radius-md`. Never mix more than two radii in one composition.

---

## 6. Shadows

Warm-tinted shadows, not flat black.

```css
--shadow-sm: 0 1px 2px rgba(58, 18, 52, 0.04);
--shadow-md: 0 4px 12px rgba(58, 18, 52, 0.06), 0 1px 3px rgba(58, 18, 52, 0.04);
--shadow-lg: 0 12px 32px rgba(58, 18, 52, 0.08), 0 4px 8px rgba(58, 18, 52, 0.04);
--shadow-xl: 0 24px 60px rgba(58, 18, 52, 0.12), 0 8px 16px rgba(58, 18, 52, 0.06);
--shadow-glow: 0 0 0 4px rgba(184, 34, 140, 0.12); /* focus rings */
```

**Rule:** Default UI uses `--shadow-md`. Hero cards lift to `--shadow-lg` on hover. Modals at `--shadow-xl`. Never use shadow for decoration — only for spatial separation.

---

## 7. Iconography

- **Library:** Lucide React (consistent stroke, large library, free)
- **Stroke:** 1.5px default, 2px on small sizes
- **Sizes:** 16, 20, 24, 32 (matching tap targets)
- **Color:** Inherits text color by default; use `--terracotta-500` for accent icons in lists
- **Custom icons:** Geometric, inspired by the three-triangle logo motif. Used sparingly for major nav items.

---

## 8. Motion

Subtle, purposeful. Never decorative.

```css
--ease-out: cubic-bezier(0.16, 1, 0.3, 1);   /* default exit */
--ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);
--ease-snappy: cubic-bezier(0.34, 1.56, 0.64, 1); /* attention */

--duration-fast: 150ms;
--duration-base: 250ms;
--duration-slow: 400ms;
--duration-page: 600ms;
```

### Motion principles
- **Page transitions:** staggered fade-up of major elements (cards, hero text). 80ms stagger.
- **Cards:** slight scale (1.02) + shadow lift on hover.
- **Buttons:** background color transition, no scale.
- **Modals:** slide-up + fade, snappy easing.
- **Lists:** items appear with 40ms stagger when filtered/sorted.
- Avoid: bounce effects, spinning loaders, color rainbows, parallax scrolling.

---

## 9. Components Inventory

### Foundation
- Button (primary, secondary, ghost, danger, sizes sm/md/lg, with-icon, icon-only, loading state)
- Input (default, with-leading-icon, with-trailing-icon, error, success)
- Textarea
- Select / Combobox
- Checkbox, Radio, Switch
- Date picker (single date, range — booking critical)
- Time picker
- Number stepper (guest count)

### Surfaces
- Card (default, elevated, interactive)
- Sheet (bottom for mobile, side for desktop)
- Modal / Dialog
- Drawer (admin navigation)
- Popover, Tooltip
- Toast / Banner

### Navigation
- Top app bar (guest mobile)
- Bottom tab bar (guest mobile) — 5 items max
- Sidebar (admin desktop)
- Breadcrumb
- Pagination
- Tabs

### Data Display
- Hotel card (small, medium, hero — three sizes)
- Room card
- Booking card
- Empty state
- Skeleton loader
- Stat card (admin dashboard)
- Table (admin)
- Badge / Tag / Pill
- Avatar
- Rating display (stars)
- Price display (with currency, with-strikethrough for discounts)

### Domain-specific
- QR key display (active stay screen)
- Service request composer
- Room status pill (clean/dirty/occupied/OOO)
- Task card (staff)
- Calendar/availability picker
- Locale switcher
- Currency switcher

### Patterns (not components, conventions)
- "Hero search" pattern (home screen)
- "Booking summary" pattern (sticky bottom bar during booking)
- "Status timeline" pattern (booking lifecycle)
- "Department dashboard" pattern (admin)

---

## 10. RTL & Bilingual Rules

- All layout uses **logical CSS properties**: `margin-inline-start`, `padding-inline-end`, `inset-inline-start`, etc.
- Tailwind: use `ps-*`, `pe-*`, `start-*`, `end-*` — never `pl-`, `pr-`, `left-`, `right-`.
- Set `dir="rtl"` on `<html>` for Arabic, `dir="ltr"` for Latin scripts.
- Numbers stay in Western Arabic numerals (1, 2, 3) for prices and dates — even in Arabic UI. Industry standard for tourism.
- Currency symbol position respects locale: `EGP 1,250` (EN) vs `١٬٢٥٠ ج.م` (formal AR) — but use Western numerals so it's `EGP 1,250` and `ج.م ١٬٢٥٠` only in formal contexts. For UI default to `EGP 1,250` everywhere.
- Date format: `Mar 15, 2026` (EN), `١٥ مارس ٢٠٢٦` (AR formal) or `15 مارس 2026` (AR practical).
- Icons that imply direction (arrows, chevrons) MUST flip in RTL. Use `rtl:rotate-180` or flipped SVG variants.
- Logos and brand wordmarks do NOT flip.

---

## 11. Photography Direction

When real photos are used:
- **Warm color grading** — pull cyans toward teal, push highlights warm
- **Golden hour or overcast natural light** — never harsh midday
- **Human-in-environment** — show life, not empty spaces (except architectural shots)
- **Editorial framing** — negative space, off-center subjects, environmental context
- **Avoid:** stock photo cliché (people pointing at laptops, fake smiles at receptions, drone-perfect symmetry)

When photos aren't available:
- Use gradient placeholders matched to the brand palette
- Add the three-triangle logo motif at low opacity as a watermark
- Never use solid grey "image-not-found" placeholders

---

## 12. Voice & Tone

### English
- **Direct but warm.** "Find your stay" not "Discover unique accommodations."
- **Conversational, not corporate.** "Your room is ready" not "Your accommodation is now available."
- **Confident, not salesy.** State facts, let the product breathe.
- **Title case for major headings**, sentence case everywhere else.

### Arabic
- **فصحى مبسطة** — Modern Standard Arabic, accessible register.
- Avoid colloquialisms in product UI (keep colloquial for marketing/social).
- Use the same direct warmth as English: "إقامتك جاهزة" not "تم تجهيز إقامتكم الكريمة".

---

## 13. Accessibility Targets

- **WCAG 2.1 AA minimum** for all interactive elements.
- Color contrast: 4.5:1 body, 3:1 large text.
- Focus rings always visible (the `--shadow-glow` token).
- Touch targets ≥ 44×44px.
- All interactive elements must be keyboard-reachable.
- Form fields always have visible labels (no placeholder-as-label).
- Arabic content uses `lang="ar"` attribute. Latin uses `lang="en"`.

---

## 14. Design Tokens (TypeScript)

The design tokens will be exported from `packages/ui/tokens.ts` and consumed by:
- Tailwind config (web + mobile)
- React Native StyleSheets
- Figma variables (manual sync for MVP, automated later via a token pipeline)

```typescript
// packages/ui/tokens.ts (sketch)
export const tokens = {
  colors: { /* ... */ },
  typography: { /* ... */ },
  spacing: { /* ... */ },
  radius: { /* ... */ },
  shadows: { /* ... */ },
  motion: { /* ... */ },
} as const;
```

This is the single source of truth. Figma references it; code references it; the design system doc explains it.
