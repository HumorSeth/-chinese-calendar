# China & US Holidays ICS Calendars (2026–2100)

[中文](README.cn.md) | English

Two ready-to-import / ready-to-subscribe iCalendar datasets covering
**2026-01-01 through 2100-12-31**: one for China, one for the United States.
Every event is a **single all-day event** with a **plain-English** name — neither
file contains a single non-ASCII byte — and both follow RFC 5545 and are tuned
for **Tuta Calendar**'s importer and URL-subscription features: no recurrence
rules, deterministic UIDs, pure ASCII, CRLF line endings, lines folded at 75 octets.

Every date is derived from the rule, never typed in by hand, and then
cross-checked against an independent implementation.

## Calendars at a glance

| File | Events | Categories (`CATEGORIES`) | Events per year |
|---|---|---|---|
| `chinese-holidays-2026-2100.ics` | 3,900 | `Public Holiday` 1,650 · `Traditional Festival` 450 · `Solar Term` 1,800 | 51–53 |
| `us-holidays-2026-2100.ics` | 1,835 | `US Federal Holiday` 1,160 · `US Traditional Holiday` 675 | 23–27 |

Both files span the same 75 Gregorian years (2026–2100). Multi-day breaks are
split into one event per day — `Chinese New Year (Day 1)` … `(Day 14)`,
`National Day (Day 1)` … `(Day 3)`, `Thanksgiving Day (Day 1)` … `(Day 4)` —
while single-day holidays carry no suffix.

## Chinese calendar — what's inside

| Category | Contents | Events |
|---|---|---|
| `Public Holiday` | New Year's Day; Chinese New Year (lunar month 1, days 1–14); Qingming Festival; Labor Day; Dragon Boat Festival; Mid-Autumn Festival; National Day (Oct 1–3) | 1,650 |
| `Traditional Festival` | Chinese New Year's Eve, Lantern Festival, Qixi Festival, Zhongyuan Festival, Double Ninth Festival, Laba Festival | 450 |
| `Solar Term` | The 24 solar terms with their standard English names (Beginning of Spring, Spring Equinox, White Dew …) | 1,800 |

52 events per Gregorian year on average — the Laba Festival occasionally occurs
twice in one Gregorian year and not at all in the next, so the per-year count
varies between 51 and 53 while the 75-year total stays fixed.

## US calendar — what's inside

| Category | Contents | Events |
|---|---|---|
| `US Federal Holiday` | New Year's Day; Martin Luther King Jr. Day; Presidents' Day; Memorial Day; Juneteenth; Independence Day; Labor Day; Columbus Day; Veterans Day; Thanksgiving Day (Day 1–4); Christmas Day — plus an `(Observed)` event whenever a fixed-date holiday lands on a weekend | 1,160 |
| `US Traditional Holiday` | Valentine's Day, St. Patrick's Day, Good Friday, Easter Sunday, Mother's Day, Father's Day, Halloween, Christmas Eve, New Year's Eve | 675 |

### US federal holiday rules

| Holiday | Rule |
|---|---|
| New Year's Day | January 1 |
| Martin Luther King Jr. Day | 3rd Monday of January |
| Presidents' Day (Washington's Birthday) | 3rd Monday of February |
| Memorial Day | Last Monday of May |
| Juneteenth | June 19 |
| Independence Day | July 4 |
| Labor Day | 1st Monday of September |
| Columbus Day | 2nd Monday of October |
| Veterans Day | November 11 |
| Thanksgiving Day | 4th Thursday of November (Fri–Sun follow as Day 2–4) |
| Christmas Day | December 25 |

**Observed rule** (5 U.S.C. § 6103): a holiday falling on Saturday is observed
the preceding Friday, one falling on Sunday the following Monday. The statutory
date always keeps its own event and the shifted date gets a separate
`(Observed)` event, so both appear on the calendar. That produces one event that
sits outside its own year: `New Year's Day (Observed)` on 2100-12-31 belongs to
1 January 2101.

## Verification

**US calendar**

- All 9 rule-based holidays were recomputed with `dateutil.rrule` (an
  independent RRULE expansion) across all 75 years: **0 differences**.
- Easter and Good Friday were recomputed with `dateutil.easter` (anonymous
  Gregorian computus): **0 differences**; Good Friday is always Easter − 2 days.
- Observed-date events were counted against a separately written rule:
  New Year's Day 22, Juneteenth 22, Independence Day 23, Veterans Day 21,
  Christmas Day 22 — exact match, including the cross-year case above.
- Thanksgiving: all 75 years have exactly 4 consecutive events starting on the
  4th Thursday of November.
- File parsed with `icalendar` 7.3: 1,835 events, unique UIDs, every event an
  all-day `DATE` value, no line longer than 75 octets, file ends with
  `END:VCALENDAR`.

2026 anchors (US): MLK Day 1/19, Presidents' Day 2/16, Memorial Day 5/25,
Juneteenth 6/19 (Fri), Independence Day 7/4 (Sat) + observed 7/3, Labor Day 9/7,
Columbus Day 10/12, Veterans Day 11/11, Thanksgiving 11/26–11/29, Christmas 12/25.

**Chinese calendar**

- 2026 anchors match the State Council notice item by item: Chinese New Year
  2/17, New Year's Eve 2/16, Qingming 4/5, Dragon Boat 6/19, Mid-Autumn 9/25,
  National Day 10/1–10/3.
- 2,000+ lunar dates agree across `lunardate` ↔ `cnlunar` ↔ Hong Kong
  Observatory tables (including 2033's leap 11th month and 2061's leap 3rd month).
- 1,788 of 1,800 solar-term dates match the HKO tables exactly; in the remaining
  12 the astronomical instant falls within 10 minutes of midnight (e.g. Major
  Cold 2082 at 23:59:46) — inherent long-term ΔT uncertainty, so the published
  HKO date is adopted.

## ICS format specification

Shared by both files:

- `CALSCALE:GREGORIAN` / `VERSION:2.0` / `PRODID`
- All-day events: `DTSTART;VALUE=DATE:YYYYMMDD` — no `TZID`, no `VTIMEZONE`, no
  `DTEND` (a `DATE` value with no `DTEND` is a one-day event, RFC 5545 §3.6.1).
  Date-only events are timezone-independent by construction, so nothing shifts
  when the client sits in another zone.
- **No `RRULE` / `RDATE`**: every occurrence is a self-contained `VEVENT`.
- `TRANSP:TRANSPARENT` (holidays do not block busy/free time).
- `DTSTAMP` fixed at `20260101T000000Z`, so regenerating yields a byte-identical file.
- Deterministic UIDs such as `mlk-day-2026@us-holidays` or
  `chinese-new-year-day2-2026@cn-holidays`; re-importing the same file
  deduplicates by UID.
- Pure ASCII, CRLF, every line ≤ 75 octets (RFC 5545 folding).
- Fixed field set: `UID` / `DTSTAMP` / `DTSTART;VALUE=DATE` / `SUMMARY` /
  `CATEGORIES` / `TRANSP:TRANSPARENT`.

Two deliberate differences:

- US events also carry a `DESCRIPTION` with the date rule, the weekday of the
  occurrence, and any observed-date note.
- The Chinese file declares `METHOD:PUBLISH`; the US file omits it, because a
  bare data file should not look like a scheduling message.

## Usage

**Subscribe (recommended)** — desktop or mobile app → `＋` next to *Your
Calendars* → *From URL* → paste a public URL below → *Subscribe*. Subscribed
calendars are read-only, refreshed server-side, and do not sync alarms.

**Import (works on the free plan)** — web or desktop app → `⋯` next to the
target calendar → *Import* → pick the `.ics`. Import into a dedicated calendar:
Tuta has no bulk event deletion. Re-importing the same file creates no
duplicates (UID dedup).

**Subscription URLs**

```
https://raw.githubusercontent.com/HumorSeth/cn-us-calendar/main/us-holidays-2026-2100.ics
https://raw.githubusercontent.com/HumorSeth/cn-us-calendar/main/chinese-holidays-2026-2100.ics

# CDN mirror (faster in some regions)
https://cdn.jsdelivr.net/gh/HumorSeth/cn-us-calendar@main/us-holidays-2026-2100.ics
https://cdn.jsdelivr.net/gh/HumorSeth/cn-us-calendar@main/chinese-holidays-2026-2100.ics
```

Other clients (Google Calendar "From URL", Apple/Outlook subscriptions) accept
the same URLs.

## Reproducibility

There is no hidden data here: every rule needed to rebuild both files is in the
tables above, and the observed-date rule plus the Easter computus are spelled
out. The US file is produced by a ~200-line Python script that walks 2026–2100,
computes each holiday, emits one `VEVENT` per day, and folds lines at 75 octets;
the Chinese file additionally needs an ephemeris (`pyephem`) for the solar terms
and a lunar-calendar library (`lunardate`, `cnlunar`) for the festivals, both
cross-checked against the Hong Kong Observatory tables. Open an issue if you
want the generator scripts committed here.

## Known limitations

- **Chinese calendar**: marks the festival days themselves, not each year's
  State Council rest-day swaps and makeup workdays (published every November).
- **US calendar**: federal holidays only — no state-level holidays, no
  early-closing or half-day events.
- Neither file carries `VALARM`; Tuta does not sync alarms from subscriptions
  either, so import a VALARM build if you need reminders.
- Far-future dates: US rules are statutory and stable; Chinese solar terms
  within 10 minutes of midnight follow the published HKO tables.

## Sources

- US: [5 U.S.C. § 6103](https://uscode.house.gov/view.xhtml?req=granuleid:USC-prelim-title5-section6103),
  [OPM federal holidays](https://www.opm.gov/policy-data-oversight/pay-leave/federal-holidays/),
  `python-dateutil` (RRULE / Easter cross-checks), `icalendar` (RFC 5545 parsing)
- China: [Notice of the General Office of the State Council on the 2026 Holiday
  Arrangement](http://www.scio.gov.cn/zdgz/jj/202511/t20251110_938367.html),
  [Hong Kong Observatory Gregorian–Lunar Conversion Tables (1901–2100)](https://www.hko.gov.hk/en/gts/time/conversion.htm),
  `pyephem`, `lunardate`, `cnlunar`

## License

MIT — see [LICENSE](LICENSE).
