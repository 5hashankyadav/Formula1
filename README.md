# 🏎️ Pit Wall — F1 Podium Intelligence & Race Analytics

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Pit%20Wall%20App-FF1801?style=for-the-badge&logo=formula1&logoColor=white)](http://mithil-f1-pitwall.duckdns.org/)
[![Python](https://img.shields.io/badge/Python-3.12%2B-blue?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastF1](https://img.shields.io/badge/Data-FastF1%20API-FF1801?style=for-the-badge)](https://docs.fastf1.dev)
[![Scikit-Learn](https://img.shields.io/badge/ML-Scikit--Learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

> 🚀 **Live Deployment**: Access the live, self-updating production dashboard at:  
> **👉 [http://mithil-f1-pitwall.duckdns.org/](http://mithil-f1-pitwall.duckdns.org/)**

**Pit Wall** is an end-to-end Formula 1 machine learning pipeline and telemetry analytics platform built by [**Shashank Yadav**](https://github.com/5hashankyadav). It predicts Grand Prix podium finishes (top 3) from qualifying pace and rolling driver/constructor form using real-time and historical session data (2024–2026) fetched via the [FastF1](https://docs.fastf1.dev) API.

The project ships as both a robust command-line CLI (`f1ml`) and an interactive, dark-mode web dashboard inspired by official F1 broadcast timing screens, complete with an in-browser streaming terminal console.

---

## 🌐 Live Demo & Preview

| Production Deployment | Repository |
| :--- | :--- |
| 🔗 **[http://mithil-f1-pitwall.duckdns.org/](http://mithil-f1-pitwall.duckdns.org/)** | 🐙 **[github.com/5hashankyadav/Formula1](https://github.com/5hashankyadav/Formula1)** |

The live application runs 24/7 with automatic daily background refresh cycles to ingest newly concluded Grand Prix sessions, rebuild features, and retrain the models autonomously.

---

## ⚡ Key Highlights & Capabilities

- 🔮 **Probabilistic Podium Prediction**: Calibrated probability distributions over the entire grid, trained using time-series walk-forward validation (achieving a **~74% top-3 hit rate in 2025**).
- ⏱️ **Telemetry Duels & Track Dominance**: Interactive high-frequency telemetry comparisons (speed, throttle, brake, gear, RPM, time delta) and GPS circuit track-dominance heatmaps between any two drivers.
- 🏁 **Computer Vision Racing Line Optimizer**: Solves the globally-optimal minimum-curvature racing line from schematic track maps, identifies corner apexes (early, geometric, late), and estimates lap times using a physics-informed downforce grip model.
- 📊 **Strategy & Tyre Stint Analysis**: Real-time pit strategy visualizer, tyre compound lifespans, and clean-air pace calculations (gap to fastest median lap).
- 🏆 **F1 Fantasy Lineup Optimizer**: Algorithmic knapsack solver that computes optimal driver and constructor combinations given dynamic budget constraints.
- 🖥️ **Integrated Web Terminal**: Stream CLI commands directly through the browser console tab with live output streaming.
- 🔄 **Autonomous Continuous Learning**: Automated background scheduler pulls latest race classifications and retrains the model without service interruption.

---

## 🧠 Engineering & ML Methodology

Podium prediction in Formula 1 is inherently imbalanced (~15% of entries reach the podium). A naive model predicting "no podium" every race achieves ~85% accuracy while being completely useless.

In developing this system, I designed the pipeline around rigorous data integrity and realistic metrics:

1. **Leakage-Safe Feature Engineering**:
   - Rolling form metrics are computed strictly per driver across past rounds—never rolled over arbitrary round-sorted frames.
   - Season statistics reset per calendar year and are lagged by one event.
   - Qualifying times and starting grid positions (known legitimately before lights-out) serve as primary signals alongside rolling team form, qualifying gap-to-pole, and circuit classifications.
   - Clean data handling: DNS/unclassified entries are removed rather than arbitrarily padded as artificial P20 finishes. Driver alias normalizations ensure continuous form tracking.

2. **Honest Evaluation & Benchmarking**:
   - Evaluated on **PR-AUC**, **ROC-AUC**, per-class precision/recall, **Brier score**, and a **top-3-per-race hit rate**.
   - Benchmarked directly against the hardest baseline in Formula 1: *picking the top 3 qualifiers*.
   - Evaluated via strict walk-forward temporal splits (retraining strictly on past events before forecasting each subsequent round).

3. **Model Ensembles**:
   - Scikit-learn `HistGradientBoostingClassifier` and `RandomForestClassifier` with balanced class weighting and calibrated probabilities.

---

## 🖥️ Dashboard Views

- **Dashboard** — Current championship leader, season progress tracker, last-race podium, and clickable event calendar.
- **Weekend Hub** — Dynamic countdown to the next Grand Prix, session schedules in local time, circuit profile auto-traced from pole telemetry, and venue-specific historical projections.
- **Races & Standings** — Full classifications, grid-to-finish delta gains, and live driver/constructor championship tables.
- **Telemetry Duel** — Overlay speed, throttle, and brake telemetry channels for any two drivers, with delta timers and track dominance map.
- **Strategy & Tyre Stints** — Stint degradation, tyre compound choices, and clean-air race pace metrics.
- **Model Analysis** — Real-time performance metrics, calibration curves, feature importance ranking, and walk-forward backtest grids.
- **Podium Predictor** — On-demand single-race scenario simulator with predicted probabilities vs. actual results.
- **F1 Fantasy** — Budget-constrained solver maximizing projected fantasy points.
- **Racing Line Analyzer** — Upload track loop schematics to extract optimal corner paths and simulated lap times.
- **Console** — Full interactive access to the `f1ml` tool suite inside the web interface.

---

## 🚀 Quick Start

### 1. Easiest: Local Script or Make

Clone the repository and launch directly:

```bash
git clone https://github.com/5hashankyadav/Formula1.git
cd Formula1

# Option A: One-line bootstrap (uses uv for fast virtualenv setup)
./run.sh

# Option B: Using Make
make serve
```

Then open **[http://127.0.0.1:5173](http://127.0.0.1:5173)** in your browser. On first boot, the app will automatically build the feature tables and train models from offline data.

### 2. Run with Docker

```bash
docker build -t pitwall .
docker run --rm -p 5173:5173 pitwall
```

---

## 🛠️ The `f1ml` CLI Reference

The project includes an extensible command-line interface `f1ml`:

```bash
# Activate environment or prefix with uv run
uv run f1ml <command>
```

| Command | Description |
|---|---|
| `f1ml info` | Display current dataset status, cached rounds, and model artifacts |
| `f1ml fetch --year 2025` | Download race and qualifying session data via FastF1 |
| `f1ml merge --year 2025` | Merge raw qualifying and race telemetry into processed datasets |
| `f1ml features` | Generate the leakage-safe model training feature table |
| `f1ml train [--model all]` | Train and evaluate models on time-split historical rounds |
| `f1ml evaluate [--model rf]` | View detailed precision/recall, PR-AUC, and race hit rates |
| `f1ml predict --year 2025 --round 23` | Generate calibrated podium probabilities for a specific GP |
| `f1ml backtest --year 2025` | Run a walk-forward temporal backtest across an entire season |
| `f1ml standings --year 2025` | Calculate driver and constructor championship standings |
| `f1ml weekend` | Generate projection & schedule for the upcoming race weekend |
| `f1ml fantasy [--budget 100]` | Optimize F1 Fantasy team lineup for best point yield |
| `f1ml h2h --year 2024 --round 8 --d1 VER --d2 LEC` | Run head-to-head telemetry analysis between two drivers |
| `f1ml raceline --image track.png` | Compute minimum-curvature racing line from track schematic |
| `f1ml refresh` | Automated one-shot update: fetch latest races, rebuild, and retrain |
| `f1ml serve [--port 5173]` | Start the web application and API server |

---

## 📁 Repository Structure

```
Formula1/
├── src/f1_ml/              # Core Python ML & analytics package
│   ├── config.py           # Project paths and environment configuration
│   ├── data.py             # FastF1 data ingestion, cleaning, and standings
│   ├── features.py         # Leakage-safe feature engineering pipeline
│   ├── modeling.py         # Model training, calibration, and artifact exports
│   ├── evaluate.py         # Metrics (PR-AUC, ROC-AUC, Brier score, top-3 hit rate)
│   ├── predict.py          # Single-race probabilistic inference
│   ├── backtest.py         # Walk-forward season backtest harness
│   ├── telemetry.py        # FastF1 high-frequency telemetry & stint analysis
│   ├── weekend.py          # Upcoming race-weekend hub & venue projection
│   ├── fantasy.py          # F1 Fantasy knapsack lineup optimizer
│   ├── racingline.py       # Computer vision racing line solver (OpenCV + SciPy)
│   ├── cli.py              # 'f1ml' command line interface
│   ├── server.py           # Flask REST API + streaming terminal backend
│   ├── pipeline.py         # End-to-end data update and retraining workflow
│   └── snapshot.py         # Static site renderer for GitHub Pages deployment
├── web/                    # Modern vanilla JS/CSS dashboard frontend
│   ├── index.html          # Timing-tower UI layout
│   ├── styles.css          # F1 broadcast dark styling
│   ├── app.js              # Application logic and routing
│   ├── charts.js           # Interactive Canvas/SVG telemetry graphs
│   ├── telemetry.js        # Driver duel comparison visualizer
│   ├── racingline.js       # Track map apex and racing line visualizer
│   └── static-shim.js      # Backend-less adapter for static deployments
├── data/                   # Bundled raw and processed historical race CSVs
├── notebooks/              # Research and exploratory data analysis notebooks
├── tests/                  # Pytest test suite (leakage, feature sanity, CLI)
├── e2e/                    # Playwright end-to-end frontend tests
├── Dockerfile              # Multi-stage production container build
├── pyproject.toml          # Package configuration & dependencies
└── README.md
```

---

## 🧪 Testing

The codebase maintains strict automated test coverage across both the Python ML core and the interactive frontend:

```bash
# Run backend test suite (unit tests, leakage guards, metrics)
uv run pytest

# Run frontend Playwright smoke tests
npm ci
npx playwright test
```

---

## 👤 Author & Contact

**Shashank Yadav**

- **GitHub**: [@5hashankyadav](https://github.com/5hashankyadav)
- **Repository**: [https://github.com/5hashankyadav/Formula1](https://github.com/5hashankyadav/Formula1)
- **Live Application**: [http://mithil-f1-pitwall.duckdns.org/](http://mithil-f1-pitwall.duckdns.org/)

---

## 📄 License

This project is open-source and distributed under the [MIT License](LICENSE).
