# actuarialpy

[![PyPI](https://img.shields.io/pypi/v/actuarialpy)](https://pypi.org/project/actuarialpy/)
[![Python](https://img.shields.io/pypi/pyversions/actuarialpy)](https://pypi.org/project/actuarialpy/)
[![License](https://img.shields.io/pypi/l/actuarialpy)](https://pypi.org/project/actuarialpy/)

Experience analysis on a single tidy table — loss ratios, PMPM, trend, credibility,
completion/reserving, seasonality, and pooling — with a clean API and no heavy
dependencies (numpy and pandas only, no scipy).

**Documentation: https://actuarialpy.github.io/docs/**

## Install

```bash
pip install actuarialpy
```

## Quickstart

```python
import pandas as pd
import actuarialpy as ap

df = pd.DataFrame({
    "month": pd.period_range("2024-01", periods=6, freq="M").astype(str),
    "product": ["PPO"] * 6,
    "paid":    [120_000, 118_000, 125_000, 130_000, 128_000, 135_000],
    "premium": [150_000] * 6,
    "member_months": [1000, 1005, 1010, 1008, 1012, 1015],
})

exp = ap.Experience(df, expense="paid", revenue="premium",
                    exposure="member_months", date="month")

exp.by("product")                          # grouped view
exp.loss_ratio                             # paid / premium

ap.pmpm(df["paid"], df["member_months"])   # per-member-per-month
```

## What it does

You build one tidy DataFrame — claims/expense, revenue, exposure, by period — and
`Experience` returns views without re-pivoting:

- ratios and rates: loss ratio, PMPM / PEPM, frequency, severity, utilization
- trend: annualized trend, log-linear fits, PMPM trend decomposition
- credibility: Bühlmann, Bühlmann-Straub, limited-fluctuation
- completion and reserving: chain-ladder, completion factors, IBNR, develop-to-ultimate
- seasonality: seasonal factors, deseasonalization
- pooling and retention: excess-over-threshold, retained CV, size-graded retention

## Part of the actuarialpy family

Four small, composable libraries that also work together:

- [**actuarialpy**](https://github.com/actuarialpy/actuarialpy) — experience analysis (this package)
- [**lossmodels**](https://github.com/actuarialpy/lossmodels) — severity / frequency distributions and collective-risk models
- [**risksim**](https://github.com/actuarialpy/risksim) — Monte Carlo portfolio simulation with reinsurance layers
- [**extremeloss**](https://github.com/actuarialpy/extremeloss) — extreme-value theory and tail estimation

See [how they fit together](https://actuarialpy.github.io/docs/overview/) in the documentation.

## License

[MIT](LICENSE) <!-- adjust if your license differs -->
