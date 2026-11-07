# 2025 test predictions

File: [`test_2025_predictions_float32.npz`](test_2025_predictions_float32.npz)

| | |
| --- | --- |
| Size | 1,561,812 bytes |
| SHA-256 | `1d458d9187846c6f579983dff1278846933276938eb9be8f231f2b5d2b62440f` |

The same digest is stored in `results/results.json` as `prediction_float32_sha256`. The private worktree note said this file was not in git. It is committed here because it is a small float32 archive of scores, not the raw NetCDF.

NumPy compressed archive, ten arrays, each `float32` with shape **(365, 267)**:

| Key | Contents |
| --- | --- |
| `y` | Observed wet-day label, 0 or 1. Column means match `test_wet_rate` in `results/cells.csv` to the four decimals stored there. Overall mean 0.400749. |
| `train_wet_rate` | Training-period wet-day rate, constant in time within a cell |
| `month_climatology` | Month climatology |
| `season_climatology` | Meteorological-season climatology (DJF, MAM, JJA, SON) |
| `season_only_l2` | Season-only L2 logistic probability |
| `qimen_plus_season_l2` | Qimen (31 columns) plus season, L2 logistic probability |
| `qimen_only_l2` | Qimen columns only, L2 logistic probability |
| `random_forest` | Random forest probability |
| `hist_gradient_boosting` | Histogram gradient boosting probability |
| `lag1_persistence` | Lead-1 persistence probability |

Axis 0 is the saved 2025 test sequence (365 days). The archive has no date coordinate. On rows 1 through 364, `lag1_persistence` is a function of `y` on the previous row, which is consistent with chronological order inside the file. Row 0 depends on the label before 2025 and is not determined by `y` alone.

Axis 1 matches **`results/cells.csv` row order**. Checked here: per-cell mean of `(qimen_plus_season_l2 − y)^2` and of `(month_climatology − y)^2`, rounded to four decimals, match `brier_full` and `brier_month`, and per-cell means of `y` match `test_wet_rate`.

Macro Brier recomputed from these arrays, mean over cells of the mean squared error over days, agrees with `results/metrics_test_2025.csv` to about 1×10^−7, which is the size expected from float32 storage. The published table was not replaced by this recomputation.

## Persistence values actually stored

`lag1_persistence` contains **two** distinct float32 values, shared by all 267 cells:

| Previous day’s observed label | Stored probability | Count of cell-days |
| --- | ---: | ---: |
| Dry (`y` of the previous row = 0), rows 1–364 | 0.19126098 | 58,219 |
| Wet (`y` of the previous row = 1), rows 1–364 | 0.69983357 | 38,969 |

Row 0 uses the same two numbers (245 cells at 0.19126098 and 22 cells at 0.69983357).

The protocol text describes cell-wise Laplace transition rates estimated on 2023, `(k+1)/(n+2)`. The array in this file does not vary by cell. The 2023 counts are not in this repository, so a per-cell table cannot be reconstructed from the public files. The published persistence Brier (0.1793) matches this two-value field. Persistence still uses yesterday’s observed wet/dry label. That is a different information set from the calendar-feature models, which do not see that label.
