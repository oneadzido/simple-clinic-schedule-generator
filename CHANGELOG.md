# Changelog

All notable changes to **SCSG — Simple Clinic Schedule Generator**.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

**Author:** Richard Korbla Adzido

---

## [1.0.0] — 2026-10-30

### Added
- Initial public release
- Three staff categories (Nursing, Support, General) with role-specific shift rules
- Automatic fair off-day rotation for nursing staff
- Manual off-day configuration (max 3 days) for support and general staff
- Automatic night-duty reinforcement when fewer than 2 nurses are on night duty
- Night-shift continuity guard across month boundaries
- Sunday special-case for morning-assignment fairness
- Month-by-month deterministic simulation from the baseline date (November 2026)
- **PDF** export — landscape A4, one page per month, colour-coded M / N / X cells with legend
- **Excel (.xlsx)** export — minimal OOXML writer, split into two day-blocks per sheet
- **JSON** export / import of the complete staff configuration
- Local persistence via `localStorage` (`clinicRosterState`)
- Responsive layout for desktop and mobile
- Full inline JSDoc annotations and file-level documentation headers

### Privacy
- No real clinic name, location, or staff name is embedded in the source
- Default staff roster uses generic placeholders (`Nurse A`, `Support A`, `Staff A`, …)
- `localStorage` key and download filenames are neutral

### Licensing
- Source code released under the **MIT License** (`LICENSE`)

### Documentation
- `README.md` — setup, customisation, backup, privacy
- `PRIVACY.md` — privacy policy
- `docs/CHANGELOG.md` — this file
- `.editorconfig`, `.gitignore` — project hygiene