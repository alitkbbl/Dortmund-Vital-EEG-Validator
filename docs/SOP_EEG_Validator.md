Dortmund Vital EEG Validator | Large-Scale Automated QC Pipeline [Python, MNE, OpenNeuro ds005385]

    Challenge & Scale: Designed an end-to-end quality screening architecture to evaluate resting-state EEG records from 608 participants (>2,200 sessions), overcoming raw data scale and limited local compute constraints.
    Engineering & Domain Logic: Implemented single-subject streaming with immediate memory cleanup, resumable batch execution, and a multi-stage triage heuristic (distinguishing non-catastrophic artifacts from unusable signal loss via spatial channel correlations and PSD profiles).
    Impact & Reproducibility: Automated generation of 15 diagnostic visual reports per subject and unified cohort-level QC metrics (summary_all_subjects.xlsx) with cryptographic run manifests for 100% reproducible downstream modeling.
