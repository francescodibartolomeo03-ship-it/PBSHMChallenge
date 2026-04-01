# Population-Based Structural Health Monitoring — Coding Challenge

**Candidate:** Francesco Di Bartolomeo  
**Position:** DIAMOND A-DC1 (PhD), ETH Zurich — Prof. Eleni Chatzi  
**Date:** April 2026

---

## Overview

This repository contains my solution to the [PBSHM coding challenge](https://github.com/ETH-IBK-SMECH/PBSHM-challenge) for the DIAMOND doctoral position. The challenge involves a population of **50 simulated shear-frame structures** (4–8 storeys each, 35 healthy / 15 damaged) and asks to explore damage detection through supervised, unsupervised, graph-based, and population-level approaches.

The guiding philosophy throughout is **physical reasoning before modelling**: understanding *why* a feature or method works (or fails) matters more than headline accuracy on a toy dataset.

## Repository Structure

| File | Description |
|------|-------------|
| `setup_0.py` | Shared utilities — data download, feature extraction, plotting constants. Imported by all notebooks via `from setup_0 import *`. |
| `1_exploration.ipynb` | **Task 1** — Exploratory data analysis. Physical interpretation of `dominant_modal_frequency_Hz`, mode-switching mechanism, feature engineering (frequency gradients, height-normalised dispersion, skewness). |
| `2_supervised.ipynb` | **Task 2** — Supervised baselines. LOO-CV comparison of Logistic Regression, Random Forest, and SVM across multiple feature sets. Best result: **LR with top-3 engineered features, AUC ≈ 0.69**. Misclassification analysis and physical interpretation of model coefficients. |
| `3_unsupervised.ipynb` | **Task 3** — Unsupervised anomaly detection. Isolation Forest on the same feature space. **AUC ≈ 0.55** — modest, but expected: without labels, separating damage from natural variability is inherently harder. |
| `4_graphBasedExtension.ipynb` | **Task 4** — Graph-based methods. Each structure modelled as a chain graph. GCN end-to-end approach honestly reported as failing (AUC ≈ 0.46). Graph-aware feature engineering (`node_dev_mean`) improves to **AUC ≈ 0.72**, with bootstrap CIs and permutation test (p = 0.501 — not significant, discussed honestly). |
| `5_populationGraph.ipynb` | **Task 5** — Population-level graph. k-NN graph over structures using modal features (replacing the provided geometry-only graph). LOO evaluation shows only marginal improvement (AUC +0.002), with discussion of why population context has limited value at n = 50. |

## Key Results

| Method | AUC | Labels? | Notebook |
|--------|-----|---------|----------|
| LR — graph + Task 2 features (best) | 0.72 | Yes | NB4 |
| LR — top-3 engineered features | 0.69 | Yes | NB2 |
| Isolation Forest | 0.55 | No | NB3 |
| GCN end-to-end | 0.46 | Yes | NB4 |

## Core Physical Insight

The single measurement per floor — `dominant_modal_frequency_Hz` — reflects which eigenmode has the largest amplitude at that storey. Damage (stiffness reduction) causes **mode switching**: the dominant mode changes at certain floors, creating spatial discontinuities in the frequency profile. This is best captured by *within-structure relative features* (gradients, deviations from neighbours) rather than simple global statistics, because inter-structure variability in baseline properties swamps the absolute damage signal.

## Honest Limitations

- **GCN failure** is reported transparently — with n = 50, training variance overwhelms the signal. The architecture is sound; the data regime is not.
- **Permutation test** on the graph-feature improvement yields p = 0.501 (non-significant). The conceptual contribution (spatial information) stands, but the numerical gain may be noise.
- **Population graph** provides negligible benefit — cosine similarities on geometry features are compressed near 1.0, and even modal-feature graphs don't meaningfully improve detection at this population size.
- **Damage localisation** is at chance level, which is physically expected: the dominant modal frequency is a global quantity that doesn't resolve spatial damage position.

## How to Run

All notebooks download the challenge data automatically from the [original repository](https://github.com/ETH-IBK-SMECH/PBSHM-challenge) on first execution. No manual data setup is needed.

### Requirements

```
python >= 3.8
numpy
pandas
matplotlib
seaborn
scipy
scikit-learn
networkx
torch
torch-geometric
```

Install with:

```bash
pip install numpy pandas matplotlib seaborn scipy scikit-learn networkx torch torch-geometric
```

### Execution Order

Run the notebooks in order (1 → 5). Each notebook is **self-contained** — all cells can be executed sequentially without dependencies on other notebooks' runtime state. The shared `setup_0.py` module handles data loading and common utilities.

## Acknowledgements

Challenge dataset and problem formulation by the [Structural Mechanics & Monitoring group](https://chatzi.ibk.ethz.ch/), ETH Zurich.
