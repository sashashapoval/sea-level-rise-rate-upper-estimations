# From Local Tide-Gauge Trends to Sea-Level Rise Estimates

## Overview

This repository contains data and computational materials accompanying *From Local Tide-Gauge Trends to Sea-Level Rise Estimates: Means, Medians, and Trimming*. The observational analysis fits local linear trends to annual tide-gauge records and calculates arithmetic means, medians, and trimmed median estimates in sliding 60- and 70-year windows within 1900–2019.

The `bin` folder contains analysis notebooks and supporting code. The `data` folder contains the annual Revised Local Reference (RLR) records from the Permanent Service for Mean Sea Level (PSMSL). The observational entry point is `bin/observational_sea_level_estimates.ipynb`.

## Folder structure and archived data

The data are distributed as a **7-Zip archive (`.7z`)** and must be extracted before running the notebooks. The archive preserves the original data-folder structure, including the catalogue, station records, and accompanying documentation.

The standard arrangement places `data.7z` in the master folder alongside `README.md` and `bin`. The archive contains the top-level folder `data`. Extract it **into the master folder**, preserving its internal paths. With 7-Zip, the **Extract Here** action in the master folder produces the intended layout.

After extraction, the following paths are relative to the master folder:

| Path | Contents |
|---|---|
| `README.md` | General description and execution instructions |
| `bin/` | Notebooks and supporting code |
| `data.7z` | Compressed copy of the data folder |
| `data/rlr_annual/filelist.txt` | Station catalogue |
| `data/rlr_annual/data/<number>.rlrdata` | Annual record for each station |


Other files supplied with the PSMSL dataset remain inside `data/rlr_annual`.


## Python environment

The observational notebook requires Python 3.10 or later, NumPy, SciPy, pandas, Matplotlib, IPython, and a Jupyter environment. A basic installation command is:

```bash
python -m pip install numpy scipy pandas matplotlib ipython jupyterlab
```

From the master folder, start JupyterLab in `bin`:

```bash
cd bin
python -m jupyterlab
```

Open `observational_sea_level_estimates.ipynb`. With `bin` as the notebook's working directory, the data setting is:

```python
DATA_DIR = Path("../data/rlr_annual")
```

This path identifies the folder containing `filelist.txt`; the reader accesses its `data` subfolder automatically. The following check confirms that the required files are accessible:

```python
print("Catalogue found:", (DATA_DIR / "filelist.txt").is_file())
print("Station files:", len(list((DATA_DIR / "data").glob("*.rlrdata"))))
```

The expected outputs are `True` and a positive station-file count.

## Running the observational analysis

1. Extract the data archive and check the data path in Section 1.
2. Run the notebook cells in order through Section 6, which calculates the final-window pilot.
3. Inspect the eligible and selected station counts and the resulting estimates.
4. Set `RUN_FULL = True` in Section 7 to calculate all sliding windows.
5. Continue through the result tables, figures, and export cell in Section 12.

The full calculation in Section 7 is the main computational step. All required functions are included in the notebook; an existing kernel from another notebook is unnecessary. Other analysis notebooks contain their own settings and execution instructions.

## Main calculation settings

| Setting | Default |
|---|---|
| Observations | Annual RLR sea levels, in mm |
| Analysis period | 1900–2019 |
| Window lengths | 60 and 70 years |
| Minimum completeness | 75% and 87% |
| Station selection | Positive OLS slope and two-sided p-value at most 0.01 |
| Equivalent one-sided threshold for positive slopes | 0.005 |
| Upper-tail trimming fractions | 0, 0.10, and 0.25 |
| Subsampling | Approximately half the retained stations, without replacement |
| Repetitions per window and trimming fraction | 1,000 |

A station's recorded time span must contain the full window. Missing years and missing-value entries count toward incompleteness; observations are not interpolated. The baseline reader parses data flags without applying additional flag-based exclusions. Retained station counts and subsample sizes use nearest-integer rounding, with half-integer values rounded upward.

The mean comparison uses the average of subsample means. The median comparison uses the average of subsample medians. The final trimmed estimate, `M_prime`, is the **median of subsample medians after upper-tail trimming**. Direct means and medians are also exported.

Subsampling standard deviations and the documented nominal 99% envelope describe variability conditional on the selected station sets. The envelope construction is specified in the notebook. Overlapping windows share observations, and the resampling procedure draws station subsets separately within each window.

## Outputs and reproducibility

Section 12 creates a new timestamped results folder under `annual_tide_gauge_results` in the notebook's working directory. The exports include:

- fitted station slopes, p-values, and completeness information;
- window estimates, station counts, subsampling standard deviations, and envelopes;
- station identities and trimming decisions;
- subsample means and medians;
- figures in JPG and PDF formats;
- calculation settings, package versions, input-file hashes, and output-field descriptions.

The data archive preserves the input snapshot used for the calculations. Later PSMSL downloads can revise historical records, so matching the manuscript's station populations requires the corresponding archived snapshot. The 2019 endpoint retains the original study period throughout the revisions.

Random seeds are specified in the notebook. The Python implementation uses independent subsets across repetitions; the earlier MATLAB helper attempted to reject repeated subsets. Fresh Monte Carlo summaries can therefore differ slightly from archived MATLAB values, while direct summaries remain deterministic for a fixed input dataset and selection rule.

The accompanying PSMSL reference and documentation files are retained with the dataset. Dataset acknowledgement and citation information are available in `data/rlr_annual/reference.txt` and the accompanying documentation.
