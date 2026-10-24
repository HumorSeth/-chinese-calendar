# Chinese Holidays & Solar Terms ICS Calendar (2026–2100)

[中文](README.md) | English

A ready-to-subscribe / ready-to-import iCalendar (.ics) dataset covering
**2026-01-01 through 2100-12-31**, with **3,900 all-day events**, all named in
**plain English** (the file contains no Chinese characters). It complies with
RFC 5545 and is tailored for **Tuta Calendar**'s importer and subscription
feature (no recurrence rules, deterministic UIDs, pure ASCII, CRLF).

Every date was computed by actually running a program and cross-checked
day-by-day against the official Hong Kong Observatory tables; the 2026 dates
match the *Notice of the General Office of the State Council on the 2026
Holiday Arrangement* item by item.

## Contents & Statistics

| Category (CATEGORIES) | Contents | Events |
|---|---|---|
| Public Holiday | New Year's Day; Chinese New Year (lunar month 1, days 1–14, split as Day 1–14); Qingming Festival; Labor Day; Dragon Boat Festival; Mid-Autumn Festival; National Day (Oct 1–3, split as Day 1–3) | 1650 |
| Traditional Festival | Chinese New Year's Eve, Lantern Festival, Qixi Festival, Zhongyuan Festival, Double Ninth Festival, Laba Festival | 450 |
| Solar Term | The 24 solar terms with standard English names (Beginning of Spring / Spring Equinox / White Dew …) | 1800 |
| **Total** | 52 events per Gregorian year (Laba Festival occurs twice in a few years and not at all in the next one; the total is unchanged) | **3900** |

Single-day holidays carry no "Day N" suffix; multi-day breaks are split into
independent per-day events, e.g. `Chinese New Year (Day 1)` ~ `(Day 14)` and
`National Day (Day 1)` ~ `(Day 3)`.

## Files

| File | Description |
|---|---|
| `chinese-holidays-and-solar-terms-2026-2100.ics` | Full file (3,900 events, 774 KB) |
| `cn-holidays-2026-2100.ics` | Public holidays only (1650) |
| `cn-traditional-festivals-2026-2100.ics` | Traditional festivals only (450) |
| `cn-solar-terms-2026-2100.ics` | Solar terms only (1800) |
| `chinese-calendar-events-2026-2100.csv` | Day-by-day audit list of all events (date / summary / category / uid) |
| `gen_cn_calendar.py` | Generator script: computes all dates, runs every check, writes the ICS and CSV |
| `hko/T2026e.txt … T2100e.txt` | Hong Kong Observatory Gregorian–Lunar conversion tables (2026–2100, the cross-validation baseline) |
| `site/` | Deployment bundle (the files above plus a landing page — see "Deployment") |

## How the Dates Were Computed (nothing from memory)

- **24 solar terms**: `pyephem` bisects the Sun's **apparent ecliptic longitude
  referred to the equinox of date** to the exact instant it reaches each
  multiple of 15°; the instant is converted to UTC+8 and the Beijing calendar
  date is taken. Key detail: you must call `sun.compute(date, epoch=date)` to
  reduce to the equinox of date; using the J2000 ecliptic instead introduces a
  precession error of about 0.36° (≈ 9 hours), which pushes many solar terms
  one day late.
- **Lunar festivals**: the `lunardate` library (its tables derive from the
  Hong Kong Observatory). The 14 Chinese New Year days are lunar month 1,
  days 1–14; Chinese New Year's Eve is the day before the next CNY (correct
  whether the 12th lunar month has 29 or 30 days).
- **Lunar year 2100**: `lunardate`'s tables end at 2099 and `cnlunar`'s at
  mid-2100, so lunar year 2100 (including CNY on 2100-02-09 and the Eve on
  2100-02-08) is derived directly from the HKO table.
- **Cross-validation**: every lunar date is checked across `lunardate` ↔
  `cnlunar` ↔ HKO; every solar term date is checked across `pyephem` ↔ HKO.

### Verification results

- All 2,000+ lunar dates agree across all three sources (including 2033's
  leap 11th month and 2061's leap 3rd month; HKO marks leap months by
  repeating the month ordinal).
- Of the 1,800 solar term dates, 1,788 are identical between `pyephem` and
  HKO. In the remaining 12, the `pyephem` instant falls within 10 minutes
  before midnight (e.g. Major Cold 2082 at 23:59:46) — an inherent
  uncertainty of long-term ΔT prediction; **the published HKO dates are
  adopted** (`cnlunar`'s independent computation sides with HKO as well).
- The 2026 anchors match the State Council notice: CNY 2/17, Eve 2/16,
  Qingming 4/5, Dragon Boat 6/19, Mid-Autumn 9/25, National Day 10/1–10/3.
- HKO's own `T2069e.txt` is missing the row for 2069-12-30 (not a solar term
  nor any festival day; no impact).

## ICS format specification

- `CALSCALE:GREGORIAN` / `VERSION:2.0` / `PRODID` / `METHOD:PUBLISH`
- All-day events `DTSTART;VALUE=DATE:YYYYMMDD`, no `TZID`, no `VTIMEZONE`, no `DTEND`
- **No `RRULE` / `RDATE`**: every date is a self-contained `VEVENT`
- `TRANSP:TRANSPARENT` (does not block busy/free), `DTSTAMP` fixed at `20260101T000000Z`
- Deterministic UIDs: `chinese-new-year-day2-2026@cn-holidays`; re-importing
  the same file deduplicates by UID. Laba Festival occurs twice in
  2041/2060/2068/2079/2098, so those UIDs get a month suffix
  (`laba-festival-2041-01@…` / `laba-festival-2041-12@…`) to stay unique
- Pure-ASCII content, CRLF line endings, every line ≤ 75 octets
  (RFC 5545 folding rule)
- Fixed event fields: `UID` / `DTSTAMP` / `DTSTART;VALUE=DATE` / `SUMMARY` /
  `CATEGORIES` / `TRANSP:TRANSPARENT`

## Usage

**Subscribe (recommended; requires a paid Tuta plan)**: desktop client or
mobile app → next to *Your Calendars* in the calendar sidebar, click `＋` →
*From URL* → paste the public URL of the .ics → *Subscribe*. Subscribed
calendars are read-only, fetched periodically by Tuta's servers, and alarms
from external calendars are not synced.

**Import (works on the free plan)**: web app or desktop client → `⋯` next to
the target calendar → *Import* → choose the .ics file. Import into a dedicated
calendar (Tuta has no bulk event deletion); re-importing the same file creates
no duplicates (UID dedup).

Other clients (Google Calendar "From URL", Apple/Outlook subscriptions, …)
use the same URL.

## Deployment (getting a public subscription URL)

The URL must be publicly reachable without login (Tuta's servers fetch it).
The `site/` directory is the deployment bundle; pick one of three options:

1. **GitHub**: upload the files from `site/` to a public repo via the web UI,
   then use the raw link or
   `https://cdn.jsdelivr.net/gh/<user>/<repo>@main/<file>.ics`;
2. **Cloudflare Pages**: Workers & Pages → Upload assets → drag in `site/` →
   `https://<project>.pages.dev/<file>.ics`;
3. **Self-hosted**: Caddy `file_server` or nginx (`text/calendar`, HTTPS).

Step-by-step details in `site/README.md`.

## Regenerating

```bash
pip install ephem lunardate cnlunar icalendar   # cnlunar/icalendar are for cross-checks
python gen_cn_calendar.py
```

The script computes all dates, runs every check described above (non-zero
exit on any failure), rewrites the ICS and CSV, and prints the 2026 detail,
the list of 12 ephem-vs-HKO differences, and the CNY dates of the boundary
years (2026 / 2033 / 2100). `sxtwl` has no prebuilt wheel for Python 3.14 and
was therefore not used (its role is covered by lunardate + cnlunar + HKO).

## Known limitations

- Does **not** include the actual rest-day swaps and makeup workdays from
  each year's State Council notice (published every November; would need an
  annual append from 2027 on — a separate "this year's arrangement" calendar
  can be built if needed).
- No alarms (VALARM); Tuta does not sync alarms from subscriptions either —
  import a VALARM-equipped build if you need reminders.
- The 12 boundary-case solar terms follow the published HKO tables (see
  above); far-future ΔT uncertainty is unavoidable.
- The calendar marks the **festival days themselves** (statutory-festival
  scope), not the consecutive rest-day arrangements.

## Sources

- [Notice of the General Office of the State Council on the 2026 Holiday Arrangement](http://www.scio.gov.cn/zdgz/jj/202511/t20251110_938367.html) (Chinese)
- [Hong Kong Observatory Gregorian–Lunar Calendar Conversion Tables (1901–2100)](https://www.hko.gov.hk/en/gts/time/conversion.htm)
  (this repo uses their per-year `T{year}e.txt` text tables)
- Python libraries: `pyephem` (positional astronomy), `lunardate` /
  `cnlunar` (Chinese lunar calendar), `icalendar` (RFC 5545 parsing checks)

The generated files are free to use and redistribute.
