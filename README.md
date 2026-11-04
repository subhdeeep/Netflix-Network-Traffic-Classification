# Netflix Network Traffic Classification

A machine learning project that classifies packet-level protocol labels in a Netflix network-traffic capture using Python and scikit-learn.

> **Academic project** — Brainware University  
> **Course:** BCA · **Semester:** 5th · **Subject:** Machine Learning  
> **Submitted by:** Subhadip Mudi  
> **Submitted to:** Riyanka Hazra

## Project overview

This project explores a packet-capture dataset collected during Netflix network activity and trains supervised machine learning models to predict the `Protocol` label recorded in the dataset. The two labels in this CSV are `TCP` and `TLSv1.3`.

The project covers:

- Loading and inspecting the network-traffic CSV
- Data cleaning and exploratory data analysis (EDA)
- Feature engineering from packet length, packet timing, and IP-pair information
- Training and comparing four classification algorithms
- Addressing class imbalance with random oversampling
- Evaluating the selected model with accuracy, precision, recall, F1-score, and a confusion matrix

**Important scope note:** This is packet-level classification of the protocol labels present in the CSV. It is not a model that detects whether an arbitrary network connection belongs to Netflix, and it is not a content or viewing-activity analysis project. TCP and TLS are also different protocol layers; the labels reflect how the source capture was dissected/exported.

## Dataset

The notebook expects a CSV named `Netflix_1.csv` with these columns:

| Column | Description |
|---|---|
| `Time` | Packet timestamp |
| `Source` | Source IP address |
| `Destination` | Destination IP address |
| `Protocol` | Protocol label recorded in the capture |
| `Length` | Packet length in bytes |
| `Info` | Packet details exported by the capture tool |

The inspected dataset contains **4,290,778 records** and seven columns in its original form (including the packet index). The class distribution is:

| Protocol label | Records |
|---|---:|
| TCP | 3,942,556 |
| TLSv1.3 | 348,222 |

The original CSV is not included in this repository. It is large (approximately 739 MB when extracted) and should be obtained from the project owner or the dataset's authorized source. Do not upload private or restricted packet captures to a public repository.

## Features used

The improved experiment uses five features:

- `Length` — current packet length
- `direction` — direction within a canonicalized source/destination IP pair
- `pair_delta_ms` — time since the previous packet in that IP pair
- `pair_prev_length` — previous packet length in that IP pair
- `pair_prev5_mean_length` — rolling mean of up to the previous five packet lengths in that IP pair

The previous-packet features have 38 missing values at the beginning of IP-pair sequences. These are handled by median imputation inside the model pipeline.

The CSV's `Info` field does not provide ports and TCP flags consistently for both labels: the inspected TCP rows contain them, while TLSv1.3 rows do not. Therefore, extracted port/flag fields are **not** used as model inputs, because their missingness would reveal the target label and create misleading results.

## Algorithms evaluated

The notebook evaluates:

1. Logistic Regression
2. K-Nearest Neighbors (KNN)
3. Decision Tree
4. Random Forest

The selected final configuration is **K-Nearest Neighbors with `k = 7`**, trained using random oversampling on the training split.

## Final reported results

The final evaluation used 50,000 sampled training records and 10,000 sampled test records from a chronological 80:20 split. The test sample retained the natural class imbalance.

| Metric | Result |
|---|---:|
| Accuracy | **91.01%** |
| Macro precision | 65.30% |
| Macro recall | 57.85% |
| Macro F1-score | **0.5993** |

### Per-class report

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| TCP | 0.9320 | 0.9733 | 0.9522 | 9,200 |
| TLSv1.3 | 0.3740 | 0.1837 | 0.2464 | 800 |

### Interpretation and limitations

- The dataset is imbalanced: approximately 92% of the records are TCP.
- The model performs strongly on TCP but has low recall for TLSv1.3. It misses many TLSv1.3 packets, so the model should not be described as highly reliable for both classes.
- Accuracy alone is misleading for this dataset. Macro precision, macro recall, macro F1-score, per-class metrics, and the confusion matrix should be considered together.
- The final experiment uses sampled training and test subsets; the reported metrics are not a full-dataset evaluation.
- No original PCAP/PCAPNG file was available during this work. Feature engineering is therefore limited to fields preserved in the CSV.
- Independent testing on a separate Netflix capture has not yet been completed.
- The IP-pair features are approximate pair-level features, not complete bidirectional transport-flow features, because the CSV does not preserve consistent port information for both classes.

## Tools and libraries

- Python
- Google Colab / Jupyter Notebook
- Pandas and NumPy
- Matplotlib and Seaborn
- scikit-learn
- imbalanced-learn
- Joblib

## How to run

1. Open the notebook in Google Colab using the link below.
2. Obtain the authorized `Netflix_1.csv` dataset separately.
3. Upload the CSV to the Colab runtime, or place it in Google Drive and update `DATA_PATH` in the notebook to the correct location.
4. Run the notebook cells in order, from imports and data loading through training and final evaluation.
5. The notebook saves the trained model and evaluation report to the Colab runtime. Copy the outputs to Google Drive if you want to keep them after the runtime ends.

### Open in Google Colab

[Open `Machine_Learning.ipynb` in Google Colab](https://colab.research.google.com/github/subhdeep/Netflix-Network-Traffic-Classification/blob/main/Machine_Learning.ipynb)

## Future improvements

- Evaluate on a separate capture session that was not used during training.
- Obtain the original PCAP/PCAPNG to extract consistent transport-layer and flow-level features.
- Investigate class imbalance and improve TLSv1.3 recall without relying on accuracy alone.
- Compare additional models and report confidence intervals or repeated-split results.
- Validate that feature generation for test data uses only information available at prediction time.

## Repository contents

```text
Netflix-Network-Traffic-Classification/
├── README.md
└── Machine_Learning.ipynb
```

---

This repository is an academic project for learning and experimentation. The reported metrics describe the specific sampled evaluation in the notebook and should not be interpreted as production-grade network classification performance.
