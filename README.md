# Data Cleaning: NASA GISTEMP Global Temperature Anomalies

Cleaning of a public dataset using Python (pandas): identify missing values, duplicates, incorrect data types and inconsistent values, then produce an analysis-ready CSV.

## Dataset

**NASA GISS Surface Temperature Analysis (GISTEMP v4), Global Land-Ocean mean**, monthly, seasonal and annual temperature anomalies in °C relative to the 1951-1980 average, 1880 to 2026.

- Official source: https://data.giss.nasa.gov/gistemp/tabledata_v4/GLB.Ts+dSST.csv
- The raw file in this repo (`data/raw/GLB.Ts+dSST.csv`) is a copy of that table. It was taken from a copy distributed in the scipy test data, because the download host was not reachable when this was prepared. The script takes the file path as an argument, so you can re-run it on a fresh NASA download.

## Repository layout

```
data/raw/GLB.Ts+dSST.csv              original file, untouched
data/clean/gistemp_global_clean.csv   cleaned dataset
data/clean/cleaning_report.txt        full before/after log from the script
scripts/clean_gistemp.py              the cleaning script
```

Run it with `python3 scripts/clean_gistemp.py` (needs only pandas).

## What was found and what was done

| Check | Finding in raw file | Action |
|---|---|---|
| Structure | Line 1 is a title (`Land-Ocean: Global Means`), not data; the real header is on line 2 | Skipped the title line |
| Missing values | 17 cells use the placeholder `***` (not a real blank): Mar-Dec 2026, the full-year and season columns for 2026, and `D-N` / `DJF` for 1880 | Converted to true blanks (NaN). **Not imputed**, since these are not random gaps (see notes) |
| Incorrect data types | Because of `***`, 16 numeric columns load as text; values like `.16` and `-.19` are written without a leading zero (2,543 cells) | Converted all temperature columns to float and `year` to integer; written as `0.16`, `-0.19` |
| Duplicates | Checked for fully duplicated rows and repeated years | None found (0 and 0). Drop logic is still in the script |
| Inconsistent values | Cross-checked the annual column against the mean of its 12 months, and each season against its 3 months (tolerance 0.01 for rounding); checked for year gaps and implausible values | No inconsistencies found: 146 complete years agree, no gaps in 1880-2026, range is -0.82 to 1.48 °C |
| Column names | Mixed and cryptic (`J-D`, `D-N`, `DJF`...) | Renamed to descriptive snake_case (`annual_jan_dec`, `annual_dec_nov`, `winter_djf`, ...) |

Two columns were added so users do not have to rediscover the gaps: `months_present` (0-12) and `is_complete_year` (True/False).

**Result:** 147 rows x 21 columns (19 original columns plus the 2 added).

## Notes on the remaining blanks

- **2026** is a partial year (data through February), so Mar-Dec and the annual and later seasonal values are legitimately missing. Filter on `is_complete_year` for annual analysis.
- **1880** has no `annual_dec_nov` or `winter_djf` because those need December 1879, which is before the record starts.
- Filling these with averages would invent temperature readings, so they are left blank.
