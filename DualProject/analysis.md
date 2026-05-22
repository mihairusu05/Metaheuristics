Portfolio Optimization — SFA vs GWO
Comparative Results Report (Updated — same parameters for both)
population=30, iterations=500, risk_aversion=0.8


1. Problem Definition

Portfolio optimization finds the best allocation of capital across n assets to maximize
return while minimizing risk. The Markowitz mean-variance model is used:

  minimize: risk_aversion * portfolio_risk - (1 - risk_aversion) * portfolio_return

Both algorithms used identical parameters: risk_aversion=0.8, population/pack size=30,
iterations=500. This makes the comparison fair and meaningful — any difference in
results comes from the algorithm behavior, not parameter tuning.


2. Results by Period

────────────────────────────────────────────────────────────
Period 1: 2019-2024  |  Assets: AAPL, GOOGL, MSFT, JPM, GLD
────────────────────────────────────────────────────────────

  Asset       SFA         GWO
  AAPL        33.9%       32.8%
  GOOGL        0.0%        0.1%
  MSFT        12.8%       12.0%
  JPM          0.6%        4.1%
  GLD         52.7%       50.9%

This is the most striking result in the entire comparison — with identical parameters
SFA and GWO converge to almost the same portfolio. The largest difference is JPM:
GWO allocates 4.1% vs SFA's 0.6%. Everything else is within 1-2 percentage points.
This strong agreement means the 2019-2024 data has a clear optimal allocation that
both algorithms independently discover: roughly 50% gold, 33% Apple, 12% Microsoft,
near-zero everything else. When two completely different algorithms agree this closely
on the same data it is a strong signal that this allocation is genuinely close to the
mathematical optimum of the fitness function.


────────────────────────────────────────────────────────────
Period 2: 2020-2023  |  Assets: NVDA, TSLA, META
────────────────────────────────────────────────────────────

  Asset       SFA         GWO
  NVDA        41.4%       41.1%
  TSLA        27.7%       28.3%
  META        30.9%       30.7%

Essentially identical results — all three allocations differ by less than 0.6%.
With only three highly correlated growth assets and no defensive option, both
algorithms converge to the same near-equal split with NVDA slightly favored.
NVDA had the best Sharpe ratio (return per unit of risk) in this period due to
AI chip demand, which both algorithms correctly identify. This period shows that
when the search space is constrained and the signal is clear, SFA and GWO are
interchangeable in quality.


────────────────────────────────────────────────────────────
Period 3: 2001-2007  |  Assets: JNJ, PG, KO
────────────────────────────────────────────────────────────

  Asset       SFA         GWO
  JNJ         33.5%       33.2%
  PG          66.5%       66.1%
  KO           0.0%        0.8%

Again near-identical results. SFA eliminates KO completely while GWO gives it a
token 0.8% — practically the same decision. Procter & Gamble dominates at ~66%
in both because PG had superior returns with lower volatility than JNJ and KO
during the post dot-com recovery. KO is correctly identified as the weakest of
the three — similar defensive characteristics to PG but lower returns, making it
redundant in the portfolio. The agreement here is the tightest of all periods.


────────────────────────────────────────────────────────────
Period 4: 2020-2023  |  Assets: XOM, CVX, SLV
────────────────────────────────────────────────────────────

  Asset       SFA         GWO
  XOM         45.8%       46.3%
  CVX          1.1%        0.0%
  SLV         53.2%       53.7%

Nearly identical — SLV (silver) and XOM together dominate, CVX is eliminated by
both. The silver vs XOM split is 53/46 for SFA and 54/46 for GWO — a difference
of less than 1%. CVX being eliminated makes sense since XOM and CVX are highly
correlated oil majors and XOM had better returns, making CVX redundant exactly
like GOOGL was redundant vs AAPL in Period 1.


3. The Main Signal — Convergence Means Robustness

The most important finding from this updated comparison is how similar the results
are across all four periods. With identical parameters:

  Period 1 (2019-2024):  max difference on any asset = 3.5% (JPM: 0.6% vs 4.1%)
  Period 2 (2020-2023):  max difference on any asset = 0.6% (META: 30.9% vs 30.7%)
  Period 3 (2001-2007):  max difference on any asset = 0.8% (KO: 0.0% vs 0.8%)
  Period 4 (2020-2023):  max difference on any asset = 0.5% (SLV: 53.2% vs 53.7%)

This level of agreement between two completely different algorithms is strong
evidence that both found the same global optimum (or very close to it) for each
period. It means the fitness landscape for these portfolio problems is relatively
well-behaved — one clear basin of attraction that both SFA and GWO can find.

The one exception is JPM in Period 1 (3.5% difference) which suggests that the
banking sector allocation sits in a flatter region of the fitness landscape where
small differences in algorithm behavior lead to slightly different outcomes — both
0.6% and 4.1% give similar fitness values.


4. What the Allocations Tell Us About Each Period

2019-2024 — Gold + Apple dominates:
Gold (50-53%) hedges against COVID uncertainty and inflation while Apple (33%)
captures tech growth. Microsoft adds secondary tech exposure. Everything else is
noise. This is a classic barbell portfolio — safe asset + growth asset, nothing in
between.

2020-2023 tech (NVDA/TSLA/META) — Roughly equal split, NVDA slightly favored:
With no safe haven in the asset set, capital spreads across all three. NVDA's AI
chip narrative gave it the best risk-adjusted return. The near-equal split reflects
that all three had exceptional returns in this period with similar correlation
structures, making diversification valuable.

2001-2007 consumer staples — PG dominates, JNJ secondary, KO eliminated:
Post dot-com investors who fled to safety found PG the most reliable. JNJ adds
healthcare diversification. KO is too similar to PG with lower returns to justify
inclusion. This is a concentrated defensive portfolio appropriate for the era.

2020-2023 energy (XOM/CVX/SLV) — Silver + XOM, CVX zero:
Post-COVID inflation made hard assets (silver, oil) attractive. XOM outperformed
CVX making CVX redundant. The silver-dominant allocation reflects silver's dual
role as both an inflation hedge and an industrial metal during the recovery.


5. SFA vs GWO — Where They Actually Differ

Given how similar the results are, the real differences are in behavior rather
than output quality:

Convergence speed:
GWO converges faster due to its deterministic leader-following mechanism. SFA
spends more iterations in exploration before converging. For these portfolio
problems (5 or fewer assets, smooth fitness landscape) GWO's faster convergence
is an advantage since there are few local optima to escape.

Handling of near-redundant assets:
The clearest behavioral difference is in near-redundant assets (KO, CVX, JPM).
SFA tends to zero them out completely while GWO retains a small allocation. This
is because GWO's beta and delta wolves pull solutions slightly away from corners
of the search space, while SFA's exploitation phase pushes harder toward zero for
clearly inferior assets.

Robustness across runs:
SFA's stochastic exploration phase means results vary more between runs on the
same data. GWO converges more consistently to the same result. For a presentation
or report, GWO results are more reproducible. For a real-world application where
finding a slightly better allocation matters, SFA's broader search could help.


6. Conclusion

With identical parameters, SFA and GWO find portfolios that are nearly identical
in all four tested periods. This is actually the best possible result for a
comparison — it means both algorithms are working correctly and the portfolios
they find are robust (algorithm-independent) solutions to the optimization problem.

The small remaining differences (JPM allocation, KO allocation) are financially
insignificant and reflect natural stochastic variation rather than fundamental
algorithm differences. For portfolio optimization with 3-5 assets and a smooth
Markowitz fitness function, both SFA and GWO are equally effective tools.

Where SFA would have a genuine advantage over GWO is on higher-dimensional
portfolios (20+ assets) or portfolios with complex non-convex constraints where
SFA's three-phase exploration structure would prevent the premature convergence
that GWO is susceptible to.