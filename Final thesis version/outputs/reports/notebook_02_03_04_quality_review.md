# Quality Review — Notebooks 02, 03, 04

This is a **read-only review**. No notebook, dataset, or code was modified while
producing this document, and no new models were run. All statements below are
based only on: the actual cell contents of `notebooks/02_eda_dataset_understanding.ipynb`,
`notebooks/03_data_engineering_and_final_dataset.ipynb`, and
`notebooks/04_baseline_econometric_models.ipynb`; the saved reports
(`outputs/reports/eda_findings.md`, `outputs/reports/data_engineering_report.md`,
`outputs/reports/baseline_econometric_interpretation.md`,
`outputs/reports/dataset_organization_report.md`); and `docs/variable_dictionary.md`
and `docs/methodology_notes.md`.

---

## 1. Executive Summary

Notebooks 02–04 implement a coherent, well-documented pipeline: raw ESS11 survey
data + Eurostat AI-adoption data + ESS Multilevel macro data are explored (NB02),
cleaned and merged into a single analysis-ready dataset of 76,628 rows × 1,291
columns (NB03), and used to fit four baseline models — LPM, Logit, Probit, and
Logit with country fixed effects — that are interpreted hypothesis-by-hypothesis
for H1–H5 (NB04). The engineering work is careful: special missing codes are
recoded, a column-name collision between ESS and ESS-Multilevel data was caught
and excluded, dtype issues were fixed, and every derived/interaction variable is
tied to a hypothesis in `docs/variable_dictionary.md`.

The main gaps are not in what was built, but in what was **not yet checked**:
there is no formal multicollinearity diagnostic beyond a pairwise correlation
threshold (no VIF), no outlier treatment decision for the final modelling
variables (only diagnostic outlier counts in EDA), no missing-data mechanism
discussion or imputation strategy, survey weights (`survey_weight`) are built but
unused in NB04, and standard errors are not clustered by country despite having
a natural 30-country grouping structure. None of these gaps invalidate the work
done so far — the association-only, cross-sectional framing in
`docs/methodology_notes.md` is appropriate — but they should be addressed or
explicitly justified before/while building notebook 05.

**Overall assessment: the pipeline is sound and ready to support notebook 05**,
provided the recommendations in Section 7 (particularly VIF, weighting, and
clustered standard errors) are incorporated into the robustness-check design
that notebook 05 already plans to run.

---

## 2. What Has Already Been Done Well

- **Data provenance and organization** (`outputs/reports/dataset_organization_report.md`):
  every raw archive was inspected before extraction, files were renamed/mapped
  transparently, and known format mismatches (CSV instead of `.sav`) were flagged
  rather than silently worked around.
- **Missing-value diagnostics in EDA (NB02)**: a full missingness table
  (`missing_count`, `missing_pct`) was computed for all 691 raw ESS columns, plus
  a `missingno` heatmap for the 30 worst columns, and the finding (289 columns
  with missingness, worst at 100%) was explicitly logged.
- **Outlier and distribution diagnostics in EDA (NB02)**: skewness (via
  `scipy.stats.skew`) and an IQR-based outlier table (`n_outliers`, `pct_outliers`,
  bounds) were computed for numeric candidates — good practice even though no
  treatment followed (see Section 4).
- **Special missing-value recoding (NB03)**: ESS's numeric sentinel codes (e.g.
  `mainact`, `agea`, `eduyrs`, trust items) were explicitly converted to `NaN`
  for 15 columns, with counts logged — this is a common and easy-to-miss survey
  data pitfall that was handled correctly.
- **Careful, defensive merging (NB03)**: the ESS Multilevel data was loaded
  column-selectively (1,268 of 9,348 columns) specifically to avoid a
  previously-hit `MemoryError`; a column-name collision (`educde2`, `educgb1`)
  between ESS and ESS-Multilevel data was detected and excluded before merging,
  with the exclusion logged in the report rather than silently overwriting data.
- **Feature engineering is hypothesis-driven and documented**: every derived
  variable (`age_squared`, `high_education`, `digital_readiness`,
  `institutional_trust_index`, `discrimination_experience`, and all 5–6
  interaction terms) is mapped to H1–H5 in `docs/variable_dictionary.md`, and
  proxy variables (e.g. `children_household` from `household_size > 1`,
  `digital_readiness` from `netusoft`) are explicitly flagged as documented
  assumptions rather than presented as exact measures.
- **`institutional_trust_index` as a composite** is a reasonable, lightweight
  dimension-reduction step (row-wise mean of 4 trust items) rather than
  including 4 correlated raw items separately.
- **Model selection and comparison in NB04**: fitting LPM, Logit, Probit, and
  Logit+country-FE side by side, with a `model_comparison_table` reporting
  `n_obs`, significant variables, direction of effects, and hypotheses
  supported, is a strong, transparent way to present baseline results.
- **Heteroskedasticity-robust standard errors (HC1)** are used for the LPM,
  which is the correct minimum requirement given LPM's known heteroskedasticity.
- **Multicollinearity screening exists**: a full pairwise correlation matrix is
  computed and pairs with `|correlation| > 0.6` are explicitly flagged for
  interpretation caution, with the matrix also saved to the results workbook.
- **Consistent association-only language**: `docs/methodology_notes.md` and
  `outputs/reports/baseline_econometric_interpretation.md` both explicitly avoid
  causal claims, which is the correct framing for cross-sectional survey data.
- **Reproducibility**: `requirements.txt` is kept in sync with actual dependency
  fixes (e.g. `tabulate` was added after the bug that required it was found),
  and `docs/variable_dictionary.md` states it is regenerated by NB03, not
  hand-edited.

---

## 3. What Has Not Been Done Yet

- **No formal multicollinearity diagnostic (VIF)** — only pairwise correlation
  (>0.6) is checked in NB04. Pairwise correlation cannot detect multicollinearity
  that arises from combinations of 3+ variables (increasingly relevant once
  interaction terms are added in NB05).
- **No outlier treatment decision recorded for modelling variables** — NB02
  computes outlier counts/bounds as a diagnostic, but NB03/04 do not state
  whether any values were capped, winsorized, or left untouched, nor why.
- **No missing-data mechanism analysis** (MCAR/MAR/MNAR discussion) and **no
  imputation strategy** — NB03/04 rely entirely on listwise deletion (dropping
  43,780 rows with missing `employed`, then further rows with missing
  covariates in NB04). This is defensible but undocumented as a deliberate choice
  versus an oversight.
- **Survey weights are unused** — `survey_weight` (from ESS `anweight`) is built
  in NB03 but none of the four NB04 models use it (no `WLS`, no `freq_weights`,
  no weighted Logit). Weighting is only mentioned as a *planned* robustness
  check for NB05.
- **No clustering of standard errors by country** — all NB04 models (including
  the country-FE model) use HC1/default standard errors, not standard errors
  clustered at the country level, despite 30 countries being a natural
  clustering unit for cross-national survey data.
- **No formal specification tests** — no Hosmer–Lemeshow test, no link test, no
  Breusch–Pagan/White test formalizing the "LPM is heteroskedastic" statement
  (only asserted, not tested).
- **No classification-quality metrics** (confusion matrix, ROC/AUC, accuracy) for
  the Logit/Probit models — only coefficients, significance, and pseudo-R².
  This is acceptable for hypothesis testing but means model fit beyond pseudo-R²
  is not assessed.
- **No explicit encoding step is documented** — binary variables are hand-built
  as 0/1 in NB03, and `country` is left as a raw string, implicitly dummy-coded
  by `statsmodels`' `C(country)` only at model-fit time in NB04's model 4. This
  works, but there is no explicit "encoding" section a reader can point to.
- **No scaling/standardization** of continuous variables (`age`,
  `education_years`, `household_size`, `institutional_trust_index`,
  `ai_adoption_enterprises_pct`) anywhere in NB03/04. Fine for LPM/Logit/Probit
  coefficient interpretability, but will matter for NB06 (regularized models).

---

## 4. What Is Missing but Necessary

- **VIF (variance inflation factor) check before/alongside the NB05 interaction
  models.** Interaction terms are well known to inflate collinearity with their
  component main effects; a pairwise-correlation-only check is not sufficient
  once `education_years_x_ai_adoption`, `digital_readiness_x_ai_adoption`, etc.
  are added.
- **A documented decision on survey weights.** Given `survey_weight` already
  exists and ESS is a weighted survey design, at least one weighted specification
  should be run and compared to the unweighted one — this is already planned in
  NB05's robustness checks, so it is necessary that it actually gets executed
  and interpreted, not just coded.
- **Country-clustered standard errors** (or acknowledgement of why they were not
  used) — with only 30 clusters this needs to be handled carefully (30 is on the
  low side for cluster-robust asymptotics), but at minimum the choice should be
  discussed in `docs/methodology_notes.md`.
- **An explicit statement of the missing-data handling philosophy** (listwise
  deletion vs. imputation) with a sentence on why imputation was not used for
  the ~2,904 dropped rows in NB04 and the still-missing macro columns retained
  in the final dataset.

## 5. What Is Missing but Not Necessary

- **Full imputation of the sparse macro-level columns** (many `reg11_*`, `n3_*`
  Eurostat regional variables are >70–99% missing per
  `outputs/reports/data_engineering_report.md`). These columns are not part of
  `BASE_VARS`/`INTERACTION_VARS` used in NB04/05, so imputing them is not
  necessary for the current hypotheses — they can remain as-is or be dropped
  from the analysis dataset without loss.
- **Dimension reduction (PCA/factor analysis) across the full 1,291-column
  dataset** — not necessary given the thesis explicitly restricts modelling to
  ~13 pre-specified theory-driven variables plus interactions; PCA would work
  against the interpretability goals of the econometric chapters.
- **Outlier winsorizing of survey-scale variables** (`health_status`,
  `digital_readiness`, `institutional_trust_index` are bounded ordinal/index
  scales) — "outliers" here are typically valid extreme responses, not data
  errors, so capping them is not necessary and could distort substantive meaning.
- **Encoding `country` as explicit one-hot dummy columns in the stored dataset**
  — `statsmodels`' formula API already handles this correctly and efficiently at
  fit time; materializing 30 dummy columns in the saved dataset would add no
  value and increase file size.

## 6. What Should Be Avoided

- **Do not introduce causal language** anywhere in NB05/06/07 — the current
  discipline (association/predicts/correlates only) is a real strength and
  should be preserved, especially once interaction and ML "importance" results
  start to look causally suggestive.
- **Do not silently impute the sparse macro columns** just to "fill in" the
  dataset — if any macro control is added later, missingness should be handled
  transparently (e.g., explicit indicator + documented imputation), not
  mean-filled without disclosure, to avoid manufacturing false precision.
- **Do not drop the `survey_weight`/robustness-check plan in NB05** in favor of
  only the unweighted specification — since it is already designed, quietly
  skipping it would understate a known limitation of the baseline models.
- **Do not add PCA/dimension reduction to the econometric notebooks** (04/05) —
  it belongs, if anywhere, to the ML notebook (06), since it would undermine
  the hypothesis-by-hypothesis interpretability that is the stated goal of the
  econometric chapters.
- **Do not compare econometric and ML results inside NB05 or NB06** — per
  `docs/methodology_notes.md`, that comparison is explicitly reserved for NB07;
  keep that separation of concerns.

---

## 7. Specific Recommendations for Notebook 05

1. **Add a VIF table** for the interaction-model variable set (base vars +
   interaction terms) before interpreting coefficients — this is the single
   highest-priority addition, since the existing correlation-threshold check in
   NB04 does not extend cleanly to multi-variable collinearity from interactions.
2. **Actually run and interpret the weighted vs. unweighted comparison** already
   planned in the robustness-check table in `docs/methodology_notes.md`, using
   the existing `survey_weight` column — do not leave it as a documented-but-unrun
   plan.
3. **Consider clustering standard errors by `country`** (`cov_type='cluster'`,
   `cov_kwds={'groups': df['country']}` in `statsmodels`) for at least one
   specification, and note the small-cluster caveat (30 countries) explicitly
   rather than silently using default/robust SEs everywhere.
4. **Report marginal effects at representative AI-adoption values (low/median/
   high)** as already planned — this is the correct way to interpret
   interaction coefficients, and should be the primary basis for the H2/H3/H4
   interpretation rather than the raw interaction coefficient sign/significance.
5. **Resolve and re-verify the pending execution issue**: the imports cell
   returned "did not finish executing" in the last session — before trusting any
   of the 10 drafted sections, re-run the notebook top-to-bottom in a fresh
   kernel and confirm every cell (especially the model-fitting and save-outputs
   cells) executes and produces the expected files
   (`outputs/tables/advanced_econometric_results.xlsx`,
   `outputs/figures/advanced_econometric_coefficients.png`,
   `outputs/reports/advanced_econometric_interpretation.md`).
6. **Double-check convergence** for the Logit models with 5–6 interaction terms
   and (in one robustness variant) country fixed effects simultaneously — this
   is a much larger parameter space than NB04's models and convergence warnings
   should be surfaced in the output, not swallowed by the `try/except` block.

## 8. Specific Recommendations for Notebook 06

1. **Add scaling/standardization** (`StandardScaler` or similar) for continuous
   features before fitting regularized models (L1/L2 logistic regression, SVM)
   — regularization penalties are scale-sensitive, unlike the OLS/Logit/Probit
   coefficients used in NB04/05.
2. **Use the same restricted, theory-driven variable set** (`BASE_VARS` +
   interactions) as the primary feature set, rather than the full 1,291-column
   dataset, to keep results comparable to NB04/05 in NB07 — the wide macro
   columns can be used in a clearly-labeled secondary/exploratory model if
   desired, but should not be mixed into the primary comparison model.
3. **Reuse the same train/test split (or cross-validation folds) across all
   ML models** in the notebook, and report it explicitly, so accuracy/F1/ROC-AUC
   comparisons across models are apples-to-apples.
4. **Explainability**: SHAP or permutation importance should be computed on the
   same restricted feature set, and results should be framed in terms of H1–H5
   (per the notebook's own stated purpose) rather than as generic
   "top predictors," to make the NB07 econometric-vs-ML comparison meaningful.
5. **Handle the same missing-data question consistently** — decide explicitly
   whether NB06 uses the same listwise-deletion sample as NB04/05 or a different
   (e.g., imputed) sample, and document the choice, since this affects
   comparability of results across notebooks 04–07.

---

## 9. Risks for the Final Thesis Project

- **Comparability risk**: if NB06 uses a different sample (different missing-
  data handling, different variable set, or scaled vs. unscaled features) than
  NB04/05, the NB07 hypothesis-by-hypothesis comparison could conflate genuine
  model differences with sample/preprocessing differences.
- **Multicollinearity risk in NB05**: adding 5–6 interaction terms without a
  VIF check risks unstable or misleading coefficients, which would directly
  affect the H2/H3/H4 conclusions.
- **Weighting risk**: ESS is a weighted, multi-country survey; presenting only
  unweighted estimates as final results (if the NB05 weighted robustness check
  is skipped) could be challenged in examination as a methodological gap, since
  weighting is standard practice for this data source.
- **Cluster-robust inference risk**: without clustering by country, standard
  errors in the pooled models may be understated (observations within a country
  are unlikely to be independent), inflating apparent statistical significance.
- **Execution-state risk**: notebook 05 was drafted but its first execution
  attempt was inconclusive (ambiguous "did not finish executing" status) and
  was never resolved. If this is not re-verified, there is a real risk of
  reporting results from code that was never actually confirmed to run
  correctly end-to-end.
- **Sparse macro-column risk**: several ESS-Multilevel macro columns are
  >90–99% missing in the final dataset; if any of these are used later (e.g.,
  as additional country-level controls) without acknowledging the missingness,
  results could be based on a very small, non-random subset of countries/years.
- **Documentation drift risk**: `docs/methodology_notes.md`'s "Interpretation
  Log" table still says "Notebooks 04 and 05 not yet implemented/run with real
  data" even though NB04 has been run and interpreted — this file should be
  updated to avoid future confusion about what has actually been validated.

---

## 10. Final Decision: Ready to Proceed to Notebook 05?

**Yes, conditionally.** Notebooks 02–04 form a sound, well-documented, and
correctly-scoped foundation: the data engineering is careful and transparent,
the baseline econometric models are appropriately chosen and interpreted with
consistent association-only language, and the identified gaps (VIF, weighting,
clustering, missing-data documentation) are refinements rather than
fundamental flaws.

Before treating notebook 05's results as final, the following should happen,
roughly in this order:
1. Re-verify notebook 05 executes cleanly end-to-end from a fresh kernel
   (resolve the outstanding "did not finish executing" issue).
2. Add the VIF check and the country-clustering discussion.
3. Actually run and interpret the weighted-vs-unweighted robustness check.
4. Update `docs/methodology_notes.md`'s interpretation log to reflect that NB04
   is complete and validated.

None of these require redoing notebook 03 or re-architecting notebook 04 — they
are additive checks layered onto the existing, working pipeline. **Proceeding
with notebook 05 execution and validation is recommended**, incorporating
Section 7's recommendations as they are reached in the existing 10-section
notebook structure.
