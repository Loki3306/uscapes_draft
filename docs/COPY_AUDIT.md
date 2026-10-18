# Forensic Byte-Level Copy Audit

This document records the exact SHA-256 cryptographic hashes and comparison between every source file in the original repository and its corresponding copy in `paper-reproduction/`.

---

## 1. Cryptographic File Comparison Table

| New File | Original File | Source Commit / Ref | SHA-256 Original (12-char) | SHA-256 Copy (12-char) | Byte-Identical? | Nature of Modification |
|---|---|---|---|---|---|---|
| `src/dtu/models/matchnet.py` | `models/matchnet.py` | `3afe657e` (`release/aad-conformer-validated`) | `d2940ca60765` | `d2940ca60765` | **YES** | None (Byte-identical) |
| `src/dtu/models/eegnet.py` | `models/eegnet.py` | `3afe657e` (`release/aad-conformer-validated`) | `be442175866b` | `be442175866b` | **YES** | None (Byte-identical) |
| `src/dtu/models/atcnet.py` | `models/atcnet.py` | `3afe657e` (`release/aad-conformer-validated`) | `a957ff3fb6a0` | `a957ff3fb6a0` | **YES** | None (Byte-identical) |
| `src/dtu/models/eegnet_tcn.py` | `models/eegnet_tcn.py` | `3afe657e` (`release/aad-conformer-validated`) | `ca9082d58e23` | `ca9082d58e23` | **YES** | None (Byte-identical) |
| `src/kul/models/aad_conformer.py` | `models/aad_conformer.py` | `3afe657e` (`release/aad-conformer-validated`) | `97291d0893a9` | `97291d0893a9` | **YES** | None (Byte-identical) |
| `src/dtu/data/extract_gammatone_envelopes.py` | `data/extract_gammatone_envelopes.py` | `3afe657e` (`release/aad-conformer-validated`) | `956012e1ae99` | `956012e1ae99` | **YES** | None (Byte-identical) |
| `src/dtu/data/audio_mapping.json` | `scripts/verify_baseline/data/audio_mapping.json` | `694e8131` (tracked working tree) | `edf525bfc27a` | `edf525bfc27a` | **YES** | None (Byte-identical) |
| `src/kul/data/build_kul_cache.py` | `preprocessing/build_kul_cache.py` | `3afe657e` (`release/aad-conformer-validated`) | `5850a3796856` | `5850a3796856` | **YES** | None (Byte-identical) |
| `src/kul/data/kul_cached_dataset.py` | `data/kul_cached_dataset.py` | `3afe657e` (`release/aad-conformer-validated`) | `221a29b2f2f7` | `221a29b2f2f7` | **YES** | None (Byte-identical) |
| `src/dtu/confidence/inference_engine.py` | `src/confidence/inference_engine.py` | `3afe657e` (`release/aad-conformer-validated`) | `3547bc3f31cf` | `3547bc3f31cf` | **YES** | None (Byte-identical) |
| `src/dtu/confidence/selective_metrics.py` | `src/confidence/selective_metrics.py` | `4da7cd26` (historical commit) | `45370277a281` | `45370277a281` | **YES** | None (Byte-identical) |
| `src/dtu/montages.py` | `scripts/verify_baseline/training/montages.py` | `694e8131` (tracked working tree) | `1e54ff806b0d` | `1e54ff806b0d` | **YES** | None (Byte-identical) |
| `src/common/utils.py` | `analysis/interpretability/utils.py` | `3afe657e` (`release/aad-conformer-validated`) | `60aa90bdc176` | `60aa90bdc176` | **YES** | None (Byte-identical) |
| `src/common/channel_ablation.py` | `analysis/interpretability/channel_ablation.py` | `3afe657e` (`release/aad-conformer-validated`) | `9ee5fc9114c7` | `9ee5fc9114c7` | **YES** | None (Byte-identical) |
| `src/common/ridge_aad.py` | `baselines/ridge_aad.py` | `3afe657e` (`release/aad-conformer-validated`) | `14722751324f` | `14722751324f` | **YES** | None (Byte-identical) |
| `src/common/_common.py` | `analysis/_common.py` | `1aaf5be` (audit commit) | `e39a348fad50` | `e39a348fad50` | **YES** | None (Byte-identical) |
| `scripts/dtu/train_matchnet_loso.py` | `training/train_matchnet_loso.py` | `3afe657e` (`release/aad-conformer-validated`) | `10dbd1c9a00e` | `6b1f4ef2a319` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/dtu/export_matchnet_predictions.py` | `training/export_matchnet_predictions.py` | `3afe657e` (`release/aad-conformer-validated`) | `ce235758ac64` | `bd92a137bd31` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/dtu/train_confidence_model.py` | `analysis/step_5_0a_train_final_model.py` | `3afe657e` (`release/aad-conformer-validated`) | `14416fcc739d` | `14416fcc739d` | **YES** | None (Byte-identical) |
| `scripts/dtu/eval_selective_prediction.py` | `analysis/step_5_1_behavior_audit.py` | `3afe657e` (`release/aad-conformer-validated`) | `370cf00fca0c` | `370cf00fca0c` | **YES** | None (Byte-identical) |
| `scripts/dtu/eval_confidence_ablations.py` | `analysis/step_5_2a_minimal_model_audit.py` | `3afe657e` (`release/aad-conformer-validated`) | `0510cd5e82d0` | `0510cd5e82d0` | **YES** | None (Byte-identical) |
| `figures/generate_publication_figures.py` | `analysis/step_5_7_generate_publication_figures.py` | `3afe657e` (`release/aad-conformer-validated`) | `64c2c1630ca3` | `64c2c1630ca3` | **YES** | None (Byte-identical) |
| `scripts/kul/train_conformer_loso.py` | `training/train_conformer_loso.py` | `3afe657e` (`release/aad-conformer-validated`) | `7f12ba584a99` | `75258ca63445` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/kul/train_conformer_multitask_loso.py` | `training/train_conformer_multitask_loso.py` | `3afe657e` (`release/aad-conformer-validated`) | `bc015c871894` | `136c9f39c7d3` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/kul/eval_multiseed_repro.py` | `analysis/run1_reproducibility.py` | `3afe657e` (`release/aad-conformer-validated`) | `2c528edf9c60` | `e35ce245b53e` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/kul/eval_falsification.py` | `analysis/run2_falsification.py` | `3afe657e` (`release/aad-conformer-validated`) | `c7042142cb4a` | `5a00976a6182` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/kul/eval_robustness.py` | `analysis/run4_robustness.py` | `3afe657e` (`release/aad-conformer-validated`) | `aa08843ccced` | `6d5a4c764650` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/kul/eval_selective_aad.py` | `analysis/run6_selective_aad.py` | `3afe657e` (`release/aad-conformer-validated`) | `2094b1ae0cfb` | `da15d62c3e83` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/kul/eval_confidence_multitask.py` | `analysis/run7_multitask_evaluation.py` | `3afe657e` (`release/aad-conformer-validated`) | `c1390420b61d` | `f8efa0f9ae69` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `scripts/cross_dataset/eval_kul_to_dtu_transfer.py` | `analysis/phase_10_4_independent_reproduction_audit.py` | `1aaf5be` (audit commit) | `a9a214f74d33` | `58e98bb0a0d4` | **NO** | Path adjustment: `parents[1]` $\rightarrow$ `parents[2]` for `REPO_ROOT` |
| `results/kul_conformer_multiseed_summary.json` | `conformer_loso_results/conformer_loso_multiseed_summary.json` | `3afe657e` (`release/aad-conformer-validated`) | `d8038893c09d` | `d8038893c09d` | **YES** | None (Byte-identical) |

---

## 2. Summary of Modifications

1. **All Library and Model Modules (16/16)**: **100% Byte-Identical**. Zero lines, variables, or functions were modified.
2. **Analysis and Figure Scripts (4/4)**: **100% Byte-Identical**.
3. **Execution Scripts (10/10)**: **Modified solely for repository portability**. In the original repository, scripts were located in 1-level subdirectories (`training/`, `analysis/`), resolving the repository root via `Path(__file__).resolve().parents[1]`. In `paper-reproduction/`, these scripts were placed into 2-level subdirectories (`scripts/dtu/`, `scripts/kul/`, `scripts/cross_dataset/`), requiring `parents[1]` to be replaced with `parents[2]` so that `REPO_ROOT` resolves to `paper-reproduction/` rather than `scripts/`.
4. **Behavioral Impact**: Zero algorithmic, mathematical, model-architectural, or hyperparameter changes were made.
