# Turbulence Case Study: Project Brief and Revision Checklist

This is the working brief for future revisions. Begin here instead of re-reading the long PDFs. The active report is `report.qmd`; `case-study.qmd` is not the current report.

## Assignment and scientific context

The collaborators ran expensive direct numerical simulations of inertial particles in homogeneous, isotropic turbulent flow. Each of the 89 training simulations is one $(Re, Fr, St)$ setting and supplies the first four *raw* moments of the cluster-volume distribution. The report must convert those to four interpretable responses: mean, standard deviation, skewness, and kurtosis.

The two required objectives are:

1. **Prediction:** quickly predict all four distribution summaries at a new input setting, as a practical surrogate for simulations that can take days (and, for the most demanding settings, much longer).
2. **Inference:** identify and interpret how turbulence intensity ($Re$), gravity/fall speed ($Fr$), and particle inertia ($St$) affect each distributional feature; determine whether effects are nonlinear, differ by response, and interact.

Scientific meanings and design:

- $Re = 90, 224, 398$ quantifies turbulence intensity; larger values mean stronger turbulence.
- $Fr = 0.052, 0.3, \infty$ encodes gravity/fall speed. Smaller $Fr$ means stronger gravity; $Fr = \infty$ means no gravity.
- $St$ quantifies particle inertia (size/density response to the flow); it has 14 observed levels in the training data, spanning 0.05--3.
- Cluster volumes come from connected low-volume Voronoi cells. Small Voronoi volumes represent high particle concentration; a cluster volume is the sum of its constituent cells.
- One design cell is absent: $(Re, Fr) = (398, 0.3)$. Treating $Re$ and $Fr$ as factors is sensible for the sparse grid, but prevents interpolation/extrapolation.

The assignment calls for a short report of no more than four pages (including figures/tables) with Introduction, Methodology, Results (including prediction uncertainty), and Conclusion sections, plus a CSV of hold-out predictions. The appendix can carry supporting EDA/diagnostics.

## Current state of `report.qmd` (reviewed 2026-10-28)

The report:

- Computes the central moments and the four target summaries correctly from raw moments (setup chunk, lines 21--39).
- Fits four separate log-response GAMs. The selected form is `Re_f * Fr_f + s(log_St, k = 5)`: a common, REML-penalized $St$ smooth plus the observed $Re \times Fr$ interaction. It omits the unavailable $Re=398, Fr=0.3$ cell.
- Uses LOOCV to compare a baseline additive model, two interaction models, the cubic model, natural splines (3--5 df), and common GAMs ($k=4,5,6$). The selected common GAM has average LOOCV MSE 0.0872, narrowly lower than the cubic's 0.0879.
- Includes response-wise skewness checks, log-transform comparisons, Box-Cox profiles, residual diagnostics, GAM-appropriate hold-out prediction intervals, scientific interpretation, and a limitations/future-work section.
- Writes `predictions.csv`, with hold-out point predictions and 95% prediction bounds.

Important current inconsistencies/risks:

- The prose, equation, predictions, and diagnostic comparison now name the common GAM as final. `glm()` remains only a fitting function for the linear-model LOOCV comparators.
- The selected GAM's reported mean-model EDF is 11.7 (not the earlier roughly 27); the final diagnostics still need a focused EDF/adequacy check.
- The main fitted-values figure is enabled, and `fig-st-effect` adds the selected partial $St$ smooth with approximate 95% bands.
- The introduction names prediction and inference, but the inference objective is currently only one incomplete sentence (line 112). It should explicitly preview nonlinear effects, effect selection, response-specific effects, and interactions.
- Do not assume the existing rendered PDFs exactly match `report.qmd`; render and inspect after substantive revisions.

## Completed feedback items

- [x] **Check response skewness separately.** `report.qmd` lines 44--46 and 135--137 calculate and discuss raw vs. log skewness for all four responses; Appendix Figure `fig-skew` visualizes it.
- [x] **Try a response-specific Box-Cox transform.** Lines 87--99 fit profiles for each response; the methodology and Appendix table/figure report the estimates and 95% profile intervals.
- [x] **Justify retaining the log transform.** Lines 135--139 state that every Box-Cox 95% interval contains $\lambda=0$, alongside the response-wise skewness and linearity rationale.
- [x] **Compare cubic and GAM models for all four responses.** The LOOCV table now includes cubic, natural-spline, and common-GAM alternatives for every response (`report.qmd` lines 50--94 and 157--167).
- [x] **Use the correct model terminology in the report.** The final model is accurately called a GAM; the cubic remains a linear-model comparator (`report.qmd` lines 125--135 and 148--169).
- [x] **Confirm the present GAM specification.** The final code is `Re_f * Fr_f + s(log_St, k = 5)`, a common $St$ smooth plus the observed factor interaction (`report.qmd` lines 69--94); the methodology states the same specification (lines 150--155).
- [x] **State the categorical-$Re/Fr$ limitation.** Lines 274--278 explain that the model cannot interpolate or extrapolate beyond the three simulated factor levels, and that nearest-level snapping discards the offset.
- [x] **Address $Fr=\infty$ for a continuous future model.** Line 278 proposes $1/Fr$, mapping $\infty$ to 0.

## Remaining revision checklist

### 1. Framing, story, and report structure

- [ ] **Strengthen the opening roadmap.** State both objectives in the first section and specify the inference targets: nonlinear $St$ effects, which effects matter for each moment, and scientifically meaningful interactions. Quantify the practical value: simulation runs cost days or longer, whereas the surrogate predicts immediately.
- [ ] **Choose one body-report storyline and prune figures accordingly.** Keep the compact model-comparison evidence plus the most informative effects/uncertainty figures. Move routine residual plots and transform profiles to the appendix and refer to them explicitly (for example, "see Appendix A").
- [ ] **Argue each modeling decision.** For every retained transformation, smooth, interaction, and final model, say what was compared, the evaluation criterion, what was selected, and why it supports both objectives. The instructor described this as practice for making a convincing applied-statistics case.
- [x] **Restore or replace the fitted-effects figure.** `fig-effects` is enabled, and `fig-st-effect` shows the selected partial $St$ effect with approximate 95% bands (`report.qmd` lines 233--301). Curate these during the four-page layout pass.
- [ ] **Keep the main report within four pages and verify the rendered PDF.** The appendix may contain supporting diagnostics, but body figures/tables must earn their space.

### 2. Transformations

- [x] **Check the skew of each moment separately.** See completed-items note above.
- [x] **Try Box-Cox separately for each response.** See completed-items note above.
- [x] **Justify the retained transform using skewness/linearity checks.** See completed-items note above.
- [ ] **Revalidate transformations after the final model changes.** The existing Box-Cox profiles use the current cubic-model predictors. If the spline/GAM or interaction structure changes materially, confirm that the chosen response transformations remain defensible and update the prose/appendix if needed.

### 3. Nonlinearity in $St$

- [x] **Replace or rigorously compare the cubic polynomial with splines.** LOOCV compares the cubic against natural splines with 3--5 df and common GAMs with $k=4,5,6$ (`report.qmd` lines 50--94 and 157--167).
- [x] **Compare the existing GAM with the cubic for all four moments.** The updated LOOCV table covers every response and shows each candidate's MSE (`report.qmd` lines 157--167).
- [x] **Tune spline flexibility by LOOCV.** The natural-spline df and GAM basis dimension are both searched by LOOCV; REML selects each GAM smoothing penalty inside a training fold. The common $k=5$ GAM wins on average (0.0872 vs. cubic 0.0879; `report.qmd` lines 152--155).
- [x] **Reconsider whether the smooth should be common or group-specific.** The final model deliberately replaces the prior group-specific smooth with `Re_f * Fr_f + s(log_St, k = 5)`, avoiding a separate curve per sparse group (`report.qmd` lines 69--94 and 150--155).
- [x] **Plot the selected $St$ effect with uncertainty.** `fig-st-effect` plots each final GAM partial smooth and an approximate 95% confidence band; the results text interprets the curvature (`report.qmd` lines 271--309).

### 4. GAM complexity and diagnostics

- [ ] **Recount and explain effective degrees of freedom.** The refit reports 11.7 EDF for the mean model (rather than the earlier roughly 27); report final EDF/model terms for every response and demonstrate that the fit is not overflexible.
- [ ] **Recheck residual misfit after the final model is selected.** The current report notes unequal residual spread for standard deviation and overprediction at the smallest $St$. Re-evaluate both after spline tuning and interaction selection; report any remaining limitation rather than hiding it.

### 5. Systematic interaction/effect selection

- [ ] **Select interactions systematically.** Fit a lasso (or comparably principled selection method) over all eligible main effects, nonlinear basis terms, and interactions, respecting factor coding and the missing design cell. Use it to identify which effects/interactions matter, rather than hand-selecting a few MSE comparisons.
- [ ] **Connect selection to all four responses.** The instructor’s inference goal is not only “are there interactions?” but also which predictors/effects matter for which of mean, SD, skewness, and kurtosis. Summarize response-specific selected effects before interpreting them scientifically.

### 6. Inference write-up

- [ ] **Interpret selected interactions physically.** Explain each supported interaction in plain scientific terms, such as how the gravity effect through $Fr$ changes with turbulence intensity $Re$.
- [ ] **Give the “why,” not only effect sizes.** Pair claims (for example, large changes in cluster size across $Re$) with a defensible physical explanation and the plot/contrast that supports it. The current gravity--turbulence paragraph begins this; the $Re$ and $St$ claims need equally clear support.
- [x] **Show uncertainty with key scientific effects.** `fig-st-effect` now provides approximate 95% bands for the selected $St$ partial effects (`report.qmd` lines 271--301). Other key contrasts still need their own inferential treatment.

### 7. Limitations and future design

- [x] **State the factor-level interpolation/extrapolation limitation.** See completed-items note above.
- [ ] **Make the future simulation-design recommendation concrete.** The report already advocates a non-grid design (line 278), but add that every simulated point has comparable cost and recommend random or space-filling sampling (for example, a Latin hypercube) over a denser full grid. Explain the scaling: moving from $3^3=27$ to $4^3=64$ grid points is costly and still restrictive.
- [x] **Plan for $Fr=\infty$ if $Fr$ becomes continuous.** See completed-items note above.

## Feedback reconciliation notes

The Zoom transcript was reviewed in full. All of its substantive feedback is represented above. The items most easily missed in the earlier checklist were:

- make the *specific* inference roadmap (nonlinearity, response-specific effects, interactions, and effect selection) explicit in the introduction;
- use the model-selection goal to choose which interactions/effects are important, not merely to compare one broad interaction model with one narrow one;
- explain the GAM’s actual structure and degrees of freedom rather than treating “GAM” as a black box;
- make the curved $St$ effect visible with uncertainty; and
- curate the four-page narrative rather than placing every diagnostic in the body.

Source files, if a fact must be checked: `sta325_casestudy.pdf` contains the assignment and report requirements; `Data_Expedition_F2026.pdf` contains the scientific/DNS/Voronoi background; `transcript` is the original feedback export (despite the expected name `transcript.txt`).
