# Qimen features and CPC wet-day probability

**Author:** Haopeng Li (李浩鹏), City University Malaysia (马来西亚城市大学).
**Repository owner:** [Gideon-li](https://github.com/Gideon-li).
**Run:** `cpc-7276563d90f15c12`, scored 2026-11-07.

This repository documents one computational experiment. Noon Qimen Dunjia symbolic features are compared with seasonal baselines, and with a few other models, as predictors of whether a day is wet on a lattice of NOAA CPC precipitation cells.

The manuscripts in [`manuscript/`](manuscript/) are unpublished drafts. Nothing here is a claim of journal acceptance.

Fitting code and the private application that produced the noon charts live in a **separate private repository** (the drafts name an isolation pull request). This public repository is the analysis record: protocol, evidence, metrics, figures, cell lists, a small prediction archive, and instructions for re-downloading the CPC subsets. It does not include production model weights, administrative-centroid source files, or raw NetCDF.

## Question

On these cells, and for this split, do 31 noon Qimen features lower the Brier score of a wet-day probability once a per-cell seasonal curve is already available?

Wet day means the CPC daily total is at least 0.1 mm. The noon chart (hour 12, minute 0, `tyme4ts` 1.5.2) is joined to the CPC calendar date of that same daily total. That is a same-day classification. It is not a forecast issued before the rain day.

## Data

Labels are the **NOAA CPC Global Unified Gauge-Based Analysis of Daily Precipitation**, retrieved as monthly subsets from **NOAA PSL THREDDS NCSS** (`precip.YYYY.nc`). File attributes on the retrieved subsets: `title=CPC GLOBAL PRCP V1.0 RT`, `dataset_title=CPC GLOBAL PRCP V1.0`, `version=V1.0`, units mm, `cell_methods=time: sum`.

The product page is <https://www.psl.noaa.gov/data/gridded/data.cpc.globalprecip.html>. Retrieval steps, the exact URL template, container sizes, and SHA-256 pins are in [`data/README.md`](data/README.md). The 36 monthly containers for 2023–2025 are about 38 MiB together. They are not committed. `*.nc` is gitignored.

These values are a gridded gauge **analysis**. The analysis sample is not a China Meteorological Administration station series, and it is not a set of rain-gauge readings at the cell centers. NASA POWER is not used as a label.

The time axis in the files is `hours since 1900-01-01 00:00:00`, gregorian. The experiment uses that calendar date. The attributes do not establish a Beijing-time observing day.

Real-time CPC files are revised. Scores are pinned to the bytes downloaded on 2026-11-07. A later NCSS container can carry a different SHA-256 because NCSS stamps a translation time; see [`data/sample_verify.json`](data/sample_verify.json).

## Sample

Download bbox: 18–54°N, 73–135°E. That rectangle includes neighboring countries and sea. It is not the China sample.

Cells were then restricted to centers within 60 km of a project administrative centroid, thinned to one cell per 2°×2° block, and dropped if any day in 2023–2025 was missing or out of range. Missing days were not filled with 0.

| Step | Removed at this step | Remaining |
| --- | ---: | ---: |
| Centers inside 18–54°N, 73–135°E | — | 8928 |
| Within 60 km of a project administrative centroid | 6142 | 2786 |
| One cell per 2° block | 2514 | 272 |
| Any invalid day in 2023–2025 | 5 | 267 |
| Only one wet/dry class in 2023 | 0 | 267 |

Source: [`results/exclusion_funnel.csv`](results/exclusion_funnel.csv). The five removed representatives are missing on all 1,096 days: 21.25°N 108.75°E, 21.75°N 114.25°E, 29.25°N 122.25°E, 30.75°N 122.75°E, 35.75°N 120.75°E. They are one group of five, not a second drop of five single-class cells. All 267 kept cells have both classes in 2023.

The 60 km rule is a neighborhood of the project’s administrative centroids, then a lattice. It is not a surveyed national land mask, not every Chinese county, and not every 0.5° land cell. Per-cell coefficients do not apply to a cell that was not fit. The centroid file itself remains in the private repository. The resulting lists are [`results/lattice_representatives.csv`](results/lattice_representatives.csv) (272 rows) and [`results/cells.csv`](results/cells.csv) (267 rows).

## Split

| Role | Year | Days | What it is allowed to do |
| --- | --- | ---: | --- |
| Train | 2023 | 365 | Fit coefficients and climatological rates |
| Validation | 2024 | 366 | Choose the penalty and the tree settings by mean log loss. Choose a secondary probability threshold by macro CSI. Validation rows are not added back into the fit. |
| Test | 2025 | 365 | Scored once after that choice |

Year 2025 overlaps a test window already reported in a private legacy district artifact (2025-01-01 through 2026-08-27). This run uses a new CPC download and a different spatial sample. **It is not an untouched external test.** The protocol was frozen locally before the 2025 scores were read. There is no external preregistration record.

A 2026-01-01 through 2026-11-05 supplement (309 days) was scored afterwards with the same frozen settings. It was not used to pick a model, a threshold, or the window, and it does not replace 2025. Brier order matches 2025 (season-only 0.1955, Qimen-plus-season 0.1993, lead-1 persistence 0.1821).

## Features and models

The same 267 cells and the same days enter every model. The feature matrix is 31 columns: eight gods, eight gates, nine stars, yin / fuyin / fanyin / empty-kan, and day-of-year sine and cosine. The extractor can emit 64 columns; column 32 (`年支合地盘`) and the rest are not in the matrix. The noon plate is shared by every cell on a date. Seed 20261107. No class weight. No probability recalibration.

Settings chosen on 2024 log loss, then frozen: season-only logistic `C=1`; Qimen-plus-season `C=0.1`; Qimen-only `C=0.1`; random forest 200 trees, depth 4, minimum leaf 20; histogram boosting learning rate 0.05, depth 3, 100 iterations, early stopping off. Primary hard threshold is 0.5.

Month and meteorological-season rates are Laplace-smoothed on 2023. Lead-1 persistence uses yesterday’s observed wet/dry label and does not use same-day precipitation. In the archived probability file those conditional probabilities are two global values, shared by every cell (about 0.1913 after a dry day and 0.6998 after a wet day). The written protocol also describes cell-wise Laplace rates. The stored array does not vary by cell. Details: [`predictions/README.md`](predictions/README.md). Persistence remains a different information set from the calendar-feature models.

## Main result (2025 test)

Lower Brier and log loss are better. The primary comparison is Qimen-plus-season against the season-only L2 logistic.

| Model | Brier |
| --- | ---: |
| Season-only L2 logistic | **0.1923** |
| Qimen + season L2 | **0.1970** |
| Difference (Qimen+season − season-only) | **+0.0047** |

Circular 10-day block bootstrap, each replicate covering all 365 test days, 400 draws, seed 20261107. The 95% interval for that Brier difference is **+0.0033 to +0.0061**. The Qimen model’s Brier is higher on this test. The same direction was already present on 2024, before the test was scored (validation log loss 0.5782 for season-only, 0.5865 for Qimen-plus-season).

Every latitude band used in the report (south of 28°N, 28–35, 35–42, and 42°N and above) and every meteorological season has a higher Brier for Qimen-plus-season than for the season-only logit. Qimen-only stays near the training-rate baseline (Brier 0.2216 vs 0.2218). Random forest (0.2028) and histogram boosting (0.2105) use the same 31 columns and do not beat the season-only logit on Brier or log loss.

Lead-1 persistence has the lowest Brier (0.1793). It sees yesterday’s rain occurrence. The calendar models were not given that label, so persistence is not a nested improvement of the Qimen model and is not “the best model on the same inputs.”

Full macro metrics, all **267/267** cells, threshold 0.5. F1 is `2TP/(2TP+FP+FN)` from each cell’s confusion counts. A cell that predicts no wet day has F1 0 and stays in the average. Rounded from [`results/metrics_test_2025.csv`](results/metrics_test_2025.csv):

| Model | Accuracy | Balanced accuracy | Precision | Recall | F1 | CSI | Brier | Log loss |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Training wet-day rate | 0.6239 | 0.5000 | 0.1341 | 0.2434 | 0.1723 | 0.1341 | 0.2218 | 0.6315 |
| Month climatology | 0.6973 | 0.6075 | 0.5105 | 0.4386 | 0.4568 | 0.3329 | 0.1981 | 0.5816 |
| Season climatology | 0.6861 | 0.5876 | 0.4491 | 0.3730 | 0.3921 | 0.2831 | 0.1995 | 0.5816 |
| Season-only L2 logistic | 0.7042 | 0.6109 | 0.4762 | 0.4289 | 0.4415 | 0.3286 | 0.1923 | 0.5656 |
| Qimen + season L2 | 0.6938 | 0.5960 | 0.5229 | 0.4033 | 0.4242 | 0.3092 | 0.1970 | 0.5769 |
| Qimen only L2 | 0.6283 | 0.5114 | 0.3572 | 0.2772 | 0.2699 | 0.1873 | 0.2216 | 0.6315 |
| Random forest | 0.6858 | 0.5763 | 0.4646 | 0.3490 | 0.3597 | 0.2628 | 0.2028 | 0.5904 |
| Histogram boosting | 0.6777 | 0.5958 | 0.5407 | 0.4438 | 0.4676 | 0.3316 | 0.2105 | 0.6194 |
| Lead-1 persistence | 0.7615 | 0.7245 | 0.6652 | 0.6639 | 0.6646 | 0.5093 | 0.1793 | 0.5432 |

Histogram boosting has the highest F1 among models that do not see yesterday’s label (0.4676 vs 0.4415 for season-only) because it issues more wet forecasts. Its Brier is worse. F1 at threshold 0.5 and Brier answer different questions.

### Corrected F1

An earlier macro F1 averaged only cells where precision was defined. For the training-rate baseline that kept 65 cells and dropped 202 cells that forecast dry on every test day, and it produced **0.7077**. That figure is withdrawn. The corrected training-rate F1, over all 267 cells, is **0.1723**. The old numbers are retained only under `deprecated_test_metrics_nanmean_f1` in [`results/results.json`](results/results.json). Accuracy, balanced accuracy, recall, CSI, Brier, log loss, and ROC-AUC were unchanged by that correction. Do not quote 0.7077 as a result of this experiment.

Pooled ROC-AUC can rank a wet climate above a dry climate even when the model is a constant local rate. The training-rate model has pooled ROC-AUC 0.6582 and macro ROC-AUC 0.5000. Prefer the macro column when the question is discrimination inside cells. Both columns are in the CSV.

## Figures

| Figure | What it shows |
| --- | --- |
| [`figures/metrics_test_2025.png`](figures/metrics_test_2025.png) | 2025 macro balanced accuracy and macro Brier for the nine models on the same cells. |
| [`figures/calibration_test_2025.png`](figures/calibration_test_2025.png) | Reliability diagram pooling cell-days. Month rates and the season logit stay nearer the diagonal. Histogram boosting bends off it. Persistence is not drawn on this figure. |
| [`figures/brier_delta_map_2025.png`](figures/brier_delta_map_2025.png) | Per-cell Brier(Qimen+season) − Brier(month climatology). Red means the Qimen logit is worse at that cell. The color scale is clipped at ±0.05 for display. [`results/cells.csv`](results/cells.csv) is unclipped. This map is the contrast against the **month** rate. The primary numerical contrast in the table above is against the **season-only logit**. |

## What the results support, and where they stop

Supported, for this design: adding the 31 noon Qimen columns to a per-cell seasonal logistic does not improve 2025 wet-day Brier. The estimated change is positive (worse) and the block-bootstrap interval stays above zero.

The same files do not support a national, all-county, or all-grid conclusion; a never-touched external test; a Beijing-time observation day; a CMA product; a first-hand station series; a long-period climate conclusion from a single training year; or any statement about marriage, wealth, health, examinations, or other human outcomes. No human-outcome labels were used.

Older district-artifact headlines (for example a stored accuracy near 67.95%, or a ten-day hit rate near 79%) are a different sample and a different score. They are not the numbers in the table above. The secondary 10-day block hit rate in this run (36 blocks, 5 days dropped, not a traditional calendar 旬) is 0.780 for the season-only logit and 0.770 for Qimen-plus-season.

## Repository layout

```text
README.md                  this file
LICENSE                    MIT, for the notes and scored outputs the author can license
CITATION.cff               citation for this analysis record
docs/PROTOCOL.md           local protocol, with the scoring erratum
docs/PAPER_EVIDENCE.md     methods and results note used to draft the manuscripts
results/metrics_test_2025.csv
results/results.json       full machine-readable scores, intervals, and hashes
results/exclusion_funnel.csv
results/cells.csv          267 comparison cells and per-cell Brier
results/lattice_representatives.csv
results/logs/              training selection log and the metric-correction log
figures/                   the three PNG figures
manuscript/                Chinese and English Word drafts, v2
data/README.md             how to re-download the CPC subsets
predictions/               float32 npz of 2025 probabilities, with SHA-256
environment/               requirements, environment.json, download_manifest.json
notes/pre_submission_checklist_v2.txt
```

Logs still contain private worktree paths (`/workspace/experiments/cma-qimen-precip/...`). They are kept as the original run log. The first training log also shows the `nanmean` warning that produced the withdrawn F1; the recomputation log is the corrected scoring pass. Brier matched the first pass to 1×10^−8, so the probabilities did not move when F1 was corrected.

`environment/environment.json` records Python 3.12.3, numpy 2.5.4, scikit-learn 1.5.2, netCDF4 1.7.2, and `tyme4ts` 1.5.2. It also records a SHA-256 of production weights that were **not** changed and are **not** in this repository.

## Re-download and cite

Follow [`data/README.md`](data/README.md). Cite NOAA/PSL for the grids, plus the method papers the CPC documentation lists:

- Xie, P., A. Yatagai, M. Chen, T. Hayasaka, Y. Fukushima, C. Liu, and S. Yang, 2007: A gauge-based analysis of daily precipitation over East Asia. *J. Hydrometeor.*, **8**, 607–626. <https://doi.org/10.1175/JHM583.1>
- Chen, M., W. Shi, P. Xie, V. B. S. Silva, V. E. Kousky, R. W. Higgins, and J. E. Janowiak, 2008: Assessing objective techniques for gauge-based analyses of global daily precipitation. *J. Geophys. Res.*, **113**, D04110. <https://doi.org/10.1029/2007JD009132>

Page range 607–626 follows the AMS landing page for Xie et al. (2007). A software citation for this repository is in [`CITATION.cff`](CITATION.cff).

## 简要说明

本仓库记录的是 NOAA CPC 全球统一雨量计日降水分析（V1.0，经 PSL NCSS 下载）上的一次对照实验，作者李浩鹏，马来西亚城市大学。数据不是中国气象局站点观测，也不是格点中心的雨量筒读数。

分析样本是 267 个代表格：外包矩形 8928 个格点，距项目行政中心不超过 60 km 的有 2786 个，每 2° 保留一格得到 272 个，再去掉 5 个全程缺测的代表格。这不是全国每个区县，也不是严格国境掩码。训练年为 2023，验证年为 2024，测试年为 2025。2025 年与旧的私有工件已经报告过的测试窗重叠，不能写成从未使用的外部测试。

主结果：正午奇门 31 个符号特征没有在逐格季节曲线之外降低湿日概率的 Brier 分数。仅季节 L2 为 0.1923，奇门加季节为 0.1970，差值 +0.0047；长度 10 日的环形块 bootstrap 共 400 次，95% 区间为 +0.0033 至 +0.0061。前日持续性使用的是昨日实测干湿，信息集不同，Brier 为 0.1793。F1 使用全部 267 格。已废止的 nanmean F1（例如训练雨日率的 0.7077）不是主结果；该基线修正后的 F1 是 0.1723。

手稿为 v2 草稿，不表示已被期刊接收。本仓库不评价人事占测。相关拟合代码在另一个私有仓库，这里只公开分析记录。

## License

[MIT](LICENSE) for this repository’s documentation and scored outputs. NOAA CPC data keep NOAA’s terms; attribution of NOAA PSL is requested by the data provider. The MIT license does not assert an institutional ownership claim beyond the author line above.
