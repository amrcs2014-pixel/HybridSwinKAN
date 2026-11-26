# HybridSwinKAN

> **Dual-Branch Swin–ConvNeXt Architecture with Kolmogorov–Arnold Networks for Explainable Network Intrusion Detection**

Official implementation and reproducibility package for the paper
**"HybridSwinKAN: Dual-Branch Swin–ConvNeXt Architecture with KAN for Explainable Network Intrusion Detection"**
(submitted to *International Journal of Computer Network and Information Security*, IJCNIS; manuscript ID **PAPER020705**).

---

## Overview

HybridSwinKAN is a dual-branch deep-learning architecture for Network Intrusion Detection Systems (NIDS) that couples three previously-separate ideas into a single end-to-end model:

1. **Swin Transformer with GR-KAN FFN** — a hierarchical shifted-window transformer in which every block's MLP feed-forward sub-layer is replaced with a Group-Rational Kolmogorov–Arnold Network (GR-KAN), yielding a 3–4× parameter reduction in the FFN and learnable rational activations.
2. **ConvNeXt local branch** — depth-wise separable convolutions for short-range inter-feature interactions.
3. **Dual-attention gating fusion** — a per-sample, per-dimension gate that combines the two branches adaptively.
4. **B-Spline KAN classification head** — inherently interpretable spline activations whose learned curves *are* the decision function.

The repository reproduces every table, figure, and statistical test reported in the manuscript.

![Architecture overview](docs/architecture.png)
*(Add architecture diagram to `docs/architecture.png` before publishing. The current notebook also generates this figure under `figures/`.)*

---

## Headline Results

Evaluated on four benchmark datasets, with 5-fold stratified cross-validation and 95% BCa bootstrap confidence intervals.

| Dataset       | Classes | Accuracy (%) | F1 (%) | MCC    | AUC-ROC |
|---------------|:-------:|:------------:|:------:|:------:|:-------:|
| NSL-KDD       |    5    |   99.90      | 99.89  | 0.9985 | 0.9998  |
| UNSW-NB15     |   10    |   99.88      | 99.86  | —      | —       |
| CIC-IDS2017   |   15    |   99.92      | 99.91  | —      | —       |
| Kaggle Malware |   2    |   99.85      | —      | —      | —       |

Cross-dataset transfer (binary, no fine-tuning): accuracy in the 78–88% range; recovers to 95–98% with 10% target-dataset fine-tuning. See manuscript §5.5.

Deployment feasibility: ≈14,800 flows/s sustained on an A100 (FP32, batch 256); **9.4 MB** model on disk; ≤2 ms CPU per-sample latency at batch 32.

---

## Repository Structure

```
HybridSwinKAN/
├── notebooks/
│   └── HybridSwinKAN.ipynb      # Self-contained end-to-end pipeline (recommended)
├── src/
│   └── hybridswinkan.py          # Same code as the notebook, as a Python module
├── docs/
│   └── (place architecture diagrams, supplementary PDFs here)
├── README.md                     # This file
├── LICENSE                       # MIT License
├── CITATION.cff                  # Citation metadata (BibTeX exported below)
├── requirements.txt              # Python dependencies
└── .gitignore
```

---

## Requirements

- **Python** ≥ 3.9
- **PyTorch** ≥ 2.1 (CUDA 11.8+ recommended; CPU works but is slow)
- **GPU** with ≥ 8 GB VRAM for training from scratch (A100 used in the paper; any recent NVIDIA GPU works)
- **RAM** ≥ 16 GB for CIC-IDS2017 preprocessing

Full dependency list in [`requirements.txt`](requirements.txt). The notebook also auto-installs everything via `pip` on first run, so on Google Colab / Kaggle you can skip manual installation.

---

## Quick Start

### Option A — Jupyter Notebook (recommended)

```bash
git clone https://github.com/amrogomma-web/HybridSwinKAN.git
cd HybridSwinKAN
pip install -r requirements.txt
jupyter notebook notebooks/HybridSwinKAN.ipynb
```

Then run all cells. On synthetic data (the default), the full pipeline finishes in ~20 minutes on an A100.

### Option B — As a Python script

```bash
python src/hybridswinkan.py
```

To use real datasets, edit the `NSL_KDD_DIR`, `UNSW_DIR`, and `CICIDS_DIR` constants near the bottom of the file, or call `main_pipeline(...)` from your own driver script.

### Option C — Google Colab

Upload `notebooks/HybridSwinKAN.ipynb` to Colab, mount Drive if you want to use real datasets, and Run All. The first cell auto-installs all dependencies.

---

## Datasets

The paper uses four publicly-available datasets:

| Dataset        | Link                                                                 |
|----------------|----------------------------------------------------------------------|
| NSL-KDD        | https://www.unb.ca/cic/datasets/nsl.html                             |
| UNSW-NB15      | https://research.unsw.edu.au/projects/unsw-nb15-dataset               |
| CIC-IDS2017    | https://www.unb.ca/cic/datasets/ids-2017.html                        |
| Kaggle Malware | Auto-downloaded via `kagglehub` on first use (API key required)      |

We do **not** redistribute the datasets. Users must download them from the respective providers and accept the datasets' license terms.

A synthetic-data fallback is built in, so you can smoke-test the pipeline before downloading anything.

---

## Reproducing the Paper

Every result in the manuscript is produced by `main_pipeline(...)` with the following switches:

| Experiment                              | Flag                                     |
|-----------------------------------------|------------------------------------------|
| Headline accuracy on four benchmarks    | `run_baselines=True`                     |
| Ablation study (Table 6)                | `run_ablation=True`                      |
| 5-fold stratified cross-validation      | `run_nested_cv=True`                     |
| SHAP + Integrated Gradients + KAN viz   | `run_xai=True`                           |
| Optuna hyperparameter search            | `run_optuna=True` (long — ≥ 6 h on A100) |

All random seeds are fixed to `SEED = 42`. Statistical tests (Wilcoxon, Friedman–Nemenyi, McNemar, bootstrap CI with BCa) are applied automatically; results are dumped to `results/` and figures to `figures/`.

Hardware used in the paper: NVIDIA **A100 40 GB** GPU (training) and Intel Xeon Gold 6248 16-core CPU (CPU-latency benchmark).

Expected wall-clock on A100 (synthetic fallback → real datasets):
- Full `main_pipeline()` with all four datasets: ≈ 6–8 hours.
- Single dataset, no Optuna, no ablation: ≈ 30–40 minutes.

---

## Explainability

The pipeline runs a four-method XAI stack:

1. **Inherent KAN interpretability** — `MultiMethodXAI.visualize_kan_activations()` plots the learned B-spline activation curves directly.
2. **SHAP** — Kernel SHAP feature attributions.
3. **Integrated Gradients** — via `captum` (optional; falls back to a manual implementation if `captum` is unavailable).
4. **Swin attention heatmaps** — attention rollout aggregated across heads and layers.

Each method also comes with a faithfulness / robustness metric (see §5.12 of the paper).

---

## Citation

If you use this code, please cite the paper:

```bibtex
@article{abdelhaliem2026hybridswinkan,
  title   = {HybridSwinKAN: Dual-Branch Swin--ConvNeXt Architecture with KAN for Explainable Network Intrusion Detection},
  author  = {Abdelhaliem, Amr H. and Tawfik, Mohammed and Fathi, Islam S.},
  journal = {International Journal of Computer Network and Information Security (IJCNIS)},
  year    = {2026},
  note    = {Manuscript ID: PAPER020705}
}
```

A machine-readable `CITATION.cff` file is also provided so that GitHub shows a "Cite this repository" button automatically.

---

## License

Code: **MIT License** — see [`LICENSE`](LICENSE).
Datasets retain their original licenses; see the Datasets section above.

---

## Contact

**Dr. Amr H. Abdelhaliem** (corresponding author)
Assistant Professor and Head, Department of Cyber Security
Faculty of Science and Information Technology
Irbid National University, Jordan
Email: [a.hasan@inu.edu.jo](mailto:a.hasan@inu.edu.jo)
ORCID: [0000-0002-6097-3993](https://orcid.org/0000-0002-6097-3993)

**Co-authors**:
- Mohammed Tawfik — Ajloun National University, Jordan
- Islam S. Fathi — Ajloun National University, Jordan

For bugs or methodological questions, please open an [Issue](https://github.com/amrcs2014-pixel/HybridSwinKAN/issues); for collaboration enquiries, please email the corresponding author.

---

## Acknowledgments

The authors thank the IJCNIS editorial office and the anonymous reviewers for their constructive feedback.
