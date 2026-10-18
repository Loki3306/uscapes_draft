# Scientific Provenance, Implementation Notes & Exclusions

This document preserves the scientific audit trail for the paper:
**"Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control"**

It explicitly records design decisions, discoveries regarding channel montages, eliminated methodological artifacts, and excluded historical code paths to maintain absolute scientific honesty.

---

## 1. Provenance and Decision Log

| Issue / Finding | Forensic Observation | Decision Made | Impact on Reproduction |
|---|---|---|---|
| **Continuous Controller Simulation Metrics** | Historical scripts reported 92.58% lock coverage, 2 true / 2 false switches, 22.53 false switches/hr, and 4.99s acquisition latency on synthetic continuous sessions. Forensic audit (`phase_validity_matrix.md`) proved these sessions were constructed by concatenating static trials with a fixed stream convention, lacking physiological attention-switching dynamics. | **Strictly excluded** from the results pipeline. The controller is documented as a conceptual architecture with selective coasting. | Prevents reporting unvalidated continuous-switching metrics as empirical paper claims. |
| **Early Confidence Label Leakage (~0.99 AUROC)** | An early confidence prototype directly incorporated raw stream similarity vectors (`sim_A`, `sim_B`) sorted by ground truth, leaking the attended-stream target identity and inflating AUROC to ~0.99. | **Excluded**. Only the final, leakage-safe XGBoost model (`scripts/dtu/train_confidence_model.py`) using 5 label-invariant features is included. | Reproduces true, uncorrupted paper AUROC of **0.8057** (and margin-only AUROC of **0.6601**). |
| **DTU 8-Channel Montage Provenance** | The 8 DTU channels `[13, 46, 43, 23, 50, 0, 52, 14]` were selected via global Ridge regression decoder weights on all 18 subjects (`analysis/discover_channels_fast.py`, commit `e0642ac`). The BioSemi electrode labels correspond to **`C5, FCz, FC6, P9, C6, Fp1, TP8, T7`**. | Documented as the authoritative montage. | Confirms the presence of vertex electrode `FCz`. When strictly peripheral ear channels were evaluated in experiment E1, accuracy collapsed by -8.91% (68.19% down to 59.28%). |
| **KUL 8-Channel Montage Derivation** | KUL 8-channel selection was defined as `['T7', 'C2', 'FT8', 'P7', 'CPz', 'Fp1', 'TP8', 'C3']` in `analysis/eegnet_channel_discovery.py`. | Preserved as exact implementation in `src/kul/data/build_kul_cache.py`. | Ensures exact reproduction of KUL 77.12% LOSO accuracy. |
| **Old KUL ContrastiveMatchNet Result (~75.8%)** | Early exploratory work ran ContrastiveMatchNet on KUL and obtained ~75.8% under non-standard window splits. | **Excluded** from primary KUL results. The authoritative KUL model in the paper is `AAD-Conformer` (**77.12% ± 9.99%**). | Eliminates confusion between competing model architectures for the KUL cohort. |
| **Cross-Dataset Directionality (KUL $\rightarrow$ DTU vs. DTU $\rightarrow$ KUL)** | KUL $\rightarrow$ DTU zero-shot transfer was evaluated across all 18 DTU subjects (1,080 trials) yielding 68.24% (accumulated Pearson). Reverse DTU $\rightarrow$ KUL was not evaluated across the full cohort. | Only include the verified KUL $\rightarrow$ DTU transfer script (`scripts/cross_dataset/eval_kul_to_dtu_transfer.py`). | Prevents fabricated or partial reverse-transfer claims. |
| **Cross-Dataset Evaluation Protocol Discrepancy** | On DTU, evaluating the frozen KUL Conformer via Accumulated Pearson correlation across trial windows gives **68.24%**, while standard window majority voting gives **54.26%**, and raw 5-s window accuracy is **52.84%**. | Both Accumulated Pearson (68.24%) and Majority Vote (54.26%) are explicitly documented in `configs/cross_dataset_kul_to_dtu.json` and the evaluation script. | Completely transparent accounting of cross-dataset aggregation effects. |
| **KUL Checkpoint Loading Strictness** | Loading `model_S1.pt` directly into `AADConformer` raises missing keys for `confidence_head` because the base checkpoint contains only the backbone weights, while the confidence head was added during multitask training. | Loaded with `strict=False` in multitask scripts, which exactly matches historical training scripts. | 100% parameter match across all 164 backbone layers. |

---

## 2. Excluded Post-Paper Developments

The following modules were developed after the paper freeze and have been deliberately excluded:
- **CA-TCN / MSCA-AAD**: Multi-Scale Cross-Attention Temporal Convolutional Networks.
- **SincAlignNet**: Parametric sinc-convolution frontends.
- **USCAPES**: Edge hardware deployment and wearable headband firmware.
- **STM32 Firmware**: Embedded C implementations and quantized integer inference.
- **Real-Time Web UIs**: Live audio amplification/suppression demos (`src/ui`, `USCAPES/`).
