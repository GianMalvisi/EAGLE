# EAGLE: Egocentric Automated Gaze-driven Lifting of Entities for 3D Segmentation

**[Gianluca Malvisi](https://github.com/GianMalvisi), Simone Melzi, Alessandro Riva**

University of Milano-Bicocca — Department of Computer Science, Systematics and Communication

**Published at [STAG 2026](https://www.stag-conference.org/) (Smart Tools and Applications in Graphics)**

[[Paper]](https://diglib.eg.org/handle/10.2312/stag20261xxx) [[BibTeX]](#citation)

![EAGLE Teaser](assets/teaser.png)

---

## Overview

EAGLE is a gaze-driven pipeline for automatic 3D object segmentation from egocentric recordings captured with [Project Aria](https://www.projectaria.com/) glasses. The system extracts isolated 3D sub-meshes of objects the user is looking at, without manual annotation.

The pipeline operates through two parallel streams:

1. **2D Segmentation** — Eye-tracking fixations are detected via the I-DT algorithm and used as point prompts for SAM 2, which propagates masks bidirectionally across the video.
2. **3D Reconstruction** — RGB frames are processed through COLMAP (SfM) and Nerfstudio (Nerfacto) to produce a dense 3D mesh via Poisson reconstruction.

The two streams converge in a **mask projection** phase: 2D masks are projected onto the 3D mesh vertices using the estimated camera poses, and a **majority voting** filter (66.6% threshold) resolves projective ambiguities (tunnel effect), yielding the final isolated sub-mesh.

Quantitative evaluation on the ([https://github.com/facebookresearch/digital-twin-catalog](https://explorer.projectaria.com/dtc)) (15 scenes) shows that EAGLE outperforms SA3D across all spatial metrics.

## Installation

### Prerequisites

- Python 3.10+
- CUDA-capable GPU (required for SAM 2 and Nerfstudio)

### Setup

```bash
git clone https://github.com/GianMalvisi/EAGLE.git
cd EAGLE
python -m venv venv
source venv/bin/activate        # Linux
# .\venv\Scripts\activate       # Windows

pip install "projectaria-tools[all]"
pip install -e ".[nerfstudio,sam2]"
```

## Project Structure

```
EAGLE/
├── eagle/              — Python package (installable)
│   ├── loaders.py      — VRS frame extraction and PyTorch datasets
│   ├── sync.py         — Multi-stream temporal synchronization
│   ├── gaze.py         — Gaze projection, I-DT fixation detection
│   ├── segmentation.py — SAM 2 inference and palettized mask I/O
│   ├── preprocessing.py— Distortion cropping and COLMAP integration
│   ├── nerfstudio.py   — Nerfacto training and mesh export helpers
│   ├── projection.py   — Mask-to-mesh projection and majority voting
│   ├── mesh.py         — PLY I/O and sub-mesh extraction
│   └── utils.py        — Recording inspection and file discovery
├── notebooks/
│   ├── end_to_end_pipeline.ipynb  — Full EAGLE pipeline demo
│   ├── sa3d_comparison.ipynb      — Quantitative comparison (Table 1)
│   └── exploration/               — Development notebooks (data loading, visualization, etc.)
├── data/
│   └── processed/dtc/  — Per-scene evaluation metrics (Table 1)
├── pyproject.toml
├── LICENSE
└── README.md
```

## Usage

```python
from eagle.loaders import extract_frames
from eagle.gaze import align_gaze_to_rgb, detect_fixations_idt, fixations_to_prompts
from eagle.segmentation import run_multi_prompt, merge_masks, export_masks
from eagle.preprocessing import apply_crop, run_colmap
from eagle.nerfstudio import load_pipeline, export_mesh
from eagle.mesh import load_ply, extract_submesh
from eagle.projection import project_masks_onto_mesh, majority_vote
```

See [`notebooks/end_to_end_pipeline.ipynb`](notebooks/end_to_end_pipeline.ipynb) for the complete pipeline walkthrough.

## Evaluation

Quantitative results on the DTC dataset (15 scenes), comparing EAGLE against SA3D:

| Method | Chamfer (mm) ↓ | Precision (%) ↑ | Recall (%) ↑ | F-Score (%) ↑ | IoU (%) ↑ |
|--------|:-:|:-:|:-:|:-:|:-:|
| **EAGLE** | **10.2** | **90.34** | **83.34** | **86.45** | **51.84** |
| SA3D | 19.2 | 71.99 | 63.33 | 66.94 | 30.87 |

Per-scene results are available in `data/processed/dtc/`.

## Citation

If you use this code in your research, please cite:

```bibtex
@inproceedings{malvisi2026eagle,
    title     = {{EAGLE}: Egocentric Automated Gaze-driven Lifting of Entities for {3D} Segmentation},
    author    = {Malvisi, Gianluca and Melzi, Simone and Riva, Alessandro},
    booktitle = {Smart Tools and Applications in Graphics (STAG)},
    year      = {2026},
    publisher = {Eurographics Association},
    doi       = {10.2312/stag.2026.xxx}
}
```

## Acknowledgements

We thank Meta for providing the Project Aria glasses through the [Aria Research Kit](https://www.projectaria.com/).

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
