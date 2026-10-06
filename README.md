# Ranking Fire Detectors by mAP50 Versus Alarm Rate: A Four-Detector, Three-Seed Stability Audit on D-Fire

Analysis code and data for the paper of the same title, submitted to *Signal, Image and Video
Processing* (Springer), 2026.

**Authors**

* Muhammad Fadhli Dzil Ikram Pohan (corresponding author), ORCID
  [0009-0008-1748-8715](https://orcid.org/0009-0008-1748-8715), muhammadfadly.mfd@gmail.com
* Samsuryadi

Master of Computer Science, Faculty of Computer Science, Sriwijaya University, Palembang, Indonesia.

**Repository:** https://github.com/FadhliPohan/Ranking-Fire-Detectors

The paper asks whether ranking fire and smoke detectors by mAP50 agrees with ranking them by how
often they raise an alarm on images that annotators verified as containing no fire. Four detectors
(YOLO11n, YOLO11s, YOLOv8s and a custom ResNet18+FPN+FCOS model), each trained with three seeds,
are compared on a frozen D-Fire test partition with 1,433 verified negatives.

This repository holds one Jupyter notebook that recomputes every table, figure and in-text number
of the paper from stored per-image predictions, and then checks each of them against the values in
the paper and in the original result tables.

## What is here

```
ranking_fire_detectors.ipynb   the complete analysis (one notebook, no other code)
requirements.txt               packages for the analysis
requirements-training.txt      extra packages for the optional training appendix
CITATION.cff, LICENSE          citation metadata and the MIT licence of the code
data/
  manifests/                   curated D-Fire split manifests (ground-truth boxes) and their hashes
  recorded/                    GPU efficiency measurements shown in Table 2 (see its README)
  predictions/                 empty: download the stored predictions here (see its README)
outputs/                       created by the notebook: tables, figures (PNG and PDF), verification report
```

The manifests are Parquet files with one row per image: the image path in the D-Fire release, its
size, its fire and smoke boxes as JSON (normalised centre and size), the release folder it came from
and content hashes. `MANIFEST_HASHES.json` records an order-independent SHA-256 of each manifest's
content; the notebook recomputes it, and stops if the frozen test manifest
(`6246e1bcca7947392097d8460a33e4e5e5823c8ff3c20ebc6d852d3c1af56ae9`) does not match.

## Installation

Python 3.10 or newer (tested with 3.11).

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Data

* **D-Fire** (images and labels) is published by its authors at
  https://github.com/gaiasd/DFireDataset. The analysis does **not** need the images: the boxes it
  uses are in `data/manifests/`. The images are needed only to retrain a detector (Appendix A).
* **Stored predictions** of the fourteen training runs (about 385 MB of JSON Lines files) are
  deposited on Zenodo under CC BY 4.0: https://doi.org/10.5281/zenodo.23166787. The record holds one archive,
  `dfire_stored_predictions.zip`, whose entries are `predictions/<run>/<split>_dfire.jsonl`, with
  one folder per run named as the runs were stored by the training project (`resnet18fpn_seed42`,
  `yolo11n_seed42`, `yolo11s_seed42`, `yolov8s_seed42`, `yolo11n_split_resmi_seed42`, and so on).
  Unzip it inside the repository's `data/` folder:

  ```bash
  cd data
  unzip dfire_stored_predictions.zip
  ```

  This gives `data/predictions/<run>/val_dfire.jsonl` and `data/predictions/<run>/test_dfire.jsonl`
  for all fourteen runs, which is where the notebook looks by default; nothing else needs to be
  set. The file format is described in [`data/predictions/README.md`](data/predictions/README.md).

## How to run

```bash
jupyter lab ranking_fire_detectors.ipynb
```

and run all cells (*Run > Run All Cells*). Nothing needs to be edited. If the predictions are stored
somewhere else, set an environment variable before starting Jupyter:

| Variable | Default | Meaning |
|---|---|---|
| `FIRE_DATA_DIR` | `data` | folder with `manifests/`, `recorded/` and `predictions/` |
| `FIRE_PRED_DIR` | `<FIRE_DATA_DIR>/predictions` | folder with one sub-folder per run (several folders may be separated by `;` on Windows or `:` elsewhere) |
| `FIRE_OUTPUT_DIR` | `outputs` | where tables, figures and the report are written |

The notebook can also run headless:

```bash
jupyter nbconvert --to notebook --execute ranking_fire_detectors.ipynb --output executed.ipynb
```

**Runtime:** about one minute on a laptop CPU, with a peak memory of about 1.2 GB. No GPU is needed.

**Result:** the last section prints a verification summary and ends with `OVERALL: PASS` when every
check passes. On the reference machine all 366 checks pass: 19 original result tables compared
cell by cell, the 248 values printed in Tables 1 to 8, 80 in-text numbers, the five-value
reproduction check, and the PNG and PDF files of the seven figures. The numerical results were
also confirmed with numpy 1.26, pandas 2.2 and matplotlib 3.8.

The figures are written as 600-dpi PNG and as vector PDF with embedded TrueType fonts, at their
final printed size: Fig. 1 is one column wide (3.00 in), Fig. 2 spans 4.55 in, and lettering is
9 pt for axis labels and panel letters and 8 pt for tick labels and legends. They carry no title
inside the image, as the journal's artwork rules require; the explanation is in the caption.
The figures of the original analysis had such titles, so the notebook's comparison of PNG hashes
with those files is listed as information only.

## Where each paper item is produced

| Paper item | Notebook section | Output |
|---|---|---|
| Table 1, dataset composition | 4 | `outputs/tables/Table1_dataset.csv` |
| Table 2, parameters / GFLOPs / size / latency / FPS | 11 (display only) | `outputs/tables/Table2_capacity.csv` |
| Table 3, main results (mean ± SD, three seeds) | 6 | `outputs/tables/Table3_main_results.csv` |
| Table 4, false alarms at fixed recall | 7 | `outputs/tables/Table4_far_at_fixed_recall.csv` |
| Table 5, paired bootstrap comparisons | 8 | `outputs/tables/Table5_paired_comparisons.csv` |
| Table 6, permutation floor by number of detectors | 9 | `outputs/tables/Table6_permutation_floor.csv` |
| Table 7, predictors of the alarm ordering | 11 | `outputs/tables/Table7_predictors.csv` |
| Table 8, expected cost and sweep-floor flag | 10 | `outputs/tables/Table8_expected_cost.csv` |
| Fig. 1, mAP50 against false-alarm rate | 6 | `outputs/figures/Fig1.png`, `Fig1.pdf` |
| Fig. 2, false-alarm rate against accuracy and capacity | 11 | `outputs/figures/Fig2.png`, `Fig2.pdf` |
| In-text numbers (ρ, 81-draw distribution, exact *p*, split influence, ...) | 9, 10, 12, 14 | `outputs/tables/intext_numbers.csv` |
| Extended analysis (longer version of the paper: 13 tables, 7 figures) | 13 | `outputs/tables/ext_*.csv`, `outputs/figures/ExtFig*.png` and `.pdf` |
| Verification of everything above | 14 | `outputs/verification_report.csv` |

## What is recomputed, and what is not

Everything that depends on the predictions is recomputed: thresholds selected on validation,
AP / mAP50 / mAP50-95, precision, recall, F1, false-alarm rates, fixed-recall comparisons, bootstrap
intervals and paired tests, rank correlations and their exact permutation nulls, the 81 single-seed
combinations, expected cost and the split-influence control. The bootstrap uses a seeded generator
(`numpy.random.default_rng(42)`), so its intervals are reproduced exactly.

What **cannot** be recomputed on a CPU are the efficiency figures of Table 2: latency, FPS and peak
GPU memory were measured on an RTX 4050 Laptop GPU, and parameter counts and GFLOPs were recorded
by the same measurement runs. The notebook reads them from `data/recorded/` and displays them as
measured; their provenance and protocol are in [`data/recorded/README.md`](data/recorded/README.md).

## Optional appendices

* **Appendix A** retrains YOLO11n, YOLO11s and YOLOv8s with Ultralytics under the locked settings
  (mosaic, mixup and copy-paste off) and exports predictions in the format the analysis reads. It
  repeats the training call as it was run: batch 8 with Ultralytics' default nominal batch size
  `nbs=64`, so gradients are accumulated over 8 steps (effective batch 64). The custom
  ResNet18+FPN+FCOS detector was trained with batch 8 and 2-step accumulation (effective batch 16). It
  needs a CUDA GPU, the D-Fire images (`DFIRE_ROOT`) and `pip install -r requirements-training.txt`.
  It is off by default (`RUN_TRAINING = False`).
* **Appendix B** contains a compact port of the custom ResNet18+FPN+FCOS detector (model, target
  assignment, loss and post-processing) with a CPU smoke test that runs one forward and backward
  pass and checks the parameter count (12,170,317) against the trained detector. It is off by
  default (`RUN_SMOKE_TEST = False`). The Markdown above it lists what the port simplifies.

Ultralytics is distributed under AGPL-3.0; it is used only in Appendix A and is not needed for the
analysis.

## How to cite

If you use this code or the stored predictions, please cite the paper:

> Muhammad Fadhli Dzil Ikram Pohan and Samsuryadi. Ranking Fire Detectors by mAP50 Versus Alarm Rate:
> A Four-Detector, Three-Seed Stability Audit on D-Fire. Manuscript submitted to *Signal, Image
> and Video Processing*, 2026. DOI: [ISI: DOI of the paper once published].

and, for the code and data themselves:

> Muhammad Fadhli Dzil Ikram Pohan and Samsuryadi. Ranking-Fire-Detectors: analysis code and stored
> predictions, 2026. Code: https://github.com/FadhliPohan/Ranking-Fire-Detectors.
> Predictions: Zenodo, https://doi.org/10.5281/zenodo.23166787.

Citation metadata for the repository is also given in [`CITATION.cff`](CITATION.cff).

## License

* **Code** (the notebook and everything else in this repository): MIT License, see [`LICENSE`](LICENSE).
* **Stored predictions** on Zenodo: Creative Commons Attribution 4.0 International (CC BY 4.0).
* **D-Fire images and labels** keep the licence set by their authors; see
  https://github.com/gaiasd/DFireDataset. The manifests in `data/manifests/` contain box
  coordinates derived from those labels.
