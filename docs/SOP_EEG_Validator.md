### Dortmund Vital EEG Validator | Large-Scale Automated QC Pipeline
*Python, MNE, OpenNeuro ds005385*

- **Challenge & Scale:** Designed an end-to-end quality screening pipeline for resting-state EEG data from 608 participants (>2,200 sessions), addressing large-scale data processing and limited local computing resources.
- **Engineering & Domain Logic:** Implemented subject-wise streaming with memory cleanup, resumable batch execution, and multi-stage QC heuristics using spatial channel correlations and power spectral density (PSD) profiles to distinguish artifacts from unusable signals.
- **Impact & Reproducibility:** Automated the generation of 15 diagnostic visual reports per subject and cohort-level QC summaries (`summary_all_subjects.xlsx`), with cryptographic run manifests to support reproducible downstream analysis.
