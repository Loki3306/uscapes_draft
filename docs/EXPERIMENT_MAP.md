# Master Experiment Map: Paper Claims Traceability

This document provides a 1-to-1 mapping connecting every empirical result reported in the paper:
**"Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control"**
to its exact source implementation, configuration, checkpoint, and evaluation script.

---

## Master Traceability Matrix

| Paper Result / Metric | Dataset | Model Architecture | Training / Extraction Script | Configuration File | Model Checkpoint | Evaluation Script | Expected Metric Value |
|---|---|---|---|---|---|---|---|
| **DTU LOSO Baseline Accuracy** | DTU (18 subjects, 1,080 trials) | ContrastiveMatchNet (EEGNet encoder, 8 ch) | `scripts/dtu/train_matchnet_loso.py` | `configs/dtu_matchnet.json` | Trained during LOSO run | `scripts/dtu/export_matchnet_predictions.py` | **69.02%** LOSO accuracy (canonical repro: 68.19% across 5,400 windows) |
| **DTU Confidence AUROC (Full Model)** | DTU | XGBoost on 5 label-invariant features | `scripts/dtu/train_confidence_model.py` | `configs/dtu_matchnet.json` | Generated `confidence_model.json` | `scripts/dtu/eval_confidence_ablations.py` | **0.8057** nested LOSO AUROC |
| **DTU Confidence AUROC (Margin-Only M1)** | DTU | XGBoost on `margin` feature only | `scripts/dtu/eval_confidence_ablations.py` | `configs/dtu_matchnet.json` | N/A (Nested cross-validation) | `scripts/dtu/eval_confidence_ablations.py` | **0.6601** nested LOSO AUROC |
| **DTU Confidence AUROC (Consistency M2)** | DTU | XGBoost on `trial_consistency` only | `scripts/dtu/eval_confidence_ablations.py` | `configs/dtu_matchnet.json` | N/A (Nested cross-validation) | `scripts/dtu/eval_confidence_ablations.py` | **0.5840** nested LOSO AUROC |
| **DTU Selective Accuracy @ 70% Coverage** | DTU | ContrastiveMatchNet + XGBoost Confidence | `scripts/dtu/eval_selective_prediction.py` | `configs/dtu_matchnet.json` | Generated `confidence_model.json` | `scripts/dtu/eval_selective_prediction.py` | **81.55%** selective accuracy (Coverage: 70.0%, $\tau = 0.65$) |
| **DTU Selective Accuracy @ 50% Coverage** | DTU | ContrastiveMatchNet + XGBoost Confidence | `scripts/dtu/eval_selective_prediction.py` | `configs/dtu_matchnet.json` | Generated `confidence_model.json` | `scripts/dtu/eval_selective_prediction.py` | **86.00%** selective accuracy (Coverage: 50.0%) |
| **DTU Sanity Control: Zero EEG** | DTU | ContrastiveMatchNet | `scripts/dtu/train_matchnet_loso.py --zero-eeg` | `configs/dtu_matchnet.json` | N/A | `scripts/dtu/train_matchnet_loso.py --zero-eeg` | **~50.0%** chance performance |
| **DTU Sanity Control: Shuffled Labels** | DTU | ContrastiveMatchNet | `scripts/dtu/train_matchnet_loso.py --shuffle-labels` | `configs/dtu_matchnet.json` | N/A | `scripts/dtu/train_matchnet_loso.py --shuffle-labels` | **~50.0%** chance performance |
| **KUL Primary Cohort Accuracy** | KUL (16 subjects, 320 trials) | AAD-Conformer (8 ch, 2 layers, 4 heads) | `scripts/kul/train_conformer_loso.py` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/`) | `scripts/kul/eval_multiseed_repro.py` | **77.12% ± 9.99%** (Grand Mean across 5 seeds: 1, 7, 21, 42, 123) |
| **KUL Seed 1 Mean Accuracy** | KUL | AAD-Conformer (8 ch) | `scripts/kul/train_conformer_loso.py --seed 1` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/model_S1.pt` - `S16.pt`) | `scripts/kul/eval_multiseed_repro.py` | **71.88%** (reported as 71.9%) |
| **KUL Evidential Confidence AUROC** | KUL | AAD-Conformer + Evidential Head (OE) | `scripts/kul/train_conformer_multitask_loso.py` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/`) | `scripts/kul/eval_confidence_multitask.py` | **0.7337** AUROC |
| **KUL Expected Calibration Error (ECE)** | KUL | AAD-Conformer + Evidential Head | `scripts/kul/train_conformer_multitask_loso.py` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/`) | `scripts/kul/eval_confidence_multitask.py` | **0.0998** ECE (Brier score: 0.2115) |
| **KUL Selective Accuracy @ High Threshold** | KUL | AAD-Conformer + Evidential Head | `scripts/kul/eval_selective_aad.py` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/`) | `scripts/kul/eval_selective_aad.py` | **94.94%** selective accuracy (Coverage: 12.32%, $\tau = 0.70$) |
| **KUL Confidence Degradation: Clean vs. Zero EEG** | KUL | AAD-Conformer + Evidential Head | `scripts/kul/eval_falsification.py` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/`) | `scripts/kul/eval_confidence_multitask.py` | Confidence collapses to near 0 under Zero EEG |
| **KUL Confidence Degradation: Random EEG** | KUL | AAD-Conformer + Evidential Head | `scripts/kul/eval_falsification.py` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/`) | `scripts/kul/eval_confidence_multitask.py` | Confidence collapses under Gaussian noise EEG |
| **KUL AWGN Robustness Sweep** | KUL | AAD-Conformer | `scripts/kul/eval_robustness.py` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/`) | `scripts/kul/eval_robustness.py` | Monotonic accuracy drop from +10 dB to -10 dB SNR |
| **KUL Channel Miniaturization (8→5→3→1)** | KUL | AAD-Conformer | `scripts/kul/eval_robustness.py` | `configs/kul_conformer.json` | Generated locally (`checkpoints/kul/`) | `scripts/kul/eval_robustness.py` | Graceful degradation across channel subsets |
| **Zero-Shot Cross-Dataset Transfer (Accumulated Pearson)** | DTU (18 subjects, 1,080 trials) | Frozen KUL AAD-Conformer | None (Zero-Shot) | `configs/cross_dataset_kul_to_dtu.json` | Generated locally (`checkpoints/kul/model_S1.pt`) | `scripts/cross_dataset/eval_kul_to_dtu_transfer.py` | **68.24%** trial accuracy (Accumulated Pearson across trial windows) |
| **Zero-Shot Cross-Dataset Transfer (Majority Vote)** | DTU | Frozen KUL AAD-Conformer | None (Zero-Shot) | `configs/cross_dataset_kul_to_dtu.json` | Generated locally (`checkpoints/kul/model_S1.pt`) | `scripts/cross_dataset/eval_kul_to_dtu_transfer.py` | **54.26%** trial accuracy (Majority vote across windows) |
| **Zero-Shot Cross-Dataset Transfer (Window Level)** | DTU | Frozen KUL AAD-Conformer | None (Zero-Shot) | `configs/cross_dataset_kul_to_dtu.json` | Generated locally (`checkpoints/kul/model_S1.pt`) | `scripts/cross_dataset/eval_kul_to_dtu_transfer.py` | **52.84%** 5-s window accuracy |

---

## Detailed Notes on Discrepancies & Excluded Results

1. **Continuous Controller Simulator Numbers (EXCLUDED)**:
   - Spliced session metrics (e.g. 92.58% lock, 2 true / 2 false switches, 22.53 false switches/hr, 4.99s latency) reported in historical exploratory work were determined to be an artifact of static trial concatenation where attended stream conventions did not simulate genuine physiological attention switching.
   - Per forensic audit, these continuous switching metrics are **excluded** from the main results pipeline.

2. **Old ContrastiveMatchNet KUL Result (~75.8%) (EXCLUDED)**:
   - An earlier exploratory run of ContrastiveMatchNet on KUL achieved ~75.8% under ad-hoc windowing. The primary, cohort-level KUL result reported in the paper is the AAD-Conformer at **77.12% ± 9.99%** across 16 subjects and 5 seeds.

3. **Early Leaked Confidence Model (~0.99 AUROC) (EXCLUDED)**:
   - Historical code contained a label leakage issue where `sim_A` and `sim_B` directly encoded target stream identity.
   - The paper's canonical result is the **leakage-safe XGBoost model** utilizing 5 label-invariant features (`margin`, `sim_chosen`, `sim_unchosen`, `rolling_std_margin`, `trial_consistency`), yielding **AUROC = 0.8057**.
