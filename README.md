# 💼 XGBoost DeFi V2 Risk Scoring

**Risk Assessment for Wallets on Compound V2 Using Machine Learning (XGBoost)**

---

## 🔍 Overview

This project introduces a machine learning–powered risk scoring model to analyze wallets interacting with the **Compound V2 DeFi protocol**. Using an **XGBoost regression model**, we assign a **risk score between 0 and 1000**, enabling protocol developers, auditors, and analysts to evaluate wallet trustworthiness and detect anomalous or malicious behavior at scale.

---

## 🎯 Goals

- 🎯 Assign each wallet a **continuous risk score (0–1000)**.
- 🧠 Use **interpretable features** derived from transaction activity on Compound V2.
- ⚙️ Leverage **XGBoost regression** for accurate, scalable predictions.
- 🔍 Provide transparency in the **data pipeline, scoring logic**, and risk assumptions.

---

## 📊 Data Collection

We obtained wallet-level transaction data by querying the **Compound V2 protocol** and Ethereum blockchain using:

- Public **on-chain APIs** and tools (e.g., Etherscan, Infura)
- Extracted fields:
  - `wallet`, `methodId`, `functionName`, `contractAddress`
  - `blockNumber`, `gasUsed`, `cumulativeGasUsed`
  - Transaction metadata: value, timestamp, frequency

---

## 🧪 Feature Selection

We engineered features with a strong correlation to wallet risk behavior:

| Feature                     | Description                                                 | Reason for Inclusion                        |
|----------------------------|-------------------------------------------------------------|---------------------------------------------|
| `num_transactions`         | Total txns by wallet                                        | High txns may indicate bots/abuse           |
| `total_ether`              | Sum of ETH transacted                                       | Captures overall financial activity         |
| `average_transaction_value`| Mean ETH per txn                                            | Filters spamming wallets                    |
| `transaction_frequency`    | Txns/time window                                            | Higher frequency = higher abnormality       |
| `log_*` versions            | Log-scaled versions of above                                | Reduces skew and variance                   |
| `flag_high_*` indicators   | Binary flags for high thresholds                            | Helps isolate extreme behavior              |

---

## ⚙️ Normalization Strategy

To maintain feature consistency and scale-invariance:

- Applied **logarithmic transformation** to skewed numerical variables.
- Applied **Min-Max normalization** to scale features to `[0, 1]` range.
- Derived **binary flags** (0/1) for top-percentile values:
  - High frequency → `flag_high_freq`
  - High average value → `flag_high_avg_value`
  - High ETH transfer → `flag_high_ether`

This ensures that the model does not bias towards disproportionately large wallets or frequent users.

---

## 🎯 Scoring Logic

We formulated the **risk score** as a **regression target**, trained using a curated risk label mapping and confidence level:

- Risk scores were assigned between **0 (lowest risk)** and **1000 (highest risk)**.
- `risk_label` was manually tagged using heuristics:
  - **High Risk**: extremely high frequency, suspicious gas usage, low value txns, repeated calls to risky functions
  - **Low Risk**: few, spaced transactions with normal gas and value behavior

### Example:

| Wallet Address | Frequency | Avg Txn Value | Label     | Risk Score |
|----------------|-----------|---------------|-----------|------------|
| A              | High      | Low           | High Risk | 150        |
| B              | Moderate  | Moderate      | Low Risk  | 700        |

---

## ✅ Model Choice: XGBoost

- **Why XGBoost?**
  - Handles skewed data and outliers robustly
  - Supports feature importance visualization
  - High predictive performance on small/medium datasets
- **Performance:**
  - RMSE: `10.08`
  - R² Score: `0.98`
  - Cross-validated R²: `0.94`

---

## 📂 Project Structure

xgboost-defi-v2-risk-scoring/
│
├── data/ # CSVs with wallet features
├── models/ # Trained XGBoost model
├── notebooks/ # Jupyter analysis and training notebooks
├── src/
│ ├── feature_engineering.py # Feature transformation logic
│ ├── model_utils.py # Save/load model utilities
│ └── predict_risk.py # Inference and score prediction
├── results/
│ └── predictions.csv # Final output with scores
├── README.md
└── requirements.txt



## 📈 Inference Pipeline

```python
from src.model_utils import load_model
from src.predict_risk import predict_wallet_risk
import pandas as pd

model = load_model("models/xgboost_risk_model.pkl")
wallet_df = pd.read_csv("data/new_wallet_data.csv")
predictions = predict_wallet_risk(model, wallet_df)

# Output: DataFrame with risk_score and predicted label
predictions.to_csv("results/final_predictions.csv", index=False)

🛡️ Risk Indicator Justification
Indicator	Risk Behavior Captured
High Frequency	Possible spam, bot, MEV actors
Low Value, High Txn	Dusting or phishing patterns
Repeated Functions	Exploit attempts or protocol gaming
Abnormal Gas Usage	Smart contract exploits

These indicators align with patterns identified in real-world DeFi attacks and scam campaigns.

🧰 Requirements
txt
Copy
Edit
pandas
numpy
scikit-learn
xgboost
joblib
Install dependencies:

bash
pip install -r requirements.txt

Author: Rahul Choudhary
