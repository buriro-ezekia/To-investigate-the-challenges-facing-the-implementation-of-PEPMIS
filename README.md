# PEPMIS Teacher Survey — Analysis

This repository contains scripts and outputs used to clean, analyse, visualise and report results from a teacher survey about challenges implementing the PEPMIS (primary/education/performance electronic system) in public secondary schools. The work covers data preparation, descriptive statistics, reliability testing (Cronbach’s alpha), construction of a composite Challenge Index, inferential tests (t-test, ANOVA), exploratory factor analysis (EFA), visualisations, and a multivariable regression.

This README describes what was done, which scripts were created, how to run them, key results, caveats, and suggested next steps.

---

## Project overview
- Input: a CSV export of a teacher questionnaire (Likert-style items + demographics).
- Goal: understand which PEPMIS challenges are most important, whether items form a cohesive scale, whether groups differ, and to create a single Challenge Index that summarises overall difficulty so it can be used in comparisons and regression.
- Approach: data cleaning → per-item descriptives → Cronbach’s alpha → Challenge Index → group comparisons (t-test/ANOVA) → EFA → regression → visuals.

---

## Files & scripts produced
All scripts assume input and working paths as listed below unless you change the INPUT constant inside a script.

Scripts (ready-to-run):
- `data_prep_pepmis.py`  
  Clean and normalise raw CSV, parse Likert responses, create `Age_clean`, remove rows missing all likert items, write cleaned CSV.  
  Input: `drive/MyDrive/Survey_Analysis/QUESTIONNAIRE GUIDES FOR TEACHERS-Form_Modified.csv`  
  Output: `drive/MyDrive/Survey_Analysis/QUESTIONNAIRE GUIDES FOR TEACHERS-Form_Modified_cleaned.csv`

- `analysis_descriptive_pepmis.py`  
  Compute per-item mean/median/std/counts and percent agree (4 or 5). Saves `pepmis_descriptive_summary.csv` and plots: `mean_challenge_scores.png`, `stacked_response_distribution.png`.

- `compute_reliability_pepmis.py`  
  Computes Cronbach’s alpha, corrected item–total correlations and "alpha if item deleted". Saves `pepmis_reliability_summary.csv`.

- `construct_challenge_index_pepmis.py`  
  Build `Challenge_Index` as the (rounded) mean of the 7 items (default requires at least 4 non-missing items). Writes `..._with_index.csv`, `pepmis_index_summary.csv`, `pepmis_index_by_group.csv` and plots: `challenge_index_histogram.png`, `challenge_index_by_age_box.png`.

- `analysis_inferential_pepmis.py`  
  Runs inferential tests:
  - Sex → Welch t-test
  - Age_clean, Education_Level, Working_Experience → One-way ANOVA (Tukey HSD if significant)  
  Saves `pepmis_inferential_results.txt`, `pepmis_tukey_<var>.csv` (if applicable), `pepmis_group_stats.csv`, and boxplots.

- `visualization_pepmis.py`  
  Create publication-ready visuals:
  - `mean_challenge_scores.png` (means + SEM)
  - `stacked_response_distribution.png` (stacked % disagree/neutral/agree; legend outside)
  - `boxplot_<group>.png` for each group variable present

- `efa_pepmis.py`  
  Exploratory Factor Analysis:
  - Computes KMO and Bartlett tests (implemented internally so it does not depend on `factor_analyzer.utils`).
  - Produces eigenvalues, scree plot, factor loadings and communalities. Saves: `efa_eigenvalues.csv`, `efa_loadings.csv`, `efa_communalities.csv`, `efa_factor_variance.csv`, `efa_scree.png`, `efa_loadings_heatmap.png`.
  - Usage: supports `--n-factors` and `--impute` flags.

- `regression_pepmis.py`  
  Multivariable linear regression predicting `Challenge_Index` from demographics:
  - Uses robust SE (HC3).
  - Reports coefficients, 95% CI, p-values, R² and adj-R².
  - Computes VIF and partial R² (by main categorical predictor).
  - Saves summary and diagnostic plots: `pepmis_regression_summary.txt`, `pepmis_regression_coefs.csv`, `pepmis_regression_vif.csv`, `pepmis_regression_partialR2.csv`, `residuals_vs_fitted.png`, `qqplot_residuals.png`, `leverage_cooks.png`, `observed_vs_predicted.png`.

Example produced outputs (from running on 27 respondents):
- Descriptives: means for each item, percent agree (best: Training_Gap_PEPMIS and Limited_Access_ICT).
- Reliability: Cronbach’s alpha ≈ 0.896 (scale is reliable).
- Challenge Index: mean ≈ 3.735, median = 4.0, sd ≈ 0.97.
- Inferential tests: no significant group differences by Sex (p≈0.356), Age (p≈0.726), Education (p≈0.966), Experience (p≈0.945).
- EFA: KMO overall ≈ 0.827, Bartlett p < 0.001, eigenvalues indicate one factor (Factor 1 explains ~62.7% variance). Items load strongly on a single factor (loadings ≈ 0.72–0.88).
- Regression: script prepared to fit model and outputs saved (interpretation to run and inspect saved files).

---

## Requirements / Environment
Recommended Python environment:
- Python 3.8+ (scripts tested under 3.10+ and 3.12 in notebook cells)
- pip packages:
  - pandas
  - numpy
  - scipy
  - matplotlib
  - seaborn
  - statsmodels
  - patsy
  - factor_analyzer (required for EFA)
  - scikit-learn (installed as a dependency for factor_analyzer)
Install with:
```
pip install pandas numpy scipy matplotlib seaborn statsmodels patsy factor_analyzer
```

Notes:
- Some installs of `factor_analyzer` may raise minor warnings from scikit-learn internals. The EFA script suppresses a narrow FutureWarning and handles rotation when only one factor is extracted.
- If you run scripts in a Jupyter/Colab notebook, they will ignore notebook kernel args (scripts use parse_known_args).

---

## How to run (examples)
All commands assume you are in the folder containing the scripts and that the input CSV is at the paths used in the scripts. Edit the INPUT constants near the top of scripts if your files are elsewhere.

1. Data cleaning:
```
python data_prep_pepmis.py
```

2. Descriptive stats + plots:
```
python analysis_descriptive_pepmis.py
```

3. Reliability:
```
python compute_reliability_pepmis.py
```

4. Build Challenge Index:
```
python construct_challenge_index_pepmis.py
```
- To require all items for the index, edit `MIN_NON_MISSING = 7` in the script.

5. Inferential tests:
```
python analysis_inferential_pepmis.py
```

6. Visualisations (recreate or tweak):
```
python visualization_pepmis.py
```

7. Exploratory Factor Analysis:
```
# Automatic factors (eigenvalues > 1)
python efa_pepmis.py

# Force 2 factors and impute missing values
python efa_pepmis.py --n-factors 2 --impute
```

8. Regression (main effects only):
```
python regression_pepmis.py
```
- To include pairwise interactions (use with caution for small n):
```
python regression_pepmis.py --interactions
```

---

## Key results and simple interpretation (summary)
- Teachers report moderate-to-high overall challenges with PEPMIS (Challenge Index mean ≈ 3.74/5; median = 4). This suggests respondents on average "agree" there are problems.
- Highest-rated challenge items: Training gaps and Limited access to ICT. These are the most urgent operational issues.
- Internal consistency: Cronbach’s alpha ≈ 0.90 → the seven items form a reliable scale; EFA suggests the items load on a single factor (common construct).
- Group differences: No statistically significant differences by sex, age group, education, or experience in this sample (p-values > 0.05), but sample size is small (n = 27).
- Regression: prepared to estimate adjusted associations between demographics and the Challenge Index; interpret coefficients with caution given sample size.

---

## Limitations & caveats
- Small sample size (n = 27) — statistical power is limited, especially for subgroup/comparative tests and multivariable models. Non-significant results may reflect low power rather than no effect.
- Some categorical groups may have very small cell counts — consider collapsing levels or collecting more data before drawing strong conclusions.
- EFA is exploratory; confirmatory testing (CFA) or larger samples are recommended before making strong claims about factor structure.
- Scripts assume reasonably consistent column names. If your CSV uses different names, either rename columns or edit the LIKERT_COLUMNS and GROUP_VARS lists in the scripts.

---

## Troubleshooting
- `factor_analyzer` install issues: try `pip install factor_analyzer` (and ensure `numpy`, `scipy`, and `scikit-learn` are up-to-date). If KMO/Bartlett functions are not present in your factor_analyzer version, the `efa_pepmis.py` script computes them internally and will still run EFA if `factor_analyzer` is available.
- Notebook/Colab argument parsing: scripts use `parse_known_args()` so kernel args won't crash the scripts.
- If plots or files are not created where expected, check the `INPUT` path and whether the cleaned CSV with index exists.

---

License / reuse
- You may reuse and adapt these scripts for the PEPMIS survey or similar Likert-based survey analyses. No warranty; check the methods and outputs before using in policy documents.

```
