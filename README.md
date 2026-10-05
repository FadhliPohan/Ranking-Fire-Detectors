# Ranking Fire Detectors by mAP50 Versus Alarm Rate

Code and data for the paper

> **Ranking Fire Detectors by mAP50 Versus Alarm Rate: A Four-Detector, Three-Seed Stability Audit on D-Fire.**
> Submitted to *Signal, Image and Video Processing* (Springer).
> Citation: [ISI: citation once published]

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
data/
  manifests/                   curated D-Fire split manifests (ground-truth boxes) and their hashes
  recorded/                    GPU efficiency measurements shown in Table 2 (see its README)
  predictions/                 empty: download the stored predictions here (see its README)
outputs/                       created by the notebook: tables, figures, verification report
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
* **Stored predictions** of the fourteen training runs (about 385 MB of JSON Lines files):
  [ISI: Zenodo DOI]. Download them and place one folder per run in `data/predictions/`, as
  described in [`data/predictions/README.md`](data/predictions/README.md). Expected layout:

  ```
  data/predictions/resnet18fpn_seed42/val_dfire.jsonl
  data/predictions/resnet18fpn_seed42/test_dfire.jsonl
  ...
  data/predictions/yolo11n_split_resmi_seed1337/test_dfire.jsonl
  ```

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
check passes. On the reference machine all 357 checks pass: 19 original result tables compared
cell by cell, the 248 values printed in Tables 1 to 8, 78 in-text numbers, the five-value
reproduction check, and the presence of the seven figures, which are also byte-identical to the
published ones. The numerical results were also confirmed
with numpy 1.26, pandas 2.2 and matplotlib 3.8; with other matplotlib versions the figures show
the same data but are not byte-identical, which the notebook reports as information, not failure.

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
| Fig. 1, mAP50 against false-alarm rate | 6 | `outputs/figures/Fig1.png` |
| Fig. 2, false-alarm rate against accuracy and capacity | 11 | `outputs/figures/Fig2.png` |
| In-text numbers (ρ, 81-draw distribution, exact *p*, split influence, ...) | 9, 10, 12, 14 | `outputs/tables/intext_numbers.csv` |
| Extended analysis (longer version of the paper: 13 tables, 7 figures) | 13 | `outputs/tables/ext_*.csv`, `outputs/figures/ExtFig*.png` |
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
  needs a CUDA GPU, the D-Fire images (`DFIRE_ROOT`) and `pip install -r requirements-training.txt`.
  It is off by default (`RUN_TRAINING = False`).
* **Appendix B** contains a compact port of the custom ResNet18+FPN+FCOS detector (model, target
  assignment, loss and post-processing) with a CPU smoke test that runs one forward and backward
  pass and checks the parameter count (12,170,317) against the trained detector. It is off by
  default (`RUN_SMOKE_TEST = False`). The Markdown above it lists what the port simplifies.

Ultralytics is distributed under AGPL-3.0; it is used only in Appendix A and is not needed for the
analysis.

## License

[ISI: license]

The D-Fire dataset is subject to the terms of its authors; see their repository.
