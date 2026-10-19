# Final Paper Ground-Truth Results Check

**Paper Title:** *Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control*  
**Authoritative Sources:** Final Paper Specification, Abstract, Manuscript Tables, and Forensic Foundation (`paper/README.md`, `docs/DTU_KUL_AAD_COMPLETE_METHODOLOGY.md`, `docs/PAPER_FOUNDATION/PAPER_FOUNDATION_V2.md`).

---

## Authoritative Ground-Truth Paper Results Table

| Paper Section | Experiment | Metric | Exact Paper Value | Paper Source / Location |
| :--- | :--- | :--- | ---: | :--- |
| **Section III.A** | DTU ContrastiveMatchNet LOSO | Baseline 10s Window Accuracy | **69.02%** | Abstract, Table 1, `PAPER_FOUNDATION_V2.md` Table 7 |
| **Section III.A** | DTU Canonical Historical Retraining | Baseline 10s Window Accuracy | **68.19%** | `experiments/E0_canonical_result.md`, `RESULT_TRACEABILITY.md` |
| **Section III.B** | DTU Temporal Window Scaling | 2-second Window Accuracy | **58.2%** | Section III.B, Figure 2 |
| **Section III.B** | DTU Temporal Window Scaling | 5-second Window Accuracy | **63.4%** | Section III.B, Figure 2 |
| **Section III.B** | DTU Temporal Window Scaling | 10-second Window Accuracy | **69.02%** | Section III.B, Figure 2 |
| **Section III.B** | DTU Temporal Window Scaling | 20-second Window Accuracy | **72.7%** | Section III.B, Figure 2 |
| **Section III.B** | DTU Temporal Window Scaling | 30-second Window Accuracy | **76.9%** | Section III.B, Figure 2 |
| **Section III.C** | DTU Sanity Negative Control | Zero EEG Accuracy | **~50.0%** | Section III.C, Table 1 |
| **Section III.C** | DTU Sanity Negative Control | Shuffled Labels / EEG Accuracy | **~50.0%** | Section III.C, Table 1 |
| **Section III.D** | DTU Confidence Model (Full 5-feature XGBoost) | Nested LOSO AUROC | **0.8057** | Abstract, Table 2, `PAPER_FOUNDATION_V2.md` Section 8.7 |
| **Section III.D** | DTU Confidence Ablation $M_1$ | Margin-Only AUROC | **0.6601** | Table 2, `PAPER_FOUNDATION_V2.md` Section 8.7 |
| **Section III.D** | DTU Confidence Ablation $M_2$ | Consistency-Only AUROC | **0.5840** | Table 2, `RESULT_TRACEABILITY.md` |
| **Section III.D** | DTU Confidence Ablation $M_3$ | Margin + Consistency AUROC | **0.7214** | Table 2, `FINAL_FORENSIC_AUDIT.md` Line 73 |
| **Section III.E** | DTU Decision-Gated Selective Prediction | Accuracy @ 70% Coverage ($\tau \approx 0.65$) | **81.55%** | Abstract, Table 3, `PAPER_FOUNDATION_V2.md` Section 8.7 |
| **Section III.E** | DTU Decision-Gated Selective Prediction | Accuracy @ 50% Coverage ($\tau \approx 0.78$) | **86.00%** | Abstract, Table 3, `PAPER_FOUNDATION_V2.md` Section 24.2 |
| **Section IV.A** | KUL AAD-Conformer Primary Cohort | Grand Mean LOSO Accuracy (5 Seeds, 16 Subs) | **77.12% ± 9.99%** | Abstract, Table 4, `PAPER_README.md` |
| **Section IV.A** | KUL AAD-Conformer Seed 1 | Seed 1 Mean LOSO Accuracy | **71.88%** (71.9%) | Section IV.A, Table 4 |
| **Section IV.A** | KUL AAD-Conformer Seed 7 | Seed 7 Mean LOSO Accuracy | **79.38%** | Table 4, `FINAL_FORENSIC_AUDIT.md` Line 59 |
| **Section IV.A** | KUL AAD-Conformer Seed 21 | Seed 21 Mean LOSO Accuracy | **78.12%** | Table 4, `FINAL_FORENSIC_AUDIT.md` Line 59 |
| **Section IV.A** | KUL AAD-Conformer Seed 42 | Seed 42 Mean LOSO Accuracy | **75.62%** | Table 4, `FINAL_FORENSIC_AUDIT.md` Line 59 |
| **Section IV.A** | KUL AAD-Conformer Seed 123 | Seed 123 Mean LOSO Accuracy | **80.62%** | Table 4, `FINAL_FORENSIC_AUDIT.md` Line 59 |
| **Section IV.C** | KUL Evidential Multi-Task Confidence | Test AUROC | **0.7337** | Abstract, Table 5, `FINAL_AAD_CONFORMER_REPORT.md` Section 3.4 |
| **Section IV.C** | KUL Evidential Multi-Task Calibration | Expected Calibration Error (ECE) | **0.0998** | Abstract, Table 5, `FINAL_AAD_CONFORMER_REPORT.md` Section 3.4 |
| **Section IV.C** | KUL Evidential Multi-Task Calibration | Brier Score | **0.2115** | Table 5, `RESULT_TRACEABILITY.md` Line 35 |
| **Section IV.D** | KUL Evidential Selective Prediction | Selective Accuracy @ $\tau = 0.70$ | **94.94%** (Coverage: 12.32%) | Abstract, Table 5, `FINAL_AAD_CONFORMER_REPORT.md` Section 3.4 |
| **Section IV.D** | KUL Evidential Selective Prediction | Selective Accuracy @ $\tau = 0.60$ | **82.60%** (Coverage: 34.30%) | `FINAL_AAD_CONFORMER_REPORT.md` Section 3.4 |
| **Section IV.E** | KUL Corrupted Input Degradation | Mean Confidence under Clean EEG | **~0.54 – 0.56** | Section IV.E, `FINAL_AAD_CONFORMER_REPORT.md` Section 3.4 |
| **Section IV.E** | KUL Corrupted Input Degradation | Mean Confidence under Zero / Noise EEG | **~0.13 (Near 0)** | Section IV.E, `FINAL_AAD_CONFORMER_REPORT.md` Section 3.4 |
| **Section V** | Cross-Dataset Transfer (KUL $\to$ DTU) | Trial Accuracy via Accumulated Pearson | **68.24%** | Abstract, Table 6, `FINAL_FORENSIC_AUDIT.md` Section 6 |
| **Section V** | Cross-Dataset Transfer (KUL $\to$ DTU) | Trial Accuracy via Standard Majority Vote | **54.26%** | Abstract, Table 6, `FINAL_FORENSIC_AUDIT.md` Section 6 |
| **Section V** | Cross-Dataset Transfer (KUL $\to$ DTU) | Window-Level Accuracy (5s windows) | **52.84%** | Table 6, `RESULT_TRACEABILITY.md` Line 43 |
