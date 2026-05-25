# Efficient Frontier & Factor Exposure-Based Stock Selection — STOXX 600

> Master's degree project — *N.F.*
> All rights reserved. See `LICENSE` for details.

---

## Overview

This project builds and evaluates a **systematic investment strategy** on the European equity universe of the STOXX 600. The core idea is that factor risk exposures — specifically market beta and momentum — can be used to classify stocks into homogeneous groups and construct portfolios with distinct risk/return profiles.

The strategy is built entirely on **in-sample data (2019–2022)** and evaluated on **out-of-sample data (2023–2025)**, ensuring a rigorous separation between the estimation and validation phases.

---

## Project Structure

```
.
├── project.ipynb     # Main Jupyter notebook (full analysis)
└── stoxx600.xlsx     # STOXX 600 constituent list (stock universe)
```

---

## Methodology

### 1. Data Collection
A random sample of **150 stocks** is drawn from the STOXX 600 universe (seed = 12345 for reproducibility). Monthly adjusted prices are downloaded via `yfinance` and cleaned to retain only stocks with a sufficiently long history. In parallel, **Carhart 4-factor data** (Market, SMB, HML, MOM) is retrieved via `fmfinance`.

### 2. In-Sample / Out-of-Sample Split

| Period | Dates |
|--------|-------|
| **In-Sample** | January 2019 → December 2022 |
| **Out-of-Sample** | January 2023 → December 2025 |

All model estimation and portfolio construction is performed exclusively on the in-sample period. The out-of-sample period is used solely for performance evaluation.

### 3. Carhart 4-Factor Regression
For each stock, a **Carhart 4-factor model** is estimated on the in-sample period via OLS:

$$r_i - r_f = \alpha_i + \beta^{Mkt}_i \cdot MKT + \beta^{SMB}_i \cdot SMB + \beta^{HML}_i \cdot HML + \beta^{MOM}_i \cdot MOM + \varepsilon_i$$

This yields, for each stock, a market beta (`β_Mkt`) and a momentum beta (`β_MOM`) — the two factor exposures used for portfolio construction. Stocks with negative market beta are filtered out.

### 4. Double Sequential Sort
Stocks are classified through a **two-step sorting procedure**:

- **Step 1:** Sort all stocks by `β_Mkt` → split into terciles (low / mid / high market beta)
- **Step 2:** Within the low and high terciles, sort by `β_MOM` → split into quartiles (low / high momentum)

This produces **four groups** with distinct and internally homogeneous factor profiles.

### 5. Portfolio Selection and Tangency Portfolio
The two most contrasting groups are selected for comparison:

| Portfolio | Factor Profile |
|-----------|---------------|
| **Portfolio 1** | Low Beta / Low Momentum |
| **Portfolio 2** | High Beta / High Momentum |

For each group, a **Tangency Portfolio** (maximum Sharpe ratio) is constructed using `PyPortfolioOpt`'s Efficient Frontier, based entirely on in-sample returns and covariances.

### 6. Out-of-Sample Evaluation
The in-sample weights are applied to out-of-sample returns. Both portfolios are benchmarked against the **STOXX 600 ETF (EXSA.DE)**. Performance is evaluated using `quantstats` (`periods_per_year=12` for monthly data).

**Metrics reported:** Cumulative return, CAGR, Sharpe ratio, Sortino ratio, Max Drawdown, Volatility.

---

## Requirements

```bash
pip install numpy pandas matplotlib seaborn yfinance fmfinance quantstats \
            statsmodels scipy pypfopt openpyxl
```

Python 3.9+ recommended. Run in Jupyter Notebook or JupyterLab.

---

## Usage

1. Clone the repository
2. Make sure `stoxx600.xlsx` is in the same folder as `project.ipynb`
3. Open and run `project.ipynb` sequentially from top to bottom

---

## License

This project is protected by copyright. See the [`LICENSE`](LICENSE) file for full terms.
**Reusing, copying, modifying, or redistributing** the code or data without explicit written permission from the author is not permitted.
