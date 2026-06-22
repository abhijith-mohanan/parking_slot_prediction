# Parking Stay-Duration Classification — San Diego

If you've ever driven around a city looking for parking, you know the frustration. This project tries to help with that — using real data from San Diego's smart parking meters, it predicts whether a parking spot is likely to be tied up for a while (more than an hour) or free up quickly.

## What This Project Does

Every time someone pays at a parking meter in San Diego, that transaction gets logged — when it started, how long they paid for, which meter, what zone, how much it cost. Using a full year of this data (almost 2 million transactions), this project trains a model to answer one simple question:

**"Is this parking spot going to be a long stay (over 60 minutes) or a short stay?"**

That's useful info — if a driver knows certain areas tend to have long-stay parkers during certain hours, they can plan around it.

> **Note:** This project does not predict real-time slot availability directly — that would require live occupancy sensor data, which isn't present here. Instead, stay duration is used as a proxy: zones dominated by long-stay transactions have slower turnover (fewer slots freeing up), while short-stay zones free up faster.

## The Data

Two datasets from the City of San Diego's open data portal:

- **Parking transactions (2020)** — about 1.95 million records, with details like start time, duration, payment method, and amount paid
- **Meter locations** — around 4,060 meters, with zone, area, pricing, and time limits

The transactions file is too big for GitHub (137MB, over the 100MB limit), so it's not included here. You'll need to download it yourself — link is below.

## What's Been Done

**Step 1 — Loading and exploring the data**
Got both files loaded, checked what's in them, looked for missing values, and confirmed the two datasets could actually be joined together using the meter ID.

**Step 2 — Cleaning**
Real data is messy. We removed duplicate transactions, fixed invalid records (where the end time was somehow before the start time), and got rid of a handful of test entries that weren't real meters — including one with coordinates of (0,0), which would've been a meter floating somewhere off the coast of Africa.

**Step 3 — Feature engineering**
This is where we turned raw timestamps and prices into something a model can actually learn from — hour of day, day of week, whether it's a weekend, whether it's during peak hours, plus some cyclical encoding so the model understands that 11pm and midnight are basically the same time of day.

**Step 4 — Exploring patterns and building models**
Made some charts to see what the data was telling us (turns out Saturdays have the highest long-stay rates, and almost nobody pays for parking on Sundays). Then trained four models — Random Forest, XGBoost, CatBoost, and an MLP neural network — to make the actual predictions.

**Step 5 — Evaluation and ablation**
Looked at accuracy, precision, recall, and which features mattered most. Also ran an ablation study removing `trans_amt_dollars` (the dominant feature at ~73% importance) to check how much the models rely on it. Built a prediction function where you can plug in a time, location, and price, and get a prediction back from all four models.

## How Well Does It Work?

| Model | Accuracy | Precision | Recall | F1 Score |
|---|---|---|---|---|
| Random Forest | 86.1% | 93.9% | 79.2% | 85.9% |
| XGBoost | 86.4% | 95.8% | 78.0% | 86.0% |
| CatBoost | 86.3% | 95.0% | 78.0% | 85.7% |
| MLP | 86.4% | 94.0% | 79.0% | 86.0% |

All four models converge at ~86% — consistent results across very different model families, which validates the finding rather than being a weakness. The MLP matches gradient boosting on tabular data, which is notable.

The ablation study (removing `trans_amt_dollars`) reveals how much the models depend on that single dominant feature — see the notebook for the full side-by-side comparison.

## Want to Run It Yourself?

**1. Clone the repo**

```bash
git clone https://github.com/abhijith-mohanan/parking_slot_prediction.git
cd parking_slot_prediction
```

**2. Set up a virtual environment** (optional but recommended)

```bash
python3 -m venv venv
source venv/bin/activate
```

**3. Install the libraries you'll need**

```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost catboost jupyter
```

On a Mac, if XGBoost complains about OpenMP when you try to import it, run:

```bash
brew install libomp
```

**4. Grab the transactions data**

This file is too big to include in the repo, so download it separately and drop it into the `data/` folder:

`https://seshat.datasd.org/parking_meters_transactions/treas_parking_payments_2020_datasd.csv`

**5. Run the notebook**

```bash
jupyter notebook parkingslot.ipynb
```

Just run all the cells in order from top to bottom.

## What's Under the Hood

| Library | What it's for |
|---|---|
| pandas | Loading and working with the data |
| numpy | Number crunching, cyclical encoding |
| matplotlib & seaborn | All the charts |
| scikit-learn | Train/test split, Random Forest, MLP, evaluation metrics |
| xgboost | XGBoost model |
| catboost | CatBoost model |
| jupyter | Running the notebook |

## Project Layout

```
parking_slot_prediction/
├── data/
│   └── parking_meters_current.csv
├── reports/
│   └── report.tex        
├── parkingslot.ipynb
├── .gitignore
└── README.md
```

---

Built by Abhijith Mohanan