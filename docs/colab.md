# Running on Google Colab

You do not need to own any data. The script downloads one subject, analyses it, saves the results, deletes the data and moves to the next one, so the Colab disk never fills up.

## Simple method (recommended)

Results are written to `/content/qc_output` and copied to Google Drive with `rsync`.

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

Files in `/content` are lost when the runtime disconnects, so run Cell 3 from time to time and always before the session ends. `rsync` only copies what changed, so repeated runs are fast.

### Resuming in a new session

After a disconnect, run Cell 1, copy the finished results back from Drive, then run the same command again:

```bash
!mkdir -p /content/qc_output
!rsync -avh /content/drive/MyDrive/qc_output/ /content/qc_output/
!python qc_eeg_subject.py 1 to 608
```

Finished subjects are skipped in a second and the work continues where it stopped. Then run Cell 3 again.

## Writing results directly to Drive

If you prefer that nothing depends on a final copy step, write straight to Drive. Slower for many small files, but a disconnect never loses results.

```bash
!python qc_eeg_subject.py 28 --download-method https --data-dir /content/data --out-root /content/drive/MyDrive/dortmund_vital_qc
```

Check that `sub-028/` and `summary_all_subjects.xlsx` appear in `MyDrive/dortmund_vital_qc`, then run all subjects:

```bash
!python qc_eeg_subject.py 1 to 608 --download-method https --data-dir /content/data --out-root /content/drive/MyDrive/dortmund_vital_qc
```

* Give the subject numbers on the command line; the interactive `Subjects >` question does not work in a notebook cell.
* Keep `--data-dir` on `/content/data` (temporary disk) and only `--out-root` on Drive.
* Before closing the tab you can run `drive.flush_and_unmount()` to make sure everything is written.

## Pinning dataset version 1.0.3

`--download-method https` reads the newest snapshot of the dataset. To pin version 1.0.3, use DataLad (run once before the analysis command):

```bash
!apt-get -qq install -y git-annex > /dev/null
!pip install -q datalad
!git config --global user.name "colab" && git config --global user.email "colab@example.com"
!python qc_eeg_subject.py 1 to 608 --data-dir /content/data --repo-dir /content/ds005385_repo
```

## How long does it take?

With the HTTPS download, a one-session subject (four recordings) took about 58 s (`sub-325`); a two-session subject takes roughly twice as long. With 285 accepted subjects having one session and 142 having two, all 427 accepted subjects take on the order of **10 hours**, so plan for one or more Colab sessions. That is what the resume function is for.
