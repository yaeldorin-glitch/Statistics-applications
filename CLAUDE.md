# Project context

Read this first. It is the handoff note for anyone — human or assistant — picking
this project up.

## What this is

A university course project for **"Statistics Applications" (יישומי סטטיסטיקה)**.
The assignment brief, **verbatim, exactly as given by the instructor** (this is the
literal text — do not paraphrase or reinterpret it):

> Project Preparation Guidelines:
>
> Create code notebooks demonstrating complete mastery of all the tools featured in
> the `pymc-1`, `pymc-2`, and `lifelines` notebooks.
> Select datasets different from those used in the original notebooks.
> For the `pymc` notebooks: Compare Bayesian problem-solving approaches side-by-side
> with classical approaches (as found in the `scipy-stats` and `statsmodels`
> notebooks).
> Record a video (or videos) of yourself walking through the notebook, explaining
> every detail thoroughly—except for basic programming concepts.
> This is an individual assignment.
> You are encouraged to work with an AI assistant. Include relevant questions
> within the notebook and review the answers you received during the video.
> Exercise critical judgment—it is your responsibility to verify the accuracy of
> the answers you obtain.
> Submit a text file containing links to your Google Drive files (Colab/Jupyter
> notebooks and recorded videos) via the submission portal.
> Do not submit large files; ask the AI how to compress videos.

## Repository layout

```
lecturer_original_notebooks/   <- the 5 original course reference notebooks (untouched)
    4_lifelines_survival.ipynb
    6_1_PyMC_bayesian_analysis_coins.ipynb
    6_2_PyMC_MCMC.ipynb
    Copy_of_2_Scipy_stats.ipynb
    Copy_of_3_statsmodels_OLS.ipynb

1_bayes_vs_frequentist/        <- new notebook 1 + its narration script
    1_bayes_vs_frequentist_school_pass_rates.ipynb
    script_1_bayes_vs_frequentist.md

2_bayes_vs_ols_tips/           <- new notebook 2 + its narration script
    2_bayes_vs_ols_restaurant_tips.ipynb
    script_2_bayes_vs_ols.md

3_survival_analysis/           <- new notebook 3 + its narration script
    3_survival_analysis_lung_cancer.ipynb
    script_3_survival_analysis.md
```

Each `script_*.md` is a **word-for-word narration script in Hebrew**, written to be
read aloud during recording, cell by cell, in the exact order the corresponding
notebook executes. It is *not* a summary — it is the actual spoken content,
including the "Chat, ..." questions posed to an AI assistant and how to explain the
answers, and explicit descriptions of every graph/table (axes, what they show, what
to conclude).

**Deliberate division of labor between notebook and script:** the notebooks themselves are kept
close to the lecturer's own minimal style — section headers, formulas, and the "Chat, ..."
question/answer pairs — with no analysis, conclusions, or debugging narratives written into
markdown cells. All of that (interpreting a plot, explaining *why* a result came out the way it
did, the debugging stories, the closing comparison tables) lives only in the matching script. When
the script covers a "Chat" answer, it says so explicitly ("...והצ'אט ענה לי ש...") rather than
quoting the answer as if it were the presenter's own conclusion.

## The three new notebooks, and how each maps to a reference notebook

### 1. `1_bayes_vs_frequentist/` — mirrors `6_1_PyMC_bayesian_analysis_coins.ipynb`

Dataset: `star98` (303 real California school districts, `NABOVE`/`NBELOW`
students per district on a reading test), bundled inside `statsmodels`. Same
underlying problem as the original coin-flip notebook (estimating an unknown
Binomial proportion), on real data instead of synthetic coin flips.

Tool coverage verified against the original by extracting every `pm.*`/`az.*`/
`stats.*` call from both notebooks and diffing the sets — full parity, with two
justified exceptions: `az.plot_posterior` was removed in the installed `arviz`
version and replaced by `az.plot_dist` (documented inline as a version note), and
`stats.bernoulli` (used to *generate* synthetic data in the original) isn't needed
since this notebook uses real recorded data throughout — that specific tool is
instead demonstrated in a coverage-probability simulation elsewhere in the same
notebook.

Beyond mirroring, this notebook adds a genuine classical-vs-Bayesian comparison for
comparing **two specific groups** (Fisher's exact test and chi-squared test on a
district-vs-district contingency table, set directly against the Bayesian
posterior difference of the two districts' rates) — pulled from the actual
`Copy_of_2_Scipy_stats.ipynb` reference notebook's own contingency-table section,
not improvised.

### 2. `2_bayes_vs_ols_tips/` — mirrors `6_2_PyMC_MCMC.ipynb`

Dataset: `tips` (244 real restaurant bills, bundled inside `seaborn`) instead of
the original's synthetic five-group Simpson's-paradox data. Same tool sequence:
basic Bayesian regression vs. `statsmodels` OLS, posterior-predictive HDI/PI bands,
the grid-approximation method, hierarchical (multilevel) regression with centered
vs. non-centered reparametrization, and MCMC diagnostics (trace, pair plots,
divergences, autocorrelation, ESS).

This notebook additionally compares the hierarchical Bayesian model against
**`statsmodels` `MixedLM`** (the classical analogue of a Bayesian hierarchical
model — not present in the original notebook at all, added specifically to make
the "compare to classical" requirement concrete for the multilevel-model section),
and against **classical ANOVA / Kruskal-Wallis / independent t-test / Mann-Whitney
U** for a simpler "do days differ" question, pulled directly from
`Copy_of_2_Scipy_stats.ipynb`'s own group-comparison section. A genuine finding is
documented in place rather than smoothed over: Bartlett's test shows ANOVA's
equal-variance assumption is violated on this data, and ANOVA and Kruskal-Wallis
disagree on significance as a direct result — the notebook explains why the
non-parametric result should be trusted more here, with the actual numbers.

### 3. `3_survival_analysis/` — mirrors `4_lifelines_survival.ipynb`

Dataset: `lung` (228 real NCCTG lung cancer patients, bundled inside `lifelines`)
instead of the original's Drosophila fly longevity experiment. Same tool sequence:
naive vs. censoring-aware MLE fitting (`ExponentialFitter`/`WeibullFitter`), the
naive-ECDF vs. Kaplan-Meier comparison, Nelson-Aalen cumulative hazard, log-rank
test, and a full multivariable `CoxPHFitter` model with `check_assumptions`. Two
real censoring biases are *quantified*, not just asserted, and both come from the
same root mechanism applied in two places: `scipy.stats.expon.fit()` has no concept
of censoring, so handing it the raw `time` column for all 228 patients silently
treats every censored patient's last-known-alive day as their actual death day —
this understates mean survival by ~28% relative to `lifelines`' censoring-aware
`ExponentialFitter`. The same shortcut applied to the empirical CDF/survival
function (instead of Kaplan-Meier) biases the naive survival curve in the
*opposite* direction. Neither case involves dropping censored patients — both
notebook and script describe this correctly; only this file's earlier summary
didn't.

## Working method used to build these (useful context, not something to repeat blindly)

- Every notebook was actually executed end-to-end (via `nbclient`/`ipykernel`),
  not hand-typed — all outputs, numbers, and plots in them are genuine.
- `pymc`/`arviz` changed several function signatures and removed some functions
  entirely between the version the lecturer wrote against and the version
  available now (`az.plot_posterior` removed → `az.plot_dist`; `az.compare(...,
  ic="waic")` removed → LOO is now the only option; `hdi_prob=` renamed to
  `prob=`/`ci_prob=` depending on the function; `az.plot_ess(..., kind="evolution")`
  removed → `kind="local"`; `az.summary()`'s default interval kind silently changed
  from HDI to equal-tailed, now requires explicit `ci_kind="hdi"`). `az.plot_trace()`
  also changed behavior in a way that isn't a removed argument but is easy to miss:
  it now renders **only** the sample-path panel, not the density panel the classic
  version showed alongside it — density is a separate `az.plot_dist()` call now.
  This was caught because notebook 1's independent and hierarchical models
  originally had no post-sampling diagnostic plot at all (only `az.summary()`'s
  `r_hat` column) — the lecturer's own notebook runs `plot_trace`+`plot_posterior`
  for its analogous models, so `plot_trace` (10 districts) and `plot_forest`
  (better suited than `plot_trace` for many parameters at once) were added for
  parity, which is when the single-panel behavior became visible. Each of these is
  called out explicitly, in place, in both the notebook and the matching script —
  this is deliberate and should not be "fixed" back to the old API/behavior without
  checking which `arviz` version is actually installed.
- Each notebook's first code cell pins exact package versions
  (`pip install -q pymc==6.3.1 arviz==1.3.0 ...` etc.) specifically so it behaves
  identically on Google Colab. **After running that cell, restarting the Colab
  runtime** before running the rest of the notebook is recommended as a safety
  measure, not a strict requirement: `pip install` always updates what's on disk,
  but anything already *imported* in the current kernel (which can include `numpy`,
  loaded transitively by Colab itself before the first user cell runs) stays at its
  old version in memory regardless, for the rest of that session — and `pytensor`
  (PyMC's backend) is specifically sensitive to a stale `numpy` since it JIT-compiles
  against it. On a genuinely fresh runtime this has been observed to work fine
  without restarting; restarting just removes the risk for the cost of one click.
  This is noted inline in each notebook and script.
- Every "Chat, ..." question in the notebooks is a real question that was actually
  asked and answered (direct, compound questions addressed to "Chat", sometimes
  referencing the code/output just above). Every question in notebooks 1 and 2 was
  individually checked against the corresponding lecturer original's own questions —
  topical overlap is expected and fine (both are demonstrating the same tool, so of
  course both ask about it), but a handful of questions that were near-verbatim
  reuses of the lecturer's own phrasing, or answerable from general dataset
  documentation without needing an AI at all (e.g. "what is the star98 dataset and
  is it a good fit"), were dropped as Chat exchanges and folded into plain narration
  in the script instead. Several remaining answers include an explicit
  self-correction — e.g. a plausible-sounding first guess about *which* district was
  causing a LOO diagnostic warning turned out to be wrong when actually checked, and
  the notebook shows the check and the corrected explanation rather than hiding the
  wrong guess. This is intentional: the assignment specifically asks students to
  demonstrate critical verification of AI-provided answers, not just present clean
  answers.

## What is NOT covered from the two classical reference notebooks

The assignment says the classical approaches "appear in" `scipy-stats` and
`statsmodels` — it does not require using literally every function in those two
notebooks, only demonstrating genuine, correct classical-vs-Bayesian comparisons
using that toolkit. For transparency, not used anywhere: the Kolmogorov-Smirnov
test, `stats.probplot` (a normal Q-Q plot via `sm.graphics.qqplot` is used
instead), `stats.norm.interval`/`stats.t.interval` for a mean's CI (Wald/Wilson/
bootstrap CIs for a *proportion* are used instead, which is the actual quantity
being estimated in notebook 1), `stats.false_discovery_control` (though the
multiple-comparisons problem itself is explicitly discussed as a caveat where
relevant), and from the `statsmodels` notebook: VIF, leverage/influence plots,
studentized residuals, the dummy-variable trap, and one-hot encoding (none of
these have a natural fit given this project's regression notebook uses a
hierarchical/mixed-effects structure rather than a dummy-coded flat OLS model).
If asked to fully replicate these too, they would need to be added as new
sections; ask before assuming they're wanted, since they'd need a design decision
about where they fit.

## What is left to do (the student's side, not code)

1. Record the three videos, reading each `script_*.md` word-for-word while
   stepping through the matching notebook on screen.
2. Compress the videos (ask for help with this if needed — the submission
   explicitly says not to submit large files).
3. Upload the notebooks (or just point to this GitHub repo) and videos to Google
   Drive, and write the final submission text file with the links.
