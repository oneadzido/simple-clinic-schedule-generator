# Changelog

All notable changes to **SCSG — Simple Clinic Schedule Generator**.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

**Author:** Richard Korbla Adzido

---

## [1.1.0] — 2026-10-31

This release adds the clinic name, shift-system selection, fairness
tracking, stricter coverage invariants, paginated exports, and silent
month-to-month chaining on top of the 1.0.0 baseline.

### Added — shift systems and configuration
- **12-Hour (M / N)** and **8-Hour (M / A / N)** shift systems, selectable
  per schedule.
- **Afternoon shift (A)** fully integrated into the 8-hour system, with a
  dedicated yellow/orange colour code in both exports.
- **Clinic name field**: labels exports and prefixes output filenames.
  Falls back to `Simple Clinic` when empty (defaulted, not required).
- **Dynamic default roster templates**: loads the recommended staff
  template matching the selected shift system (the 8-hour template adds
  `Nurse D` and `Nurse E` on top of the 12-hour template).
- **App footer** displaying dynamic version metadata (sourced from
  `<meta name="version">`) and author copyright information.

### Added — coverage and safety invariants
- **Strict night coverage invariant**: at least 2 workers on night under
  the 12-hour system (at least 1 must be a nurse when any support/general
  staff is on night), or at least 1 nurse on night under the 8-hour
  system. The engine escalates through Nurse → Support → Night-eligible
  General staff (under the 8-hour system only nurses are eligible, since
  the floor requires a nurse on night).
- **Retry-then-abort gate**: if the coverage floor cannot be reached within
  the retry budget (up to **50 attempts** with fresh seeds), the app
  refuses to export and reports the unachievable day(s) rather than
  downloading an unsafe schedule.
- **Night requires a nurse**: support/general staff are never the sole
  night attendant.
- **Burnout prevention**: nurses are deprioritized for shift assignment
  once they reach **5 consecutive working days** (target: 4). A nurse at
  5+ days can still be assigned when needed to maintain minimum coverage.
- **Pre-off night avoidance (INV3)**: a nurse is not assigned Night
  immediately before an off-day when a workaround is available. The
  engine first tries a peer swap with another nurse on Morning; if no
  safe swap exists, it downgrades the Night to Morning. Enforced when
  the nursing team has **4 or more** members.
- **Automatic invariant repair passes** (INV1–INV4) plus emergency floor
  enforcement.

### Added — fairness and rotation
- **Fairness tracking counters**: per-staff counters for M / A / N / X
  duties balance shift assignments across the generated month.
- **Automated nursing off-day rotation**: each nurse receives **one**
  off-day per week, positioned on a rotating day of week so off-days are
  spread across the team.
- **Manual off-day configuration** for support and general staff, capped
  at 3 days; system-managed rotation is used for support staff when nurse
  coverage is limited.

### Added — cross-month chaining
- **Per-schedule state entries** stored in `localStorage` under
  `<stem>_scsg_state` keys, where `<stem>` mirrors the exported schedule
  filename. The previous month's outgoing state is read on every
  generation and fed into the scheduler as `incomingStateData`. Chaining
  is silent; the UI never reports whether a predecessor was found.
- **Forward invalidation on store**: writing a month deletes every
  stored state entry strictly after it for the same clinic and shift
  system, so an out-of-order regeneration cannot leave later months
  chaining onto a stale predecessor.
- **Exponential duty-count decay**: inherited `dutyCounts` are
  multiplied by `DUTY_COUNT_DECAY = 0.5` on read, giving roughly a
  three-month effective memory and preventing a long-tenured staff
  member's counters from dwarfing a new hire's.
- **Linear off-day debt amortization**: outstanding `offDayDebt` is
  repaid by `OFF_DAY_DEBT_REPAYMENT = 1` per generation, after the
  phase shift has been consumed and before this month's incursions land.
  Off-day debt perturbs the weekly rotation phase rather than adding an
  extra rest day, so a burst of N lost off-days fades over roughly N
  months rather than persisting indefinitely.

### Added — exports
- **Pagination for PDF and Excel**: staff are split into groups of **12
  per page / worksheet**. Each PDF page contains the full title block,
  both day-blocks, and legend; each Excel sheet carries the same title
  and legend. The worksheet is named `Roster` when there is a single
  sheet and `Roster 1`, `Roster 2`, … when there are multiple. A
  `(Page N of M)` / `(Sheet N of M)` suffix is injected into the month
  line only when there is more than one page or sheet.
- **Dynamic OOXML packaging**: `[Content_Types].xml`, `xl/workbook.xml`,
  and `xl/_rels/workbook.xml.rels` are generated on the fly so the
  workbook stays valid for any number of worksheets.
- **PDF page tree and xref** are compiled after all staff groups have
  been rendered, guaranteeing a valid file regardless of page count.
- **Legend reorganized horizontally** in both PDF and Excel outputs;
  reduced row heights in the PDF for efficient single-page scaling.

### Added — tooling
- **Deployment self-test**: appending `?selftest=1` to the URL runs a
  small runtime check on load and logs results to the console.
- **Deterministic seeding with retry**: `buildMonthSeed` hashes the target
  date, attempt index, and a stable serialization of the current staff
  list, so any given (year, month, staff, attempt) tuple produces exactly
  one schedule.

### Changed
- **Internal field naming normalized to camelCase** across staff objects
  and chained state. Staff objects now use `offDays` and `nightStaff`;
  chained state uses `lastShifts`, `consecutiveDays`, `dutyCounts`, and
  `offDayDebt`. Legacy snake_case keys (`off_days`, `night_staff`,
  `last_shifts`, `consecutive_days`, `duty_counts`, `off_day_debt`) are
  accepted on read and migrated on the next write, so existing
  `localStorage` and `.scsg` backups continue to load without loss.
- **Removed two dead bookkeeping fields**: `carryover_streaks` (a
  redundant copy of `consecutiveDays`) and `off_history` (written by the
  off-day planner but never read by any consumer). Both are dropped from
  every write path; `carryover_streaks` and its variants are still read
  by the chain-migration normalizer so pre-1.1.0 entries keep loading.
- **Responsive mobile layout**: control panels stack cleanly on smaller
  screens while preserving side-by-side date selection.
- **Invariant summary surface**: each generation records the applied
  repairs and residual violations, exposed on the result `meta` for
  callers that want to inspect the outcome.

### Design decisions
- **Cross-month chaining is silent**: chained state lives outside the
  configuration backup as a derived artifact keyed off the export
  filename. The UI never surfaces whether a predecessor was read, so the
  observed behaviour of a first-time generation and a mid-chain
  generation are indistinguishable to a user.
- **Forward-invalidation on store**: writing a month deletes every
  stored state entry strictly after it for the same clinic and shift
  system. Cumulative chaining means each stored entry is a function of
  all months before it; rewriting an earlier month therefore invalidates
  every later entry, and clearing them prevents a later month from
  silently chaining onto a predecessor that no longer matches what it
  was built on.
- **Exponential duty-count decay**: inherited `dutyCounts` are
  multiplied by `DUTY_COUNT_DECAY = 0.5` on read. A hard N-month window
  was rejected for creating an ordering cliff; the decay is the chosen
  approach. Stored entries keep reflecting what was computed — the decay
  is a view applied on read, so tuning the constant later does not
  corrupt existing chains.
- **Linear off-day debt amortization**: outstanding `offDayDebt` is
  repaid by `OFF_DAY_DEBT_REPAYMENT = 1` per generation, after the phase
  shift has been consumed and before this month's incursions land. Off-day
  debt perturbs the weekly rotation phase rather than adding an extra
  rest day, and the coverage floor cannot deliver true compensation
  without being violated. A burst of N lost off-days therefore fades over
  roughly N months.
- **camelCase-only writes**: after the migration pass, every stored
  config, `.scsg` export, and `_scsg_state` entry emits camelCase keys
  exclusively. The dual-style readers in `sanitizeStaffMember` and
  `applyDutyCountDecay` exist only to absorb pre-migration data on its
  next load; they are not part of the write surface and disappear
  naturally as old entries are regenerated.
- **Determinism preserved across the rename**: `buildMonthSeed` hashes
  the *contents* of the off-day list, not the property name, so
  pre-migration and post-migration staff lists with identical data
  produce byte-identical seeds and byte-identical schedules.

### Known limitations
- **No undo** across regeneration: hitting Generate runs the scheduler
  from scratch; there is no "previous output" recall.

---

## [1.0.0] — 2026-10-29

Initial public release.

### Added
- Three staff categories (Nursing, Support, General) with role-specific
  shift rules.
- Automatic fair off-day rotation for nursing staff.
- Manual off-day configuration (max 3 days) for support and general staff.
- Automatic night-duty reinforcement when fewer than 2 nurses are on
  night duty.
- Deterministic simulation for the selected month from the baseline date
  (November 2026).
- **PDF** export — landscape A4, one page per month, colour-coded
  M / N / X cells with legend.
- **Excel (.xlsx)** export — OOXML, split into two day-blocks per sheet.
- **JSON** export / import of the complete staff configuration.
- Local persistence via `localStorage` (`clinicRosterState`).
- Responsive layout for desktop and mobile.
- Full inline JSDoc annotations and file-level documentation headers.