# BarrierLab — Exotic Option Pricing & Model Validation

A deployable portfolio demo for a continuously monitored **down-and-out European call**. The project converts the accompanying research notebook into a web application with a Next.js frontend and a Python/FastAPI pricing backend.

## What the demo does

- Prices a vanilla Black–Scholes call as an upper reference.
- Prices a zero-rebate down-and-out call with the closed-form Black–Scholes barrier formula.
- Prices the same barrier option with naïve discrete-monitoring Monte Carlo.
- Reprices with a Brownian-bridge continuous-monitoring correction.
- Reports Monte Carlo standard error and a 95% confidence interval.
- Computes finite-difference Delta, Gamma, Vega, and Rho using common random numbers.
- Runs automated implementation and economic sanity checks.
- Runs on-demand Monte Carlo convergence, monitoring-bias, and Greek bump-stability diagnostics.
- Separates **implementation risk** from **model specification risk** and identifies Heston stochastic volatility as the natural challenger model.

## Model scope

The analytic benchmark is intentionally restricted to the same product used in the notebook:

- down-and-out European call;
- zero rebate;
- continuous barrier monitoring;
- `H < K` and `H < S0`;
- Black–Scholes geometric Brownian motion with constant `r`, `q`, and `sigma`.

The API rejects inputs outside that analytic benchmark domain rather than silently changing formulas.

## Project structure

```text
barrier-option-vercel-demo/
├── app/
│   ├── globals.css        # Dashboard styling
│   ├── layout.tsx         # Next.js metadata/layout
│   └── page.tsx           # Interactive pricing + validation dashboard
├── api/
│   └── index.py           # FastAPI + pricing/validation engine
├── docs/
│   └── Barrier_Option_Pricing_Validation_Report.tex
├── reference/
│   └── Barrier_Option_Pricing.ipynb
├── tests/
│   └── test_model.py
├── package.json
├── pyproject.toml
└── tsconfig.json
```

## API

### `POST /api/price`

Example body:

```json
{
  "S0": 100,
  "K": 100,
  "H": 90,
  "T": 1,
  "r": 0.05,
  "q": 0,
  "sigma": 0.20,
  "n_paths": 30000,
  "n_steps": 52,
  "seed": 42
}
```

Returns analytic and Monte Carlo prices, confidence interval, Greeks, bias metrics, and validation checks.

### `POST /api/diagnostics`

Runs the heavier convergence, monitoring-frequency, and Greek bump-size studies. This is deliberately separate from the main pricing call to keep the UI responsive.

## Local checks

Run the Python unit tests:

```bash
python -m unittest discover -s tests -v
```

For full-stack local development, install Node dependencies and the Vercel CLI, then run:

```bash
npm install
npm install -g vercel
vercel dev
```

Vercel serves the Next.js frontend and the Python function under the same local origin.

## Deploy to Vercel

1. Put this folder in a Git repository and push it to GitHub, GitLab, or Bitbucket.
2. Import the repository into Vercel, or from the project root run:

```bash
vercel deploy --prod
```

3. Vercel detects the Next.js application from `package.json` and the FastAPI application from `api/index.py` plus `pyproject.toml`.
4. Verify:
   - `/` loads the dashboard.
   - `/api/health` returns `{"status":"ok"}`.

No `vercel.json` is required for this layout.

## Reference case

For the notebook/default parameters

```text
S0 = 100
K  = 100
H  = 90
T  = 1 year
r  = 5%
q  = 0%
sigma = 20%
```

the closed-form down-and-out call value is approximately:

```text
8.66547166
```

The exact Monte Carlo result changes with the selected path count and seed. The validation criterion therefore compares the Monte Carlo/analytic difference to the simulated standard error rather than requiring exact equality.

## Why this is framed as model validation

The application deliberately distinguishes three error sources:

1. **Sampling error** — finite Monte Carlo paths; measured with standard error and confidence intervals.
2. **Numerical implementation error** — for example, missed barrier crossings under discrete monitoring; investigated through Brownian-bridge correction and analytic benchmarking.
3. **Model specification risk** — constant volatility, continuous paths, no jump risk, and other Black–Scholes assumptions; this is not removed by increasing the number of paths.

The next extension should be an independent Heston stochastic-volatility challenger and a systematic `V_Heston - V_BS` model-risk study across barrier distance, maturity, spot-volatility correlation, and volatility of volatility.

## Disclaimer

Educational and portfolio use only. This is not a production trading, valuation, or investment system.
