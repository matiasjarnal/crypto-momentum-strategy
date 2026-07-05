# Cross-Sectional Crypto Momentum Strategy

A systematic long/short cryptocurrency strategy built in Python. Uses cross-sectional momentum signals, Bitcoin regime conditioning, and realistic transaction cost modeling to generate market-neutral alpha across a 35-asset crypto universe.

---

## Strategy Overview

This is a **cross-sectional relative-strength strategy**: at each rebalance, assets are ranked by momentum signals, and the portfolio goes long the top-ranked and short the bottom-ranked, scaled by regime and volatility conditions.

### Key Design Decisions

| Component | Detail |
|---|---|
| Universe | 35 liquid cryptos (BTC, ETH, SOL, BNB, XRP, and 30+ alts) |
| Signal | Multi-horizon momentum (10/20/60-day lookbacks), ensemble of top 3 specs selected in-sample |
| Regime | Bitcoin 200-day MA regime: bull/bear determines position sizing, gross exposure, and signal weighting |
| Weighting | Softmax-normalized within basket; inverse-volatility weighting across long/short legs |
| Volatility targeting | Portfolio-level vol target (20% annualized) with max 3× leverage |
| Transaction costs | 20 bps flat per trade (7 bps commission + 13 bps slippage) |
| Eligibility screen | Rolling 60-day median ADV ≥ $5M; minimum 365 days of price history |

---

## Methodology

### Signal Construction
- Momentum computed as compound return over rolling lookback windows (10, 20, 60 days)
- Cross-sectional z-score with winsorization at ±3σ to suppress outlier influence
- Ensemble: top 3 parameter specifications selected on in-sample Sharpe; equal-weighted combination in out-of-sample

### Bitcoin Regime Conditioning
- Bull regime: BTC price > 200-day MA → full gross exposure (1.0×), momentum-tilted weighting
- Bear regime: BTC price < 200-day MA → reduced gross exposure (0.5×), low-volatility-tilted weighting, top-K holdings cut by 30%

### Portfolio Construction
- Softmax normalization of signal scores to smooth weight concentration
- Intra-basket inverse-volatility weighting (30-day lookback)
- Exponential smoothing of weights (α = 0.25) to reduce turnover-driven costs
- Volatility targeting: scale leverage daily to hit 20% annualized portfolio vol, capped at 3×

### Backtesting Framework
- **Train period**: 2018–2022 (in-sample parameter selection)
- **Test period**: 2023–2024 (out-of-sample evaluation)
- Walk-forward validation across 2023 and 2024 independently
- Performance attribution vs. BTC buy-and-hold and equal-weight basket benchmarks
- Rolling 90-day beta to BTC and equal-weight basket (market neutrality check)

---

## Results

### Out-of-Sample Performance (2023–2024)

| Metric | Value |
|---|---|
| Annualized Return | 24.1% |
| Annualized Volatility | 21.5% |
| Sharpe Ratio | 1.12 |
| Max Drawdown | -25.3% |
| Annualized Alpha vs. BTC | 16–18% |
| Alpha t-statistic | ~2.6 |
| Beta vs. BTC | ~0.32 |
| Beta vs. EW Basket | ~0.29 |
| Avg Turnover per Rebalance | ~5.8% |
| Transaction Cost Assumption | 20 bps/trade (7 bps commission + 13 bps slippage) |

The strategy generates statistically significant alpha (t-stat > 2.6) with low beta to both Bitcoin and the equal-weight crypto basket, confirming that returns are driven by cross-sectional relative momentum rather than directional market exposure.

### In-Sample Performance (2018–2022)

| Metric | Value |
|---|---|
| Annualized Return | 24.4% |
| Annualized Volatility | 20.9% |
| Sharpe Ratio | 1.17 |
| Max Drawdown | -31.5% |

The near-identical Sharpe ratios in train (1.17) and test (1.12) indicate minimal overfitting and strong parameter stability across regimes.

### Best Ensemble Specifications (selected in-sample)

| Momentum Lookback | Volatility Filter | Top-K Assets |
|---|---|---|
| 20 days | 60-day vol | 10 |
| 20 days | 30-day vol | 10 |
| 10 days | 30-day vol | 10 |

---

## Code Structure

```
crypto_momentum_strategy.ipynb
│
├── 0) Global Settings          # All parameters in one place (universe, signal grid, costs)
├── 1) Data                     # yfinance download, OHLCV parsing, return computation
├── 2) Eligibility              # Dynamic ADV screen, price history filter
├── 3) Metrics                  # Sharpe, Sortino, CVaR, alpha/beta, rolling beta
├── 4) Factors & CS Helpers     # Rolling returns, z-scoring, BTC beta estimation
├── 5) Signal Construction      # Multi-horizon momentum, regime-adjusted signal
├── 6) Weighting                # Softmax, inverse-vol, weight smoothing
├── 7) Portfolio Construction   # Vol targeting, leverage scaling, exposure limits
├── 8) Backtest Engine          # Daily P&L, cost deduction, turnover calculation
├── 9) Train Selection          # Grid search over signal specs, ensemble selection
└── 10) Run & Visualize         # Train/test split, equity curves, rolling betas
```

---

## Dependencies

```
numpy
pandas
yfinance
matplotlib
statsmodels
```

Install with:
```bash
pip install numpy pandas yfinance matplotlib statsmodels
```

---

## How to Run

```bash
git clone https://github.com/matiasjarnal/crypto-momentum-strategy
cd crypto-momentum-strategy
pip install numpy pandas yfinance matplotlib statsmodels
jupyter notebook crypto_momentum_strategy.ipynb
```

The notebook downloads data directly from Yahoo Finance on run. All parameters are configurable in the **Global Settings** block at the top.

---

## Author

**Matias Arnal** — Quantitative Economist  
[LinkedIn](https://linkedin.com/in/matiasarnal)

Built as part of a systematic strategy research portfolio alongside 15+ years of institutional quantitative modeling at the World Bank, IMF, and U.S. Department of Energy.
