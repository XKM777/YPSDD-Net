# YPSDD-Net: Multiscale Slender Defect Detection in Aeroengine Preform Manufacturing

**Status:** Private reviewer release — this repository contains a reviewer-only project skeleton and a reviewer checkpoint.  

---

## One-line summary

YPSDD-Net is a dual-branch detection framework built on a **YOLOv8** backbone that introduces a **Slender Mamba module**, a **Category-Aware Head (CAHead)** and a detachable **Few-Shot Adaptation Branch (FAB)** to reliably detect slender, irregular defects in carbon-fiber yarns and woven preforms.

---

## Architecture

- **Backbone (baseline):** YOLOv8 (C2f blocks) as the feature extractor and base detection pipeline.  
- **Slender Mamba Module (SMM):** a hybrid mixer designed to better model elongated, slender defects by combining:
  - a Mamba (SSM-based) sub-branch to capture long-range dependencies efficiently,
  - a non-SSM (lightweight feed-forward) sub-branch for local detail enrichment,
  - a Slender convolution sub-branch that learns deformable offsets to align the receptive field to slender geometries.  
  This module improves sensitivity to elongated structures that standard convolutional kernels struggle with.
- **Category-Aware Head (CAHead):** reworks localization into a center-point + class-guided refinement paradigm, with parallel center/offset/size attention branches that fuse category priors to adapt aspect-ratio and improve localization for morphologically diverse defect classes.
- **Few-Shot Adaptation Branch (FAB):** a detachable training-time branch using a class-agnostic aggregation (CAA) and support–query episodes (3-shot 1-way setup) that enables rapid adaptation to novel defect types; it is removed during inference to preserve runtime speed.

A schematic of the YPSDD system (image acquisition + YPSDD-Net) and representative module illustrations are described in the paper.

---

## Datasets

We assembled and used two industrial datasets collected on production lines:

- **CFYD — Carbon Fiber Yarn Defects Dataset**  
  - Raw: 2,108 images at 4096×3000 (yarn delivery stage).  
  - Processed: images partitioned into 1024×1024 patches; after filtering ~**10,000** patches retained.  
  - Annotations: COCO-format bounding boxes for **4 defect categories** (Fuzz, Broken Yarns, Fiber Clumps, Loose Filaments) + normal samples.

- **WPSD — Woven Preform Surface Defects Dataset**  
  - Raw: 1,574 preform images at 4096×3000 (weaving stage).  
  - Processed: partitioned into 1024×1024 patches; after screening ~**5,000** patches retained.  
  - Annotations: COCO-format bounding boxes for **3 defect categories** (Knots, Yarn Breakage, Floating Yarns) + normal samples. Defect instances are comparatively rare (realistic distribution).

> Note: both datasets were resized to **512×512** for network input during training.

---

## Training & Environment (as reported)

- **Framework:** PyTorch  
- **Hardware used in experiments:** Intel Core i9-14900HX CPU and NVIDIA GeForce RTX 4080 (16 GB)  
- **Input size:** 512×512 (images were downsampled from 1024×1024 patches)  
- **Training schedule (two-stage):**  
  1. **Stage 1 (Main detection branch):** end-to-end training on abundant base samples to learn structural priors — reported to run **500 epochs**, batch size **4**, `num_workers=4`.  
  2. **Stage 2 (Few-shot adaptation):** freeze backbone, use a 3-shot 1-way episodic setup to fine-tune CAHead and FAB (support–query episodes). FAB is removed at inference.

---

## Provided reviewer checkpoint

A reviewer-only checkpoint is included in this private release as `models/ypsdd_net_reviewer.pt`. It is provided for reproducibility inspection and reviewer validation. The checkpoint may be:

> The full training scripts, reproducible training logs and the complete pretrained models used in the paper will be released publicly after paper acceptance.

---

## Reported results (high level)

YPSDD-Net demonstrates strong performance on industrial yarn and preform defect detection benchmarks reported in the paper; for example **AP50 ≈ 96.7% on yarn defects** and **AP50 ≈ 90.7% on preform defects**, while outperforming several recent baselines in ablation and comparative studies.
