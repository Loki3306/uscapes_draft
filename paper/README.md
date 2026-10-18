# Paper Reference: Decision-Gated Auditory Attention Decoding from Low-Density EEG

## Title
**Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control**

---

## Abstract

Auditory Attention Decoding (AAD) allows brain-computer interfaces and smart hearing instruments to infer which speaker a listener attends to in multi-talker environments. However, traditional AAD architectures suffer from two primary limitations: (1) reliance on high-density scalp EEG montages impractical for real-world hearing aids, and (2) open-loop, continuous classification that forces uncalibrated decisions during cognitive drift, speech pauses, or acoustic ambiguity, inducing catastrophic false-switch chatter.

In this work, we propose a **decision-gated AAD framework** operating on ultra-low-density 8-channel EEG montages. By coupling neural AAD decoders (ContrastiveMatchNet on DTU; AAD-Conformer on KUL) with confidence estimation engines (label-invariant XGBoost; evidential multi-task learning with Outlier Exposure), the system estimates the epistemic and aleatoric reliability of each inference step. 

On the DTU benchmark (18 subjects, 1,080 trials), the proposed framework achieves 69.02% leave-one-subject-out (LOSO) accuracy with ContrastiveMatchNet. Decision gating via a label-invariant XGBoost confidence model (AUROC = 0.8057) enables selective operating points yielding **81.55% accuracy at 70% coverage** and **86.00% accuracy at 50% coverage**. 

On the KUL benchmark (16 subjects, 320 trials), the AAD-Conformer attains a grand-mean LOSO accuracy of **77.12% ± 9.99%** across 5 random seeds. Evidential multi-task gating achieves **94.94% selective accuracy** at 12.32% coverage ($\tau = 0.70$) with AUROC = 0.7337 and Expected Calibration Error (ECE) = 0.0998. 

Furthermore, zero-shot cross-dataset evaluation of the frozen KUL-trained AAD-Conformer directly on DTU yields **68.24%** trial accuracy (via accumulated Pearson correlation) without retraining or fine-tuning, demonstrating neural representation generalizability across recording setups.

---

## Key Experimental Results Summary

| Dataset | Primary Model | Baseline LOSO Accuracy | Confidence Estimator | Gated Selective Accuracy | AUROC |
|---|---|---|---|---|---|
| **DTU** (18 Subjects) | ContrastiveMatchNet (8 ch) | **69.02%** | Label-Invariant XGBoost | **81.55%** (@ 70% coverage)<br>**86.00%** (@ 50% coverage) | **0.8057** |
| **KUL** (16 Subjects) | AAD-Conformer (8 ch) | **77.12% ± 9.99%** | Evidential Multi-Task (OE) | **94.94%** (@ 12.32% coverage) | **0.7337** |
| **KUL $\rightarrow$ DTU** (Zero-Shot) | Frozen AAD-Conformer | **68.24%** (acc-Pearson)<br>**54.26%** (majority vote) | Cross-dataset Transfer | N/A (Zero-Shot) | N/A |

---

## Citation

```bibtex
@article{decision_gated_aad_2026,
  title={Decision-Gated Auditory Attention Decoding from Low-Density EEG: Confidence-Conditioned Neural Inference for State-Aware Auditory Control},
  author={Research Authors},
  journal={IEEE Transactions on Neural Systems and Rehabilitation Engineering (Submitted)},
  year={2026}
}
```
