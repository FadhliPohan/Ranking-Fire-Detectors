# Stored predictions

This folder is empty in the repository. The prediction files are too large for version control
(about 385 MB in total, most of it the custom detector's files) and are deposited separately on
Zenodo under CC BY 4.0:

**Download:** [ISI: Zenodo DOI]

The Zenodo record holds one archive, `dfire_stored_predictions.zip`, whose entries are
`predictions/<run>/<split>_dfire.jsonl`. Unzip it inside the repository's `data/` folder
(`cd data && unzip dfire_stored_predictions.zip`), which fills this folder as follows:

```
data/predictions/
├── resnet18fpn_seed42/             val_dfire.jsonl   test_dfire.jsonl
├── resnet18fpn_seed1337/           val_dfire.jsonl   test_dfire.jsonl
├── resnet18fpn_seed2024/           ...
├── yolo11n_seed42/
├── yolo11n_seed1337/
├── yolo11n_seed2024/
├── yolo11s_seed42/
├── yolo11s_seed1337/
├── yolo11s_seed2024/
├── yolov8s_seed42/
├── yolov8s_seed1337/
├── yolov8s_seed2024/
├── yolo11n_split_resmi_seed42/     (YOLO11n retrained on the official D-Fire partition)
└── yolo11n_split_resmi_seed1337/
```

This is the default location of the notebook, so nothing needs to be configured. The folder names
are those under which the training project stored the runs; the notebook also accepts the English
run identifiers it uses internally (for example `yolo11n_official_seed42` instead of
`yolo11n_split_resmi_seed42`). To keep the files elsewhere, set `FIRE_PRED_DIR` to the folder that
contains the run folders.

| Run folder | Detector | Seed | Partition | Experiment code | Images (val / test) |
|---|---|---|---|---|---|
| `resnet18fpn_seed{42,1337,2024}` | ResNet18+FPN+FCOS | 42, 1337, 2024 | curated | E0b | 3,418 / 3,257 |
| `yolo11n_seed{42,1337,2024}` | YOLO11n | 42, 1337, 2024 | curated | E0 | 3,418 / 3,257 |
| `yolo11s_seed{42,1337,2024}` | YOLO11s | 42, 1337, 2024 | curated | E9 | 3,418 / 3,257 |
| `yolov8s_seed{42,1337,2024}` | YOLOv8s | 42, 1337, 2024 | curated | E10 | 3,418 / 3,257 |
| `yolo11n_split_resmi_seed{42,1337}` | YOLO11n | 42, 1337 | official D-Fire | E11 | 2,725 / 4,306 |

## File format

Each file is JSON Lines, one image per line, in UTF-8. The first line may be a metadata record:

```json
{"_meta": {"split": "test", "domain": "dfire", "experiment_id": "E9", "n_samples": 3257,
           "conf_threshold_used": 0.05, "inference_seconds": 23.76}}
```

Every other line describes one image:

```json
{"sample_id": "dfire/test/images/AoF06735", "source": "dfire",
 "detections": [{"bbox": [88.97, 307.84, 117.31, 355.93], "score": 0.19945,
                 "class_idx": 1, "class": "smoke"}]}
```

| Field | Meaning |
|---|---|
| `sample_id` | Image identifier, identical to `sample_id` in the manifests (`data/manifests/*.parquet`) |
| `detections` | All detections after non-maximum suppression at IoU 0.60, down to confidence 0.05; empty list if none |
| `bbox` | `[x1, y1, x2, y2]` in pixels of the original image, rounded to 0.01 |
| `score` | Confidence, rounded to five decimals |
| `class`, `class_idx` | `fire` = 0, `smoke` = 1 (this project's order, the reverse of D-Fire's own label files) |

Every image of the partition has a line, including images without detections. The notebook checks
this, checks that the confidence floor is 0.05, and checks that the experiment code in `_meta`
matches the run.
