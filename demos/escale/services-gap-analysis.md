# Escalé — Services Gap Analysis

> Source: 16 screenshots of the **legacy Escalé app** (`services/I1–I16.jpeg`), supplied 2026-08-09.
> Purpose: extract everything the old app did around **in-hotel services** and map it onto the
> current Escalé design + `schema.prisma`, so nothing proven is lost in the rebuild.
>
> Companion docs: `DOCUMENTATION.md` (decision **D14**), `CLAUDE.md` (conventions), `schema.prisma`.

**Created:** 2026-08-09

---

## 1. What the legacy app is

The legacy app is a **hotel services management app** — the guest browses the services that exist
*inside* a hotel and orders/books them, and each department runs its own order queue in a
back-office. It is the direct ancestor of Escalé's Project 1 (Services Management App).

Crucially, an in-hotel service is **not always operated by the hotel**. It can be an independent
business that leases space inside the property — e.g. the Mohamed El Saghir salon inside the
Semiramis InterContinental. Both cases live under the same hotel and appear in the same catalog.
This is the single biggest structural idea missing from the current schema.

---

## 2. Screen-by-screen extraction

| Ref | Legacy screen | What it contains |
|-----|---------------|------------------|
| I1, I16 | **Services hub** — `الخدمات` | 8 image cards: `الاشراف الداخلي` housekeeping · `المطاعم` restaurants · `سيارات` cars · `منتجع صحي` health club · `القاعات` halls · `حامل حقائب` bellboy · `الصيانة` maintenance · `الغرف` rooms. Above them: a search field ("type the hotel name to see its services") + a filter button. |
| I2 | **Housekeeping sub-menu** | The category opens into **four sub-services**: `المغسلة` laundry · `نظافة الغرف` room cleaning · `طلب مستلزمات` amenity request · `المفروشات` linens. Header image, "choose the service you want". Bottom nav swaps `استكشف` → **`طلباتي` (My orders)**. |
| I3 | **Restaurant menu** | Outlet name in the bar (`مطعم فندق بيلاميرا`) + outlet logo. Horizontal **menu-section tabs** (`الكل` · `عروض خاصة` · `بيتزا` · `فطائر` · `مشويات` · `اخري`). Each item card: photo, name, short description, **prep time (`40–45 دقيقة`)**, price (`85.00 جنيه`), **favourite (heart)**, **add-to-cart**. |
| I4 | **Halls booking** — `حجز القاعات` | Hero photo, a `قاعات الفندق` grid of named halls (`قاعة الملكة`, `قاعة الجوهرة`), then a separate `العروض` grid of the same halls on offer. |
| I10 | **Maintenance** — `الصيانة` | Hero, then a `الخدمات` list: `فني كهرباء` / `فني تكيف` / `سباك`, each with a price column (`مجاني` = free) and a **multi-select toggle** (+ / ✓). A reviews block (`لا يوجد مراجعات`) and a single `حجز` (book) CTA at the bottom. |
| I11 | **Offers** — `العروض` | Search + filter, category pills (`الكل` · `الفنادق` · `المطاعم` · `القاعات` · `التجميل` · `أخري`), a promo banner (60% OFF), then `عروض اليوم`: each row = photo, discount headline, venue, **rating**, **opening hours**, and a `حجز` button. |
| I13 | **Outlet profile — info tab** | Salon `تجميل وحلاقة`. Hero + **four tabs**: `معلومات` · `الخدمات` · `معرض الاعمال` · `المراجعات`. Info tab shows a description with *read more*, **`ساعات العمل`** per day-range with a green open dot, and **`العنوان`** + `أحصل على الإتجاهات`. |
| I14 | **Outlet profile — services tab** | A `الخدمات` / `العروض` segmented switch, a **priced service list with + selectors**, then **`تاريخ الوصول` (date) + `ميعاد الوصول` (time)**, and a **live running total** button (`0.0`). I.e. appointment booking with a cart. |
| I12 | **Cars — request form** | `طريق السيارة`: radio **`ذهاب` / `إياب` / `ذهاب وإياب`** (one-way / return / round trip), a destination dropdown, `تاريخ الوصول` + `وقت الوصول`, **`عدد الأشخاص`** stepper, and free-text `ملاحظات`. |
| I15 | **Cars — fleet list** | Vehicle cards: name (`اتوبيس رقم 1`, `مرسيدس`), **`مقاعد` seat capacity**, price, rating, favourite. |
| I7 | **Home** | Three entry tiles: `الفنادق` · `خدمات الفنادق` · `العروض والخصومات`. Offers carousel with rating badge + `10% OFF` badge + price/night. `فنادق تم زيارتها` (recently visited). |
| I5 | **Login** | Username + password only. |
| I6 | **Back-office — restaurant dashboard** | `عدد الاوردرات` · `الملغاه` · **`اجمالي قيمه الاوردرات`** · `مبيعات الاسبوع` · `الاصناف الاكثر مبيعا` · `قائمه العملاء الاكثر شراءا` · `تعليقات وشكاوي العملاء` · `رسائل العملاء`. Per-outlet growth bars (today / month). |
| I8 | **Back-office — all departments** | One stat card per department (housekeeping, spa, cars, restaurants, maintenance, bellboy/`pollman`, rooms, halls), each with: `عدد الاوردرات` · `تحت التجهيز` · `قيد التسليم` · `الاوردرات المتأخرة` · `تم تسليمها` · **`اجمالي قيمه الاوردرات`**. |
| I9 | **Back-office — catalog CRUD** | Sidebar: `إدارة المطاعم` · `إدارة الغرف` · `إدارة القاعات` · `إدارة الاسبا` · `إدارة السيارات` · `إدارة الهاوس كيبينج`, each with **create service group / create service / offers on services**, and an active/inactive (`فعال` / `غير فعال`) flag per row. |

---

## 3. What Escalé has today

```
ServiceCategory (flat, code + icon)
  └── HotelService  (one price, one availability window, one department)
        └── ServiceRequest (free-text description, one service, no money)
```

`ServiceRequest` carries no amount, no quantity, no scheduled time, and no line items.
`Review` is hotel/booking-level. There is no offer/promotion model, no favourites,
no bookable inventory, and no notion of an outlet.

---

## 4. The gaps

Ordered by structural impact.

### G1 — In-hotel outlet (blocking)
A service belongs to an **outlet** — a restaurant, a spa, a salon, a hall complex, a transport
desk — which sits inside the hotel and is either **hotel-operated** or an independent
**concession**. An outlet has its own name, logo, hero, description, per-day opening hours,
location within the property, gallery, and reviews (I3, I13). Nothing in the schema represents
this. Everything else in this list hangs off it.

### G2 — Cart, order, and money (blocking)
The legacy app orders *multiple items in one order with a total* (I3, I14) and the back-office
reports **total order value** per department (I6, I8). Escalé's `ServiceRequest` is a single
free-text ask with no amount. Needs `ServiceOrder` + `ServiceOrderLine` with quantities, unit
price, line total, order total, currency, and a payment link (room charge or pay-in-app).

### G3 — Catalog items with real attributes
Menu/service items need: photo, description, **prep/duration time**, price, section grouping
(`بيتزا` / `مشويات`), availability, and a *free* flag (`مجاني`, I10). Today `HotelService` is the
only level and has no photo, no duration, no section.

### G4 — Two-level category tree
`الاشراف الداخلي` → laundry / cleaning / amenities / linens (I2). `ServiceCategory` is flat.

### G5 — Appointment scheduling
Spa, salon, and maintenance orders carry an **arrival date + time** (I12, I14) — not "as soon as
possible". `ServiceRequest` has no scheduled slot.

### G6 — Bookable assets with capacity
Halls (`قاعة الملكة`) and vehicles (`مرسيدس · 4 مقاعد`) are **inventory units** booked for a time
window, not services ordered by quantity (I4, I15). They need their own model with capacity and an
availability calendar, so two guests can't book the same hall at the same hour.

### G7 — Transport-specific request fields
Trip direction (one-way / return / round trip), destination, passenger count, notes (I12).

### G8 — Offers & discounts
Percentage or fixed discount attached to an item or an outlet, with validity dates and a category
filter, surfaced in a dedicated offers feed (I11, I4's `العروض` block, I14's offers tab). Screen 19
exists in the design but has no model behind it.

### G9 — Favourites
Heart on menu items and vehicles (I3, I15).

### G10 — My Orders
A guest-side order list (`طلباتي`, I2's nav). Escalé has screen 23 for **one** request but no list.

### G11 — Richer order lifecycle
`تحت التجهيز` (preparing) → `قيد التسليم` (out for delivery) → `تم تسليمها` (delivered), plus
`الاوردرات المتأخرة` (overdue vs. SLA) and `الملغاه` (cancelled) — I6, I8. Escalé's
`SUBMITTED → ASSIGNED → IN_PROGRESS → COMPLETED` has no delivery leg. **Decision: keep the existing
state machine and add a `preparing`/`delivering` sub-stage only where the department needs it,
rather than forking the enum per department.**

### G12 — Outlet-level reviews & gallery
Reviews and a work portfolio (`معرض الاعمال`) attached to an outlet, not to the hotel (I13).

### G13 — Admin catalog management
Screens to create/edit categories, outlets, items, and offers, with an active/inactive flag (I9).
Escalé has no admin catalog screens designed at all.

### G14 — Department order dashboards
Per-department counters including **order value** and **late orders** (I8), plus an outlet-level
sales view — best sellers, top customers, complaints (I6). Screen 07 covers part of this for
requests but not for money or per-department breakdown.

---

## 5. Scope call

| Gap | MVP (Project 1) | Later |
|-----|-----------------|-------|
| G1 Outlet (hotel-operated + concession) | ✅ | — |
| G2 Cart / order / total / payment | ✅ | — |
| G3 Catalog items (photo, duration, section, free flag) | ✅ | — |
| G4 Two-level categories | ✅ | — |
| G5 Appointment date + time | ✅ | — |
| G6 Bookable assets (halls, vehicles) | ✅ | Recurring/seasonal calendars |
| G7 Transport request fields | ✅ | Live driver tracking |
| G8 Offers & discounts | ✅ basic % / fixed | Rules engine, targeting |
| G9 Favourites | ✅ | — |
| G10 My Orders list | ✅ | — |
| G11 Delivery sub-stage + overdue | ✅ | Per-department custom flows |
| G12 Outlet reviews + gallery | ✅ reviews · gallery basic | Moderation, replies |
| G13 Admin catalog CRUD | ✅ | Bulk import, versioning |
| G14 Department dashboards with order value | ✅ | Sales analytics, top customers |

**Not carried over from the legacy app:** username/password login (I5) — Escalé uses phone/OTP +
Apple/Google (screens 16–17); and the legacy home's hotel-first framing, which Escalé already
improves on.

---

## 6. Design impact

New screens:

| # | Screen | Covers |
|---|--------|--------|
| 24 | Restaurant Menu & Cart | G2, G3, G8, G9 |
| 25 | Halls & Bookable Assets | G6, G8 |
| 26 | Transport / Cars | G6, G7 |
| 27 | Outlet Profile (tabs) | G1, G5, G12 |
| 28 | My Orders | G10, G11 |

Updates to existing screens: **11–12** (two-level categories), **19** (offer categories, hours,
rating, discount badge), **07** (per-department stats incl. order value + late orders),
**18** (outlets rather than flat services).

---

## 7. Open questions for the client

1. **Concession settlement.** For an independent business inside the hotel (the salon case): does
   Escalé take payment and settle with the outlet, or does the outlet bill the guest directly and
   Escalé only routes the order? This decides whether we need commission/payout models now.
2. **Room charge vs. pay-in-app.** Is "charge to my room" available for all outlets, or only
   hotel-operated ones?
3. **Delivery leg.** Does room service need a real `out for delivery` state visible to the guest,
   or is "in progress" enough for MVP?
4. **Halls.** Is a hall booking an instant confirmation or a request the hotel must approve
   (likely the latter, given it is priced per event)?
5. **Cancellation windows** per service type — spa appointments and hall bookings need them; a
   towel request does not.
