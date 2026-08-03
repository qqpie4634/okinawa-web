# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This repo is a single self-contained static HTML page: `index.html`. It is a
mobile-first itinerary/PWA for a family trip to Okinawa ("2026 沖繩親子自駕全攻略"),
in Traditional Chinese (`lang="zh-TW"`). There is no build system, package
manager, server, or test suite — the entire site is one HTML file with inline
`<style>` and `<script>` blocks, served as-is (e.g. opened directly or hosted
via static file hosting / GitHub Pages).

## Development workflow

- **No build/install/lint/test commands exist.** Do not add a bundler,
  package.json, or test framework unless the user explicitly asks for one.
- To preview changes, just open `index.html` in a browser (or serve the
  directory with any static file server, e.g. `python3 -m http.server`).
- All styling and behavior is edited directly in `index.html`. There are no
  separate CSS/JS files to keep in sync.
- Dependencies are loaded from CDNs at runtime (no local install step):
  - Tailwind CSS (via `cdn.tailwindcss.com`)
  - Font Awesome 6.4.0 icons
  - Google Fonts (`Noto Sans TC`)
  - Open-Meteo API for live weather data (called client-side via `fetch`)

## Architecture

### Single-page, tab-based itinerary

The page models an 8-day trip as tabs: Day 0 ("行前" / pre-trip prep) through
Day 7. Structure to know before editing:

- Each day is a `<div id="content-N" class="day-content ...">` block
  (`N` = 0–7), toggled via `showDay(N)` in the inline `<script>`. Only one
  `day-content` is visible at a time (`.hidden` class), driven by nav buttons
  `#tab-N` with matching `onclick="showDay(N)"`.
- If you add/remove a day, you must update **three places in sync**: the
  `<nav>` tab buttons, the `content-N` div, and the `ids` arrays / `showDay`
  range checks in the script (see `checkAutoRedirect`, which hardcodes the
  Day 1–7 window as `diffDays >= 0 && diffDays <= 6`).
- Each day's content follows a consistent internal pattern: an optional
  "查看今日動線圖" button (`toggleMap(N)`) that expands a Google Maps deep
  link in a `#map-area-N` container, a timeline of `card` entries (each with
  a Google Maps search link, a `time-badge`, and often a `tip-box`), a
  `plan-b-section` for alternate/backup plans, and an `expert-tips-section`
  ("導遊小叮嚀") with curated tips. When adding a new stop, copy an existing
  `card`/`plan-b-item`/`expert-item` block rather than inventing new markup —
  consistency across days matters more than cleverness here.
- All location links use `https://www.google.com/maps/search/?api=1&query=...`
  (single-stop) or `https://www.google.com/maps/dir/.../.../` (multi-stop
  route for the day's "總導覽/總導航" button). Keep this convention when
  adding new links.

### Client-side state and behavior (inline `<script>`)

- **Date-driven auto-navigation**: `checkAutoRedirect()` (run on
  `DOMContentLoaded`) reads a trip start date (`localStorage.tripStartDate`,
  default `2026-01-22`) and auto-switches to the tab matching today's date
  relative to that start date, falling back to Day 0 outside the trip window.
  This is the reason the page has both a real-world clock dependency and a
  `#trip-start-date` input (`saveDate()`/`loadDate()`) for the user to adjust it.
- **Test/simulation hook**: `testSimulateDate(dayIndex)` sets
  `localStorage.simDayDiff` and reloads, letting a user preview any day's tab
  without waiting for the real date. `simDayDiff` is read once then deleted
  ("用一次即焚"). Keep this working if you touch the date logic — it's the
  only way to manually QA `checkAutoRedirect` across all 8 days.
- **Packing checklist persistence**: checkboxes with ids like `cb-license-tw`
  are persisted individually to `localStorage` via `saveCheckbox(id)` /
  `loadCheckboxes()`. The list of tracked ids is hardcoded in
  `loadCheckboxes()` — if you add a new packing-list checkbox, add its id to
  that array too, or it won't be remembered.
- **Copy-to-clipboard**: `copyText(text)` (used for phone numbers) writes to
  the clipboard and shows a toast (`#toast`) via a CSS class toggle, not a
  JS animation library.
- **Weather modal**: `showWeatherModal()` fetches live conditions for Naha
  (hardcoded lat/lon 26.2124/127.6809) from Open-Meteo and maps WMO weather
  codes to Font Awesome icons/labels inline in the function — there's no
  shared weather-code lookup table, so extend the if/else chain in place if
  adding new codes.

### Versioning convention

The page title/header string embeds a version number (currently v6.9) that
the previous author bumped by hand with each feature change, and inline HTML
comments near changed sections note what changed per version (e.g. "v6.8
Updated", "v6.7 新增"). Follow this convention: when making a non-trivial
content or feature change, bump the version in the `<title>` and header `<p>`,
and leave a short version-tagged HTML comment near the change, consistent
with the existing history in the file.
