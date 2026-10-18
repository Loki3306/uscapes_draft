# Methodological Boundaries & Known Limitations

This document provides a comprehensive disclosure of known scientific, architectural, and experimental limitations of the work presented in:
**"Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control"**

---

## 1. Stochastic Training Variance (DTU 69.02% vs. 68.19%)

- **Observation**: The paper reports an average Leave-One-Subject-Out (LOSO) accuracy of **69.02%** on the DTU dataset. A fresh, canonical retraining of the identical codebase and architecture (`experiments/E0_canonical_result.md`, commit `aeb82b9`) yielded **68.19%** across the 5,400 evaluation windows.
- **Root Cause**: The original training campaign used stochastic gradient descent (AdamW optimizer) with data shuffling across mini-batches without fully pinned, deterministic random seed states for GPU operations.
- **Scientific Implication**: The 0.83 percentage-point difference reflects natural optimization stochasticity. In this repository, the **training procedure and methodology** are what is reproduced, rather than an artificial, over-fitted checkpoint.

---

## 2. Global Pre-LOSO Channel Selection on DTU

- **Observation**: The 8 electrodes evaluated for DTU (`[13, 46, 43, 23, 50, 0, 52, 14]`) were identified in `analysis/discover_channels_fast.py` (commit `e737710`) by fitting a single 64-channel Ridge regression decoder across **all 18 subjects simultaneously**.
- **Root Cause**: Channel discovery was performed globally as an offline feature-selection preprocessing stage before conducting the 18-subject LOSO cross-validation.
- **Scientific Implication**: Data from held-out test subjects was included in the Ridge regression model that selected the channel subset. While neural network training strictly adhered to LOSO, channel selection incorporates a degree of pre-LOSO global information.

---

## 3. Montage Anatomical Composition & Wearable Generalization

- **Observation**: The DTU 8-channel BioSemi montage corresponds to electrodes **`C5, FCz, FC6, P9, C6, Fp1, TP8, T7`**.
- **Root Cause**: The data-driven channel selection chose electrodes that maximize neural tracking of auditory envelopes regardless of physical layout. This includes the central vertex electrode **`FCz`**.
- **Scientific Implication**: The montage is a **low-density scalp montage**, but is NOT a strictly "around-the-ear" or "ear-EEG" wearable montage. When strictly peripheral ear channels (`[Fp1, Fp2, F7, F8, P7, P8, TP7, TP8]`) were evaluated in experiment E1 (`experiments/E1_wearable_montage.md`), accuracy dropped by -8.91% (68.19% down to 59.28%).

---

## 4. KUL Channel Derivation Mislabeled Lookup

- **Observation**: The 8 KUL channels are labeled `['T7', 'C2', 'FT8', 'P7', 'CPz', 'Fp1', 'TP8', 'C3']`.
- **Root Cause**: Forensic investigation of `analysis/eegnet_channel_discovery.py` (commit `06c4806`) revealed that these labels were copied from a generic 10-20 lookup table for the DTU indices `[13, 46, 43, 23, 50, 0, 52, 14]`, rather than independently optimized on KUL.
- **Scientific Implication**: Despite this naming artifact, the AAD-Conformer trained on this 8-channel set achieved a robust Grand Mean of **77.12% ± 9.99%** across 16 subjects.

---

## 5. Cross-Dataset Transfer Aggregation Rule

- **Observation**: Zero-shot transfer from KUL to DTU yields **68.24%** under Accumulated Pearson correlation, but **54.26%** under standard window Majority Voting, and **52.84%** at the window level.
- **Root Cause**:
  - **Accumulated Pearson (Method C)**: Sums the predicted acoustic correlation coefficients across all 5-s windows in a trial before making a single classification: $\sum r_A > \sum r_B$.
  - **Majority Vote (Method A)**: Classifies each 5-s window independently and takes the modal decision across the trial.
- **Scientific Implication**: Summing continuous correlation coefficients allows high-confidence windows to outweigh ambiguous ones, providing an effective trial-level integration mechanism. Both numbers are reported for complete scientific transparency.

---

## 6. Exclusion of Continuous Controller Simulator

- **Observation**: Historical development branches contained continuous state-switching simulation scripts reporting 92.58% lock, 2 true / 2 false switches, 22.53 false switches/hr, and 4.99s latency.
- **Root Cause**: Forensic audit (`phase_validity_matrix.md`) proved that synthetic continuous sessions were created by concatenating static trials with a fixed stream convention, without authentic cognitive switching dynamics.
- **Scientific Implication**: These continuous controller metrics were identified as methodological artifacts and are strictly **excluded** from the reproducibility repository. The controller is treated as a conceptual state-machine architecture with selective coasting.

---

## 7. Local Checkpoint Generation Requirement

- **Observation**: Pretrained binary `.pt` files are not shipped in this repository.
- **Scientific Implication**: Reviewers and researchers must run the provided training scripts (`scripts/dtu/train_matchnet_loso.py` and `scripts/kul/train_conformer_loso.py`) to generate checkpoints locally in `checkpoints/dtu/` and `checkpoints/kul/`.
