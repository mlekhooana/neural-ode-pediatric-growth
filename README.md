# neural-ode-pediatric-growth
Code and synthetic data for Paper 1: coupled Latent Neural ODE for pediatric growth
# Coupled Pediatric Growth Forecasting with a Latent Neural ODE

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mlekhooana/neural-ode-pediatric-growth/blob/main/Paper1_Experiments.ipynb)

Code and synthetic data accompanying the paper:

> **Continuous-Time Monitoring of Pediatric Growth: A Coupled Neural ODE Approach to Forecasting, Classification, and Cross-Variable Inference**
> Mosito Lekhooana and Montaz Ali.
> *[Journal name]*, [year]. DOI: [to be added]

The repository contains a probabilistic latent Neural ODE that forecasts the joint growth of
**weight (W), length (L) and head circumference (HC)** from birth to 60 months, given a child's
measurements over the first 24 months. A **coupled** model (one shared vector field for W, L
and HC) is compared against a matched **uncoupled** model (three independent univariate
models with every other setting identical).

---

## Repository contents

| File | Description |
|------|-------------|
| `generate_cohort.ipynb` | Generates the synthetic pediatric cohort. |
| `pediatric_cohort_geraedts_who.csv` | The synthetic cohort used in the paper. |
| `CoupledVsUncoupled.ipynb` | Trains the coupled (Experiment 2) and uncoupled (Experiment 1) models on the same split and compares them.
| `PartialHistory.ipynb` | Compares the results from coupled (Experiment 2) as history is thinned.
| `CrossVariableInference.ipynb` | Reconstructs the unobserved variable when the other two are observed.
| `LICENSE` | MIT licence (code). |

---

## Data

The cohort is **fully synthetic**; no real patient data are used, so it can be shared freely.

- 1,020 boys, measured at 12 visit ages: 0, 1.5, 3, 6, 9, 12, 18, 24 months
  (history window) and 30, 36, 48, 60 months (forecast horizons).
- Growth is generated from the official **WHO Multicentre Growth Reference Study (MGRS)**
  LMS parameters for boys.
- Cross-variable coupling between W, L and HC is calibrated to Geraedts et al. (2011).
- Columns: `Child_ID`, `Age_Months`, true values (`*_true`), observed (noisy) values,
  WHO z-scores (`z_W`, `z_L`, `z_C`) and an `Observed` flag for missed visits.

Running `GenerateCohort.ipynb` reproduces the CSV exactly (fixed random seed).

---

## Method summary

- **State:** WHO z-score vector (z_W, z_L, z_C); the LMS reference is recovered from the
  training children.
- **Encoder:** bidirectional GRU over the 0–24 month history; the forecast is launched at
  24 months and the same vector field is integrated backward to reconstruct the history.
- **Output:** a predictive mean and standard deviation per variable and age
  (β-NLL loss, β = 0.5).
- **Risk flagging:** Healthy / Warning / Critical zones (p15/p85 and p3/p97 bands), flagged on
  P(Critical) and P(not Healthy) with thresholds tuned on the validation set to 80% recall.
- **Faltering augmentation:** synthetic sustained-decline children are added to the training
  set only; a faltering stress set built from test children is scored separately.

---

## How to reproduce the results

### Option A: Google Colab (recommended)

1. Click the **Open in Colab** badge at the top of this page.
2. Select **Runtime → Run all**.

The cohort CSV is downloaded from this repository automatically, and `torchdiffeq` is
installed if missing. A GPU runtime is optional but faster.

### Option B: Local machine

```bash
git clone https://github.com/mlekhooana/neural-ode-pediatric-growth.git
cd neural-ode-pediatric-growth
pip install torch torchdiffeq pandas numpy matplotlib scipy
jupyter notebook Paper1_Experiments.ipynb
```

### Experimental settings (fixed for all reported results)

| Setting | Value |
|---------|-------|
| Random seed | 42 |
| Train / validation / test split | 70 / 15 / 15 % (714 / 153 / 153 children) |
| Epochs | 300 |
| Optimiser | Adam, learning rate 1e-3, cosine schedule |
| Batch size | 128 |
| ODE solver | `dopri5` (adaptive) |
| Illustrative children | Child 201 (critical) and Child 1008 (healthy), both forced into the test set |

Both models are trained in one run on the same split, the same augmented training set and the
same stress set. Results are reported with paired-bootstrap 95% intervals over test children.

---

## Outputs

| Location | Contents |
|----------|----------|
| `E2_coupled_v3/` | Coupled model: weights (`coupled_growth_ode_v2.pt`), loss curve, coupling plot, per-child figures and statistics |
| `E1_uncoupled_v3/` | Uncoupled model: the same outputs |
| `comparison_summary.csv` | Coupled vs uncoupled metrics on the test and stress sets |
| `comparison_children.csv` | Coupled vs uncoupled results for the illustrative children |

---

## Citation

If you use this code or data, please cite the paper above.

## Licence

Code: MIT Licence (see `LICENSE`). Synthetic data: CC BY 4.0.

## Contact

Mosito Lekhooana, Department of Mathematics and Computer Science, National University of
Lesotho, and School of Computer Science and Applied Mathematics, University of the
Witwatersrand.
Email: m.lekhooana@nul.ls · ORCID: [0009-0006-6281-8416](https://orcid.org/0009-0006-6281-8416)
