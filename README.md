# SCSG — Simple Clinic Schedule Generator

A single-page, offline-first staff scheduling tool for any clinic.

- No installation
- No build step
- No internet required
- Runs entirely in the browser
- **No personal data baked in** — you enter your own staff

## Table of contents

- [Author](#author)
- [Version](#version)
- [License](#license)
- [Privacy](#privacy)
- [Scope](#scope)
- [Features](#features)
- [How to run](#how-to-run)
- [Project layout](#project-layout)
- [Data & backups](#data--backups)
- [Scheduling rules at a glance](#scheduling-rules-at-a-glance)
- [Exports](#exports)
- [Current limitations](#current-limitations)
- [Development notes](#development-notes)

## Author

**Richard Korbla Adzido**

## Version

**1.1.0** — current release.

## License

Source code is released under the **MIT License** — see `LICENSE`.

## Privacy

This app is fully offline. It collects **no** data, sends **no** data,
uses **no** cookies, and calls **no** external services. All configuration
lives in your browser's `localStorage` under the key **`clinicRosterState`**.

No real clinic name, location, or staff name is embedded in this source
code. The default staff members are **generic placeholders**
(`Nurse A`, `Support A`, `Staff A`, …).

Full details: [`PRIVACY.md`](PRIVACY.md).

## Scope

SCSG generates **one month per schedule, by design**. Each generation is a
self-contained simulation for the selected month, seeded deterministically
from the target date and current staff configuration, so the same inputs
always produce the same output.

The engine also carries per-staff continuity state (last shift, consecutive
working days, duty counts, off-day debt). A generated month's outgoing state
is persisted in `localStorage` under a per-schedule key so the **following**
month can chain from it when regenerated later. Chaining is silent: no UI
surface reports whether a predecessor was found or whether the chain was
truncated.

Chained state uses **camelCase keys exclusively** (`lastShifts`,
`consecutiveDays`, `dutyCounts`, `offDayDebt`). Legacy snake_case keys from
pre-1.1.0 entries (`last_shifts`, `consecutive_days`, `duty_counts`,
`off_day_debt`, plus the retired `carryover_streaks`) are accepted on read
and collapsed to the canonical camelCase view; they never appear on write.

## Features

### Staffing model
- Three staff categories (**Nursing**, **Support**, **General**) with
  distinct shift rules.
- **12-Hour (M / N)** or **8-Hour (M / A / N)** shift systems, selectable
  per schedule.
- **Afternoon shift (A)** is fully supported in the 8-hour system, rendered
  with a dedicated yellow/orange colour code in both exports.

### Coverage and safety
- **Strict night coverage invariant**: at least 2 workers on night under
  the 12-hour system (or at least 1 nurse on night under the 8-hour
  system). The engine escalates through Nurse → Support → Night-eligible
  General staff (under the 8-hour system only nurses can satisfy the
  floor, since a nurse is required).
- **Retry-then-abort**: if no attempt meets the coverage floor within
  the retry budget (up to **50 attempts** with fresh seeds), the app
  refuses to export and reports the unachievable day(s) rather than
  downloading an unsafe schedule.
- **Night requires a nurse**: support/general staff are never the sole
  night attendant.
- **Burnout prevention**: nurses are deprioritized for shift assignment
  once they reach **5 consecutive working days** (target: 4). A nurse at
  5+ days can still be selected if doing so is necessary to maintain
  minimum coverage.
- **Pre-off night avoidance (INV3)**: a nurse is not assigned Night
  immediately before an off-day when a workaround is available. The engine
  first tries a peer swap with another nurse on Morning, then downgrades
  the Night to Morning if no safe swap exists. This invariant is enforced
  when the nursing team has **4 or more** members.

### Fairness and rotation
- **Fairness tracking**: per-staff counters for M / A / N / X duties
  balance shift assignments across the generated month.
- **Automated nursing off-day rotation**: each nurse receives **one**
  off-day per week, scheduled on a rotating day of week so off-days are
  spread across the team. Off-day positions rotate on a weekly basis.
- **Manual off-day configuration** for support and general staff, capped
  at 3 days; support staff are system-managed when nurse coverage is
  limited.

### Scheduling engine
- Deterministic simulation for the selected month, seeded from the target
  date, the retry attempt index, and a stable serialization of the staff
  list, so identical inputs always produce identical output.
- Automatic invariant repair passes (INV1–INV4) plus emergency floor
  enforcement.

### Configuration and persistence
- **Clinic name field** — used in exports and as the filename prefix.
  Falls back to **"Simple Clinic"** when empty (so the field is defaulted,
  not required).
- **Dynamic default staff templates** matched to the active shift system.
- **Local persistence** via `localStorage` under `clinicRosterState`.
- **JSON export / import** of the full configuration as a `.scsg` file.
- **Silent migration** from pre-1.1.0 snake_case keys on load; every
  subsequent write emits camelCase only.

### Exports
- **PDF** — landscape A4, colour-coded M / A / N / X cells with legend.
  Staff are paginated **12 per page**; when there is more than one page
  the month line includes a `(Page N of M)` suffix.
- **Excel (.xlsx)** — OOXML package with one worksheet per 12-staff group.
  The sheet is named `Roster` when there is a single sheet, and
  `Roster 1`, `Roster 2`, … when there are multiple. The workbook's
  content-types, sheet list, and relationships are all built dynamically
  so the package stays valid for any number of sheets.

### Interface
- Responsive layout for desktop and mobile (single-column stack below 680px).
- Custom-rendered Night toggle with disabled/locked states for
  nursing/support staff.
- App footer with dynamic version metadata injected from
  `<meta name="version">`.

## How to run

1. Open `scsg.html` in any modern browser (Chrome, Edge, Firefox, Safari).

No toolchain, no build step, no dependencies. The app consists of three plain
files: `scsg.html`, `assets/scsg.css`, `assets/scsg.js`.

### Deployment self-test

Append `?selftest=1` to the URL to run the built-in self-test on load:

```
file:///path/to/scsg.html?selftest=1
```

The test logs `SCSG Self-Test Initialized.` and
`SCSG Self-Test Completed Successfully.` to the console when all
core-state assertions pass.

## Project layout

```
scsg.html              Entry point / page shell
assets/
  scsg.css             Stylesheet
  scsg.js              Scheduler + exporters + state management
README.md              This file
PRIVACY.md             Privacy policy
CHANGELOG.md           Version history
LICENSE                MIT license text
```

## Data & backups

The current configuration is saved automatically to your browser's
`localStorage`. **Clearing browser history/data will delete it.**

To preserve it:

1. Click **Export Configuration** — downloads
   `[clinic_name]_[shift_tag]_shift_config.scsg`.
2. Restore it any time with **Import Configuration**.

The exported `.scsg` file is a plain JSON payload wrapped with
`format`, `version`, and `exportedAt` metadata around the state object.
You can inspect it safely in a text editor. Staff entries are written with
camelCase keys (`offDays`, `nightStaff`); imports accept both camelCase and
the legacy snake_case spellings so older backups round-trip cleanly.

## Scheduling rules at a glance

A condensed view of the invariants enforced by the engine — the in-app
**Scheduling Invariants** panel has the authoritative wording.

| Invariant | Meaning |
|---|---|
| **INV1 — Night floor** | 12h: at least 2 night staff (at least 1 must be a nurse if any support/general is on night). 8h: at least 1 nurse on night. Escalates Nurse → Support → Night-eligible General under the 12-hour system. |
| **INV2 — Day coverage** | 12h: at least 1 nurse on Morning. 8h: at least 1 nurse on Morning **and** at least 1 nurse on Afternoon. |
| **INV3 — Pre-off night** | A nurse is not assigned Night immediately before an off-day. First the engine tries a safe swap with another nurse on Morning; if none exists, it downgrades the Night to Morning. Enforced when 4+ nurses are configured. |
| **INV4 — General night ban** | General staff without Night eligibility are never assigned Night. |

## Exports

| Staff count | PDF pages | Excel sheets |
|---|---|---|
| 0–12  | 1 | 1 (`Roster`) |
| 13–24 | 2 | 2 (`Roster 1`, `Roster 2`) |
| 25–36 | 3 | 3 (`Roster 1`, `Roster 2`, `Roster 3`) |
| n     | `ceil(n / 12)` | `ceil(n / 12)` |

Each PDF page / Excel sheet renders the same title block, two day-blocks,
and legend — only the staff rows differ.

Filename pattern:
`[clinic_name]_[shift_tag]_schedule_[month]_[year].[pdf|xlsx]`
where `shift_tag` is `12h` or `8h`.

## Current limitations

- **Clinic name is defaulted, not required**: an empty value is replaced
  with `Simple Clinic` on blur/export.
- **No undo** across regeneration: hitting Generate runs the scheduler
  from scratch; there is no "previous output" recall.

## Development notes

- **Determinism**: `buildMonthSeed(year, month, salt, staff)` combines the
  target date, retry attempt index, and a stable serialization of the
  current staff list into an FNV-1a hash. The scheduler feeds this into a
  mulberry32 PRNG, so any given (year, month, staff, attempt) tuple
  produces exactly one schedule. The serialization reads `staff[i].offDays`
  — the property name changed with the camelCase pass, but the produced
  string (name + role + joined off-days) is byte-identical to the
  pre-migration form, so schedules do not drift across the rename.
- **Cross-month chaining**: successive generations of the same clinic and
  shift system are stitched together through per-schedule state entries
  stored in `localStorage` under `<stem>_scsg_state` keys, where `<stem>`
  is the exported schedule filename minus the extension. The previous
  month's outgoing state is read on every generation and fed into the
  scheduler as `incomingStateData`. Chaining is silent — no UI surface
  reports whether a predecessor was found.
- **Out-of-order generation truncates the chain**: writing a month deletes
  every stored state entry strictly after it for the same clinic and shift
  system. Chained state is cumulative, so rewriting an earlier month
  invalidates every later entry — clearing them forces subsequent
  generations to fall back to empty state rather than silently chaining
  from a predecessor that no longer matches what they were built on.
- **camelCase state, camelCase writes**: every persisted entry emits
  `lastShifts`, `consecutiveDays`, `dutyCounts`, and `offDayDebt`.
  `applyDutyCountDecay` normalizes legacy snake_case aliases and the
  retired `carryover_streaks` / `carryover` fields on read so
  pre-migration entries keep working. There is no dual-write path — the
  legacy names survive only as read fallbacks and disappear as old
  entries are regenerated.
- **Duty counts decay with age**: inherited `dutyCounts` are multiplied
  by `DUTY_COUNT_DECAY = 0.5` on read. Chained counts are cumulative, so
  without decay a long-tenured staff member's counters dwarf a new hire's
  and `autonomousCandidateOrder` — which sorts ascending by load — would
  punish the newcomer for months while veterans coast. Decay weights
  older months geometrically lower, bounds total load to
  `(this month's load) / (1 - DUTY_COUNT_DECAY)`, and gives roughly a
  three-month effective memory.
- **Off-day debt amortizes**: outstanding `offDayDebt` is repaid by
  `OFF_DAY_DEBT_REPAYMENT = 1` per generation, after the phase shift has
  been consumed and before this month's incursions land. Off-day debt
  perturbs the weekly off-day rotation phase rather than adding an extra
  rest day — the scheduler enforces exactly one off-day per nurse per
  week and cannot deliver true compensation without violating the
  coverage floor. A burst of N lost off-days therefore fades over roughly
  N months instead of persisting indefinitely.
- **No build step**: every module is a plain script loaded with `defer`.
  Functions referenced from `onclick=` attributes are intentionally global.
- **Exports**: PDFs are generated by writing raw PDF 1.4 syntax directly
  (catalog, page tree, embedded Helvetica / Helvetica-Bold, WinAnsi
  encoding). XLSX files are generated as stored (uncompressed) ZIP
  archives containing OOXML parts.