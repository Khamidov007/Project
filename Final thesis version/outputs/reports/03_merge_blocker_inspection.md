# Notebook 03 Merge Blocker Inspection

## Scope

This inspection is based only on the saved content and outputs of Notebook 03, the dataset notes, the variable dictionary, and the data-engineering report. No notebook or data file was modified or rerun.

## Finding

The Eurostat AI-adoption merge is unsafe and is the suspected source of the row expansion. Notebook 03 reduces the ESS data from 50,116 rows to 6,336 rows after removing records with missing employment status. The final saved artifact nevertheless has 76,628 rows. The Eurostat preparation retains 578 non-missing rows for 2025 and calls them countries, but it does not filter the other Eurostat dimensions or verify that country is unique. It then performs a left merge on country without merge validation. Repeated Eurostat country keys can therefore duplicate ESS respondent rows.

The saved notebook does not print the row count immediately before and after the Eurostat merge. The evidence supports the Eurostat merge as the source, but the exact duplication pattern is not logged and must be confirmed when the blocker is fixed.

## Code-section inspection

| Question | Notebook location | Inspection result |
|---|---|---|
| 1. Where is Eurostat loaded? | Section 2, cell 6 | The notebook selects the first CSV returned from the Eurostat raw-data folder and loads it with `pd.read_csv(eurostat_files[0])`. The saved output records `isoc_eb_ai.csv` with shape 620 rows by 10 columns. |
| 2. Where is Eurostat filtered? | Section 6, cell 23 | The code identifies the geography column and year columns, selects the latest year, coerces that year to numeric, keeps only geography and the selected year, and drops missing rows. No concept-level row filter is applied. |
| 3. Which columns are used for filtering? | Section 6, cell 23 | No Eurostat dimension column is used for row filtering. The code uses `geo\\TIME_PERIOD` or `geo` as the country column and digit-named columns to identify years. It then selects only the geography column and the latest-year value. No filter for frequency, enterprise-size class, economic activity, AI indicator, or unit appears in Notebook 03. |
| 4. Is 2025 selected? | Section 6, cell 23 | Yes. `ai_year_used = max(year_cols)` selects the latest digit-named year. The saved output identifies 2025. |
| 5. Is one AI-adoption concept selected? | Section 6, cell 23 | No. The code renames the selected 2025 value as `ai_adoption_enterprises_pct`, but it does not filter the raw table to one indicator, unit, enterprise-size category, economic activity, or frequency. The intended definition in the dataset notes and variable dictionary is therefore not established by this code. |
| 6. Is there one row per country before merging? | Section 6, cell 23 | Not established. The cleaned object has 578 rows after `dropna()`, far more than a plausible set of countries. There is no `groupby`, `drop_duplicates`, duplicate-key assertion, or uniqueness validation for country. |
| 7. Where is ESS merged with Eurostat? | Section 7, cell 25 | `df = df.merge(df_ai, on='country', how='left')`. |
| 8. Is merge validation used? | Section 7, cell 25 | No. The Eurostat merge does not use `validate='many_to_one'` or an equivalent key check. |
| 9. Where is ESS Multilevel merged? | Section 7, cell 26 | Candidate numeric columns are aggregated by country using `groupby(...).mean()`, country duplicates are removed, and the result is left-merged with `validate='many_to_one'`. This merge is protected against row multiplication. |
| 10. Where is the final dataset saved? | Section 10, cell 35 | `final_df` is written to the processed CSV and pickle paths. The saved output records shape 76,628 rows by 1,291 columns. |

## Recorded row counts

| Processing point | Recorded count |
|---|---:|
| Raw ESS Round 11 file | 50,116 rows |
| Rows removed because `mainact` was treated as missing | 43,780 rows |
| ESS rows remaining before country-level merges | 6,336 rows |
| Raw Eurostat file | 620 rows |
| Eurostat rows retained for 2025 after numeric conversion and missing-value removal | 578 rows |
| Final saved dataset | 76,628 rows |

The final count exceeds both the cleaned ESS sample and the original ESS file. A valid many-to-one country merge cannot produce this increase.

## Safety assessment

**Eurostat merge status: Unsafe.**

The merge is unsafe because the intended AI concept is not isolated, country uniqueness is not checked, repeated country keys are possible, and merge validation is absent. By contrast, the ESS Multilevel merge aggregates to one row per country and uses many-to-one validation.

## Required work in Prompt 2

1. Identify the exact Eurostat dimension values that represent the intended measure: enterprises using at least one AI technology.
2. Filter the raw Eurostat table to one frequency, enterprise-size category, economic-activity category, AI indicator, unit, and year.
3. Retain 2025 only if it is the documented analytical year.
4. Verify that the filtered table has exactly one non-missing observation per country. Stop with a clear error if country keys are duplicated.
5. Merge with `validate='many_to_one'` and record row counts immediately before and after the merge.
6. Assert that the Eurostat merge does not change the number of ESS rows.
7. Regenerate the final dataset and report only after the valid sample size is established.
8. Keep the ESS Multilevel many-to-one safeguard unchanged while separately reviewing whether its broad indicator selection is appropriate.

No fix has been applied in this inspection.
