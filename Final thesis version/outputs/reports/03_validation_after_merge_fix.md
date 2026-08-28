# Notebook 03 Validation After Eurostat Merge Fix

**Validation date:** 2026-08-17  
**Status:** **VALIDATED**

## Validation Scope

This validation reviewed the current Notebook 03 code and saved outputs after the Eurostat merge correction. It read the processed CSV, the pickle dataset, the engineering report, the dataset notes, and the variable dictionary. The raw Eurostat file was read only to reproduce the filter and uniqueness checks. No raw data were modified. Notebooks 04, 05, 06, and 07 were not modified or run.

The corrected dataset is structurally safe to use as the input for downstream modelling. This conclusion concerns file consistency, merge cardinality, required-variable availability, and interaction construction. It does not remove the separate requirement to verify ESS variable codes against the official Round 11 codebook.

## 1. Final Modelling Dataset

- Processed CSV: `data/processed/final_employment_ai_dataset.csv`
- Pickle: `data/pickle/final_employment_ai_dataset.pkl`
- Final row count: **6,336**
- Final column count: **1,291**
- Countries represented: **30**
- Employment outcome values present: **0 and 1**
- CSV and pickle shapes: **identical, 6,336 × 1,291**
- CSV and pickle column names and order: **identical**
- Numeric content across the CSV and pickle: **equal within floating-point tolerance**
- Non-numeric content across the CSV and pickle: **equal**

The engineering row audit records 6,336 rows before the Eurostat merge, 6,336 rows after the Eurostat merge, 6,336 rows after the ESS Multilevel merge, and 6,336 rows in the final saved dataset. The previous expansion to 76,628 rows is therefore fixed.

## 2. Eurostat AI Merge

Notebook 03 applies the following filter to `isoc_eb_ai.csv`:

- Year: **2025**
- `freq=A`
- `size_emp=GE10`
- `nace_r2=C10-S951_X_K`
- `indic_is=E_AI_TANY`
- `unit=PC_ENT`

A read-only reproduction of this filter found 36 dimension-matched geographic rows, of which 35 had a non-missing 2025 value. After harmonising Eurostat `EL` to ESS `GR` and restricting the table to countries in the cleaned ESS sample, the merge table contained **25 rows for 25 unique countries**. No duplicate country keys were found.

Notebook 03 checks country uniqueness before merging and uses `validate='many_to_one'` in the ESS-Eurostat left merge. It also raises an assertion if the row count changes. The final dataset contains no country with more than one non-missing AI-adoption value.

The five unmatched countries are documented consistently as:

- CH
- GB
- IL
- IS
- UA

These countries account for **944 rows with missing `ai_adoption_enterprises_pct`**. This is documented missing coverage, not merge expansion.

## 3. ESS Multilevel Merge

Notebook 03 aggregates the selected ESS Multilevel numeric columns to one row per country before merging. It retains `validate='many_to_one'` and raises an assertion if the row count changes.

- Rows before ESS Multilevel merge: **6,336**
- Rows after ESS Multilevel merge: **6,336**
- Row expansion: **none**

The ESS Multilevel merge is therefore not responsible for duplicate respondent rows.

## 4. Required Variables

All required variable groups are present in both corrected saved datasets.

### Outcome

- `employed`

### Education and human capital

- `education_years`
- `education_level`
- `high_education`
- `age`
- `age_squared`

### Digital readiness

- `digital_readiness`

### Gender and family

- `female`
- `children_household`
- `household_size`
- `married`

### Migration, health, trust, and discrimination

- `migration_background`
- `health_status`
- `institutional_trust_index`
- `discrimination_experience`

### Country-level AI adoption

- `ai_adoption_enterprises_pct`

### Hypothesis-related interactions

- `education_years_x_ai_adoption`
- `digital_readiness_x_ai_adoption`
- `female_x_children`
- `migration_x_education`
- `migration_x_digital_readiness`
- `female_x_children_x_ai_adoption`

All six saved interaction columns were recalculated from their component variables during validation and matched the saved values, including missing-value locations. Main effects support H1, the education and digital-readiness interactions support H2, the gender-family interaction supports H3, the migration interactions support H4, and the optional triple interaction is available for the H5-related moderation analysis.

## 5. Documentation Review

The following files reflect the corrected dataset consistently:

- `docs/dataset_notes.md` records the exact Eurostat dimensions, 2025 year, `EL` to `GR` harmonisation, 25 unique matched countries, five unmatched countries, many-to-one validation, and the preserved 6,336 rows.
- `docs/variable_dictionary.md` records the corrected definition and full filter for `ai_adoption_enterprises_pct`, the required individual-level variables, and the interaction terms.
- `outputs/reports/data_engineering_report.md` records 6,336 rows before and after both country-level merges, 25 filtered Eurostat rows and unique countries, both many-to-one validations, the unmatched countries, and the final 6,336 × 1,291 shape.

No material contradiction was found among these documents and the saved datasets.

## 6. Remaining Blockers and Limitations

The Eurostat merge blocker is resolved. The following separate issues remain:

1. The current `mainact` employment construction and special missing-value treatment must still be verified against the official ESS Round 11 codebook. This issue is especially important because the current procedure removes 43,780 of 50,116 ESS rows and leaves 6,336 observations.
2. `children_household` remains a documented proxy based on `household_size > 1` because the intended children-at-home variable was not found. It must not be described as a verified children measure.
3. `digital_readiness` remains an individual internet-use-frequency proxy.
4. AI adoption is missing for 944 respondents in CH, GB, IL, IS, and UA. Downstream models using AI adoption or its interactions must handle and report the resulting analytical sample explicitly.
5. The 1,265 added ESS Multilevel columns are broad candidate contextual variables, not a theory-selected macro-control set. Downstream specifications must not treat all of them as confirmed controls without a separate selection and year-alignment decision.
6. Previous model outputs from Notebooks 04–07 remain stale because they were produced before the corrected modelling dataset was established.

These issues do not recreate the row-expansion problem, but they remain necessary qualifications for modelling and thesis interpretation.

## 7. Downstream Impact

The following notebooks must be rerun in order after this validation:

1. `notebooks/04_baseline_econometric_models.ipynb`
2. `notebooks/05_advanced_econometric_models.ipynb`
3. `notebooks/06_machine_learning_models.ipynb`
4. `notebooks/07_model_comparison_and_results.ipynb`

Their existing coefficients, diagnostics, predictive metrics, tables, figures, saved models, hypothesis summaries, and interpretation reports must be treated as stale until regenerated from the corrected 6,336-row dataset.

## Final Decision

**VALIDATED** for downstream rerunning of Notebooks 04–07.

- Final row count: **6,336**
- Final column count: **1,291**
- Duplicate row expansion: **fixed**
- Eurostat merge validation: **many-to-one**
- ESS Multilevel merge validation: **many-to-one**
- Remaining blockers: ESS codebook verification, documented proxy limitations, missing AI coverage for five countries, explicit downstream sample reporting, macro-control selection, and regeneration of all downstream results
- Downstream notebooks to rerun: **04, 05, 06, and 07**
- Thesis writing status: **remain paused until Notebooks 04–07 have been rerun and their corrected outputs have been validated**
