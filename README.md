# 🛡️ Deep Learning-Based Anomaly Detection for Defence Communication Networks

A comprehensive anomaly detection system built with deep learning to identify network intrusions and cyber threats in defence communication networks. Uses the **UNSW-NB15** dataset and combines a Dense Autoencoder, LSTM Autoencoder, and Isolation Forest for robust unsupervised threat detection.

---

## 📌 Overview

Traditional intrusion detection systems rely on known attack signatures. This system takes an **unsupervised approach** — training only on normal traffic so it can detect novel, zero-day attacks without ever having seen them before. This is critical in defence contexts where adversaries constantly evolve their tactics.

---

## 🧠 Models Used

| Model | Type | Purpose |
|---|---|---|
| Dense Autoencoder | Unsupervised Deep Learning | Learns compact representation of normal traffic; flags deviations |
| LSTM Autoencoder | Sequence-based Deep Learning | Captures temporal patterns in traffic sequences |
| Isolation Forest | Ensemble ML | Classical baseline for comparison |

---

## 📂 Repository Structure

```
anomaly-detection-defence/
│
├── anomaly_detection_defence_2.ipynb   # Main notebook (all models + analysis)
├── README.md                           # Project documentation
├── .gitignore                          # Files to exclude from version control
```

---

## 📊 Dataset

**UNSW-NB15** — A realistic network intrusion dataset containing both normal traffic and 9 attack categories (Fuzzers, DoS, Exploits, Reconnaissance, Backdoors, etc.).

🔗 Download: [https://research.unsw.edu.au/projects/unsw-nb15-dataset](https://research.unsw.edu.au/projects/unsw-nb15-dataset)

Files needed:
- `UNSW_NB15_training-set.csv`
- `UNSW_NB15_testing-set.csv`

---

## 🔍 Key Features

- **Unsupervised training** — trained on normal traffic only; no attack labels needed during training
- **Three-tier threat levels** — classifies detections as Low / Medium / High threat for operator prioritization
- **Zero-day detection** — evaluated on attack categories held out from training
- **Feature-level explainability** — identifies which network features drove each anomaly alert
- **Synthetic attack simulation** — generates DoS and data exfiltration scenarios for extended testing
- **Model comparison** — benchmarks autoencoder against Isolation Forest with full metrics

---

## 📈 Evaluation Metrics

Models are evaluated on:
- Accuracy, Precision, Recall, F1 Score
- ROC-AUC
- Confusion Matrix
- Per-attack-category detection rate (zero-day simulation)

---

## 🏗️ Pipeline Summary

```
Raw CSVs → Merge → Drop irrelevant cols → Handle missing values
        → Encode categoricals → Remove low-variance features
        → Remove high-correlation features → Train/test split
        → StandardScaler → Train Autoencoder (normal only)
        → Compute reconstruction error → Threshold → Predict
        → Evaluate + Explain
```

---

## 📌 Why Anomaly Detection Over Classification for Defence?

1. **New attacks appear constantly** — zero-day exploits are by definition unseen
2. **Attack data is scarce** — labeled samples for advanced threats are hard to obtain
3. **Normal behaviour is stable** — defence networks have predictable baselines
4. **Adaptability** — the model updates its understanding of "normal" as the network evolves

---

