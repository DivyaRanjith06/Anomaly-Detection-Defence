# Anomaly Detection for Defence Networks

Unsupervised network intrusion detection using a novel Dual-Path Autoencoder architecture evaluated against three baselines on the UNSW-NB15 benchmark dataset.

---

## Overview

This project investigates anomaly-based intrusion detection in network traffic under a strict one-class unsupervised setting — the model trains exclusively on benign traffic and detects attacks entirely through reconstruction deviation at inference time.

The central contribution is a **Dual-Path Autoencoder** that processes feature-level and temporal signals simultaneously through parallel Dense and LSTM encoders, fused via a learnable bottleneck layer. This stands in contrast to all existing published AE+LSTM work, which runs the two paths sequentially. The architecture is evaluated against Dense AE, LSTM AE, and Isolation Forest baselines using standard classification metrics and ROC-AUC on real UNSW-NB15 data.

---

## Dataset

**UNSW-NB15** — University of New South Wales Network Benchmark 2015

| File | Description |
|---|---|
| `UNSW_NB15_training-set.csv` | Official training split (~175k records) |
| `UNSW_NB15_testing-set.csv` | Official test split (~82k records) |

The two CSVs are merged and re-split internally to enforce the one-class training setup: the model trains on 80% of normal-only traffic and is evaluated on the remaining 20% normal traffic combined with all attack samples.

Download from the official source: https://research.unsw.edu.au/projects/unsw-nb15-dataset

Place both CSV files in the same directory as the notebook before running.

---

## Architecture

### Dual-Path Autoencoder (Novel)

```
Input (2D feature vector)         Input (10-step sequence)
          |                                  |
     Dense Encoder                     LSTM Encoder
  Dense(64) -> BN ->               LSTM(32) -> LSTM(16)
  Dense(32) -> BN ->                          |
  Dense(encoding_dim)               RepeatVector -> LSTM -> LSTM
          |                                  |
          +----------  Concatenate  ----------+
                             |
                   Learnable Fusion Layer
                             |
                       Shared Bottleneck
                     (encoding_dim = 24)
                       Dropout(0.25)
                             |
                    Dense Reconstruction
```

**Key design decisions:**
- Parallel paths run simultaneously, not sequentially — each specialises independently before fusion
- Learnable fusion layer (Dense projection) instead of raw concatenation
- Denoising training: Gaussian noise (std=0.02) added to inputs; model reconstructs clean signal
- Hybrid anomaly score: `score = 0.6 * reconstruction_error + 0.4 * latent_distance_from_centroid`
- Threshold selected by maximising F1 on a held-out validation split

### Baselines

| Model | Type | Captures Feature Patterns | Captures Temporal Patterns |
|---|---|---|---|
| Isolation Forest | Classical ML | Structural | No |
| Dense Autoencoder | Deep Learning | Yes | No |
| LSTM Autoencoder | Deep Learning | Partial | Yes |
| Dual-Path AE (this work) | Novel Deep Learning | Yes | Yes |

---

## Methodology

**Preprocessing**
- Categorical columns label-encoded
- Zero-variance features removed via VarianceThreshold
- Missing values imputed (median for numeric, mode for categorical)
- StandardScaler fitted on training data only; applied to test without leakage

**One-Class Split**
- Train: 80% of normal (label=0) samples only
- Test: remaining 20% normal + 100% of all attack samples, shuffled

**Thresholding**
- Dense AE and LSTM AE: 92nd percentile of training reconstruction errors
- Isolation Forest: contamination=0.05
- Dual-Path AE: F1-maximising threshold over a validation reconstruction error grid

**Evaluation**
- Accuracy, Precision, Recall, F1, ROC-AUC
- Confusion matrices with percentage breakdowns
- Reconstruction error distributions (log-scale) per model
- ROC curves, all four models overlaid
- Ablation table quantifying per-path contribution

---

## Project Structure

```
Anomaly Detection Defence/
|-- dual_path_autoencoder.ipynb    # Full implementation and evaluation
|-- UNSW_NB15_training-set.csv     # Training data (place here before running)
|-- UNSW_NB15_testing-set.csv      # Test data (place here before running)
|-- README.md
```

---

## Requirements

```
Python >= 3.8
tensorflow >= 2.x
numpy
pandas
scikit-learn
matplotlib
seaborn
```

Install all dependencies:

```bash
pip install numpy pandas scikit-learn matplotlib seaborn tensorflow
```

---

## Usage

1. Clone the repository and navigate into it:

```bash
git clone https://github.com/yourusername/anomaly-detection-defence.git
cd anomaly-detection-defence
```

2. Place `UNSW_NB15_training-set.csv` and `UNSW_NB15_testing-set.csv` in the project root.

3. Launch Jupyter and run all cells top-to-bottom:

```bash
jupyter notebook dual_path_autoencoder.ipynb
```

All four models train sequentially. Evaluation plots and the metrics table are generated at the end.

---

## Novelty Claim

Every existing autoencoder + LSTM paper in the literature (IEEE, Elsevier, Springer, ACM, Nature) routes data through one encoder before feeding it into the other — a sequential design. This notebook implements a **parallel** dual-path architecture where Dense and LSTM encoders operate independently on the same input and their latent representations are fused through a jointly trained projection layer. No prior publication of this exact architecture evaluated on UNSW-NB15 was found at the time of development.


Dataset: Moustafa, N. and Slay, J. (2015). UNSW-NB15: A comprehensive data set for network intrusion detection systems. Military Communications and Information Systems Conference (MilCIS), IEEE.
