# Netflix Network Traffic Classification

A machine-learning project that classifies packet-level protocol labels in a Netflix network-traffic capture using Python and scikit-learn.

> **Academic project:** Brainware University  
> **Programme:** Bachelor of Computer Applications (BCA)  
> **Semester:** 5th Semester · **Subject:** Machine Learning  
> **Course code:** BCA57205(P)  
> **Student:** Subhadip Mudi

## Project overview

The objective is to train supervised machine-learning models to predict the `Protocol` label recorded for a network packet in the supplied Netflix traffic CSV.

The inspected dataset contains two labels: `TCP` and `TLSv1.3`.

This project follows a practical machine-learning workflow: dataset inspection, cleaning, exploratory data analysis (EDA), feature engineering, model training, evaluation, comparison, and prediction on a held-out packet.

**Scope clarification:** This is packet-level classification of the protocol labels recorded in the CSV. It does not identify whether arbitrary network traffic belongs to Netflix, and it does not analyse viewing activity. TCP and TLS operate at different protocol layers; the labels represent the protocol descriptions exported by the capture tool.

## Objectives

- Inspect and understand a real-world network packet dataset.
- Clean and prepare packet metadata for machine learning.
- Explore protocol distribution and packet-length behaviour.
- Engineer packet timing and IP-pair features.
- Train and compare four supervised classification algorithms.
- Evaluate class-wise performance using precision, recall, F1-score, and confusion matrices.
- Demonstrate prediction on an unseen test packet.

## Dataset

The notebook expects a CSV named `Netflix_1.csv` with Wireshark-style packet fields.

| Field | Description |
|---|---|
| Packet index (`No.` / `index`) | Row or packet identifier |
| `Time` | Packet timestamp |
| `Source` | Source IP address |
| `Destination` | Destination IP address |
| `Protocol` | Protocol label recorded in the capture |
| `Length` | Packet length in bytes |
| `Info` | Packet details exported by the capture tool |

The inspected dataset contains **4,290,778 records** and seven original columns.

### Protocol distribution

| Protocol label | Records |
|---|---:|
| TCP | 3,942,556 |
| TLSv1.3 | 348,222 |
| **Total** | **4,290,778** |

A duplicate check on the original dataset reported **0 duplicate rows**.

The original CSV is not included in this repository. It is approximately 739 MB when extracted and should be obtained separately from an authorised source or supplied by the project owner. Do not upload private or restricted packet captures to a public repository.

## Data preparation and exploratory analysis

The notebook includes:

- Dataset shape, column, datatype, and summary inspection.
- Missing-value and duplicate checks.
- Protocol frequency analysis and protocol-distribution visualisation.
- Packet-length summary, histogram, and box plot.
- Packet traffic over time.
- Extraction of `Hour`, `Minute`, and `Second` from the timestamp as an additional time-feature exercise.

The extracted time-of-day fields are shown as part of the data-analysis workflow. They are **not** inputs to the final KNN experiment described below.

## Feature engineering

The final improved experiment uses five numeric input features:

| Feature | Description |
|---|---|
| `Length` | Current packet length |
| `direction` | Direction within a canonicalised source/destination IP pair |
| `pair_delta_ms` | Time since the preceding packet in that IP pair |
| `pair_prev_length` | Previous packet length in that IP pair |
| `pair_prev5_mean_length` | Mean length of up to the previous five packets in that IP pair |

The first packet in an IP-pair sequence has no preceding-packet values. The engineered timing and previous-length features contained **38 missing values**, which are handled with median imputation in the model pipeline.

### Feature limitations

The CSV `Info` field does not expose ports and TCP flags consistently for both labels. The inspected TCP rows contain port/flag text, while TLSv1.3 rows do not. Those extracted fields are therefore excluded from the model inputs because their missingness could reveal the target label and create misleading predictions.

The IP-pair features are approximate pair-level features, not complete bidirectional transport-flow features. The original PCAP/PCAPNG capture was not available during this work, so feature engineering is limited to information preserved in the CSV.

## Machine-learning algorithms

Four supervised classifiers were trained and compared:

1. Logistic Regression
2. K-Nearest Neighbors (KNN)
3. Decision Tree
4. Random Forest

The notebook also includes a Decision Tree visualisation, Random Forest feature-importance analysis, and a new-packet prediction demonstration using a held-out test record.

## Model comparison

The initial comparison used the improved five-feature dataset and sampled training/testing subsets. The results recorded in the notebook were:

| Model | Accuracy | Macro F1-score |
|---|---:|---:|
| Logistic Regression | 56.95% | 0.4362 |
| K-Nearest Neighbors | 92.57% | 0.5979 |
| Decision Tree | 48.73% | 0.4080 |
| Random Forest | 54.36% | 0.4385 |

KNN had the best macro F1-score in this comparison.

## Final KNN experiment

The final selected configuration is **K-Nearest Neighbors with `k = 7`**, using random oversampling in the training pipeline to address class imbalance.

The reported final evaluation used **50,000 sampled training records** and **10,000 sampled test records**. The test set retained the natural class distribution.

### Overall metrics

| Metric | Result |
|---|---:|
| Accuracy | **91.01%** |
| Macro precision | 65.30% |
| Macro recall | 57.85% |
| Macro F1-score | **0.5993** |

### Class-wise classification report

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| TCP | 0.9320 | 0.9733 | 0.9522 | 9,200 |
| TLSv1.3 | 0.3740 | 0.1837 | 0.2464 | 800 |

### Majority-class baseline

For context, a baseline that always predicts the majority class achieved **92.00% accuracy** and **0.4792 macro F1-score** on the same test sample.

The final KNN model has slightly lower accuracy than this majority-class baseline, but a higher macro F1-score. This indicates improved balance across the two classes, while also showing that the model's overall accuracy should not be interpreted on its own.

## Interpretation and limitations

- The dataset is imbalanced: approximately 92% of records are TCP.
- The final KNN model identifies TCP well but has low TLSv1.3 recall (**18.37%**). It misses many TLSv1.3 packets.
- The model should therefore not be described as highly reliable for both classes.
- Accuracy alone is misleading on this dataset. Macro precision, macro recall, macro F1-score, per-class metrics, and the confusion matrix should be considered together.
- The reported final metrics come from sampled subsets, not a full-dataset evaluation.
- Independent testing on a separate Netflix capture has not yet been completed.
- The new-packet demonstration predicts a held-out test record; it is not live network monitoring.

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
2. Obtain the authorised `Netflix_1.csv` dataset separately.
3. Upload `Netflix_1.csv` to the Colab runtime, or place it in Google Drive and update `DATA_PATH` in the notebook.
4. Run notebook cells in order, from imports and data loading through feature engineering, model training, and final evaluation.
5. The notebook writes model and report outputs to the Colab runtime. Copy any files you need to keep to Google Drive before the runtime is cleared.

### Open in Google Colab

[Open `Machine_Learning.ipynb` in Google Colab](https://colab.research.google.com/github/subhdeeep/Netflix-Network-Traffic-Classification/blob/main/Machine_Learning.ipynb)

## Repository contents

```text
Netflix-Network-Traffic-Classification/
├── README.md
└── Machine_Learning.ipynb
```

The notebook contains the implementation and experiment outputs. The raw dataset and generated model file are not currently stored in this repository.

## Future improvements

- Evaluate the selected model on a separate capture session not used during training.
- Obtain the original PCAP/PCAPNG file to extract consistent transport-layer and flow-level features.
- Improve TLSv1.3 recall while monitoring precision and macro F1-score.
- Investigate additional class-balancing strategies and classifiers.
- Validate that feature generation for each test packet uses only information available at prediction time.

---

This repository is an academic project for learning and experimentation. The reported metrics describe the sampled evaluation recorded in the notebook and should not be interpreted as production-grade network-classification performance.
