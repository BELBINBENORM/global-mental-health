# 🧠 Global Mental Health Crisis Tracker (2000–Present)

[![Update Kaggle Dataset](https://github.com/belbino/global-mental-health/actions/workflows/update-dataset.yml/badge.svg)](https://github.com/belbino/global-mental-health/actions/workflows/update-dataset.yml)

A longitudinal cross-country dataset tracking the global mental health crisis from 2000 through the most recent available year. **Auto-updated weekly via GitHub Actions** and pushed directly to Kaggle using the Kaggle API — no IP restrictions, no manual runs.

---

## 📦 Output Files

| File | Description | Coverage |
|------|-------------|----------|
| `prevalence.csv` | Depression + anxiety age-standardised prevalence rates by country × year | 2000–2021 |
| `suicide_rates.csv` | WHO age-standardised suicide mortality rate per 100k + sex breakdown | 2000–present |
| `system_capacity.csv` | Psychiatrists, psychologists, hospital beds, outpatient facilities per 100k | 2001–present |
| `policy_financing.csv` | Policy score, legislation flag, MH expenditure % health budget | 2001–present |
| `mental_health_panel.csv` | Merged country × year panel + all derived metrics — primary analysis file | 2000–present |
| `ml_features.csv` | ML-ready: lags, rolling windows, YoY rates, burden score, regime flags, next-year targets | 2000–present |

---

## 📡 Data Sources

- **World Bank Open Data API** — suicide rates, physicians, health expenditure (free, no key)
- **WHO Global Health Observatory (GHO) OData API** — mental health capacity & policy (free, no key)
- **Our World in Data / IHME GBD** — depression & anxiety prevalence mirrors (free, no key)

---

## 🚀 Setup

### 1. Fork / clone this repository

```bash
git clone https://github.com/belbino/global-mental-health.git
cd global-mental-health
```

### 2. Add Kaggle secrets to GitHub

Go to **Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret name | Value |
|-------------|-------|
| `KAGGLE_USERNAME` | Your Kaggle username |
| `KAGGLE_KEY` | Your Kaggle API key (from kaggle.com → Account → API) |

### 3. Create the Kaggle dataset (first time only)

```bash
pip install kaggle
export KAGGLE_USERNAME=your_username
export KAGGLE_KEY=your_key

# Run the notebook locally once to generate the CSV files
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace notebook.ipynb

# Create a new Kaggle dataset from the current directory
kaggle datasets create --path . --dir-mode zip
```

> After creation, copy the dataset slug (e.g. `your_username/global-mental-health-crisis`) into `dataset-metadata.json`.

### 4. Trigger an update manually

Go to **Actions → Update Kaggle Dataset → Run workflow** to trigger an immediate run, or wait for the weekly Monday 06:00 UTC schedule.

---

## ⚙️ How It Works

```
GitHub Actions (weekly cron / manual)
        │
        ▼
Execute notebook.ipynb
  ├── Fetch prevalence     ← OWID / IHME GBD CSV mirrors
  ├── Fetch suicide rates  ← World Bank API → WHO GHO fallback
  ├── Fetch system cap.    ← WHO GHO OData (MH_12, MH_13, MH_6, MH_7)
  ├── Fetch policy data    ← WHO GHO OData (MH_1, MH_2, MH_17)
  ├── Merge → panel CSV
  ├── Compute burden score, crisis flags, treatment gap proxy
  └── Build ML feature matrix (lags, rolling windows, targets)
        │
        ▼
Upload 6 CSVs + README.md → Kaggle dataset (kaggle datasets version)
```

---

## 📐 Derived Metrics

| Metric | Formula |
|--------|---------|
| **Combined Prevalence** | `depression_prevalence_pct + anxiety_prevalence_pct` (capped at 100%) |
| **Psychiatrist Gap** | `max(0, 1 − psychiatrists_per_100k)` — 0 = at/above WHO minimum |
| **Burden Score (0–100)** | Mean z-score of combined prevalence, suicide rate, psychiatrist gap — scaled 0–100 |
| **Treatment Gap Proxy** | % of prevalent cases not reached by outpatient services |
| **Policy Score (0–3)** | `policy_flag + legislation_flag + (expenditure_pct_health > 5 ? 1 : 0)` |

---

## 📄 License

CC0 — Public Domain. Data originates from WHO GHO and IHME GBD, both freely available without API keys.
