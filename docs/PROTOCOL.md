> **Public copy.** This file is the local CPC protocol from the 2026-11-07 consistency review, including the scoring erratum at the top. It was written as a local freeze before the 2025 scores were read. It is not a public preregistration and it has no external registry record.
>
> Read the erratum before the body. The body still describes the first interval (200 draws, last five days outside the block index). The scored interval used for the numbers in this repository is the corrected one: circular 10-day blocks, every replicate trimmed to all 365 test days, 400 draws, seed 20261107. Machine-readable copy: [`results/results.json`](../results/results.json) field `bootstrap`.
>
> `src/lib/qimen/district-coords.json` is not in this public repository. The cells that rule produced are [`results/lattice_representatives.csv`](../results/lattice_representatives.csv) (272 representatives) and [`results/cells.csv`](../results/cells.csv) (267 comparison cells). Fitting code lives in a separate private repository and is not vendored here.

# Frozen NOAA CPC protocol (written before the 2025 scores)

Erratum after scoring, 2026-11-07. The sample rule, split, and hyperparameters below were not changed. Two scoring details were wrong in the first write-up and are corrected without refitting a new model. First, F1 must be 2TP/(2TP+FP+FN) on every cell; a cell with wet days and no wet forecast has F1 0. The first macro F1 dropped those cells because precision was undefined. Second, the interval now uses circular blocks of length 10 trimmed to all 365 test days, with 400 draws. The first interval omitted the last five calendar days while the point estimate used 365. This file is a local protocol, not a public preregistration.

This is a new computational experiment on the public NOAA CPC Unified Gauge daily grid, product title `CPC GLOBAL PRCP V1.0`. It is not a China Meteorological Administration station study, and it is not a first-hand raingauge observation at the cell center.

## Window and splits

- Download and analysis window: 2023-01-01 through 2025-12-31.
- Train 2023, validation 2024, test 2025.
- Hyperparameters and the secondary threshold are chosen on 2024 only.
- Coefficients are fit on 2023 only. Validation rows are not added back into the fit.
- 2025 is scored once after that choice.
- The repository’s older district artifact already reported a test window that includes 2025-01-01 through 2026-08-27. This rerun uses newly downloaded CPC grids and a different spatial sample. It is not an untouched external test.
- If 2026-01-01 through 2026-11-05 can be downloaded afterwards, those days are a supplement. They do not replace 2025 and are not used to pick a model or a window.

## Space

- NCSS bbox is 18–54°N, 73–135°E. That rectangle contains neighboring countries and sea. It is not the China sample.
- Returned cell centers outside that closed range are dropped.
- A cell enters the candidate set only when its center is within 60 km of a centroid in `src/lib/qimen/district-coords.json` (the app’s Aliyun DataV administrative list, 3245 places). This is not an official boundary. Coastal and border cells can fall slightly outside a surveyed coastline.
- From those candidates, keep one cell per 2°×2° block: the candidate closest to the block center, ties broken by lower latitude then lower longitude. The result is a lattice, not every 0.5° land cell and not 3143 counties.
- A cell with any non-finite value, any value outside [0, 1000] mm, or any CPC missing marker in the three-year window is removed. Those days are not filled with 0.
- Cells with only one wet/dry class in 2023 cannot fit a logistic or tree model. The comparison table uses the cells that have both classes in 2023. Always-dry cells therefore do not inflate accuracy.

## Label, time, and features

- Wet day: CPC daily total ≥ 0.1 mm. The file unit is mm and `cell_methods` is `time: sum`.
- The CF stamp is hours since 1900-01-01 00:00:00, gregorian. The experiment joins that calendar date to the Qimen plate. The attributes do not establish a Beijing-time observing day. A ±1 calendar-day shift of the plate is reported on 2024 only and cannot replace the same-date join.
- Features are the existing noon chart (hour 12, minute 0) from `tyme4ts` 1.5.2, first 31 columns: eight gods, eight gates, nine stars, yin/fuyin/fanyin/empty-kan, and day-of-year sine and cosine. The plate is shared by every cell on a date.
- Previous-day persistence uses only the preceding cell’s wet/dry label. Transition rates are estimated on 2023 with Laplace smoothing `(k+1)/(n+2)`. It does not use same-day precipitation as a covariate.
- Month and meteorological-season rates (DJF/MAM/JJA/SON) are also Laplace-smoothed on 2023.

## Models

Same cells, same days. One hyperparameter for all cells, chosen by the mean 2024 log loss. Ties keep the earlier grid item. Seed 20261107. No class weight. No Platt scaling.

- Training wet-day rate
- Month climatology
- Season climatology
- Season-only L2 logistic, C in {0.1, 1, 10, 100}
- Qimen plus season, same C grid
- Qimen only, same C grid
- Random forest, four pre-listed trees, `n_jobs=1`
- Histogram gradient boosting, four pre-listed settings, early stopping off
- Lead-1 persistence

Primary hard threshold is 0.5. A second threshold maximizes the 2024 macro CSI and is then applied once. It does not choose the model.

## Metrics and uncertainty

Macro-average across cells is the primary summary: accuracy, balanced accuracy, F1, CSI, Brier, and log loss at threshold 0.5. ROC-AUC is sklearn’s ranking AUC. PR-AUC is sklearn average precision, not the trapezoid. Undefined ratios stay null.

The 95% interval resamples 10-day blocks of the test calendar (200 draws, seed 20261107). The last five days of the 365-day year are outside the block index; the point estimate still uses all 365 days. This is not an interval that pretends cell-days are independent, and it is not a calendar 旬 score. Ten-day block accuracy is secondary: a block is wet if at least five days are wet, predicted wet if the mean probability is at least 0.5, stub dropped.

Latitude bands and meteorological seasons are descriptive cuts of the same predictions. No band is selected because its score is higher.

A per-cell model is not a forecast for a cell that was not fit. This lattice does not estimate skill at an unseen location.

Three years, with a single training year, do not support a claim about long climate cycles. No human-outcome labels are used.
