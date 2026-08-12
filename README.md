# 🪦 Product Graveyard

A little tribute to every product Big Tech has ever killed.

`product_graveyard` pulls together three well-known "graveyard" datasets —
[Killed by Google](https://killedbygoogle.com/), [Killed by Microsoft](https://killedbymicrosoft.info/), and
[Killed by Apple](https://killedbyapple.theden.sh/) — into one normalized dataset, plus a single-file
web dashboard for browsing, filtering, and sorting the carnage.

## What's in here

| File | Purpose |
|---|---|
| [`graveyard.py`](graveyard.py) | CLI script that fetches all three sources, merges/dedupes them into one schema, runs basic QA checks, and exports JSON/CSV (optionally XLSX). |
| [`graveyard.html`](graveyard.html) | Standalone, no-build-step dashboard. Fetches the same sources live in the browser and renders a sortable, filterable, searchable table. |

Data is always pulled fresh from source via [jsDelivr](https://www.jsdelivr.com/) CDN mirrors of the
respective GitHub repos — nothing is scraped or vendored, so both the script and the page reflect the
latest known deaths (and scheduled ones) each time you run them.

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

Outputs `graveyard_all.json`, `graveyard_all.csv`, and (with `--xlsx`) `graveyard_all.xlsx` in the
current directory. Use `--out <name>` to change the output filename prefix. The script also prints a
per-source summary, a breakdown of shutdowns by decade, and any QA warnings it finds (e.g. missing
death year, death date before launch date).

### Web dashboard

Just open `graveyard.html` in a browser — it fetches all three sources client-side. Since it uses
`fetch()`, opening it directly via `file://` may be blocked by some browsers; if so, serve it locally:

```bash
python3 -m http.server
```

then visit `http://localhost:8000/graveyard.html`.

The dashboard lets you:
- Toggle sources on/off (Google / Microsoft / Apple)
- Filter by product type and status (dead vs. scheduled)
- Search by name/description
- Sort by any column
- Download the currently filtered view as CSV or JSON

## Requirements

- Python 3.7+ (standard library only — no dependencies needed for JSON/CSV output)
- `openpyxl` only if using `--xlsx`
- Any modern browser for `graveyard.html`

## Credits

All data comes from the excellent open-source graveyard projects maintained by their respective
authors — this repo just merges and re-presents it:

- [codyogden/killedbygoogle](https://github.com/codyogden/killedbygoogle)
- [fabianoriccardi/killed-by-microsoft](https://github.com/fabianoriccardi/killed-by-microsoft)
- [TheDen/killed-by-apple](https://github.com/TheDen/killed-by-apple)
