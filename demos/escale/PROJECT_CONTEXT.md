# Escalé — Project Context

> **Handoff document.** This file captures everything decided so far for the Escalé hospitality platform. When starting a new Claude session (Cowork, Claude Project, Claude Code, etc.), read this first.

---

## 1. Project Overview

**Escalé** is a multi-tenant hospitality SaaS platform — **targeting Egypt and the Gulf (GCC) together**, built multi-market (the wider region can follow). It combines:

- Hotel booking (B2C)
- In-stay guest services (concierge, QR room key, service requests)
- Hotel operations (front-office dashboard, housekeeping, F&B)
- **B2B auction marketplace** for travel agencies (the competitive moat — no regional competitor offers this)

**Audience:** Independent hotels and small/mid hotel groups across **Egypt and the Gulf (GCC)** that want a premium operations stack with bilingual Arabic/English from day one. The platform is multi-tenant and multi-market — the wider region can follow. Not a Booking.com clone — closer to a "Mews + agency marketplace" for the regional market.

**Brand voice:** "Editorial Hospitality with an Arabian Soul." Sophisticated, restrained, warm — earth tones and the three-triangle desert/tent motif. The opposite of generic SaaS purple gradients and Material design. `design-system.md` carries the same framing with pan-regional Arab references spanning Egypt and the Gulf.

**Project name origin:** "إسكالي escalé" — French for "port of call" / stopover. Logo is an aubergine→magenta gradient circle with three white triangles (mountain/tent motif).

**Client status:** Initial scope review with client in progress. Voice message feedback received (to be transcribed in Cowork). Pending: scope confirmation, timeline, cost agreement.

---

## 2. Authoritative Tech Stack

Locked in. Don't propose alternatives unless explicitly asked.

| Layer | Choice |
|---|---|
| **Backend** | NestJS 10 + TypeScript (modular monolith) |
| **Database** | PostgreSQL 16 + Prisma ORM + **Row-Level Security** for tenant isolation |
| **Cache / Queue** | Redis + BullMQ |
| **Search** | Meilisearch (Arabic-aware tokenization) |
| **Mobile** | Expo (React Native) SDK 52+ |
| **Web** | Next.js 15 (App Router) for admin dashboard + booking web |
| **UI (web)** | shadcn/ui + Tailwind CSS |
| **UI (mobile)** | NativeWind (Tailwind for RN) |
| **Forms & validation** | React Hook Form + Zod (schemas shared between FE/BE) |
| **State management** | TanStack Query (server) + Zustand (client) |
| **Auth** | Better Auth + JWT, RBAC |
| **Payments** | Pluggable per market. Egypt: Paymob / Fawry (cards, Meeza, mobile wallets). Gulf: mada, Apple Pay, STC Pay, Tabby/Tamara (BNPL). Stripe (international). |
| **File storage** | Cloudflare R2 |
| **AI features** | Claude API via shared `packages/ai` abstraction |
| **Monorepo** | Turborepo + pnpm workspaces |
| **Deployment** | Railway/Fly (API), Vercel (web), EAS (mobile), Cloudflare CDN |
| **Real-time** | Socket.io + Redis adapter |
| **Observability** | Sentry + structured logs |

### Monorepo structure (planned)

```
escale/
├── apps/
│   ├── api/              # NestJS
│   ├── admin-web/        # Next.js admin dashboard
│   ├── booking-web/      # Next.js public booking site
│   ├── guest-mobile/     # Expo (guest app)
│   └── staff-mobile/     # Expo (staff app)
├── packages/
│   ├── ui/               # Design tokens, shadcn primitives
│   ├── db/               # Prisma schema + client
│   ├── shared/           # Zod schemas, types
│   ├── ai/               # Claude API abstraction
│   └── i18n/             # Translation utilities
└── turbo.json
```

---

## 3. Multi-Tenancy

Every tenant-scoped table has `hotel_id`. PostgreSQL Row-Level Security policies enforce isolation:

```sql
CREATE POLICY tenant_isolation ON bookings
  USING (hotel_id = current_setting('app.current_hotel_id')::uuid);
```

NestJS sets `app.current_hotel_id` per request via middleware that reads the authenticated user's hotel context. Zero data leakage even if a query forgets a `WHERE hotel_id =`.

---

## 4. Multilingual Strategy

**10 locales planned:** `en, ar, fr, de, es, it, ru, zh-CN, ja, tr`
**Priority pair:** Arabic + English (parity, day one).

### Translation tables, NOT JSON columns

Curated content (hotel descriptions, amenities, room types, services, rate plans) → dedicated translation tables:

```prisma
model Hotel {
  id           String              @id @default(uuid())
  // ... non-translated fields
  translations HotelTranslation[]
}

model HotelTranslation {
  id        String @id @default(uuid())
  hotelId   String
  locale    String  // 'en', 'ar', etc.
  name      String
  tagline   String?
  description String?

  hotel     Hotel  @relation(fields: [hotelId], references: [id], onDelete: Cascade)

  @@unique([hotelId, locale])
}
```

User-generated content (reviews, messages, service request notes) → single column + optional `language` field.

Notifications → rendered at send-time using user's preferred locale.

### RTL strategy

- Use Tailwind **logical properties** mandatorily: `ps-*`/`pe-*`/`ms-*`/`me-*` (NOT `pl-*`/`pr-*`)
- Set `dir="rtl"` at the document level for Arabic
- Test every screen by toggling — design must work in both directions
- Arabic font: **Tajawal** (Google Fonts)
- Latin display font: **Fraunces** (variable serif)
- Latin body font: **Geist**
- Numbers/code: **Geist Mono**

---

## 5. Brand & Design System

### Palette

```css
/* Aubergine — primary */
--aubergine-900: #3A1234;
--aubergine-700: #4A1942;
--aubergine-500: #5B2C5F;
--aubergine-300: #8E5A8E;
--aubergine-100: #EFE3EF;

/* Magenta — accent (used sparingly for the "wow" moment) */
--magenta-500: #B8228C;
--magenta-300: #D86CB0;

/* Terracotta — warm earth */
--terracotta-700: #A05A3A;
--terracotta-500: #C9956B;
--terracotta-300: #E2BC9C;
--terracotta-100: #F5E8DC;

/* Neutrals */
--ink-900: #1C1410;
--ink-700: #3D332D;
--ink-500: #6B5D54;
--ink-300: #A89A8E;
--sand-100: #F0E9DF;
--sand-50: #F7F2E9;
--cream: #FAF7F2;
--paper: #FDFBF7;

/* Semantic */
--success: #5A8C5F;
--danger: #B53F3F;
--warning: #C9956B;
--inspected: #2E7A85;  /* HK state */
```

### Typography

- **Display (Latin):** Fraunces, variable, weights 400–500
- **Body (Latin):** Geist, weights 400–600
- **Arabic:** Tajawal, weights 400, 500, 700
- **Mono:** Geist Mono for numbers, codes, technical
- **Italic Fraunces** is the signature accent — used for "moment" words: *find your stay*, *the moment*, *one glance*, etc.

### Design principles (recurring across all 14 screens)

1. **Editorial layout** — like a hotel magazine, not a SaaS dashboard
2. **One highlighted element per screen** — the brand-gradient card is rationed
3. **Magenta is expensive** — reserved for the primary action moment only
4. **Italic Fraunces for emotional accents** — never for body, only for highlights
5. **Three triangles motif** from the logo as subtle decoration (low opacity)
6. **Editorial pull-quotes** for reviews and testimonials
7. **Priority bars (thin colored lines)** instead of loud badges
8. **Numbers in Fraunces serif**, not sans (even for KPIs)

---

## 6. Critical Domain Rules

### Booking state machine

```
DRAFT → PENDING_PAYMENT → CONFIRMED → CHECKED_IN → CHECKED_OUT → COMPLETED
                                 ↘ CANCELLED
                                 ↘ NO_SHOW
```

State transitions are enforced server-side. Each transition fires BullMQ side effects (emails, key generation, billing).

### Housekeeping state machine

```
DIRTY → IN_PROGRESS → CLEAN → INSPECTED → READY
                                 ↘ OUT_OF_ORDER
```

Mobile UI only ever shows the next valid transition as the primary action. User can't move a room into an illegal state.

### Digital key (QR)

- Signed JWT with `bookingId`, `roomId`, `guestId`, `validFrom`, `validUntil`
- Rotates every 30 seconds
- Door lock validates offline against rotated public key set
- Activates at check-in time, deactivates at check-out

> **Revisit (see `DOCUMENTATION.md` D9):** the digital key is a **smart-lock vendor integration** — the real unlock mechanism (BLE / NFC / PIN-code / QR-reader) depends on the hotel's door hardware, not built in-house. The offline-QR model above assumes QR-reader locks, a narrow choice. Lock hardware + vendor subscription are the client's cost.

### B2B Auction flow

- Travel agency posts demand (rooms, dates, meal plan, target rate)
- Eligible hotels see live request stream with countdown
- Hotels submit bids (rate + terms)
- Agency reviews bids; first acceptable bid wins (or manual select)
- Status: `OPEN → BIDDING → AWARDED → CONFIRMED → COMPLETED`
- Trust signals: tier-based (Tier 1/2/New), historical booking count, verified badge

### Online check-in & digital key (new — D11)

- The guest completes check-in **before arrival** — identity verification, guest details, room assignment — and receives the QR digital key in advance.
- On arrival they skip the front desk and go straight to the room — a deliberate competitive advantage.
- The first door-open event drives the `CONFIRMED → CHECKED_IN` transition and notifies the operations dashboard that the room is now occupied.

### Live chat (new — D12)

- Real-time in-app messaging across three channels: guest ↔ staff, guest ↔ management, management ↔ staff.
- Purpose: faster service response, a direct complaint line to management (heading off negative public reviews), and effective staff oversight.

### Service-request live tracking (new — D13)

- A guest requests a service directly from the relevant specialist; the request and the staff member are time-tracked against an SLA.
- The guest can comment on a request while it is in progress; comments are visible to management — so accountability is built in.
- New screen: **23 — Request Tracking** — the request tracked live (status timeline, assigned specialist, comment thread).

---

## 7. Screens Built (Design Preview)

The screen prototypes are polished bilingual HTML/CSS files (in the project root). Each has a phone/desktop frame, a working AR⇄EN toggle, and a side panel with design rationale. **The live, authoritative screen inventory is in `DOCUMENTATION.md` §2 — now 23 screens**, including the new Online Check-in, Live Chat and Request Tracking screens (21–23). The table below is the original 14-screen set, kept for reference.

| # | File | Title | Category | Description |
|---|---|---|---|---|
| 01 | `01-guest-discover.html` | Discover | Guest mobile | Hero search, curated stays, last-minute deals, bottom nav |
| 02 | `02-hotel-detail.html` | Hotel detail | Guest mobile | Hero photo, rooms, amenities, review-as-pull-quote, sticky Reserve |
| 03–05 | `03-05-booking-flow.html` | Booking flow | Guest mobile | 3 phones: dates+room, guest+Digital ID, payment (Paymob first) |
| 06 | `06-active-stay.html` | Active stay · QR key | Guest mobile | **The wow moment.** QR key as hero, services, concierge |
| 07 | `07-admin-operations.html` | Operations dashboard | Admin web | KPIs, arrivals timeline, 48-room grid, open requests |
| 08–09 | `08-09-staff-tasks.html` | My tasks + Task detail | Staff mobile | Shift progress ring, prioritised tasks, big room numbers |
| 10 | `10-booking-confirmed.html` | Booking confirmed | Guest mobile | Keepsake card, timeline, concierge signature (not confetti) |
| 11–12 | `11-12-service-request.html` | Service request | Guest mobile | Catalog + composer (second guest wow) |
| 13 | `13-hk-room-status.html` | HK room status | Staff mobile | 50 rooms in one glance, color bands, scan-to-jump QR |
| 14 | `14-b2b-auctions.html` | B2B auction portal | Admin web | **The competitive moat.** Live agency demand, countdown bids |

**App breakdown:**
- Guest mobile: 7 screens (01, 02, 03–05, 06, 10, 11–12)
- Admin web: 2 screens (07, 14)
- Staff mobile: 3 screens (08–09, 13)

### Known issue (fixed)

All screens previously had a parent-child ID conflict in JS — setting `textContent` on a parent element wiped child elements with IDs being looked up next. **Fixed in all 10 files** by restructuring: `<h2 id="ph-1">A <em id="ph-1em">B</em></h2>` → `<h2><span id="ph-1">A</span> <em id="ph-1em">B</em></h2>`. Verified clean with jsdom. **When adding new screens, never put an ID on a parent that has children with IDs being updated via textContent.**

---

## 8. Deliverables in This Folder

```
escale/
├── PROJECT_CONTEXT.md          ← this file, master handoff
├── CLAUDE.md                   ← AI context for Claude Code (Cursor/Claude Code reads this)
├── schema.prisma               ← Full Prisma schema with 18+ models + translation tables
├── docs/
│   ├── architecture.md         ← System architecture, modules, flows
│   └── design-system.md        ← Brand foundation, tokens, components, RTL
└── screens/
    ├── 01-guest-discover.html
    ├── 02-hotel-detail.html
    ├── 03-05-booking-flow.html
    ├── 06-active-stay.html
    ├── 07-admin-operations.html
    ├── 08-09-staff-tasks.html
    ├── 10-booking-confirmed.html
    ├── 11-12-service-request.html
    ├── 13-hk-room-status.html
    └── 14-b2b-auctions.html
```

### Database schema highlights (see `schema.prisma`)

18+ models including: `Hotel`, `HotelTranslation`, `HotelImage`, `Amenity`, `AmenityTranslation`, `User`, `Session`, `HotelStaff`, `RoomType`, `RoomTypeTranslation`, `Room`, `RatePlan`, `RatePlanTranslation`, `Booking`, `BookingRoom`, `BookingGuest`, `Payment`, `ServiceCategory`, `ServiceCategoryTranslation`, `HotelService`, `HotelServiceTranslation`, `ServiceRequest`, `StaffTask`, `Review`, `LoyaltyAccount`, `LoyaltyTransaction`, `Notification`.

---

## 9. Pending Work

### Immediate
- [ ] **Transcribe client voice message** (`Fathy_Al_Sayed_Street_15.m4a`) and classify feedback (scope / design / timeline / cost)
- [ ] Build the landing/demo page (`index.html` in root of `escale/`) — was started but not finished. Goal: deploy to `ahmedbeshir.com/escale/` for client review

### After client sign-off
- [ ] Agree on final scope (features + timeline + cost)
- [ ] Create dedicated Claude Project with PROJECT_CONTEXT.md as knowledge
- [ ] Subscribe to Figma (Starter plan was insufficient — only 3 pages, MCP rate limits hit hard)
- [ ] Begin actual development with Claude Code:
  - Phase 1: Database + auth + hotel CRUD + booking flow (API)
  - Phase 2: Guest mobile app
  - Phase 3: Admin web
  - Phase 4: Staff mobile + HK
  - Phase 5: B2B auctions

### Backlog screens (not yet designed)
- Auth (login, signup, OTP, forgot password)
- Onboarding (hotel setup wizard for admin)
- Search/filter results (between Discover and Hotel Detail)
- Reviews submission
- Profile / settings (guest)
- Reports & analytics (admin)
- Channel manager integration (admin)
- Loyalty program (guest + admin)

---

## 10. Workflow Notes

### Figma context

- File: `https://www.figma.com/design/ADAyD3HeOI0dx3uZpmykGH`
- Plan key: `team::1016577715869587917`
- Email: `ahmedoneee@gmail.com`
- Status: Starter plan was hitting limits during the design preview phase. **Pivoted to HTML/CSS artifact mode** (no rate limits, faster iteration). Will return to Figma after subscribing.

### Spec source

Original 41-point Arabic feature scenario from the client:
`Escale__for_Omer_damour.pdf` (in user uploads). Logo: `Option_01.png`.

### Owner

**Ahmed Beshir** — Cairo, Egypt. Senior web developer at Illa Holding. Communicates in Egyptian Arabic mixed with English technical terms. Portfolio: `ahmedbeshir.com` (Cloudflare Pages).

---

## 11. Communication Style

When working with Ahmed:

- **Egyptian Arabic + English tech terms** — e.g., "نعمل الـ booking flow الأول" not formal Arabic
- **Be direct** — recommend, don't over-ask. Use `ask_user_input_v0` for choices on mobile-friendly questions
- **Test before declaring done** — always validate HTML/JS with jsdom before saying "shipped"
- **Use `present_files`** to share deliverables; don't dump long markdown when a file does it better
- **Keep momentum** — Ahmed is a senior dev shipping fast. Show, don't lecture

---

## 12. Suggested First Prompt for the New Claude Session

> اقرأ الـ `PROJECT_CONTEXT.md` الأول، بعدين اسمع الـ voice message `Fathy_Al_Sayed_Street_15.m4a` (ده رد العميل على الـ design preview).
>
> طلبي:
> 1. لخّص لي بالنقاط العميل قال إيه (positive / objections / changes / scope questions)
> 2. صنّف الفيدباك: تصميم / scope / timeline / تكلفة
> 3. لو في تعديلات مطلوبة على الـ screens، اقترح action plan
> 4. اقترحلي رد رسمي للعميل (بالعربي) بناءً على الفيدباك

---

*Last updated: 25 May 2026 — target market is Egypt and the Gulf (GCC) together, multi-market by design; new features added — full pre-arrival online check-in, 3-channel live chat, and service-request live tracking.*
