# Escalé — Living Project Documentation

> The single up-to-date record of what's been done, decided, and what's next.
> Updated after **every** step. For the static project handoff see `PROJECT_CONTEXT.md`;
> for coding conventions see `CLAUDE.md`.

**Last updated:** 2026-08-09

---

## 1. Where the project stands

- Design phase. **28 screens** exist as bilingual HTML/CSS prototypes, all linked from the `index.html` review hub.
- **Legacy-app services parity done 2026-08-09.** The client supplied 16 screenshots of an older version of the app (`services/I1–I16.jpeg`) covering its in-hotel services. Everything in them was extracted into `services-gap-analysis.md`, the schema was extended, and five new screens (24–28) plus three updated ones were built. See decision **D14**.
- Client returned per-screen feedback; **the full feedback round is now applied** (see §4 — every item checked).
- Design-rationale text simplified to plain Arabic + English across all screens.
- **Screen 19 — Offers** built 2026-05-24: markets in-hotel services (dining, spa, pools, experiences) to hotel guests *and* external visitors nearby. See decision D7.
- **Screen 20 — Service Detail** built 2026-05-24: the full detail page for a single service (description, what's included, session options, hours, a guest review), sitting between the catalog/offers and the request composer. See decision D8.
- **Screens 21 — Online Check-in** and **22 — Live Chat** built 2026-05-25 for new client-requested features — see D11 (full pre-arrival check-in) and D12 (3-channel live chat).
- **Screen 23 — Request Tracking** built 2026-05-26 — the guest-side of D13: a service request tracked live (status timeline, SLA timing, assigned specialist, and a comment thread visible to management).
- **Target market corrected 2026-05-25 → Egypt + the Gulf (GCC), together** (see D10). Docs, `design-system.md`, `integrations.md` and the 20 earlier screen prototypes were updated to span both markets.
- Next: a second client review pass, then scope/cost sign-off and development.
- Commercials: proposal **prepared** — `Escale-Project-Proposal.pdf` (bilingual, on-brand). **$9,000 USD**, full platform delivered as **3 projects** (Services Management App — main, Bookings App, B2B Auction), each paid in **3 installments** — no separate deposit, ~5-month timeline, plus a **$400/mo** support-&-development retainer. Not signed yet.

---

## 2. Screen inventory

| # | File | Title | Surface | State |
|---|------|-------|---------|-------|
| 01 | `01-guest-discover.html` | Discover | Guest mobile | Needs edits |
| 02 | `02-hotel-detail.html` | Hotel Detail | Guest mobile | Needs example data |
| 03–05 | `03-05-booking-flow.html` | Booking Flow | Guest mobile | Needs edits |
| 06 | `06-active-stay.html` | Active Stay | Guest mobile | Approved |
| 07 | `07-admin-operations.html` | Operations | Admin web | Needs edits |
| 08–09 | `08-09-staff-tasks.html` | Staff Tasks | Staff mobile | Approved |
| 10 | `10-booking-confirmed.html` | Booking Confirmed | Guest mobile | Approved |
| 11–12 | `11-12-service-request.html` | Service Request | Guest mobile | Needs detail views |
| 13 | `13-hk-room-status.html` | HK Room Status | Staff mobile | Needs edits |
| 14 | `14-b2b-auctions.html` | B2B Auctions | Admin web | Approved |
| 15 | `15-b2b-agency-portal.html` | B2B Agency Portal | B2B web | Built 2026-05-23 |
| 16–17 | `16-17-login.html` | Login & Sign-in | Guest mobile | Built 2026-05-23 |
| 18 | `18-hotel-home.html` | Hotel Home (in-stay) | Guest mobile | Built 2026-05-23 |
| 19 | `19-offers.html` | Offers | Guest mobile | Built 2026-05-24 |
| 20 | `20-service-detail.html` | Service Detail | Guest mobile | Built 2026-05-24 |
| 21 | `21-online-checkin.html` | Online Check-in | Guest mobile | Built 2026-05-25 |
| 22 | `22-live-chat.html` | Live Chat | Guest mobile | Built 2026-05-25 |
| 23 | `23-request-tracking.html` | Request Tracking | Guest mobile | Built 2026-05-26 |
| 24 | `24-restaurant-menu.html` | Restaurant Menu & Cart | Guest mobile | Built 2026-08-09 |
| 25 | `25-halls-booking.html` | Halls & Bookable Assets | Guest mobile | Built 2026-08-09 |
| 26 | `26-transport.html` | Transport & Transfer | Guest mobile | Built 2026-08-09 |
| 27 | `27-outlet-profile.html` | Outlet Profile | Guest mobile | Built 2026-08-09 |
| 28 | `28-my-orders.html` | My Orders | Guest mobile | Built 2026-08-09 |
| — | `index.html` | Design Preview hub | Review tool | Built 2026-05-23 |

All 28 screens now exist. The hub links all of them; no placeholders remain.

Note: `visual-preview.html` (a stale duplicate of an old screen 01) was deleted on 2026-05-23.

---

## 3. Key decisions

- **D1 — Services layer is channel-agnostic.** The in-stay services part must serve *every* hotel guest regardless of booking channel (Escalé, Booking.com, Agoda, Musafir, TripAdvisor, walk-in). Each guest gets a link → registers → digital key + services. The schema partly supports this (`Booking.source` enum already has `OTA/WALK_IN/PHONE/B2B`; `ServiceRequest.bookingId` is optional). **Open issue:** `Booking.userId` is required — a hotel creating a stay for an unregistered OTA guest has no user to attach. Needs a "claim your stay" flow or a nullable `userId`.
- **D2 — B2B portal = reverse auction (InDrive-style).** A travel agency posts a booking request (dates, rooms, guests, target rate); hotels see it and submit price bids; the agency picks. Screen 14 is the *hotel* side; the *agency* side is a new screen.
- **D3 — Commercials.** Milestone-based pricing, quoted in **USD** (not EGP — FX risk). ~15–20% non-refundable deposit, each milestone paid on acceptance, plus a monthly maintenance retainer. Third-party/hosting costs billed to the client. **Finalised 2026-05-24** (iterated several times the same day; the client then reorganised the structure): total **$9,000 USD** for the full platform, delivered as **3 projects** — the client's own grouping, by screen:
  - **Project 1 — Services Management App, $5,700** (the *main* project): the full in-stay journey — login → QR digital key issue → in-stay services (restaurant, HK, health club, maintenance, cars, bellboy, and more by hotel) → check-out → QR key deactivation. Carries the shared foundation/API. Screens 6,7,8,9,11,12,13,16,17,18,19,20. ~13 wks.
  - **Project 2 — Bookings App, $2,100**: hotel discovery, detail, booking flow, payment, confirmation. Screens 1,2,3,4,5,10. ~5 wks.
  - **Project 3 — B2B Auction, $1,200**: the reverse-auction marketplace. Screens 14,15. ~3 wks.
  Each project is an independent unit paid in **3 installments** (start / progress / delivery) — no separate deposit; the first payment is the Project 1 start installment (**$1,500**, non-refundable). Optional **$400/month** part-time (~2 hrs/day) support-&-development retainer (separate from the $9,000). **~5-month** timeline (with buffer). Delivered as `Escale-Project-Proposal.pdf`.
- **D4 — Review hub.** `index.html` built as bespoke HTML following `design-system.md` (the project's own design system is the source of truth; no generic skill fits better).
- **D5 — Client communication.** Client relies on visuals and finds dense text hard. All client-facing material uses simple Arabic + simple English, side by side.
- **D6 — Simplify screen text.** Each screen carries two text layers: the app UI text (the real product — kept as-is, bilingual via toggle) and the design-rationale notes (dense, jargon-heavy). Per client request, the rationale notes in all 15 screens are being rewritten into plain Arabic + English.
- **D7 — Offers screen targets external customers, not just guests.** Screen 19 markets a hotel's in-stay services (restaurants, spa/health club, pools, experiences) as bookable offers. The client's strategic point: booking apps sell only rooms, while hotels have high-margin services that sit half-empty. The offers surface is open to **two audiences** — hotel guests *and* external visitors within the hotel's vicinity ("guests & visitors") — to drive incremental F&B/wellness revenue. This extends D1 (channel-agnostic services) one step further: services are sold not only to guests of any booking channel, but to people who never booked a room at all. Schema implication: an offer purchase by an external visitor has no `Booking` to attach to — flagged in §6.
- **D8 — The services flow now has a detail step.** Screen 20 (Service Detail) is a dedicated full page for a single service — description, what's included, selectable session options, hours/location, a guest review — modelled on the depth of the HK Room Status screen (per the client: "a service-detail screen, like the HK screen"). It sits in the navigation between browsing and requesting: **catalog (11) / offers (19) → service detail (20) → request composer (12).** It is shared by both audiences from D7 — in-stay guests reach it from the catalog, external visitors reach it from an offer. Session/option selection happens here, so the composer stays short.
- **D9 — The digital/QR key is a lock-vendor integration, not built in-house.** A QR code alone cannot open a physical door — the unlock mechanism depends on the hotel's door hardware (Bluetooth/BLE, NFC, PIN-code, or a QR-reader lock). Even Duve — a mature competitor — does not build locks; it integrates with smart-lock vendors (ASSA ABLOY, Dormakaba, Salto, Nuki, etc.). Escalé will do the same: integrate with **one** lock vendor chosen per the hotel's hardware, and the lock hardware + the vendor subscription are the **client's cost** (excluded, like hosting and gateway fees). The in-app QR stays as the UI / check-in pass; the real unlock = whatever the integrated lock supports. **Open:** choose the lock vendor and confirm with the client before development; the JWT-QR "offline-validating" model in `PROJECT_CONTEXT.md` §6 should be revisited — it assumes QR-reader locks, which is a narrow hardware choice. **The full vendor landscape, integration approach and costs are now documented in `integrations.md` (2026-05-25)** — recommendation: a unified lock API (Seam) with PIN-code unlock for MVP, BLE as a later phase.
- **D10 — Target market: Egypt and the Gulf (GCC), together.** The product targets two regions at once — Egypt and the Gulf — not a single launch market. Multi-tenant, multi-market, pluggable per market; the wider region can follow. *Decision history:* originally framed Egypt-only → narrowed to a "Saudi Arabia launch market" earlier on 2026-05-25 → **corrected again 2026-05-25 by the client to Egypt + the Gulf both** — this is the current decision. Applied: payments → pluggable per market: Egypt (Paymob, Fawry, Meeza, mobile wallets) + Gulf (mada, Apple Pay, STC Pay, Tabby/Tamara) + Stripe (intl); compliance/identity → per market: Egypt (ETA e-invoicing, Ministry-of-Tourism guest registration) + Gulf (Saudi: Nafath, ZATCA/Fatoora, Shomoos, PDPL); SMS → Unifonic (Gulf) / a local aggregator (Egypt) / Twilio (intl); brand framing → pan-regional Arab references spanning Egypt and the Gulf; screen example data → diversified across the prototypes to span both markets (Cairo, Alexandria, El Gouna, Aswan + Riyadh, Jeddah, AlUla, Dubai; EGP/SAR/AED in context). Docs updated 2026-05-25: `PROJECT_CONTEXT.md`, `CLAUDE.md`, `design-system.md`, `integrations.md`, the 20 earlier screen prototypes, and the proposal's payment line. Integration approach + 2026 costs for both markets are in `integrations.md`.
- **D11 — Full online (pre-arrival) check-in.** The guest completes check-in before arriving — identity verification, guest details, room assignment — and receives the room number and QR digital key in advance, so on arrival they skip the front desk and go straight to the room. Per the client, this is a real competitive advantage. Tied to it: the **first door-open event** (the guest using the digital key) automatically (a) moves the booking `CONFIRMED → CHECKED_IN`, and (b) notifies the operations dashboard that the room is now occupied — management sees "arrived & occupied" in real time. New screen: **21 — Online Check-in**. Flow/schema implication: the door-open event becomes a `CHECKED_IN` trigger and an ops-notification source; the unlock mechanism itself is the smart-lock integration from D9.
- **D12 — In-app live chat, three channels.** Real-time messaging inside the apps across three channels: (1) **guest ↔ assigned staff** — speeds the response to service requests; (2) **guest ↔ hotel management** — a guest can raise a complaint straight to management, who have the authority to resolve it fast, heading off negative public reviews and lifting satisfaction; (3) **management ↔ staff** — speeds coordination and gives management effective oversight of staff performance. New screen: **22 — Live Chat**. Schema implication: chat thread + message models, hotel-scoped (`hotelId`); messages are user-generated content (single language, optional `language`) — flagged in §6.
- **D13 — Service-request accountability & live tracking.** A guest requests a service directly from the relevant specialist; the request and the assigned staff member are tracked against time (an SLA). The guest can comment on a request while it is in progress, and the comment is visible to hotel management. Because guest, staff and management all see the request live, the staff member knows the work is being watched — which drives speed and quality. Extends the existing Service Request lifecycle (no new states); builds on D12's chat. Screen 07 gains the room-occupancy notification (D11). The guest-side — a request tracked live with the comment thread — is built as **Screen 23 (Request Tracking)**, 2026-05-26.

- **D14 — In-hotel services rebuilt to legacy-app parity, around the *outlet*.** The client supplied 16 screenshots of an older version of Escalé (`services/I1–I16.jpeg`) whose services module was considerably deeper than the current design: a restaurant menu with a cart and a real order total, halls and vehicles as bookable units, appointment booking with a running total, a service-provider profile with tabs (info / services / gallery / reviews), a "my orders" tab, and a back-office with per-department order counts **including total order value**. Every one of those was extracted and mapped in **`services-gap-analysis.md`** (14 gaps, G1–G14, each scoped MVP vs. later).
  The structural decision underneath all of it: **a service belongs to an in-hotel *outlet*, not directly to the hotel.** An outlet is a restaurant, a spa, a salon, a halls complex, a transport desk — and, per the client, it is **not always run by the hotel**. An independent business can operate inside the property (his example: a well-known hairdresser leasing space inside a Cairo five-star). Both kinds appear in the same catalog and the guest journey is identical; only settlement differs. Modelled as `ServiceOutlet.operatorType = HOTEL_OPERATED | CONCESSION`.
  Schema changes made 2026-08-09 (`schema.prisma`, validated with `prisma validate`): new models `ServiceOutlet` (+translations, opening hours, sections, images), `BookableAsset` (+translations, `AssetReservation`), `ServiceOffer` (+translations), `Favorite`, `ServiceReview`, `ServiceRequestLine`, `ServiceRequestComment`; `ServiceCategory` gained a self-relation for a two-level tree; `HotelService` gained outlet/section/image/prep-time/duration; and **`ServiceRequest` was widened into a full order** — reference code, `type` (instant / order / appointment / asset booking / transfer), scheduling, transfer fields, money totals, settlement method, payment link, SLA `dueAt`, and an optional `stage` sub-status for the delivery leg. `ServiceRequest.userId` became **nullable** (with `guestName`/`guestPhone`), which resolves the D1/D7 open issue of a service customer who has no booking and no account.
  Deliberately *not* forked: the `ServiceRequestStatus` state machine. The legacy app's `تحت التجهيز` / `قيد التسليم` / `تم التسليم` are modelled as an optional `FulfilmentStage` inside `IN_PROGRESS` rather than new top-level states, so CLAUDE.md §4.4 still holds.
  Design: five new screens **24–28** (restaurant menu & cart, halls, transport, outlet profile, my orders) and three updated ones — **11–12** (two-level categories + named outlets), **19** (offer category filters, discount badges, struck prices, ratings, hours), **07** (per-department order counts, overdue counts and order value). Not carried over: the legacy username/password login, superseded by screens 16–17.
  **Open with the client** — five questions listed in `services-gap-analysis.md` §7, the important two being how a concession's money is settled, and whether "charge to room" is available for independent operators.

---

## 4. Client feedback tracker

### New client direction — 2026-05-25 (market + features)
- [x] Target market corrected to **Egypt + the Gulf (GCC)** together — see D10. Applied across all docs, `design-system.md`, `integrations.md`, the 20 screen prototypes, and the proposal.
- [x] Full **pre-arrival online check-in** (D11) — built as Screen 21.
- [x] **Room-occupancy notification** on first door-open (D11) — added to Screen 07 Operations.
- [x] **3-channel live chat** — guest↔staff, guest↔management, management↔staff (D12) — built as Screen 22.
- [x] **Service-request live tracking** — SLA timing + guest comments visible to management (D13). Operations side on Screen 07; guest side built as **Screen 23 — Request Tracking** (2026-05-26).

### Concept
- [x] B2B bookings: agency-facing request screen (reverse auction — see D2). — built as Screen 15, 2026-05-23.

### Screen 01 — Guest Discover — DONE 2026-05-23
- [x] Hidden content — the review hub now shows the full scrollable screen; the screen is also fuller now.
- [x] Added check-in / check-out dates + guest count — a trip row inside the search block.
- [x] Added filters: Filters / Price / Rating / Services row.
- [x] Added voice search — a microphone button in the search row.
- [x] Added a "Most searched" section with popular-destination pills.
- [x] Added a "Recently viewed" section with recent hotel cards.
- [x] Hotel cards now show review counts next to the rating; the featured card already showed reviews.

### Screen 02 — Hotel Detail
- [x] Already populated with a full realistic example (Adrère Amellal, Siwa — description, rooms, amenities, a guest review). The conditional request is satisfied; no change needed.

### Screen 03–05 — Booking Flow — DONE 2026-05-23
- [x] Step 01: added a "carried over from your search" note above the dates/guests, showing the data came from screen 01.
- [x] Step 02: added Nationality and ID/Passport fields, plus a note that they let the hotel finish check-in before arrival.
- [x] Step 03: approved as-is.

### Screen 06 — Active Stay
- [x] Approved. One of the standout screens. Minor improvements possible later.

### Screen 07 — Admin Operations — DONE 2026-05-23
- [x] Added an orders-stats strip on the Open Requests card: Completed today (+ View), In service, Overdue (+ View), Active staff.
- [x] Overdue tile is highlighted and has a View action; Completed has a View action.
- [x] Added a "Send instructions to staff" button on the card.

### Screen 08–09 — Staff Tasks
- [x] Approved. Improvements can come during build.
- Note: client wants to clearly see the guest's ordering screen (covered by screen 11–12).

### Screen 10 — Booking Confirmed
- [x] Approved on first look.

### Screen 11–12 — Service Request — DONE 2026-05-23
- [x] Design approved overall — a core screen.
- [x] Each service category now shows a detail line (hours + price/availability) under its count.

### Screen 13 — HK Room Status
- [x] Design approved.
- [x] Added a guest-requested cleaning-time indicator on a stayover room ("Guest asked: clean after 14:00"). Built 2026-05-23.

### Screen 14 — B2B Auctions
- [x] Design approved.
- [ ] New screen required for travel agencies to post their requests (see Concept above).

### New screens
- [x] B2B Agency Portal (post request + receive bids). — Screen 15, built 2026-05-23.
- [x] Login screen + welcome/options screen. — Screens 16–17, built 2026-05-23.
- [x] Hotel in-stay home page (info + available services). — Screen 18, built 2026-05-23.
- [x] Offers screen — markets in-hotel services (dining, spa, pools, experiences) to guests + external visitors nearby. — Screen 19, built 2026-05-24. See D7.
- [x] Service Detail screen — full detail of one service before requesting, modelled on the HK screen's depth. — Screen 20, built 2026-05-24. See D8.

### Cross-cutting
- [ ] Populate screens with realistic example data (client reviews visually).
- [x] Build the demo/review hub page (`index.html`).
- [x] Use simple Arabic + English in client-facing material.

---

## 5. Work log

### 2026-08-09
- **Studied 16 screenshots of the legacy Escalé app** (`services/I1–I16.jpeg`, supplied by the client) covering its in-hotel services module — services hub, housekeeping sub-menu, restaurant menu, halls, maintenance, offers, a salon profile with tabs, the two car screens, and three back-office views. Produced **`services-gap-analysis.md`**: a screen-by-screen extraction, a 14-item gap list (G1–G14) against the current design and schema, an MVP/later scope call for each, and five open questions for the client.
- **Recorded decision D14** — services are rebuilt around the **in-hotel outlet**, which may be hotel-operated or an independent concession inside the property.
- **Extended `schema.prisma`** with 12 new models and a widened `ServiceRequest` (see D14 for the full list). Validated with `prisma validate` — schema is valid. `ServiceRequest.userId` is now nullable, closing the long-standing D1/D7 gap of a service customer with no booking and no account.
- **Built five new screens** — **24 Restaurant Menu & Cart**, **25 Halls & Bookable Assets**, **26 Transport & Transfer**, **27 Outlet Profile**, **28 My Orders**. All bilingual EN/AR, matching `design-system.md` and the existing screen architecture, no external images.
- **Updated three existing screens** — **11–12** now shows the second category level (housekeeping → laundry / cleaning / amenities / linens) and named dining outlets tagged hotel-run vs. independent; **19** gained category filter pills, discount badges, struck-through original prices, ratings and opening hours on every offer, plus a halls offer, with the D7 "guests & visitors" tagging preserved; **07** gained a per-department order strip (housekeeping, dining, spa, maintenance, transport) with order count, overdue count and **total order value today** — keeping the existing orders-stats strip and the digital-key occupancy alert intact.
- **Aligned the Arabic register** in screens 24–26, which had been drafted in formal MSA, to the warm simplified Egyptian register the other 25 screens use (decision D6).
- **Updated `index.html`** — five new cards under Guest App (now 18 screens), masthead → 28 screens, legend → "All 28 screens ready", footer date.
- **Full validation sweep**: all 23 screen files parse clean, the EN⇄AR toggle runs three times per file with zero JS errors, every `setLocale` id resolves, no duplicate ids, no "undefined" text; the hub's 23 card links and 23 previews all resolve.
- **Second-pass audit and fix round (same day).** A deliberate skeptical re-review of every screen against the 16 screenshots found real defects that the first pass had missed. All were fixed and re-validated:
  - **Screen 11–12 was missing four of the legacy eight categories** — transport, halls, bellboy/porter and in-room requests — which left screens 25 and 26 **unreachable from the guest catalog**. The catalog now carries 10 categories, plus the search field and filter control the legacy hub had, and its hardcoded prices and the untranslated "Now" chip were pulled into the translation table.
  - **Screen 07 was missing three departments** (halls, porter, rooms) and three of the six legacy metrics. It now shows all eight departments with orders · in preparation · out for delivery · overdue · delivered · order value, with a dash where a stage does not apply. The figures were rebuilt so the whole screen reconciles: department orders (31) = completed today (23) + in service (8); delivered = 23; prep + on-the-way = 8; overdue = 2; values sum to the displayed total.
  - **Screen 19 had no book button on any offer card** — only the featured banner was actionable. Every card now has a CTA. Two discount badges were also arithmetically wrong (−30% on a 28% saving, −25% on 24%); all six offers are now exact, and the static HTML now matches the dictionary so the first paint is correct.
  - **Screens 25 and 28 contained example data that contradicted their own copy** — a 180-guest party returned two halls too small to hold it, and a room total of EGP 3,240 against EGP 2,085 of visible charges. Both fixed.
  - **Screen 28 contradicted screen 23** on the same request (room 312 vs 412, `REQ-` vs `SR-`). Screens 23, 24, 27 and 28 now agree on one property (Beit Zamalek, Cairo), one room (312) and one reference format.
  - Two screens named their hotel after the platform (`Escalé Zamalek`, `Escalé Sahl Hasheesh`) — renamed, since Escalé is the multi-tenant platform, not a hotel brand.
  - **`28-my-orders.html` had no way in.** A **My Orders / طلباتي** tab was added to the guest bottom navigation on screens 01, 06, 18 and 19 (on 19, which was already at the 5-tab maximum, Discover was replaced). The staff screens were deliberately left alone — a housekeeper has no "my orders".
  - **Numerals normalised across the whole set.** `design-system.md` §10 requires Western numerals even in Arabic; screens 01–23 were using Arabic-Indic throughout. 571 digits across 18 files converted, plus `٬`/`٫`/`٪`. Zero remain.
  - **Stale screen totals fixed** — files still said "of 13/14/18/19/20/22/23". All now read "of 28".
  - Final sweep: 23/23 files pass, hub links and previews all resolve, zero Arabic-Indic characters remain.

### 2026-05-26
- **Built Screen 23 — Request Tracking** (`23-request-tracking.html`) — the guest-facing side of decision D13. One service request tracked live: a status timeline (Submitted → Assigned → In progress → Completed) with SLA timing, the assigned specialist, an accountability card (the request is seen live by guest + staff + management), and a request comment thread. Completes the services flow — catalog → detail → composer → tracking. Bilingual EN/AR, validated with jsdom (zero JS errors, all 60 toggle ids resolve). Linked into `index.html` (now 23 screens; Guest App → 13); screen 23 added to Project 1's screen list in the proposal, PDF regenerated.
- **Built a client-facing integrations guide as a PDF** — `Escale-Integrations-Guide.pdf` (source: `integrations-guide.html`). A brand-styled Arabic PDF (9 pages) for the client: it explains every external integration the platform connects to, **organized by the 3 projects in implementation order** (Project 1 first, then 2, then 3) — for each integration: what it is, how it's connected, and what the client must set up or contract. **No pricing anywhere** — costs stay only in the internal `integrations.md`. Ends with a per-project client setup checklist. Project 1 carries 8 integrations (hosting, SMS/OTP, email, push & app-store accounts, smart lock, identity, guest registration, AI); Project 2 adds 3 (payments, e-invoicing, maps); Project 3 adds none (reuses the foundation). Egypt/Gulf splits kept where relevant. Verified page-by-page — 9 clean pages, scanned to confirm zero pricing/cost terms.
- The two integration files now have distinct roles: `integrations.md` = **internal** reference with 2026 cost figures (for budgeting); `Escale-Integrations-Guide.pdf` = **client-facing** deliverable, by project, no pricing.

### 2026-05-25
- **Re-corrected the target market to Egypt + the Gulf (GCC), together** (client direction — superseding the earlier same-day "Saudi launch market" framing). Revised D10; updated `DOCUMENTATION.md`, `PROJECT_CONTEXT.md`, `CLAUDE.md`, `design-system.md` (brand references now span Egypt + the Gulf — Cairo, AlUla, the Red Sea; single-currency examples → multi-currency), and `integrations.md` (every market-specific integration — payments, identity, e-invoicing, guest registration, SMS — now covers both Egypt and the Gulf, pluggable per market; the Egypt side researched via a sub-agent — Paymob/Fawry/Meeza, ETA e-invoicing, Ministry-of-Tourism guest registration). The 20 earlier screen prototypes had their example data diversified to span both markets (Cairo, Alexandria, El Gouna, Aswan, Luxor + Riyadh, Jeddah, AlUla, Dubai; EGP/SAR/AED in context) — done via a sub-agent, all 15 files re-validated with jsdom (zero JS errors). The proposal's Project-2 payment line generalised for both markets.
- **Recorded three new feature decisions from the client — D11, D12, D13** (see §3): full pre-arrival online check-in + a room-occupancy notification on door-open; 3-channel live chat; service-request accountability & live tracking.
- **Built Screen 21 — Online Check-in** (`21-online-checkin.html`): the pre-arrival check-in flow — a check-in checklist (identity / details / payment / arrival time), the assigned room, and the QR digital key issued in advance, with a note that the first door-open marks the room occupied on the ops dashboard. Bilingual EN/AR, validated with jsdom (zero JS errors, all toggle ids resolve).
- **Built Screen 22 — Live Chat** (`22-live-chat.html`): in-app chat — a Staff/Management channel switch, a guest↔hotel conversation thread with an inline service-request card and a typing indicator, a "message management" escalation card, and a sticky input bar. Bilingual EN/AR, validated with jsdom.
- **Added the room-occupancy notification to Screen 07** (Operations): a live "digital key" alert — when a guest opens the door, the dashboard shows the room as just-occupied. Bilingual, validated with jsdom. Linked screens 21–22 into the `index.html` review hub (now 22 screens; Guest App count → 12; counts, legend and date updated).
- **Updated the proposal** (`proposal.html`): generalised the Project-2 payment line for Egypt + the Gulf, and added the new features (pre-arrival online check-in, live chat, service-request tracking) to Project 1's scope, with screens 21–22 added to Project 1's screen list. Total unchanged at **$9,000** per Ahmed. `Escale-Project-Proposal.pdf` regenerated and re-verified — 7 clean pages, project totals and installment columns still sum to $9,000.
- **Researched and documented the platform's external integrations and their costs** in a new client-facing file, `integrations.md` (Arabic, per Ahmed's request — he wants the client to see which integrations are needed and what they cost). Covers 11 areas: the smart-lock / QR digital key (decision D9), payment gateways, Nafath, ZATCA/Fatoora, Shomoos, SMS, email, maps, push & app-store accounts, hosting, and the AI API. For each: what it is, how it's integrated, the cost, and who pays — distinguishing Escalé's build cost (in the project price) from the client's ongoing / setup / hardware costs. Includes a cost-summary table and source links. Two parallel research sub-agents gathered current 2026 vendor pricing. Key calls: integrate a **unified lock API (Seam, ~$5/door/month)** starting with PIN-code unlock — BLE "tap to unlock" via enterprise vendor SDKs (ASSA ABLOY/SALTO/dormakaba) is a priced-separately later phase; use **one Saudi payment PSP (Moyasar/HyperPay) + Stripe** for international cards; Nafath and BNPL (Tabby/Tamara) are post-MVP. Lock hardware (~$800–$3,500 installed/door) is the hotel's cost. Nafath, Shomoos, BNPL and the big lock vendors have no public pricing — flagged as "quote required".
- **Reworked `design-system.md` brand references for the Saudi market** (§6 next-step 4, decision D10): aesthetic line "Editorial Hospitality with North African Soul" → "…with an Arabian Soul"; the imagery "a restored Cairene townhouse / a Siwa retreat" → "a restored Najdi mud-brick house in Diriyah / an AlUla retreat among the sandstone cliffs"; "a 3-star city hotel in Cairo to a luxury resort in Sharm" → "…in Riyadh to a luxury Red Sea resort"; brand pillar "Egyptian/North African motifs" → "Najdi/Arabian motifs"; §10 currency examples `EGP`/`ج.م` → `SAR`/`ر.س`. Design philosophy, color tokens and typography left untouched per the request — the "Cairo" name in §3 is the Google font, not the city, so it stayed.
- **Fixed the proposal payment line** (§6 next-step 3): `proposal.html` Project 2 (Bookings App) scope read "بوابات الدفع المصرية (Paymob و Fawry)" — changed to "بوابات الدفع السعودية (mada و Apple Pay)". Regenerated `Escale-Project-Proposal.pdf` with weasyprint and verified page-by-page: **7 clean pages**, the payment line shows mada/Apple Pay, project totals ($5,700+$2,100+$1,200) and the installment columns ($2,450+$3,600+$2,950) both still sum to exactly **$9,000**, no text clipping, no orphaned headers.
- Studied competitor **Duve** (`duve.com`) at the client's request — the client cited Duve as strong. Duve is a global guest-experience platform (1,050+ hotels, 64 countries, raised $60M Dec 2025); its strengths are online check-in, a data-driven upsell engine, omnichannel guest communication, and AI agents. **Key finding:** Duve is a *layer on top of the hotel's PMS*, not a full system, and has **no booking engine and no B2B marketplace** — so Escalé's full-stack scope, the B2B auction, Arabic-first fit, and operational depth are genuine differentiators.
- Produced two deliverables: `Duve-Competitive-Analysis.md` (detailed internal analysis — capability matrix + action plan) and `Escale-vs-Duve-Summary.pdf` (a 2-page bilingual client-ready summary).
- Action plan: adopt 4 proven Duve ideas — full pre-arrival check-in, a smart/data-driven upsell engine, a unified WhatsApp inbox, and real use of `packages/ai`; double down on the differentiators (B2B auction, full system, Arabic-first, operations depth).
- Recorded decision **D9** — the digital/QR key is a smart-lock vendor integration, not built in-house.
- **Corrected the target market to Saudi Arabia** (the client is Saudi-based; the product launches there; the platform stays multi-market). Updated `PROJECT_CONTEXT.md` and `CLAUDE.md` — payments (mada / Apple Pay / STC Pay), SMS (Unifonic), timezone (`Asia/Riyadh`), glossary (ZATCA, Shomoos, PDPL, mada), brand voice ("Arabian Soul").
- **Saudi-ized the example data across all 16 screen prototypes** — Egyptian places, hotels, agencies and currency (Cairo, Siwa, Maadi, the Nile, EGP, Paymob/Fawry…) replaced with Saudi equivalents (Riyadh, Jeddah, AlUla, Al Olaya, Diriyah, SAR, mada/Apple Pay…). Adrère Amellal → Dar AlUla; Cleopatra Spa → Najd Spa; the screen-20 spa theme moved from Siwa salt to AlUla volcanic stone; etc. Done partly via a delegated sub-agent, partly directly. All 16 screens re-validated with jsdom — EN/AR toggle, zero JS errors. Recorded decision **D10**.

### 2026-05-24
- Built **Screen 19 — Offers** (`19-offers.html`): the last screen the client asked for. A guest-mobile discovery surface that markets a hotel's in-stay services — dining, spa & wellness, pools/day passes, experiences — as bookable offers. Layout: a location header, a featured-offer brand-gradient card, a category filter row, and a "Near you" list of four offer cards. Every offer is tagged open to guests *and* external visitors ("guests & visitors"), plus a closing pay-in-app / show-the-code note. Bilingual EN/AR, matches the design system and existing screens.
- Recorded decision **D7** — the Offers screen targets external customers in the hotel's vicinity, not only guests, to drive incremental service revenue (extends D1).
- Validated `19-offers.html` (jsdom): HTML parses clean, all tags balanced, EN⇄AR toggle runs with zero JS errors, no missing-translation text, no parent-child ID conflict, all 72 toggle targets resolve. 27/27 checks pass.
- Linked Screen 19 into `index.html` under the Guest App section; updated hub counts (Guest App → 9 screens, masthead → "19 screens · 4 apps", legend → "All 19 screens ready", footer date). Hub re-validated — parses clean, 14 cards, zero broken links.
- Built **Screen 20 — Service Detail** (`20-service-detail.html`) at the client's request ("a service-detail screen, like the HK screen"). A guest-mobile screen showing one service in full — hero, key facts (duration / price / rating), an "about" description, a "what's included" checklist, three selectable session options, a "good to know" info list (hours, location, booking, a caution note), and a guest-review pull-quote — with a sticky "Request this service" CTA. Example service: Mountain Salt Massage, continuous with the catalog's featured services. Bilingual EN/AR.
- Recorded decision **D8** — the services flow now has a detail step (catalog/offers → service detail → composer); the detail screen is shared by in-stay guests and external offer visitors.
- Validated `20-service-detail.html` (jsdom): 27/27 checks pass — tags balanced, parses clean, EN⇄AR toggle zero errors, no missing-translation text, no parent-child ID conflict, all 61 toggle targets resolve.
- Linked Screen 20 into `index.html` (Guest App → 10 screens, masthead → "20 screens", legend → "All 20 screens ready"). Hub re-validated — 15 cards, zero broken links.
- Prepared the **client proposal** (`Escale-Project-Proposal.pdf`) — a bilingual AR/EN, on-brand PDF built with the Escalé design system (Fraunces + Tajawal, brand palette; HTML → PDF via weasyprint). $8,000 USD total for the full platform, split into 5 milestones (Foundation $1,400, Web $1,800, Guest App $1,500, Staff App $800, B2B $900) plus a 20% / $1,600 non-refundable deposit. Includes overview, milestone scope, payment schedule, ~14-week timeline, an optional $250/month support retainer, exclusions, terms, and a signature block. Per D3. **Open item:** the "Prepared for / مُقدّم إلى" field is a blank placeholder — the client name still needs to be filled in.
- Verified the proposal PDF: 6 pages, render checked page-by-page — deposit + 5 milestones sum to exactly $8,000, bilingual text correct, no text clipping, no orphaned headers.
- **Revised the proposal** per Ahmed's review: raised the total to **$9,000**, set a **~5-month** timeline (with buffer), and restructured the 5 milestones as *vertical slices* — each milestone now ships a working, standalone product surface, not invisible "foundation" work. M1 (Foundation + Booking Website) is the longest as it carries the shared technical foundation. Added a per-milestone "Deliverable" highlight to each card.
- **Reworked the payment model** per Ahmed: dropped the single 20% deposit; each milestone is now an independent mini-project paid in **3 installments** (start / progress / delivery, ~30/40/30). Upfront kept small. Rebuilt the payment table as a 5-column installment schedule. Raised the monthly retainer to **$400** and reframed it as part-time (~2 hrs/day) support **and ongoing development** (Ahmed offsets the low project price with the higher retainer).
- **Restructured the proposal into 3 projects** per the client's new arrangement — the client regrouped the 20 design screens into 3 projects (superseding the earlier 5-milestone split). **Project 1 (main) — Services Management App** $5,700, screens 6,7,8,9,11,12,13,16,17,18,19,20: the full in-stay journey — login, QR key issue/deactivation, and all hotel services (restaurant, HK, health club, maintenance, cars, bellboy, etc.); carries the shared foundation/API. **Project 2 — Bookings App** $2,100, screens 1,2,3,4,5,10. **Project 3 — B2B Auction** $1,200, screens 14,15. Total unchanged at $9,000; each project paid in 3 installments — P1 $1,500/$2,200/$2,000, P2 $600/$900/$600, P3 $350/$500/$350. Each project card lists its screen numbers, tying the proposal to the design preview the client reviewed. Re-verified: 7 pages, project totals and installment columns both sum to exactly $9,000, no clipping or orphaned headers.

### 2026-05-23
- Read all project docs (`PROJECT_CONTEXT.md`, `architecture.md`, `design-system.md`) and `schema.prisma`.
- Received and organised the full client feedback on all 14 screens.
- Resolved the open B2B question — confirmed reverse-auction model (D2).
- Built `index.html` — bilingual (AR + EN) visual review hub linking all 14 screens with live previews, plus placeholder cards for the 4 upcoming screens.
- Validated `index.html`: HTML parses clean (jsdom), all tags balanced, all 10 screen links and iframe sources resolve, every card opens in a new tab.
- Created this documentation file.
- Built **Screen 15 — B2B Agency Portal** (`15-b2b-agency-portal.html`): the agency side of the reverse auction — an agency posts a request, receives competing hotel offers sorted by price, and accepts the best. Bilingual EN/AR, matches the design system and Screen 14.
- Validated Screen 15 (jsdom): tags balanced, the EN⇄AR toggle runs with zero errors, no parent-child ID conflicts.
- Linked Screen 15 into `index.html` under a new "Agency Portal" section.
- Client confirmed (D6): simplify the design-rationale text in every screen to plain Arabic + English; keep the app UI text untouched.
- Design-notes simplification pass: **complete — all 15 screens done.** Rewrote every screen's design-rationale text (notes sections, side panels, editorial headings) into plain Arabic + English, dropped the design/technical jargon, kept the app UI text untouched. Every screen validated with jsdom — EN/AR toggle works, zero JS errors.
- Built **Screens 16–17 — Login & Sign-in** (`16-17-login.html`): a welcome screen with sign-in choices (Apple, Google, phone) and a phone-number + code sign-in screen. Built with plain language from the start. Validated — toggle works, zero errors. Linked into the hub.
- Built **Screen 18 — Hotel Home** (`18-hotel-home.html`): the in-stay hotel page — hotel info, Wi-Fi, dining venues, a services grid, "good to know" facts, and a concierge prompt. Validated, linked into the hub. **All 18 screens now exist; the hub's placeholder cards are all gone.**
- Applied all **Screen 01 (Discover)** feedback edits — search block with dates + guests, voice-search button, a filters row, a "Most searched" section, a "Recently viewed" section, and review counts on hotel cards. Bilingual, validated (toggle works, zero errors).
- **Screen 13** — added the guest-requested cleaning-time indicator (validated). **Screen 02** — confirmed it already carries a full realistic hotel example, so its conditional request is satisfied with no change.
- **Screen 07** — added the orders-stats strip (completed / in service / overdue / active staff) and a "Send instructions to staff" button. Bilingual, validated, zero errors.
- **Screen 03** — Step 1 now shows a "carried over from your search" note; Step 2 gained Nationality + ID/Passport fields with a check-in note. **Screen 11–12** — each service category now shows an hours/price detail line. Both validated.
- **The full client-feedback round is now applied across all screens.** All 13 screen files validate clean (EN/AR toggle, zero JS errors).
- Final pre-send QA pass: automated check on all 13 screen files + the hub — HTML parses clean, EN/AR toggle runs with zero errors, no missing-translation ("undefined") text, tag balance correct, every hub card link and preview resolves. Simplified the hub legend (every screen is "ready" now). Housekeeping: deleted `visual-preview.html` (a stale duplicate of an old screen 01). The folder now holds 13 screen files + `index.html` and is clean to send.

---

## 6. Next steps

1. Send the updated review hub to the client for a second review pass.
2. Resolve open schema issue D1 — `Booking.userId` for unregistered OTA guests, and (per D7) how an external visitor with no booking at all purchases an offer/service. Both point to a service customer who is not tied to a `Booking`.
3. ~~Finalise the milestone-based client proposal.~~ **Done 2026-05-24** — `Escale-Project-Proposal.pdf`. ~~Update its payment line (Paymob/Fawry → mada/Apple Pay).~~ **Done 2026-05-25** — Project 2 now reads "بوابات الدفع السعودية (mada و Apple Pay)"; PDF regenerated, 7 pages verified. **Still open before sending:** fill in the client-name placeholder ("مُقدّم إلى / Prepared for").
4. ~~Rework `design-system.md` brand references for the Saudi market (Siwa/Cairene townhouse → AlUla, Diriyah/Najdi, the Red Sea). See D10.~~ **Done 2026-05-25.**
5. Competitor (Duve) follow-ups: optional Phase-2 quote for the four "adopt" features (pre-arrival check-in, smart upsell, WhatsApp inbox, AI). See `Duve-Competitive-Analysis.md`.
6. Integrations: `integrations.md` is the **internal** cost reference (Egypt + Gulf, 2026 figures); `Escale-Integrations-Guide.pdf` is the **client-facing** version (by project, no pricing) — built 2026-05-26 to hand to the client so they know which accounts/contracts to arrange. Next: get direct vendor quotes for the items with no public pricing — the big lock vendors, Nafath (via TCC), the Shomoos integrator, BNPL (Tabby/Tamara, valU/Sympl), and the ETA e-invoicing build. Confirm the lock vendor per the first hotels' hardware before development.
7. New features (D11–D13): ~~reflect D13's service-request live tracking in the screens~~ **Done 2026-05-26 — built as Screen 23 (Request Tracking).** ~~the request comment-thread (D13)~~ **Done 2026-08-09 — `ServiceRequestComment` added.** Still open: chat thread/message models (D12), and the door-open → `CHECKED_IN` trigger + ops notification (D11).
8. **Services parity (D14), open items:**
   - Get the client's answers to the five questions in `services-gap-analysis.md` §7 — above all **how a concession's money is settled** (does Escalé collect and pay out, or does the outlet bill the guest directly?) and whether **room charge** is offered by independent operators. This decides whether payout/commission models are needed before development.
   - Design the **admin catalog screens** (G13) — create/edit categories, outlets, sections, items, assets and offers, with an active/inactive flag. The legacy app had these (I9) and Escalé has none designed. A hotel cannot use the platform without them, so this is a real MVP gap, not a nice-to-have.
   - Write the raw migration for the **asset overlap constraint** (`btree_gist` exclusion on `asset_reservations`) — Prisma cannot express it, and without it two guests can book the same hall.
   - Reprice **Project 1** if needed: the proposal's $5,700 was scoped before the cart/order, bookable-asset and outlet work was known. Screens 24–28 should be added to Project 1's screen list in `proposal.html` and the PDF regenerated.
