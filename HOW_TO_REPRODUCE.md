# Quick Reproduction Guide

This guide walks you through running the full pipeline from start to finish on a fresh Kaggle GPU instance (Tesla T4).

All commands are standard terminal commands that you can run directly inside Jupyter notebook cells with a `!` prefix.

---

## 1. Kaggle Environment Setup

1. **Accelerator**: Select **GPU T4 x2** or **GPU T4**.
2. **Internet**: Turn **ON** (in notebook settings on the right panel).
3. **Attach Datasets** (Add Input):
   - `lokeshgile/dataset-eeg` (Contains DTU preprocessed `.mat` files)
   - `lokeshgile/eeg-audio` (Contains DTU raw `.wav` trial audio files)
   - `lowk1ee/kul-preprocessed-cache` (Contains KUL preprocessed `S1.pt` to `S16.pt`)

---

## 2. Clone & Setup Workspace

In your first cell, clone the repository and navigate into it:

```bash
cd /kaggle/working
!git clone https://github.com/Loki3306/uscapes_draft.git
%cd /kaggle/working/uscapes_draft

# Link the preprocessed KUL cache into data/
!mkdir -p data
!ln -s /kaggle/input/datasets/lowk1ee/kul-preprocessed-cache/data/processed_kul data/processed_kul
```

---

## 3. Part 1: DTU Experiments (ContrastiveMatchNet)

### Step 3.1: Extract Gammatone Audio Envelopes (~8 mins)
Extracts 28 sub-band envelopes for the 129 DTU trial audio files:

```bash
!python src/dtu/data/extract_gammatone_envelopes.py \
    --audio_dir /kaggle/input/datasets/lokeshgile/eeg-audio \
    --out_file data/gammatone_envelopes.pkl
```

### Step 3.2: Train MatchNet on 18 DTU Subjects (~14 mins)
Trains the 8-channel EEGNet + AudioCNN across all 18 LOSO folds:

```bash
!python scripts/dtu/train_matchnet_loso.py \
    --dtu_dir /kaggle/input/datasets/lokeshgile/dataset-eeg \
    --env_file data/gammatone_envelopes.pkl
```
*Expected output: Baseline 10s window accuracy ~67.5% – 69.0%, Zero EEG control ~48.8%, Shuffled control ~50.5%.*

### Step 3.3: Export Predictions for Confidence Modeling (<1 min)
Evaluates the saved checkpoints to generate window-level cosine similarities:

```bash
!python scripts/dtu/export_matchnet_predictions.py \
    --dtu_dir /kaggle/input/datasets/lokeshgile/dataset-eeg \
    --env_file data/gammatone_envelopes.pkl \
    --out_file matchnet_predictions.csv
```

### Step 3.4: Train & Evaluate Confidence Model (<1 min)
Trains the label-invariant XGBoost model on 5 temporal features:

```bash
# Train the production confidence model
!python scripts/dtu/train_confidence_model.py --csv matchnet_predictions.csv

# Run nested LOSO feature ablation audit
!python scripts/dtu/eval_confidence_ablations.py --csv matchnet_predictions.csv
```
*Expected output: Full 5-feature model AUROC ~0.75 – 0.81 (outperforming raw margin ~0.65 – 0.66).*

### Step 3.5: Run Selective Prediction Gating (<1 min)
Evaluates accuracy when low-confidence windows are rejected:

```bash
!python scripts/dtu/eval_selective_prediction.py --csv matchnet_predictions.csv
```
*Expected output: Accuracy climbs from ~67% at 100% coverage up to ~85% at 50% coverage.*

---

## 4. Part 2: KUL Experiments (AAD-Conformer)

### Step 4.1: Verify Historical Multi-Seed Baseline (<5 secs)
Checks the verified 5-seed cohort artifact (Seeds 1, 7, 21, 42, 123):

```bash
!python scripts/kul/eval_multiseed_repro.py
```
*Expected output: Grand Mean 77.12% ± 9.99% (subject std) / ± 3.10% (seed std).*

### Step 4.2: Train KUL Conformer on Seed 1 (~8 mins)
Trains the 8-channel Conformer across all 16 KUL subjects:

```bash
!python scripts/kul/train_conformer_loso.py --seeds 1 --epochs 40
```
*Expected output: Mean trial accuracy across 16 folds ~72% – 74%. Checkpoints saved in `checkpoints/kul/`.*

### Step 4.3: Train Evidential Confidence Head with Outlier Exposure (~3 mins)
Fine-tunes the late-fusion confidence head using 25% synthetic noise to reject out-of-distribution inputs:

```bash
!python scripts/kul/train_conformer_multitask_loso.py
```
*Expected output: Validation margin ~0.026 across folds. Checkpoints saved in `checkpoints/kul_multitask/`.*

### Step 4.4: Evaluate Evidential Confidence & Calibration (<1 min)
Computes calibration metrics and tests corrupted input collapse:

```bash
!python scripts/kul/eval_confidence_multitask.py
```
*Expected output: AUROC ~0.73, Global ECE < 0.10, Brier score ~0.21. Confidence collapses from ~0.56 (clean) down to ~0.09 (zero EEG).*

---

## 5. Part 3: Cross-Dataset Transfer (KUL → DTU)

Tests how well the KUL-trained model transfers zero-shot to all 18 DTU subjects without any fine-tuning (~1 min):

```bash
!python scripts/cross_dataset/eval_kul_to_dtu_transfer.py \
    --dtu_dir /kaggle/input/datasets/lokeshgile/dataset-eeg \
    --checkpoint checkpoints/kul/model_S1.pt
```

*Expected output:*
- **Method A (Discrete Majority Vote across windows)**: ~53.9% (chance level baseline)
- **Method C (Accumulated Pearson Correlation)**: ~68.1% (matches continuous evidence transfer)

---

## Quick Reference Summary Table

| Step | Script | Primary Output | Typical Runtime |
|---|---|---|---|
| **Audio Envelopes** | `src/dtu/data/extract_gammatone_envelopes.py` | `data/gammatone_envelopes.pkl` | ~8 min |
| **DTU MatchNet** | `scripts/dtu/train_matchnet_loso.py` | ~67.5% – 69.0% baseline accuracy | ~14 min |
| **DTU Confidence** | `scripts/dtu/eval_confidence_ablations.py` | AUROC ~0.75 – 0.81 | <1 min |
| **DTU Selective** | `scripts/dtu/eval_selective_prediction.py` | ~85% accuracy @ 50% coverage | <1 min |
| **KUL Conformer** | `scripts/kul/train_conformer_loso.py` | ~72% – 74% Seed 1 accuracy | ~8 min |
| **KUL Evidential** | `scripts/kul/eval_confidence_multitask.py` | AUROC ~0.73, ECE ~0.08, Noise collapse | <1 min |
| **Cross-Dataset** | `scripts/cross_dataset/eval_kul_to_dtu_transfer.py` | 53.9% (Method A) vs. 68.1% (Method C) | ~1 min |

Total runtime for the complete pipeline is around **30–35 minutes** on a single Tesla T4 GPU.
