# NIFTY 50 Next-Day Direction Prediction

Small project trying to predict whether NIFTY 50 goes up or down the next day, using a proper
setup (no future data leaking into training, real trading costs factored in, honest results
either way).

## Why direction, not the actual price

Guessing the exact next price of an index like NIFTY 50 from just its own history is basically
not realistic — too many people are watching it, too much money is already arbitraging it. What
you can actually go for is a decent guess on direction with some confidence attached to it,
which is closer to how trading desks actually think about positions anyway.

## What's in here

```
.
├── NIFTY50_Direction_Prediction.ipynb   # main notebook, just run it top to bottom
├── figures/                             # every chart gets saved here automatically
│   ├── 01_price_trend.png
│   ├── 02_returns_distribution.png
│   ├── 03_rolling_volatility.png
│   ├── 04_volume_vs_abs_return.png
│   ├── 05_day_of_week_effect.png
│   ├── 06_autocorrelation.png
│   ├── 07_confusion_matrix.png
│   ├── 08_strategy_vs_buyhold.png
│   ├── 09_feature_importance.png
│   └── 10_strategy_comparison.png
└── README.md
```

**About the figures in this folder right now:** they were made from a fake/placeholder price
series, not real NIFTY data, just so there'd be something to look at immediately (the environment
I built this in doesn't have internet access to pull real market data). The first cell of the
notebook pulls real NIFTY 50 data the moment you run it in Colab or Jupyter with internet — just
run it and the figures folder updates with the real charts, same names, same layout.

## How to run it

1. Open `NIFTY50_Direction_Prediction.ipynb` in Google Colab (easiest option) or local Jupyter.
2. Run all cells top to bottom. Colab already has everything installed; locally you'll need
   `pip install yfinance scikit-learn xgboost seaborn scipy pandas numpy`.
3. It pulls NIFTY 50 (`^NSEI`) plus S&P 500, India VIX, and USD/INR from Yahoo Finance — needs
   internet.
4. Charts show up inline and also get saved to `figures/`.

## What the notebook actually does

**Data:** NIFTY 50 daily prices since 2015, plus three extra series that might actually help:
prior day's S&P 500 return (US market closes a few hours before NIFTY opens next day, so it's
real info, not circular), India VIX, and USD/INR.

**EDA:** price trend, return distribution (it's not normal, has fat tails), rolling volatility,
volume vs. returns, day-of-week patterns, and autocorrelation of returns — that last one basically
shows why this whole problem is hard in the first place.

**Features:** lagged returns, rolling averages/std, RSI, MACD, Bollinger Bands, distance from
moving averages, volume z-score, day of week, plus the three extra series above. Everything only
uses information available up to that point in time — nothing from the future leaks in.

**Models:** Logistic Regression, Random Forest, and XGBoost, compared using `TimeSeriesSplit`
instead of normal cross-validation, since normal CV would shuffle future data into training and
make everything look artificially good.

**Two backtests:**
- A simple long/flat version — one model trained on the first 80% of data, tested on the last
  20%. Simple, but it can never make money on a down day since it just goes to cash.
- A better version — retrains every ~3 months, and instead of just "in or out," sizes the
  position based on how confident the model is (can go short too, not just flat).

Both get compared to buy-and-hold using Sharpe ratio, Sortino ratio, max drawdown, Calmar ratio,
win rate, and number of trades, all after subtracting a small cost per trade.

## How to actually read the results

- An AUC around 0.52-0.55 for next-day direction is what you'd realistically expect. It's better
  than a coin flip but not by much — and that's fine, it means nothing's obviously broken or
  leaking. If a simple model could call daily index moves with high accuracy, that edge wouldn't
  exist for long.
- The better (long/short, retrained) version is the stronger design, but it's not guaranteed to
  beat buy-and-hold — especially through a strong bull run, where just holding the index beats
  almost anything. Whatever number you get is the number to report.
- Don't keep tweaking thresholds and parameters until the backtest beats buy-and-hold on this one
  stretch of history — that's basically how people accidentally build a backtest that looks great
  and means nothing. A smaller, honestly reported edge is worth more than a suspiciously perfect
  equity curve.

## Things worth trying next

- Only trade when volatility is in a certain range instead of all the time.
- Size positions based on an actual risk budget instead of a made-up scaling number.
- Add oil prices or US bond yields as features, since the role this is for also touches
  commodities and fixed income, not just equities.
- Add event-day flags for things like RBI policy announcements or big US data releases.

## One disclaimer

This is a learning/research project, not investment advice, and definitely not a production
trading system. Costs and slippage are simplified. A backtest doing well doesn't mean it'll
keep doing well.
