# Slither — Build Brief

Brief for Claude Code. Builds on what is in `index.html` today: plan a snake of bars through Queenstown, send it to the crew on WhatsApp, track the night, lock the snake on the board; Slither Deals; Claim your bar for venues. Nothing here replaces that loop. Each item hooks into it.

Goal: nail Queenstown so visitors scan it, finish a night on it, and carry it home. Two audiences: the tourist on the night, and the bar owner who decides whether a Slither QR sits on the table.

Priority key: **Now** = pre-launch · **Next** = first 90 days · **Later** = after venues and scans are proven.
Effort key: S = days · M = 1–3 weeks · L = month-plus.

Build order and dependencies are at the bottom. Start there.

---

## Product

### 1. Scan-to-value in 5 seconds — M — Now
- Each bar gets a QR that opens `/v/:venueSlug?src=:placement`.
- First render, no signup: venue card, tonight's Slither Deal, and one button: "Add to tonight's snake" (or "Start a snake here" if none exists).
- If the scanner was sent a Slither, the QR marks that stop as reached and advances "Next up".
- Account / install prompt only when the user locks a snake or wants it on the board, never before.
- Log every scan as an event: `venue_id, placement_code, locale, ts, session_id, user_id?`. This one table feeds the owner dashboard, the recap, placement tracking and the board. Get the schema right first.

### 2. Offline-first — M — Now
- Already a one-page PWA shell. Add a service worker that caches the venue list, the hand-drawn CBD map assets, the current snake and locked snakes.
- Writes (stop reached, lock, rating) queue locally and sync on reconnect.
- A snake must be fully usable with no signal between bars.

### 3. Multilingual — S (UI) / M (venue copy) — Now (UI) / Next (copy)
- i18n layer with JSON string files. Auto-select from phone locale, manual override.
- Languages: en, zh-CN, ja, ko, de, es, pt.
- Venue descriptions and deal copy stored per-language with English fallback.
- Ship English-only strings in step 1; translate once the UI is stable.

### 4. City-as-config — M — Next (before city 2, not before city 1)
- `cities` table: `name, slug, bbox, map_asset, languages[], season_calendar, currency, timezone`.
- Every venue, deal, placement and snake carries a `city_id` from day one, even with one city.
- Adding a city = new row + a map + content. No code change.

---

## Growth loop (take it home)

### 5. Shareable night recap — M — Now
- "Your nights, remembered" exists. Turn a locked snake into a one-tap share card: the snake drawn on the map, the stops, the score, the snake name.
- Generated server-side as an image + a public link (`/s/:shareId`). Link lands on the snake + an install prompt.
- Share targets: WhatsApp (already the crew channel), Instagram, WeChat, LINE.

### 6. Queenstown passport — M — Now
- Stops already behave like stamps. Formalise it: 10 distinct partner bars reached (via QR scan, geofenced to 150 m) across any number of nights = passport complete.
- Progress ring on the board tab. 10/10 unlocks a voucher redeemed at one partner venue for a physical reward.
- Voucher record: `user_id, redeem_at_venue_id, issued_at, redeemed_at`.

### 7. Arrival QR placements — S — Next
- Each physical QR has its own `placement_code`: ZQN baggage claim, Skyline queue, Fergburger queue, hostel receptions.
- Landing for a non-venue placement = "Suggested Slithers" with tonight's deals.
- Reuses scan events from #1. Dashboard view: scans by placement by day.

### 13. "Left Queenstown?" hometown request — S — Now
- On app open, compare device location (coarse, with permission; fall back to timezone + locale if denied) against the active city's `bbox`. Outside it for 24h+ = "left" state.
- Show a one-time card: **"Left Queenstown? Request Slither in your hometown."** One tap sends `city_name` (autocompleted from reverse geocode), `country`, `locale`, `user_id?`, `snakes_locked`. No form, no typing.
- Store in a `city_requests` table. Count per city = the roadmap for step 8 (City-as-config) and the investor story: a map of where tourists carried Slither.
- Confirmation copy: "You're #N asking for Slither in Melbourne. We'll message you when it lands." Capture email/phone only after the tap, optional.
- Never show while inside the bbox; never show more than once per city per user.

### 14. Night carousel builder — M — Next
- From a locked snake: pick 3–10 photos from the night (camera roll or the ones added at stops). Output is a square carousel (1080×1080) for Instagram, plus a 9:16 variant for stories.
- Each slide gets the Slither snake mascot in the bottom-right corner, a different pose per slide, cycling through a pose set (e.g. peeking, waving, holding a drink, asleep, sunglasses, pointing left to "swipe", curled in a loop). Pose chosen by slide index so the crew sees a different snake on every swipe; last slide = the full snake route drawn on the map (reuses #5's card).
- Mascot = SVG sprite set in the repo, 8 poses minimum, one accent colour, drawn client-side onto a canvas so it works offline. Render server-side only for the shareable link.
- Slide 1 overlay: snake name + date + "Slithered in Queenstown". Every other slide: stop name + time reached, bottom-left, small.
- Share targets as #5. Each export logs a `carousel_exported` event; the carousel is the overseas awareness unit, so count it.

### 8. Seasonal mode flip — S — Later
- Season calendar on the city record. Suggested Slithers, featured deals and home copy switch on a date.
- Winter: après, early starts, ski-town pubs. Summer: lakefront, late sun, longer snakes.

---

## Venue side

### 9. Free owner dashboard — M — Now
- "Claim your bar" exists. Behind it: a phone-friendly page per venue with magic-link login (email, no password).
- Metrics: scans, snakes that included this bar, position in the snake (first stop / mid / last), repeat visitors, busiest hour, visitor origin (from phone locale, not GPS).
- Nightly roll-up job from the scan-event table.

### 10. Quiet-hour deal lever — S — Next
- Slither Deals exist. Add an owner toggle with a time window (default 3–6 pm) and a redemption cap.
- Deal shows on the venue card and in Suggested Slithers only during the window.

### 11. Private review intercept — S — Next
- Prompt the morning after a locked snake, per stop. 1–3 stars → private note to the venue dashboard. 4–5 stars → deep link to that bar's Google review URL.

### 12. Stay out of POS / bookings — principle
- No ordering, no payments, no table bookings. Deep-link out when needed. Slither is the layer between the crew and the bar.

---

## Go-to-market (not code, but shapes the data model)

- **30 curated launch bars** hand-onboarded; venue records need `tier` (launch partner / standard) and `onboarded_by`.
- **Hostels** get a dashboard and count as a passport stop; venue `type` must include `hostel`.
- **Partner physical reward** for 10/10 (see #6).
- **Source-market content** is a marketing workstream; the share card and the reward are the hero visuals.

---

## Build order

1. Scan-to-value: QR routes, scan-event table, "Add to tonight's snake", service worker, i18n layer (English only)
2. Passport: distinct-bar count, geofence, progress ring, 10/10 voucher
3. Owner dashboard behind Claim your bar: magic-link login, nightly roll-up
4. Night recap share card + public link
5. "Left Queenstown?" hometown request + `city_requests` table
6. Translated UI strings (6 languages)
7. Night carousel builder with snake mascot poses (needs the mascot SVG set first)
8. Arrival QR placements with per-placement tracking
9. Quiet-hour deals + review intercept
10. City-as-config refactor (before city 2; `city_requests` tells you which city)
11. Seasonal flip

**Dependencies:** 2, 3, 4 and 8 all read the scan-event table from step 1. Step 7 reuses step 4's map card as its last slide. Step 5 needs the city `bbox`, so add that column in step 1 even before the full cities table exists. Every venue, deal and placement carries `city_id` from the start so step 10 is a refactor, not a rewrite.
