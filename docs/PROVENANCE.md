# Scientific Provenance & Source-Code Lineage

This document establishes the official provenance, source commit lineage, and dataset attribution for the code and benchmarks in this repository.

---

## 1. Source Commit Lineage

All implementations in `paper-reproduction/` were extracted from the research team's historical git repository (`EEG_Training_New`):

| Component | Source Git Reference | Original File Path | Description |
|---|---|---|---|
| **ContrastiveMatchNet** | Commit `3afe657e` (`release/aad-conformer-validated`) | `models/matchnet.py` | Primary DTU Siamese matching network |
| **EEGNet Backbone** | Commit `3afe657e` (`release/aad-conformer-validated`) | `models/eegnet.py` | Depthwise-separable EEG encoder |
| **Gammatone Extractor** | Commit `3afe657e` (`release/aad-conformer-validated`) | `data/extract_gammatone_envelopes.py` | 28-band cochlear Gammatone filterbank |
| **DTU Audio Mapping** | Commit `694e8131` | `scripts/verify_baseline/data/audio_mapping.json` | 1,080 trial-to-audio correspondences |
| **DTU Montages** | Commit `694e8131` | `scripts/verify_baseline/training/montages.py` | Authoritative BioSemi channel indices |
| **DTU LOSO Training** | Commit `3afe657e` (`release/aad-conformer-validated`) | `training/train_matchnet_loso.py` | MatchNet 18-subject cross-validation |
| **DTU Prediction Export** | Commit `3afe657e` (`release/aad-conformer-validated`) | `training/export_matchnet_predictions.py` | 5,400-window prediction exporter |
| **DTU Confidence Model** | Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/step_5_0a_train_final_model.py` | Canonical XGBoost on 5 label-invariant features |
| **DTU Selective Prediction** | Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/step_5_1_behavior_audit.py` | Selective accuracy & coverage evaluation |
| **DTU Confidence Ablations**| Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/step_5_2a_minimal_model_audit.py` | M1–M5 feature ablations (AUROC 0.66–0.81) |
| **Publication Figures** | Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/step_5_7_generate_publication_figures.py` | Generates Figures 1–5 |
| **AAD-Conformer** | Commit `3afe657e` (`release/aad-conformer-validated`) | `models/aad_conformer.py` | KUL Conformer with evidential head |
| **KUL Cache Preprocessing** | Commit `3afe657e` (`release/aad-conformer-validated`) | `preprocessing/build_kul_cache.py` | CAR, 1–8 Hz filter, 8-channel selection |
| **KUL Cached Loader** | Commit `3afe657e` (`release/aad-conformer-validated`) | `data/kul_cached_dataset.py` | Fast memmap dataset loader |
| **KUL LOSO Training** | Commit `3afe657e` (`release/aad-conformer-validated`) | `training/train_conformer_loso.py` | 16-subject Conformer cross-validation |
| **KUL Multitask Training** | Commit `3afe657e` (`release/aad-conformer-validated`) | `training/train_conformer_multitask_loso.py` | Outlier Exposure confidence training |
| **KUL Multi-Seed Summary** | Commit `3afe657e` (`release/aad-conformer-validated`) | `conformer_loso_results/conformer_loso_multiseed_summary.json` | Raw 5-seed 16-subject metrics |
| **KUL Multi-Seed Verifier** | Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/run1_reproducibility.py` | Grand Mean 77.12% evaluation script |
| **KUL Confidence Audit** | Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/run7_multitask_evaluation.py` | Evidential head AUROC & ECE evaluation |
| **KUL Selective AAD** | Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/run6_selective_aad.py` | Selective accuracy at tau = 0.70 |
| **KUL Falsification** | Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/run2_falsification.py` | Zero-EEG, permutation, random EEG controls |
| **KUL Robustness** | Commit `3afe657e` (`release/aad-conformer-validated`) | `analysis/run4_robustness.py` | SNR sweep & channel miniaturization |
| **Cross-Dataset Transfer** | Commit `1aaf5be` | `analysis/phase_10_4_independent_reproduction_audit.py` | Zero-shot KUL -> DTU evaluation |
| **Selective Metrics** | Commit `4da7cd26` | `src/confidence/selective_metrics.py` | Selective risk and coverage math |

---

## 2. Dataset Attribution

1. **DTU Auditory Attention Dataset**:
   - Reference: Fuglsang, S. A., Dau, T., & Hjortkjær, J. (2017). "Noise-robust cortical tracking of attended speech in normal-hearing and impaired listeners." *NeuroImage*, 156, 435–444.
   - Subjects: 18 normal-hearing native Danish speakers.
   - Setup: 64-channel BioSemi ActiveTwo system at 512 Hz. 60 trials of dual-talker stories per subject.

2. **KUL Auditory Attention Dataset**:
   - Reference: Das, N., Biesmans, W., Bertrand, A., & Francart, T. (2016 / 2020). "The effect of head-related filtering and reverberation on auditory attention decoding." *Journal of Neural Engineering*, 13(4), 046022.
   - Subjects: 16 normal-hearing native Flemish-Dutch speakers.
   - Setup: 64-channel BioSemi ActiveTwo system. 20 trials of dual-talker stories per subject (competing male and female speakers).

---

## 3. License & Academic Attribution

This software is released under the **MIT License** (see [LICENSE](../LICENSE)).
Academic use of this codebase should cite the manuscript:

```bibtex
@article{decision_gated_aad_2026,
  title={Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control},
  author={Research Authors},
  journal={IEEE Transactions on Neural Systems and Rehabilitation Engineering (Submitted)},
  year={2026}
}
```
