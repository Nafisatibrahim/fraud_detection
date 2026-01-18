# Finance Fraud Detection (Freddy.ai) 🕵🏾‍♀️💳

A reproducible, end-to-end pipeline for training and evaluating a **fraud detection model** on financial transactions. The project is designed to be easy to adapt to different datasets by defining a clear **data contract** (via YAML), keeping preprocessing consistent, and tracking experiments.

---

## What this project does

* Loads a transaction dataset (CSV/Parquet)
* Validates the dataset using a **schema/data contract**
* Builds baseline + ML features
* Trains one or more fraud detection models
* Evaluates performance with fraud-appropriate metrics (PR-AUC, recall at fixed precision, etc.)
* Saves artifacts (models, metrics, plots) for reproducibility

---

## Project structure (suggested)

```
.
├── configs/
│   └── dataset_schema.yaml
├── data/
│   ├── raw/                # raw input files (not committed)
│   └── processed/          # cleaned/feature-ready data
├── notebooks/              # exploration / sanity checks
├── src/
│   ├── data/               # loading + validation + splits
│   ├── features/           # feature engineering
│   ├── models/             # training + inference
│   ├── evaluation/         # metrics + plots
│   └── utils/              # helpers (logging, seeding, paths)
├── artifacts/
│   ├── models/
│   └── reports/
├── tests/
├── requirements.txt
├── README.md
└── .gitignore
```

---

## Data contract (YAML)

This project expects a dataset with a binary target label (e.g., `is_fraud`) and a timestamped transaction history.

Example: `configs/dataset_schema.yaml`

```yaml
target: is_fraud

required_columns:
  - transaction_id
  - timestamp
  - amount
  - is_fraud

optional_columns:
  - customer_id
  - merchant_id
  - channel
  - country
  - city
  - device_id

timestamp_format: "auto"  # parse with pandas

split_strategy:
  type: time
  train_end: "2020-09-30"
  val_end: "2020-11-30"
  test_end: "2020-12-31"
```

Notes:

* **Time-based splits** are recommended to reduce leakage (train on past → test on future).
* If you don’t have a timestamp, use a random split but document the risk.

---

## Setup

### 1) Create environment

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Mac/Linux:
source .venv/bin/activate
```

### 2) Install dependencies

```bash
pip install -r requirements.txt
```

---

## Quickstart

### 1) Put your dataset in `data/raw/`

Example:

```
data/raw/transactions.csv
```

### 2) Validate + preprocess

```bash
python -m src.data.make_dataset \
  --input data/raw/transactions.csv \
  --schema configs/dataset_schema.yaml \
  --out data/processed/transactions.parquet
```

### 3) Train a baseline model

```bash
python -m src.models.train \
  --data data/processed/transactions.parquet \
  --schema configs/dataset_schema.yaml \
  --out artifacts/models/
```

### 4) Evaluate

```bash
python -m src.evaluation.evaluate \
  --data data/processed/transactions.parquet \
  --model artifacts/models/model.pkl \
  --out artifacts/reports/
```

---

## Evaluation metrics (fraud-focused)

Fraud detection is typically imbalanced, so accuracy is not useful by itself. Recommended metrics:

* **PR-AUC (Average Precision)**
* **Recall at fixed precision** (e.g., recall when precision ≥ 90%)
* **Precision@K / Recall@K** (top K alerts)
* **Confusion matrix** at an operational threshold
* Optional: cost-based evaluation (false positives vs false negatives)

---

## Reproducibility

* Fix random seeds in training and data splits
* Log:

  * dataset version/hash
  * schema used
  * feature set version
  * hyperparameters
  * metrics and plots
* Save artifacts to `artifacts/` (notebooks should be optional, not required)

---

## Safety & ethics notes

* Avoid label leakage (features that directly encode the target or post-event signals).
* Consider fairness and disparate impact if attributes correlate with protected classes.
* Treat this repo as a **prototype** unless it has been validated with production constraints (latency, drift, monitoring, auditability).

---

## Roadmap (optional)

* [ ] Add a simple baseline (logistic regression) + stronger model (LightGBM/XGBoost)
* [ ] Add feature store-style pipeline (consistent train/serve features)
* [ ] Add threshold selection aligned to ops goals (precision target, alert budget)
* [ ] Add model monitoring plan (drift, performance decay, data quality)
* [ ] Add explainability reports (global + per-transaction)

---

## License

Choose one:

* MIT (open, permissive)

---

## Contact

Maintainer: **Nafisat Ibrahim**
