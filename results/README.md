# Empirical Results Summary & Data Files

This directory contains the numerical experiment summaries, per-subject logs, and exported metric tables that reproduce the paper's reported values.

---

## 1. KUL Multi-Seed Evaluation Summary

### File: `results/kul_conformer_multiseed_summary.json`
- **Description**: Contains granular evaluation outputs across 5 random seeds (`1`, `7`, `21`, `42`, `123`) and all 16 KUL subjects (`S1` through `S16`).
- **Keys per subject record**:
  - `trial_accuracy`: Trial-level AAD classification accuracy (evaluated by trial Pearson correlation).
  - `median_margin`: Median correlation margin ($\rho_{\text{attended}} - \rho_{\text{unattended}}$).
  - `mean_margin`: Mean correlation margin across windows.
  - `window_accuracy`: Percentage of individual 10-s windows correctly classified.

### Master Statistics Reproducible From This JSON:
- **Seed 1**: Accuracy = `71.88%`, Median Margin = `0.0208`
- **Seed 7**: Accuracy = `79.38%`, Median Margin = `0.0222`
- **Seed 21**: Accuracy = `78.12%`, Median Margin = `0.0226`
- **Seed 42**: Accuracy = `75.62%`, Median Margin = `0.0216`
- **Seed 123**: Accuracy = `80.62%`, Median Margin = `0.0203`
- **Grand Mean Accuracy**: **`77.12% ± 3.10%`** (seed-level std) / **`77.12% ± 9.99%`** (subject-level std across 80 evaluations).

Run verification command:
```bash
python scripts/kul/eval_multiseed_repro.py
```

---

## 2. Generated Outputs Directory

When evaluation scripts are executed, they generate:
- `results/conformer_loso/reproducibility_table.csv`: CSV table of per-seed accuracy and margin statistics.
- `results/conformer_loso/figures/reproducibility_boxplot.png`: Boxplot across the 5 random seeds.
- `matchnet_predictions.csv`: 5,400 DTU evaluation window predictions exported by `scripts/dtu/export_matchnet_predictions.py`.
