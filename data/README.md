# NOAA CPC retrieval

The labels in this experiment are daily precipitation totals from the **NOAA CPC Global Unified Gauge-Based Analysis of Daily Precipitation**, file attributes `title=CPC GLOBAL PRCP V1.0 RT`, `dataset_title=CPC GLOBAL PRCP V1.0`, `version=V1.0`. Units are millimetres. `cell_methods` is `time: sum`. The missing marker stored in the files is about −9.96921×10^36.

Distribution used here: NOAA Physical Sciences Laboratory THREDDS NetCDF Subset Service (NCSS), yearly dataset `precip.YYYY.nc`.

- Product page: <https://www.psl.noaa.gov/data/gridded/data.cpc.globalprecip.html>
- THREDDS catalog: <https://psl.noaa.gov/thredds/catalog/Datasets/cpc_global_precip/catalog.html>
- Source path recorded inside the files: `ftp://ftp.cpc.ncep.noaa.gov/precip/CPC_UNI_PRCP/`

These grids are a gauge-based **analysis**. They are not China Meteorological Administration station series, and they are not rain-gauge readings taken at the CPC cell centers. Real-time CPC files are revised. This experiment is pinned to the containers downloaded on **2026-11-07** (about 08:04–08:07 UTC for the 36 analysis months). A later NCSS response can differ.

## What was requested

Serial monthly NCSS subsets, not the global multi-gigabyte yearly files:

- Variable: `precip`
- Bounding box: 18–54°N, 73–135°E (`north=54`, `south=18`, `east=135`, `west=73`)
- Horizontal stride 1, time stride 1, `accept=netcdf`
- Analysis window: 2023-01-01 through 2025-12-31 (1,096 days, 36 monthly files)
- Supplement, scored after the 2025 test and not used to choose a model: 2026-01-01 through 2026-11-05

The bounding box contains neighboring countries and sea. It is the download window only. The analysis sample is the 267-cell lattice described in the root README.

After one edge latitude and one edge longitude were dropped, the kept grid has 72 × 124 = 8,928 cell centers, from 53.75°N to 18.25°N and 73.25°E to 134.75°E (0.5° spacing). Recorded in [`../environment/download_manifest.json`](../environment/download_manifest.json).

The 36 analysis containers total **40,119,328 bytes** (about 38 MiB). They are **not committed**. Download them into a local directory that stays out of git (`*.nc` is gitignored).

## URL template

Replace `YYYY` and the month’s start and end dates. `time_end` in the manifest is the last day of the month at `00:00:00Z` (January 2023 ends on `2023-01-31T00:00:00Z`, 31 times).

```text
https://psl.noaa.gov/thredds/ncss/grid/Datasets/cpc_global_precip/precip.YYYY.nc?var=precip&north=54.0&south=18.0&east=135.0&west=73.0&horizStride=1&time_start=YYYY-MM-01T00:00:00Z&time_end=YYYY-MM-DDT00:00:00Z&timeStride=1&accept=netcdf
```

January 2023, as stored in the manifest:

```text
https://psl.noaa.gov/thredds/ncss/grid/Datasets/cpc_global_precip/precip.2023.nc?var=precip&north=54.0&south=18.0&east=135.0&west=73.0&horizStride=1&time_start=2023-01-01T00:00:00Z&time_end=2023-01-31T00:00:00Z&timeStride=1&accept=netcdf
```

Example retrieval of that one month (writes a local NetCDF; do not commit it):

```bash
curl -fsSL -o precip_202301.nc \
  "https://psl.noaa.gov/thredds/ncss/grid/Datasets/cpc_global_precip/precip.2023.nc?var=precip&north=54.0&south=18.0&east=135.0&west=73.0&horizStride=1&time_start=2023-01-01T00:00:00Z&time_end=2023-01-31T00:00:00Z&timeStride=1&accept=netcdf"
```

Every analysis-month URL, byte size, retrieval time, and container SHA-256 is in [`../environment/download_manifest.json`](../environment/download_manifest.json) under `files`. The 2026 supplement names and container hashes, without URLs repeated, are in [`supplement_2026_files.json`](supplement_2026_files.json).

## Hashes, and why a re-download may not match them

NCSS writes a translation timestamp into the NetCDF container. The container SHA-256 can change across retrievals even when the decoded precipitation values do not. The manifest hashes are the **2026-11-07 containers**. They are a pin for that retrieval, not a guarantee that a new response will hash the same.

A check that was done before the monthly download: a Beijing-area subset for 2025-07-01 through 2025-07-07 (`north=40`, `south=39`, `east=117`, `west=116`). Its container SHA-256 differed from an earlier retrieval. The seven values at 39.75°N, 116.25°E matched a prior decode, in millimetres:

`3.04322934`, `7.38294697`, `1.78654253`, `4.37161636`, `0.82105678`, `0.05515119`, `0.88242400`.

Details: [`sample_verify.json`](sample_verify.json).

Invalid or out-of-range values were set to missing. None were filled with 0. A representative cell with any non-finite value, any value outside [0, 1000] mm, or any CPC missing marker in 2023–2025 was removed. Five representatives were missing on all 1,096 days.

## Time coordinate

`hours since 1900-01-01 00:00:00`, calendar `gregorian`. The experiment joins the resulting calendar date to the noon Qimen chart (hour 12, minute 0) on that same date string. The file attributes do not establish a Beijing-time observing day (neither 00:00–24:00 nor 20:00–20:00). Gauge days inside CPC can follow the contributing networks.

Wet day in this experiment: daily total ≥ 0.1 mm.

## Arrays that are not in git

| Item | Where the hash is recorded | In this repository |
| --- | --- | --- |
| Selected precipitation float32 used in the fit | `results/results.json` → `hashes.selected_precip_float32_sha256` = `5969cb104a45058e31b61ba11d023d898312eae622a716b7db389d9de76ccd49` | No |
| Qimen feature JSON (31 columns) | `hashes.qimen_feature_json_sha256` = `93818ff4a305d26608ebeec476e9a906528418928986ded9a04fafdcf430734f` | No |
| 2025 predicted probabilities | `predictions/README.md` | Yes, the npz only |

## How to cite the product

Give NOAA/PSL as the data source, and cite the method papers NOAA lists for this gauge-analysis family. Bibliographic details checked against the journal landing pages on 2026-11-07:

- Xie, P., A. Yatagai, M. Chen, T. Hayasaka, Y. Fukushima, C. Liu, and S. Yang, 2007: A gauge-based analysis of daily precipitation over East Asia. *Journal of Hydrometeorology*, **8**(3), 607–626. <https://doi.org/10.1175/JHM583.1>
- Chen, M., W. Shi, P. Xie, V. B. S. Silva, V. E. Kousky, R. W. Higgins, and J. E. Janowiak, 2008: Assessing objective techniques for gauge-based analyses of global daily precipitation. *Journal of Geophysical Research: Atmospheres*, **113**, D04110. <https://doi.org/10.1029/2007JD009132>

NOAA’s own documentation also points at a 2008 conference abstract by Chen, Xie, and co-authors on construction of the daily gauge analysis. That abstract is a different document from the JGR paper above and is not given a DOI here.

The 2007 paper is the East Asia gauge-analysis method that CPC documentation cites for interpolation. The files actually read in this experiment are the real-time global V1.0 grids (`CPC GLOBAL PRCP V1.0 RT`) served by PSL. The first 2023 monthly file in the manifest records `history=Updated 2024-01-02 23:30:47`.

NOAA requests attribution of NOAA PSL as the source. This repository’s MIT license covers the notes and scored outputs that the author can license. It does not re-license the CPC data.
