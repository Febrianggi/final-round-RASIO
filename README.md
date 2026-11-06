# Multilevel Analysis of Students' Mathematics Achievement

An interactive dashboard and write-up of a multilevel (linear mixed model) analysis, built for the final round of the RASIO statistics olympiad. The case asks what drives students' mathematics scores when students are nested in schools, and whether interventions should target students, schools, or both.

## The case

An institution noticed that mathematics achievement varies a lot between schools. A survey randomly selected 30 schools and 30 students from each, giving 900 students in a two-level hierarchy. Students in the same school share facilities, teachers and academic environment, so their scores are not independent, and an ordinary regression would understate the standard errors of school-level variables.

The dataset was provided by the competition organizers and is not included in this repository.

| Level | Variable | Description |
|-------|----------|-------------|
| 1, student | math_score (outcome) | Mathematics test score |
| 1, student | prior_score | Mathematical ability before instruction |
| 1, student | ses | Socioeconomic status of the family |
| 1, student | study_hours | Average weekly hours studying mathematics |
| 1, student | gender | Male or female |
| 2, school | facility | Quality of learning facilities |
| 2, school | teacher_exp | Years of experience of mathematics teachers |

There are no missing values or duplicate rows. Level 1 predictors are z-scored over the 900 students and level 2 predictors over the 30 schools, so coefficients are effects per 1 SD.

## Research questions and answers

1. How much of the variation lies between students and how much between schools?
   About 8.2% lies between schools and 91.8% between students (ICC = 0.082). The design effect is 3.38, so a multilevel model is needed.
2. Can student characteristics explain achievement?
   Yes. SES, prior ability, study hours and gender are all significant, and together they explain 98.6% of the within-school variance. SES is the strongest predictor.
3. Can school characteristics explain differences between schools?
   Partly. Teacher experience is a robust positive predictor. Facility quality is positive but not consistently significant.
4. After accounting for both levels, do between-school differences remain?
   Yes. About 62% of the initial school variance remains, and it is significant.

## Models

All models are linear mixed models with a random school intercept. REML is used for estimation, and ML for likelihood ratio tests, AIC and BIC.

| Model | Specification | Purpose |
|-------|---------------|---------|
| M1 Null | Intercept + random school intercept | Variance decomposition and ICC |
| M2 Student | M1 + prior_score, ses, study_hours, gender | Question 2 |
| M3 Full | M2 + facility, teacher_exp | Questions 3 and 4 |
| M4 Extension | M3 + random slope of study_hours by school | Heterogeneity of the study hours effect |

Supporting checks: OLS and cluster-robust OLS, a two-stage regression on school means, leave-one-school-out refits of M3, random slope screening for ses and prior_score, and an exploratory contextual model with school means.

## Key results

| Quantity | M1 | M2 | M3 | M4 |
|----------|----|----|----|----|
| School variance (tau00) | 3.165 | 2.340 | 1.957 | 2.308 |
| Student variance (sigma squared) | 35.408 | 0.507 | 0.507 | 0.354 |
| AIC (ML) | 5808.4 | 2099.7 | 2096.3 | 1834.5 |
| BIC (ML) | 5822.8 | 2133.3 | 2139.5 | 1887.3 |

Fixed effects in M3, per 1 SD:

| Predictor | Coefficient | p |
|-----------|-------------|---|
| ses | 5.382 | < 0.001 |
| prior_score | 2.345 | < 0.001 |
| gender (male) | 0.422 | < 0.001 |
| study_hours | 0.332 | < 0.001 |
| teacher_exp | 0.702 | 0.010 |
| facility | 0.478 | 0.081 |

Robustness of the school effects:

- teacher_exp is significant in all four specifications (M3, M4, cluster-robust OLS, two-stage) and in 29 of 30 leave-one-school-out refits.
- facility is significant only in M4 and in 2 of 30 leave-one-school-out refits, so the evidence for it is not consistent. With only 30 schools, power at level 2 is limited, and "not significant" does not mean "no effect".

The random slope for study_hours is clearly needed (LRT = 265.8, p < 0.001). The average slope is 0.307 per SD with a between-school SD of 0.759, so the benefit of extra study time differs a lot across schools. Random slopes for ses and prior_score are not supported.

## Diagnostics

Residuals and school random effects pass normality checks (Shapiro-Wilk), the relationships look linear, and the largest VIF is 1.10. M3 fails the homoscedasticity check across schools (Levene p < 0.001), which is consistent with the varying study hours effect. M4 resolves it (Levene p = 0.270).

## Recommendations

- Prioritize student-level support, especially for students from lower SES backgrounds and with weaker prior ability.
- Invest in teacher quality and experience, the most robust school-level factor.
- Treat study hours programmes as school specific rather than one-size-fits-all.
- Look for other school factors, since a large part of the between-school variance is still unexplained.

## Limitations

- Observational data, so the results are associations and not causal effects.
- Only 30 schools, which limits power and makes school-level effects sensitive to individual schools.
- Homoscedasticity across schools holds only after adding the random slope in M4.
- The M4 inference relies on an intercept-slope correlation of 0.81 estimated from 30 schools.
- The contextual model with school means is exploratory.

## The dashboard

The dashboard is a single self-contained HTML file with no server or build step. Open it in any modern browser. It includes the fonts and Chart.js inline, and it has tabs for the background, an overview, each model, data exploration, each research question, the study hours slope, diagnostics, conclusions, and a simulator that predicts a score from the fixed effects of M3.

## Data

The data were supplied by the competition organizers. Check the organizers' terms before sharing the raw data, and note that the dashboard file contains the per-student values it needs for its charts.
