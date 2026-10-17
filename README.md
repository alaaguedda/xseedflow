# XSeedFlow: How few X-ray views does a seed need?

An independent mini-study on sparse-view X-ray tomography of wheat kernels, inspired by the XSeedFlow research theme (Université d'Angers, LARIS, CEA). It asks which projection angles carry complementary information, and how far the number of views can be reduced before the biological information is lost.

Live report: https://<your-project>.vercel.app
Notebook: `notebooks/xseedflow_main_study.ipynb` (runs on free Google Colab, T4 GPU)

## Motivation

A tomographic scan of a seed needs many X-ray projections, which makes it slow and expensive. If a seed can be reconstructed from far fewer views with no loss of the measurements biologists care about (length, width, thickness, volume), acquisition can be made faster and cheaper. This project builds a small, reproducible pipeline to quantify that trade-off on real micro-CT data.

## Dataset

Public wheat-kernel micro-CT dataset:
Shi et al., Scientific Data 13, 804 (2026). Figshare DOI 10.6084/m9.figshare.30145858 (CC BY).

A subset of 12 kernels is used so that everything runs within the free Colab limits (about 12 GB GPU memory and 12 GB RAM). The processed subset is stored on Google Drive and reused by every later step.

## Research questions

| ID  | Question |
|-----|----------|
| RQ1 | How many views are actually necessary? Where do extra projections stop paying off? |
| RQ2 | Are some viewing angles more informative than others at a fixed view budget? |
| RQ3 | Does the best acquisition depend on seed shape (elongation, cross-section)? |
| RQ4 | Do sparse reconstructions preserve the biological measurements (volume, length, width, thickness)? |
| RQ5 | Do the conclusions survive a less ideal simulation (detector noise, rotation-centre error)? |
| RQ6 | Can the number of views needed be predicted from shape descriptors? |

## Method

- **Geometry:** parallel-beam simulation. The kernel long axis is the stack axis, so each axial slice is an independent 2-D problem and views are rotations about that axis. The candidate angle library is 0 to 179 degrees in 1-degree steps.
- **Projector:** a pure PyTorch implementation (rotation by grid sampling followed by a row sum). The back-projection adjoint was checked numerically (ratio close to 1.000).
- **Reconstruction:** filtered back-projection (Ram-Lak filter) and SIRT with a [0, 1] box constraint. SIRT is clearly better in the sparse regime.
- **Angle selection strategies:** uniform spacing, random selection, and a greedy oracle that uses the reference volume.
- **Image metrics:** NRMSE and SSIM against the dense reference.
- **Task metrics:** Dice overlap, and percentage error on volume, length, width and thickness, measured after Otsu thresholding using PCA extents of the largest connected component.
- **Knee point:** the smallest number of views that reaches 90 percent of the achievable gain between 4 and 180 views.
- **Robustness:** additive detector noise and a one-pixel rotation-centre shift.
- **Checkpointing:** results are saved to Drive after every (kernel, experiment) block, so an interrupted Colab session can resume.

## Repository content

```
.
├── index.html                      Interactive report served by Vercel
├── notebooks/
│   └── xseedflow_main_study.ipynb  Full pipeline (RQ1 to RQ6) and report builder
├── .vercelignore                   Keeps the notebook out of the deployed site
├── .gitignore
└── README.md
```

## The interactive report

`index.html` is a single self-contained page (Plotly is inlined, no server and no build step). It contains:

- A 3-D view of a real kernel, colored by its surface deviation from the reference. It can be rotated and zoomed, and has its own slider for the number of X-ray views.
- A slice view with the reference, the reconstruction and the error map, with its own independent slider.
- One section per research question, each with its plots and a short written reading of the results.

## Running the notebook

1. Open the notebook in Google Colab and select a GPU runtime (free T4 is enough).
2. Run the cells in order. The first cell mounts Google Drive and reads the configuration.
3. Outputs are written to a project folder on Drive:
   - `results/` : `results.csv`, shape descriptors, summary JSON and figures
   - `web/` : an importable core module, per-kernel meshes, precomputed reconstructions and a manifest with the UI parameter schema, prepared for a future interactive web application
   - `report/xseedflow_report.html` : the report (copy it here as `index.html`)

## Website readiness

The notebook exports `xseedflow_core.py` (the exact code used in the study), a `manifest.json` describing every user control and the research question it serves (number of views, angle selection mode, algorithm, iterations, noise, rotation-centre offset, segmentation threshold, tolerance), and 3-D surface meshes for each kernel. A backend only needs `get_ref` and `run_scenario`; a browser only needs the meshes and the manifest.

## Limitations

- Projections are simulated from already reconstructed CT slices and inverted with the same operator, so absolute errors are optimistic (an inverse crime). Comparisons between strategies on the same operator remain meaningful.
- The greedy angle search is an oracle: it uses the reference volume and is not available in a real scan.
- Segmentation is a global Otsu threshold.
- Only a small subset of kernels is analysed, so shape-dependence results (RQ3, RQ6) are indications and not statistical conclusions. Adding kernels is a configuration change.
- The voxel size is self-calibrated from the published major-axis length and is an estimate.
- Fewer projections does not translate into a proportionally shorter scan time.

## Reference

Shi et al., "Wheat kernel micro-CT dataset", Scientific Data 13, 804 (2026). Figshare: https://doi.org/10.6084/m9.figshare.30145858

## License

Code: choose a license (for example MIT).
Data: CC BY, see the dataset reference above.