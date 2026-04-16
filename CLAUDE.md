# DM Clinic Dashboard Project

## Overview
Diabetes clinic patient visit dashboard for analyzing DM remission study data. Built as an offline-capable, interactive HTML dashboard with Thai language UI.

## Data Source
- **Excel file**: `DM_remiss_analyz(1).xlsx`
- **Main sheet**: `CollectVer` (278 visits, 106 patients)
- **Date range**: Feb 2025 - Mar 2026 (Thai Buddhist calendar in original)

## Project Structure
```
D:\AI_tung2\
  dashboard.html          # Main dashboard (self-contained with embedded JSON data)
  DM_remiss_analyz(1).xlsx # Source Excel data
  request.txt             # Original requirements
  libs/
    chart.min.js          # Chart.js v4.4.7 (offline)
    chartjs-datalabels.min.js  # Chart.js datalabels plugin v2.2.0
    fonts/
      sarabun-*.ttf       # Sarabun Thai font (400/500/600/700 weights)
```

## Dashboard Tabs
1. **ภาพรวม (Overview)** - KPIs, goal achievement, A1C distribution, timeline
2. **ควบคุม DM** - A1C/FBS goals, medication adjustments, summary table
3. **ควบคุม DLP** - LDL/TG goals, medication adjustments
4. **ควบคุม HT** - SBP/DBP goals, BP scatter plot
5. **เภสัชกรรม (Pharmacy)** - Polypharmacy, DM remission, med counts
6. **DRPs** - Drug-related problems, hypoglycemia/hyperglycemia
7. **ข้อมูลประชากร (Demographics)** - Sex, age, scheme, BMI, eGFR
8. **ค้นหาผู้ป่วย (Patient Lookup)** - Search by NO/HN, visit history & trends

## Clinical Thresholds
| Metric | Goal | Related Control Column |
|--------|------|----------------------|
| A1C | < 6.5 | DMCONTROL |
| FBS | < 120 | DMCONTROL |
| LDL | < 90 | DLPCONTROL |
| TG | < 150 | DLPCONTROL |
| SBP | < 130 | HTCONTROL |
| DBP | < 80 | HTCONTROL |

## Control Values (DMCONTROL / DLPCONTROL / HTCONTROL)
- 0 = ไม่ปรับ (No change)
- 1 = ลดยา (Decrease)
- 2 = เพิ่มยา (Increase)
- 3 = เปลี่ยนยา (Switch)

## Key Definitions
- **Polypharmacy**: ALLMED_COUNT > 5
- **DM Remission**: DMMED_COUNT = 0
- **DRPS_TYPE**: DRP from diabetes medications
- **OTHERDRPS**: DRP from non-diabetes medications

## Language
- UI is in **Thai** (font: Sarabun)
- Medical/professional terms kept in **English** (A1C, FBS, DRP, Polypharmacy, etc.)

## Tech Stack
- Pure HTML/CSS/JS (no build tools)
- Chart.js + datalabels plugin
- Data embedded as JSON in HTML
- Fully offline-capable

## How to Rebuild Data
If the Excel changes, re-extract data using Python (openpyxl):
1. Read `CollectVer` sheet with `data_only=True`
2. Parse Thai Buddhist dates from PREDATE column (year - 543)
3. Compute: VISIT (count per patient), SEX (from column H), BMI (BW*10000/HT^2)
4. Export to JSON and embed in `dashboard.html` replacing `RAW_DATA`
