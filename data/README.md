# Data

| Folder | Contents | Produced by |
|---|---|---|
| `raw/` | `gll_psc_v35.fit` (Fermi-LAT 4FGL-DR4 source catalog), `table_4LAC.fits` (4LAC-DR4 table with redshifts) | Downloaded from the public Fermi-LAT / FSSC archive |
| `interim/` | Blazar feature table, and the training / target split | `notebooks/01_data_extraction.ipynb` |
| `processed/` | Engineered features (log10 fluxes, hardness ratios) | `notebooks/02_redshift_estimation_and_analysis.ipynb` |

## Sources

- 4FGL-DR4: Ballet et al. 2023, [arXiv:2307.12546](https://arxiv.org/abs/2307.12546)
- 4LAC-DR3/DR4: Ajello et al. 2022, ApJS 263, 24

Catalogs are public data products of the Fermi-LAT Collaboration. Cite the original papers when reusing them.

## Splits

- **Training set** (2,777 rows): blazars (BL Lac, FSRQ, BCU) matched to 4LAC. Rows without a valid redshift (non-finite after the cross-match) are dropped before training, leaving 1,519.
- **Target set** (1,156 sources): blazars with no redshift (780 BCU, 226 BL Lac, 150 FSRQ).

Redshift predictions for the target set are in [`../results/`](../results/).
