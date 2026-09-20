# Distortion-Aware Object Localization

A synthetic computer-vision dataset for object localisation under **per-image**
nonlinear camera distortion.

Each image is rendered from a canonical 256×256 scene containing six fixed
object slots and a 6×6 calibration dot grid, then warped by an imaging model
whose parameters are redrawn for every image. The released images are the warped
ones; the labels are the canonical, pre-warp bounding boxes.

## Motivation

Most distortion-correction benchmarks apply one fixed camera model across a whole
capture, so the correction can be learned once and reused. Here the geometry
changes frame to frame, which makes a single dataset-wide correction inadequate
by construction and forces the per-image calibration that real camera pipelines
perform.

## Contents

| Split | Images | Labelled boxes |
|---|---|---|
| train | 800 | 4,800 |
| test | 240 | 1,440 (held out) |

Labels use one row per object:

| Column | Type | Description |
|---|---|---|
| `image` | string | image filename |
| `object_slot` | integer | fixed semantic object slot, 0–5 |
| `x1, y1, x2, y2` | float | canonical pre-warp box, in pixels |

Exactly six rows per image, so there are no missing objects.

| Slot | Object | Colour (R,G,B) |
|---|---|---|
| 0 | circle | (220, 70, 70) |
| 1 | rectangle | (70, 150, 230) |
| 2 | triangle | (80, 190, 110) |
| 3 | ellipse | (205, 90, 200) |
| 4 | diamond | (220, 170, 40) |
| 5 | hexagon | (80, 190, 190) |

## Imaging model

Applied to every image, with parameters drawn independently per image:

1. **Radial distortion** — two coefficients about an optical centre jittered up
   to 9 px from the image centre.
2. **Affine term** — up to 5% scale, 0.055 shear, 3.5° rotation.
3. **Residual displacement field** — a smooth, low-frequency, non-parametric
   displacement with a peak of about 3.5 px. It lies outside the radial and
   affine families, so it cannot be removed by fitting a parametric camera
   model, and it is what bounds the achievable accuracy.
4. **Sensor noise** — Gaussian, σ = 4.

## Calibration grid

Every image carries 36 dark dots (RGB ≈ (35, 35, 35), radius ≈ 2.4 px) whose
canonical centres are x, y ∈ {48, 80, 112, 144, 176, 208}. They pass through the
same warp as the objects and act as the per-image geometric reference.

## Generation and reproducibility

The data comes from a seeded deterministic procedural renderer. Every per-image
quantity — scene layout, distortion parameters, residual field and pixel noise —
derives from that image's own integer seed, so images are independent and
regenerating from the same seeds reproduces the data byte for byte.

**The generator source is distributed with the dataset itself rather than here.**
The test split's seeds are deliberately not published: releasing them would let
anyone reconstruct the held-out labels, which would make the benchmark
meaningless. Reviewers and anyone working with the full dataset receive the
complete, seeded generator alongside the data.

No LLM-generated labels are used, no external benchmark's data is redistributed,
and the dataset contains no personal or sensitive content.

## Licence

CC BY 4.0. Creator-generated; no third-party content is included.
