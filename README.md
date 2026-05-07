# Detecting Covert Channels in Encrypted QUIC and HTTP/3 Traffic

A statistical and behavioral machine learning framework for detecting covert communication in encrypted QUIC and HTTP/3 traffic — without payload decryption.

This repository accompanies the paper:

> **A Statistical and Behavioral Approach for Detecting Covert Channels in Encrypted QUIC and HTTP/3 Traffic**
> Nour Huwio, Prof. Mohammad Al Nabhan
> Cybersecurity Department, Princess Sumaya University for Technology

-----

## Overview

Modern web protocols such as QUIC and HTTP/3 integrate TLS 1.3 directly into the transport layer. While this strengthens privacy, it also blinds traditional network defenses such as deep packet inspection (DPI) — making it harder to detect attackers who hide information inside legitimate-looking encrypted traffic (covert channels).

This project demonstrates that covert communication can still be detected from **flow-level metadata alone** — packet sizes, inter-arrival times, flow duration, directionality, and burst patterns — without ever inspecting payload content. Privacy is preserved, and detection works regardless of the underlying encryption scheme.

-----

## Highlights

- **Privacy-preserving:** operates only on observable flow-level metadata. No payload decryption.
- **Protocol-aware:** tailored to QUIC / HTTP/3 (UDP transport, multiplexed streams, connection migration).
- **Strong empirical results:**
  - Random Forest: **97.21% accuracy**, **97.67% precision**, **96.73% recall**, **97.19% F1**, **AUC 0.9895**
  - SVM baseline: 90.11% accuracy, AUC 0.9496
  - 5-fold cross-validation: **97.10% ± 0.14%**
  - 20-component PCA retains **99.39%** of variance
- **Reproducible:** fixed random seed (42), documented hyperparameters, end-to-end pipeline.

-----

## Methodology

The pipeline follows seven stages:

1. **Data collection** — three complementary datasets:
- **CICIDS2017** — primary labeled benign + attack ground truth
- **USTC-TFC2016** — encrypted benign + malware behavioral signatures
- **Real QUIC pcap captures** — genuine QUIC behavior (UDP, multiplexing, migration)
1. **Covert behavior simulation** — multiplicative scaling (∈ [1.5, 3.0]) applied to the first five statistical features for 30,000 randomly sampled flows. Models burst-amplification and statistical-distribution-shift covert signatures.
1. **Feature extraction** — 78 numeric flow-level features grounded in covert-channel theory (packet size statistics, inter-arrival times, flow duration, directionality ratios, burst features).
1. **Preprocessing** — stratified 80/20 train/test split *before* normalization (no leakage), then Min-Max scaling and PCA reduction to 20 components.
1. **Classification** — Random Forest (primary, n_estimators=100, max_depth=12) and SVM (RBF baseline, 10,000-sample subset).
1. **Evaluation** — accuracy, precision, recall, F1, ROC-AUC, confusion matrix, feature importance.
1. **Cross-validation** — 5-fold stratified CV to confirm result stability.

-----

## Results

### Held-out test set (n = 16,000)

|Metric   |Random Forest|SVM   |
|---------|-------------|------|
|Accuracy |97.21%       |90.11%|
|Precision|97.67%       |87.52%|
|Recall   |96.73%       |93.56%|
|F1-score |97.19%       |90.44%|
|ROC-AUC  |0.9895       |0.9496|

### Random Forest confusion matrix

|                 |Predicted Normal|Predicted Covert|
|-----------------|----------------|----------------|
|**Actual Normal**|7,815 (TN)      |185 (FP)        |
|**Actual Covert**|262 (FN)        |7,738 (TP)      |

False positive rate: 2.31% · Miss rate: 3.27%

### 5-fold cross-validation (Random Forest)

|Fold          |Accuracy          |Precision         |F1                |
|--------------|------------------|------------------|------------------|
|1             |97.17%            |97.71%            |97.15%            |
|2             |96.93%            |97.70%            |96.91%            |
|3             |97.01%            |97.70%            |96.99%            |
|4             |97.08%            |97.51%            |97.06%            |
|5             |97.33%            |97.71%            |97.32%            |
|**Mean ± Std**|**97.10% ± 0.14%**|**97.67% ± 0.08%**|**97.09% ± 0.14%**|

The tight standard deviation confirms the hold-out result is not an artifact of a favorable random split.

-----

## Repository Contents

```
.
├── README.md                       # This file
├── notebook.ipynb                  # Main notebook (data prep, training, evaluation)
└── LICENSE                         # MIT license
```

-----

## How to Run

### Option 1 — Open in Google Colab (easiest)

1. Click the badge at the top of the notebook (or the **Open in Colab** link).
1. Upload the three CSV datasets when prompted, or mount Google Drive.
1. Run all cells (`Runtime → Run all`).

### Option 2 — Run on Kaggle

1. Open the notebook on Kaggle.
1. Attach the CICIDS2017, USTC-TFC2016, and QUIC datasets as inputs.
1. Run all cells.

### Option 3 — Run locally

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
jupyter notebook notebook.ipynb
```

**Dependencies:** Python 3.9+, NumPy, pandas, scikit-learn, matplotlib, seaborn.

-----

## Datasets

The three source datasets are not redistributed in this repository due to size and licensing.

- **CICIDS2017** — https://www.unb.ca/cic/datasets/ids-2017.html
- **USTC-TFC2016** — https://github.com/yungshenglu/USTC-TFC2016
- **QUIC pcap captures** — capture your own with `tcpdump -i <iface> -w quic.pcap "udp port 443"`, then convert to flow features with [CICFlowMeter](https://github.com/ahlashkari/CICFlowMeter).

-----

## Limitations

- The covert-channel labels are produced by a controlled simulation (multiplicative scaling of low-level statistical features). The simulation is grounded in covert-channel theory but does not capture sophisticated mimicry attacks.
- No publicly labeled QUIC covert-channel dataset currently exists; constructing one is left as future work.
- The SVM baseline was trained on a 10,000-sample subset due to the quadratic complexity of the RBF kernel.
- Network traffic distributions are non-stationary, so periodic retraining may be required for sustained deployment.

-----

## Citation

If you use this work in your research, please cite:

```bibtex
@article{huwio2026quiccovert,
  title       = {A Statistical and Behavioral Approach for Detecting Covert Channels in Encrypted QUIC and HTTP/3 Traffic},
  author      = {Huwio, Nour and Al Nabhan, Mohammad},
  year        = {2026},
  institution = {Princess Sumaya University for Technology}
}
```

-----

## License

Released under the [MIT License](LICENSE).

-----

## Contact

For questions, bug reports, or collaboration, please open an issue on this repository, or contact the authors via the Cybersecurity Department, Princess Sumaya University for Technology.
