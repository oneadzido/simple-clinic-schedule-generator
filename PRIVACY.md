# Privacy Policy

**SCSG — Simple Clinic Schedule Generator**  
**Author:** Richard Korbla Adzido  
**Version:** 1.1.0  
**Last updated:** 2026-10-31  

---

## Summary

SCSG is a **fully offline, client-side application**. It does not collect,
transmit, or store any data on any server, anywhere, ever. No real clinic
name, location, or individual staff name is embedded in this source code.

---

## Contents

1. [What data the app handles](#1-what-data-the-app-handles)
2. [What the app does not do](#2-what-the-app-does-not-do)
3. [How your data is stored](#3-how-your-data-is-stored)
4. [Default staff roster](#4-default-staff-roster)
5. [Data lifecycle](#5-data-lifecycle)
6. [Contact](#6-contact)

## 1. What data the app handles

| Data | Where it lives | Who can see it |
|---|---|---|
| Staff names, roles, off-days, night toggle | Your browser's `localStorage` | Only you, on your device |
| Target month / year, output format, clinic name | Your browser's `localStorage` | Only you, on your device |
| Cross-month scheduling continuity state (last shift, consecutive working days, duty counters, off-day debt) | Your browser's `localStorage` | Only you, on your device |
| Generated PDF / XLSX schedules | Wherever you save the download | Only you |
| Exported JSON backup | Wherever you save the download | Only you |

No data is sent outside the browser at any point. All computation —
scheduling, invariant repair, PDF/XLSX rendering, ZIP packaging — happens
client-side using standard browser APIs (`TextEncoder`, `Blob`,
`URL.createObjectURL`).

## 2. What the app does **not** do

- No server, no backend, no API calls
- No analytics, no telemetry, no tracking pixels
- No cookies
- No third-party scripts, fonts, or CDNs
- No error reporting to the author
- No advertising
- No account system, no login, no email collection
- No data sharing of any kind

## 3. How your data is stored

All configuration is persisted in your browser's `localStorage`:

- **`clinicRosterState`** — the current clinic name, shift system, target
  month/year, output format, and the full staff list (names, roles,
  off-days, night-eligibility).
- **`<schedule>_scsg_state`** — derived per-month scheduling continuity
  state, one entry per successfully generated schedule. Each entry is
  keyed to a specific clinic + shift system + month (e.g.
  `simple_clinic_12h_schedule_december_2026_scsg_state`) and carries only
  the shift bookkeeping the scheduler needs to chain into the following
  month — last shift per staff member, consecutive working days, duty
  counters, and outstanding off-day debt. It contains **no data beyond
  what you have already entered**; it is a summary of what the scheduler
  computed from that data.

Both key families are scoped to your browser profile and page origin. You
can delete them at any time by clearing your browser's site data or using
the app's internal reset buttons.

> ⚠️ Clearing browser history/data will wipe your configuration. Use
> **Export Configuration** regularly to keep a JSON backup.

The exported `.scsg` backup is a plain JSON file containing exactly the
state that lives under `clinicRosterState` — nothing else. Chained
`_scsg_state` entries are a derived artifact and are intentionally **not**
part of the backup; they regenerate naturally the next time you generate
the corresponding month.

No scheduling bookkeeping is retained beyond the fields listed above. In
particular, the app does **not** accumulate an off-day history log or a
consecutive-days streak log — earlier releases tracked these internally but
they were removed in 1.1.0 because nothing read them. Neither was ever
exported.

## 4. Default staff roster

The default staff members are **generic placeholders** — they do not
correspond to any real person and allow first-time users to test schedule
generation immediately.

- **12-hour template:** `Nurse A`, `Nurse B`, `Nurse C`, `Support A`,
  `Staff A`, `Staff B`, `Staff C`, `Staff D`.
- **8-hour template:** the same plus `Nurse D` and `Nurse E`.

When you clear the application state and reopen the app, these placeholders
are the only staff entries that appear. No real names are ever loaded
automatically.

## 5. Data lifecycle

1. **Creation** — you enter staff names, roles, off-days, and configuration
   via the UI. Each change is written to `localStorage` immediately.
2. **Persistence** — state survives page reloads and browser restarts,
   but is scoped to the browser profile and origin of the page. When you
   generate a schedule, the app also writes one `_scsg_state` entry for
   that month so the next month can chain from it; writing a month
   deletes every stored entry strictly after it for the same clinic and
   shift system.
3. **Export** — you may download the current configuration as a `.scsg`
   file at any time via **Export Configuration**. The file stays on your
   device.
4. **Deletion** — click **Remove All Staff** and clear the clinic name,
   or clear the browser's site data, to reset. There is no "server-side"
   copy to delete because none exists.

## 6. Contact

Have questions about this policy? Contact the developer: **Richard Korbla Adzido** (Email: oneadzido@gmail.com).