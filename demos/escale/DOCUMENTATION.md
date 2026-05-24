# Escalé — Living Project Documentation

> The single up-to-date record of what's been done, decided, and what's next.
> Updated after **every** step. For the static project handoff see `PROJECT_CONTEXT.md`;
> for coding conventions see `CLAUDE.md`.

**Last updated:** 2026-05-24

---

## 1. Where the project stands

- Design phase. **20 screens** exist as bilingual HTML/CSS prototypes, all linked from the `index.html` review hub.
- Client returned per-screen feedback; **the full feedback round is now applied** (see §4 — every item checked).
- Design-rationale text simplified to plain Arabic + English across all screens.
- **Screen 19 — Offers** built 2026-05-24: markets in-hotel services (dining, spa, pools, experiences) to hotel guests *and* external visitors nearby. See decision D7.
- **Screen 20 — Service Detail** built 2026-05-24: the full detail page for a single service (description, what's included, session options, hours, a guest review), sitting between the catalog/offers and the request composer. See decision D8.
- Next: a second client review pass, then scope/cost sign-off and development.
- Commercials: not signed yet — milestone-based proposal in preparation.

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
| — | `index.html` | Design Preview hub | Review tool | Built 2026-05-23 |

All 20 screens now exist. The hub links all of them; no placeholders remain.

Note: `visual-preview.html` (a stale duplicate of an old screen 01) was deleted on 2026-05-23.

---

## 3. Key decisions

- **D1 — Services layer is channel-agnostic.** The in-stay services part must serve *every* hotel guest regardless of booking channel (Escalé, Booking.com, Agoda, Musafir, TripAdvisor, walk-in). Each guest gets a link → registers → digital key + services. The schema partly supports this (`Booking.source` enum already has `OTA/WALK_IN/PHONE/B2B`; `ServiceRequest.bookingId` is optional). **Open issue:** `Booking.userId` is required — a hotel creating a stay for an unregistered OTA guest has no user to attach. Needs a "claim your stay" flow or a nullable `userId`.
- **D2 — B2B portal = reverse auction (InDrive-style).** A travel agency posts a booking request (dates, rooms, guests, target rate); hotels see it and submit price bids; the agency picks. Screen 14 is the *hotel* side; the *agency* side is a new screen.
- **D3 — Commercials.** Milestone-based pricing, quoted in **USD** (not EGP — FX risk). ~15–20% non-refundable deposit, each milestone paid on acceptance, plus a monthly maintenance retainer. Third-party/hosting costs billed to the client.
- **D4 — Review hub.** `index.html` built as bespoke HTML following `design-system.md` (the project's own design system is the source of truth; no generic skill fits better).
- **D5 — Client communication.** Client relies on visuals and finds dense text hard. All client-facing material uses simple Arabic + simple English, side by side.
- **D6 — Simplify screen text.** Each screen carries two text layers: the app UI text (the real product — kept as-is, bilingual via toggle) and the design-rationale notes (dense, jargon-heavy). Per client request, the rationale notes in all 15 screens are being rewritten into plain Arabic + English.
- **D7 — Offers screen targets external customers, not just guests.** Screen 19 markets a hotel's in-stay services (restaurants, spa/health club, pools, experiences) as bookable offers. The client's strategic point: booking apps sell only rooms, while hotels have high-margin services that sit half-empty. The offers surface is open to **two audiences** — hotel guests *and* external visitors within the hotel's vicinity ("guests & visitors") — to drive incremental F&B/wellness revenue. This extends D1 (channel-agnostic services) one step further: services are sold not only to guests of any booking channel, but to people who never booked a room at all. Schema implication: an offer purchase by an external visitor has no `Booking` to attach to — flagged in §6.
- **D8 — The services flow now has a detail step.** Screen 20 (Service Detail) is a dedicated full page for a single service — description, what's included, selectable session options, hours/location, a guest review — modelled on the depth of the HK Room Status screen (per the client: "a service-detail screen, like the HK screen"). It sits in the navigation between browsing and requesting: **catalog (11) / offers (19) → service detail (20) → request composer (12).** It is shared by both audiences from D7 — in-stay guests reach it from the catalog, external visitors reach it from an offer. Session/option selection happens here, so the composer stays short.

---

## 4. Client feedback tracker

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

### 2026-05-24
- Built **Screen 19 — Offers** (`19-offers.html`): the last screen the client asked for. A guest-mobile discovery surface that markets a hotel's in-stay services — dining, spa & wellness, pools/day passes, experiences — as bookable offers. Layout: a location header, a featured-offer brand-gradient card, a category filter row, and a "Near you" list of four offer cards. Every offer is tagged open to guests *and* external visitors ("guests & visitors"), plus a closing pay-in-app / show-the-code note. Bilingual EN/AR, matches the design system and existing screens.
- Recorded decision **D7** — the Offers screen targets external customers in the hotel's vicinity, not only guests, to drive incremental service revenue (extends D1).
- Validated `19-offers.html` (jsdom): HTML parses clean, all tags balanced, EN⇄AR toggle runs with zero JS errors, no missing-translation text, no parent-child ID conflict, all 72 toggle targets resolve. 27/27 checks pass.
- Linked Screen 19 into `index.html` under the Guest App section; updated hub counts (Guest App → 9 screens, masthead → "19 screens · 4 apps", legend → "All 19 screens ready", footer date). Hub re-validated — parses clean, 14 cards, zero broken links.
- Built **Screen 20 — Service Detail** (`20-service-detail.html`) at the client's request ("a service-detail screen, like the HK screen"). A guest-mobile screen showing one service in full — hero, key facts (duration / price / rating), an "about" description, a "what's included" checklist, three selectable session options, a "good to know" info list (hours, location, booking, a caution note), and a guest-review pull-quote — with a sticky "Request this service" CTA. Example service: Mountain Salt Massage, continuous with the catalog's featured services. Bilingual EN/AR.
- Recorded decision **D8** — the services flow now has a detail step (catalog/offers → service detail → composer); the detail screen is shared by in-stay guests and external offer visitors.
- Validated `20-service-detail.html` (jsdom): 27/27 checks pass — tags balanced, parses clean, EN⇄AR toggle zero errors, no missing-translation text, no parent-child ID conflict, all 61 toggle targets resolve.
- Linked Screen 20 into `index.html` (Guest App → 10 screens, masthead → "20 screens", legend → "All 20 screens ready"). Hub re-validated — 15 cards, zero broken links.

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
3. Finalise the milestone-based client proposal.
