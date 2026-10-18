# Forensic Checkpoint Audit

This document presents the detailed architectural and forensic audit of all 16 historical model checkpoints extracted from `conformer_checkpoints_seed1.zip` (`46145461`). This audit proves why shipping binary checkpoints was incomplete (they lacked the evidential confidence head) and why generating models locally via training scripts is methodologically required.

---

## 1. Master Checkpoint Audit Table

| Checkpoint File | Source Git Archive / Commit | Expected Architecture | Parameterization | Confidence Head Present? | State Dict Tensor Keys | Primary Purpose | Verified? |
|---|---|---|---|---|---|---|---|
| `model_S1.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S1 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S2.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S2 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S3.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S3 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S4.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S4 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S5.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S5 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S6.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S6 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S7.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S7 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S8.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S8 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S9.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S9 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S10.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S10 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S11.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S11 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S12.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S12 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S13.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S13 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S14.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S14 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S15.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S15 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |
| `model_S16.pt` | `conformer_checkpoints_seed1.zip` (`46145461`) | `AADConformer` | 8 ch, 32 temp, 64 spat, 64 emb, 4 heads, 2 layers | **NO** (85 keys) | 85 backbone keys | Base AAD decoding for S16 (Seed 1) / Pretrained initialization for multitask fine-tuning | **YES** (162,849 params match) |

---

## 2. Critical Findings on Checkpoint Provenance

1. **Checkpoints Correspond Strictly to Base Decoding Models**:
   - Every checkpoint file contains exactly **85 state_dict tensor keys** corresponding to **162,849 trainable parameters**.
   - These parameters exactly match the base `AADConformer` architecture:
     - `temporal_conv`: `(32, 1, 1, 33)`
     - `spatial_conv`: `(64, 1, 8, 1)`
     - `conformer_blocks.0` and `conformer_blocks.1`: Multi-Head Attention, ConvModule, FeedForward
     - `upsample` & `head`: Projection layers
   - Zero confidence head parameters exist in these files:
     - Missing: `confidence_head.0.weight`, `confidence_head.0.bias`, `confidence_head.3.weight`, `confidence_head.3.bias` (4 tensor keys, 2,241 parameters).

2. **Decoupling of Decoding and Confidence Checkpoints**:
   - **Base AAD Decoding Evaluation**: The included checkpoints are 100% verified to reproduce Seed 1 decoding accuracy (**71.88%**) and contribute to the multi-seed Grand Mean of **77.12% ± 9.99%**.
   - **Cross-Dataset Evaluation**: `model_S1.pt` is the exact frozen checkpoint evaluated on all 1,080 DTU trials (producing **68.24%** accumulated Pearson accuracy).
   - **Evidential Confidence Head Evaluation (`eval_confidence_multitask.py`)**:
     - Cannot be directly executed using these base checkpoints without fine-tuning.
     - The multitask confidence script (`scripts/kul/train_conformer_multitask_loso.py`) takes these base checkpoints as pretrained starting points, adds the 2,241-parameter late-fusion confidence head, trains with Outlier Exposure (OE), and outputs fine-tuned multitask checkpoints to `results/run7_multitask_conformer_loso/checkpoints/seed_1/`.
     - `eval_confidence_multitask.py` explicitly checks for those fine-tuned checkpoints and prompts the user to run `train_conformer_multitask_loso.py` if they are not present.
