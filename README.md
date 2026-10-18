# Decision-Gated Auditory Attention Decoding from Low-Density EEG
## Confidence-Conditioned Neural Inference for State-Aware Auditory Control

This repository contains the official code, configuration, scripts, and documentation accompanying the paper:

> **"Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control"**

---

> [!IMPORTANT]
> **Reproduction Scope:**  
> This repository provides the preprocessing, model implementations, training procedures, evaluation scripts, configurations, and analysis required to reproduce the experiments reported in the paper. **Trained model checkpoints are not included.** Running the training procedures generates the required subject-specific models locally.

---

## 1. High-Level Architecture & Reproduction Flow

The repository is organized around two end-to-end experimental tracks (DTU and KUL) and a zero-shot cross-dataset evaluation track:

```
README
  │
  ▼
Dataset Setup  (Place raw DTU / KUL data in data/)
  │
  ▼
Configuration  (configs/dtu_matchnet.json, configs/kul_conformer.json)
  │
  ▼
Training Scripts  (Generate LOSO models locally)
  │
  ▼
Checkpoints Generated Locally  (checkpoints/dtu/, checkpoints/kul/)
  │
  ▼
Evaluation Scripts & Confidence Modeling
  │
  ▼
Reported Metrics & Publication Figures
```

### Detailed Pipeline Maps

#### Track 1: DTU Pipeline (ContrastiveMatchNet & XGBoost Confidence)
```text
DTU Dataset (18 Subjects)
        │
        ▼
scripts/dtu/train_matchnet_loso.py
        │
        ▼
18 Subject-Specific Models (saved to checkpoints/dtu/)
        │
        ▼
scripts/dtu/export_matchnet_predictions.py
        │
        ▼
DTU Window Predictions CSV (matchnet_predictions.csv, ~68.19% / 69.02% accuracy)
        │
        ▼
scripts/dtu/train_confidence_model.py
        │
        ▼
Trained XGBoost Confidence Model (models/confidence_model.json, AUROC 0.8057)
        │
        ▼
scripts/dtu/eval_selective_prediction.py & eval_confidence_ablations.py
        │
        ▼
Selective Accuracy (81.55% @ 70% coverage, 86.00% @ 50% coverage)
```

#### Track 2: KUL Pipeline (AAD-Conformer & Evidential Confidence)
```text
KUL Dataset (16 Subjects)
        │
        ▼
scripts/kul/train_conformer_loso.py
        │
        ▼
16 Subject-Specific Conformer Models (saved to checkpoints/kul/)
        │
        ▼
scripts/kul/eval_multiseed_repro.py / eval_selective_aad.py
        │
        ▼
77.12% ± 9.99% Grand Mean LOSO Accuracy Across 5 Seeds
        │
        ▼
scripts/kul/train_conformer_multitask_loso.py  (Outlier Exposure fine-tuning)
        │
        ▼
Multitask Models with Evidential Confidence Head
        │
        ▼
scripts/kul/eval_confidence_multitask.py
        │
        ▼
AUROC 0.7337, ECE 0.0998, 94.94% Selective Accuracy @ 12.32% Coverage
```

#### Track 3: Cross-Dataset Evaluation (KUL → DTU Zero-Shot)
```text
Locally Trained KUL Conformer (model_S1.pt)
        │
        ▼
Frozen Evaluation on all 18 DTU Subjects (1,080 trials, zero target retraining)
        │
        ▼
scripts/cross_dataset/eval_kul_to_dtu_transfer.py
        │
        ▼
Accumulated Pearson: 68.24% | Majority Vote: 54.26% | Window-Level: 52.84%
```

---

## 2. Repository Structure

```text
uscapes_draft/
├── README.md                          # Main project overview and reproduction instructions
├── LICENSE                            # MIT License
├── requirements.txt                   # Pip requirements
├── environment.yml                    # Conda environment specification
│
├── configs/
│   ├── dtu_matchnet.json              # Full hyperparameter config for DTU pipeline
│   ├── kul_conformer.json             # Full hyperparameter config for KUL pipeline
│   └── cross_dataset_kul_to_dtu.json  # Config for zero-shot KUL -> DTU transfer
│
├── src/
│   ├── common/                        # Shared evaluation and I/O utilities
│   │   ├── utils.py                   # safe_corr_np, majority vote, normalizers
│   │   ├── ridge_aad.py               # Ridge regression baseline and loader logic
│   │   └── _common.py                 # Mat-file reader, DTU trial parsing
│   ├── dtu/                           # DTU-specific models and confidence
│   │   ├── models/
│   │   │   ├── matchnet.py            # ContrastiveMatchNet architecture
│   │   │   ├── eegnet.py              # EEGNet depthwise-separable encoder
│   │   │   ├── atcnet.py              # ATCNet module dependency
│   │   │   └── eegnet_tcn.py          # EEGNet-TCN module dependency
│   │   ├── data/
│   │   │   ├── extract_gammatone_envelopes.py # 28-band cochlear Gammatone extractor
│   │   │   └── audio_mapping.json     # 1,080 DTU trial audio mappings
│   │   ├── confidence/
│   │   │   ├── inference_engine.py    # Online FIFO confidence state tracker
│   │   │   └── selective_metrics.py   # Selective risk, coverage, and excess risk
│   │   └── montages.py                # Authoritative BioSemi channel indices & labels
│   └── kul/                           # KUL-specific models and data loading
│       ├── models/
│       │   └── aad_conformer.py       # AADConformer with evidential confidence head
│       └── data/
│           ├── build_kul_cache.py     # Preprocessing (CAR, 1-8 Hz, 8-channel selection)
│           └── kul_cached_dataset.py  # Fast memmap/cached data loader
│
├── scripts/
│   ├── dtu/                           # Runnable DTU training and evaluation scripts
│   │   ├── train_matchnet_loso.py     # Train ContrastiveMatchNet LOSO
│   │   ├── export_matchnet_predictions.py # Export window predictions to CSV
│   │   ├── train_confidence_model.py  # Fit canonical XGBoost confidence model
│   │   ├── eval_selective_prediction.py # Evaluate selective coverage sweep
│   │   └── eval_confidence_ablations.py # M1-M5 feature ablations (AUROC 0.66 -> 0.81)
│   ├── kul/                           # Runnable KUL training and evaluation scripts
│   │   ├── train_conformer_loso.py    # Train AADConformer LOSO
│   │   ├── train_conformer_multitask_loso.py # Train with Outlier Exposure confidence
│   │   ├── eval_multiseed_repro.py    # Multi-seed reproducibility verification
│   │   ├── eval_confidence_multitask.py # Evaluate evidential confidence head
│   │   ├── eval_selective_aad.py      # Evaluate selective accuracy and coverage
│   │   ├── eval_falsification.py      # Zero-EEG, permutation, and random EEG controls
│   │   └── eval_robustness.py         # AWGN SNR sweep and channel miniaturization
│   └── cross_dataset/
│       └── eval_kul_to_dtu_transfer.py # Zero-shot KUL -> DTU evaluation
│
├── checkpoints/
│   ├── dtu/                           # Empty local target directory for DTU models
│   ├── kul/                           # Empty local target directory for KUL models
│   └── README.md                      # Documentation on local checkpoint generation
│
├── results/
│   ├── kul_conformer_multiseed_summary.json # Raw multi-seed trial metrics across 5 seeds
│   └── README.md
│
├── figures/
│   └── generate_publication_figures.py # Generates publication figures (Calibration, ROC, Risk)
│
├── docs/
│   ├── DATASET_SETUP.md               # Raw dataset placement and directory structures
│   ├── REPRODUCTION_MAP.md            # Detailed end-to-end execution maps
│   ├── EXPERIMENT_MAP.md              # Master mapping from paper claims to exact code paths
│   ├── LIMITATIONS.md                 # Full documentation of methodology limits and variance
│   ├── PROVENANCE.md                  # Detailed code and dataset provenance audit
│   ├── COPY_AUDIT.md                  # Cryptographic SHA-256 byte comparison table
│   └── FINAL_FORENSIC_AUDIT.md        # Comprehensive forensic audit evaluation
│
└── paper/
    └── README.md                      # Paper abstract, structure, and formal citations
```

---

## 3. Environment Setup

### Using Conda (Recommended)

```bash
conda env create -f environment.yml
conda activate aad-paper-reproduction
```

### Using Pip

```bash
python -m venv venv
# On Linux/macOS:
source venv/bin/activate
# On Windows:
.\venv\Scripts\activate

pip install -r requirements.txt
```

---

## 4. Dataset Placement

Place raw datasets in the designated local folders:
- **DTU Dataset**: `data/dtu/` (expects `S1.mat` through `S18.mat`).
- **KUL Dataset**: `data/kul/` (expects raw trial folders for subjects `S1` through `S16`).

See [docs/DATASET_SETUP.md](docs/DATASET_SETUP.md) for complete formatting details.

---

## 5. End-to-End Execution Guide

### Track 1: DTU Pipeline Execution

```bash
# 1. Preprocess audio: extract 28-band cochlear Gammatone envelopes
python src/dtu/data/extract_gammatone_envelopes.py \
    --audio_dir data/dtu_audio \
    --out_file data/gammatone_envelopes.pkl

# 2. Train ContrastiveMatchNet LOSO across 18 DTU subjects
python scripts/dtu/train_matchnet_loso.py \
    --data_dir data/dtu \
    --epochs 30 \
    --batch_size 64

# 3. Export 5,400 window predictions to CSV
python scripts/dtu/export_matchnet_predictions.py \
    --checkpoint_dir checkpoints/dtu \
    --out_csv matchnet_predictions.csv

# 4. Train the canonical XGBoost confidence model
python scripts/dtu/train_confidence_model.py \
    --csv matchnet_predictions.csv \
    --out_model models/confidence_model.json

# 5. Evaluate confidence feature ablations (M1-M5)
python scripts/dtu/eval_confidence_ablations.py \
    --csv matchnet_predictions.csv

# 6. Evaluate selective prediction coverage sweep
python scripts/dtu/eval_selective_prediction.py \
    --csv matchnet_predictions.csv
```

### Track 2: KUL Pipeline Execution

```bash
# 1. Build preprocessed KUL cache (CAR across 64 ch, 1-8 Hz filter, 8 ch subsampling)
python src/kul/data/build_kul_cache.py \
    --raw_dir data/kul \
    --out_dir data/processed_kul

# 2. Train base AAD-Conformer under LOSO
python scripts/kul/train_conformer_loso.py \
    --cache_dir data/processed_kul \
    --seed 1 \
    --epochs 40

# 3. Verify multi-seed reproducibility summary
python scripts/kul/eval_multiseed_repro.py

# 4. Fine-tune with evidential confidence head & Outlier Exposure (OE)
python scripts/kul/train_conformer_multitask_loso.py \
    --cache_dir data/processed_kul \
    --seed 1 \
    --epochs 20

# 5. Evaluate evidential confidence metrics (AUROC 0.7337, ECE 0.0998)
python scripts/kul/eval_confidence_multitask.py

# 6. Evaluate selective accuracy (94.94% @ 12.32% coverage at tau = 0.70)
python scripts/kul/eval_selective_aad.py

# 7. Evaluate falsification & corruption controls (Zero EEG, Random EEG, Permutation)
python scripts/kul/eval_falsification.py
```

### Track 3: Zero-Shot Cross-Dataset Transfer (KUL → DTU)

```bash
# Evaluate locally trained KUL Conformer (model_S1.pt) directly on DTU
python scripts/cross_dataset/eval_kul_to_dtu_transfer.py \
    --dtu_dir data/dtu \
    --checkpoint checkpoints/kul/model_S1.pt
```

---

## 6. Numerical Variance & Known Limitations

- **Stochastic Optimization Variance**: The paper reports **69.02%** DTU accuracy. Canonical fresh training reproduces **68.19%** across 5,400 windows due to unpinned SGD optimizer seeds. The experimental procedure is what is reproduced.
- **Global Channel Selection Disclosure**: The 8 DTU channels (`C5, FCz, FC6, P9, C6, Fp1, TP8, T7`) were selected globally across all 18 subjects via Ridge regression prior to LOSO cross-validation, and include vertex electrode `FCz`.
- For complete disclosure of scientific provenance, result traceability, and methodology limits, see:
  - [docs/RESULT_TRACEABILITY.md](docs/RESULT_TRACEABILITY.md) (Master traceability table for every numerical claim)
  - [docs/FINAL_SUBMISSION_CHECK.md](docs/FINAL_SUBMISSION_CHECK.md) (10-point audit and paper coverage checklist)
  - [docs/LIMITATIONS.md](docs/LIMITATIONS.md) (Channel selection provenance, stochastic variance, and montages)
  - [docs/PROVENANCE.md](docs/PROVENANCE.md) (Git source commits and dataset citations)

---

## 7. Intentionally Excluded Work

To ensure strict scientific integrity, the following components from exploratory or invalidated branches are explicitly **excluded** from this repository:
1. **Pretrained Binary Checkpoints**: Pretrained `.pt` files are excluded. In accordance with academic submission standards, the repository provides the experimental methodology and training procedures, with subject-specific models generated locally.
2. **Continuous Controller Switching Simulator**: Spliced session metrics (e.g., 92.58% lock, 22.53 switches/hr) were an artifact of static trial concatenation where speaker identities did not simulate genuine physiological switching.
3. **Leaked Early Confidence Model**: Early exploratory scripts with ~0.99 AUROC leaked ground-truth speaker identity through `sim_A`/`sim_B`. The canonical model uses 5 label-invariant features (AUROC = 0.8057).
4. **Post-Paper Explorations & Edge Hardware**: Architectures developed after paper submission (CA-TCN, MSCA-AAD, SincAlignNet, USCAPES edge runtime, STM32 embedded C firmware, and live browser UIs) are excluded to preserve the exact paper scope.
