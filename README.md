# Jigsaw Puzzle Image Processing & Assembly Pipeline

This repository contains the **complete end‑to‑end jigsaw puzzle reconstruction pipeline**, covering **Milestone 1 (Image Processing & Feature Preparation)** and **Milestone 2 (Puzzle Solving & Reconstruction)**.

After extensive experimentation in Milestone 2, the role of Milestone 1 was clarified and **restricted to deterministic image standardization and slicing only**. No perceptual filtering or feature extraction is performed at this stage.

The final and authoritative implementation for **both milestones** is consolidated inside:

`Milestone1notebook.ipynb`

This notebook contains the full pipeline, experiments, and final algorithmic choices used throughout the project.

---

## Full Pipeline Overview (Step-by-Step)

The complete system follows a **clear, sequential pipeline** designed to transform a raw, unscrambled image into a fully reconstructed jigsaw puzzle. Each step contributes directly to ensuring correctness, consistency, and reliable final assembly.

### Step 1: Input Acquisition

Raw, unscrambled images are loaded from the `data/` directory. Each image represents the ground-truth puzzle before scrambling.

**Contribution to the overall goal:**
Provides a clean and consistent starting point for all subsequent processing stages.

---

### Step 2: Preprocessing (Structural Standardization)

Each input image is resized to a fixed resolution of **224×224 pixels** using area-based interpolation (`cv2.INTER_AREA`).

**Contribution to the overall goal:**
Ensures uniform spatial dimensions across all puzzles, allowing each tile to correspond to a predictable region in the grid and eliminating scale-related ambiguity during reconstruction.

---

### Step 3: Deterministic Grid Slicing

The standardized image is sliced into uniform grids (2×2, 4×4, or 8×8). Tiles are extracted using fixed index arithmetic with no overlap.

**Contribution to the overall goal:**
Transforms the image into discrete puzzle pieces while preserving exact spatial relationships, which is essential for accurate global matching.

---

### Step 4: Tile Representation

Each puzzle piece is represented as a raw image patch. Instead of contour or edge-based descriptors, the system relies on **patch-level color information** as the primary representation.

**Contribution to the overall goal:**
Preserves all visual information needed for reliable comparison while avoiding premature information loss that could harm matching accuracy.

---

### Step 5: Global Matching

Puzzle pieces are compared against reference positions using mean absolute difference in **LAB color space**. The resulting cost matrix is solved using the **Hungarian algorithm (linear sum assignment)**.

**Contribution to the overall goal:**
Produces a globally optimal one-to-one assignment between tiles and grid positions, preventing greedy or locally optimal placement errors.

---

### Step 6: Reconstruction

Each matched tile is placed exactly once into its assigned grid position to form the final reconstructed image.

**Contribution to the overall goal:**
Ensures deterministic, repeatable reconstruction that faithfully restores the original image structure.

---

## Updated Folder Structure

```
Project-img/
│
├─ data/                     # Raw input images (unscrambled)
│   ├─ 2x2/
│   ├─ 4x4/
│   └─ 8x8/
│
├─ Output/                   # Generated puzzle tiles per grid size
│
├─ Visuals/                  # Saved figures & diagnostics
│   ├─ Milestone 1/          # Preprocessing & edge visuals
│   └─ Milestone 2/          # Matching & reconstruction visuals
│
├─ final_result/             # Final corrected images from Milestone 2
│   ├─ 2x2                   # final 2x2 images
│   ├─ 4x4                   # final 4x4 images
│   ├─ 8x8                   # final 8x8 images
│   └─ full_solving_log.json # full solving logic
├─ notebooks/
│   └─ Milestone1notebook.ipynb   # Complete code for Milestones 1 & 2
│
├─ Correct Images/            # Ground‑truth reference images
│
├─ Report/
│   ├─ Milestone1_Report.pdf
│   └─ Milestone2_Report.pdf
│
└─ README.md
```

---

## Milestone 1 – Image Slicing & Structural Standardization

Milestone 1 **does NOT apply mean shift filtering, CLAHE, or Canny edge detection**.

Its sole responsibility is to convert raw images into a **structurally consistent puzzle representation** that the Milestone 2 solver can rely on without ambiguity.

All Milestone 1 logic is implemented in the `PuzzleSlicer` class inside `Milestone1notebook.ipynb`.

### What Milestone 1 Actually Does

#### 1. Fixed Global Resize (224×224)

Each input image is resized to **224×224 pixels** using `cv2.INTER_AREA`.

**Why this is correct (Accuracy):**

* Guarantees identical spatial resolution for all puzzles.
* Ensures each tile maps exactly to a reference patch in Milestone 2.

**Why this is correct (Efficiency):**

* One resize per image.
* Low computational cost.

---

#### 2. Deterministic Grid Slicing

Images are sliced into uniform grids:

* 2×2
* 4×4
* 8×8

Tiles are extracted using direct array slicing with fixed index arithmetic.

**Why this is correct (Accuracy):**

* Produces perfectly aligned, non-overlapping tiles.
* Preserves true spatial relationships.

**Why this is correct (Efficiency):**

* O(1) slicing per tile.
* No filtering or feature computation.

---

#### 3. Strict Output Convention

Tiles are saved as:

```
Output/<grid>/<image_id>/piece_i_j.jpg
```

**Why this is correct:**

* Deterministic ordering.
* Matches solver expectations exactly.
* Simplifies indexing and loading.

---

## Design Comparison & Rationale (Before Milestone 2)

Before proceeding to Milestone 2, it is important to justify why the **final adopted methodology** was chosen over alternative approaches explored earlier in the project.

### Comparison with Earlier / Alternative Milestone 1 Approaches

Earlier iterations experimented with perceptual preprocessing techniques such as contrast enhancement, edge detection, and contour-based representations. While these methods are commonly used in classical jigsaw puzzle solvers, they were ultimately found to be **suboptimal for the specific constraints of this project**.

**Limitations of earlier approaches:**

* Edge and contour extraction introduced sensitivity to noise and illumination changes.
* Filtering operations caused irreversible information loss, negatively affecting downstream matching.
* Contour-based features increased implementation complexity without providing consistent accuracy gains for rectangular, grid-based pieces.

**Advantages of the final Milestone 1 approach:**

* Deterministic resizing and slicing preserve *all* original visual information.
* Structural standardization ensures perfect alignment between tiles and reference positions.
* The absence of perceptual filtering keeps the pipeline fast, reproducible, and free of parameter tuning.

As a result, the final Milestone 1 design provides a **clean, lossless, and solver-agnostic representation**, which proved more reliable than earlier perceptual or edge-driven alternatives.

---

### Why the Chosen Methods Outperform Other Possible Strategies

The combination of **patch-based color representation** and **global optimization** was selected after evaluating multiple possible solving strategies.

**Compared to local or greedy matching methods:**

* Greedy approaches often produce locally optimal but globally incorrect assemblies.
* The Hungarian algorithm guarantees a globally optimal assignment across all pieces.

**Compared to feature-heavy or learning-based methods:**

* Feature descriptors and learning models require large training datasets and careful tuning.
* The chosen approach is fully deterministic, interpretable, and data-efficient.

**Compared to edge-only or contour-only solvers:**

* Edge-based solvers struggle when pieces lack distinctive geometric variation.
* Color-patch similarity provides stronger and more stable cues for rectangular puzzle pieces.

Overall, the adopted methodology achieves an optimal balance between **accuracy, robustness, interpretability, and computational efficiency**, making it better suited than alternative strategies explored during development.

---

## Milestone 2 – Puzzle Solving & Reconstruction

Milestone 2 builds directly on the refined Milestone 1 outputs to perform **global tile‑to‑slot assignment** and reconstruction.

### Core Methods

#### 1. Patch‑Based LAB Color Matching

Each scrambled tile is compared against reference patches using mean absolute difference in LAB space.

**Why it works (Accuracy):**

* LAB space correlates with human perceptual similarity.
* Robust against illumination changes.

**Why it works (Efficiency):**

* Simple arithmetic operations.
* No feature learning or descriptors.

---

#### 2. Hungarian Algorithm (Linear Sum Assignment)

Solves the global matching problem optimally.

**Why it works (Accuracy):**

* Guarantees globally optimal assignments.
* Prevents greedy local mismatches.

**Why it works (Efficiency):**

* Polynomial time O(n³).
* Scales safely for 2×2, 4×4, and 8×8 puzzles.

---

#### 3. Deterministic Reconstruction

Each tile is placed exactly once in the final grid.

**Why it works:**

* Ensures consistency and repeatability.
* Eliminates ambiguity in placement.

---

**Milestone 2 – Algorithm Comparison and Analysis**

This section presents a comparison between the adopted Milestone 2 reconstruction approach and alternative puzzle-solving strategies, highlighting why the final design was selected.

**Comparison with Greedy and Local Matching Approaches**

Greedy and local matching methods assign puzzle pieces sequentially based on the best immediate match. While simple to implement, these approaches suffer from a major limitation: early placement errors propagate and cannot be corrected.

In contrast, the adopted Milestone 2 solution formulates puzzle reconstruction as a global assignment problem. By constructing a complete cost matrix and solving it using the Hungarian (linear sum assignment) algorithm, all tiles are assigned to positions simultaneously. This guarantees a globally optimal one-to-one assignment, eliminating cascading errors and outperforming greedy strategies in both accuracy and consistency.

**Comparison with Edge-Based and Contour-Based Solvers**

Classical jigsaw puzzle solvers often rely on contour extraction, edge curvature, or geometric boundary matching. While effective for irregularly shaped puzzle pieces, these methods are less suitable for the current project due to the perfectly rectangular, grid-based nature of the pieces.

Edge-based methods are sensitive to noise, illumination changes, and low-contrast boundaries. Additionally, rectangular pieces provide limited geometric information, leading to ambiguous or unreliable matches.

The adopted Milestone 2 approach replaces geometric reasoning with patch-based color similarity in LAB color space, which leverages interior visual content rather than boundary shape. LAB color space provides perceptual robustness, making color-based matching more stable and discriminative under the given constraints.

**Comparison with Feature-Descriptor Methods**

Feature-descriptor approaches such as SIFT or ORB rely on detecting and matching keypoints across image regions. These methods introduce additional complexity and require strong local features, which may be absent in smooth or homogeneous image areas.

The selected LAB color matching approach avoids explicit feature extraction entirely. It relies on simple, interpretable distance measures, reducing computational overhead while maintaining reliable matching performance for structured, grid-based puzzles.

**Comparison with Learning-Based Methods**

Learning-based puzzle solvers require large labeled datasets, training procedures, and careful hyperparameter tuning. While powerful, these methods reduce interpretability and introduce unnecessary complexity for small-scale, deterministic reconstruction tasks.

The final Milestone 2 solution is fully deterministic, training-free, and reproducible. Its behavior is transparent and directly explainable, making it more suitable for controlled academic evaluation and validation.

**Overall Evaluation**

The combination of patch-based LAB color matching and the Hungarian algorithm provides a strong balance between global optimality, robustness, simplicity, and computational efficiency. By solving the reconstruction problem globally rather than through local heuristics, the adopted approach avoids common failure modes and delivers consistent, reliable results under the project’s constraints.
---

## Final Outputs

* **Final reconstructed images** are saved in `final_result/`.
* Diagnostic figures are saved under `Visuals/Milestone 1` and `Visuals/Milestone 2`.
* The pipeline works reliably across all supported grid sizes.

---

## Notes

* The project emphasizes deterministic, interpretable algorithms.
* All design decisions are validated empirically.
* `Milestone1notebook.ipynb` reflects the final corrected implementation.

---

## License

MIT License – free to use and modify for educational and research purposes.
