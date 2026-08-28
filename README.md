# Land Use / Land Cover Classification — South-Western Rwanda

## Overview
This project combines Sentinel-2 multispectral satellite imagery with machine
learning to produce a land use / land cover (LULC) map of a region in the
South-Western province of Rwanda. The model classifies land into four categories:
**Forest, Tea, Water, and Other**.

## Data
- **Training-validation points**: 800 labeled points (shapefile format), 200 per
  class, provided as `data/train_val.shp` (+ associated `.shx`, `.dbf`, `.prj`, `.cpg` files)
- **Satellite imagery**: `data/S2_srw.tif` — an 11-band Sentinel-2 surface
  reflectance GeoTIFF (bands B2–B9, B11, B12), covering the area of interest
- Both datasets are in **EPSG:4326** (WGS84), so no reprojection was required

## Environment Setup
A conda environment (`rwanda-oath`) was created with the following key packages:
`geopandas`, `rasterio`, `scikit-learn`, `pandas`, `numpy`, `matplotlib`, `jupyter`.

To recreate it:
```bash
conda create -n rwanda-oath python=3.11 -y
conda activate rwanda-oath
conda install -c conda-forge geopandas rasterio shapely fiona scikit-learn pandas numpy matplotlib jupyter -y
```

## Methodology
1. **Data loading**: Loaded the training-validation points with `geopandas` and
   inspected the raster with `rasterio` (band count, CRS, bounds).
2. **Data exploration**: Verified class balance (200 points per class, no missing
   values) and visually inspected spectral separability of engineered features
   across classes.
3. **Feature engineering**: Sampled all 11 raw spectral bands at each training
   point location, then computed three spectral indices:
   - **NDVI** (Normalized Difference Vegetation Index) — separates vegetated
     from non-vegetated land
   - **NDWI** (Normalized Difference Water Index) — isolates water bodies
   - **NDMI** (Normalized Difference Moisture Index) — helps distinguish
     vegetation types by canopy moisture
4. **Train/validation split**: 80/20 stratified split (640 training / 160
   validation points), preserving class balance in both sets.
5. **Model training**: Trained a **Random Forest Classifier** (300 trees) on the
   14 features (11 bands + 3 indices), validated with 5-fold stratified
   cross-validation on the training set.
6. **Prediction**: Applied the trained model to every pixel in the full Sentinel-2
   image (~576,000 pixels) to generate a complete predicted land cover map.
7. **Evaluation**: Assessed final performance on the held-out validation set using
   a classification report and a **normalized confusion matrix**.

## Results
- **Cross-validation accuracy** (training set, 5-fold): **91.7%** (± 3.2%)
- **Validation set performance** (normalized confusion matrix):
  - Water: 100% correctly classified
  - Other: 95% correctly classified
  - Forest: 90% correctly classified (10% confused with Tea)
  - Tea: 78% correctly classified (20% confused with Other, 3% with Forest)

**Interpretation**: Water and non-vegetated "Other" land are highly spectrally
distinct and classified with high accuracy. Tea shows the most confusion, likely
due to spectral overlap with forest (shared vegetation signal) and with "other"
(variable canopy density and exposed soil between plantation rows). The large,
contiguous forest block in the predicted output aligns well with the known
location of Nyungwe Forest, supporting the plausibility of the results.

## Output
- Predicted land cover map: `outputs/predicted_lulc.tif` (georeferenced GeoTIFF,
  same CRS/extent as the input image)
- Class code mapping: `{0: 'forest', 1: 'other', 2: 'tea', 3: 'water'}`

## Repository Structure

'''
rwanda-lulc-project/
├── data/
│ ├── train_val.shp (+ .shx, .dbf, .prj, .cpg) # training-validation points
│ └── S2_srw.tif # Sentinel-2 image (not committed — large file)
├── notebooks/
│ └── lulc_classification.ipynb # full analysis notebook
├── outputs/
│ └── predicted_lulc.tif # final predicted LULC map
└── README.md

'''