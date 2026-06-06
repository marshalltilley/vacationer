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
└── hawaii-trip/
    ├── fact-sheet.md
    └── index.html
```

**One trip = one folder.** Each folder holds exactly two files:
a markdown `fact-sheet.md` and an `index.html`. The root `index.html` is the
landing page that links to each trip card.

## The fact sheet is the source of truth — always

`fact-sheet.md` is canonical. **Build the HTML *from* the fact sheet, never from
memory.** When a date, time, booking, or count changes:

1. **Edit `fact-sheet.md` first.** Bump its `HTML version stamp:` line.
2. *Then* regenerate or hand-edit `index.html` to match.
3. Bump the footer version (see Versioning) so the two stay in lockstep.

If the HTML and the fact sheet ever disagree, the fact sheet wins — reconcile by
fixing the HTML, not by quietly editing the fact sheet to match stale markup.

## Page template (per-trip `index.html`)

The structure is consistent across trips — copy an existing trip as the starting
point rather than inventing layout.

1. **Hero** — eyebrow (`Vacationer · No. NN` + season), big serif title, italic
   dek, then a 4-up `hero-meta` grid: **Window · Base · Travelers · Tone**.
2. **Sticky tab nav** (`.tabs`, `position: sticky; top: 0`) with a **Print
   button** pinned to the right. Tabs, in order:
   **Overview · Eat & do · Locked events · Logistics**, plus a sub-trip tab when
   the trip has one (Hawaii: `Big Island`; Amsterdam: `Day trip`).
   *(Tab DOM order may vary; keep the labels and their meaning consistent.)*
3. **Overview tab** holds, in this order:
   - a **presence map / Gantt** (`.timeline`) — one bar per party with arrival→
     departure windows, plus a headcount-by-day strip;
   - **phases** (`.phases`) — trip broken into named windows by group composition;
   - a **booking pulse** (`.pulse`) — Confirmed / Still-to-book / Flight-leg counts;
   - a **master calendar** (`.cal-wrap`) — full month grid, color-coded by phase,
     with locked events marked.
4. Remaining tabs hold their sections (locked events, restaurants + activities,
   reservations + transport + packing, sub-trip).
5. **Footer** with the version stamp, then a hidden `.print-stamp` div + the
   `<script>`.

## Tabs, deep links, and print — the JS contract

A small IIFE at the bottom of each page wires three things:

- **Tab switching:** clicking a `.tab` sets `.active` and toggles
  `panel.hidden` so only one `.tab-panel` shows.
- **Deep links via URL hash:** on load, `#<data-tab>` opens that tab. The four
  canonical deep links are **`#events`, `#bigisland`, `#eatdo`, `#logistics`**
  (Amsterdam uses `#daytrip` for its sub-trip). The hash must equal a tab's
  `data-tab` value — keep them in sync. *(Amsterdam also writes the hash back to
  the URL on tab click via `history.replaceState`; Hawaii currently only reads
  it. Prefer the Amsterdam read+write pattern for new trips.)*
- **Print:** the Print button (and `beforeprint`) stamps the hidden
  `.print-stamp` and calls `window.print()`. Print CSS hides the hero/tabs/footer
  and prints **only the active tab**, **landscape, one section per page**
  (`@page { size: letter landscape }` + `.tab-panel > .section { break-after: page }`).
  The **print stamp reads the version from `.footer-mark` automatically** — do
  not hardcode the version string in the JS (so the footer is the single source
  of the version number).

## Scope badges — flag anything that isn't whole-group

When an item applies to only part of the group, tag it with a color-coded
`.scope-badge` and explain the badges in a `.scope-legend` at the top of the
section. Hawaii's set: `bi` (Big Island sub-trip / M&A only), `golf` (golfers),
`pod` (a subset of people), `anna` (ballet/kid-specific). Never imply a
sub-trip, golf, or kid-specific item applies to everyone.

## Logistics — keep Confirmed and Still-to-book split

In the Logistics/bookings section, **"Confirmed — no action needed" and
"Still to book — action needed" are two separate `.res-grid` blocks under their
own `.res-subhead`. Never mix booked and unbooked items in one list.** Confirmed
items use `.res-item.confirmed` with a checked box; to-book items carry an
urgency chip (`high` / `med` / `low`).

The **booking-pulse counts on the Overview tab must equal the Logistics lists** —
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
- **No external dependencies except Google Fonts** (Fraunces + Manrope). No CDN
  frameworks, no JS libraries, no build tooling. Everything is inline `<style>`
  and a single inline `<script>`.

## Design system (shared chrome)

Shared `:root` tokens across trips: `--ink` navy `#1f3050`, `--paper` cream
`#faf6ee`, `--sage`/`--sage-deep` structural labels, `--pink` `#c08793`
masthead accent. Each trip then layers a **destination palette** (Hawaii: ocean
/ coral / gold / volcanic / palm / sage; Amsterdam: canal blue / mustard / brick
/ tulip / bottle green) used for the card swatch strip, phase colors, calendar
phase bars, and tags. Type: **Fraunces** (serif display) + **Manrope** (sans
body). Keep the palette declared in the trip's `fact-sheet.md`.

## Versioning

Every revision bumps the version (`v1`, `v2`, …) in **three** places that must
agree:

1. the footer `<span class="footer-mark">vNN</span>`,
2. the fact sheet's `HTML version stamp:` line,
3. (read automatically) the print stamp — sources from `.footer-mark`, so just
   updating the footer is enough.

When adding a new trip, also add its card to the root `index.html` (with
`trip-num`, dates, base, group, status) and bump the landing page's footer
version.
