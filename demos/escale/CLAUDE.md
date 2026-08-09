# Escalé — AI Context

This file gives Claude Code (and any AI assistant) the context it needs to be productive in this codebase. **Read it before generating any code.**

## 1. Project Overview

**Escalé** is a multi-tenant hospitality platform that unifies hotel booking with in-stay guest services. **Target markets: Egypt and the Gulf (GCC), together** — neither is a sole "launch market"; built multi-market (the wider region can follow), so keep market-specific concerns (payment rails, compliance, locale) pluggable per market, never hard-coded. Four product surfaces share one backend:

| Surface | Stack | Audience |
|---------|-------|----------|
| Guest mobile app | Expo (React Native) | Travelers |
| Staff mobile app | Expo (React Native) | Hotel employees |
| Admin web | Next.js | Hotel admins, super admin |
| Public booking web | Next.js | Anonymous visitors, SEO |
| API | NestJS | Everything talks to this |

**Languages:** Multilingual via translation tables. Default `en`. RTL support required (Arabic is a primary language; Hebrew, Farsi, and Urdu may be added). Supported locales are configured in `packages/config/locales.ts` — start with: `en`, `ar`, `fr`, `de`, `es`, `it`, `ru`, `zh-CN`, `ja`, `tr`. Hotels opt into a subset via `Hotel.supportedLocales`.

**Current phase:** MVP (Phase 1). See `docs/roadmap.md` for what's in/out of scope.

---

## 2. Tech Stack (Authoritative)

| Layer | Choice |
|-------|--------|
| API framework | NestJS 10 + TypeScript |
| Database | PostgreSQL 16 |
| ORM | Prisma |
| Cache / Queues | Redis + BullMQ |
| Search | Meilisearch |
| Mobile | Expo (React Native), SDK 52+ |
| Web | Next.js 15 (App Router) |
| UI library | shadcn/ui + Tailwind CSS |
| Mobile styling | NativeWind |
| Forms | React Hook Form + Zod |
| Server state | TanStack Query |
| Client state | Zustand |
| Auth | Better Auth + JWT (RBAC) |
| Payments | Pluggable per market. Egypt: Paymob / Fawry (cards, Meeza, mobile wallets). Gulf: mada, Apple Pay, STC Pay (+ Tabby/Tamara BNPL). Stripe for international cards. |
| File storage | Cloudflare R2 |
| AI | Claude API (via `packages/ai`) |
| Email | Resend |
| SMS | Pluggable per market: Unifonic (Gulf), a local aggregator (Egypt), Twilio (international) |
| Push | Expo Push Notifications |
| Monitoring | Sentry + PostHog |
| Monorepo | Turborepo + pnpm workspaces |
| Deployment | Railway/Fly (API), Vercel (web), EAS (mobile), Cloudflare (CDN) |

**Do not introduce new top-level dependencies without explicit discussion.**

---

## 3. Monorepo Structure

```
escale/
├── apps/
│   ├── api/                NestJS API (the only backend)
│   ├── guest-mobile/       Expo - guest-facing
│   ├── staff-mobile/       Expo - hotel staff
│   ├── admin/              Next.js - hotel admin + super admin
│   └── booking-web/        Next.js - public marketing/booking site
├── packages/
│   ├── db/                 Prisma schema + generated client
│   ├── types/              Shared TypeScript types
│   ├── validators/         Zod schemas (shared FE/BE)
│   ├── ui/                 shadcn components + design tokens
│   ├── ai/                 Claude API wrapper
│   └── config/             Shared ESLint, TS, Tailwind configs
├── docs/                   Architecture, ADRs, runbooks
├── .github/workflows/      CI
├── turbo.json
├── pnpm-workspace.yaml
└── CLAUDE.md               This file
```

---

## 4. Core Domain Concepts

### 4.1 Multi-tenancy

Every business entity belongs to a **Hotel** (the tenant). The `hotelId` column is mandatory on tenant-scoped tables.

**Two layers of isolation:**

1. **Application layer.** Every service method receives `hotelId` from the auth context and filters explicitly. **NEVER write a query against a tenant-scoped table without a `hotelId` filter.** This is non-negotiable.
2. **Database layer.** PostgreSQL Row-Level Security (RLS) policies enforce isolation as a safety net. See `packages/db/migrations/rls.sql`.

**User types and tenancy:**

| User type | Tenancy |
|-----------|---------|
| `Guest` | Not bound to a hotel. Books across many hotels. |
| `HotelStaff` | Bound to exactly one hotel via `HotelStaff` table. |
| `B2BUser` | Bound to a B2B company; transacts with many hotels. |
| `SuperAdmin` | Platform-level. Bypasses RLS via service role. |

### 4.2 RBAC

```
GUEST
B2B_USER, B2B_ADMIN
HOTEL_STAFF, HOTEL_SUPERVISOR, HOTEL_MANAGER, HOTEL_ADMIN
SUPER_ADMIN
```

Permission checks live in NestJS guards (`apps/api/src/auth/guards/`). Roles map to permission sets in `apps/api/src/auth/permissions.ts`. Always check permissions in a guard, never inline in a controller.

### 4.3 Booking State Machine

```
DRAFT → PENDING_PAYMENT → CONFIRMED → CHECKED_IN → CHECKED_OUT → COMPLETED
                                   ↘ CANCELLED
                                   ↘ NO_SHOW
```

Transitions are enforced in `BookingService.transition()`. **Never update `Booking.status` directly from a controller, repository, or anywhere else.** All state changes go through the service method that validates the transition.

### 4.4 Service Request Lifecycle

```
SUBMITTED → ASSIGNED → IN_PROGRESS → COMPLETED
                                  ↘ ESCALATED
                                  ↘ CANCELLED
```

Requests are time-tracked against an SLA, and the guest can post live comments on a request while it is in progress — comments are visible to hotel management. Because guest, staff, and management all see the request live, accountability is built in. See decision D13 in `DOCUMENTATION.md`.

Departments that need a delivery leg (room service) use the optional `stage` field — `PREPARING → READY → OUT_FOR_DELIVERY → DELIVERED` — **inside** `IN_PROGRESS`. `stage` is presentation detail; it is never a substitute for a status transition, and `BookingService`-style transition validation applies only to `status`.

### 4.4a In-hotel outlets and service orders

A service belongs to a **`ServiceOutlet`** — a restaurant, spa, salon, halls complex, or transport desk that sits inside one hotel. An outlet is either `HOTEL_OPERATED` or a `CONCESSION` (an independent business leasing space inside the property). The guest journey is identical for both; only settlement differs. Never attach a catalog item straight to a `Hotel` when an outlet exists.

`ServiceRequest` is the single order model for every shape of service request. `type` selects which field groups apply:

| `type` | Used for | Field groups that apply |
|--------|----------|-------------------------|
| `INSTANT` | "extra towels" | none beyond the basics |
| `ORDER` | restaurant, minibar | `lines`, money totals |
| `APPOINTMENT` | spa, salon, maintenance visit | `scheduledFor`, `scheduledEndAt`, `lines` |
| `ASSET_BOOKING` | a hall, a cabana | `AssetReservation`, `scheduledFor/EndAt` |
| `TRANSFER` | a vehicle trip | `tripType`, `destination`, `passengers` |

Rules:
- **Line items are snapshots.** `ServiceRequestLine.nameSnapshot` and `unitPrice` are copied at order time. Never resolve a past order's price or name through the live catalog.
- **Totals are computed server-side**, never trusted from the client, and always `Decimal`.
- **A `BookableAsset` can be held by one reservation at a time.** Overlap is prevented by a Postgres exclusion constraint in `packages/db/migrations/asset_overlap.sql`, not by an application-level read-then-write.
- `ServiceRequest.userId` is **nullable** — an OTA/walk-in guest or an external visitor (D7) orders with `guestName` + `guestPhone`. Exactly one of the two identifications must be present; enforce it in the service layer.

See decision D14 in `DOCUMENTATION.md` and `services-gap-analysis.md`.

### 4.5 Room Operational Status

```
VACANT_CLEAN ↔ VACANT_DIRTY ↔ OCCUPIED ↔ INSPECTED
                                       ↓
                                  OUT_OF_ORDER ↔ OUT_OF_SERVICE
```

### 4.6 Online check-in & digital key

Guests can complete **check-in before arrival** — identity verification, guest details, room assignment — and receive the QR digital key in advance, so on arrival they go straight to their room. The digital-key door-open event is a trigger for the `CONFIRMED → CHECKED_IN` booking transition and fires an operations-dashboard notification that the room is now occupied. The digital key itself is a smart-lock vendor integration, not built in-house. See decisions D9 and D11 in `DOCUMENTATION.md`.

### 4.7 Live chat

Real-time in-app messaging across three channels: guest ↔ staff, guest ↔ hotel management, and management ↔ staff. Threads are hotel-scoped (`hotelId` mandatory). Messages are user-generated content (single language, optional `language`). See decision D12 in `DOCUMENTATION.md`.

---

## 5. Conventions

### Naming
- **Files:** kebab-case (`booking.service.ts`)
- **Classes:** PascalCase (`BookingService`)
- **Variables / functions:** camelCase
- **Enums (TS) and constants:** SCREAMING_SNAKE_CASE
- **DB tables:** snake_case
- **DB columns:** snake_case (mapped to camelCase in Prisma)

### Multilingual content

There are three content categories. Each is stored differently:

**1. Curated content (admin-entered, must support all locales the hotel offers)**

Hotel info, room type names/descriptions, service names, rate plan names, amenity names, service category names, **outlet names/descriptions, menu section names, bookable asset names, offer titles**. Stored in dedicated translation tables:

```prisma
model HotelTranslation {
  hotelId String
  locale  String  // BCP 47: "en", "ar", "fr-FR", "zh-CN"
  name    String
  description String?
  ...
  @@unique([hotelId, locale])
}
```

When reading: API resolves the active locale (see chain below) and joins the right translation row. When writing: the admin endpoint accepts all translations at once.

**2. User-generated content (single language)**

Reviews, service request descriptions, internal notes, special requests, booking guest names. Stored in a single column. An optional `language String?` column captures BCP 47 for analytics or future auto-translation:

```prisma
title    String?
comment  String?
language String?  // optional
```

Don't try to translate user content automatically in MVP — store as written.

**3. System-generated content (rendered at send time)**

Notifications and emails. Templates live in `packages/notifications/templates`, versioned in code. At send time:
- Resolve recipient's locale (preferred → Accept-Language → default)
- Render the template
- Store the rendered `title`, `body`, and `locale` in the `Notification` row

This means a notification row is a snapshot — never re-rendered.

### Locale resolution chain

In the API, every request resolves an active locale via this priority:

1. `?locale=ar` query parameter (explicit override)
2. Authenticated user's `User.preferredLocale`
3. `Accept-Language` header (parsed and matched against supported locales)
4. `Hotel.defaultLocale` (for hotel-scoped endpoints)
5. Platform default: `"en"`

The resolved locale is attached to the request context and used by every service method that returns translatable content.

### Locale validation

Supported locales are defined in `packages/config/locales.ts` as a TypeScript const array. A Zod validator (`localeSchema`) enforces input. Never accept arbitrary strings as locales from clients.

### RTL handling

The `dir` attribute is set at the HTML root from the active locale. RTL locales are listed in `packages/config/locales.ts`. Tailwind logical properties (`ps-`, `pe-`, `start-`, `end-`) are required everywhere — no `pl-` / `pr-` / `left-` / `right-`.

### Money
- Always `Decimal` in Prisma. **Never `Float`.**
- Always store `currency` (ISO 4217) alongside `amount`.
- For external APIs (Paymob, Stripe), convert to smallest unit at the boundary.
- Shared type: `Money { amount: Decimal; currency: string }`.

### Dates & times
- All timestamps stored in **UTC**.
- `Booking.checkInDate` / `checkOutDate` are `Date` (no time) — they're hotel calendar dates, not moments.
- Each hotel has its own `timezone` (IANA, e.g., `Africa/Cairo`, `Asia/Riyadh`). Display in the hotel's timezone for staff, user's timezone for guests.

### IDs
- Primary keys: `String @id @default(cuid())`
- Public-facing references (booking confirmation): short codes like `BK-X7Y2A9` generated server-side, stored in a `reference` column.

### Soft delete
Use `deletedAt DateTime?` on user-impacting tables. **Never hard-delete user data.** Add a Prisma middleware to scope queries to non-deleted rows by default.

### Audit fields
Every business entity:
```prisma
createdAt   DateTime  @default(now())
updatedAt   DateTime  @updatedAt
createdById String?
```

---

## 6. API Conventions

### REST structure
- Resources are plural: `/bookings`, `/rooms`
- Nest where natural: `/hotels/:hotelId/rooms`
- State actions: `POST /bookings/:id/cancel`, `POST /bookings/:id/check-in`

### Response shape
```json
{
  "data": { ... },
  "meta": { "pagination": { ... } }
}
```

### Error shape
```json
{
  "error": {
    "code": "BOOKING_OVERLAP",
    "message": "Room is not available for the selected dates",
    "messageAr": "الغرفة غير متاحة في التواريخ المختارة",
    "details": { ... }
  }
}
```

All errors extend `AppError`. Error codes live in `apps/api/src/common/errors/error-codes.ts`.

### Pagination
- **Cursor-based** for lists that grow unbounded (bookings, transactions, notifications).
- **Offset-based** acceptable for admin tables capped at ~10k rows.

### Versioning
URL-prefix versioning: `/v1/bookings`. Bump only on breaking changes.

---

## 7. How to Add a New Module

1. Create `apps/api/src/modules/[name]/` with:
   - `[name].module.ts`
   - `[name].controller.ts` (HTTP layer only — no business logic)
   - `[name].service.ts` (business logic)
   - `[name].repository.ts` (Prisma queries, only if complex)
   - `dto/` (request/response DTOs with class-validator)
   - `[name].service.spec.ts` (unit tests)
2. Add Zod schemas to `packages/validators/[name].ts`
3. Add shared types to `packages/types/[name].ts`
4. Register the module in `app.module.ts`
5. Add at least one E2E test for the happy path
6. Update OpenAPI tags (auto-generated from decorators)

---

## 8. Testing Strategy

| Layer | Tool | Target |
|-------|------|--------|
| Unit (services) | Vitest | 80%+ coverage on business logic |
| Integration (API) | Vitest + Testcontainers | Critical paths with real DB |
| E2E (web) | Playwright | Top user flows |
| E2E (mobile) | Detox | Booking, check-in, service request |

Run `pnpm test` before any PR. CI blocks merge on test failures or coverage regression.

---

## 9. What to AVOID

- ❌ Writing raw SQL unless absolutely necessary (use Prisma; if you must, add a comment explaining why)
- ❌ Bypassing guards or auth checks "for testing" — use test-only auth helpers instead
- ❌ Business logic in controllers — controllers are HTTP glue
- ❌ Using `any` in TypeScript — use `unknown` and narrow
- ❌ Hard-deleting user data
- ❌ Storing secrets in code or `.env` committed files
- ❌ Querying tenant tables without a `hotelId` filter
- ❌ Hardcoding locale strings in business logic — resolve from request context
- ❌ Storing curated content in a single column when it must support multiple locales — use a translation table
- ❌ Auto-translating user-generated content in MVP — store as written, with `language` if known
- ❌ Using `pl-`/`pr-`/`left-`/`right-` Tailwind classes — use logical properties for RTL safety
- ❌ Using `Float` for money
- ❌ Creating global mutable state outside Zustand stores
- ❌ Adding dependencies without justification — check bundle size for client libs
- ❌ Mixing `then/await`, `var`, or `function` declarations — use `const` + `async/await` consistently
- ❌ Calling external APIs directly from controllers — wrap in a service / client class
- ❌ Sending raw error messages to clients — translate via the `AppError` system

---

## 10. AI Integration Rules

The Claude API is called only through `packages/ai`. Reasons:
- Single place for prompt versioning
- Single place for cost tracking and rate limiting
- Easy to swap providers if needed
- Built-in PII redaction before sending

Never call the Claude API directly from a controller or React component.

---

## 11. Questions to Ask Before Writing Code

Before generating significant code, confirm:

1. **Tenant scope:** Does this touch tenant-scoped data? If yes, where does `hotelId` come from in this code path?
2. **Authorization:** What roles are allowed to perform this action? Is there an existing guard?
3. **State machine:** Does this change a booking, request, or task status? Does the transition exist?
4. **Translatable content:** Are there user-facing strings? Which category — curated (translation table), user-generated (single + optional language), or system (rendered at send time)?
5. **Locale resolution:** Where does the active locale come from in this code path? Is the fallback chain wired up?
6. **Audit:** Should this action create an audit log entry?
7. **Idempotency:** If a payment or external write is involved, is it idempotent?

If any answer is unclear, stop and ask before writing code.

---

## 12. Local Dev

```bash
pnpm install
pnpm db:up          # docker-compose: postgres + redis + meilisearch
pnpm db:migrate     # run prisma migrations
pnpm db:seed        # seed with a demo hotel + users
pnpm dev            # turbo run dev (API + admin in parallel)
```

Default URLs:
- API: `http://localhost:3000`
- Admin: `http://localhost:3001`
- Booking web: `http://localhost:3002`
- API docs (Swagger): `http://localhost:3000/docs`

Env vars are documented in `.env.example`. Use **Doppler** or **1Password CLI** for secrets — never commit `.env`.

---

## 13. Glossary

- **HK** — Housekeeping
- **OOO** — Out of Order (room status)
- **OOS** — Out of Service
- **F&B** — Food and Beverage
- **B2B** — Business-to-business (corporate bookings, travel agencies)
- **OTA** — Online Travel Agency (Booking.com, Expedia, etc.)
- **RatePlan** — A priced offering tied to a RoomType (BAR, NRF, packages, B2B-only)
- **MealPlan** — Room-only, BB, HB, FB, All-Inclusive
- **PMS** — Property Management System (what hotels use internally)
*Gulf market:*

- **Nafath** — Saudi national digital-identity service (planned integration; relevant for online check-in / guest verification)
- **ZATCA / Fatoora** — Saudi mandatory e-invoicing (Zakat, Tax and Customs Authority). Planned integration if the platform issues invoices
- **Shomoos** — Saudi Ministry of Interior platform for hospitality establishments to register guests. Planned integration
- **PDPL** — Saudi Personal Data Protection Law (regulator: SDAIA). Governs handling of guest data
- **mada** — Saudi national debit-card network; the essential local payment rail

*Egypt market:*

- **Meeza** — Egypt's national card scheme; widely held, accept it alongside Visa/Mastercard
- **Paymob / Fawry** — the two main Egyptian payment gateways (cards, Meeza, mobile wallets, cash/reference codes)
- **ETA** — Egyptian Tax Authority; runs the mandatory e-invoice (B2B) and e-receipt (B2C) systems
- **InstaPay** — Egypt's bank-to-bank instant payment network (consumer app)

Integration approach and 2026 cost figures for both markets are documented in `integrations.md`.
