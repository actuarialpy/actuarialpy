# OpenActuarial

A family of four small, composable Python libraries for actuarial and risk modeling —
experience analysis, loss-distribution modeling, Monte Carlo risk simulation, and
extreme-value tail estimation. Each does one thing with a clean API; together they cover
a full large-claim and pooling workflow.

**Documentation: https://openactuarial.github.io/docs/**

## Install

```bash
pip install actuarialpy lossmodels risksim extremeloss
```

Each package can also be installed and used on its own.

---

## [actuarialpy](https://github.com/openactuarial/actuarialpy-core)

Experience analysis on a single tidy table — you build one DataFrame and get views
without re-pivoting. numpy and pandas only (no scipy).

Loss ratios, PMPM / PEPM, frequency, severity, utilization; annualized and log-linear
trend with PMPM trend decomposition; credibility (Bühlmann, Bühlmann-Straub,
limited-fluctuation); completion and reserving (chain-ladder, completion factors, IBNR,
develop-to-ultimate); seasonality; and pooling with retained-CV primitives.

```python
import pandas as pd
import actuarialpy as ap

df = pd.DataFrame({
    "month": pd.period_range("2024-01", periods=6, freq="M").astype(str),
    "paid":    [120_000, 118_000, 125_000, 130_000, 128_000, 135_000],
    "premium": [150_000] * 6,
    "member_months": [1000, 1005, 1010, 1008, 1012, 1015],
})

exp = ap.Experience(df, expense="paid", revenue="premium",
                    exposure="member_months", date="month")

exp.loss_ratio                              # paid / premium
ap.pmpm(df["paid"], df["member_months"])    # per-member-per-month
```

---

## [lossmodels](https://github.com/openactuarial/lossmodels)

Severity and frequency distributions, maximum-likelihood fitting, goodness-of-fit, and
collective-risk (compound) models — the loss-distribution toolkit from the FAM/STAM
syllabus as composable objects.

Twenty-plus severity distributions and the standard frequency families (Poisson,
negative binomial, binomial, with zero-truncated/modified variants); MLE fitting and
model comparison by AIC/BIC, KS, and Anderson-Darling; spliced severities; and
collective-risk models with aggregate VaR/TVaR and stop-loss.

```python
import lossmodels as lm

sev = lm.Lognormal(mu=8.0, sigma=1.5)
sev.quantile(0.99)                     # 99th-percentile claim
sev.limited_expected_value(250_000)    # E[min(X, 250k)]

model = lm.CollectiveRiskModel(lm.Poisson(lam=3.0), sev)
model.mean(), model.var(0.99), model.tvar(0.99)
```

---

## [risksim](https://github.com/openactuarial/risksim)

Monte Carlo simulation of portfolios of risk models, with reinsurance contracts and
layers. Wrap any sampling model in a `PortfolioItem`, collect them into a `Portfolio`,
and simulate the aggregate — gross, retained, ceded, and by layer.

Aggregate layers and multi-layer contract programs; gross/retained/ceded decomposition;
per-component results; and exceedance probabilities and tail measures on the simulated
distribution.

```python
import lossmodels as lm
import risksim as rs

group = lm.CollectiveRiskModel(lm.Poisson(lam=3.0), lm.Lognormal(8.0, 1.5))
pf = rs.Portfolio([rs.PortfolioItem("group_a", group)])

result = pf.simulate(50_000)
result.mean()                          # expected aggregate loss
result.prob_exceeding(1_000_000)       # P(aggregate > 1M)
```

---

## [extremeloss](https://github.com/openactuarial/extremeloss)

Extreme-value theory for loss data: peaks-over-threshold (generalized Pareto) and
block-maxima (generalized extreme value) tail fitting, tail risk measures, rare-event
importance sampling, and bootstrap uncertainty.

GPD/POT and GEV fitting with threshold diagnostics; VaR/TVaR and return levels;
importance-sampling estimators for rare exceedances; bootstrap confidence intervals; and
optional bridges to `lossmodels` and `risksim`.

```python
import numpy as np
import extremeloss as el

losses = np.random.default_rng(0).pareto(2.0, 50_000) * 100_000

fit = el.fit_pot(losses, threshold=np.quantile(losses, 0.95))
fit.xi, fit.beta                       # GPD shape and scale
el.return_level(1000, fit)             # the 1-in-1000 return level
```

---

## How they fit together

A typical large-claim and pooling analysis uses all four: `extremeloss` fits one
book-wide tail, `lossmodels` turns a frequency and that severity into a collective-risk
model, `risksim` simulates the portfolio and applies reinsurance layers, and
`actuarialpy` handles the exposure, experience, and retained-stability side. See the
[overview](https://openactuarial.github.io/docs/overview/) for the full picture.

The libraries are general mathematics, kept public and reusable — they carry no data and
no proprietary conventions. The same call runs on a synthetic demo or on real claims;
only the input changes.

## License

[MIT](LICENSE) <!-- adjust if your license differs -->
