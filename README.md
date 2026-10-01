# Redshift (Distance) Estimation for BL Lacs with Machine Learning

Many BL Lac blazars have featureless optical spectra, so their redshifts cannot be measured by optical spectroscopy.
This project estimates redshifts for such sources from **Fermi-LAT gamma-ray photometry alone**, using an XGBoost regressor trained on blazars with known redshifts. The model relies on the imprint of Extragalactic Background Light (EBL) absorption on the high-energy spectrum.

The predicted redshifts are then used for four follow-up analyses: the blazar sequence, cosmological evolution (V/Vmax), VHE target selection, and EBL/IGMF probes.

## Pipeline

```
 4FGL-DR4 + 4LAC            feature engineering             XGBoost regression
 (FITS catalogs)  ───────►  log10 fluxes, hardness  ───────►  train on known z,
 filter blazars,            ratios between adjacent           predict z for sources
 split by known z           energy bands                      without a redshift
                                                                    │
        ┌───────────────────────────────────────────────────────────┘
        ▼
 Blazar sequence · V/Vmax evolution · VHE targets · EBL / IGMF probes
```

| Notebook | Purpose |
|---|---|
| [01_data_extraction.ipynb](notebooks/01_data_extraction.ipynb) | Read 4FGL-DR4, select blazar classes, cross-match with 4LAC, split into training and target sets |
| [02_redshift_estimation_and_analysis.ipynb](notebooks/02_redshift_estimation_and_analysis.ipynb) | Feature engineering, model training/validation, inference, and the five science objectives |

## Results

| Metric (20% hold-out, ≈300 sources) | Value |
|---|---|
| RMSE | 0.557 |
| R² | 0.318 |

| Objective | Result | Output |
|---|---|---|
| Distance estimation | Predicted z for 1,156 sources without a measured redshift | [results/featureless_bllacs_with_z.csv](results/featureless_bllacs_with_z.csv) |
| Blazar sequence | Intrinsic γ-ray luminosity vs. spectral index | [results/featureless_bllacs_with_luminosity.csv](results/featureless_bllacs_with_luminosity.csv) |
| Cosmological evolution | ⟨V/Vmax⟩ = 0.272 (below the 0.5 expected for no evolution) | [results/featureless_bllacs_v_vmax.csv](results/featureless_bllacs_v_vmax.csv) |
| VHE target selection | 2 candidates with z < 0.2 and Γ ≤ 1.9 | [results/vhe_observing_targets.csv](results/vhe_observing_targets.csv) |
| EBL / IGMF probes | Distance and look-back-time catalog for cascade simulations | [results/cosmological_probes_igmf_ebl.csv](results/cosmological_probes_igmf_ebl.csv) |

Figures are in [`figures/`](figures/).

### Limitations

- **Small training sample.** Only 1,519 of the 2,777 labelled sources have a valid redshift and are used for training.
- **Modest predictive power.** R² ≈ 0.32 means predicted redshifts are useful for population-level statistics, not for individual sources.
- **Covariate shift.** The training set is FSRQ-heavy, whereas the target set is mostly BL Lac-like. Predicted redshifts for the target set have a median of z ≈ 0.79 (5th-95th percentile 0.31-1.68), higher than is typical for BL Lacs, which is consistent with the model inheriting the FSRQ-dominated training distribution. Downstream results (V/Vmax, luminosities) inherit this bias.
- **Target set composition.** Of the 1,156 targets, 780 are classified as BCU (blazar of uncertain type) and only 226 as BL Lac.
- **V/Vmax** uses an empirical flux limit taken from the target set, which is an approximation of the true Fermi-LAT sensitivity.

## Getting started

```bash
git clone https://github.com/astrophile73/Redshift-Distance-Estimation-for-BL-Lacs-with-ML.git
cd Redshift-Distance-Estimation-for-BL-Lacs-with-ML

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter lab notebooks/
```

Run the notebooks in order, from inside the `notebooks/` directory (paths are relative to it). The raw catalogs are included in `data/raw/`, and every intermediate and output file is already committed, so notebook 02 can be run without re-running 01.

## Repository layout

```
├── notebooks/     analysis notebooks (run in order)
├── data/
│   ├── raw/       Fermi-LAT 4FGL-DR4 and 4LAC FITS catalogs
│   ├── interim/   cleaned tables, training / target split
│   └── processed/ engineered features
├── models/        trained XGBoost model (native JSON)
├── results/       predicted redshifts and downstream catalogs
├── figures/       output plots
├── requirements.txt
└── LICENSE
```

## Data and acknowledgements

This work uses public data from the Fermi Large Area Telescope Collaboration (4FGL-DR4 and 4LAC). See [data/README.md](data/README.md) for references.

## License

Code is released under the [MIT License](LICENSE). Fermi-LAT catalogs remain the property of their respective collaboration and are subject to their own terms of use.
