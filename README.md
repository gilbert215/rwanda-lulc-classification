# Land Use / Land Cover Classification for  South-Western Rwanda

## Overview

This project uses Sentinel-2 satellite imagery and machine learning to classify land use and land cover (LULC) in a region of South-Western Rwanda.

The model classifies the area into four classes:

* **Forest**
* **Tea**
* **Water**
* **Other**

The final result is a georeferenced LULC map covering the study area.

## Data

The project uses two main datasets:

### Training Data

* **File:** `data/train_val.shp`
* **Format:** Shapefile
* **Total points:** 800
* **Points per class:** 200
* **Classes:** Forest, Tea, Water, Other

The shapefile also includes the required `.shx`, `.dbf`, `.prj`, and `.cpg` files.

### Sentinel-2 Imagery

* **File:** `data/S2_srw.tif`
* **Format:** GeoTIFF
* **Bands:** 11 Sentinel-2 surface reflectance bands (B2–B9, B11, B12)
* **Coverage:** Study area in South-Western Rwanda

Both the training points and satellite imagery use **EPSG:4326 (WGS84)**, so no reprojection was required.

## Environment Setup

The project was developed using a Conda environment called `rwanda-oath`.

### Create the environment

```bash
conda create -n rwanda-oath python=3.11 -y
conda activate rwanda-oath
conda install -c conda-forge geopandas rasterio shapely fiona scikit-learn pandas numpy matplotlib jupyter -y
```

### Main packages

* `geopandas` — working with the training points
* `rasterio` — reading and processing satellite imagery
* `scikit-learn` — machine learning
* `pandas` and `numpy` — data processing
* `matplotlib` — visualization
* `jupyter` — running the analysis notebook

## Methodology

The classification workflow follows these main steps:

### 1. Load and inspect the data

The training points were loaded using GeoPandas, while Rasterio was used to inspect the Sentinel-2 image, including its bands, CRS, bounds, and dimensions.

### 2. Explore the data

The training data was checked for class balance and missing values.

There are **200 points for each class**, giving a total of 800 labeled points.

The spectral features were also visualized to see how well the classes could be separated.

### 3. Feature engineering

The 11 Sentinel-2 spectral bands were sampled at each training point.

Three additional spectral indices were calculated:

* **NDVI (Normalized Difference Vegetation Index)** — helps identify vegetation.
* **NDWI (Normalized Difference Water Index)** — helps identify water.
* **NDMI (Normalized Difference Moisture Index)** — provides information about vegetation moisture.

This resulted in **14 features** in total:

**11 spectral bands + 3 spectral indices**

### 4. Train-validation split

The dataset was split into:

* **80% training:** 640 points
* **20% validation:** 160 points

A stratified split was used to keep the same class proportions in both datasets.

### 5. Train the model

A **Random Forest Classifier** with 300 trees was trained using the 14 features.

Five-fold stratified cross-validation was performed on the training data to evaluate the model during training.

### 6. Generate the LULC map

The trained model was applied to every pixel in the Sentinel-2 image.

This produced a complete land cover map containing predictions for approximately **576,000 pixels**.

### 7. Evaluate the model

The final model was evaluated using the held-out validation data.

A classification report and normalized confusion matrix were used to measure performance and identify which classes were being confused with each other.

## Results

### Cross-validation

The Random Forest achieved:

**91.7% ± 3.2% accuracy** using 5-fold cross-validation on the training set.

### Validation performance

The normalized confusion matrix showed the following results:

| Class  | Correctly Classified |
| ------ | -------------------: |
| Water  |                 100% |
| Other  |                  95% |
| Forest |                  90% |
| Tea    |                  78% |


### Interpretation

Water was the easiest class to identify, with all validation samples correctly classified. The **Other** class also performed well.

**Tea** was the most difficult class to classify. This is likely because tea plantations and forests can have similar vegetation signals. Some tea areas may also contain exposed soil or areas with different canopy densities, which can make them more similar to the **Other** class.

The predicted map also shows a large, continuous forest area
## Output

The main output is:

```text
outputs/predicted_lulc.tif
```

This is a georeferenced GeoTIFF containing the predicted land cover classes. It uses the same CRS and spatial extent as the input Sentinel-2 image.


## Repository Structure

```text
rwanda-lulc-project/
│
├── data/
│   ├── train_val.shp
│   ├── train_val.shx
│   ├── train_val.dbf
│   ├── train_val.prj
│   ├── train_val.cpg
│   └── S2_srw.tif              # Sentinel-2 image (not committed)
│
├── notebooks/
│   └── lulc_classification.ipynb
│
├── outputs/
│   └── predicted_lulc.tif      # Final LULC map
│
└── README.md
```
