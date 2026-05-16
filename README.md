# Crop Yield Forecasting — Rice, Kharif Season, India

Satellite-based district-level rice yield forecasting for **Punjab** and **Madhya Pradesh**, India. The project tests whether progressively tighter cropland masks applied to peak GCVI (Green Chlorophyll Vegetation Index) improve yield prediction R² when combined with ERA5 climate features.

---

## Experiment Design

| Dimension | Details |
|---|---|
| **Regions** | Punjab, Madhya Pradesh |
| **Crop / Season** | Rice, Kharif |
| **GCVI conditions** | `none` (ERA5 only), `raw` (no mask), `esa` (ESA WorldCover cropland), `crop` (rice crop mask) |
| **Models** | Ridge Regression, Random Forest, Gradient Boosting |
| **Features** | ERA5 monthly climate Jul–Nov: precipitation, PET, 2m temperature, skin temperature, soil moisture, LAI + district-level mean peak GCVI |
| **Validation** | Leave-One-Group-Out CV by year (12 folds, 2013–2024) |
| **Metric** | R², RMSE (mean ± std across folds) |
| **Total experiments** | 2 regions × 4 conditions × 3 models = 24 |

---

## Repository Structure

```
crop-yield-forecasting/
│
├── experiment_masks.ipynb          # Main experiment notebook
│   ├── §1  Config                  # Regions, masks, name fixes
│   ├── §2  Helper functions        # extract_gcvi, load_era5, load_yield, run_experiment
│   ├── §2b Sanity checks           # TIF counts, GCVI stats, panel join checks
│   ├── §3  Run all experiments     # Punjab + MP × 4 masks × 3 models
│   ├── §4  Summary table           # Mean ± std R² / RMSE across folds
│   ├── §5  R² plot                 # Per-fold LOGO-CV line chart (2×3 grid)
│   ├── §6  SHAP analysis           # TreeExplainer beeswarm + mean |SHAP| bar chart
│   └── §7  MP sensitivity (−2022)  # Reruns MP excl. anomalous 2022–23 season
│
├── data/
│   ├── gcvi/                       # Peak GCVI TIFs — naming: peak_gcvi_kharif_{year}_{region}_{mask}.tif
│   ├── yield/                      # DES district yield CSVs (2013–2025, multi-year files)
│   ├── processed/                  # ERA5 wide-format feature CSVs (one per region)
│   ├── era5/                       # Raw ERA5 monthly data
│   ├── punjab_districts_GAUL/      # Punjab district shapefile (GAUL)
│   └── mp_districts_GAUL/          # Madhya Pradesh district shapefile (GAUL)
│
└── notebooks/
    ├── data/
    │   ├── era5_exploration.ipynb          # ERA5 feature exploration
    │   ├── madhyaPradesh.ipynb             # MP data exploration
    │   ├── madhyaPradesh_ricecrop.ipynb    # MP rice yield vs peak GCVI (2024 correlation)
    │   ├── madhyaPradesh_cropmasked.ipynb  # MP crop-masked GCVI exploration
    │   ├── punjab_cropmasked.ipynb         # Punjab crop-masked GCVI exploration
    │   └── process_gcvi_tif_files.ipynb    # TIF pre-processing pipeline
    └── modeling/
        ├── era5_model.ipynb                # ERA5-only baseline model
        └── distribution_shift_analysis.ipynb
```

---

## Data Sources

| Dataset | Source | Notes |
|---|---|---|
| Peak GCVI TIFs | Google Earth Engine (Sentinel-2) | Exported per year/region/mask condition |
| ESA WorldCover mask | ESA WorldCover 10m (2020/2021) | Class 40 = cropland |
| Rice crop mask | GEE custom asset (Jordi) | Class 2 = rice |
| ERA5 climate | Copernicus/ECMWF via GEE | Monthly aggregates Jul–Nov |
| District yield | DES India | Kharif rice, kg/ha, district level |
| District boundaries | GAUL Level 2 | Punjab: 22 districts, MP: 53 districts |

---

## Setup

```bash
pip install pandas numpy geopandas rasterio rasterstats
pip install scikit-learn shap matplotlib seaborn
pip install earthengine-api geemap
```

Yield data requires access to DES India district-level crop statistics. GCVI TIFs are exported from Google Earth Engine — see `notebooks/data/process_gcvi_tif_files.ipynb` for the export pipeline.

---

## Reproducing the Main Experiment

1. Ensure all TIFs are in `data/gcvi/` with the naming convention `peak_gcvi_kharif_{year}_{region}_{mask}.tif`
2. Ensure ERA5 CSVs are in `data/processed/`
3. Run `experiment_masks.ipynb` top to bottom — sanity checks (§2b) will flag any missing files before the main loop
