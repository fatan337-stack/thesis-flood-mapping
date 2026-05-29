# Flood Mapping with Sentinel-1 SAR Data

Geospatial Python pipeline for preparing Sentinel-1 SAR data and Copernicus EMS reference data for deep learning–based flood extent mapping. Part of a degree project at Karlstad University (Surveying and Geographical IT).

This repository will grow into the full pipeline used in the thesis. Currently it contains the **data preparation** notebook; additional notebooks (Random Forest, Otsu thresholding, U-Net inference, and evaluation) will be added after the thesis is finalized.

## Project context

The thesis evaluates the generalization capability of the pre-trained **STURM-Flood** U-Net model (Notarangelo et al., 2025) when applied to Swedish flood conditions. Three Copernicus EMSR427 areas (Storm Dennis, February 2020) are used as test sites: Karlstad, Göteborg, and Kristianstad.

## Notebook overview

### `01_data_preparation.ipynb`

Prepares a Sentinel-1 scene and the corresponding Copernicus EMS reference vectors into a tiled dataset ready for ML model inference and training.

Key steps:

1. **Interactive file selection** — Google Drive scanning with `ipywidgets`
2. **Vector processing** — parsing EMS shapefiles with `geopandas`, building exclusion masks from AOI and permanent water polygons
3. **Coordinate transformations** — reprojection between WGS84 and the Sentinel-1 scene's native UTM CRS using `pyproj`
4. **Raster operations** — AOI-based clipping, dB normalization (−30 to 10 dB → 0–1), and rasterization of EMS vectors to match the SAR grid
5. **Tile extraction** — generation of 128×128 GeoTIFF tiles matching the STURM-Flood input format
6. **Visualization** — interactive `folium` map of tile locations with ground truth water percentage overlay

## Configuration

The notebook expects data to live in your Google Drive. Mount your Drive in the first runnable cell, then set the `BASE` variable in the configuration cell to point to your project folder:

```python
BASE = '/content/drive/MyDrive/Project_data_Stefan_Edvinsson'
OUTPUT_DATASET_DIR = f'{BASE}/Dataset_goteborg'   # adjust for other study areas
```

All other paths are built from `BASE`. To run the notebook with the shared data folder, add the folder as a shortcut to the root of your Google Drive (right-click the shared folder → 'Add shortcut to Drive' → 'My Drive'). The default `BASE` path then matches the shared folder's location and no changes are needed.

## Dependencies

The notebook is designed to run in **Google Colab**, which already provides most of the required packages. For local execution, install the dependencies in `requirements.txt`:

```bash
pip install -r requirements.txt
```

## Data sources

- **Sentinel-1 SAR** — Copernicus Data Space Ecosystem (https://dataspace.copernicus.eu)
- **Reference data** — Copernicus Emergency Management Service, EMSR427 (https://emergency.copernicus.eu/mapping/list-of-components/EMSR427)
- **STURM-Flood pre-trained model and dataset** — Zenodo (https://doi.org/10.5281/zenodo.12748982)

Data is not included in this repository due to size constraints. The Sentinel-1 scenes must be preprocessed in SNAP (thermal noise removal, orbit file, calibration, terrain correction, speckle filtering, dB conversion) before being used as input to the notebook.

## Course context

This notebook was also submitted as the project work for **Geospatial Python (GMAF01)** at Karlstad University. It demonstrates several core learning outcomes of the course:

- Use of multiple GIS libraries (`rasterio`, `geopandas`, `shapely`, `folium`, `pyproj`)
- Modification and analysis of geometric objects
- Coordinate system handling and transformations
- Raster analysis (clipping, normalization, tiling)
- Creation of interactive maps

## LLM usage declaration

The Claude AI assistant (Anthropic) was used to help with the path configuration setup and to write this README. All code logic and the analytical pipeline are my own work. I reviewed and understood the assistant's suggestions before incorporating them.

## License

MIT License — see `LICENSE` for details.

## Author

Stefan Edvinsson — Karlstad University, 2026
