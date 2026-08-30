# 🪦 Product Graveyard

A little tribute to every product Big Tech has ever killed.

`product_graveyard` pulls together three well-known "graveyard" datasets —
[Killed by Google](https://killedbygoogle.com/), [Killed by Microsoft](https://killedbymicrosoft.info/), and
[Killed by Apple](https://killedbyapple.theden.sh/) — into one normalized dataset, plus a single-file
web dashboard for browsing, filtering, and sorting the carnage.

## What's in here

| File | Purpose |
|---|---|
| [`graveyard.py`](graveyard.py) | CLI script that fetches all three sources, merges/dedupes them into one schema, runs basic QA checks, and exports `graveyard_all.json` / `.csv` / `.meta.json` (optionally `.xlsx`). |
| [`graveyard.html`](graveyard.html) | Standalone, no-build-step dashboard. Reads `graveyard_all.json` and renders a sortable, filterable, searchable table. |
| [`changelog.html`](changelog.html) | Version history view, styled to match the dashboard — see [`CHANGELOG.md`](CHANGELOG.md) for the plain-text source. |
| [`.github/workflows/refresh-data.yml`](.github/workflows/refresh-data.yml) | Runs `graveyard.py` daily (and on demand) and commits the refreshed snapshot back to the repo. |

### Architecture: static snapshot, not a live client-side fetch

`graveyard.py` is the only thing that talks to the three upstream sources ([jsDelivr](https://www.jsdelivr.com/)
CDN mirrors of their GitHub repos). It writes a normalized snapshot — `graveyard_all.json` — which acts
as this project's "database." `graveyard.html` only ever reads that committed file; it does **not** call
the upstream sources itself. A [GitHub Actions workflow](.github/workflows/refresh-data.yml) re-runs the
script daily (and on every push that touches `graveyard.py`, or manually via "Run workflow") and commits
the updated snapshot, so the dashboard stays current without needing a server or a real database — it's
still just static files.

One wrinkle a static snapshot introduces: whether a product counts as `"dead"` or `"scheduled"` depends
on today's date, which drifts between refreshes. `graveyard.html` recomputes that status client-side from
each row's `date_close`/`year_close` against the viewer's own clock, so it stays accurate even if the
snapshot is a few days old.

## Data schema

Every product, regardless of source, is normalized to the same set of fields:

```
company, name, type, date_open, date_close, year_open, year_close,
date_precision, lifespan_years, status, description, link, source
```

- `date_precision` — `"day"` for Google/Microsoft, `"year"` for Apple (Apple's source only tracks birth/death years).
- `status` — `"dead"` if already discontinued, `"scheduled"` if the shutdown date is still in the future.
- `lifespan_years` — time between launch and shutdown, in years.

## Usage

### Command line (JSON + CSV)

```bash
python3 graveyard.py
```

Add `--xlsx` to also produce an Excel workbook (requires `pip install openpyxl`):

```bash
python3 graveyard.py --xlsx
```

Outputs `graveyard_all.json`, `graveyard_all.csv`, `graveyard_all.meta.json` (generation timestamp +
per-source counts, used by the dashboard's "last updated" line), and — with `--xlsx` —
`graveyard_all.xlsx`, all in the current directory. Use `--out <name>` to change the output filename
prefix (the dashboard expects the default `graveyard_all` prefix, so only change it for one-off exports).
The script also prints a per-source summary, a breakdown of shutdowns by decade, and any QA warnings it
finds (e.g. missing death year, death date before launch date).

### Web dashboard

Run `python3 graveyard.py` at least once to produce `graveyard_all.json`, then open `graveyard.html` in
a browser — it reads that file via `fetch()`, which some browsers block when opening the page directly
via `file://`. If so, serve it locally instead:

```bash
python3 -m http.server
```

then visit `http://localhost:8000/graveyard.html`. In production this repo's
[GitHub Actions workflow](.github/workflows/refresh-data.yml) keeps `graveyard_all.json` refreshed for
you, so you don't need to run the script yourself unless you're developing locally.

The dashboard lets you:
- Toggle sources on/off (Google / Microsoft / Apple)
- Filter by product type and status (dead vs. scheduled)
- Search by name/description
- Sort by any column
- Download the currently filtered view as CSV or JSON

## Requirements

- Python 3.7+ (standard library only — no dependencies needed for JSON/CSV output)
- `openpyxl` only if using `--xlsx`
- Any modern browser for `graveyard.html` / `changelog.html`. Both load
  [Anime.js](https://animejs.com/) from cdnjs for animation — this needs
  outbound internet access; if it's blocked, both pages still work
  correctly, just without motion.

## Credits

All data comes from the excellent open-source graveyard projects maintained by their respective
authors — this repo just merges and re-presents it:

- [codyogden/killedbygoogle](https://github.com/codyogden/killedbygoogle)
- [fabianoriccardi/killed-by-microsoft](https://github.com/fabianoriccardi/killed-by-microsoft)
- [TheDen/killed-by-apple](https://github.com/TheDen/killed-by-apple)
