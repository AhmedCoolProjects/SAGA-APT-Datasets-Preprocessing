# SAGA APT Datasets Preprocessing

This repository contains a Jupyter notebook–based pipeline for preparing the SAGA APT (Advanced Persistent Threat) telemetry into log-style CSV datasets suitable for downstream analytics or ML.

## Contents

- Notebook: [github/SAGA APT Datasets Preprocessing.ipynb](github/SAGA APT Datasets Preprocessing.ipynb)
<!-- - Raw JSON folders (expected): `known/`, `composite/`, `generated/` -->
- Processed CSV outputs (per chunk & merged): typically under `prod/`
- Compressed per‑chunk cleaned files: `known/known_c1.csv.gz` … `known/known_c8.csv.gz`
- Plot: `known/normal_anomalous.png`

## Pipeline Overview

1. Discover JSON files via [`get_json_files`](github/SAGA APT Datasets Preprocessing.ipynb).
2. Load & flatten each JSON into a DataFrame with [`load_json_to_df`](github/SAGA APT Datasets Preprocessing.ipynb):
   - Expands `srcNode` / `dstNode`
   - Adds `datetime` column from epoch `timestamp`
3. Convert each chunk into a concise log format with [`make_log_df`](github/SAGA APT Datasets Preprocessing.ipynb) producing:
   - `timestamp`, `label`, `content`
4. Merge all chunk DataFrames, sort by `timestamp`, write `known.csv`.
5. Derive binary label column `Label` (0 = benign, 1 = anomalous).
6. Normalize / anonymize high‑variance tokens in `content` via [`clean_log_content`](github/SAGA APT Datasets Preprocessing.ipynb):
   - Replaces hex blobs, memory addresses, PIDs, IPs, long numbers, paths, URLs, emails, timestamps, etc.
7. Export per‑chunk cleaned CSVs `known_c{n}.csv` and (optionally) gzip them.
8. Load processed slices with [`load_prod`](github/SAGA APT Datasets Preprocessing.ipynb) for analysis.
9. Generate distribution bar plot (Normal vs Anomalous per chunk) saved as `normal_anomalous.png`.

## Directory Structure (Example)

```
known/
  known_c1.csv.gz
  known_c2.csv.gz
  ...
  known_c8.csv.gz
  normal_anomalous.png
composite/
  M1.json ...
generated/
  G1.json ...
prod/
  C1_log.csv ...
  known.csv
```

## Usage

1. Set `BASE_PATH` in the notebook to the root containing `known/`, `composite/`, `generated/`.
2. Run cells in order:
   - Load & flatten raw JSON
   - Build `_known` log DataFrames
   - Merge & label
   - Clean `content`
   - Save per‑chunk and merged CSVs
3. (Optional) Compress outputs:
   ```bash
   gzip known_c*.csv
   ```
4. Re-run the Analysis section to regenerate the Normal/Anomalous distribution plot.

## Key Functions

| Purpose                      | Symbol                                                              |
| ---------------------------- | ------------------------------------------------------------------- |
| Enumerate dataset JSON files | [`get_json_files`](github/SAGA APT Datasets Preprocessing.ipynb)    |
| Load & flatten a JSON file   | [`load_json_to_df`](github/SAGA APT Datasets Preprocessing.ipynb)   |
| Assemble log-style rows      | [`make_log_df`](github/SAGA APT Datasets Preprocessing.ipynb)       |
| Clean high-variance tokens   | [`clean_log_content`](github/SAGA APT Datasets Preprocessing.ipynb) |
| Load processed CSV slices    | [`load_prod`](github/SAGA APT Datasets Preprocessing.ipynb)         |

## Plot Generation

The analysis section:

- Aggregates counts (Total / Normal / Anomalous) per dataset chunk
- Melts to long form
- Computes percentages
- Renders a Seaborn grouped bar chart with percentage labels

## Notes

- Large counts: Y‑axis labeled “Count (in Millions)”—adjust scaling if exporting subsets.
- Ensure memory adequacy; consider chunked reading if expanding beyond eight slices.
- Cleaning patterns are conservative; tailor regexes as needed for your threat model.

<!-- ## Citation

(Add citation or reference info if -->
