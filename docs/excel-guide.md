# 🗂️ Excel Guide — Every Sheet & Column

This guide documents [`summary_all_subjects.xlsx`](../summary_all_subjects.xlsx): the automatic QC summary for all **608 subjects** in [OpenNeuro ds005385 v1.0.3](https://openneuro.org/datasets/ds005385/versions/1.0.3), generated with QC logic version `screening-2.2`.

> **CSV equivalent:** [`summary_all_subjects.csv`](../summary_all_subjects.csv) is identical to the `Subjects` sheet.

**Contents**

1. [Sheets at a glance](#1-sheets-at-a-glance)
2. [`Overview`](#2-overview)
3. [`Subjects` (33 columns)](#3-subjects-33-columns)
4. [`Recordings` (35 columns)](#4-recordings-35-columns)
5. [`Recording_Ranking` (18 columns)](#5-recording_ranking-18-columns)
6. [`Issues` (10 columns)](#6-issues-10-columns)
7. [`Legend`](#7-legend)
8. [Reading tips](#8-reading-tips)

---

## 1. Sheets at a glance

| Sheet | Rows × columns | One row is… |
|---|---|---|
| `Overview` | dashboard | – (all numbers are live formulas on `Subjects`) |
| `Subjects` | 608 × 33 | a subject |
| `Recordings` | 2,276 × 35 | an EEG recording |
| `Recording_Ranking` | 2,276 × 18 | a recording, ranked by quality score |
| `Issues` | 4,184 × 10 | a finding (FAIL / FLAG / MISSING) or a PASS note |
| `Legend` | – | a definition |

In all data sheets the rows are sorted by subject number. Cells of verdicts, statuses and labels are coloured, the header row is frozen and filters are on.

> ℹ️ `qc_eeg_subject.py` writes the sheets `Subjects`, `Recordings`, `Issues` and `Legend`. `Overview` and `Recording_Ranking` were added on top of that output.

## 2. `Overview`

A one-look summary. Every number is a formula on the `Subjects` sheet, so the page updates when the data change. It shows:

* data coverage (how many of the 608 subjects were analysed),
* status and the reasons for rejection,
* verdicts and the usability label,
* statistics of the recordings,
* demographics and age groups,
* the most frequent problems,
* bad critical channels,
* signal metrics,
* a few plain-language conclusions,
* **8 charts**.

## 3. `Subjects` (33 columns)

One row per subject. **Rejected** subjects (age above 60 or late triggers) only have the basic columns (status, reason, sex, age, handedness); they were not downloaded and not QC-analysed.

| Column | Meaning |
|---|---|
| `subject` | subject ID, `sub-001` … `sub-608` |
| `status` | `COMPLETED` = analysed · `REJECTED` = rejected before the download (see `rejection_reasons`) · others (`NO_DATA`, `ERROR`, `INTERRUPTED`) are possible when a run is not finished |
| `qc_verdict` | worst verdict of the analysed recordings: `PASS` / `FLAG` / `FAIL` (empty for rejected subjects) |
| `screening_label` | subject label: the most demanding label of its recordings, or `PARTIALLY_USABLE` when some recordings are usable and others rejected |
| `headline` | one-line summary of the subject |
| `rejection_reasons` | for rejected subjects, e.g. `Age above 60 (67)` or `Late triggers (late_ses1=1901, late_ses2=n/a)` |
| `sex`, `age`, `handedness` | from the participant table |
| `sessions_analysed` | `ses-1` or `ses-1, ses-2` |
| `recordings_expected` | 4 (one session) or 8 (two sessions) |
| `PASS`, `FLAG`, `FAIL`, `MISSING` | number of recordings with that verdict |
| `usable_recordings` | recordings whose label is not `REJECT` |
| `USABLE`, `USABLE_WITH_FLAGS`, `USABLE_NEEDS_INTERPOLATION`, `REJECT` | number of recordings with that label |
| `bad_channels (count of recordings)` | each channel flagged bad and in how many recordings, e.g. `F5 x4, F6 x4` |
| `critical_bad_channels` | critical channels (Cz, Fz, Pz, O1, O2) that were bad in at least one recording |
| `reason_codes` | all reason codes found in the recordings of the subject |
| `alpha_ratio_EC_EO_median` | median Eyes Closed / Eyes Open alpha power ratio (O1, O2, Oz); values above about 1.5 mean clear alpha blocking |
| `line50_db_median` | median 50 Hz line noise in dB above the neighbouring frequencies |
| `duration_s` | seconds the script needed for this subject (including download and clean-up) |
| `started`, `finished` | time stamps of the run |
| `data_deleted_after_qc`, `mb_freed` | whether the downloaded data were deleted after the analysis, and how many MB were freed |
| `retry_needed` | `True` if the download was incomplete and the subject will be analysed again |
| `qc_logic_version` | QC logic version used (`screening-2.2`) |
| `folder` | results folder on the machine that ran the script |

## 4. `Recordings` (35 columns)

One row per recording. A subject has four recordings per session: **Eyes Closed / Eyes Open × pre / post** the cognitive task.

| Column | Meaning |
|---|---|
| `subject`, `session`, `task`, `acq` | which recording: `ses-1` / `ses-2`, `EyesClosed` / `EyesOpen`, `pre` / `post` |
| `overall` | verdict of the recording: `PASS`, `FLAG`, `FAIL` (or `MISSING`) |
| `label` | `USABLE`, `USABLE_WITH_FLAGS`, `USABLE_NEEDS_INTERPOLATION` or `REJECT` |
| `codes` | all reason codes of the recording |
| `verdict_header`, `verdict_integrity`, `verdict_amplitude`, `verdict_spectral`, `verdict_consistency` | verdict of each of the five layers (`N/A` if not available) |
| `duration_s` | duration of the recording in seconds |
| `n_bad_channels`, `bad_channels`, `bad_detail` | number and names of bad channels, and why (`F3 (noisy+uncorrelated)`) |
| `n_critical_bad`, `critical_bad_channels` | bad channels among Cz, Fz, Pz, O1, O2 |
| `n_uncorrelated`, `uncorrelated_channels` | channels whose correlation with the median of all channels is below the threshold (|r| < 0.20; 0.18 for O1, O2, Oz) |
| `interpolation_needed` | `True` if 1–6 bad channels (at most one critical) can be interpolated |
| `median_spatial_corr` | median over channels of the (median) |r| with the median of all channels |
| `frozen_s` | seconds in which all channels were frozen at the same time |
| `typical_amp_uv` | typical amplitude in µV: median over channels of the 1-second STD of the baseline-corrected signal |
| `median_kurtosis` | median excess kurtosis of the channels (baseline-corrected signal) |
| `alpha_peak_hz` | alpha peak frequency (8–12 Hz), mean over O1, O2, Oz |
| `alpha_prom_db` | how much the alpha peak stands out over its neighbouring bands, in dB (median over O1, O2, Oz) |
| `alpha_ratio_EC_EO` | Eyes Closed / Eyes Open alpha power of the pair (same session and pre/post); the same value is shown on both recordings of the pair |
| `alpha_non_responder_possible` | `True` if the alpha peak is weak **and** the alpha ratio is weak (only a flag; many healthy people are alpha non-responders) |
| `line50_db`, `line100_db`, `line150_db` | mains noise at 50, 100 and 150 Hz in dB above the neighbouring frequencies (highest of O1, O2, Oz) |
| `notch_suggested` | `True` if `line50_db` is above 10 dB (a notch filter is advisable in preprocessing) |
| `trigger_events` | number of events on the trigger channel (the codes are not interpreted) |
| `load_error` | error text if the file could not be read (empty otherwise) |

## 5. `Recording_Ranking` (18 columns)

The same 2,276 recordings as in `Recordings`, but **independent of the subject and ordered by a quality score from 0 to 100** (best first).

* The score is built from **seven signal criteria**: bad channels, critical bad channels, uncorrelated channels, spatial correlation, 50 Hz noise, frozen segments and duration.
* **All `REJECT` recordings are placed last.**
* The weights and thresholds are in **blue cells** next to the table. Change them and the ranking follows.

## 6. `Issues` (10 columns)

One row per `FAIL`, `FLAG` or `MISSING` finding and per `PASS` note (for example `USABLE_NEEDS_INTERPOLATION`), worst first within the file.

| Column | Meaning |
|---|---|
| `subject`, `session`, `task`, `acq` | which recording |
| `layer` | `header`, `integrity`, `amplitude`, `spectral`, `consistency` or `load` |
| `code` | short reason code (see the [reason codes](../README.md#reason-codes)) |
| `finding` | name of the measured quantity |
| `value` | the measured value |
| `limit` | the limit it was compared with |
| `verdict` | `FAIL`, `FLAG`, `MISSING` or `PASS` (note) |

## 7. `Legend`

Definitions of the statuses, verdicts, labels and reason codes and the main thresholds. Examples of rules listed there: more than 6 bad channels, or two bad critical channels, is a `FAIL`; 50 Hz line noise above 20 dB is a `FLAG` (above 30 dB `HIGH_LINE_NOISE`, still only a flag); weak alpha is only a flag.

## 8. Reading tips

* **Which subjects can I use?** On `Subjects` filter `status = COMPLETED` and look at `screening_label` and `usable_recordings`.
* **Why was a subject not analysed?** `status = REJECTED` and read `rejection_reasons`.
* **Which recordings are the cleanest?** Open `Recording_Ranking` and take the top of the list.
* **Why was a recording rejected or flagged?** Filter `Issues` by `subject` and read `code`, `value` and `limit`.
* **What does a code or threshold mean?** See `Legend` and the [main README](../README.md#7-how-a-recording-is-screened).
* The figures and the other per-subject files are not part of the Excel file. See the [examples](../examples/sub-028/README.md) or [contact me](../README.md#17-contact-and-full-outputs) for the full outputs.
