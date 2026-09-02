# Citrus orchard UAV bare-soil soil-property mapping: reproducibility release v1.0.0

This repository contains anonymised data, Python modules, configuration templates, and validation records for the multi-scale UAV workflow described in the associated manuscript:

**Multi-scale UAV imagery for estimating and mapping soil properties in a citrus orchard with fragmented bare-soil exposure**.

## Public release and patent chronology

- **Related Chinese invention patent application:** 果园裸土土壤 TN 预测方法及装置
- **Application number:** 202611253944.9
- **Patent application date:** 18 August 2026
- **CNIPA acceptance notice issue date:** 18 August 2026
- **CNIPA status before public release:** application accepted
- **First public GitHub release of this code package:** 2 September 2026
- **Release version:** v1.0.0

The source code was developed before the public release date. The date **2 September 2026** records the first public GitHub release of this repository, not the date on which the code was created. The related patent application has an application date of **18 August 2026**, and the CNIPA acceptance notice was also issued on **18 August 2026**, preceding this public release. See `RELEASE_NOTICE.md` for the release-provenance statement.


The release covers:

1. multi-scale ROI construction and CleanV bare-soil pixel purification;
2. shared pixel-level feature formulas and ROI statistics;
3. auxiliary five-fold feature screening followed by conventional LOOCV feature reassessment, local refinement, and TN ablation analyses;
4. prediction-domain-oriented NNDM spatial validation introduced after the feature configuration is fixed, for algorithm and hyperparameter comparison, final model selection, and primary spatial-performance assessment;
5. six-model TN comparison, repeated ten-fold sensitivity analysis, SHAP interpretation, and ROI-grid mapping;
6. coordinate preparation for aligning a target prediction grid with projected sample coordinates.

All feature formulas used by sample-centred extraction and ROI-grid mapping are implemented in `scripts/workflow_core.py`. The NNDM algorithm and fold-plan utilities are implemented in `scripts/nndm_validation.py`.

## Repository structure

```text
.
├── data/
│   ├── original_data/
│   └── reproducibility_current/
│       ├── final_5_features_TN_modeling_data_current.csv
│       ├── final_GBDT_reproduction_parameters_current.csv
│       ├── final_selected_feature_subset_current.csv
│       ├── model_hyperparameter_optimization_summary_current.csv
│       ├── nndm_folds_current.csv
│       └── nndm_settings_current.csv
├── docs/
│   ├── feature_formula_reference.csv
│   └── NNDM_VALIDATION.md
├── examples/templates/
│   ├── sample_points_template.csv
│   ├── soil_lab_results_template.csv
│   ├── final_feature_list_template.csv
│   └── prediction_grid_template.csv
├── outputs/
│   └── minimal_reproduction_current/
├── scripts/
│   ├── workflow_core.py
│   ├── nndm_validation.py
│   ├── 01_multiscale_roi_cleanv_features.py
│   ├── 02_feature_screening_selection.py
│   ├── 03_model_training_and_comparison.py
│   ├── 04_shap_interpretation.py
│   ├── 05_roi_grid_mapping.py
│   ├── 06_reproduce_final_gbdt.py
│   ├── 07_six_model_nndm_fixed_parameters.py
│   └── 08_prepare_prediction_grid_coordinates.py
├── config_example.json
├── config_paper_template.json
├── requirements.txt
├── PACKAGE_CHANGELOG.md
├── RELEASE_NOTICE.md
├── SHA256SUMS.txt
├── VALIDATION.md
├── .gitignore
└── README.md
```

## Installation

```bash
pip install -r requirements.txt
```

Python 3.10 or later is recommended.

## Final five TN features

| Feature | ROI | Image source | Pixel-level definition | ROI statistic |
|---|---:|---|---|---|
| `R0.50m__RGB_Gray_max` | 0.50 m | RGB camera | `(R_rgb + G_rgb + B_rgb) / 3` | maximum |
| `R0.50m__Visible_Brightness_GR_max` | 0.50 m | Multispectral Green and Red | `(G_ms + R_ms) / 2` | maximum |
| `R0.80m__RGB_ExB_std` | 0.80 m | RGB camera | `ExB = 1.4b - g` | population standard deviation |
| `R0.80m__RGB_b_chroma_std` | 0.80 m | RGB camera | `b = B_rgb / (R_rgb + G_rgb + B_rgb + eps)` | population standard deviation |
| `R0.80m__Redness_RG_cv` | 0.80 m | Multispectral Red and Green | `(R_ms - G_ms) / (R_ms + G_ms + eps)` | coefficient of variation |

A complete formula dictionary is provided in `docs/feature_formula_reference.csv`.

## CleanV definition

CleanV retains valid bare-soil pixels after conservative removal of vegetation, coloured field markers, and conspicuous non-soil disturbances. For the manuscript configuration, strong vegetation is identified when NDVI > 0.35, GNDVI > 0.32, and NDRE > 0.25 are simultaneously satisfied; RGB chromatic components and ExG are additionally used for residual-vegetation screening. A one-pixel morphological dilation is applied to the vegetation mask. Thresholds are stored in the configuration files rather than hard-coded in the workflow modules.

## Quick reproduction of the final GBDT NNDM result

Run from the repository root:

```bash
python scripts/06_reproduce_final_gbdt.py
```

Default inputs:

```text
data/reproducibility_current/final_5_features_TN_modeling_data_current.csv
data/reproducibility_current/final_GBDT_reproduction_parameters_current.csv
data/reproducibility_current/nndm_folds_current.csv
```

Expected result, subject only to minor software-version rounding:

```text
NNDM R²   ≈ 0.771065
RMSE      ≈ 122.404 mg kg⁻¹ = 0.122404 g kg⁻¹
MAE       ≈ 97.273 mg kg⁻¹ = 0.097273 g kg⁻¹
RPD       ≈ 2.111207
```

Outputs are written to `outputs/minimal_reproduction_current/`:

```text
final_GBDT_NNDM_predictions.csv
final_GBDT_NNDM_metrics.csv
final_GBDT_NNDM_folds.csv
final_GBDT_full_fit_bundle.joblib
```

The compact release does not disclose raw RTK coordinates or the complete 10,343-point prediction grid. Instead, it includes the anonymised fold plan generated from those spatial inputs. This preserves the exact manuscript validation geometry and allows the reported NNDM model result to be reproduced from the compact feature table.

## Fixed-parameter six-model NNDM comparison

The exact manuscript parameters for GBDT, XGBoost, Random Forest, ExtraTrees, SVR, and PLSR are retained in a separate executable script:

```bash
python scripts/07_six_model_nndm_fixed_parameters.py
```

This script uses the same final five features and archived NNDM folds for all six algorithms. It writes the model summary, sample-level predictions, and folds used to `outputs/six_model_fixed_nndm/`.

The optional repeated ten-fold sensitivity analysis can be recomputed with:

```bash
python scripts/07_six_model_nndm_fixed_parameters.py --run-repeated-tenfold --repeats 100
```

Repeated ten-fold results are auxiliary and are not used for feature selection, hyperparameter selection, final model determination, or primary spatial-performance assessment.

## Full workflow on user-provided UAV data

Prepare co-registered RGB, Green, Red, RedEdge, and NIR orthomosaics, projected sample coordinates, laboratory measurements, and a target prediction-grid file. Copy `config_example.json`, replace the placeholder paths and column names, and run:

```bash
python scripts/01_multiscale_roi_cleanv_features.py --config your_config.json
python scripts/02_feature_screening_selection.py --config your_config.json
python scripts/03_model_training_and_comparison.py --config your_config.json
python scripts/04_shap_interpretation.py --config your_config.json
python scripts/05_roi_grid_mapping.py --config your_config.json
```

### Module 1: ROI, CleanV, and features

`01_multiscale_roi_cleanv_features.py` constructs 0.30-, 0.50-, and 0.80-m ROIs, applies Raw and CleanV pixel schemes, extracts RGB and multispectral features, and exports ROI-quality diagnostics.

### Module 2: feature screening and selection

`02_feature_screening_selection.py` performs feature-quality auditing, cross-scale family reduction, near-duplicate control, balanced candidate-pool construction, auxiliary five-fold beam search, conventional LOOCV reassessment and local refinement, and LOOCV-based Raw–CleanV, ROI-scale, and feature-category ablations. NNDM is not used in this module.

### Module 3: model training and comparison

`03_model_training_and_comparison.py` begins after the feature configuration has been fixed. It compares GBDT, XGBoost, Random Forest, ExtraTrees, SVR, and PLSR using prediction-domain-oriented NNDM folds resolved from the configuration. The model grids supplied to this module should represent the candidate ranges retained after preliminary five-fold narrowing. Repeated ten-fold validation is optional and auxiliary.

### Module 4: model interpretation

`04_shap_interpretation.py` loads the final full-fit GBDT pipeline, computes SHAP values in model-output space, and exports feature-importance and sample-level SHAP tables.

### Module 5: ROI-grid mapping

`05_roi_grid_mapping.py` loads the final model, builds the regular ROI grid, applies bare-soil quality control, extracts the selected property-specific features, and exports prediction points, raster layers, and maps. Results should be described as ROI-grid-scale predictions rather than pixel-wise continuous inversion.

## Generating NNDM folds for a new dataset

`config_example.json` contains a `validation` section. For a new study, provide:

- a sample-coordinate table containing unique sample IDs and projected coordinates in metres;
- a target prediction-grid table in the same coordinate system;
- the coordinate-column names;
- `phi` and `min_train` settings.

The workflow then generates `outputs/validation/nndm_folds.csv` and distance diagnostics. The default manuscript settings are:

```text
phi = max
minimum retained training proportion = 0.5
```

The same property-specific NNDM folds are reused for algorithm comparison, shortlisted hyperparameter comparison, final model selection, and primary spatial-performance assessment after the feature configuration has been fixed. Feature-subset determination and TN ablations are performed earlier using conventional LOOCV.

## Coordinate preparation

When a prediction grid is stored as longitude/latitude or differs from projected sample coordinates by a fixed offset, use:

```bash
python scripts/08_prepare_prediction_grid_coordinates.py \
  --sample-coordinates sample_points.csv \
  --prediction-grid prediction_grid.csv \
  --sample-x Easting --sample-y Northing \
  --output aligned_prediction_grid.csv
```

The script can project geographic coordinates to the local UTM zone and estimate translation-only alignment. It does not rotate, scale, or deform the prediction grid. Spatial overlay and the generated JSON diagnostics must be checked before NNDM fold generation.

## Configuration files

- `config_example.json`: portable template for user-provided imagery, sample coordinates, and prediction grids.
- `config_paper_template.json`: manuscript-scale settings, including the archived anonymised TN fold plan and 100 repeated ten-fold runs.

## Data scope

The release does not contain raw UAV orthomosaics, raw RTK coordinates, or the complete spatial prediction grid. The current anonymised five-feature table and archived NNDM fold plan are sufficient to reproduce the final TN GBDT result and the fixed-parameter six-model comparison. The remaining modules implement the full workflow for application to user-provided spatial data.

## Citation and contact

When using this repository, please cite the associated manuscript, **“Multi-scale UAV imagery for estimating and mapping soil properties in a citrus orchard with fragmented bare-soil exposure,”** together with the NNDM method paper where relevant. Author contact information is provided in the manuscript.

## License and rights

No open-source software license is included in this v1.0.0 public release. The repository is provided for scientific transparency and reproducibility. Copyright and patent rights are not waived by public availability of the code. Users who require reuse or redistribution rights beyond those provided by applicable law should obtain permission from the relevant rights holders.
