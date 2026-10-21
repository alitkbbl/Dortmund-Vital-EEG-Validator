# Dortmund Vital EEG Validator

[![Dataset: OpenNeuro-ds005385](https://img.shields.io/badge/OpenNeuro-ds005385%20v1.0.3-orange.svg)](https://openneuro.org/datasets/ds005385)
[![Python: 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/)
[![MNE-Python](https://img.shields.io/badge/MNE--Python->=1.6-purple.svg)](https://mne.tools/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

An automated, high-throughput Quality Control (QC) and triage screening pipeline designed for large-scale clinical resting-state EEG datasets. Optimized specifically for **The Dortmund Vital Study** ([OpenNeuro ds005385](https://openneuro.org/datasets/ds005385), v1.0.3 — 608 participants, multi-session resting-state EEG).

---

## Key Highlights & Philosophy

- **High-Throughput Triage Philosophy:**  
  *Keep every usable recording, flag borderline ones, and discard only catastrophic failures.* Distinguishes between clean data, data requiring standard bad-channel interpolation, and unrecoverable recordings.
- **Selective On-Demand Hydration & Auto-Cleanup:**  
  Integrates natively with **DataLad / git-annex** and fallback HTTPS. Downloads only the target EDF files for the active subject and automatically frees disk space (`datalad drop` / local cleanup) post-analysis unless explicitly retained. Ideal for constrained cloud environments (e.g., Google Colab).
- **Five-Layer Diagnostic Screening:**  
  Evaluates raw signals without altering source files:
  1. **Header & Metadata:** Duration, sampling rates, coordinate frames, channel counts, and session consistency.
  2. **Channel Integrity:** Identifies flat, clipped, high-impedance/noisy, frozen, and spatially uncorrelated channels (windowed spatial cross-correlation with safeguards for dominant occipital alpha rhythms).
  3. **Amplitude Dynamics:** High-pass filtered ($0.5\text{ Hz}$) evaluation against drift, DC offsets, extreme peaks, and $z$-score deviations.
  4. **Spectral Metrics:** 50 Hz line noise quantification ($>30\text{ dB}$ flag), occipital alpha rhythm prominence ($8\text{--}12\text{ Hz}$ vs. broadband background), and dead-spectrum detection.
  5. **Cross-Run Consistency:** Inter-session impedance shifts and stability checks.
- **Fail-Safe Batch Processing:**  
  Subject-level error isolation ensures an unreadable or corrupt file does not interrupt batch execution. Handles `SIGINT` (Ctrl+C) gracefully with partial-progress saving.

---

## Decision Matrix & Screening Categories

Each recording (up to 8 runs per subject: `ses-1`/`ses-2`, `EyesOpen`/`EyesClosed`, `pre`/`post`) receives an individual decision, synthesized into an overall subject-level triage status:

| Status | Trigger Criteria | Action Recommended |
| :--- | :--- | :--- |
| **`PASS` / `USABLE`** | Clean recordings ($\le 0$ or isolated minor bad channels, healthy SNR/alpha). | Direct inclusion in downstream pipelines. |
| **`USABLE_NEEDS_INTERPOLATION`** | $1\text{--}6$ non-critical bad channels ($\le 1$ critical channel from `Cz, Fz, Pz, O1, O2`). | Spatial spherical spline interpolation required. |
| **`PARTIALLY_USABLE`** | Subject has both usable and unrecoverable sessions (e.g., clean eyes-closed, corrupt eyes-open). | Filter specific sessions prior to task-level analysis. |
| **`FLAGGED`** | Usable amplitude, but exhibits weak alpha, elevated $50\text{ Hz}$ noise, or border SNR. | Visual inspection recommended. |
| **`FAIL`** | $\ge 2$ critical channels missing, $>6$ bad channels, dead spectrum, $<60\text{ s}$ duration, scale errors, or severe clipping. | Exclude recording from scientific analysis. |

---

## Dataset Layout (ds005385)

The Dortmund Vital Study contains 608 participants with resting-state EEG collected across two sessions:
```text
sub-XXX/
└── ses-1 / ses-2/
└── eeg/
├── sub-XXX_ses-1_task-EyesClosed_acq-pre_eeg.edf
├── sub-XXX_ses-1_task-EyesClosed_acq-post_eeg.edf
├── sub-XXX_ses-1_task-EyesOpen_acq-pre_eeg.edf
└── sub-XXX_ses-1_task-EyesOpen_acq-post_eeg.edf
```
---

## Installation

### Prerequisites
- Python 3.9 or higher
- [Git](https://git-scm.com/) and [Git-Annex](https://git-annex.branchable.com/) (recommended for DataLad dataset management)

### Setup
```bash
# Clone the validator repository
git clone https://github.com/<your-username>/dortmund-vital-eeg-validator.git
cd dortmund-vital-eeg-validator
```
# Create and activate virtual environment
```python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```
# Install dependencies
pip install -r requirements.txt

#### `requirements.txt`
text
mne>=1.6.0
edfio>=0.4.0
numpy>=1.22.0
scipy>=1.9.0
pandas>=2.0.0
matplotlib>=3.6.0
datalad>=0.19.0
colorama>=0.4.6

---

## Usage

### 1. Single Subject Analysis
Run full QC on a single participant:
```bash
python validator.py --subjects 28
```
### 2. Multi-Subject & Range Processing
Process sequential ranges, individual subjects, or mixed selections:
```bash
# Range of subjects
python validator.py --subjects 1 to 20
```
# Mixed selection
python validator.py --subjects 5 12 28 35 to 40

### 3. CLI Options & Storage Flags

| Flag | Description |
| :--- | :--- |
| `--subjects` | Subject IDs, space-separated or range format (`1 to 10`). |
| `--keep-data` | Retain downloaded raw `.edf` files locally after QC (disables auto-drop). |
| `--no-download` | Run QC only on existing, locally cached files (skips DataLad fetching). |
| `--force-reanalysis`| Bypass cached QC results and re-run signal processing steps. |
| `--force-download`  | Force redownload of input files even if present locally. |
| `--verify-table`    | Verify subject metadata against the study's participant registry. |
| `--output-dir`      | Destination directory for logs, figures, and CSV summaries (default: `qc_output`). |
| `--debug`           | Enable verbose tracebacks and detailed diagnostic outputs. |

---

## Output Architecture

Results are systematically structured inside `qc_output/`:

```text
qc_output/
├── summary_all_subjects.csv        # Aggregated triage status across all analyzed subjects
├── sub-028/
│   ├── status.json                 # Machine-readable triage verdict & metadata
│   ├── status.txt                  # Human-readable summary card
│   └── qc_results/
│       ├── run_manifest.json       # Execution environment, checksums, package versions
│       ├── run_log.txt             # Console execution trace
│       ├── qc_report.txt           # Comprehensive tabular report across all 8 runs
│       ├── figures/
│       │   ├── FigA_Raw_Traces.png
│       │   ├── FigB_PSD_Spectra.png
│       │   ├── FigC_Alpha_Topomap.png
│       │   ├── FigD_Noise_Distribution.png
│       │   ├── FigE_Spatial_Correlation_Matrix.png
│       │   ├── FigF_Bad_Channels_Topomap.png
│       │   └── FigG_Run_Consistency_Grid.png
│       └── tables/
│           ├── channel_metrics.csv
│           ├── spectral_metrics.csv
│           └── run_level_summary.csv

---
```

## Example Case Study: Participant 28

Validation run on `sub-028` (8 raw runs, 392 MB total):
- **Eyes-Closed Runs:** Robust occipital alpha rhythm ($10.2\text{ Hz}$ peak at $O1, O2$), clean central sensors ($Cz, Pz, Fz$), full duration ($>300\text{ s}$). Result: **`USABLE`**.
- **Eyes-Open Runs:** Identified detached/high-noise frontal channels ($F3, F4, F5, F6, AF7, AF8$) via windowed spatial cross-correlation and high-frequency noise ceilings.
- **Triage Result:** Categorized as **`PARTIALLY_USABLE`** (Eyes-Closed runs passed cleanly; Eyes-Open requires frontal interpolation or selective omission).
- **Cleanup:** 392 MB of temporary EDF storage dropped automatically, leaving only lightweight reports and diagnostic figures ($<5\text{ MB}$).

---

## Citation & Acknowledgments

If you use this QC pipeline in your research, please cite the underlying dataset and MNE-Python:

- **Dortmund Vital Study Dataset:**  
  *OpenNeuro ds005385 (v1.0.3).* DOI: [10.18112/openneuro.ds005385.v1.0.3](https://doi.org/10.18112/openneuro.ds005385.v1.0.3)
- **MNE-Python:**  
  Gramfort et al., *MEG and EEG data analysis with MNE-Python*, NeuroImage, 2013.

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.


