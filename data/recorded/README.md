# Recorded GPU measurements (display only)

The files in this folder are measurements taken on the training machine. They cannot be
recomputed on a CPU, and the notebook does not try to: it reads them to display Table 2 and to use
the parameter counts in Table 7.

| File | Content | Original name and location |
|---|---|---|
| `efficiency_summary_server.csv` | Efficiency harness of the training pipeline: parameters, model size, mean and median latency, FPS and peak memory for every run of the custom detector family. The rows `E0b` (seeds 42, 1337, 2024) are the ResNet18+FPN+FCOS detector of the paper; the other rows belong to experiments of the original project that this paper does not use. | same name; `journalv2/apps/hasil/sumber_versi2/` |
| `yolo_efficiency_benchmark.csv` | The same measurements for YOLO11n, YOLO11s and YOLOv8s (seed 42), taken with the same protocol, plus GFLOPs. | `efisiensi_yolo_versi2.csv`, same folder |
| `ultralytics_model_summary.txt` | The model summary lines Ultralytics printed at training time (layers, parameters, GFLOPs under the vendor's convention). | same name, same folder |

**Measurement protocol.** NVIDIA GeForce RTX 4050 Laptop GPU (6 GB) of the training machine
(Linux, PyTorch 2.5.1, CUDA 12.4); batch size 1, input 640 × 640, 20 warm-up iterations followed
by 100 timed iterations, idle GPU. The YOLO file records its precision setting (`amp_dtype`,
bfloat16) and its protocol columns explicitly. Image decoding, letterboxing, non-maximum
suppression and transport are excluded for all four detectors.

**GFLOPs.** The YOLO GFLOPs in `yolo_efficiency_benchmark.csv` count one multiply-accumulate as one
operation. The Ultralytics summary counts it as two, so it prints roughly double (6.5, 21.7 and
28.6). GFLOPs for the custom detector were not measured, because the profiling dependency was not
installed in the training environment.

The files are copied unchanged, apart from the renaming noted above.
