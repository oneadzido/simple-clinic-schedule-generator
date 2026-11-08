# SCSG — Simple Clinic Schedule Generator

SCSG is a single-page, offline-first application for generating monthly clinic staff schedules. It runs in a modern browser without installation, a build step, a backend, or network access.

## Project information

- **Author:** Richard Korbla Adzido
- **Scheduler source version:** 1.2.0 (`assets/scsg.js`)
- **License:** MIT; see [`LICENSE`](LICENSE).
- **Scheduling baseline:** November 2026. Earlier target months are not available.

The HTML document's `<meta name="version">` supplies the version shown in the application footer. Keep it aligned with the release version.

## Privacy

The application processes roster information locally. It does not use a server, analytics, tracking pixels, cookies, or third-party scripts. The clinic name and staff details are entered by the user; the default roster contains generic placeholders only.

See [`PRIVACY.md`](PRIVACY.md) for the data inventory, browser storage behavior, and backup lifecycle.

## Capabilities at a glance

- **Offline operation** — scheduling and document generation run entirely in the browser.
- **Configurable staffing** — nursing, support, and general staff roles with role-specific rules.
- **Two shift systems** — 12-hour (Morning / Night) and 8-hour (Morning / Afternoon / Night).
- **Support-assisted rosters** — a small nursing team plus one or more support workers is a first-class configuration, not a degraded mode.
- **Local persistence** — the roster and settings are stored in the browser.
- **Portable configuration** — export and restore a `.scsg` configuration backup.

## Run the application

1. Extract the project archive while preserving its folder structure.
2. Open `scsg.html` in a modern browser such as Chrome, Edge, Firefox, or Safari.
3. Configure the clinic, target month, shift system, and staff roster.
4. Generate the schedule and choose PDF or Excel output.

No package installation or compilation is required. The primary runtime files are `scsg.html`, `assets/scsg.css`, and `assets/scsg.js`.

### Run the built-in self-test

Open the page with `?selftest=1` appended to the URL:

```text
file:///path/to/simple-clinic-schedule-generator/scsg.html?selftest=1
```

This is a lightweight runtime check that logs initialization and completion to the browser console. It asserts that `applicationState.staff` is an array and that `applicationState.shiftSystem` is a string. It does **not** exercise schedule generation.

### Run the regression suite

Requires Node.js 18 or later. Run from the project root:

```bash
node --check assets/scsg.js
node --test tests/scheduling-regression.test.js
```

See [`tests/README.md`](tests/README.md) for what the suite covers and what it does not.

## Project layout

```text
simple-clinic-schedule-generator/
├── scsg.html                      Application shell and interface markup
├── assets/
│   ├── scsg.css                   Responsive layout and visual styling
│   ├── scsg.js                    Scheduling engine, validation, state, and exports
├── tests/
│   ├── README.md                  Test suite scope, coverage, and gaps
│   └── scheduling-regression.test.js
│                                  Node.js regression suite (scheduler + validator)
├── docs/
│   └── CHANGELOG.md               Release history
├── ALGORITHM_AUDIT.md             Algorithm audit and defect record (1.2.0)
├── backups/                       Reserved location for local project backups
├── .github/workflows/             Repository workflow configuration
├── LICENSE                        MIT license text
├── PRIVACY.md                     Privacy and local data handling policy
└── README.md                      Project overview and usage guide
```

## Configuration and persistence

The application saves its current state under the `clinicRosterState` key in browser `localStorage`. Continuity between months is stored separately as derived per-month entries; those are **not** included in a configuration export and regenerate automatically the next time the corresponding month is generated.

- **Configuration export** produces a `.scsg` JSON file containing the current clinic name, shift system, target month/year, output format, per-role off-day limits, and staff list. The payload is wrapped in `format`, `version`, and `exportedAt` metadata.
- **Configuration import** accepts `.scsg` and `.json` files. Legacy snake_case field names are normalized to camelCase on import.
- **`?selftest=1`** on the URL runs the small runtime check described above.

### Creating and restoring a backup

1. Select **Export Configuration**. The browser downloads a clinic- and shift-specific `.scsg` file.
2. Store the file somewhere appropriate for your clinic's data-handling practices.
3. To restore it, select **Import Configuration** and choose the saved file.

Browser site-data cleanup, a different browser profile, or opening the page under a different origin may make the saved configuration unavailable. Exporting is the only way to keep a portable copy.

## Staff tiers

`classifyScheduleTier()` labels the roster before generation begins. The label determines whether the scheduler proceeds and which rules apply:

| Tier | Condition | Scheduling behaviour |
|---|---|---|
| **Non-nursing** | No nurses configured | Uses Support + General staff only. INV6 (Sunday support Night) still applies unless Sunday is a configured off-day. |
| **Compliant** | ≥ 4 nurses (12h) or ≥ 5 nurses (8h) | Full nursing rules. |
| **Support-assisted** | ≥ 2 nurses + ≥ 1 support (12h) or ≥ 3 nurses + ≥ 1 support (8h) | Full nursing rules; support workers fill the second Night slot when fewer than two nurses are on Night. |
| **Understaffed** | Fewer nurses than the support-assisted minimum | Refused before generation begins. |

The nurse-count preflight is driven by the tier, **not** by a fixed number: a 2-nurse + 1-support roster (12h) is a valid support-assisted configuration and will generate. Only rosters that `classifyScheduleTier()` labels `understaffed` are refused.

## Scheduling invariants

All implemented rules are numbered contiguously **INV1–INV7**. Every invariant is export-blocking: if any remains unresolved after the retry and repair budget is exhausted, the generator aborts rather than downloading an unsafe schedule. The in-app **Scheduling Invariants** panel is the primary reference for the current build; the table below is the high-level summary.

| ID | Rule |
|---|---|
| **INV1 — Night coverage** | 12-hour: at least 2 total Night staff, with at least 1 nurse when Support or General also work Night. 8-hour: at least 1 nurse on Night. Without nurses, at least 2 staff must be on duty each day. |
| **INV2 — Day coverage** | At least 1 nurse on Morning each day; the 8-hour system additionally requires at least 1 nurse on Afternoon. |
| **INV3 — Morning before off-day** | Every in-month nursing off-day must be preceded by a Morning shift. |
| **INV4 — General night eligibility** | General staff with Night eligibility disabled cannot be assigned Night. |
| **INV5 — Off-day count** | Each nurse receives exactly the configured number of off-days in every complete Monday–Sunday week contained within the month. |
| **INV6 — Sunday support Night** | Support staff work Night on Sundays unless Sunday is their configured off-day. |
| **INV7 — Sunday Morning fairness** | Cumulative Sunday Morning assignments are balanced within a tolerance of one among nurses who work at least one Sunday. |

## How generation works

`generateAndDownloadSchedule()` performs the following, in order:

1. **Readiness check** — rejects an empty roster or blank staff names.
2. **Tier classification** — classifies the roster as `compliant`, `support-assisted`, `understaffed`, or `non-nursing`, and names the schedule accordingly.
3. **Tier gate** — refuses only when the tier is `understaffed`. Support-assisted rosters proceed.
4. **Capacity preflight** — `checkOffDayCapacity` simulates the weekly off-day plan across every weekday phase and rejects configurations that cannot sustain minimum nurse coverage or the required Morning-before-off slots.
5. **Seeded generation, up to 50 attempts** — each attempt uses a seed derived from year, month, attempt index, and the staff list. Both the standard nursing scheduler and the non-nursing scheduler are deterministic given a seed.
6. **Automatic repair** — `autoResolveInvariants` iterates up to 500 passes and calls the repair functions for the invariants that have a repair path:
   - `applyInv1Repairs` — raise the night count when the floor is short. Counts nurses, support, and Night-eligible General staff together in the 12-hour system; nurses only in the 8-hour system.
   - `applyInv2Repairs` — restore Morning coverage, and in 8-hour mode also restore Afternoon coverage.
   - `applyInv3Repairs` — swap or downgrade a shift when a nursing off-day is not preceded by Morning.
   - `applyInv4Repairs` — demote Night to Morning for General staff who are not night-eligible.
   - `applySupportSundayNightRepairs` — enforce INV6 unless Sunday is a configured off-day.
   - `applyInv7Repairs` — rebalance Sunday Morning assignments toward the least-served nurse.
7. **Emergency floor enforcement** — if INV1 still fails after the repair passes, `enforceEmergencyFloorCoverage` attempts to raise the night count directly using the same eligibility rule as the repair step.
8. **Residual check** — `resolveRemainingViolations` is a last-ditch pass that tries any eligible staff member for the failing shift.
9. **Export gate** — export only if `meta.residualViolations.length === 0`. Otherwise the run reports the offending days and invariant names.
10. **State store** — the outgoing month is persisted for continuity, and forward-month cached state is invalidated.

There is **no dedicated repair function for INV5**; the off-day count is a structural property of the weekly off-day planner, not something a post-hoc repair can fix. A roster that violates INV5 will therefore abort after the retry budget, which is the intended behaviour.

## Export formats

Both formats group staff in sets of **12 per PDF page or Excel worksheet**, each carrying the full title block and legend. Multi-page and multi-sheet outputs add a `(Page N of M)` / `(Sheet N of M)` suffix to the month line.

- **PDF** — landscape A4, colour-coded shift cells, horizontal legend. Page tree and cross-reference table are compiled after all staff groups render, so the file is valid for any page count.
- **Excel (`.xlsx`)** — OOXML workbook with dynamically constructed `[Content_Types].xml`, `xl/workbook.xml`, and workbook relationships, so the file stays valid for any number of worksheets. Worksheet names are `Roster` (single) or `Roster 1`, `Roster 2`, … (multiple).

Output filenames combine the sanitized clinic name, the shift-system tag (`8h` or `12h`), the month, and the year.

## Current limitations

- **One month at a time.** Continuity can be inherited from the previous month if a corresponding state entry exists, but only one schedule is generated per run.
- **Continuity is silent.** The UI does not indicate whether a previous month's state was found or used.
- **No undo.** Hitting Generate re-runs the scheduler from scratch; there is no "previous output" recall.
- **Self-test scope.** The built-in runtime self-test checks selected state assertions only. The Node regression suite exercises both supplied default rosters across 122 consecutive months but does **not** cover the PDF/XLSX exporters, the UI, or non-default rosters. See [`tests/README.md`](tests/README.md).
- **General Night in 8-hour mode.** General staff are never assigned Night in the 8-hour system; the eligibility checkbox is shown disabled in that mode. The stored preference is preserved for a 12-hour switch and is surfaced honestly: if it is set, the disabled checkbox renders checked with the tooltip explaining why Night is not assigned.
- **Consecutive-workday policy.** INV5 requires exactly one off-day per nurse in each complete Monday–Sunday week (with the default limit). Rotating that off-day's weekday to share weekday/weekend rest allows up to **seven consecutive working days**. A hard five-day maximum is incompatible with the exact-weekly-count rule and would require additional rest days or a revised INV5.

## Development notes

- **Determinism.** The scheduler uses a seeded Mulberry32 PRNG and an FNV-1a hash to derive repeatable attempt seeds. Identical inputs and seeds produce identical schedules.
- **Naming.** Staff and continuity fields use camelCase as the canonical representation. Legacy snake_case keys are accepted on read and migrated on the next write.
- **Diagnostics.** Each generation records repairs applied, repair pass count, and residual violations on the result `meta`. The in-app debug display is disabled by default.
- **Keeping docs in sync.** When changing scheduling behavior, update the implementation comments, the in-app invariant descriptions, [`tests/README.md`](tests/README.md), [`ALGORITHM_AUDIT.md`](ALGORITHM_AUDIT.md), this README, and the release history together.