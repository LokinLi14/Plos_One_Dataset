# PU Learning for Hidden-Positive Student Identification

This repository provides a **Positive–Unlabeled (PU) learning** framework for student-aid screening. The research goal is to identify potentially underserved students among an unlabeled population when only a subset of students with confirmed aid records is available.

> The current repository focuses on controlled synthetic experiments for validating the methods and research assumptions. The `y_true` field is available only in simulation and is used for evaluation.

## Problem setting

- `labeled = 1`: an observed positive, representing a student who received aid;
- `labeled = 0`: an unlabeled sample that may contain both true negatives and hidden positives;
- `y_true`: the simulated ground-truth poverty label, available only for synthetic evaluation.

The experiments compare standard supervised baselines with PU methods and use an Oracle model as an upper-bound reference.

## Methods

| Method | Description |
| --- | --- |
| Naive supervised | Treats every unlabeled sample as a negative example. |
| Cost-sensitive | Adds class-balancing weights to the naive baseline. |
| Spy two-step | Hides a subset of known positives, identifies reliable negatives, and retrains the classifier. |
| Elkan–Noto weighted | Estimates the labeling probability and retrains with positive/negative weights for unlabeled samples. |
| Oracle | Trains with `y_true`; used only as an unobservable upper-bound reference. |

All methods use the same `HistGradientBoostingClassifier` base model to make the comparison consistent.

## Repository structure

```text
.
├── synth_data.py          # Synthetic campus consumption and behavior generator
├── pu_methods.py          # PU learning and baseline implementations
├── run_experiments.py     # End-to-end experiments and figure generation
├── sensitivity.py         # Mechanism sweep and feature-group ablation
├── multi_seed_agg.csv     # Aggregated results across random seeds
├── requirements.txt       # Python dependencies
└── README.md
```

## Installation

Python 3.9 or newer is recommended. Create a virtual environment and install the dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate       # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Reproducing the experiments

Run the end-to-end synthetic-data experiment:

```bash
python3 run_experiments.py --mode synthetic --outdir results
```

The command produces:

- `metrics.csv`: PR-AUC, P@5/10/15%, and R@5/10/15% metrics;
- `summary.json`: sample counts, label statistics, and model information;
- `fig1_pr_curves.png`: Precision–Recall curves;
- `fig2_hidden_recall.png`: hidden-positive recall in Top-K lists;
- `fig3_profiles.png`: behavioral profiles of student groups;
- `fig4_score_scatter.png`: score comparison between naive and Spy models.

Run the sensitivity analyses:

```bash
python3 sensitivity.py
```

By default, five seeds (42–46) are used to study:

1. performance as the labeling-mechanism slope `beta` increases from SCAR (`beta=0`) to stronger instance dependence;
2. the effect of removing consumption-level, consumption-structure, behavioral-regularity, and non-card-record feature groups.

Outputs are written to `results/`, including `sweep_raw.csv`, `sweep_agg.csv`, `ablation_raw.csv`, and `ablation_agg.csv`.

## Real-data mode

Real-data mode requires a prepared feature table at:

```text
data/features_real.csv
```

Run it with:

```bash
python3 run_experiments.py --mode real --outdir results_real
```

A complete ground-truth poverty label is generally unavailable in real applications. Therefore, this mode uses `labeled` as the available evaluation label. Any research report should clearly discuss the resulting missing-label and selection biases.

## Evaluation metrics

- **PR-AUC**: overall ranking quality, especially useful when positives are rare;
- **P@K**: proportion of true positives among the top K% of students;
- **R@K**: proportion of all true positives covered by the top K% list;
- **HiddenR@K**: proportion of hidden positives recovered in the top K% list.

`multi_seed_agg.csv` contains means and standard deviations across multiple random seeds and can be used for tables or plots.

## Reproducibility notes

The train/test split is stratified by the combination of `y_true` and `labeled`, uses a 30% test set, and defaults to random seed 7. If you change data-generation parameters, feature sets, or seeds, record those changes with the reported results.

## Citation and responsible use

This repository is intended for research prototypes, method comparisons, and paper-reproduction experiments. Synthetic data validate behavior under a known mechanism, while real-data analyses require additional discussion of label bias, privacy, and fairness.
