# Chlamydomonas Flagella Holding Segmentation Dataset

Manually annotated flagella instance masks of **microneedle-held** *Chlamydomonas reinhardtii* (strain **CC125**) for recognition and segmentation model training and evaluation.

**Maintainer:** Lucien (`lucien-6@qq.com`)
**Affiliation:** University of Electronic Science and Technology of China (UESTC), Chengdu, China
**Release date:** 2026-08-17
**License:** [CC BY 4.0](LICENSE)
**Version:** 1.0.0

---

## Overview

*Chlamydomonas reinhardtii* is a biflagellate green alga and a standard model organism for flagellar motility, waveform mechanics, and ciliary biology. When the cell body is immobilized with a glass microneedle (or micropipette), the two flagella remain free in the focal plane and can be imaged as thin, high-curvature filaments. That holding geometry is widely used in flagellar beat analysis, but it is also a demanding case for automated segmentation: the structures occupy only a small fraction of the field of view, they frequently overlap or come into contact, and they move rapidly from frame to frame.

This repository releases a **training-oriented** dataset of grayscale microscopy frames together with **pixel-accurate, manually drawn instance masks**. Each mask distinguishes:

| Pixel value | Meaning |
| --- | --- |
| `0` | Background (medium, cell body, microneedle, debris) |
| `1` | Flagellum instance 1 |
| `2` | Flagellum instance 2 |

The two non-zero labels are **instance identifiers** for the two flagella of the same cell. They are not guaranteed to encode a biologically named identity (for example cis versus trans, or a fixed left/right assignment) across groups. For binary flagella-versus-background tasks, use `mask > 0`.

The dataset can be used for:

- semantic segmentation of flagella (`background` vs `flagella`);
- two-instance / three-class segmentation (`background`, `flagellum 1`, `flagellum 2`);
- pretraining or fine-tuning of U-Net, DeepLab, Mask R-CNN, SAM-family, or similar models;
- held-out testing of an already trained flagella segmentation model.

---

## Dataset summary

| Item | Value |
| --- | --- |
| Organism | *Chlamydomonas reinhardtii*, strain CC125 |
| Experimental state | Cell body held by a microneedle; flagella free |
| Annotation | Manual instance masks, one pair per frame |
| Groups (folders) | `01` … `16` (16 independent recording groups) |
| Image–mask pairs | **6,972** (13,944 TIFF files) |
| Spatial size | 400 × 400 pixels |
| Channels | 1 (grayscale) |
| Bit depth | 8-bit unsigned integer |
| Image intensity range | 0–255 (microscopy gray values) |
| Mask label set | `{0, 1, 2}` |
| File format | Uncompressed TIFF (`.tif`) |
| Approximate volume | 2.2 GiB |

Every image file has a matching mask. There are **no unpaired frames**.

---

## Directory layout

```text
Chlamydomonas-Flagella-Holding-Dataset/
├── 01/ … 16/          # one folder per experimental group
├── README.md          # this document
├── LICENSE            # Creative Commons Attribution 4.0
├── CITATION.cff       # machine-readable citation metadata
└── .gitignore
```

Inside each group folder, files come in pairs:

```text
<stem>.tif            # raw grayscale frame
<stem>_mask.tif       # instance mask aligned to that frame
```

Example (`01/`):

```text
01/Train_01_00000.tif
01/Train_01_00000_mask.tif
01/Train_01_00001.tif
01/Train_01_00001_mask.tif
…
```

---

## File naming conventions

Two historical naming schemes coexist. Both follow the same pairing rule: the mask name is the image stem plus `_mask`.

| Groups | Image pattern | Index range (inclusive) |
| --- | --- | --- |
| `01`–`06` | `Train_{gg}_{fffff}.tif` | `00000`–`00499` |
| `07`–`08` | `CC125-{ffff}.tif` | `0000`–`0399` |
| `09` | `CC125_07-{fff}.tif` | `001`–`400` (6 frames missing) |
| `10`–`16` | `CC125-{fff}.tif` | `001`–`400` (some frames missing) |

`{gg}` is the two-digit group id used in the `Train_*` series. `{fffff}` / `{ffff}` / `{fff}` are zero-padded frame indices. The `CC125` prefix refers to the wild-type *C. reinhardtii* strain.

A robust loader should **not** parse a single regex for the whole dataset. Discover files by suffix and pair by stem, for example:

```python
from pathlib import Path

def iter_pairs(group_dir: Path):
    for image_path in sorted(group_dir.glob("*.tif")):
        if image_path.name.endswith("_mask.tif"):
            continue
        mask_path = image_path.with_name(image_path.stem + "_mask.tif")
        if not mask_path.is_file():
            raise FileNotFoundError(mask_path)
        yield image_path, mask_path
```

---

## Inventory by group

Frames that failed visual quality control (defocus, empty field, annotation not possible, and similar) were removed. Missing indices are listed explicitly so that loaders which assume a contiguous sequence do not silently skip holes.

| Group | Pairs | Naming | Index range | Missing indices |
| --- | ---: | --- | --- | --- |
| 01 | 500 | `Train_01_*` | 0–499 | none |
| 02 | 500 | `Train_02_*` | 0–499 | none |
| 03 | 500 | `Train_03_*` | 0–499 | none |
| 04 | 500 | `Train_04_*` | 0–499 | none |
| 05 | 500 | `Train_05_*` | 0–499 | none |
| 06 | 500 | `Train_06_*` | 0–499 | none |
| 07 | 400 | `CC125-*` (4 digits) | 0–399 | none |
| 08 | 400 | `CC125-*` (4 digits) | 0–399 | none |
| 09 | 394 | `CC125_07-*` | 1–400 | 116, 204, 218, 220, 252, 322 |
| 10 | 399 | `CC125-*` (3 digits) | 1–400 | 263 |
| 11 | 399 | `CC125-*` (3 digits) | 1–400 | 371 |
| 12 | 399 | `CC125-*` (3 digits) | 1–400 | 245 |
| 13 | 400 | `CC125-*` (3 digits) | 1–400 | none |
| 14 | 398 | `CC125-*` (3 digits) | 1–400 | 290, 337 |
| 15 | 399 | `CC125-*` (3 digits) | 1–400 | 210 |
| 16 | 384 | `CC125-*` (3 digits) | 1–400 | 267, 355, 356, 368, 373, 374, 378, 381, 382, 386–388, 395–397, 399 |
| **Total** | **6,972** | | | |

Each folder is one experimental group (typically one holding session / one cell recording). Consecutive frames inside a folder are temporally correlated.

---

## Image and mask specifications

### Microscopy images

- Single-channel 8-bit TIFF, 400 × 400.
- Intensities occupy the full 8-bit range and should be treated as raw gray values.
- The cell body and the holding microneedle are usually visible; only the two flagella are annotated.

### Masks

- Single-channel 8-bit TIFF, same height and width as the corresponding image.
- Integer labels `{0, 1, 2}` as defined above.
- Flagella are thin structures: in a typical frame, each instance occupies on the order of 10<sup>2</sup>–10<sup>3</sup> pixels out of 160,000.
- Labels are mutually exclusive: a pixel belongs to at most one instance.

### Recommended label mappings

```text
Binary semantic segmentation
    y = (mask > 0).astype(uint8)          # 0 = background, 1 = flagella

Three-class semantic segmentation
    y = mask.astype(int64)                # classes {0, 1, 2}

Instance segmentation
    instance_ids = {1, 2}                 # ignore 0
```

If a downstream tool expects `{0, 255}` binary masks, convert with `(mask > 0) * 255`. Do **not** linearly stretch the `{0, 1, 2}` labels; that would mix the two instances.

---

## How to use

### 1. Clone the repository

The full dataset is about **2.2 GiB**. A complete clone is required for training.

```bash
git clone https://github.com/Lucien-6/Chlamydomonas-Flagella-Holding-Dataset.git
cd Chlamydomonas-Flagella-Holding-Dataset
```

The repository contains this documentation together with the 16 group folders `01/` … `16/`. GitHub also provides a Source ZIP from the repository page; that archive is equivalent to `git clone`.

### 2. Python environment

```bash
pip install numpy tifffile
```

Pillow, imageio, scikit-image, and OpenCV can also read these TIFFs. `tifffile` is recommended for scientific TIFF files.

### 3. Load one pair

```python
from pathlib import Path

import numpy as np
import tifffile

root = Path("Chlamydomonas-Flagella-Holding-Dataset")
image = tifffile.imread(root / "01" / "Train_01_00000.tif")
mask = tifffile.imread(root / "01" / "Train_01_00000_mask.tif")

assert image.shape == (400, 400)
assert mask.shape == image.shape
assert image.dtype == np.uint8
assert mask.dtype == np.uint8
assert set(np.unique(mask)).issubset({0, 1, 2})

flagella = mask > 0
flagellum_1 = mask == 1
flagellum_2 = mask == 2
```

### 4. Build a file list for training

```python
from pathlib import Path

import tifffile


def list_pairs(root: Path):
    pairs = []
    for group_dir in sorted(p for p in root.iterdir() if p.is_dir() and p.name.isdigit()):
        for image_path in sorted(group_dir.glob("*.tif")):
            if image_path.name.endswith("_mask.tif"):
                continue
            mask_path = image_path.with_name(image_path.stem + "_mask.tif")
            pairs.append((image_path, mask_path))
    return pairs


def load_pair(image_path, mask_path):
    return tifffile.imread(image_path), tifffile.imread(mask_path)
```

### 5. Suggested group-level data split

Do **not** draw a random train/validation/test split over individual frames. Frames that belong to the same group come from the same holding experiment and are strongly correlated in appearance, pose, and beat phase. A frame-level shuffle leaks information from training into validation.

Split **by group id**. One example protocol:

| Split | Groups | Approximate pairs |
| --- | --- | ---: |
| Training | `01`–`12` | 5,791 |
| Validation | `13`–`14` | 798 |
| Test | `15`–`16` | 783 |

Any other partition is acceptable as long as every group is assigned to exactly one split. Report the group lists together with quantitative results so that splits are reproducible.

### 6. Training notes

- **Class imbalance.** Flagella cover well below 2% of the pixels. Use loss functions that are robust to imbalance (for example Dice, Tversky, or class-weighted cross-entropy), and report both pixel metrics and region metrics (Dice / IoU on the flagella class, not only overall accuracy).
- **Thin structures.** Boundary errors dominate the Dice score. Consider boundary losses, skeleton-aware metrics, or a small morphological evaluation tolerance if the scientific goal is waveform extraction rather than pixel-perfect overlap.
- **Two-instance tasks.** If the model must separate the two flagella, keep labels `{1, 2}`. Hungarian matching of predicted instances to ground-truth instances is appropriate at evaluation time because the numeric ids are not a globally consistent biological identity.
- **Augmentation.** Geometric transforms (flip, rotation, modest elastic warp, crop/resize) are useful. Photometric jitter should stay mild so that the microneedle and cell-body contrast remain realistic.
- **Input normalization.** Per-image z-score or min–max normalization both work. Do not assume a fixed microscope gain across groups.

### 7. Evaluation checklist

For a binary flagella model, report at least:

- Dice / F1 and IoU on the foreground class;
- precision and recall on the foreground class;
- optionally Hausdorff distance or mean contour error if masks will be used to reconstruct waveforms.

For a two-instance model, report instance-averaged AP or matched Dice after optimal assignment of the two predicted filaments.

---

## Intended use and limitations

**Intended use**

- academic research on microbial flagellar imaging;
- development and benchmarking of segmentation models;
- teaching examples for biomedical image analysis.

**Limitations**

- Pixel size (µm/pixel), frame rate, objective, and illumination modality are not included in this release.
- Labels `1` and `2` identify the two flagella inside a frame; they should not be treated as a dataset-wide cis/trans annotation.
- Missing indices were removed during quality control; the remaining sequences are not guaranteed to be uniformly sampled in time.
- The dataset covers the **microneedle-held** configuration only. Models trained here may not transfer directly to freely swimming cells or to other imaging modalities without additional fine-tuning.

---

## Citation

If this dataset contributes to a publication, software, or trained model, please cite the GitHub repository (and this version tag):

```bibtex
@misc{lucien2026flagella,
  author       = {Lucien},
  title        = {Chlamydomonas Flagella Holding Segmentation Dataset},
  year         = {2026},
  publisher    = {GitHub},
  version      = {1.0.0},
  howpublished = {\url{https://github.com/Lucien-6/Chlamydomonas-Flagella-Holding-Dataset}},
  note         = {University of Electronic Science and Technology of China (UESTC)}
}
```

GitHub users can also use the **Cite this repository** button, which reads `CITATION.cff`.

---

## License

This dataset is released under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0). You may copy, redistribute, and adapt the material for any purpose, including commercial model training, provided that appropriate credit is given, a link to the license is supplied, and any changes are indicated.

---

## Contact

- **Author / maintainer:** Lucien
- **Email:** lucien-6@qq.com
- **Affiliation:** University of Electronic Science and Technology of China (UESTC)

Issues and questions can be opened on the GitHub repository.

---

## Acknowledgements

Manual annotation of thin flagellar filaments is labour-intensive. This release is intended to make that effort reusable by the flagellar motility and bioimage-analysis communities.
