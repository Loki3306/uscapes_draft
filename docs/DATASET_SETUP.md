# Dataset Setup and Directory Layout

This document describes the expected directory structure, naming conventions, and preprocessing specifications for the two datasets used in the paper:
1. **DTU Auditory Attention Dataset** (Fuglsang et al.)
2. **KUL Auditory Attention Dataset** (Das et al.)

---

## 1. DTU Dataset Structure

The DTU dataset consists of 18 healthy subjects listening to dual-speaker competing stories under various acoustic conditions (anechoic, reverberant).

### Expected Directory Hierarchy
```text
data/
└── dtu/
    ├── S1.mat
    ├── S2.mat
    ├── S3.mat
    ...
    └── S18.mat
```

### MATLAB File Format
Each `S<id>.mat` file contains:
- `data`: Cell array of 60 trials.
  - `eeg`: Continuous EEG recorded at 512 Hz across 64 BioSemi active electrodes.
  - `fsample`: Original sampling rate (512 Hz).
  - `wavA`, `wavB`: Filenames of the attended and unattended audio streams.
- `metadata`: Trial condition details.

### 8-Channel Montage Selection
The 8 electrodes selected via Ridge regression decoder weights on all 18 subjects are:
- **0-indexed indices**: `[13, 46, 43, 23, 50, 0, 52, 14]`
- **BioSemi Labels**: `C5, FCz, FC6, P9, C6, Fp1, TP8, T7`
- *Note*: As documented in [REPRODUCTION_NOTES.md](REPRODUCTION_NOTES.md), this montage includes the vertex electrode `FCz` alongside lateral scalp electrodes.

### Audio Mapping
Trial audio correspondences are mapped in `data/audio_mapping.json`:
- Contains trial-by-trial mappings for all 1,080 trials across the 18 subjects.

---

## 2. KUL Dataset Structure

The KUL dataset comprises 16 normal-hearing subjects listening to competing male-female audio tracks across 20 trials per subject.

### Expected Directory Hierarchy
```text
data/
└── kul/
    ├── S1/
    │   ├── S1_trial1.mat
    │   └── ...
    ├── S2/
    ...
    └── S16/
```

### MATLAB File Format
Each trial `.mat` contains:
- `TrialData`: Struct with fields:
  - `EEG`: Continuous 64-channel EEG sampled at 8192 Hz (resampled to 128 Hz or 64 Hz).
  - `Audio`: Left and right acoustic channels.
  - `fs`: Sampling frequency.

### Preprocessing & Caching
Executing `python src/kul/data/build_kul_cache.py` processes raw files into `data/processed_kul/`:
1. **Common Average Referencing (CAR)**: Computed across all 64 channels prior to channel selection.
2. **Bandpass Filtering**: 1.0–8.0 Hz (Chebyshev / Butterworth filter).
3. **Electrode Subsampling**: 8 channels: `['T7', 'C2', 'FT8', 'P7', 'CPz', 'Fp1', 'TP8', 'C3']`.
4. **Gammatone Envelope Extraction**: 28 ERB-spaced cochlear subbands (50–8000 Hz), $\gamma = 0.6$ power compression, 8 Hz lowpass, downsampled to 64 Hz.
5. **Output Structure**:
   ```text
   data/processed_kul/
   ├── S1_data.pt
   ├── S2_data.pt
   ...
   └── S16_data.pt
   ```

---

## 3. Storage and Resource Recommendations

- **Raw DTU Dataset**: ~15 GB
- **Raw KUL Dataset**: ~22 GB
- **Processed KUL Cache (`data/processed_kul/`)**: ~1.8 GB
- **DTU Gammatone Envelopes (`gammatone_envelopes.pkl`)**: ~650 MB
- **Checkpoints**: ~35 MB (all 16 KUL Conformer models)
