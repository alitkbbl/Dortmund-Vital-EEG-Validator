<div align="center">

#  Dortmund Vital EEG Validator

**First-pass EEG quality screening for the Dortmund Vital Study resting-state dataset**

*Screen all 608 participants one subject at a time, without ever storing the data.*

<br>

![Dataset](https://img.shields.io/badge/OpenNeuro-ds005385%20v1.0.3-f28c28)
![License](https://img.shields.io/badge/license-MIT-2ea44f)
![MNE](https://img.shields.io/badge/built%20with-MNE--Python-1f77b4)
![Subjects](https://img.shields.io/badge/Cohort-608%20Subjects%20%7C%202200%2B%20Runs-purple)


<br>

[📊 **Excel: all 608 subjects**](summary_all_subjects.xlsx) ·
[🔍 **Examples**](#example-three-real-subjects) ·
[📖 **Output guide**](docs/output-guide.md) ·
[📧 **Contact**](#16--contact-and-full-outputs)

</div>


---

```bash
python qc_eeg_subject.py 28          # one subject
python qc_eeg_subject.py 1 2 3       # several subjects
python qc_eeg_subject.py 1 to 608    # a range (the whole dataset)
```

| | |
|---|---|
| **Triage, not strict QC** | Keeps healthy or repairable recordings, rejects only catastrophic ones, with a reason code for every decision |
| **One command** | Single file, accepts numbers or ranges, downloads data automatically |
| **Minimal disk use** | Each subject's data is deleted right after analysis; runs on free Colab |
| **Resumable** | Re-run the same command after a crash; finished subjects are skipped |
| **Complete output** | CSV tables, 15 figures, a report, a run manifest and one Excel summary |
| **Reproducible** | Versions, thresholds and input checksums stored with every subject |

> 📊 **The Excel file [`summary_all_subjects.xlsx`](summary_all_subjects.xlsx) contains the complete results for all 608 subjects.**
> 📧 **If you need the figures and all other output files of the script, e-mail me and I will share them: [alitaghi8305@gmail.com](mailto:alitaghi8305@gmail.com)** (see [Contact](#16--contact-and-full-outputs)).

---

## 📋 Contents

1. [Overview](#1--overview)
2. [The dataset](#2--the-dataset)
3. [Installation](#3--installation)
4. [Usage](#4--usage)
5. [Which subjects are analysed (filters)](#5--which-subjects-are-analysed-filters)
6. [How a recording is screened](#6--how-a-recording-is-screened)
7. [Outputs](#7--outputs)
8. [The Excel overview](#8--the-excel-overview)
9. [Resuming, caching and disk space](#9--resuming-caching-and-disk-space)
10. [Run it on Google Colab](#10--run-it-on-google-colab)
11. [Reproducibility](#11--reproducibility)
12. [Configuration and FAQ](#12--configuration-and-faq)
13. [Limitations](#13--limitations)
14. [Citation](#14--citation)
15. [License](#15--license)
16. [Contact and full outputs](#16--contact-and-full-outputs)
17. [Author](#17--author)

---

## 1. 📌 Overview

Large public EEG datasets contain recordings that are perfectly good, recordings that can easily be repaired, and a few that are truly broken. Before any real analysis you need a quick, reproducible answer to one question: **which recordings can I use?**

Looking at 608 participants with up to eight recordings each by eye is slow and subjective, and strict automatic rules throw away good data. A recording should **not** be discarded only because

* the person is an *alpha non-responder* (a minority of healthy adults, roughly 10–15 %, show no strong alpha peak with closed eyes),
* the signal drifts because of sweat or skin impedance,
* there is 50 Hz mains noise (a notch filter removes it later),
* a handful of channels are bad (they can be interpolated).

This tool is a **triage**: it keeps everything that is healthy or repairable and rejects only what is catastrophic and cannot be rescued. Every decision is written down with a short reason code, so you can always see *why*.

For every subject you ask for, in this order:

1. **Eligibility check** from the participant table that is embedded in the script. Subjects that are too old or have late triggers are rejected **before anything is downloaded**; the reason is saved in their status file.
2. **Automatic download** of exactly the files that are needed. Files already on disk are not downloaded again.
3. **Quality screening** of every recording: header, channel integrity (flat, clipped, noisy, spatially uncorrelated), amplitude, spectrum (alpha, 50 Hz), consistency.
4. **Results**: CSV tables, up to 15 figures including a channel-quality topomap, a text report, a machine-readable status and a run manifest.
5. **Clean-up**: the analysed data are deleted, so the disk never fills up.

After **every** subject the Excel overview of all subjects is updated. A crash, a lost connection or Ctrl+C never destroys finished work (see [section 9](#9--resuming-caching-and-disk-space)).

---

## 2. 📚 The dataset

| | |
|---|---|
| Dataset | [OpenNeuro **ds005385**](https://openneuro.org/datasets/ds005385/versions/1.0.3) |
| Paper | *Resting-state EEG data before and after cognitive activity across the adult lifespan and a 5-year follow-up* (Scientific Data, 2024), [open access](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11387823/) |
| **Version this tool is built for** | **1.0.3** |
| Participants | **608** (`sub-001` … `sub-608`), ages 20–70 |
| Sessions | session 1, and for some participants a second session (`ses-2`) |
| Recordings per session | 4: **Eyes Closed / Eyes Open** × **pre / post** a cognitive task |
| Expected signal | 64 EEG channels, 1000 Hz, EDF files; recordings of about 3 minutes |
| Mirror used for DataLad | <https://github.com/OpenNeuroDatasets/ds005385> |

File names follow BIDS, for example `sub-028/ses-1/eeg/sub-028_ses-1_task-EyesClosed_acq-pre_eeg.edf`.

**About the version.** With DataLad (the default when it is installed) the script checks out the tag `1.0.3`, so you always get exactly that version. Every `status.txt` records the download method and, with DataLad, the tag and the Git commit that was used (sub-028 and sub-053: tag `1.0.3`, commit `6a558cd58525`; sub-325 was downloaded with the HTTPS fallback, which records no tag). With the plain HTTPS fallback it reads the newest snapshot on the server and prints a warning, because that is not guaranteed to be 1.0.3.

---

## 3. 🔧 Installation

Requires Python **3.10 or newer** (developed and tested with 3.12).

```bash
git clone https://github.com/alitkbbl/dortmund-vital-eeg-validator.git
cd dortmund-vital-eeg-validator
pip install mne edfio numpy scipy pandas matplotlib openpyxl
```

Data is downloaded automatically over HTTPS.

---

## 4. 🚀 Usage

Type **numbers only** (no `sub-` needed). There are 608 subjects, numbered 1 to 608.

```bash
python qc_eeg_subject.py 28              # one subject
python qc_eeg_subject.py 1 2 3           # several subjects, separated by spaces
python qc_eeg_subject.py 1 to 20         # a range
python qc_eeg_subject.py 1 5 to 8 12     # mixed: 1, 5, 6, 7, 8 and 12
python qc_eeg_subject.py                 # no argument: the script asks "Subjects >"
```

The same text can be typed at the `Subjects >` prompt. Duplicates are ignored, wrong input gets a clear message.

### Options

| Option | Meaning |
|---|---|
| `--out-root qc_output` | where the results go (default `qc_output`) |
| `--data-dir data` | where recordings are put temporarily (default `data`) |
| `--repo-dir ds005385_repo` | DataLad clone of the dataset (default `ds005385_repo`) |
| `--download-method auto\|datalad\|https` | how to download |
| `--no-download` | do not download; use files already in `--data-dir` (your own files are **never** deleted) |
| `--keep-data` | do not delete the downloaded data after the analysis |
| `--force-reanalysis` | analyse again even if results of the same QC version exist |
| `--force-download` | download again even if the files are on disk |
| `--stop-after-failures N` | stop after N subjects *in a row* whose download failed (default 3, `0` = never) |
| `--verify-table FILE.tsv` | compare the participant table inside the script with a TSV and exit |
| `--debug` | show the full traceback and stop at the first problem |

Run `python qc_eeg_subject.py --help` for the full text. Exit codes: `0` everything finished, `1` a technical problem (error, no data, Ctrl+C), `2` invalid input or every requested subject was rejected.

### What you see in the terminal

Every subject is shown in five steps (eligibility, download, analysis, saving, clean-up) with a live spinner and timing for each task, then a coloured result block. At the end of a batch you get a summary table. A shortened example (the values are illustrative):

```text
====================================================================================================
[1/1]  sub-028     sex M, age 48, right-handed     elapsed 54 s
====================================================================================================

  Step 1/5  Eligibility check
    [PASS] Age <= 60              age 48
    [PASS] Session 1 completed    yes
    [PASS] No late triggers       late_ses1=0, late_ses2=0
    [PASS] Session 2 completed    yes
    ✔ Eligible. Sessions to analyse: ses-1, ses-2

  Step 2/5  Data: download
    Source: OpenNeuro ds005385 v1.0.3  |  method: https  |  8 EDF file(s) to download
    ! HTTPS downloads the newest OpenNeuro snapshot, which is not guaranteed to be v1.0.3. Use DataLad for an exactly pinned version.
    → participants.tsv cross-check: participants.tsv not found (check skipped)

  Step 3/5  Quality-control analysis
    ✔ 8 of 8 expected EDF files are on disk.
    ✔ Analysing recording 1/8: ses-1  EyesClosed  pre  (5.0 s)
    ✔ Analysing recording 2/8: ses-1  EyesClosed  post  (4.1 s)
    ✔ Analysing recording 3/8: ses-1  EyesOpen  pre  (4.6 s)
    ✔ Analysing recording 4/8: ses-1  EyesOpen  post  (4.6 s)
    ✔ Analysing recording 5/8: ses-2  EyesClosed  pre  (4.0 s)
    ✔ Analysing recording 6/8: ses-2  EyesClosed  post  (4.0 s)
    ✔ Analysing recording 7/8: ses-2  EyesOpen  pre  (4.2 s)
    ✔ Analysing recording 8/8: ses-2  EyesOpen  post  (4.4 s)
    Verdict per recording

  Step 4/5  Saving tables, figures and report
    ✔ Saving tables (CSV)  7 files  (0.1 s)
    ⠋ Drawing figures (A-G and the channel quality topomap)  5s ....
```

---

## 5. 🎯 Which subjects are analysed (filters)

The participant table of the dataset (age, completed sessions, late triggers) is **embedded in the script**, so a subject can be judged before a single byte is downloaded.

| Rule | Decision |
|---|---|
| **Age above 60** | ❌ **rejected, nothing is downloaded** |
| **Late triggers** in any session (`late_ses1` or `late_ses2` greater than 0) | ❌ **rejected, nothing is downloaded** |
| Session 1 not completed | ❌ rejected |
| **Only session 1** completed | ✅ accepted, only `ses-1` (4 recordings) is analysed |
| Both sessions completed | ✅ accepted, 8 recordings are analysed |

The reason for every rejection is written into that subject's `status.txt`, for example `Age above 60 (67)` or `Late triggers (late_ses1=1901, late_ses2=n/a)`.

**What this means for the 608 subjects** (computed from the embedded table):

| | Subjects |
|---|---:|
| Age above 60 | 94 |
| Late triggers | 99 |
| Both reasons | 12 |
| **Rejected (not downloaded)** | **181** |
| **Accepted** | **427** |
| … of which both sessions (8 recordings) | 142 |
| … of which session 1 only (4 recordings) | 285 |
| Recordings to screen if all accepted subjects are run | 2,276 |

> 💡 Check that the embedded table matches your own copy of the participant file: `python qc_eeg_subject.py --verify-table participants.tsv` lists every difference. After a download the script also compares each subject's row with the dataset's own `participants.tsv` and warns about differences.

---

## 6. 🔬 How a recording is screened

Every recording passes five layers. Every finding carries a verdict and a short **reason code**.

* **PASS** – nothing to note
* **FLAG** – small problem, the recording is kept
* **FAIL** – clear problem, the recording is not usable
* **MISSING** – the file is not available (never stops the run; the other recordings are analysed normally)

### Layers

| # | Layer | What is checked | FAIL | FLAG |
|---|---|---|---|---|
| 1 | **Header / metadata** | EEG channel count, sampling rate, duration, unique channel names, O1/O2/Oz present, JSON sidecar | not 64 EEG channels, not 1000 Hz, shorter than 1 min, duplicate or empty names, O1/O2/Oz missing | duration outside 2–4 min, sidecar differs from the file |
| 2 | **Channel integrity** | flat, clipped, noisy and **spatially uncorrelated** channels; frozen segments | see the decision logic below | one critical channel bad; frozen segment over 5 s |
| 3 | **Amplitude** | typical amplitude (median 1-second STD of the baseline-corrected signal) | median outside 1–100 µV (probable scale problem) | more than 10 % of channels above 100 µV |
| 4 | **Spectrum** | alpha peak and Eyes Closed / Eyes Open alpha blocking, 50 Hz line noise | dead spectrum on O1/O2/Oz; 50 Hz noise together with clipping of half of the channels | weak alpha, line noise |
| 5 | **Consistency** | same channels, order and sampling rate in all recordings of the subject | differs | – |

### Design decisions (what is deliberately *not* a FAIL)

1. **Alpha is only a FLAG.** A weak alpha peak or an Eyes Closed / Eyes Open ratio below 1 is common in healthy people. Codes: `LOW_ALPHA_PROMINENCE`, `WEAK_ALPHA_BLOCKING`, `NO_ALPHA_BLOCKING`. The only alpha-related FAIL is a **dead spectrum** (no signal power at all on O1, O2 and Oz) or a metadata error.
2. **Slow drift does not fool the statistics.** Amplitude, variance, skewness, kurtosis and the spatial correlation are computed on a baseline-corrected **copy** of the signal (median removed, 0.5 Hz zero-phase high-pass), **in memory only**. Raw signals are never modified. Flat, clipping and frozen-sample checks and the DC-offset / drift numbers use the raw signal, because correcting it would hide saturation.
3. **Spatial correlation finds bad channels that look normal.** Each channel's correlation (|Pearson r|, 1–45 Hz, 5-second windows) with the median of all channels is computed. A channel below **0.20** is counted as *uncorrelated / noisy* (this is the most data-dependent criterion, see [Limitations](#13--limitations)) (**0.18** for O1, O2 and Oz, because strong local occipital alpha lowers their correlation). *Safeguard:* if more than half of the channels fall below their threshold, the reference itself is not trusted: nobody is marked bad and the recording gets `FLAG LOW_GLOBAL_SPATIAL_CORRELATION`.
4. **50 Hz line noise is not a FAIL.** Above 20 dB it is a `FLAG (LINE_NOISE)`, above 30 dB a `FLAG (HIGH_LINE_NOISE)`; a notch filter fixes it later. It becomes a FAIL only when it comes **with permanent saturation** (at least half of the channels clipped): `LINE_NOISE_SATURATION`.
5. **Bad channels are judged by importance.** Cz, Fz, Pz, O1 and O2 are *critical channels*.

### Bad-channel decision logic

| Situation | Verdict | Code |
|---|---|---|
| **2 or more critical channels** bad (for example O1 and O2) | ❌ **FAIL** | `CRITICAL_CHANNELS_BAD` |
| **More than 6** bad channels in total | ❌ **FAIL** | `TOO_MANY_BAD_CHANNELS` |
| 1 critical channel bad | ⚠️ FLAG (still usable after interpolation) | `CRITICAL_CHANNEL_BAD` |
| **1 to 6** bad channels, at most one critical | ✅ **PASS**, label *needs interpolation* | `USABLE_NEEDS_INTERPOLATION` |

### Labels and reason codes

Every recording gets one label next to its verdict: `USABLE`, `USABLE_WITH_FLAGS`, `USABLE_NEEDS_INTERPOLATION` or `REJECT`. A subject gets the most demanding label of its recordings, or **`PARTIALLY_USABLE`** when some recordings are usable and others are rejected (keep the usable ones).

The full list of reason codes and labels is in the [output guide](docs/output-guide.md) and in the `Legend` sheet of the Excel file. All thresholds live in the `CONFIG` dictionary at the top of the script ([section 12](#12--configuration-and-faq)).

---

## 7. 📦 Outputs

Results go to `qc_output/` (change with `--out-root`):

```text
qc_output/
├── summary_all_subjects.xlsx        ← ALL subjects in one Excel file, updated after every subject
├── summary_all_subjects.csv         ← the "Subjects" sheet as CSV
├── sub-002/                         ← a rejected subject: only the status files, nothing downloaded
│   ├── status.txt
│   └── status.json
└── sub-028/                         ← an analysed subject
    ├── status.txt                   ← the whole story of the subject in plain text
    ├── status.json                  ← the same, machine-readable
    └── qc_results/
        ├── df_recordings.csv        ← one row per recording (verdict, label, codes, key numbers)
        ├── df_recordings_full.csv   ← every metric of every layer
        ├── df_subjects.csv          ← one row for the subject
        ├── df_channels.csv          ← one row per channel and recording (all statistics and flags)
        ├── qc_issues.csv            ← every FAIL / FLAG / MISSING finding and PASS note, worst first
        ├── qc_signoff.csv           ← a sheet for the human reviewer (suggested decision, empty decision column)
        ├── qc_pivot.csv             ← verdict per recording in one row
        ├── qc_report.txt            ← readable report
        ├── run_manifest.json        ← everything needed to reproduce the run
        ├── run_log.txt              ← the terminal text of this subject
        └── figures/                 ← up to 15 figures: 15 for a two-session subject, 11 for session 1 only (example: sub-325)
```

**`status.txt` / `status.json`** exist for **every** requested subject, also for rejected ones. They contain the status (`COMPLETED`, `REJECTED`, `NO_DATA`, `ERROR`, `INTERRUPTED`), a one-line summary, the eligibility rules with their result, what was downloaded, the verdict and label of every recording, key findings, a "needs attention" list, the clean-up result and warnings.

### Figures

| File | What it shows |
|---|---|
| `sub-<ID>_task-rest_channel_quality_topomap.png` | **Channel Quality Topomap.** One head per recording, every channel coloured: 🟢 good · 🟡 noisy / poor spatial correlation · 🟠 high 50 Hz or voltage drift · 🔴 bad / flat / clipped. A bad **critical** channel (Cz, Fz, Pz, O1, O2) gets a thick red square and a star. |
| `A_psd_occipital_sub-<ID>.png` | Power spectrum of O1/O2/Oz, Eyes Closed vs Eyes Open: alpha peak and alpha blocking, 50 Hz line |
| `B_amplitude_heatmap_sub-<ID>.png` | Typical amplitude of every channel in every recording; red crosses = flagged bad |
| `C_traces_sub-<ID>.png` | Six seconds of EEG of all channels plus a bar chart of typical amplitude per channel |
| `D_psd_all_channels_sub-<ID>.png` | Spectrum of all channels as an image: a bad channel is a horizontal band, line noise a vertical stripe |
| `E_scalp_3d_sub-<ID>.png` | Four scalp maps (amplitude, alpha blocking, 50 Hz noise, share of recordings flagged bad), top and side view |
| `F_overview_…_<ses>_<task>_<acq>.png` | One per recording: the whole recording on O1/O2/Oz with frozen segments shaded, and its spectrum |
| `G_spatial_correlation_sub-<ID>.png` | Spatial correlation of every channel: per recording, sorted, and over time |

➡️ **Examples with every figure:** [sub-028](examples/sub-028/README.md) · [sub-053](examples/sub-053/README.md) · [sub-325](examples/sub-325/README.md).  **Every file and figure explained:** [output guide](docs/output-guide.md).

### Example: three real subjects

`sub-028` and `sub-053` have two sessions (eight recordings); `sub-325` completed only session 1 (four recordings) and is a clean example. The numbers are taken from the `status.txt` files in the examples.

| Subject | Verdict | Subject label | PASS / FLAG / FAIL | Usable recordings | Highlights | Time* |
|---|---|---|---|---|---|---|
| [**sub-028**](examples/sub-028/README.md) | FLAG | `USABLE_NEEDS_INTERPOLATION` | 2 / 6 / 0 | **8 of 8** | bad channels in 6 recordings (mostly F5, F6, F3), alpha blocking OK (median ratio 4.5), notch filter suggested for 4 recordings | 310 s |
| [**sub-053**](examples/sub-053/README.md) | FAIL | `PARTIALLY_USABLE` | 1 / 3 / 4 | **4 of 8** | 4 recordings rejected for too many bad channels (more than 6), the 4 others usable after interpolation, alpha blocking OK (median ratio 6.1) | 174 s |
| [**sub-325**](examples/sub-325/README.md) | PASS | `USABLE` | 4 / 0 / 0 | **4 of 4** | no bad channels, alpha blocking OK (median ratio 4.7), no notch filter needed | 58 s |

\* wall-clock time for one subject including download and clean-up. sub-028 and sub-053 were downloaded with DataLad, sub-325 with the HTTPS download.

---

## 8. 📊 The Excel overview

📊 **[`summary_all_subjects.xlsx`](summary_all_subjects.xlsx)** in the root of this repository is the **complete automatic QC summary of all 608 subjects** (`sub-001` … `sub-608`), produced with QC logic version `screening-2.2`. **[`summary_all_subjects.csv`](summary_all_subjects.csv)** has the same content as the `Subjects` sheet. In every data sheet the rows are sorted by subject number. Subjects that were rejected before the download (age above 60 or late triggers) are listed with their basic information and the reason, but they were not QC-analysed.

The workbook has **six sheets**:

| Sheet | Size | What it contains |
|---|---|---|
| 🗂️ **Overview** | one page | a one-look dashboard. Every number is a **live formula** on `Subjects`, so it updates when the data change. It covers data coverage, status and rejection reasons, verdicts, the usability label, recording statistics, demographics and age groups, the most frequent problems, bad critical channels, signal metrics, a few plain-language conclusions and **8 charts** |
| 👤 **Subjects** | 608 rows × 33 columns | **one row per subject** |
| 🎞️ **Recordings** | 2,276 rows × 35 columns | **one row per EEG recording** |
| 🏅 **Recording_Ranking** | 2,276 rows × 18 columns | the same recordings, independent of the subject, **ranked from best to worst by a 0–100 quality score** |
| ⚠️ **Issues** | 4,184 rows × 10 columns | every finding |
| 📖 **Legend** | – | definitions of statuses, verdicts, labels, reason codes and the main thresholds |

### `Subjects`: one row per subject

Status (`COMPLETED` or `REJECTED`), QC verdict (`PASS` / `FLAG` / `FAIL`), subject label, one-line summary, reason for rejection (age above 60 or late triggers), sex, age and handedness, sessions analysed, the number of recordings by verdict and by label, usable recordings, bad channels (with the number of recordings in which each was flagged), critical channels that were bad, reason codes, the median alpha ratio (Eyes Closed / Eyes Open) and the median 50 Hz noise, run time, whether the data were deleted, and the QC version. 427 subjects are `COMPLETED` and 181 are `REJECTED`.

### `Recordings`: one row per recording

An analysed subject has four recordings per session (Eyes Closed and Eyes Open, each before and after the task); 142 subjects have two sessions, so there are **2,276 recordings** in total. Columns: the verdict of every layer (header, integrity, amplitude, spectral, consistency), the overall verdict, label and reason codes, the number and names of bad, critical and spatially uncorrelated channels, the spatial correlation, amplitude and kurtosis, the alpha peak, its prominence and the Eyes Closed / Eyes Open ratio, the 50, 100 and 150 Hz line noise, frozen time and the number of trigger events.

### `Recording_Ranking`: the best recordings first

The same 2,276 recordings, ordered by a **quality score from 0 to 100** (best first). The score is built from seven signal criteria: bad channels, critical bad channels, uncorrelated channels, spatial correlation, 50 Hz noise, frozen segments and duration. All `REJECT` recordings are placed last. The weights and thresholds are in **blue cells** next to the table and can be changed, and the ranking follows. Use it to pick the best recordings of the dataset without looking at the subjects.

### `Issues`: what exactly is wrong

One row per `FAIL`, `FLAG` or `MISSING` finding and per `PASS` note: subject, session, task, layer, reason code, the measured value and the limit it was compared with, and the verdict. Example rules from the `Legend`: more than 6 bad channels or two bad critical channels is a `FAIL`; 50 Hz noise above 20 dB is a `FLAG`.

📖 **[Every sheet and column explained](docs/excel-guide.md)**

> ℹ️ **Which sheets does the script write?** `qc_eeg_subject.py` writes `summary_all_subjects.xlsx` (and `.csv`) into your results folder with the data sheets **Subjects, Recordings, Issues and Legend**, and rebuilds the file from the subject folders after every subject, so it also contains subjects from earlier runs. The `Overview` dashboard and the `Recording_Ranking` sheet of the workbook in this repository were added on top of that output and are not written by the script. The file is written atomically (a half-written file never replaces a good one); if it is open in Excel on Windows, the script warns and updates it again after the next subject.

---

## 9. 🔁 Resuming, caching and disk space

### Safe to stop at any time 🔁

Every subject is saved when it is finished. If you press **Ctrl+C**, the computer crashes, the connection drops or Colab disconnects, **nothing that was finished is lost or touched**. Simply run **the same command again**:

```bash
python qc_eeg_subject.py 1 to 100      # crashed at subject 50
python qc_eeg_subject.py 1 to 100      # same command: 1–49 are skipped in a second, work continues at 50
```

Finished subjects appear as one line (`already analysed, skipped`); nothing is downloaded or rewritten for them. At the end of an unfinished run the script prints exactly what is missing and a command for only those subjects.

*"Already analysed"* means: results exist **and** they were produced by the same QC logic and thresholds (a fingerprint is stored in each `status.json`). If you change `CONFIG`, old results are re-analysed automatically. `--force-reanalysis` forces a new analysis.

### Network problems

If the download of a subject is incomplete, that subject is analysed with what arrived, marked `DOWNLOAD INCOMPLETE`, **never treated as finished**, and its files are kept so the retry does not download them again. If the download fails for **3 subjects in a row** (`--stop-after-failures`), the batch stops by itself and tells you how to continue, instead of filling the results with failures.

### Disk space

* The data of a subject are **deleted right after it has been analysed**: the copy in `data/` and the downloaded content in the DataLad clone (`datalad drop`). At any time only the current subject is on disk.
* Files that are already in `data/` are not downloaded again.
* Data are **kept** with `--keep-data`, with `--no-download` (your own files are never deleted), after an unexpected error, and when the download was incomplete.
* Problems while deleting (permissions on Windows/WSL, locked git-annex files) only produce a warning; the analysis stays `COMPLETED` and the batch goes on.

---

## 10. 🌐 Run it on Google Colab

You do **not** need to own any data. The script downloads one subject, analyses it, deletes the data and moves on, so the Colab disk never fills up.

**Cell 1: mount Drive, install, get the script**

```python
from google.colab import drive
drive.mount('/content/drive')
```

```bash
!pip install -q "mne>=1.6" edfio openpyxl
!wget -q https://raw.githubusercontent.com/alitkbbl/dortmund-vital-eeg-validator/main/qc_eeg_subject.py
```

**Cell 2: run**

```bash
!python qc_eeg_subject.py 1 to 608
```

**Cell 3: copy the results to Drive**

```bash
!rsync -avh --progress /content/qc_output/ /content/drive/MyDrive/qc_output/
```

Run Cell 3 from time to time, and always before the session ends: files in `/content` are lost when the runtime disconnects. All 427 accepted subjects take roughly **10 hours**, so plan for more than one session.

📘 For writing results directly to Drive, resuming in a new session and pinning exact version 1.0.3 with DataLad, see the **[Colab guide](docs/colab.md)**.

---

## 11. 🧾 Reproducibility

* `run_manifest.json` of every subject records the exact command line, the versions of Python, MNE, NumPy, SciPy, pandas and matplotlib, the dataset version and Git commit (DataLad), the complete `CONFIG`, the SHA-256 and size of every input file, the participant row used for the eligibility decision and the list of outputs.
* There is no randomness anywhere in the analysis.
* `QC_LOGIC_VERSION` and a fingerprint of `CONFIG` are stored with the results; results are only reused when both match.
* Raw files are never modified; baseline correction exists only in memory.

---

## 12. 🔩 Configuration and FAQ

### Changing the thresholds

Open `qc_eeg_subject.py` and edit the `CONFIG` dictionary near the top. Every threshold has a comment. After editing, old results are re-analysed automatically because the `CONFIG` fingerprint changed.

| Key | Default | Meaning |
|---|---|---|
| `corr_threshold`, `corr_threshold_relaxed` | 0.20, 0.18 | spatial-correlation threshold; the relaxed one is for O1, O2, Oz |
| `critical_channels` | Cz, Fz, Pz, O1, O2 | channels whose failure matters most |
| `critical_bad_fail_min` | 2 | number of bad critical channels that makes a recording FAIL |
| `interpolatable_max_channels` | 6 | up to this many bad channels are repairable (PASS) |
| `detrend_method`, `detrend_hp_hz` | highpass, 0.5 | baseline correction before the voltage statistics |
| `alpha_prominence_min_db`, `alpha_ratio_pass` | 3.0, 1.5 | alpha FLAG thresholds |
| `line_flag_db`, `line_high_db` | 20, 30 | 50 Hz FLAG thresholds |
| `typical_amp_min_uv`, `typical_amp_max_uv` | 1, 100 | physiological amplitude range |

### FAQ

**Why was a subject rejected without being downloaded?** Age above 60 or late triggers. Open that subject's `status.txt` for the exact reason.

**What does `NO_DATA` mean?** None of the expected recordings could be analysed (missing in the dataset or the download failed). It is not treated as finished and will be tried again.

**A recording is `FLAG` for alpha. Is it bad?** No. Weak alpha is common in healthy people and is only a flag.

**How do I run everything again?** `python qc_eeg_subject.py 1 to 608 --force-reanalysis`.

**`ModuleNotFoundError: edfio`** Run `pip install edfio`.

**`Excel overview skipped: openpyxl`** Run `pip install openpyxl`. The CSV overview is always written.

**`datalad` or `git-annex` not found.** Install them, or use `--download-method https`.

**The Excel file is locked.** Close it in Excel. The script writes it again after the next subject.

---

## 13. 🚧 Limitations

* This is a **first-pass screening**, not a replacement for expert review or a full preprocessing pipeline. The thresholds are practical heuristics and have **not** been validated against manual ratings. Check them on a few subjects of your own and adjust `CONFIG`.
* The **spatial-correlation criterion is the most data-dependent one**. It compares each channel with the median of all channels, so referencing, the position of the reference electrode and frontal eye activity can lower the correlation of some channels (typically frontal and fronto-central ones) in many recordings of a subject. The built-in safeguard (`LOW_GLOBAL_SPATIAL_CORRELATION`) only covers the extreme case. Look at `corr_median` in `df_channels.csv` for the channels that are flagged again and again, and adjust `corr_threshold` / `corr_threshold_relaxed` if needed.
* The script judges the signal only. It does not interpret trigger codes or check the cognitive task.
* The electrode positions in the topomap come from MNE's standard templates (10-20, with 10-05 for the other channel names), not from individual digitisation.
* The eligibility filters (age, late triggers) are project criteria, not statements about data quality.

---

## 14. 📖 Citation

Please cite the **dataset** and its paper (see the [dataset page](https://openneuro.org/datasets/ds005385/versions/1.0.3) for the exact reference and license terms): *Resting-state EEG data before and after cognitive activity across the adult lifespan and a 5-year follow-up*, Scientific Data (2024).

---

## 15. 📄 License

Released under the [MIT License](LICENSE): free to use, modify and share, with attribution.

---

## 16. 📧 Contact and full outputs

The repository contains

* the script,
* [`summary_all_subjects.xlsx`](summary_all_subjects.xlsx): the **complete results for all 608 subjects** (one row per subject, one row per recording, every finding), and
* the full output of three subjects ([sub-028](examples/sub-028/README.md), [sub-053](examples/sub-053/README.md), [sub-325](examples/sub-325/README.md)) including all figures.

The figures and the other per-subject files (CSV tables, reports, manifests) of the other subjects are not stored here because of their size.

📧 **Need them for all or for selected subjects? Send me an e-mail with the subject numbers and I will share them with you:**

**[alitaghi8305@gmail.com](mailto:alitaghi8305@gmail.com)**

Questions, bug reports and suggestions are welcome as GitHub issues, too.

---

## 17. 👤 Author

**Ali Taghizadeh**
📧 [alitaghi8305@gmail.com](mailto:alitaghi8305@gmail.com) · [GitHub](https://github.com/alitkbbl)
