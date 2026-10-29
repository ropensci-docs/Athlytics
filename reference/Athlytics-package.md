# Athlytics: A Reproducible Framework for Endurance Data Analysis

Tools for reproducible, offline analysis of endurance-training data
exported from 'Strava'. Provides data import, quality-control,
cohort-reference, and visualization helpers for sports-science
indicators including acute:chronic workload ratio, aerobic efficiency,
cardiovascular decoupling, exposure, and personal-best profiles.

Athlytics provides tools for reproducible, offline analysis of endurance
training data exported from Strava. It includes data import,
quality-control, cohort-reference, and visualization helpers for
sports-science indicators.

## Main Functions

**Data Loading:**

- [`load_local_activities()`](https://docs.ropensci.org/Athlytics/reference/load_local_activities.md):
  Load activities from Strava export ZIP or directory

- [`parse_activity_file()`](https://docs.ropensci.org/Athlytics/reference/parse_activity_file.md):
  Parse individual FIT/TCX/GPX files

**Training Load Analysis:**

- [`calculate_acwr()`](https://docs.ropensci.org/Athlytics/reference/calculate_acwr.md):
  Calculate Acute:Chronic Workload Ratio

- [`calculate_acwr_ewma()`](https://docs.ropensci.org/Athlytics/reference/calculate_acwr_ewma.md):
  ACWR using exponentially weighted moving averages

- [`calculate_exposure()`](https://docs.ropensci.org/Athlytics/reference/calculate_exposure.md):
  Calculate training load exposure metrics

**Physiological Metrics:**

- [`calculate_ef()`](https://docs.ropensci.org/Athlytics/reference/calculate_ef.md):
  Calculate Efficiency Factor (EF)

- [`calculate_decoupling()`](https://docs.ropensci.org/Athlytics/reference/calculate_decoupling.md):
  Calculate cardiovascular decoupling

- [`calculate_pbs()`](https://docs.ropensci.org/Athlytics/reference/calculate_pbs.md):
  Calculate personal bests

**Visualization:**

- [`plot_acwr()`](https://docs.ropensci.org/Athlytics/reference/plot_acwr.md),
  [`plot_acwr_enhanced()`](https://docs.ropensci.org/Athlytics/reference/plot_acwr_enhanced.md):
  Plot ACWR trends

- [`plot_ef()`](https://docs.ropensci.org/Athlytics/reference/plot_ef.md):
  Plot Efficiency Factor trends

- [`plot_decoupling()`](https://docs.ropensci.org/Athlytics/reference/plot_decoupling.md):
  Plot decoupling analysis

- [`plot_exposure()`](https://docs.ropensci.org/Athlytics/reference/plot_exposure.md):
  Plot training load exposure

- [`plot_pbs()`](https://docs.ropensci.org/Athlytics/reference/plot_pbs.md):
  Plot personal bests progression

**Quality Control & Cohort Analysis:**

- [`flag_quality()`](https://docs.ropensci.org/Athlytics/reference/flag_quality.md):
  Flag activities based on quality criteria

- [`summarize_quality()`](https://docs.ropensci.org/Athlytics/reference/summarize_quality.md):
  Summarize stream quality flags

- [`calculate_cohort_reference()`](https://docs.ropensci.org/Athlytics/reference/calculate_cohort_reference.md):
  Generate cohort reference bands

## Sample Datasets

The package includes simulated datasets for examples and testing:

- [sample_acwr](https://docs.ropensci.org/Athlytics/reference/sample_acwr.md):
  Sample ACWR data

- [sample_ef](https://docs.ropensci.org/Athlytics/reference/sample_ef.md):
  Sample Efficiency Factor data

- [sample_decoupling](https://docs.ropensci.org/Athlytics/reference/sample_decoupling.md):
  Sample decoupling data

- [sample_exposure](https://docs.ropensci.org/Athlytics/reference/sample_exposure.md):
  Sample exposure data

- [sample_pbs](https://docs.ropensci.org/Athlytics/reference/sample_pbs.md):
  Sample personal bests data

## Getting Started

    library(Athlytics)

    # Load your Strava export
    activities <- load_local_activities("path/to/strava_export.zip")

    # Calculate ACWR
    acwr_data <- calculate_acwr(activities, activity_type = "Run")

    # Visualize
    plot_acwr(acwr_data)

## See also

Useful links:

- <https://docs.ropensci.org/Athlytics/>

- <https://github.com/ropensci/Athlytics>

- Report bugs at <https://github.com/ropensci/Athlytics/issues>

&nbsp;

- Package website: <https://docs.ropensci.org/Athlytics/>

- GitHub repository: <https://github.com/ropensci/Athlytics>

- Strava: <https://www.strava.com/>

## Author

**Maintainer**: Zhiang He <ang@hezhiang.com>

Authors:

- Zhiang He <ang@hezhiang.com>

Other contributors:

- Eunseop Kim (Eunseop Kim reviewed the package (v. 1.0.4) for rOpenSci,
  see https://github.com/ropensci/software-review/issues/728)
  \[reviewer\]

- Simon Nolte (Simon Nolte reviewed the package (v. 1.0.4) for rOpenSci,
  see https://github.com/ropensci/software-review/issues/728)
  \[reviewer\]
