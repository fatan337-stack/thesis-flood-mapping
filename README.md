# Flood Mapping with Sentinel-1 SAR Data

Geospatial Python pipeline for preparing Sentinel-1 SAR data and Copernicus EMS reference data for deep learning based flood extent mapping, and for running and evaluating flood inference with a pretrained U-Net. Part of a degree project at Karlstad University (Surveying and Geographical IT).

This repository contains the data preparation, U-Net inference, and Random Forest notebooks. The remaining notebooks (Otsu thresholding and evaluation) will be added shortly.

## Project context

The thesis evaluates the generalization capability of the pre-trained **STURM-Flood** U-Net model (Notarangelo et al., 2025) when applied to Swedish flood conditions. Three Copernicus EMSR427 areas (Storm Dennis, February 2020) are used as test sites: Karlstad, Göteborg, and Kristianstad.

## Notebook overview

### `01_data_preparation.ipynb`

Prepares a Sentinel-1 scene and the corresponding Copernicus EMS reference vectors into a tiled dataset ready for ML model inference and training.

Key steps:

1. **Interactive file selection**: Google Drive scanning with `ipywidgets`
2. **Vector processing**: parsing EMS shapefiles with `geopandas`, building exclusion masks from AOI and permanent water polygons
3. **Coordinate transformations**: reprojection between WGS84 and the Sentinel-1 scene's native UTM CRS using `pyproj`
4. **Raster operations**: AOI-based clipping, dB normalization (the -30 to 10 dB range mapped to 0 to 1), and rasterization of EMS vectors to match the SAR grid
5. **Tile extraction**: generation of 128×128 GeoTIFF tiles matching the STURM-Flood input format
6. **Visualization**: interactive `folium` map of tile locations with ground truth water percentage overlay

### `02_unet_inference.ipynb`

Runs flood extent inference with the pretrained STURM-Flood U-Net on the prepared Sentinel-1 tiles, computes pixel-wise evaluation metrics against the ground truth, and visualizes the predictions.

Key steps:

1. **Environment setup**: clones the STURM-Flood repository and installs dependencies
2. **Configuration**: sensor selection, data source (upload or Google Drive), and study area
3. **Model setup**: downloads the pretrained Sentinel-1 U-Net weights from Zenodo and rebuilds the model
4. **Inference**: per-tile flood probability maps thresholded into binary water masks
5. **Evaluation**: micro-averaged Precision, Recall, F1, IoU and accuracy, reported to match the STURM-Flood paper
6. **Output**: binary and probability GeoTIFFs, comparison visualizations, and a per-tile metrics CSV

This notebook is adapted from the STURM-Flood project's inference notebook. The U-Net architecture and the core inference, I/O and visualization functions are imported from the STURM-Flood repository, which is cloned at runtime, and are not redistributed here. See the License section below.

### `03_random_forest.ipynb`

A two-stage Random Forest pipeline that serves as a machine learning baseline. The model is trained on STURM-Flood SAR tiles (VV and VH bands) and tested on the external EMSR427 areas.

Key steps:

1. **Configuration and data copy**: folder selection and copying tiles from Google Drive to local disk for faster reads
2. **Event split**: an event-level 80/20 train and validation split on the STURM-Flood data, so no event appears in both sets
3. **Tile percentage search**: a learning curve over the share of tiles used per event, evaluated on the validation split
4. **Depth search**: a `max_depth` search at the chosen tile percentage, on the same validation split
5. **Final model**: trained with the selected parameters and saved as a `.joblib` model plus a `.json` config
6. **External test**: the final model is run on all three EMSR427 areas, writing per-tile metrics and binary GeoTIFF predictions named `RF_<tile>` for compatibility with the downstream TileExplorer notebook

All model selection happens within the STURM-Flood data only. The external test areas are used solely for final evaluation, never for tuning.

## Configuration

`01_data_preparation.ipynb` expects data to live in your Google Drive. Mount your Drive in the first runnable cell, then set the `BASE` variable in the configuration cell to point to your project folder:

```python
BASE = '/content/drive/MyDrive/Project_data_Stefan_Edvinsson'
OUTPUT_DATASET_DIR = f'{BASE}/Dataset_goteborg'   # adjust for other study areas
```

All other paths are built from `BASE`. To run the notebook with the shared data folder, add the folder as a shortcut to the root of your Google Drive (right-click the shared folder, 'Add shortcut to Drive', 'My Drive'). The default `BASE` path then matches the shared folder's location and no changes are needed.

`02_unet_inference.ipynb` has its own configuration cell for sensor selection, data source, and study area. Set these before running the rest of the notebook.

## Dependencies

The notebooks are designed to run in **Google Colab**, which already provides most of the required packages. For local execution, install the dependencies in `requirements.txt`:

```bash
pip install -r requirements.txt
```

## Data sources

- **Sentinel-1 SAR**: Copernicus Data Space Ecosystem (https://dataspace.copernicus.eu)
- **Reference data**: Copernicus Emergency Management Service, EMSR427 (https://emergency.copernicus.eu/mapping/list-of-components/EMSR427)
- **STURM-Flood dataset**: Zenodo (https://doi.org/10.5281/zenodo.12748982)
- **STURM-Flood Sentinel-1 model weights**: Zenodo (https://doi.org/10.5281/zenodo.15189664)
- **STURM-Flood code**: https://github.com/STURM-WEO/STURM-Flood

Data is not included in this repository due to size constraints. The Sentinel-1 scenes must be preprocessed in SNAP (thermal noise removal, orbit file, calibration, terrain correction, speckle filtering, dB conversion) before being used as input to the notebooks.

## Course context

`01_data_preparation.ipynb` was also submitted as the project work for **Geospatial Python (GMAF01)** at Karlstad University. It demonstrates several core learning outcomes of the course:

- Use of multiple GIS libraries (`rasterio`, `geopandas`, `shapely`, `folium`, `pyproj`)
- Modification and analysis of geometric objects
- Coordinate system handling and transformations
- Raster analysis (clipping, normalization, tiling)
- Creation of interactive maps

## License

The notebooks authored for this project (`01_data_preparation.ipynb`, `03_random_forest.ipynb`, and the upcoming Otsu and evaluation notebooks) are released under the MIT License. See `LICENSE`.

`02_unet_inference.ipynb` is an adaptation of the STURM-Flood project's inference notebook, which is licensed under [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/). As an adaptation of that work, this notebook is distributed under the same CC BY-SA 4.0 license. The STURM-Flood model code and weights are obtained from the STURM-Flood repository and Zenodo at runtime and are not redistributed here.

## Citation

This project uses the STURM-Flood dataset and inference code. If you build on this work, please also cite:

Notarangelo, N. M., Wirion, C., & van Winsen, F. (2025). STURM-Flood: A curated dataset for deep learning-based flood extent mapping leveraging Sentinel-1 and Sentinel-2 imagery. Big Earth Data. https://doi.org/10.1080/20964471.2025.2458714

## Author

Stefan Edvinsson, Karlstad University, 2026
