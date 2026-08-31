# East River — Passage Planner Tool

Single-file HTML tool that computes recommended East River entry times — northbound
(enter at The Battery) and southbound (enter at Throgs Neck) — for a fair current
the full length of the river.

## Live deployment

Unlike the other tools in this repo, this one is hosted on a different server —
GBR, not the Blue Horizon Sailing site:

```
https://burgeeregister.org/east_river/east_river_timing.html
```

Files live in `public_html/east_river/` on the GBR server:

```
public_html/east_river/
├── east_river_passage_planner.html
├── 8518750_annual_2026.xml
└── 8518750_annual_2027.xml
```

Updating the tool or a year's tide file is a straight file replace in that folder —
no build step, no restart. Data files are fetched fresh on every page load
(cache-busted), so a replaced file takes effect immediately without needing to
clear browser cache.

## Method

Rule of thumb, referenced to NOAA's Battery tide predictions:

- **Northbound (NYC → LIS):** enter at The Battery, Low Water + 1h45m.
- **Southbound (LIS → NYC):** enter at Throgs Neck, High Water + 1h45m.

A ~7 kt boat carries a fair current roughly 3 hours from that departure time. Faster
vessels get a longer window but risk outrunning the current if they depart too
early.

Hell Gate no longer has an official long-range NOAA current table, so this is a
Battery-only method — it does not attempt to time arrival at Hell Gate slack water.

Source rule: *East River — Timing Your Passage* (fair current, 1h45m rule of thumb).

## Data files

One file per calendar year, same directory as the HTML file. The tool loads
whichever year's file matches the selected date automatically — no manual
switching, and no edits to the HTML needed when a new year starts, as long as the
filename pattern (`8518750_annual_{YEAR}.xml`) is kept.

### Getting the data

NOAA Battery tide predictions (station 8518750), one year at a time:

```
https://api.tidesandcurrents.noaa.gov/api/prod/datagetter?product=predictions&begin_date={YEAR}0101&end_date={YEAR}1231&datum=MLLW&station=8518750&time_zone=lst_ldt&units=english&interval=hilo&format=xml
```

Replace `{YEAR}` with the year, save the response as `8518750_annual_{YEAR}.xml`.
Format doesn't matter beyond that — `.txt`, `.xml`, or `.csv` all parse the same
way; the tool detects XML vs. delimited text from the content, not the file
extension.

Refresh once or twice a year — whenever a new calendar year needs covering, or
NOAA revises predictions.

## Using the tool

1. **Date** — pick a "From date." Weekday shown alongside it.
2. Leave **"To date"** blank for a single day, or set it for a date range (can span
   multiple years; each year's file loads automatically as needed).
3. **Recommended entry times** — single day shows two cards (Northbound /
   Southbound); a range shows one row per date instead.
4. **Download CSV for this range** — exports the currently selected range (single
   day or multi-day) as a CSV: `Date, Direction, Reference, ReferenceTime,
   EntryTime`.
5. **Battery tide events for this date** (single-day mode only) — shows the raw
   high/low events read from file for that date. Editable: add, remove, or
   override for a one-off case without touching the source file.

## Coverage warning

If any requested date falls outside the loaded file's actual date range (wrong
year, missing file, or a file that doesn't cover the full year), a banner appears
above the results naming the affected dates and what coverage *is* loaded. Those
dates fall back to whatever's in the manual override table (empty, unless entered
by hand).

## Known limitations

- NOAA's printed tide tables do not always correct for Daylight Saving Time.
  Verify DST when reading times from data pulled outside the API's `lst_ldt` time
  zone.
- No Hell Gate current-table method — see *Method* above.
- Assumes a single rule-of-thumb offset (1h45m) rather than a full current-velocity
  model; faster or slower vessels should adjust the window accordingly (see
  *Method*).

## Changelog

- **v1.0** — Initial release: Battery-referenced entry-time calculation for both
  directions, single-day and date-range modes, per-year tide file auto-loading,
  CSV export, manual override table, coverage-gap warnings.
