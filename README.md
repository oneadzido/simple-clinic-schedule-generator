# SCSG — Simple Clinic Schedule Generator

A single-page, offline-first staff scheduling tool for any clinic.

- No installation
- No build step
- No internet required
- Runs entirely in the browser
- **No personal data baked in** — you enter your own staff

## Author

**Richard Korbla Adzido**

## Version

**1.0.0**

## License

Source code is released under the **MIT License** — see `LICENSE`.

## Privacy

This app is fully offline. It collects **no** data, sends **no** data,
uses **no** cookies, and calls **no** external services. All configuration
lives in your browser's `localStorage` under the key **`clinicRosterState`**.

No real clinic name, location, or staff name is embedded in this source
code. The eight default staff members are **generic placeholders**
(`Nurse A`, `Support A`, `Staff A`, …).

Full details: [`PRIVACY.md`](PRIVACY.md).

## How to run

### For clinic staff (end-users)

1. Open the Releases page of this repository and download
   `SCSG-Clinic-Tool-vX.Y.Z.zip`.
2. Extract the folder (right-click → Extract All).
3. Double-click **Run Me.html**.
4. No installation. No internet. Fully offline.

### For developers (forks)

1. Download `scsg-vX.Y.Z.zip` from the Releases page.
2. Extract it.
3. Open `scsg.html` in any modern browser (Chrome, Edge, Firefox, Safari).

No toolchain, no build step, no dependencies. The app is three plain
files: `scsg.html`, `assets/scsg.css`, `assets/scsg.js`.

## Repository layout

The public repository is intentionally minimal — **exactly two items**:

```
.
├── .github/
│   └── workflows/
│       └── main.yml                       # release pipeline
└── simple-clinic-schedule-generator.zip   # complete source, folder structure included
```

The entire project — code, docs, license, workflow — lives inside
`simple-clinic-schedule-generator.zip`. The workflow in `main.yml`
extracts that archive, verifies its contents, and publishes two release
assets:

| Release asset | Who it's for |
|---|---|
| `scsg-vX.Y.Z.zip` | **Developers & forks** — full source, docs, license, workflow |
| `SCSG-Clinic-Tool-vX.Y.Z.zip` | **Clinic staff** — extract, then double-click **Run Me.html** |

The project structure is kept inside the ZIP because the repository's
upload interface cannot create folders directly. The workflow handles
extraction, so every published release has a normal, unzipped folder
layout.

## Contents of the source archive

```
simple-clinic-schedule-generator/
├── scsg.html                    entry point
├── assets/
│   ├── scsg.css                 styles
│   └── scsg.js                  application logic
├── backups/                     exported JSON configurations (git-ignored)
├── docs/
│   └── CHANGELOG.md
├── .github/
│   └── workflows/
│       └── main.yml             (ships to forks)
├── README.md                    this file
├── LICENSE                      MIT
├── PRIVACY.md                   privacy policy
├── .gitignore
└── .editorconfig
```

## Customising for your clinic

The app displays a generic **"CLINIC"** heading. Two ways to make it yours:

**Option A — In-app, no code changes**

1. Enter your staff via **Add New Staff** and **Remove** as needed.
2. Your staff names appear in the generated PDF/XLSX output.

**Option B — Edit the source (permanent)**

- Open `scsg.html` and change the `<h1>CLINIC</h1>` line and the
  `<title>` tag to your clinic's name.
- Open `assets/scsg.js` and change `"CLINIC"` inside
  `generatePdfDocument()` and `generateExcelSpreadsheet()` to match.
- Optionally edit `DEFAULT_STAFF_ROSTER` to pre-load your team.

Do **not** commit real staff names to a public fork — keep them in
`backups/` (git-ignored) or in your local copy only.

## Features

- Three staff categories (Nursing, Support, General) with distinct shift rules
- Automatic fair off-day rotation for nursing staff
- Manual off-day configuration (max 3 days) for support and general staff
- Automatic night-duty reinforcement when nursing coverage is thin
- Month-by-month simulation from the baseline date (November 2026)
- **PDF** export — landscape A4, one page per month
- **Excel (.xlsx)** export — OOXML, split into two day-blocks per month
- **JSON** export / import of the full staff configuration
- Local persistence via `localStorage` under the key `clinicRosterState`
- Responsive layout for desktop and mobile

## Data & backups

The current configuration is saved automatically to your browser's
`localStorage`. **Clearing browser history/data will delete it.**

To preserve it:

1. Click **Export Staff Configuration** — downloads `clinic_staff_configuration.json`
2. Store that file in the `backups/` folder (git-ignored)
3. Restore it any time with **Import Staff Configuration**

## Releasing

The workflow in `.github/workflows/main.yml` handles the full release
pipeline: extraction, verification, both archive builds, artefact upload,
and release publishing. No local tooling is required.

1. On your machine, update `docs/CHANGELOG.md` and bump the version in
   `assets/scsg.js` if needed.
2. Zip the entire project folder as `simple-clinic-schedule-generator.zip`.
3. Upload that ZIP to the repository root, overwriting the previous one.
4. Commit and push to `main`.
5. Either:
   - **Tag:** `git tag v1.0.0 && git push origin v1.0.0`, or
   - **Manual:** *Actions → Release → Run workflow → enter `v1.0.0` → Run*.
6. Both release archives are attached to a new GitHub Release automatically.

## Contributing / forking

Please **do not commit real personal data** to a public fork. Keep staff
names, clinic names, and any identifiable information out of the source
tree — put them in `backups/` (git-ignored) or in a private local copy.