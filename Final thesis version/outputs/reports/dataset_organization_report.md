# Dataset Organization Report

Generated manually (not by a notebook) while organizing the raw thesis datasets
downloaded as ZIP/GZ archives into
`Desktop/Project/` (the parent folder, outside the project itself).

## 1. ZIP/Archive Files Found

Located in `C:\Users\khamidov.m\OneDrive - Procter and Gamble\Desktop\Project\`:

| File | Size | Identified as |
|---|---|---|
| `ESS10e03_3.zip` | ~60 MB | ESS Round 10 integrated file (edition 3.3) |
| `ESS11e04_2.zip` | ~83 MB | ESS Round 11 integrated file (edition 4.2) |
| `ESS11MD_e01_2.zip` | ~1.7 GB (uncompressed CSV inside) | ESS11 Multilevel Data (edition 1.2) |
| `estat_isoc_eb_ai$defaultview_filtered.tsv.gz` | ~8 KB | Eurostat table `isoc_eb_ai` — enterprises using AI technologies |

All four archives were inspected (file listing / content preview) before extracting
anything. **No original archive was deleted or modified** — they remain in place in
`Desktop/Project/`.

## 2. Archive Contents Inspected

| Archive | Contents |
|---|---|
| `ESS10e03_3.zip` | `ESS10e03_3.csv` (data), `ESS10e03_3 codebook.html` |
| `ESS11e04_2.zip` | `ESS11e04_2.csv` (data), `ESS11e04_2 codebook.html` |
| `ESS11MD_e01_2.zip` | `ESS11MD_e01_2.csv` (data), `ESS11MD_e01_2 codebook.html` |
| `estat_isoc_eb_ai$defaultview_filtered.tsv.gz` | Single tab-separated file, wide format: dimension columns (`freq`, `size_emp`, `nace_r2`, `indic_is`, `unit`, `geo\TIME_PERIOD`) + one column per year (2021, 2023, 2024, 2025) |

**Important finding**: neither the ESS10 nor the ESS11 ZIP contains an SPSS
`.sav` file — only CSV + an HTML codebook. See "Missing / Unclear Items" below.

## 3. Files Extracted / Converted

- `ESS11e04_2.csv` extracted from `ESS11e04_2.zip` (ESS11 preferred over ESS10 per
  project convention — main survey dataset used, since ESS11 is the more recent round).
- `ESS10e03_3.zip` was **inspected but not extracted**, since ESS11 is available and
  preferred. It is left untouched in `Desktop/Project/` in case ESS11 turns out to be
  unsuitable later.
- `ESS11MD_e01_2.csv` extracted from `ESS11MD_e01_2.zip` (ESS Multilevel data).
- `estat_isoc_eb_ai$defaultview_filtered.tsv.gz` decompressed and reformatted from
  tab-separated to comma-separated (dimension columns split out, one column per
  year), preserving all original values including Eurostat's missing/confidential
  data flags (see below).

## 4. Files Moved / Copied Into the Project

| Source | Destination | Renamed to |
|---|---|---|
| `ESS11e04_2.csv` (from ZIP) | `data/raw/ess/` | `ESS11.csv` |
| `ESS11MD_e01_2.csv` (from ZIP) | `data/raw/ess_multilevel/` | `ESS_multilevel_data.csv` |
| `estat_isoc_eb_ai$defaultview_filtered.tsv.gz` (decompressed + reformatted) | `data/raw/eurostat_ai/` | `isoc_eb_ai.csv` |

No existing files were overwritten — each destination folder only contained a
`.gitkeep` placeholder before this operation, so no backup/overwrite conflict arose.

## 5. Final File Paths

- `employment_ai_europe_thesis/data/raw/ess/ESS11.csv`
- `employment_ai_europe_thesis/data/raw/ess_multilevel/ESS_multilevel_data.csv`
- `employment_ai_europe_thesis/data/raw/eurostat_ai/isoc_eb_ai.csv`

## 6. Main ESS Dataset Decision

**ESS11 will be used as the main ESS dataset** (preferred round per project
convention). `ESS10e03_3.zip` remains available, unextracted, in
`Desktop/Project/` as a fallback if ESS11 proves unusable for any reason.

## 7. Missing or Unclear Items (needs attention before running notebooks)

1. **Format mismatch — ESS main data is CSV, not SPSS `.sav`.**
   Notebooks 01–03 currently look for `ESS11.sav` / `ESS10.sav` and load them with
   `pyreadstat.read_sav()`. Only a CSV version was available in the downloaded ZIP
   (`ESS11e04_2.csv`), saved here as `ESS11.csv` — it was **not** renamed to `.sav`,
   since simply changing the file extension would not make it a valid SPSS file and
   `pyreadstat` would fail to read it. **This notebook loading logic was intentionally
   left unchanged in this task** ("do not modify notebooks yet"), so a follow-up task
   will be needed to either (a) download the SPSS `.sav` version from the ESS Data
   Portal, or (b) update the notebook loading logic to also accept `ESS11.csv` /
   `ESS10.csv` via `pandas.read_csv()`.
2. **ESS Multilevel data**: extracted as `ESS_multilevel_data.csv`, which **does**
   match one of the two filenames notebook 01/03 already accept — no action needed.
3. **Eurostat AI data**: saved as `isoc_eb_ai.csv`, matching the expected filename —
   no action needed. Note that the raw values include Eurostat's own missing/
   confidential-data flags (e.g. `: @C` meaning "not available, confidential") for
   some countries (e.g. Montenegro) and years — these are genuine values from the
   source, not introduced during conversion, and will need to be handled as missing
   data when this file is parsed numerically (left for the relevant notebook to
   handle, not addressed in this organization-only task).
4. **Codebook HTML files** (`ESS10e03_3 codebook.html`, `ESS11e04_2 codebook.html`,
   `ESS11MD_e01_2 codebook.html`) were left inside their original ZIP files and were
   **not** extracted into the project, since notebook 01 only checks for the data
   file itself. Extract them manually into `data/raw/ess/` (or elsewhere convenient)
   if you want to consult them directly while verifying the `# TODO: verify against
   codebook` variable mappings in notebooks 02–03.

## Scope of This Task

Only dataset organization was performed: inspecting archives, extracting/converting
the correct files, renaming them to match notebook expectations where possible, and
reporting the outcome. **No notebook was modified, no EDA was run, and no data
engineering was performed** in this task.
