# Multilevel Analysis of Students' Mathematics Achievement

**English** · [Bahasa Indonesia](README.id.md)

A multilevel (linear mixed model) analysis and interactive HTML dashboard examining what drives mathematics scores across 900 students nested in 30 schools, and whether interventions should target students, schools, or both.

- **Interactive dashboard file:** [`index.html`](index.html)
- **Analysis notebook & data:** [`FINAL_RASIO.ipynb`](FINAL_RASIO.ipynb) · [`Final Round Data.xlsx`](Final%20Round%20Data.xlsx)

## Research questions and answers

1. **How much variation lies between students vs between schools?**
   About **8.2%** lies between schools and **91.8%** between students (`ICC = 0.082`, design effect `3.38`), so a multilevel model is required.
2. **Do student characteristics explain achievement?**
   Yes. `ses`, `prior_score`, `study_hours`, and `gender` are all significant (`p < 0.001`) and together explain **98.6%** of within-school variance. `ses` is the strongest predictor.
3. **Do school characteristics explain between-school differences?**
   Partly. `teacher_exp` is a robust positive predictor across specifications, whereas `facility` is positive but not consistently significant.
4. **Do between-school differences remain after accounting for both levels?**
   Yes. About **62%** of the initial school variance remains (`p < 0.001`).

## Key findings

| Quantity | M1 (Null) | M2 (Student) | M3 (Full) | M4 (+ Random slope) |
|---|---:|---:|---:|---:|
| School variance (`tau00`) | 3.165 | 2.340 | 1.957 | 1.876 |
| Student variance (`sigma^2`) | 35.408 | 0.507 | 0.507 | 0.365 |
| AIC (ML) | 5808.4 | 2099.7 | 2096.3 | 1834.5 |
| BIC (ML) | 5822.8 | 2133.3 | 2139.5 | 1887.3 |

Fixed effects in M3 (per 1 SD of each predictor):

| Predictor | Coefficient | p-value |
|---|---:|---:|
| `ses` | 5.382 | < 0.001 |
| `prior_score` | 2.345 | < 0.001 |
| `gender` (male) | 0.422 | < 0.001 |
| `study_hours` | 0.332 | < 0.001 |
| `teacher_exp` | 0.702 | 0.010 |
| `facility` | 0.478 | 0.081 |

- **School-effect robustness:** `teacher_exp` stays significant in all four checks (M3, M4, cluster-robust OLS, and two-stage school-mean regression) and in **29 of 30** leave-one-school-out refits. `facility` is significant only in M4 and in **2 of 30** leave-one-school-out refits; with 30 schools, level-2 power is limited, so non-significance does not prove zero effect.
- **Varying study-hours payoff (M4):** Adding a school-level random slope for `study_hours` improves fit strongly (`LRT = 265.8, p < 0.001`). The average slope is `0.310` per SD with a between-school SD of `0.384`, meaning extra study time pays off very differently across schools. Random slopes for `ses` and `prior_score` are not supported.
- **Diagnostics:** Residuals and school random effects pass Shapiro-Wilk normality checks, relationships are linear, and the largest VIF is `1.10`. M3 fails cross-school homoscedasticity (`Levene p < 0.001`), which M4 resolves (`Levene p = 0.348`).

### Recommendations

- Prioritize student-level support, especially for students from lower-SES households and weaker prior scores.
- Invest in teacher quality and experience, the most robust school-level factor.
- Tailor study-hours programmes per school rather than using a one-size-fits-all target.
- Investigate additional school factors, since 62% of initial school variance remains unexplained.

## How to read the dashboard

[`index.html`](index.html) is a single self-contained HTML dashboard (fonts and Chart.js inlined; no build step required). Navigate across its 12 tabs in the top bar:

| Tab group | Tabs on screen | What to look for |
|---|---|---|
| **Context & setup** | **Background**, **Overview**, **Models**, **Data & EDA** | Survey design (`30 schools × 30 students`), hierarchical variable table, M1–M4 specifications, and distributions across schools. |
| **Core questions** | **1 Variance**, **2 Student traits**, **3 School traits**, **4 Residual variance** | ICC decomposition (`8.2%` between schools), standardized fixed effects, robustness checks for `teacher_exp` vs `facility`, and remaining school variance. |
| **Extensions & checks** | **Study-hours slope**, **Diagnostics**, **Conclusions** | School-by-school variation in the `study_hours` slope (M4), normality/homoscedasticity checks, and observed-vs-predicted chart (`r = 0.994`). |
| **Interactive tool** | **Simulator** | Move the sliders for `ses`, `prior_score`, `study_hours`, `facility`, `teacher_exp`, and **Gender** to inspect the predicted `math_score` and each variable's contribution relative to an average student in an average school (M3 fixed effects, school random effect = 0). |

## Data and models

A survey sampled 30 schools and 30 students per school (900 students total), with zero missing values or duplicate rows. Because students within the same school share teachers and facilities, ordinary regression understates level-2 standard errors.

| Level | Variable | Description |
|---|---|---|
| 1, student | `math_score` (outcome) | Mathematics test score |
| 1, student | `prior_score` | Mathematical ability before instruction |
| 1, student | `ses` | Family socioeconomic status |
| 1, student | `study_hours` | Average weekly hours studying mathematics |
| 1, student | `gender` | Male or female (`Laki-laki` / `Perempuan` in the Excel file) |
| 2, school | `facility` | Quality of learning facilities |
| 2, school | `teacher_exp` | Years of mathematics teacher experience |

Level-1 predictors are z-scored across the 900 students and level-2 predictors across the 30 schools, so coefficients represent effects per 1 SD. All four linear mixed models (M1 Null, M2 Student, M3 Full, M4 + random `study_hours` slope) are fitted with Python `statsmodels` `MixedLM` (REML for estimation; ML for LRT, AIC, and BIC), supported by cluster-robust OLS, two-stage regression, leave-one-school-out refits, and an exploratory contextual model. In [`Final Round Data.xlsx`](Final%20Round%20Data.xlsx), column names use dots (`study.hours`, `prior.score`, `teacher.exp`, `math.score`), which the notebook renames with underscores.

## Limitations

- Observational dataset, so estimates reflect associations rather than causal effects.
- Only 30 schools, which limits statistical power at level 2 and makes school-level estimates sensitive to individual schools.
- Cross-school homoscedasticity holds only after introducing the random slope in M4.
- M4 inference relies on an intercept-slope correlation of `0.96` estimated from 30 schools.
- The contextual model with school means is exploratory.

## How to run locally and reproduce

### 1. Open the dashboard locally

Serve the repository root over HTTP:

```bash
python -m http.server 8000
```

Then open http://localhost:8000 in your browser (stop with `Ctrl+C`). Because `index.html` bundles Chart.js and fonts inline, it can also be opened offline directly in any modern browser.

### 2. Reproduce the analysis notebook

The full Python workflow lives in [`FINAL_RASIO.ipynb`](FINAL_RASIO.ipynb):

- **Google Colab:** Open [`FINAL_RASIO.ipynb`](FINAL_RASIO.ipynb) and run all cells. When prompted in the data cell, upload [`Final Round Data.xlsx`](Final%20Round%20Data.xlsx).
- **Local Jupyter:** Keep `FINAL_RASIO.ipynb` and `Final Round Data.xlsx` in the same folder, install dependencies if needed, and run all cells top to bottom:

```bash
pip install pandas openpyxl statsmodels scipy matplotlib seaborn
```

A successful run prints `ICC = 0.0821` in section 3 and reproduces the exact M1–M4 model summary table shown in the dashboard in section 9.

## Repository contents

| File | Contents |
|---|---|
| [`index.html`](index.html) | Self-contained interactive dashboard |
| [`FINAL_RASIO.ipynb`](FINAL_RASIO.ipynb) | Full Python analysis notebook with outputs |
| [`Final Round Data.xlsx`](Final%20Round%20Data.xlsx) | Dataset (900 students, 30 schools) |
| [`README.md`](README.md), [`README.id.md`](README.id.md) | Documentation in English and Indonesian |
