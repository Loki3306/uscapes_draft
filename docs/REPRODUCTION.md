# Step-by-Step Reproduction Guide

This guide provides the complete, runnable sequence of commands to reproduce the empirical results from:

> **"Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control"**

All commands assume execution from the root directory of `paper-reproduction/`.

---

## Stage A: Environment Setup

### 1. Create and Activate Environment

Using Conda:
```bash
conda env create -f environment.yml
conda activate aad-paper-reproduction
```

Or using standard Python venv:
```bash
python -m venv venv
# On Linux/macOS:
source venv/bin/activate
# On Windows:
.\venv\Scripts\activate

pip install -r requirements.txt
```

### 2. Verify Installation and Imports

Run the test suite to ensure all modules, models, and dependencies compile and resolve:
```bash
python -c "
import models.matchnet
import models.aad_conformer
import models.eegnet
import src.common.utils
import src.dtu.confidence.selective_metrics
import xgboost
print('Environment check passed!')
"
```

---

## Stage B: Dataset Setup

Ensure datasets are organized according to [docs/DATASET_SETUP.md](DATASET_SETUP.md):

- **DTU**: `.mat` files for 18 subjects placed at `data/dtu/`
- **KUL**: Preprocessed or raw MAT files for 16 subjects placed at `data/kul/`

---

## Stage C: DTU Preprocessing & Data Extraction

### 1. Extract 28-Band Gammatone Acoustic Envelopes
Computes cochlear Gammatone representations (50–8000 Hz, 28 ERB subbands, $\gamma = 0.6$ power compression, 8 Hz lowpass, resampled to 64 Hz):
```bash
python src/dtu/data/extract_gammatone_envelopes.py \
    --audio_dir data/dtu_audio \
    --out_file data/gammatone_envelopes.pkl
```

---

## Stage D: DTU Model Training

### 1. Train ContrastiveMatchNet Under Leave-One-Subject-Out (LOSO)
Trains `ContrastiveMatchNet` (8 EEG channels, 28 Gammatone subbands, latent dim 64) across all 18 DTU subjects using 5s windows with 2s hop:
```bash
python scripts/dtu/train_matchnet_loso.py \
    --data_dir data/dtu \
    --epochs 30 \
    --batch_size 64 \
    --lr 0.001
```
Checkpoints will be saved to `checkpoints/`.

---

## Stage E: DTU Evaluation & Prediction Export

### 1. Export 5,400 Window Predictions to CSV
Evaluates trained LOSO checkpoints across 5,400 non-overlapping evaluation windows and exports prediction metrics (`sim_A`, `sim_B`, `margin`, `correct`):
```bash
python scripts/dtu/export_matchnet_predictions.py \
    --checkpoint_dir checkpoints \
    --out_csv matchnet_predictions.csv \
    --window_sec 10.0
```
*Expected Result*: Reproduces the canonical DTU LOSO accuracy (~69.02% / 68.19%).

---

## Stage F: DTU Confidence Modeling & Selective Evaluation

### 1. Fit the Canonical XGBoost Confidence Model
Trains the XGBoost model on the 5 label-invariant features (`margin`, `sim_chosen`, `sim_unchosen`, `rolling_std_margin`, `trial_consistency`):
```bash
python scripts/dtu/train_confidence_model.py \
    --csv matchnet_predictions.csv \
    --out_model models/confidence_model.json
```

### 2. Run Confidence Feature Ablations (M1–M5)
Computes nested LOSO AUROC across feature subsets:
```bash
python scripts/dtu/eval_confidence_ablations.py \
    --csv matchnet_predictions.csv
```
*Expected Output*:
- `M1_Margin` (Margin-only): AUROC = **0.6601**
- `M2_Consistency`: AUROC = **0.5840**
- `M3_Margin_Consist`: AUROC = **0.7214**
- `Full` (all 5 features): AUROC = **0.8057**

### 3. Evaluate Selective Prediction Curves
Sweeps confidence threshold $\tau$ and measures selective accuracy across coverage levels:
```bash
python scripts/dtu/eval_selective_prediction.py \
    --csv matchnet_predictions.csv
```
*Expected Output*:
- **81.55%** selective accuracy at **70.0%** coverage ($\tau = 0.65$)
- **86.00%** selective accuracy at **50.0%** coverage

---

## Stage G: KUL Preprocessing & Cache Construction

### 1. Build Preprocessed KUL Cache
Executes Common Average Referencing (CAR) across 64 channels, applies 1–8 Hz bandpass filtering, extracts 28-band Gammatone envelopes, and subsamples the 8 selected electrodes (`T7, C2, FT8, P7, CPz, Fp1, TP8, C3`):
```bash
python src/kul/data/build_kul_cache.py \
    --raw_dir data/kul \
    --out_dir data/processed_kul
```

---

## Stage H: KUL Model Training

### 1. Train KUL AAD-Conformer Under LOSO
Trains the 8-channel AAD-Conformer (2 layers, 4 heads, embed_dim 64) across 16 subjects:
```bash
python scripts/kul/train_conformer_loso.py \
    --cache_dir data/processed_kul \
    --seed 1 \
    --epochs 40
```

### 2. Multi-Task Training with Outlier Exposure (Confidence Head)
Fine-tunes the Conformer with the evidential confidence head using negative Pearson correlation + BCE confidence loss on Outlier Exposure samples:
```bash
python scripts/kul/train_conformer_multitask_loso.py \
    --cache_dir data/processed_kul \
    --seed 1 \
    --epochs 20
```

---

## Stage I: KUL Evaluation & Multi-Seed Verification

### 1. Evaluate Multi-Seed Reproducibility (77.12% Grand Mean)
Evaluates the multi-seed summary across seeds 1, 7, 21, 42, and 123:
```bash
python scripts/kul/eval_multiseed_repro.py
```
*Expected Output*:
```
=== SEED LEVEL STATISTICS ===
Seed 1: Accuracy = 71.9%, Median Margin = 0.0208
Seed 123: Accuracy = 80.6%, Median Margin = 0.0203
Seed 21: Accuracy = 78.1%, Median Margin = 0.0226
Seed 42: Accuracy = 75.6%, Median Margin = 0.0216
Seed 7: Accuracy = 79.4%, Median Margin = 0.0222

=== OVERALL REPRODUCIBILITY ===
Global Accuracy: 77.12% ± 3.10% (CV: 4.0%)
Global Margin: 0.0215 ± 0.0009 (CV: 4.0%)
```

---

## Stage J: KUL Confidence, Selective Gating & Controls

### 1. Evaluate Evidential Confidence Metrics
Evaluates AUROC, Expected Calibration Error (ECE), and corruption sensitivity:
```bash
python scripts/kul/eval_confidence_multitask.py
```
*Expected Output*:
- AUROC: **0.7337**
- ECE: **0.0998**
- Brier Score: **0.2115**

### 2. Evaluate Selective AAD
Computes selective accuracy at the high-confidence threshold $\tau = 0.70$:
```bash
python scripts/kul/eval_selective_aad.py
```
*Expected Output*:
- Selective Accuracy: **94.94%** at **12.32%** coverage ($\tau = 0.70$)

### 3. Evaluate Falsification & Corruption Controls
Evaluates model behavior under Zero-EEG, Shuffled Labels, and Random Gaussian noise:
```bash
python scripts/kul/eval_falsification.py
```

### 4. Evaluate Noise Robustness & Channel Miniaturization
Sweeps AWGN SNR (-10 dB to +10 dB) and drops channels from 8 down to 5, 3, and 1:
```bash
python scripts/kul/eval_robustness.py
```

---

## Stage K: Zero-Shot Cross-Dataset Transfer (KUL → DTU)

### 1. Evaluate Frozen KUL Conformer on DTU Dataset
Executes zero-shot transfer using the frozen `model_S1.pt` checkpoint on all 18 DTU subjects (1,080 trials) without retraining or parameter selection:
```bash
python scripts/cross_dataset/eval_kul_to_dtu_transfer.py \
    --dtu_dir data/dtu \
    --checkpoint checkpoints/kul/model_S1.pt
```
*Expected Output*:
- **Accumulated Pearson Trial Accuracy**: **68.24%**
- **Trial Majority Vote Accuracy**: **54.26%**
- **Window-Level Accuracy**: **52.84%**

---

## Stage L: Publication Figures Generation

### 1. Generate Figures 1–5
```bash
python figures/generate_publication_figures.py \
    --csv_file matchnet_predictions.csv \
    --model_path models/confidence_model.json
```
Figures will be generated in `analysis/figures/publication/`:
- `figure1_calibration.png`
- `figure2_roc_curves.png`
- `figure3_reliability.png`
- `figure4_selective_risk.png`
- `figure5_shap_importance.png`
