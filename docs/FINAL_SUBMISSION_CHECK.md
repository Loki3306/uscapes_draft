# Final Submission Check & Scientific Coverage Audit

**Evaluation Target:** `uscapes_draft` (extracted from `paper-reproduction/`)  
**Associated Manuscript:** *"Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control"*  
**Date of Audit:** October 2026  
**Auditor:** Automated Forensic Engineering Agent (DeepMind Antigravity)  

---

## Executive Verdict

### **READY WITH DOCUMENTED LIMITATIONS**

The repository contains every implementation, preprocessing routine, model architecture, training loop, evaluation metric, and visualization tool required to understand and reproduce the experiments presented in the paper. Pre-trained binary checkpoints have been excluded per academic submission standards, and training scripts generate subject-specific models locally. All personal developer paths and exploratory artifacts have been purged.

---

## Part 1: Paper $\rightarrow$ Code Coverage Audit Table

| Category | Paper Component | Required Code / Implementation | Present? | Correct Source Identified? | Reproduction Instructions Exist? | Status |
|---|---|---|:---:|:---:|:---:|:---:|
| **A. Dataset Handling** | DTU Dataset Loading | `src/common/_common.py`, `src/common/ridge_aad.py` | Yes | `_common.py` (commit `694e8131`) | Yes ([docs/DATASET_SETUP.md](DATASET_SETUP.md)) | **COVERED** |
| | KUL Dataset Loading | `src/kul/data/kul_cached_dataset.py`, `build_kul_cache.py` | Yes | `preprocessing/build_kul_cache.py` (`3afe657e`) | Yes ([docs/DATASET_SETUP.md](DATASET_SETUP.md)) | **COVERED** |
| | Audio Loading & Mapping | `src/dtu/data/audio_mapping.json`, `extract_gammatone_envelopes.py` | Yes | `scripts/verify_baseline/data/` (`694e8131`) | Yes ([docs/DATASET_SETUP.md](DATASET_SETUP.md)) | **COVERED** |
| | EEG Trial Segmentation | `prepare_dataset()` in DTU (`5s`) and KUL (`2s` / `5s`) | Yes | `training/train_matchnet_loso.py` | Yes ([docs/REPRODUCTION_MAP.md](REPRODUCTION_MAP.md)) | **COVERED** |
| | Dataset Preprocessing | CAR across 64 ch, 1–8 Hz bandpass filtering, 8 ch subsampling | Yes | `src/kul/data/build_kul_cache.py`, `src/common/_common.py` | Yes ([docs/DATASET_SETUP.md](DATASET_SETUP.md)) | **COVERED** |
| **B. EEG Preprocessing**| Sampling Rate Conversion | Resampling to 64 Hz (`FS = 64`) | Yes | `src/kul/data/build_kul_cache.py`, `src/common/_common.py` | Yes | **COVERED** |
| | Bandpass Filtering | 1–8 Hz 4th-order Butterworth forward-backward `filtfilt` | Yes | `src/common/ridge_aad.py`, `src/kul/data/build_kul_cache.py` | Yes | **COVERED** |
| | Normalization | Z-score normalization per channel / window | Yes | `src/common/utils.py` (`normalize_eeg`, `normalize_audio`) | Yes | **COVERED** |
| | Channel Configuration | DTU: 8 channels (`C5, FCz, FC6, P9, C6, Fp1, TP8, T7`)<br>KUL: 8 channels (`T7, C2, FT8, P7, CPz, Fp1, TP8, C3`) | Yes | `src/dtu/montages.py`, `src/kul/data/build_kul_cache.py` | Yes ([docs/LIMITATIONS.md](LIMITATIONS.md)) | **COVERED** |
| **C. Audio Preprocessing**| Gammatone Representation | 28 ERB frequency bands (50 Hz to 8,000 Hz) | Yes | `src/dtu/data/extract_gammatone_envelopes.py` | Yes | **COVERED** |
| | Envelope & Compression | Analytic Hilbert envelope with $e(t)^{0.6}$ power compression | Yes | `src/dtu/data/extract_gammatone_envelopes.py` | Yes | **COVERED** |
| | Resampling & Alignment | Lowpass filtering and resampling to 64 Hz matching EEG | Yes | `src/dtu/data/extract_gammatone_envelopes.py` | Yes | **COVERED** |
| **D. Windowing** | Training Windows | 5-s non-overlapping (DTU); 2-s with 1-s hop (KUL) | Yes | `scripts/dtu/train_matchnet_loso.py`, `scripts/kul/train_conformer_loso.py` | Yes | **COVERED** |
| | Evaluation Windows | 5-s non-overlapping windows (DTU & KUL selective / transfer) | Yes | `scripts/dtu/export_matchnet_predictions.py`, `scripts/cross_dataset/eval_kul_to_dtu_transfer.py` | Yes | **COVERED** |
| **E. DTU Model** | ContrastiveMatchNet | Siamese architecture coupling EEGNet with 1D Audio ConvNet | Yes | `src/dtu/models/matchnet.py` (`3afe657e`) | Yes | **COVERED** |
| | EEGNet Encoder | Depthwise separable CNN (4 temp, 2 spatial/temp, pointwise) | Yes | `src/dtu/models/eegnet.py` | Yes | **COVERED** |
| | Model Configuration | 50,928 parameters, contrastive margin $m=0.2$ | Yes | `configs/dtu_matchnet.json` | Yes | **COVERED** |
| | Training Procedure | 18-fold LOSO cross-validation, 30 epochs, batch size 64 | Yes | `scripts/dtu/train_matchnet_loso.py` | Yes ([docs/REPRODUCTION_MAP.md](REPRODUCTION_MAP.md)) | **COVERED** |
| | Attention Prediction | Cosine similarity ranking ($\text{sim}(e, a_{att}) > \text{sim}(e, a_{unatt})$) | Yes | `scripts/dtu/export_matchnet_predictions.py` | Yes | **COVERED** |
| **F. KUL Model** | AAD-Conformer | Spatial-temporal front-end + 2 Conformer blocks ($d=64$, 4 heads) | Yes | `src/kul/models/aad_conformer.py` (`3afe657e`) | Yes | **COVERED** |
| | Parameterization | 162,849 base parameters; 165,090 with evidential head | Yes | `configs/kul_conformer.json` | Yes | **COVERED** |
| | Training Procedure | 16-fold LOSO cross-validation, 40 epochs, AdamW ($10^{-3}$) | Yes | `scripts/kul/train_conformer_loso.py` | Yes ([docs/REPRODUCTION_MAP.md](REPRODUCTION_MAP.md)) | **COVERED** |
| | Multi-Seed Benchmark | 5 seeds (1, 7, 21, 42, 123) reproducing 77.12% ± 9.99% grand mean | Yes | `scripts/kul/eval_multiseed_repro.py`, `results/kul_conformer_multiseed_summary.json` | Yes ([docs/RESULT_TRACEABILITY.md](RESULT_TRACEABILITY.md)) | **COVERED** |
| **G. Confidence Gating**| DTU Confidence Features | 5 label-invariant features (`margin`, `sim_chosen`, `sim_unchosen`, `rolling_std_margin`, `trial_consistency`) | Yes | `src/dtu/confidence/inference_engine.py`, `scripts/dtu/train_confidence_model.py` | Yes | **COVERED** |
| | DTU XGBoost Model | Nested LOSO cross-validation, AUROC = 0.8057 | Yes | `scripts/dtu/train_confidence_model.py`, `eval_confidence_ablations.py` | Yes | **COVERED** |
| | DTU Selective Gating | 81.55% @ 70% coverage ($\tau = 0.65$); 86.00% @ 50% coverage | Yes | `scripts/dtu/eval_selective_prediction.py` | Yes | **COVERED** |
| | KUL Evidential Head | Multitask Conformer with Outlier Exposure (OE) | Yes | `scripts/kul/train_conformer_multitask_loso.py` | Yes | **COVERED** |
| | KUL Calibration Metrics | AUROC = 0.7337, ECE = 0.0998, Brier = 0.2115 | Yes | `scripts/kul/eval_confidence_multitask.py` | Yes | **COVERED** |
| | KUL Selective AAD | 94.94% selective accuracy at $\tau = 0.70$ (coverage = 12.32%) | Yes | `scripts/kul/eval_selective_aad.py` | Yes | **COVERED** |
| **H. Controls / Falsification** | Zero-EEG Control | EEG set to zeros; accuracy drops to ~50% chance, confidence $\rightarrow$ 0 | Yes | `scripts/kul/eval_falsification.py`, `scripts/dtu/train_matchnet_loso.py` | Yes | **COVERED** |
| | Permutation Control | Trial-level acoustic/EEG stream permutation; chance performance | Yes | `scripts/kul/eval_falsification.py` | Yes | **COVERED** |
| | Random Noise Control | Gaussian noise EEG; confidence collapses to near 0 | Yes | `scripts/kul/eval_falsification.py` | Yes | **COVERED** |
| | Noise & Channel Sweeps | AWGN sweep (+10 to -10 dB) & channel miniaturization (8→5→3→1) | Yes | `scripts/kul/eval_robustness.py` | Yes | **COVERED** |
| **I. Cross-Dataset Transfer** | Zero-Shot KUL $\rightarrow$ DTU | Frozen KUL `model_S1.pt` evaluated on 18 DTU subjects (1,080 trials) | Yes | `scripts/cross_dataset/eval_kul_to_dtu_transfer.py` | Yes | **COVERED** |
| | Transfer Metrics | Accumulated Pearson: 68.24% | Majority Vote: 54.26% | Window: 52.84% | Yes | `configs/cross_dataset_kul_to_dtu.json` | Yes | **COVERED** |
| **J. Publication Figures** | Figures 1–5 | Calibration diagrams, ROC curves, risk-coverage curves, transfer comparisons | Yes | `figures/generate_publication_figures.py` | Yes ([README.md](../README.md)) | **COVERED** |

---

## Part 2: Answers to the 10 Mandatory Questions

### 1. Does the repository contain every implementation component required by the paper?
**YES.** Every component—from raw Gammatone envelope extraction and CAR filtering, to Siamese and Conformer architectures, evidential heads, XGBoost confidence models, and selective prediction sweeps—is fully implemented and present.

### 2. Can a technically competent researcher identify the script corresponding to every paper experiment?
**YES.** The repository provides three dedicated mapping documents:
- [docs/REPRODUCTION_MAP.md](REPRODUCTION_MAP.md): Step-by-step sequential pipelines for DTU, KUL, and Cross-Dataset tracks.
- [docs/EXPERIMENT_MAP.md](EXPERIMENT_MAP.md): Direct mapping from paper metrics to script execution lines.
- [docs/RESULT_TRACEABILITY.md](RESULT_TRACEABILITY.md): Granular traceability table covering every number, required input, and regeneration status.

### 3. Are all reported numerical results traceable to code/artifacts?
**YES.** Every empirical claim has been verified against historical git commits and logged execution outputs. Discrepancies (specifically DTU reported 69.02% vs. 68.19% canonical retraining) are transparently disclosed rather than hidden.

### 4. Are missing checkpoints handled correctly by documented training instructions?
**YES.** Pre-trained binary `.pt` files were intentionally purged. Empty target folders (`checkpoints/dtu/.gitkeep`, `checkpoints/kul/.gitkeep`) and explicit training instructions (`train_matchnet_loso.py`, `train_conformer_loso.py`, `train_conformer_multitask_loso.py`) instruct the researcher how to generate local models before evaluating downstream selective prediction and cross-dataset scripts.

### 5. Are all post-paper experiments excluded?
**YES.** The following non-paper exploratory or invalidated components are strictly absent:
- Continuous controller simulator (invalid static trial concatenation)
- Early leaked confidence models (using raw `sim_A`/`sim_B`)
- Post-paper architectures: CA-TCN, MSCA-AAD, SincAlignNet, USCAPES edge deployment, STM32 firmware, and live web UIs.

### 6. Are all known methodological limitations documented?
**YES.** Documented in [docs/LIMITATIONS.md](LIMITATIONS.md):
- Stochastic optimization jitter between fresh training (68.19%) and reported baseline (69.02%).
- Global pre-LOSO DTU channel selection leakage on all 18 subjects via `analysis/discover_channels_fast.py` (commit `e737710`).
- Presence of vertex electrode `FCz` in the DTU 8-channel montage.
- Historical KUL channel label lookup mapping.

### 7. Are there any hidden dependencies on the original developer machine?
**NO.** All 42 Python source files, JSON configs, and documentation files contain zero references to personal developer paths (`<DEVELOPER_HOME>/...`) or Kaggle user paths (`<REMOTE_DATASETS>/...`). All paths are constructed relative to `REPO_ROOT` or supplied via standard CLI flags (`--data_dir`, `--cache_dir`, `--audio_dir`, `--checkpoint_dir`).

### 8. Are there any unnecessary files remaining?
**NO.** All compatibility shims (`models/`, `baselines/`, `training/`, `analysis/`, and shim files in `data/`) have been removed. Python imports now use standard modular package imports (`from src.dtu.models.matchnet import ...`).

### 9. Is licensing/provenance sufficiently documented?
**YES.** [docs/PROVENANCE.md](PROVENANCE.md) details the exact source commit hashes (`3afe657e`, `694e8131`, `1aaf5be`, `4da7cd26`), original file locations, dataset citations (Fuglsang et al. 2017, Das et al. 2016/2020), and MIT licensing terms.

### 10. Is the repository suitable to give to a professor as the paper's accompanying code?
**YES.** The repository is cleanly structured, methodologically honest, reproducible, and free of personal artifacts.

---

## Verdict: READY WITH DOCUMENTED LIMITATIONS
Proceed with preparing the final GitHub repository `uscapes_draft`.
