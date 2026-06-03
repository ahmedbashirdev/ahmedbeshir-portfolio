# Escalé — Architecture Overview

This is the high-level architecture document. For coding conventions, see `CLAUDE.md`.

## 1. System Diagram

```
                                    ┌─────────────────────────────────────┐
                                    │           Cloudflare CDN            │
                                    │   (CDN + WAF + DDoS Protection)     │
                                    └──────────────────┬──────────────────┘
                                                       │
        ┌──────────────────────────────────────────────┼───────────────────────────────────────────┐
        │                                              │                                           │
        ▼                                              ▼                                           ▼
┌──────────────────┐                          ┌──────────────────┐                       ┌──────────────────┐
│  Guest Mobile    │                          │  Staff Mobile    │                       │   Admin Web      │
│  (Expo / RN)     │                          │  (Expo / RN)     │                       │   (Next.js)      │
└────────┬─────────┘                          └────────┬─────────┘                       └────────┬─────────┘
         │                                             │                                          │
         └────────────────────────┬────────────────────┴──────────────────────────────────────────┘
                                  │
                                  │  HTTPS / WebSocket
                                  ▼
                  ┌───────────────────────────────────────────────┐
                  │            NestJS API (api.escale.com)        │
                  │                                               │
                  │   ┌─────────────┬─────────────┬───────────┐   │
                  │   │   Modules   │   Guards    │  Filters  │   │
                  │   ├─────────────┼─────────────┼───────────┤   │
                  │   │  Auth       │  Hotels     │  Bookings │   │
                  │   │  Rooms      │  Services   │  Payments │   │
                  │   │  Tasks      │  Reviews    │  Loyalty  │   │
                  │   │  Notifs     │  B2B        │  Admin    │   │
                  │   └─────────────┴─────────────┴───────────┘   │
                  │                                               │
                  └─────┬─────────┬─────────┬───────────┬─────────┘
                        │         │         │           │
                        ▼         ▼         ▼           ▼
                 ┌──────────┐ ┌──────┐ ┌──────────┐ ┌─────────────┐
                 │ Postgres │ │Redis │ │Meilisearch│ │ Object Store│
                 │   (RDS)  │ │      │ │  (search) │ │  (R2)       │
                 │   + RLS  │ │BullMQ│ │           │ │             │
                 └──────────┘ └──────┘ └──────────┘ └─────────────┘

                 External services (over HTTPS, called from API layer only):
                 ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
                 │  Paymob  │ │  Stripe  │ │  Resend  │ │  Twilio  │ │  Claude  │
                 │ (Payment)│ │ (Payment)│ │  (Email) │ │  (SMS)   │ │   API    │
                 └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
```

## 2. Architectural Principles

### Modular Monolith (for now)
We start as a modular monolith for a reason: it's faster to ship, easier to refactor, and we don't yet know the right service boundaries. The modular structure (clean module boundaries, no cross-module DB access, services talking through interfaces) makes future extraction painless if we ever need it.

**Heuristic for extracting a microservice:** Only when a module has independent scaling needs (e.g. AI processing) or a different team owning it. Until then, stay monolithic.

### One Backend, Many Clients
All four client apps (guest mobile, staff mobile, admin web, booking web) talk to the same NestJS API. No backend-for-frontend (BFF) layer in MVP. We may add one if Next.js server components need heavy data orchestration.

### Database as Source of Truth
- Postgres is the only durable store. Redis is purely a cache / queue.
- Meilisearch is reindexed from Postgres; never write to it directly from product code.
- File storage (R2) stores blob content; metadata lives in Postgres.

### Tenant Isolation at Two Layers
Application code filters by `hotelId`. RLS policies in Postgres enforce isolation as a defense-in-depth measure. **Both layers are required.** Code reviews check that every tenant-scoped query has a `hotelId` filter; RLS ensures that even a bug can't leak data.

### Idempotency for Side Effects
Any external call that costs money or sends a message (payments, SMS, push notifications, emails) must be idempotent. Use idempotency keys for outbound calls and unique constraints + upserts for inbound webhooks.

## 3. Module Boundaries

Each NestJS module owns its data and exposes a service interface that other modules consume. Cross-module data access happens only through service calls — never by querying another module's tables directly.

```
auth          → users, sessions, RBAC
hotels        → hotels, images, amenities
rooms         → room_types, rooms, rate_plans
inventory     → availability, rate, restrictions (computed)
bookings      → bookings, booking_rooms, booking_guests, state machine
payments      → payments, payment provider integrations
services      → service_categories, hotel_services, service_requests
tasks         → staff_tasks
reviews       → reviews
loyalty       → loyalty_accounts, loyalty_transactions
notifications → notifications, delivery (push/email/sms)
search        → Meilisearch sync, query layer
b2b           → B2B companies, auctions, offers (Phase 2)
ai            → Claude API wrapper, prompt management
```

## 4. Critical Flows

### 4.1 Search → Book → Pay → Confirm

```
1. Guest searches → search module (Meilisearch) returns hotel list
2. Guest selects hotel → hotels module returns details, room types
3. Guest selects dates + rooms → inventory module checks availability
4. Booking created in DRAFT → bookings module
5. Guest enters payment details → payments module initiates with Paymob/Stripe
6. Booking → PENDING_PAYMENT
7. Payment webhook arrives → payments module marks payment SUCCEEDED
8. payments module fires BookingPaidEvent
9. bookings module transitions booking → CONFIRMED
10. notifications module sends confirmation (push + email)
```

**Key properties:**
- Booking is created BEFORE payment. Allows abandoned-cart recovery.
- Booking transition to CONFIRMED is event-driven. Webhooks can arrive late or twice.
- All async work runs through BullMQ to survive restarts.

### 4.2 Check-in with QR Key

```
1. Booking is CONFIRMED. checkInDate arrives.
2. Guest opens app → sees check-in available.
3. Guest taps "Check in" → bookings module:
   - Validates identity (national ID / passport via BookingGuest)
   - Assigns physical room (rooms module)
   - Transitions booking to CHECKED_IN
   - Updates room.status = OCCUPIED
4. bookings module fires GuestCheckedInEvent
5. notifications module generates QR key payload, signs it, sends via push
6. Guest scans QR at door → external lock system validates signature
```

### 4.3 Guest Service Request

```
1. Guest submits request via app → services module
2. ServiceRequest created in SUBMITTED state
3. tasks module auto-creates StaffTask, assigns based on:
   - Service.defaultDepartment
   - Current shift staffing
   - Workload balance
4. Push notification sent to assigned staff
5. Staff acknowledges → ASSIGNED → IN_PROGRESS
6. Staff completes → COMPLETED
7. Guest sees status updates in real-time (WebSocket)
8. Guest rates the service
```

## 5. Authentication & Authorization

### Authentication
- **Mobile:** OAuth (Apple, Google) or phone-OTP. JWT access token (15 min) + refresh token (30 days, rotated on use).
- **Web admin:** Email/password + 2FA (TOTP) for HOTEL_ADMIN and above.
- **B2B:** Email/password + optional SSO (Phase 3).

### Session storage
Refresh tokens stored in `sessions` table. Revocation by deleting / marking revoked. Access tokens are stateless JWTs.

### Authorization (RBAC)
- Roles are stored on the user-hotel link (`HotelStaff`) for staff, on the user for guests.
- Permission set is computed at login and embedded in the JWT.
- Guards check permissions on every endpoint.
- Tenant scoping: every JWT carries `hotelId` (for staff) or none (for guests). Service methods always re-validate against the database.

## 6. Real-time

Socket.io server inside the NestJS API. Redis adapter for horizontal scaling.

**Channels:**
- `user:{userId}` — guest's personal channel (notifications, request updates)
- `hotel:{hotelId}:staff` — broadcast to all hotel staff
- `hotel:{hotelId}:dept:{department}` — department-scoped
- `request:{requestId}` — service request thread

Authorization on connect: validate JWT, attach `userId` and (if staff) `hotelId`, restrict channel subscription.

## 7. Background Jobs

BullMQ queues (one Redis instance, multiple queues for separation):

| Queue | Job examples |
|-------|--------------|
| `email` | Booking confirmation, password reset, review request |
| `sms` | OTP, booking SMS, check-in code |
| `push` | All push notifications |
| `payments` | Webhook processing, refund execution |
| `search-sync` | Reindex on hotel/room changes |
| `analytics` | Aggregation jobs |
| `ai` | Claude API calls for non-real-time tasks |
| `scheduled` | Cron-style: daily reports, expiring loyalty points |

Jobs are idempotent. Retries with exponential backoff. Dead-letter queue for permanent failures.

## 8. Search

Meilisearch indexes (one per hotel? no — one global, filtered by hotelStatus=ACTIVE):

- `hotels` — full-text on names, descriptions, city, country, amenities
- `bookings` — admin-side search

Reindexing strategy: Postgres triggers fire NOTIFY on row changes; a worker subscribes, dedupes, and updates Meilisearch in batches.

Search filters available:
- Location (city, country, lat/lng radius)
- Dates (availability check happens AFTER Meilisearch returns candidates)
- Price range
- Star rating
- Amenities
- Category

## 9. AI Integration

The AI module wraps Claude API. Responsibilities:
- Prompt versioning (prompts as code, in `packages/ai/prompts/`)
- Token usage and cost tracking per hotel
- Rate limiting per hotel tier
- PII redaction before sending
- Response caching where appropriate

Use cases (MVP):
- Guest chatbot (FAQs, hotel info)
- Booking recommendations
- Review sentiment analysis

Use cases (Phase 2+):
- B2B request matching to hotel inventory
- Anomaly detection in operations
- Auto-generated insights for managers

## 10. Observability

| Concern | Tool |
|---------|------|
| Errors | Sentry (API + mobile + web) |
| Product analytics | PostHog |
| Logs | Pino → CloudWatch / Better Stack |
| Tracing | OpenTelemetry → Honeycomb (Phase 2) |
| Uptime | Better Uptime |

**Logging conventions:**
- Structured JSON logs only
- Always include `traceId`, `userId` (if known), `hotelId` (if applicable)
- Log levels: trace, debug, info, warn, error, fatal
- Never log PII (national IDs, passport numbers, full card numbers)

## 11. Deployment Topology (MVP)

| Component | Where | Why |
|-----------|-------|-----|
| API | Railway or Fly.io | Easy to deploy, autoscale |
| Postgres | Railway managed / Neon | Daily backups, point-in-time recovery |
| Redis | Railway managed / Upstash | |
| Meilisearch | Meilisearch Cloud or self-hosted on Fly | |
| File storage | Cloudflare R2 | Cheap, zero egress |
| Admin / booking web | Vercel | First-class Next.js |
| Mobile | EAS Build → App Store / Play Store | |
| CDN / WAF | Cloudflare (in front of API and web) | |

**Scaling path (when MVP demand justifies):**
- API → multiple instances behind a load balancer (already stateless)
- Postgres → read replicas for analytics / search-sync
- Redis → cluster or split into multiple instances by purpose

## 12. Security

- All traffic HTTPS (TLS 1.2+)
- Helmet middleware on the API (CSP, HSTS, X-Frame-Options)
- Rate limiting per IP and per user
- Input validation via Zod / class-validator at every boundary
- SQL injection: Prisma parameterizes everything
- Secrets in Doppler / 1Password CLI, never in repo
- Payment card data never touches our servers — tokenized at provider
- PII (national IDs, passports) encrypted at rest (column-level encryption)
- Audit log for all admin actions (Phase 2)

## 13. Phasing

### Phase 1 — MVP (3 months)
Search, booking, payment, check-in, basic services, staff tasks, admin dashboard, multilingual (launching with `en` + `ar`, infrastructure ready for 8 more).

### Phase 2 (+2 months)
B2B auction, restaurant POS, real-time chat, loyalty redemption, reviews moderation, additional locale rollouts (`fr`, `de`, `es`, `it`).

### Phase 3 (+2 months)
AI chatbot (locale-aware), all-inclusive module, IoT (door locks, lights, AC), advanced analytics, multi-currency, remaining locales (`ru`, `zh-CN`, `ja`, `tr`).

### Phase 4 (+TBD)
Nafath integration, group bookings, event ticketing, VIP organizer module, auto-translation for user-generated content.
