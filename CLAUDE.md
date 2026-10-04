# Vacationer — repo conventions

Visual, printable family trip itineraries published as static GitHub Pages.
Deploys from the **`main` branch / root folder**, served at
**https://marshalltilley.github.io/vacationer**.

There is no build step, framework, or package manager. Every page is a single
hand-authored `index.html`. Editing the file *is* the deploy — push to `main`
and Pages republishes.

---

## Layout

```
/
├── index.html              ← landing page (the trip index)
├── CLAUDE.md               ← this file
├── amsterdam-2026/         ← one trip = one folder
│   ├── fact-sheet.md       ← source of truth for the trip
│   └── index.html          ← the rendered itinerary
├── hawaii-trip/
│   ├── fact-sheet.md
│   └── index.html
└── italy-2027/
    ├── fact-sheet.md
    └── index.html
```

**One trip = one folder.** Each folder holds exactly two files:
a markdown `fact-sheet.md` and an `index.html`. The root `index.html` is the
landing page that links to each trip card.

## The fact sheet is the source of truth — always

`fact-sheet.md` is canonical. **Build the HTML *from* the fact sheet, never from
memory.** When a date, time, booking, or count changes:

1. **Edit `fact-sheet.md` first.**
2. *Then* regenerate or hand-edit `index.html` to match.
3. Keep the fact sheet's `HTML version stamp:` and the footer version in
   lockstep (bumped once per merge — see Versioning).

If the HTML and the fact sheet ever disagree, the fact sheet wins — reconcile by
fixing the HTML, not by quietly editing the fact sheet to match stale markup.

## Building a trip page — borrow, don't clone

**Every trip is different.** There is no single master template. Start from the
existing trip most like the new one, then borrow the most helpful pieces from
the others. Pick modules for what *this* trip needs; leave out the rest.

| If the new trip is… | Start from | Why |
|---|---|---|
| A couple, ~1 week, one or two hotels | `amsterdam-2026` | Day cards, compact food list by type, one day-trip tab |
| A big group, weeks long, people arriving/leaving at different times | `hawaii-trip` | Presence map, phases by group make-up, month calendar, scope badges |
| A couple, ~2 weeks, several cities | `italy-2027` | Day cards + route ribbon, phases by city, Do/Eat by city, booking timeline |

### Every trip has

1. **Hero** — eyebrow (`Vacationer · No. NN` + season), big display title,
   italic dek, then a 4-up `hero-meta` grid: **Window · Base · Travelers · Tone**
   (the second label can adapt, e.g. Amsterdam's "Hotels").
2. **Sticky tab nav** (`.tabs`, `position: sticky; top: 0`) with a **Print
   button** on the right. Core tabs: **Overview · Eat & do · Locked events ·
   Logistics**. Optional: a **Bookings** tab split out of Logistics when the
   booking list is long (Italy), and **one** sub-trip tab (see Sub-trips).
3. **Overview** that opens with an at-a-glance view of the trip (day cards,
   presence map, or both) and a **booking pulse** if the trip has bookings.
4. **Footer** with the version stamp, then a hidden `.print-stamp` div + the
   `<script>`.

### Module library — pick what fits

| Module | Shows | Best for | Used in |
|---|---|---|---|
| **Trip at a glance** (`.day-grid` / `.day-card`) | One card per day: anchor, notes, where we sleep | Trips up to ~2 weeks | Amsterdam, Italy |
| **Presence map** (`.timeline`) + headcount strip | One bar per party, arrival → departure; people per day | Groups with staggered arrivals | Hawaii |
| **Route ribbon** + transfers | Nights per base as one bar; how we move between bases | Multi-city trips | Italy |
| **Phases** (`.phases`) | Trip split into named windows — by group make-up *or* by city | Long or multi-city trips | Hawaii (group), Italy (city) |
| **Master calendar** (`.cal-wrap`) | Month grid colored by phase, locked events marked | Month-long trips | Hawaii |
| **Booking pulse** (`.pulse`) | Confirmed / Still to book / Flight legs counts | Any trip with bookings | All |
| **Locked events** | Booked anchors, chronological, with type tags | All trips | All |
| **Eat & do — by type** | Category cards: coffee, restaurants, bites, bars, shopping | Single-city trips | Amsterdam |
| **Eat & do — by region** | Region cards with tiered restaurant lists + activity cards | Spread-out bases | Hawaii |
| **Eat & do — Do / Eat by city** | Two headed parts, one-line description per place | Multi-city trips | Italy |
| **When to book** | Booking triggers sorted by date | Trips booked months ahead | Italy |
| **Hotels / flights cards** | Stays and legs as cards | All trips | Amsterdam, Hawaii, Italy |
| **Packing & style** | Weather, style, shoes, gear | All trips | All |
| **Sub-trip panel** | Day trip with phases + alternate; expedition day cards; weather swap decision box | Trips with day trips or side legs | Amsterdam, Hawaii, Italy |

Copy a module's markup and CSS from the trip that has it, then swap in the
destination palette and fonts.

## Tabs, deep links, and print — the JS contract

A small IIFE at the bottom of each page wires three things:

- **Tab switching:** clicking a `.tab` sets `.active` and toggles
  `panel.hidden` so only one `.tab-panel` shows.
- **Deep links via URL hash:** on load, `#<data-tab>` opens that tab. Standard
  hashes: **`#eatdo`, `#events`, `#logistics`**, plus whatever the trip adds
  (`#bookings`, and the sub-trip tab: Hawaii `#bigisland`, Amsterdam `#daytrip`,
  Italy `#subtrips`). The hash must equal a tab's `data-tab` value — keep them in
  sync. Use the read+write pattern (write the hash back on tab click via
  `history.replaceState`) — Amsterdam and Italy do; Hawaii currently only reads.
- **Print:** the Print button (and `beforeprint`) stamps the hidden
  `.print-stamp` and calls `window.print()`. Print CSS hides the hero/tabs/footer
  and prints **only the active tab**, **landscape, one section per page**
  (`@page { size: letter landscape }` + `.tab-panel > .section { break-after: page }`).
  The **print stamp reads the version from `.footer-mark` automatically** — do
  not hardcode the version string in the JS (so the footer is the single source
  of the version number). *Known gap: Amsterdam's stamp still hardcodes `v4`.*

## Scope badges — flag anything that isn't whole-group

When an item applies to only part of the group, tag it with a color-coded
`.scope-badge` and explain the badges in a `.scope-legend` at the top of the
section. Hawaii's set: `bi` (Big Island sub-trip / M&A only), `golf` (golfers),
`pod` (a subset of people), `anna` (ballet/kid-specific). Never imply a
sub-trip, golf, or kid-specific item applies to everyone.

On a couple's trip everything is whole-group, so badges aren't needed for
*who*. Italy reuses them for constraints instead (`trip` sub-trip day, `swap`
weather swap, `nonref` non-refundable, `opt` optional) — use that pattern when
it helps.

## Logistics — keep Confirmed and Still-to-book split

Wherever bookings live (Logistics, or a Bookings tab), **"Confirmed" and
"Still to book" are two separate lists under their own headings. Never mix
booked and unbooked items in one list.** Preferred markup: two `.res-grid`
blocks under `.res-subhead` (Hawaii, Italy); confirmed items use
`.res-item.confirmed` with a checked box; to-book items carry an urgency chip
(`high` / `med` / `low`). Amsterdam's older `.bookings` lists follow the same
split.

The **booking-pulse counts on the Overview tab must equal those lists** —
if you add/confirm a booking, update the pulse number *and* the pulse sub-text
*and* move the item between the two Logistics blocks together.

## Sub-trips — one tab, never per-person pages

A sub-trip (e.g. Big Island) gets **its own single tab**. If a trip has multiple
sub-trips, group them all under one **"Sub-trips"** tab — do not split into one
tab per person or per sub-trip. There are no per-person pages anywhere.

## Assets & dependencies

- **Relative paths only.** Trip pages link back to the landing page and assets
  with relative hrefs (e.g. `../index.html`, `./index.html`). Nothing assumes a
  domain or absolute path, so it works at the `/vacationer/` Pages subpath.
- **No external dependencies except Google Fonts** (see Typography). No CDN
  frameworks, no JS libraries, no build tooling. Everything is inline `<style>`
  and a single inline `<script>`.

## Design system (shared chrome)

Shared `:root` tokens across trips: `--ink` navy `#1f3050`, `--paper` cream
`#faf6ee`, `--sage`/`--sage-deep` structural labels, `--pink` `#c08793`
masthead accent. Each trip then layers a **destination palette** (Hawaii: ocean
/ coral / gold / volcanic / palm / sage; Amsterdam: canal blue / mustard / brick
/ tulip / bottle green) used for the card swatch strip, phase colors, calendar
phase bars, and tags. Keep the palette declared in the trip's `fact-sheet.md`.

## Typography

**Default set** — use for the landing page and any new trip unless the trip
picks its own:

- **Federo** — display: hero title, section titles, card / city names, dates.
- **Jost** — body, labels, tabs, small caps (free Futura-style geometric sans).
- **Bodoni Moda** — italics and large numerals (section numbers, pulse counts).
  Set `font-variation-settings: "opsz" 11` on `body` so its hairlines (em
  dashes, apostrophes) stay visible at large sizes.

Declare them as `:root` tokens and use the tokens, not family names:
`--display` (Federo), `--sans` (Jost), `--serif` (Bodoni Moda). Federo has no
italic or bold, so italic text uses `--serif`; set `font-synthesis: none` to
avoid faux bold/italic.

**Trip-specific fonts are allowed.** A trip may swap in other free Google Fonts
that suit the destination. Keep the same three roles (display / sans / serif)
and the token names, and record the choice in a `## Typography` section of the
trip's `fact-sheet.md`.

*Legacy:* Amsterdam and Hawaii still use the original Fraunces + Manrope pair.

## Versioning

Every revision that ships to `main` bumps the version (`v1`, `v2`, …) once.
Work in progress on a branch does **not** bump it per commit — a new trip stays
`v1` until it first merges, and an edit batch gets a single bump when it merges.
The version lives in **three** places that must agree:

1. the footer `<span class="footer-mark">vNN</span>`,
2. the fact sheet's `HTML version stamp:` line,
3. (read automatically) the print stamp — sources from `.footer-mark`, so just
   updating the footer is enough.

When adding a new trip, also add its card to the root `index.html` (with
`trip-num`, dates, base, group, status) and bump the landing page's footer
version (once per merge, not per commit).
