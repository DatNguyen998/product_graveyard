# Changelog

All notable changes to this project are documented here, newest first. Each
entry calls out exactly which file(s) and function(s)/rule(s) it touched, so
you can tell at a glance what a version actually changed under the hood.

This project doesn't publish releases/tags — "version" here just numbers
each meaningful round of changes in order (loosely following
[Keep a Changelog](https://keepachangelog.com/) style: Added / Changed / Fixed).

## [0.6.0] - 2026-08-30 — Animation engine swapped to Anime.js

### Added
- [Anime.js](https://animejs.com/) v3.2.1 (cdnjs) as the animation engine
  for both pages, loaded via a pinned CDN `<script>` tag. All animation is
  progressive enhancement: if the CDN is blocked/unreachable, `HAS_ANIME`
  is `false`, `<html>` gets a `.no-anime` class, and both pages still work
  correctly — just without motion (a small CSS-only fallback keeps the
  spinner spinning and the "scheduled" badge pulsing in that case).
  **Impacts:** `graveyard.html`, `changelog.html` — new `<script src=cdnjs.../anime.min.js>` tag
- `graveyard.html`: a FLIP-style reorder animation — row positions from
  before a sort/filter are captured and animated into their new position,
  instead of the table just jumping. New rows revealed by a filter still
  fade/slide in as before. Skipped automatically above 150 visible rows
  to avoid layout-thrashing jank.
  **Impacts:** `graveyard.html` — `render()` (rewritten), new `keyOf()`
- `graveyard.html`: an `anime.timeline()` sequences the initial page
  reveal (card → stats → controls) instead of all three fading in at once.
  **Impacts:** `graveyard.html` — new `introAnimation()`
- `graveyard.html`: a small scale "pop" on company chip click.
  **Impacts:** `graveyard.html` — `buildChips()`
- `changelog.html`: matching entrance treatment — card fades in, then each
  timeline entry staggers in via `anime.stagger()`.
  **Impacts:** `changelog.html` — new inline `<script>` block

### Changed
- Replaced every hand-rolled animation with an Anime.js equivalent:
  stat count-up, table-row entrance, lifespan-bar growth, the "scheduled"
  badge pulse, and the loading spinner.
  **Impacts:** `graveyard.html`
  - `animateCount()` — rewritten to tween a plain object via `anime()`
    instead of a manual `requestAnimationFrame` loop
  - Removed the old CSS `@keyframes fadeInUp` (unused now) and the
    `animation:` declarations on `.app`, `tr`, `.sched`, `.spinner`
  - New `schedPulseAnim` (module-level instance, paused and replaced on
    every `renderStats()` call — needed because the badge's DOM node is
    recreated each render, so a stale `loop:true` instance would otherwise
    keep ticking in the background indefinitely)
- `changelog.html`: same swap — `@keyframes fadeInUp` and the `animation:`
  declarations on `.app`/`.entry` removed in favor of the Anime.js timeline.
  **Impacts:** `changelog.html`

## [0.5.0] - 2026-08-17 — Static data snapshot ("database") instead of live client-side fetch

### Added
- `graveyard.py` now also writes `graveyard_all.meta.json` (generation
  timestamp, per-company counts, source credits) alongside its existing
  JSON/CSV/XLSX output.
  **Impacts:** `graveyard.py` — `main()` (new metadata block, `timezone`
  import added)
- `.github/workflows/refresh-data.yml` — runs `graveyard.py` daily (plus
  on-demand and on pushes touching `graveyard.py`) and commits the
  refreshed snapshot back to the repo. This is what keeps the "database"
  current without a server.
  **Impacts:** new file, no code touched

### Changed
- `graveyard.html` no longer calls the three upstream sources itself. It
  now reads the committed `graveyard_all.json` snapshot instead, removing
  the JS reimplementation of the parsing logic entirely.
  **Impacts:** `graveyard.html`
  - Removed: `SOURCES` (the 3 CDN URLs), `parseKbg()`, `parseKba()`,
    `PARSERS`, `clean()`, `yr()`
  - Added: `DATA_URL`/`META_URL` constants, `computeStatus()` (recomputes
    dead/scheduled client-side from each row's own date, so status stays
    accurate between snapshot refreshes without re-fetching anything),
    `loadMeta()` (populates the "last updated" line)
  - Rewritten: `load()` (single local fetch + friendlier error message),
    `buildChips()` (company list now derived from the data instead of a
    hardcoded source list), `render()` (dropped the now-unused fetch-error
    parameter)
  - Copy updated: header subtitle, footer, loading/error messages
- `README.md` — documented the new snapshot architecture and updated the
  usage instructions accordingly.
  **Impacts:** `README.md` — no code impact

## [0.4.1] - 2026-08-13 — Accent color matched to the glass theme

### Changed
- Swapped the leftover warm orange/gold accent (a holdover from the old dark
  theme) for a blue-to-violet gradient pulled from the background, so the UI
  reads as one cohesive palette instead of clashing.
  **Impacts:** `graveyard.html`
  - CSS custom properties `--accent`, `--accent-2`, `--accent-deep`, `--accent-grad`
  - Rules: `.chip.on`, `.bar`, `input:focus,select:focus`, `#csv:hover,#json:hover`

## [0.4.0] - 2026-08-13 — Frosted-glass redesign

### Changed
- Replaced the dark "graveyard" theme with a glassmorphism UI: one
  translucent, blurred card (`backdrop-filter: blur + saturate`) floating
  over a blue/purple gradient background, dark-navy text on light glass,
  and tinted pill backgrounds on company tags.
  **Impacts:** `graveyard.html`
  - New `.app` wrapper element (header/stats/controls/table/footer moved inside it)
  - Full `:root` color-token rewrite (`--text`, `--muted`, `--glass*`, `--border-soft*`,
    `--google-bg/-bd`, `--msft-bg/-bd`, `--apple-bg/-bd`, `--sched`, etc.)
  - Rules: `body`, `.app`, `header`, `.stat`, `input/select/button`, `.chip`,
    `table th` (added its own blur so it stays legible while sticky), `.tag`
- No behavioral/logic changes — all JS (data fetch, parsing, filtering,
  sorting, animation) is untouched.
  **Impacts:** none — `parseKbg()`, `parseKba()`, `load()`, `filtered()`,
  `render()`, `renderStats()`, `animateCount()` unchanged

## [0.3.0] - 2026-08-12 — Brighter UI + motion

### Added
- Count-up animation for the stat tiles on every filter change.
  **Impacts:** `graveyard.html` — new `animateCount()` function; `renderStats()`
  extracted as its own function (previously inline in `render()`) to track
  and animate from the previous displayed value
- Staggered fade-in for table rows, animated bar growth for the lifespan
  column, a pulsing "scheduled" badge, and a spinner for the loading state.
  **Impacts:** `graveyard.html` — `render()` (row `animation-delay`, two-phase
  bar width via `requestAnimationFrame`), `load()` (loading markup), new
  `@keyframes fadeInUp / pulseGlow / spin`
- Respect for `prefers-reduced-motion` — animation/transition durations
  collapse to ~0 for users who've asked for reduced motion.
  **Impacts:** `graveyard.html` — new `REDUCE_MOTION` constant (gates the
  JS-driven animations); new `@media (prefers-reduced-motion: reduce)` rule
  (gates the CSS-driven ones)

### Changed
- Brighter, more saturated color palette (still dark-based at this point) —
  gradient accent, higher-saturation Google/Microsoft/Apple tag colors,
  glow/shadow accents on hover.
  **Impacts:** `graveyard.html` — CSS `:root` tokens only; no HTML/JS changes

## [0.2.0] - 2026-08-12 — Documentation

### Added
- Full project README: overview, file inventory, data schema reference,
  CLI + dashboard usage instructions, requirements, and source credits.
  **Impacts:** `README.md` (rewritten from a 1-line stub) — no code impact

## [0.1.0] - 2026-08-12 — Initial release

### Added
- The data-aggregation CLI: fetches the three upstream graveyard datasets,
  normalizes them to one schema, dedupes, runs basic QA, and exports
  JSON/CSV (optionally XLSX).
  **Impacts:** `graveyard.py` (new) — `fetch_json()`, `parse_date()`,
  `years_between()`, `clean()`, `parse_kbg()`, `parse_kba()`, `build()`,
  `qa()`, `main()`
- The browser dashboard: fetches the same three sources client-side and
  renders a sortable, filterable, searchable table with CSV/JSON export.
  **Impacts:** `graveyard.html` (new) — `parseKbg()`, `parseKba()`, `load()`,
  `buildChips()`, `buildTypes()`, `filtered()`, `render()`, `download()`
