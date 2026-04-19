# DM Remission Dashboard Project

## Overview
Diabetes clinic patient visit dashboard for analyzing DM remission study data. Built as an offline-capable, interactive HTML dashboard with Thai language UI. Title: **DM Remission Dashboard**.

## Data Source
- **Excel file**: `data/DM_remiss_analyz2.xlsx` (current, correct source)
- **Main sheet**: `CollectVer`
- **Embedded dataset**: 223 visits / 54 patients (after excluding rows with blank VISIT). **HN (hospital number) is stripped** before embedding for PDPA/privacy reasons — patient lookup uses `NO` only.
- **Date range**: Feb 2568 - Mar 2569 (Thai Buddhist; Feb 2025 - Mar 2026 Gregorian)
- The legacy file `DM_remiss_analyz(1).xlsx` is **wrong** — do not use.

### Source data notes / known data-quality issues
- **Blank VISIT rows are excluded** during extraction. In the source, ~55 rows have the VISIT column blank (mostly first/only visits where the data-entry person forgot to write `1`); excluding them drops ~53 single-visit patients.
- **NO=75** is missing from the sequence (NO jumps 74 → 76); max NO = 108, but unique NOs in source = 107, and after blank-VISIT exclusion only 54 remain.
- A few stray rows have `NO=None` with isolated values (e.g. `VISIT=54`, `HN=' '`) — also skipped.
- **BMI in Excel is computed wrong** (`=10000/HT` instead of `BW*10000/HT^2`). Re-compute BMI in the extractor.

## Project Structure
```
D:\AI_tung2\
  index.html               # Main dashboard (self-contained with embedded JSON data)
  CLAUDE.md                # Project documentation
  request.txt              # Original requirements
  data/
    DM_remiss_analyz2.xlsx # Current source Excel
  DM_remiss_analyz(1).xlsx # Legacy/incorrect source (kept untracked, do not use)
  libs/
    chart.min.js           # Chart.js v4.4.7 (offline)
    chartjs-datalabels.min.js  # Chart.js datalabels plugin v2.2.0
    fonts/
      sarabun-*.ttf        # Sarabun Thai font (400/500/600/700 weights)
```

## Dashboard Tabs (8)
1. **ภาพรวม (Overview)** — KPIs (incl. avg visit/patient, % Post Counseling), goal achievement, med-adjustment distribution, A1C distribution, monthly visit timeline, Post Counseling doughnut, visits-per-patient bar, **8 lab-trend line charts** (A1C/FBS/LDL/TG/SBP/DBP with goal lines; BMI/eGFR no goal line) showing avg per visit number
2. **ควบคุม DM** — A1C/FBS goals, DM med adjustments, A1C-vs-DMCONTROL chart, **Hypoglycemia/Hyperglycemia events** (moved here from old DRPs tab), summary table
3. **ควบคุม DLP** — LDL/TG goals, DLP med adjustments
4. **ควบคุม HT** — SBP/DBP goals, BP scatter plot
5. **เภสัชกรรม (Pharmacy)** — Polypharmacy, DM Remission, med counts + **DRPs charts merged in** (DRP type, Other DRP, DRP count distribution). Standalone DRPs tab no longer exists.
6. **ข้อมูลประชากร (Demographics)** — Sex (doughnut), **Age (pie, filter buckets)**, Scheme (pie), BMI, eGFR, visits-per-patient
7. **visit ที่ถึงเป้าหมาย (Visits to Goal)** — How many visits before achieving each clinical goal; stacked bar, achievement rate, avg visits, trend lines, per-patient detail table
8. **ค้นหาผู้ป่วย (Patient Lookup)** — Search by NO (1-108) only (HN removed for PDPA), visit history & trends

## Global Filters
Sex, Age range (`<40, 40-49, 50-59, 60-69, 70+`), Scheme, Visit number, Pharmacy (Polypharmacy ≥5 / ปกติ <5), DM Remission, **Post Counseling** (yes/no).

## Clinical Thresholds
| Metric | Goal | Related Control Column |
|--------|------|----------------------|
| A1C | **≤ 6.5** (inclusive) | DMCONTROL |
| FBS | < 120 | DMCONTROL |
| LDL | < 90 | DLPCONTROL |
| TG | < 150 | DLPCONTROL |
| SBP | < 130 | HTCONTROL |
| DBP | < 80 | HTCONTROL |

`computeVisitsToGoal(data, col, threshold, inclusive=false)` takes an `inclusive` flag — only A1C uses `inclusive: true`.

## Control Values (DMCONTROL / DLPCONTROL / HTCONTROL)
- 0 = ไม่ปรับ (No change)
- 1 = ลดยา (Decrease)
- 2 = เพิ่มยา (Increase)
- 3 = เปลี่ยนยา (Switch)

## Key Definitions
- **Polypharmacy**: `ALLMED_COUNT >= 5` (was `> 5` before)
- **DM Remission**: `DMMED_COUNT = 0`
- **POSTCOUNS**: 1 = had post-counseling at that visit, 0 = none
- **HYPOGLYCEMIA / HYPERGLYCEMIA**: 1 = event happened at that visit
- **DRPS_TYPE**: DRP(s) from diabetes medications (comma-separated)
- **OTHERDRPS**: DRP(s) from non-diabetes medications

## Language
- UI is in **Thai** (font: Sarabun)
- Medical/professional terms kept in **English** (A1C, FBS, DRP, Polypharmacy, etc.)
- The word "การเยี่ยม"/"ครั้งที่" was replaced with English **"visit"** throughout (e.g., `visit 1`, `5 visit`, `visit เฉลี่ย/ผู้ป่วย`) — preserve this convention in any new UI.
- Header subtitle: `วิเคราะห์ข้อมูลการ visit ของผู้ป่วย — Feb 2568 - Mar 2569`

## Hosting
- **GitHub repo**: https://github.com/Aunggrid/dm-dashboard-tung (public)
- **Live site**: https://aunggrid.github.io/dm-dashboard-tung/
- Deployed via GitHub Pages (legacy mode, serves from `master` branch root)
- **Privacy:** repo is public on the free tier (GitHub Pages requires it). Embedded data must therefore stay PDPA-safe — no HN, no names, no addresses. Patient identifiers in the embedded JSON are limited to the `NO` column (1-108, opaque sequence number).
- Git history was rewritten with `git filter-repo` on 2026-04-19 to scrub HN values from earlier commits (force-pushed). GitHub may still serve unreachable old commits by direct SHA for ~90 days before garbage collection.

## Tech Stack
- Pure HTML/CSS/JS (no build tools)
- Chart.js + datalabels plugin
- Data embedded as JSON in HTML (single `RAW_DATA` constant near top of `<script>`)
- Fully offline-capable (also hosted on GitHub Pages)

## How to Rebuild Data
If the Excel changes, re-extract via Python (openpyxl):
1. Read `CollectVer` sheet of `data/DM_remiss_analyz2.xlsx` with `data_only=True`
2. **Skip rows where `NO` is blank** (stray cells)
3. **Skip rows where `VISIT` is blank** (data-entry inconsistency)
4. **Recompute BMI** as `BW * 10000 / HT^2` (the Excel column is wrong)
5. Convert SEX from string ('1'/'0') to int; format VISIT_DATE as `YYYY-MM-DD`
6. **Do NOT include the `HN` column** in the JSON (PDPA — patient identifier). Also skip `NAME`. Only `NO` is allowed as a patient identifier.
7. Serialize to JSON and replace the single-line `const RAW_DATA = [...];` near the top of the `<script>` block in `index.html`
8. After changes: `git add index.html && git commit && git push` to update the live site

### If HN/PII leaks into a commit by mistake
The repo is public, so even one commit containing HN must be scrubbed from history, not just reverted.
1. `pip install git-filter-repo`
2. Create a replace-text spec: `echo 'regex:"HN": \d+==>"HN": null' > _hn_replace.txt`
3. `git filter-repo --replace-text _hn_replace.txt --force`
4. Re-add origin (filter-repo strips it as a safety): `git remote add origin https://github.com/Aunggrid/dm-dashboard-tung.git`
5. Force-push: `git push --force origin master`
6. Anyone with another clone must `git fetch && git reset --hard origin/master` — normal pull will fail.
