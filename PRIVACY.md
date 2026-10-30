# Privacy Policy

**SCSG — Simple Clinic Schedule Generator**
**Author:** Richard Korbla Adzido
**Version:** 1.0.0
**Last updated:** 2026-10-30

---

## Summary

SCSG is a **fully offline, client-side application**. It does not collect,
transmit, or store any data on any server, anywhere, ever. No real clinic
name, location, or individual staff name is embedded in this source code.

---

## 1. What data the app handles

| Data | Where it lives | Who can see it |
|---|---|---|
| Staff names, roles, off-days, night toggle | Your browser's `localStorage` | Only you, on your device |
| Target month / year, output format | Your browser's `localStorage` | Only you, on your device |
| Generated PDF / XLSX schedules | Wherever you save the download | Only you |
| Exported JSON backup | Wherever you save the download | Only you |

## 2. What the app does **not** do

- ❌ No server, no backend, no API calls
- ❌ No analytics, no telemetry, no tracking pixels
- ❌ No cookies
- ❌ No third-party scripts, fonts, or CDNs
- ❌ No error reporting to the author
- ❌ No advertising
- ❌ No account system, no login, no email collection
- ❌ No data sharing of any kind

## 3. How your data is stored

All configuration is persisted in your browser's `localStorage` under the
key **`clinicRosterState`**. You can delete it at any time by:

- Clearing your browser's site data / local storage for this page, or
- Using the app's **Remove All Staff** and **Restore Default Staff** buttons

> ⚠️ Clearing browser history/data will wipe your configuration. Use
> **Export Staff Configuration** regularly to keep a JSON backup.

## 4. Default staff roster

The eight default staff members (`Nurse A`, `Nurse B`, `Nurse C`,
`Support A`, `Staff A`, `Staff B`, `Staff C`, `Staff D`) are **generic
placeholders**. They do not correspond to any real person. They exist so
that a first-time user can click **Generate Schedule** immediately and
see what the output looks like.

Replace them with your own clinic's staff using the **Add New Staff**
and **Remove** buttons, or by editing `DEFAULT_STAFF_ROSTER` in
`assets/scsg.js`.

## 5. Clinic name

The app displays a generic **"CLINIC"** heading. No real clinic name or
location is embedded in this source code. If you want your clinic's name
to appear in the UI and in generated PDF/XLSX files, see the
"Customising for your clinic" section in `README.md`.

## 6. Data export and import

The **Export Staff Configuration** button produces a plain JSON file on
your device. The **Import Staff Configuration** button reads a JSON file
*you* select. Neither operation contacts any external service — the file
never leaves your computer.

## 7. Children's data

The app is not directed at children and collects no data of any kind.

## 8. Changes to this policy

If this policy ever changes, the "Last updated" line above will change
and the new text will ship with the next release.

## 9. Contact

Have questions about this policy?
Contact the developer: **Richard Korbla Adzido**. Email: oneadzido@gmail.com

---

## Relationship to the LICENSE

| Content | Governed by | Permitted use |
|---|---|---|
| Source code (`scsg.html`, `assets/scsg.css`, `assets/scsg.js`) | **MIT License** | Use, copy, modify, merge, publish, distribute, sublicense, sell — with attribution |
| Default staff roster (`Nurse A`, `Support A`, …) | **This policy (§4)** | Generic placeholders; no personal data |
| Your own entered staff data | **You, entirely** | Never leaves your device |

No personal data is embedded in this project.

---

*End of Privacy Policy.*