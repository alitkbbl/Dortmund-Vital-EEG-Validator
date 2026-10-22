# 🔍 sub-028: example output

The unmodified output of

```bash
python qc_eeg_subject.py 28
```

for `sub-028` of [OpenNeuro ds005385 v1.0.3](https://openneuro.org/datasets/ds005385/versions/1.0.3) (two sessions, eight recordings).
How to read each file: [output guide](../../docs/output-guide.md)  ·  back to the [main README](../../README.md)

## Result at a glance

| | |
|---|---|
| **QC verdict** | **FLAG** |
| **Subject label** | `USABLE_NEEDS_INTERPOLATION` |
| Recordings PASS / FLAG / FAIL | 2 / 6 / 0 |
| Usable recordings | **8 of 8** |
| Time for this subject (incl. download) | 309.5 s |

> overall FLAG; USABLE_NEEDS_INTERPOLATION: 8/8 recordings usable; bad channels in 6 recording(s) (F5, F6, F3); alpha blocking OK (median 4.5); notch filter suggested for 4/8 recordings

| Session | Task | Acq | Verdict | Label | Bad channels | Which | Alpha EC/EO | 50 Hz (dB) |
|---|---|---|---|---|---:|---|---:|---:|
| ses-1 | EyesClosed | pre | FLAG | `USABLE_WITH_FLAGS` | 0 | - | 4.98 | 1.15 |
| ses-1 | EyesClosed | post | FLAG | `USABLE_WITH_FLAGS` | 0 | - | 31.99 | 0.23 |
| ses-1 | EyesOpen | pre | PASS | `USABLE_NEEDS_INTERPOLATION` | 2 | F5, F6 | 4.98 | 3.67 |
| ses-1 | EyesOpen | post | PASS | `USABLE_NEEDS_INTERPOLATION` | 5 | F7, F3, F4, F5, F6 | 31.99 | 2.1 |
| ses-2 | EyesClosed | pre | FLAG | `USABLE_NEEDS_INTERPOLATION` | 1 | Fz | 0.17 | 24.07 |
| ses-2 | EyesClosed | post | FLAG | `USABLE_NEEDS_INTERPOLATION` | 1 | Fz | 4.01 | 20.96 |
| ses-2 | EyesOpen | pre | FLAG | `USABLE_NEEDS_INTERPOLATION` | 5 | F3, F4, AF8, F5, F6 | 0.17 | 20.78 |
| ses-2 | EyesOpen | post | FLAG | `USABLE_NEEDS_INTERPOLATION` | 4 | F7, FC2, F5, F6 | 4.01 | 23.13 |

## Folder structure

```text
sub-028/
├── README.md                          (this file)
├── status.txt                         summary of the whole subject
├── status.json                        the same, machine-readable
└── qc_results/
    ├── df_recordings.csv              one row per recording
    ├── df_recordings_full.csv         every metric of every layer
    ├── df_subjects.csv                one row for the subject
    ├── df_channels.csv                one row per channel and recording
    ├── qc_issues.csv                  every FAIL / FLAG / MISSING finding
    ├── qc_signoff.csv                 sheet for the human reviewer
    ├── qc_pivot.csv                   verdict per recording in one row
    ├── qc_report.txt                  readable report
    ├── run_manifest.json              everything needed to reproduce the run
    ├── run_log.txt                    terminal text of this subject
    └── figures/
        ├── sub-028_task-rest_channel_quality_topomap.png
        ├── G_spatial_correlation_sub-028.png
        ├── A_psd_occipital_sub-028.png
        ├── B_amplitude_heatmap_sub-028.png
        ├── C_traces_sub-028.png
        ├── D_psd_all_channels_sub-028.png
        ├── E_scalp_3d_sub-028.png
        └── F_overview_sub-028_<session>_<task>_<acq>.png    (8 files)
```

## Key files

| | |
|---|---|
| [`status.txt`](status.txt) | verdict, label, per-recording table, "needs attention" |
| [`qc_results/qc_report.txt`](qc_results/qc_report.txt) | readable report |
| [`qc_results/df_recordings.csv`](qc_results/df_recordings.csv) | one row per recording |
| [`qc_results/qc_issues.csv`](qc_results/qc_issues.csv) | all findings with reason codes |
| [`qc_results/df_channels.csv`](qc_results/df_channels.csv) | per-channel statistics and flags |
| [`qc_results/run_manifest.json`](qc_results/run_manifest.json) | reproducibility record |

## Figures

### Channel quality topomap
![Channel quality topomap](qc_results/figures/sub-028_task-rest_channel_quality_topomap.png)

### Spatial correlation
![Spatial correlation](qc_results/figures/G_spatial_correlation_sub-028.png)

### Occipital spectrum (alpha and 50 Hz)
![Occipital spectra](qc_results/figures/A_psd_occipital_sub-028.png)

### Amplitude per channel and recording
![Amplitude heatmap](qc_results/figures/B_amplitude_heatmap_sub-028.png)

### Six seconds of EEG
![Traces](qc_results/figures/C_traces_sub-028.png)

### Spectrum of all channels
![Spectrum of all channels](qc_results/figures/D_psd_all_channels_sub-028.png)

### 3D scalp maps
![Scalp maps](qc_results/figures/E_scalp_3d_sub-028.png)

<details>
<summary><b>Whole-recording overviews (8 figures, one per recording)</b></summary>

**ses-1, Eyes Closed, pre**  
![sub-028 ses-1 EyesClosed pre](qc_results/figures/F_overview_sub-028_ses-1_EyesClosed_pre.png)

**ses-1, Eyes Closed, post**  
![sub-028 ses-1 EyesClosed post](qc_results/figures/F_overview_sub-028_ses-1_EyesClosed_post.png)

**ses-1, Eyes Open, pre**  
![sub-028 ses-1 EyesOpen pre](qc_results/figures/F_overview_sub-028_ses-1_EyesOpen_pre.png)

**ses-1, Eyes Open, post**  
![sub-028 ses-1 EyesOpen post](qc_results/figures/F_overview_sub-028_ses-1_EyesOpen_post.png)

**ses-2, Eyes Closed, pre**  
![sub-028 ses-2 EyesClosed pre](qc_results/figures/F_overview_sub-028_ses-2_EyesClosed_pre.png)

**ses-2, Eyes Closed, post**  
![sub-028 ses-2 EyesClosed post](qc_results/figures/F_overview_sub-028_ses-2_EyesClosed_post.png)

**ses-2, Eyes Open, pre**  
![sub-028 ses-2 EyesOpen pre](qc_results/figures/F_overview_sub-028_ses-2_EyesOpen_pre.png)

**ses-2, Eyes Open, post**  
![sub-028 ses-2 EyesOpen post](qc_results/figures/F_overview_sub-028_ses-2_EyesOpen_post.png)

</details>
