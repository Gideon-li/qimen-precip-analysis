> **Public copy.** The text below is the CPC evidence note dated with the 2026-11-07 scoring run (`run_id` `cpc-7276563d90f15c12`). Paths inside it that begin with `experiments/cma-qimen-precip/`, `/workspace/`, or `data/cpc/` refer to the private worktree where the fit was run. In this public repository the same artifacts are:
>
> | Private path mentioned below | This repository |
> | --- | --- |
> | `experiments/cma-qimen-precip/runs/cpc-2023-2025/results.json` | [`results/results.json`](../results/results.json) |
> | metrics table | [`results/metrics_test_2025.csv`](../results/metrics_test_2025.csv) |
> | `exclusion_funnel.csv` | [`results/exclusion_funnel.csv`](../results/exclusion_funnel.csv) |
> | cell and lattice lists | [`results/cells.csv`](../results/cells.csv), [`results/lattice_representatives.csv`](../results/lattice_representatives.csv) |
> | `download_manifest.json` | [`environment/download_manifest.json`](../environment/download_manifest.json) |
> | `data/cpc/test_2025_predictions_float32.npz` (originally noted as not in git) | [`predictions/test_2025_predictions_float32.npz`](../predictions/test_2025_predictions_float32.npz) |
>
> The runtime git HEAD stored in `results.json` is `6fa07b5bbc4d76f995077d045f6cf97e7b2ee102`, with a dirty worktree. The v2 manuscripts and [`notes/pre_submission_checklist_v2.txt`](../notes/pre_submission_checklist_v2.txt) name private commit `4e55230a1bd84ae26495cf67118e8254af4dc8fe` and a private pull request. This public repository does not contain that code and does not contain production weights. The weight SHA-256 in `environment.json` is an integrity note that those weights were not retrained; the weight file is absent.
>
> One archive check that is not in the original note: `lag1_persistence` in the public npz takes only two distinct values, about 0.191261 after a dry previous day and about 0.699834 after a wet previous day, and those two values are shared by every cell. Macro Brier recomputed from the npz matches the published table to about 1e-7. The paragraph below still says the protocol estimated cell-wise Laplace rates. Both records are kept. This repository does not include the 2023 transition counts, so it cannot display a per-cell rate table.

# Evidence for the manuscript

Two different claims must stay in different paragraphs.

The production district weights are still the old NOAA-labelled artifact. This run did not retrain them. The China Meteorological Administration files are still absent, so nothing here is a CMA result. The numbers below are a new download of NOAA CPC Unified Gauge daily precipitation, scored on a lattice whose membership was fixed in this repository before the 2025 scores were read. That local freeze is not a public preregistration: there is no external registry record. They are gridded gauge analyses, not raingauge readings at cell centers, and not a national census of every 0.5° land cell.

Run id: `cpc-7276563d90f15c12`, the SHA-256 prefix of the protocol id, the selected precipitation bytes, and the Qimen feature file. The fit was executed on 2026-11-07. Machine-readable copy: `experiments/cma-qimen-precip/runs/cpc-2023-2025/results.json`. The runtime git HEAD recorded inside that file is the parent of the commit that adds these scripts; the worktree was dirty because the CPC code had not been committed yet. The commit that contains both the code and this results file is the one to cite.

## Methods you can write

**Product.** NOAA CPC Global Unified Gauge-Based Analysis of Daily Precipitation, file attributes `title=CPC GLOBAL PRCP V1.0 RT`, `version=V1.0`, `dataset_title=CPC GLOBAL PRCP V1.0`, units mm, `cell_methods=time: sum`, missing value about −9.97×10^36. Distribution: NOAA PSL THREDDS NCSS, `precip.YYYY.nc`. Reference page stored in the file: https://www.psl.noaa.gov/data/gridded/data.cpc.globalprecip.html . Source path stored in the file: `ftp://ftp.cpc.ncep.noaa.gov/precip/CPC_UNI_PRCP/`. Method papers commonly cited for this family are Xie et al. (2007, J. Hydrometeorology) and Chen et al. (2008, J. Geophys. Res.). The retrieved 2023 monthly file says `history=Updated 2024-01-02`. Real-time CPC files are revised; this experiment is pinned to the bytes downloaded on 2026-11-07, not to a later revision.

**Retrieval.** Serial monthly NCSS requests, bbox 18–54°N, 73–135°E, 2023-01-01 through 2025-12-31 (1096 days). Thirty-six files. A Beijing-area sample for 2025-07-01–07 was downloaded again first. Its NetCDF container SHA-256 differs from an earlier retrieval because NCSS stamps a translation time; the seven values at 39.75°N, 116.25°E matched the earlier decode (3.04322934, 7.38294697, 1.78654253, 4.37161636, 0.82105678, 0.05515119, 0.88242400 mm). Hashes of the monthly containers are in `download_manifest.json`. Invalid values were set to missing. None were replaced with 0.

**Time.** The coordinate is `hours since 1900-01-01 00:00:00`, gregorian. The experiment uses the resulting calendar date. The attributes do not say that this is a Beijing-time 00:00–24:00 or 20:00–20:00 total. Gauge days inside CPC can follow the contributing networks. The Qimen plate is the repository’s noon chart (hour 12, minute 0, `tyme4ts` 1.5.2) on that same date string. A one-day shift of the plate, refit at the already chosen penalty and scored only on 2024, changed validation log loss from 0.58650 to 0.58588 (previous date) and 0.58510 (next date). The primary join was not changed.

**Sample.** The bbox has 8928 cell centers and includes foreign land and sea. It is not the analysis sample. Sequential counts, from `exclusion_funnel.csv`, are not additive in the way “272 − 5 − 5” would suggest:

| Step | Removed at this step | Remaining |
| --- | ---: | ---: |
| Centers inside 18–54°N, 73–135°E | — | 8928 |
| Within 60 km of a project administrative centroid | 6142 | 2786 |
| One cell per 2° block | 2514 | 272 |
| Any invalid day in 2023–2025 | 5 | 267 |
| Only one class in 2023 | 0 | 267 |

The five removed representatives are missing on all 1096 days (21.25°N 108.75°E, 21.75°N 114.25°E, 29.25°N 122.25°E, 30.75°N 122.75°E, 35.75°N 120.75°E). They have no observed wet day and no observed dry day. They are the same five cells, not a second group. Among the 267 complete cells, every cell has both classes in 2023. The 60 km rule is a neighborhood of the project’s administrative centroids (`district-coords.json`), then thinned to one cell per 2° block. It is not a surveyed national land mask and not a claim to cover every county or every CPC land cell. Per-cell coefficients do not apply to a cell that was not fit.

**Label and split.** Wet if the daily total is at least 0.1 mm. Train 2023 (365 days), validation 2024 (366 days), test 2025 (365 days). The 2025 mean wet-day rate on these cells is 0.4007. The 2023 mean is 0.3888. Penalty and tree settings minimize the mean 2024 log loss; ties keep the earlier grid point. Coefficients use 2023 only. Threshold 0.5 is primary. A second threshold maximizes 2024 macro CSI and is not used to pick the model. The old district artifact already used 2025-01-01–2026-08-27 as a reported test window, so 2025 here is **not** an untouched external test. It is a new download and a different spatial design.

**Features.** The extractor returns 64 columns. This experiment keeps the first 31 only, and those names were checked against the frozen list: eight gods, eight gates, nine stars, yin / fuyin / fanyin / empty-kan, and day-of-year sine and cosine. Column 32 in the full extractor is `年支合地盘`, a pillar feature, and it is not in the matrix. Every model uses this same 31-column matrix and the same 267 cells. Seed 20261107. No class weight and no probability recalibration. Settings were chosen on 2024 and were not chosen again for this correction: season-only logistic C=1; Qimen-plus-season C=0.1; Qimen-only C=0.1; random forest 200 trees, depth 4, minimum leaf 20; histogram boosting learning rate 0.05, depth 3, 100 iterations, early stopping off. The noon chart is joined to the CPC date label. That is a same-day classification of the daily total, not a forecast issued before the rain day, and it is not a deployable lead-time product. The metric correction refit these same settings; stored Brier matched the first run to 1e-8, so the probabilities did not move.

**Persistence.** For each cell, two Laplace rates from 2023, P(wet | yesterday wet) and P(wet | yesterday dry), with `(k+1)/(n+2)`. The 2025 forecast uses the observed label of the previous calendar day. It does not use same-day precipitation. Month and season climatologies use the same Laplace form. Season names are DJF, MAM, JJA, SON.

**Metrics.** Macro-average across all 267 cells, unless a denominator is truly zero. Threshold 0.5. F1 is 2TP/(2TP+FP+FN) from the cell’s confusion counts. A cell that has wet days but predicts none has F1 0 and stays in the average. F1 is undefined only when 2TP+FP+FN is 0; that did not occur in 2025, so every model’s F1 uses 267/267 cells. Precision is TP/(TP+FP), and is set to 0 when there is no predicted wet day; the count of those cells is reported and is not used to drop the cell from F1. Recall is TP/(TP+FN) and was defined for all 267 cells because each had at least one wet day. Balanced accuracy needs both classes; all 267 had both. CSI is TP/(TP+FP+FN) and was already defined for all 267; it did not change. The first published F1 of 0.7077 for the training-rate baseline is withdrawn. It averaged only the 65 cells that predicted wet and ignored the 202 cells that predicted dry. That old F1, and the old precision, are kept in `results.json` under `deprecated_test_metrics_nanmean_f1`. Accuracy, balanced accuracy, recall, CSI, Brier, log loss, and both ROC-AUC columns match the first run. ROC-AUC is the rank statistic. PR-AUC is scikit-learn average precision, not the trapezoid. Pooled AUC lets a constant local rate rank wet climates above dry ones, so the training-rate model has pooled ROC-AUC 0.6582 and macro ROC-AUC 0.5000. Intervals are a circular moving-block bootstrap, block length 10, each replicate trimmed to 365 days, 400 draws, seed 20261107. The point estimate is also the metric on all 365 days. The earlier interval dropped the last five calendar days; that mismatch is withdrawn. Raising the draw count from 200 to 400 changes only the percentile stability. The interval is time uncertainty for these fixed cells. It does not treat 267×365 cell-days as independent. The optional 10-day hit rate is not a calendar 旬.

## 2025 test results

Same 267 cells. Lower Brier and log loss are better.

| Model | Accuracy | Balanced accuracy | Precision | Recall | F1 | CSI | Brier | Log loss | ROC-AUC pooled | ROC-AUC macro |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Training wet-day rate | 0.6239 | 0.5000 | 0.1341 | 0.2434 | 0.1723 | 0.1341 | 0.2218 | 0.6315 | 0.6582 | 0.5000 |
| Month climatology | 0.6973 | 0.6075 | 0.5105 | 0.4386 | 0.4568 | 0.3329 | 0.1981 | 0.5816 | 0.7476 | 0.6772 |
| Season climatology | 0.6861 | 0.5876 | 0.4491 | 0.3730 | 0.3921 | 0.2831 | 0.1995 | 0.5816 | 0.7413 | 0.6681 |
| Season-only L2 logistic | 0.7042 | 0.6109 | 0.4762 | 0.4289 | 0.4415 | 0.3286 | 0.1923 | 0.5656 | 0.7628 | 0.7022 |
| Qimen + season L2 | 0.6938 | 0.5960 | 0.5229 | 0.4033 | 0.4242 | 0.3092 | 0.1970 | 0.5769 | 0.7497 | 0.6837 |
| Qimen only L2 | 0.6283 | 0.5114 | 0.3572 | 0.2772 | 0.2699 | 0.1873 | 0.2216 | 0.6315 | 0.6630 | 0.5438 |
| Random forest | 0.6858 | 0.5763 | 0.4646 | 0.3490 | 0.3597 | 0.2628 | 0.2028 | 0.5904 | 0.7354 | 0.6745 |
| Histogram boosting | 0.6777 | 0.5958 | 0.5407 | 0.4438 | 0.4676 | 0.3316 | 0.2105 | 0.6194 | 0.7259 | 0.6583 |
| Lead-1 persistence | 0.7615 | 0.7245 | 0.6652 | 0.6639 | 0.6646 | 0.5093 | 0.1793 | 0.5432 | 0.7516 | 0.7245 |

The training-rate row predicts wet for all 365 days in 65 cells and dry for all 365 days in the other 202. Its precision of 0.1341 averages those 202 zeros with the 65 cells that do predict wet. Its old precision 0.5507 and old F1 0.7077 are the averages after deleting those 202 cells. Histogram boosting has the highest F1 among models that do not see yesterday’s rain (0.4676), because it issues more wet forecasts, while its Brier (0.2105) is worse than the season-only logit (0.1923). F1 at 0.5 and Brier are different questions.

Circular 10-day-block intervals, 400 draws, full 365 days in every replicate:

- Brier, Qimen-plus-season minus month climatology: −0.00110, interval −0.00431 to +0.00232. The interval contains zero.
- Brier, Qimen-plus-season minus season-only logit: +0.00470, interval +0.00334 to +0.00614. The Qimen model’s Brier is higher.
- Balanced accuracy, Qimen-plus-season minus month climatology: −0.0115, interval −0.0191 to −0.0037.
- F1, Qimen-plus-season minus season-only logit: −0.0172, interval −0.0256 to −0.0074.

On 2024, before the test was scored, the selection log loss was already 0.5782 for the season-only logit and 0.5865 for Qimen-plus-season. The test does not reverse that.

Every latitude band (south of 28°N, 28–35, 35–42, and 42°N and above) and every meteorological season has a higher Brier for Qimen-plus-season than for the season-only logit. No band was picked after the fact. Qimen-only stays near the training-rate baseline. Random forest and histogram boosting, using the same 31 columns, do not beat the season-only logit on Brier or log loss. Lead-1 persistence, which sees yesterday’s wet/dry label, has the lowest Brier. The calendar models were not given that label, so this is a separate information set, not a nested increment.

The calibration plot pools cell-days. Month rates and the season logit stay nearer the diagonal. Histogram boosting bends off it. The map of cell Brier(Qimen+season) − Brier(month) is mixed in sign; red means the Qimen logit is worse at that cell. The color scale is clipped at ±0.05 for display. The cell table is unclipped.

The secondary 10-day block hit rate (36 blocks, 5 days dropped, not a calendar 旬) is 0.780 for the season-only logit, 0.770 for Qimen-plus-season, 0.777 for month rates, and 0.817 for persistence.

## 2026 supplement, not a replacement test

After the 2025 scores were stored, the same coefficients settings were refit on 2023 only and applied to 2026-01-01 through 2026-11-05 (309 days). That end date is the last day named by the earlier CPC 2026 metadata read; this run did not ask for a later day. All 267 cells had complete values. The 2026 wet-day rate is 0.4263. Brier is 0.1955 for the season-only logit, 0.1993 for Qimen-plus-season, 0.2005 for month climatology, 0.2220 for Qimen-only, and 0.1821 for lead-1 persistence. Corrected F1 on the same 267 cells is 0.4600, 0.4440, 0.4715, 0.2839, and 0.6696 respectively; histogram boosting’s F1 is 0.4881. The Brier order matches 2025. The F1 order does not put the Qimen logit first. These days were not used to choose a model, a threshold, or the window, and a partial year does not replace the 2025 test.

## What the results do not support

- A gain from the Qimen columns once a per-cell seasonal curve is available.
- Skill from the Qimen columns with the seasonal sine and cosine removed.
- A better probability than yesterday’s wet/dry state.
- A national, all-county, or all-grid result. The sample is 267 cells.
- A never-touched external test. The year 2025 was already inside the old artifact’s reported test window.
- A Beijing-time observation day, or a first-hand station series.
- A CMA product. NASA POWER was not used as a label or as an independent check; its corrected precipitation is documented to use CPC.
- A long-period climate conclusion. There is one training year.
- Any statement about marriage, wealth, health, examinations, or other human outcomes.

## 中文对照（可直接改写入论文的方法与结果）

本次实验重新下载 NOAA CPC 全球统一雨量计日降水分析（V1.0，单位 mm，日累计），不是中国气象局站点观测，也不是格点中心的雨量筒读数。窗口为 2023-01-01 至 2025-12-31。2023 年训练，2024 年只用来选择惩罚系数和树参数，2025 年只评分一次。下载范围 18–54°N、73–135°E 含有邻国和海洋，不能称为中国样本。湿日定义为日降水量 ≥ 0.1 mm。日界按 CPC 文件的公历日期标签连接，文件属性没有证明这是北京时间日界。提前或推后一天的起局只在 2024 年做了敏感性检查，对数损失只从 0.5865 变到 0.5859 和 0.5851，主分析没有改用错位日期。2025 年与旧区县工件已经报告过的测试期重叠，不能写成从未使用的外部测试。

2025 年概率分数没有因本次 F1 修正而改变：仅季节 L2 的 Brier 0.1923、对数损失 0.5656；奇门加季节 Brier 0.1970、对数损失 0.5769；月份气候态 Brier 0.1981；只含奇门 Brier 0.2216；随机森林 0.2028；直方图提升 0.2105；前日持续性 0.1793。修正后的 F1 见上表，训练期雨日率是 0.1723。四个纬度带和四个气象季节里，奇门加季节的 Brier 都高于仅季节逻辑回归。

2026-01-01 至 2026-11-05 共 309 天用同一套已选参数重评分，没有用来选模型。Brier 仍是仅季节 0.1955、奇门加季节 0.1993、前日持续性 0.1821。修正后 F1 分别是 0.4600、0.4440、0.6696。这段不能代替 2025 全年测试。

样本按顺序是 8928 个外包矩形格点、2786 个距行政中心不超过 60 km 的格点、272 个 2°代表格、再去掉 5 个全年 1096 天都缺测的代表格，得到 267。这 5 个格没有观测日，不是另外 5 个单类别格。267 个完整格在 2023 年都同时有干日和湿日。60 km 是行政中心邻域，不是严格国境掩码，也不要写成全国全量。

F1 按每个格的 2TP/(2TP+FP+FN) 计算，267 个格全部有定义。训练期雨日率的 F1 是 0.1723，不是原先把 202 个全年报干的格删掉之后的 0.7077。那个旧 F1 和旧精确率已作废，保存在结果文件的 `deprecated_test_metrics_nanmean_f1`。准确率、平衡准确率、召回率、CSI、Brier、对数损失和 ROC-AUC 与第一次评分一致。精确率在没有预报湿日时记 0，并单独给出这种格的个数：训练期雨日率 202 格，仅季节逻辑回归 73 格，奇门加季节 44 格，直方图提升 8 格，前日持续性 0 格。直方图提升的 F1 为 0.4676，高于仅季节逻辑回归的 0.4415，但它的 Brier 更差（0.2105 对 0.1923）。阈值 0.5 的 F1 和 Brier 不是同一个问题。

区间改为长度 10 的环形移动块，每次重抽样都覆盖全部 365 天，重复 400 次。点估计也是这 365 天。块长没有为了显著性而改。奇门加季节相对仅季节模型的 Brier 差值仍是 +0.0047，区间 +0.0033 到 +0.0061；F1 差值 −0.0172，区间 −0.0256 到 −0.0074。

特征确认是声明的 31 列，不是 64 列提取器里错位的前 31 列；第 32 列 `年支合地盘` 没有进入矩阵。同一 267 格、同一套已选参数。当日正午盘对应当日 CPC 日累计，只是日期分类，不是提前发布的预报。这里的“事先规定”只是仓库内冻结的协议，没有外部预注册记录。生产区县权重的 SHA-256 仍是 `2777979421117c88adbace02c2419b922f257378753870d2368a1bd6a5101d11`，本次没有改它。

因此论文可以写：在这 267 个行政中心邻域格点上，正午奇门符号没有在季节曲线之外降低 2025 年的 Brier；按统一 F1，它也低于仅季节逻辑回归。不能引用 0.7077，不能把 272−5−5 写成 267，不能写成全国或严格境内掩码，也不能把旧工件的 67.95% 或十日 79% 写成这次结果。人事效验没有数据，不能外推。
