# Jigsaw Puzzle Image Processing - Milestone 1

This repository contains the **High-Fidelity Image Processing Pipeline** designed to prepare jigsaw puzzle tiles for later assembly. The pipeline focuses on denoising, anti-aliasing, and clean edge extraction, ensuring consistent tile features across various puzzle sizes (2×2, 4×4, 8×8).

---

## Folder Structure

Project-img/
│
├─ data/ # Raw puzzle images
│ ├─ 2x2/
│ ├─ 4x4/
│ └─ 8x8/
│
├─ notebooks/ # Jupyter notebooks
│ └─ Milestone1_pipeline.ipynb
│
├─ Visuals/ # Saved figures/screenshots
│ └─ example 1....
│
├─ Output/ # Saved processed images
│
├─ Report/ # Milestone 1 report
│ └─ Milestone1_Report.pdf
│
├─ Correct Images/ # Optional: reference or manually corrected images
│
└─ README.md

---

## Pipeline Overview

The **high-fidelity pipeline** consists of the following steps:

1. **Upscaling**  
   - Each tile is upscaled 4× using `INTER_LANCZOS4` to reduce aliasing and improve edge continuity.

2. **Mean Shift Filtering**  
   - Removes JPEG artifacts and smooths colors while preserving edges.

3. **Light Enhancement (CLAHE)**  
   - Improves local contrast in the L-channel of LAB color space without introducing color distortion.

4. **Canny Edge Detection**  
   - Extracts clean, continuous edges suitable for tile matching and puzzle reconstruction.

---
Notes

The pipeline works across all puzzle grid types (2×2, 4×4, 8×8).

Visualizations are intended for interactive notebooks and GitHub documentation.

For large datasets, processing may be slow due to high-fidelity operations (upsampling + mean shift).

Make sure dataset_path is updated relative to your local Project-img folder.

License

MIT License – feel free to reuse and modify for educational purposes.


This `README.md` is ready for GitHub — it explains your project, folder structure, pipeline steps, usage, and outputs clearly.  

If you want, I can also **write a compact version** with **inline screenshots links** for the Visuals folder so your GitHub looks polished immediately. Do you want me to do that?

