# Reproduction Map: End-to-End Execution Workflows

This document outlines the explicit execution pathways required to reproduce every experiment reported in:
**"Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control"**

In this repository, **all models are generated locally through the training scripts**.

---

## 1. DTU Reproduction Map (ContrastiveMatchNet & Selective Gating)

```text
[ Raw DTU Data: data/dtu/ ] + [ Raw Audio: data/dtu_audio/ ]
                       │
                       ▼
   Step 1: Extract 28-Band Gammatone Envelopes
   Script: src/dtu/data/extract_gammatone_envelopes.py
   Output: data/gammatone_envelopes.pkl
                       │
                       ▼
   Step 2: Train 18 Subject-Specific Models (LOSO)
   Script: scripts/dtu/train_matchnet_loso.py
   Config: configs/dtu_matchnet.json
   Output: checkpoints/dtu/matchnet_S1.pt ... matchnet_S18.pt
                       │
                       ▼
   Step 3: Export Window Decisions & Margins (5,400 Windows)
   Script: scripts/dtu/export_matchnet_predictions.py
   Output: matchnet_predictions.csv (reproduces 68.19% / 69.02% baseline)
                       │
                       ▼
   Step 4: Train Canonical XGBoost Confidence Model
   Script: scripts/dtu/train_confidence_model.py
   Output: models/confidence_model.json
                       │
                       ▼
   Step 5: Evaluate AUROC & Confidence Feature Ablations (M1-M5)
   Script: scripts/dtu/eval_confidence_ablations.py
   Metrics: M1 (Margin-only) = 0.6601 | Full (5 features) = 0.8057
                       │
                       ▼
   Step 6: Evaluate Selective Prediction & Accuracy Gain
   Script: scripts/dtu/eval_selective_prediction.py
   Metrics: 81.55% @ 70% coverage (τ = 0.65) | 86.00% @ 50% coverage
                       │
                       ▼
   Step 7: Generate Publication Figures (Figures 1-5)
   Script: figures/generate_publication_figures.py
   Output: analysis/figures/publication/
```

---

## 2. KUL Reproduction Map (AAD-Conformer & Evidential Head)

```text
[ Raw KUL Data: data/kul/ ]
           │
           ▼
   Step 1: Preprocess & Construct Memmap Cache
   Script: src/kul/data/build_kul_cache.py
   Output: data/processed_kul/S1_data.pt ... S16_data.pt
           │
           ▼
   Step 2: Train 16 Base Conformer Models (LOSO)
   Script: scripts/kul/train_conformer_loso.py
   Config: configs/kul_conformer.json
   Output: checkpoints/kul/model_S1.pt ... model_S16.pt
           │
           ▼
   Step 3: Verify Multi-Seed Benchmark Reproducibility
   Script: scripts/kul/eval_multiseed_repro.py
   Metrics: 77.12% ± 9.99% Grand Mean across 5 seeds (1, 7, 21, 42, 123)
           │
           ▼
   Step 4: Fine-Tune with Evidential Head & Outlier Exposure (OE)
   Script: scripts/kul/train_conformer_multitask_loso.py
   Output: checkpoints/kul/ (fine-tuned models with confidence head)
           │
           ▼
   Step 5: Evaluate Evidential Confidence Metrics
   Script: scripts/kul/eval_confidence_multitask.py
   Metrics: AUROC = 0.7337 | ECE = 0.0998 | Brier = 0.2115
           │
           ▼
   Step 6: Evaluate High-Threshold Selective AAD
   Script: scripts/kul/eval_selective_aad.py
   Metrics: 94.94% Selective Accuracy @ 12.32% Coverage (τ = 0.70)
           │
           ▼
   Step 7: Falsification Controls & Noise Robustness
   Scripts: scripts/kul/eval_falsification.py & eval_robustness.py
   Tests: Zero-EEG, Random Gaussian, Shuffled Labels, SNR sweep (-10 to +10 dB)
```

---

## 3. Cross-Dataset Transfer Map (KUL → DTU Zero-Shot)

```text
[ Locally Trained KUL Model: checkpoints/kul/model_S1.pt ]
                           │
                           ▼
   Step 1: Zero-Shot Transfer Evaluation on all 18 DTU Subjects
   Script: scripts/cross_dataset/eval_kul_to_dtu_transfer.py
   Config: configs/cross_dataset_kul_to_dtu.json
   Constraints: Zero DTU retraining, zero fine-tuning, zero parameter selection
                           │
                           ▼
   Reported Evaluation Metrics:
   • Accumulated Pearson Correlation: 68.24% trial accuracy
   • Trial Majority Vote: 54.26% trial accuracy
   • Window-Level Accuracy: 52.84% (5-s windows)
```
