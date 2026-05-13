# Code Explanation

## Project Overview

**Aura med** (Аура мед / Долоон Чакра) is a Mongolian-language wellness/spirituality website served from a static GitHub Pages site at **auramed.com**. The site introduces visitors to the seven chakras through a designed long-form landing page, lets them book a consultation through a calendar + time-slot picker, and gives the operator a live admin dashboard of incoming bookings. There is no build step, framework, or backend code in the repo — the four pages are hand-written HTML/CSS/JS, and persistence + realtime updates are handled by Supabase (loaded directly from `esm.sh` as an ES module).

## File-by-File Breakdown

### `CNAME`
**Purpose:** Single-line file that tells GitHub Pages which custom domain serves this repo.

**Notes:** Contents: `auramed.com`. No code; required by GitHub Pages.

---

### `zurag1.jpg`
**Purpose:** The mandala/portrait photograph used inside the circular hero figure on `index.html` and `introduction.html`.

**Notes:** Binary asset (~230 KB). Referenced from the two landing pages as `<img src="zurag1.jpg">` inside the `.mandala-img` element.

---

### `index.html`
**Purpose:** The main public landing page — introduces "aura" and the seven chakras, and links out to the booking flow.

#### Page sections (in source order)
| Section | Description |
|---|---|
| `<nav>` | Fixed top bar with the "✦ Аура мед" wordmark and three links: `#aura`, `#chakras`, and `booking.html`. |
| `.hero` | Three-column grid (left text / mandala figure / right text) that collapses to a single column under 900px. The right column is hidden on mobile. |
| `.figure-wrap` | The animated mandala: three blurred `.glow-ring` layers, one slow-spinning dashed `.spin-ring`, an `#particles` container that JS fills with 18 floating dots, and the circular `zurag1.jpg`. |
| `.legend` | Color-dot key listing the seven chakra Sanskrit names. Each item passes its color via an inline `--lc` custom property. |
| `.intro #aura` | "01 — Аура" prose block describing the aura concept. |
| `.section-header #chakras` | "02 — Долоон Чакра" header introducing the cards below. |
| `.chakras` | Seven `<article class="card">` blocks (crown, third, throat, heart, solar, sacral, root). Each card has an orb, name + Sanskrit, description, and a meta column with element / color / mantra badge. |
| `<footer>` | Closing ornament, Mongolian blessing blockquote, and "© 2026" small text. |

#### Key CSS tokens / patterns
| Name | Description |
|---|---|
| `:root` custom properties | Defines the dark cream-on-near-black palette (`--bg-0`, `--ink`, `--gold`) plus one color per chakra (`--c-crown` … `--c-root`). |
| `body::before` grain overlay | A fixed, full-screen SVG `feTurbulence` noise PNG-data URI laid over the page at 5.5% opacity with `mix-blend-mode: overlay` for a subtle film-grain texture. |
| `.card.{name}` modifier classes | Each card sets `--color: var(--c-{chakra})` so the orb gradient, hover glow line, mantra badge border, and meta values all derive from one variable. |
| `color-mix(in srgb, var(--color) X%, transparent)` | Used throughout for tinted shadows/borders without writing one-off rgba values per chakra. |
| `@media(max-width:640px)` block | Re-lays out the cards as a tappable accordion: tiny orb in column 1, title + chevron in column 2, body and meta collapse to `max-height: 0` until the card has the `.open` class. |
| `@media(prefers-reduced-motion:reduce)` | Globally collapses animations and transitions to ~0ms. |
| Keyframes | `breathe` (scale pulse for glow rings + image), `pulse` (legend dots), `spin` (spin-ring + orb halo), `fadeUp` / `fadeIn` (load-in), `float` (particles drift up). |

#### Inline `<script>` (~50 lines at the bottom)
| Name | Description |
|---|---|
| Particle generator | Creates 18 `<div class="particle">` children inside `#particles`, each with random position, size, color (picked from a 7-color array), drift `--dx`, animation delay, and duration. |
| `IntersectionObserver` (`io`) | Watches every `.reveal` element; when ≥10% visible, adds `.in` (which fades it up from `translateY(28px)`) and unobserves it. |
| `initAccordion()` | On viewports ≤640px, attaches a click handler to each `.card` that closes any other open card and toggles `.open` on the clicked one. |
| `resize` listener | If the viewport crosses the 640px breakpoint, closes all open cards and re-runs `initAccordion()` so desktop ↔ mobile transitions don't leave stale state. |

**Notes:** This file embeds **all** CSS and JS — no external stylesheets, no bundler. The only network dependencies are Google Fonts (Cormorant Garamond, Cormorant Unicase, Outfit) and the local `zurag1.jpg`. Page is in Mongolian (`<html lang="mn">`).

---

### `introduction.html`
**Purpose:** An earlier or alternate version of the landing page, branded "Долоон Чакра" instead of "Аура мед" and without the booking link.

**Notes:**
- Almost line-for-line identical to `index.html` — same hero, legend, intro, section header, seven chakra cards, footer, particle script, scroll reveal, and accordion logic.
- **Differences vs `index.html`:**
  - Nav wordmark says `✦ Долоон Чакра` (vs `✦ Аура мед`).
  - The third nav link points to `#footer` ("03 · Дадлага" — "Practice") rather than `booking.html` ("03 · Цаг авах" — "Book a time").
  - CSS is written in compact one-line-per-rule form rather than multi-line.
- Functionally a candidate for deletion or redirect — it is not linked from any other page in the repo.

---

### `booking.html`
**Purpose:** Three-step booking flow — pick a date on a custom calendar, pick a time slot, fill name + phone, write to Supabase.

#### Page sections
| Section | Description |
|---|---|
| `<nav>` | Same top bar as `index.html`, with "Цаг авах" marked `active`. |
| `.booking-header` | Eyebrow + h1 "Уулзалтын *цаг* захиалах" + lede. |
| `.steps` | Three pill-style step indicators (`Өдөр` → `Цаг` → `Мэдээлэл`) that JS toggles between `active` / `done`. |
| `.booking-grid` | Two-column panel layout: calendar on the left, time-slot list on the right. Collapses to one column under 740px. |
| `#form-section` | Hidden until a slot is picked. Shows the chosen date+time as a summary, then a name + phone form with a submit button. |
| `#success-section` | Hidden until submit succeeds. Shows a confirmation glyph, the booked date+time, and a "new booking" button that reloads the page. |
| `.mob-nav` | Fixed bottom navigation visible only ≤680px (Аура / Чакрууд / Цаг авах). Reserves space at the bottom of `body` via `padding-bottom: calc(72px + env(safe-area-inset-bottom))`. |

#### Inline `<script type="module">`
| Name | Description |
|---|---|
| `createClient(url, anonKey)` | Initializes the Supabase client. The project URL and **publishable anon key** are inlined as plain strings — see Notes below. |
| `SLOTS` | `['09:00','11:00','13:00','15:00','17:00']` — the five fixed appointment times offered every working day. |
| `MN_MONTH` | Array of 12 ordinal Mongolian month names (`Нэгдүгээр`, `Хоёрдугаар`, …). |
| `viewDate`, `selectedDate`, `selectedTime` | Module-level state. `viewDate` controls which month the calendar renders; the other two track user selection. |
| `pad(n)` | Zero-pads a single number to two digits (used for ISO date formatting). |
| `toISO(d)` | Returns `YYYY-MM-DD` in local time (deliberately not `Date.toISOString()`, which would shift by the UTC offset). |
| `labelDate(d)` | Returns `"<MN month> сарын <day>"` for human display. |
| `renderCalendar()` | Builds the month grid: writes the month label, computes `firstDow` (Monday = 0 — uses `(getDay() + 6) % 7`), and the number of days. Pads the start with empty cells, then renders one button per day. Marks today, the currently selected day, and disables past days + Sat/Sun. |
| `onDateSelect(date)` | Stores the date, re-renders the calendar (so highlights update), advances the step indicator to 2, queries Supabase for any existing `bookings` rows on that date, then renders one slot button per `SLOTS` entry — disabling and labeling "Захиалагдсан" any slot that's already booked. |
| `onTimeSelect(time, btn)` | Marks the chosen slot button `.active`, advances to step 3, fills the summary text, and reveals the form (scrolling it into view). |
| Month-nav handlers | `‹` / `›` buttons mutate `viewDate.setMonth(±1)` and re-render. |
| Form `submit` handler | Disables the button, calls `supabase.from('bookings').insert({ name, phone, booking_date, booking_time })`. On error, re-enables and `alert()`s the message. On success, hides the calendar/form/steps and shows the success panel. |
| `setStep(n)` | Updates the three `.step` elements: ones below `n` get `.done`, the one at `n` gets `.active`. |

**Notes:**
- **Hardcoded Supabase credentials.** The Supabase project URL `qvsveyaarinogkquyisb.supabase.co` and the anon JWT (issued 2026-04-30, expires 2036-04-25) are checked into the HTML. This is the *anon* publishable key (intended to be public when paired with row-level-security policies on the `bookings` table) — but anyone with the key can `INSERT` rows from anywhere unless RLS is enforced. This same key is duplicated in `admin.html`.
- The DB row shape inferred from the code: `bookings { id, name TEXT, phone TEXT, booking_date DATE (YYYY-MM-DD), booking_time TIME (HH:MM), created_at TIMESTAMPTZ }`.
- The "phone" `<input type="tel">` is `required` but is not pattern-validated — the placeholder suggests `+976 xxxxxxxx` but any string is accepted.

---

### `admin.html`
**Purpose:** Operator dashboard — lists all bookings with stats, filters, and live updates when a new booking is submitted from `booking.html`.

#### Page sections
| Section | Description |
|---|---|
| `<nav>` | "✦ Аура мед" wordmark, "Admin" pill, and a `#live-indicator` on the right that starts as "Холбогдож байна…" and switches to "Realtime" when Supabase confirms the subscription. |
| `.stats` | Three cards: today's bookings, upcoming in next 7 days, all-time total. |
| `.filters` | Pill buttons (`Бүгд` / `Өнөөдөр` / `Удахгүй` / `Өнгөрсөн`) plus a date input that filters to a single day. |
| `.table-wrap` | A `<table>` with columns: date, time, name, phone, status badge, and registered-at timestamp. The "registered" column is hidden under 680px. |
| Toast | A right-bottom notification that appears for 4 seconds whenever a new INSERT comes in over the realtime channel. |

#### Inline `<script type="module">`
| Name | Description |
|---|---|
| `createClient(...)` | Same Supabase URL and anon JWT as `booking.html`. |
| `MN_MONTH` | Short form (`'1-р сар'` … `'12-р сар'`) — different from `booking.html`'s ordinal form. |
| `allBookings`, `activeFilter` | Module state — full list from the DB, and the currently active filter id. |
| `todayISO()` | Returns local-date `YYYY-MM-DD`. |
| `formatDisplayDate(iso)` | `'2026-05-01'` → `'5-р сар 1'`. |
| `formatRegistered(iso)` | Formats a timestamp as `YYYY/MM/DD HH:MM` using local time. |
| `getStatus(bookingDate)` | Returns `'today'` / `'upcoming'` / `'past'` by string-comparing ISO dates. |
| `statusLabel(s)` | Maps those keys to Mongolian labels. |
| `applyFilter(bookings)` | If the date input has a value, filter to that exact day; otherwise dispatch on `activeFilter` (today / upcoming-within-7-days / past / all). |
| `updateStats(bookings)` | Recomputes the three stat numbers and writes today's M-D into the subtitle of the "Өнөөдрийн захиалга" card. |
| `renderTable(bookings, newId?)` | Filters, sorts by date+time ascending, then writes one `<tr>` per row. If `newId` is passed, that row gets `class="new-row"` which triggers the `highlight` keyframe (gold flash → transparent). Empty result renders the empty-state cell. |
| `showToast(name, date, time)` | Removes any existing toast, builds a new one, appends to body, then after 4s swaps the in-animation for `slideDown` and removes the node. |
| `loadBookings()` | One-shot fetch of all rows ordered by date then time, populates `allBookings`, then triggers stats + table render. |
| Realtime subscription | `supabase.channel('bookings-admin').on('postgres_changes', { event: 'INSERT', table: 'bookings' }, …)` — on every INSERT, prepends the new row to `allBookings`, re-renders, and shows the toast. The `.subscribe(status => …)` callback flips the live indicator to "Realtime" once `status === 'SUBSCRIBED'`. |
| Filter button handlers | Click handler updates `activeFilter`, clears the date input, and re-renders. |
| Date-input handler | Clears the active filter pill, sets `activeFilter = 'custom'`, re-renders. |

**Notes:**
- Same hardcoded anon key as `booking.html` — see security note in that file's section.
- The page has no auth gate. Anyone who finds the URL `/admin.html` can read every booking (name + phone). If this is meant to be operator-only, it should be moved behind Supabase auth or at minimum a Pages-level access restriction.
- Realtime requires that the `bookings` table has been added to the `supabase_realtime` publication on the Supabase side.

---

## Key Concepts & Architecture

**Stack.** Pure static HTML — three user-facing pages (`index.html`, `booking.html`, `admin.html`) plus a leftover earlier draft (`introduction.html`), one image, and a `CNAME`. Deployed via GitHub Pages to `auramed.com`. No bundler, no package.json, no server. The only runtime dependency is Supabase, loaded as an ES module from `https://esm.sh/@supabase/supabase-js@2`.

**Design system, copy-pasted not shared.** Every page redefines the same `:root` color tokens (`--bg-0`, `--ink`, `--gold`, …), the same fixed nav with backdrop blur, and the same SVG-noise grain overlay (`body::before`). Refactoring this into a shared stylesheet would remove a meaningful amount of duplication, but the project deliberately ships zero build step, so the duplication is the price of that simplicity.

**Per-chakra theming via CSS custom properties.** On `index.html`, each card's color is set once with a modifier class (`.card.crown { --color: var(--c-crown) }`), and every visual detail — orb gradient, hover glow line, meta-value text color, mantra-badge border + tinted background — derives from `var(--color)` through `color-mix()`. Adding an eighth chakra would mean adding one token and one modifier rule.

**Mobile-first transformation, not duplication.** The card grid on `index.html` is one HTML structure that the `@media(max-width:640px)` block re-lays-out into a tap-to-expand accordion. The JavaScript `initAccordion()` only attaches handlers under 640px, and the resize listener cleanly resets state if you cross the breakpoint mid-session.

**Booking data flow.** `booking.html` reads the calendar's existing bookings for a chosen date (`SELECT booking_time FROM bookings WHERE booking_date = ?`) so it can grey out taken slots, then `INSERT`s a new row on submit. `admin.html` opens a Supabase realtime channel on the `bookings` table and prepends + highlights any new INSERT in real time. The two pages share state only through the database — no client-to-client messaging.

**Security posture.** The anon JWT is checked into source for both the booking and admin pages. This is conventional for Supabase publishable keys, **but it only works safely if row-level-security policies on `bookings` allow the right things and forbid the rest** (e.g. allow `INSERT` from anon, forbid `SELECT` to anon — and require an authenticated role for `admin.html`). As shipped, `admin.html` is reachable by anyone who knows the URL and would expose customer names + phone numbers. This is the most important thing to verify before production use.
