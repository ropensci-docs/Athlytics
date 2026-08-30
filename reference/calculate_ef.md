# Calculate Efficiency Factor (EF)

Efficiency Factor measures how much work you perform per unit of
cardiovascular effort. Higher EF indicates better aerobic fitness -
you're able to maintain faster pace or higher power at the same heart
rate. Tracking EF over time helps monitor aerobic base development and
training effectiveness.

**EF Metrics:**

- **speed_hr** (for running): Speed (m/s) / Average HR

  - Higher values = faster speed at same HR = better fitness

- **power_hr** (for cycling): Average Power (watts) / Average HR

  - Higher values = more power at same HR = better fitness

**What Improves EF?**

- Aerobic base building (Zone 2 training)

- Improved running/cycling economy

- Enhanced cardiovascular efficiency

- Increased mitochondrial density

## Usage

``` r
calculate_ef(
  activities_data,
  activity_type = c("Run", "Ride"),
  ef_metric = c("speed_hr", "gap_hr", "power_hr"),
  start_date = NULL,
  end_date = Sys.Date(),
  min_duration_mins = 20,
  min_steady_minutes = 20,
  steady_cv_threshold = 0.08,
  min_hr_coverage = 0.9,
  quality_control = c("filter", "flag", "off"),
  export_dir = NULL
)
```

## Arguments

- activities_data:

  A data frame of activities from
  [`load_local_activities()`](https://docs.ropensci.org/Athlytics/reference/load_local_activities.md).
  Must contain columns: `date`, `type`, `moving_time`, `distance`,
  `average_heartrate`, and `average_watts` (for power_hr metric).

- activity_type:

  Character vector or single string specifying activity type(s) to
  analyze. Common values: `"Run"`, `"Ride"`, or `c("Run", "Ride")`.
  Default: `c("Run", "Ride")`.

- ef_metric:

  Character string specifying the efficiency metric:

  - `"speed_hr"`: Speed-based efficiency (for running). Formula: speed
    (m/s) / avg_HR. Units: m/s/bpm (higher = better fitness)

  - `"gap_hr"`: Grade Adjusted Speed efficiency (for running on hilly
    terrain). Formula: GAP speed (m/s) / avg_HR. Accounts for elevation
    changes. Units: m/s/bpm

  - `"power_hr"`: Power-based efficiency (for cycling). Formula:
    avg_watts / avg_HR. Units: W/bpm (higher = better fitness) Default:
    `c("speed_hr", "power_hr")` (uses first matching metric for activity
    type). Note: `"pace_hr"` is accepted as a deprecated alias for
    `"speed_hr"` for backward compatibility.

- start_date:

  Optional. Analysis start date (YYYY-MM-DD string, Date, or POSIXct).
  Defaults to one year before `end_date`.

- end_date:

  Optional. Analysis end date (YYYY-MM-DD string, Date, or POSIXct).
  Defaults to current date (Sys.Date()).

- min_duration_mins:

  Numeric. Minimum activity duration in minutes to include in analysis
  (default: 20). Filters out very short activities that may not
  represent steady-state aerobic efforts.

- min_steady_minutes:

  Numeric. Minimum duration (minutes) for steady-state segment (default:
  20). Activities shorter than this are automatically rejected for EF
  calculation.

- steady_cv_threshold:

  Numeric. Coefficient of variation threshold for steady-state (default:
  0.08 = 8%). Activities with higher variability are rejected as
  non-steady-state.

- min_hr_coverage:

  Numeric. Minimum HR data coverage threshold (default: 0.9 = 90%).
  Activities with lower HR coverage are rejected as insufficient data
  quality.

- quality_control:

  Character. Quality control mode: "off" (no filtering), "flag" (mark
  issues), or "filter" (exclude flagged data). Default "filter" for
  scientific rigor.

- export_dir:

  Optional. Path to a Strava export directory or ZIP file containing
  activity files. When provided, enables stream data analysis for more
  accurate steady-state detection; when omitted, EF falls back to
  activity-summary averages.

## Value

A tibble with the following columns:

- date:

  Activity date (Date class)

- activity_type:

  Activity type (character: "Run" or "Ride")

- ef_value:

  Efficiency Factor value (numeric). Higher = better fitness. Units:
  m/s/bpm for speed_hr, W/bpm for power_hr.

- status:

  Character status code describing the outcome of the calculation. See
  **Status vocabulary** below.

- ef_metric_requested:

  The metric the user asked for (`"speed_hr"`, `"gap_hr"`, or
  `"power_hr"`). Mirrors the `ef_metric` argument.

- ef_metric_used:

  The metric that was actually computed. Usually matches
  `ef_metric_requested`; for `gap_hr` inputs processed through the
  stream path (where no GAP channel is available) this is `"speed_hr"`
  and the row is also marked with status
  `"gap_stream_unavailable_fallback_to_speed"`.

- quality_score:

  Numeric in \[0, 1\]. Fraction of stream samples that passed
  quality-control range checks (HR, velocity, watts). `NA` when no
  stream was parsed or when the activity was rejected before QC.

- hr_coverage:

  Numeric in \[0, 1\]. Time-weighted fraction of the stream that carried
  a valid heart-rate sample. `NA` for activity-level paths
  (`status = "no_streams"`).

- steady_duration_minutes:

  Wall-clock duration of the contiguous steady-state block the EF value
  was derived from. `NA` for activity-level paths or when no qualifying
  block existed.

- n_steady_blocks:

  Number of contiguous steady-state blocks that met the
  `min_steady_minutes` threshold. `NA` for activity-level paths.

- sampling_interval_seconds:

  Observed median sampling interval of the stream (seconds). `NA` for
  activity-level paths.

## Details

Computes Efficiency Factor (EF) for endurance activities, quantifying
the relationship between performance output (speed or power) and heart
rate. EF is a key indicator of aerobic fitness and training adaptation
(Allen et al., 2019).

**Algorithm:**

1.  Filter activities by type, date range, and minimum duration

2.  For each activity, calculate:

    - speed_hr: (distance / moving_time) / average_heartrate

    - power_hr: average_watts / average_heartrate

3.  Return one EF value per activity

**Steady-State Detection Method:**

When stream data is available (via `export_dir`), the function applies a
rolling coefficient of variation (CV) approach to identify steady-state
periods and then enforces *contiguous* block duration so scattered
"steady" islands are not averaged together:

1.  **Sampling-interval awareness**: The observed median sampling
    interval is estimated via `diff(time)` so the rolling window targets
    a fixed number of seconds regardless of recording frequency (1 Hz,
    0.5 Hz smart-recording, multi-Hz). This removes the implicit 1 Hz
    assumption from earlier versions.

2.  **Rolling window**: A sliding window targeting 300 seconds (capped
    by `nrow / 4`, floor at 60 seconds' worth of samples) computes
    rolling mean and SD of the output metric.

3.  **CV calculation**: CV = rolling SD / rolling mean at each time
    point.

4.  **Continuous blocks**: All time points with CV \<
    `steady_cv_threshold` are marked steady, then grouped into
    contiguous runs via [`rle()`](https://rdrr.io/r/base/rle.html). A
    block qualifies only if its wall-clock span is \>=
    `min_steady_minutes` AND it has \>= 100 samples. If no block
    qualifies, status is `"insufficient_steady_duration"` (activities
    with no steady CV window at all are marked `"non_steady"`).

5.  **EF computation**: The longest qualifying block is selected; EF is
    the median output/HR ratio across that block.
    `steady_duration_minutes`, `n_steady_blocks` and
    `sampling_interval_seconds` are returned alongside the EF value for
    auditability.

This mirrors the block-based steady-state logic used in
[`calculate_decoupling()`](https://docs.ropensci.org/Athlytics/reference/calculate_decoupling.md)
and follows the principle that valid EF comparisons require
quasi-constant output intensity, as outlined by Coyle & González-Alonso
(2001), who demonstrated that cardiovascular drift is meaningful only
under steady-state exercise conditions. The rolling CV method is a
common signal-processing technique for detecting stationarity in
physiological time series.

**Data Quality Considerations:**

- Requires heart rate data (activities without HR are excluded)

- power_hr requires power meter data (cycling with power)

- Best for steady-state endurance efforts (tempo runs, long rides)

- Interval workouts may give misleading EF values

- Environmental factors (heat, altitude) can affect EF

**Interpretation:**

- **Upward trend**: Improving aerobic fitness

- **Stable**: Maintenance phase

- **Downward trend**: Possible overtraining, fatigue, or environmental
  stress

- **Sudden drop**: Check for illness, equipment change, or data quality

**Typical EF Ranges (speed_hr for running):**

- Beginner: 0.01 - 0.015 (m/s per bpm)

- Intermediate: 0.015 - 0.020

- Advanced: 0.020 - 0.025

- Elite: \> 0.025

Note: EF values are relative to individual baseline. Focus on personal
trends rather than absolute comparisons with other athletes.

## Status vocabulary

- `"ok"`: Full steady-state analysis succeeded on stream data.

- `"no_streams"`: No stream file was available; `ef_value` was computed
  from activity-level averages. QC and HR-coverage metrics are `NA`.

- `"gap_stream_unavailable_fallback_to_speed"`: Caller requested
  `ef_metric = "gap_hr"` but the stream does not expose a grade-adjusted
  channel, so `ef_value` was computed from plain speed/HR.
  `ef_metric_used` is `"speed_hr"` on these rows so downstream code can
  filter them out.

- `"missing_velocity_data"`: Stream lacked both `distance` and
  `velocity_smooth` when `ef_metric = "speed_hr"`.

- `"missing_power_data"`: Stream lacked `watts` when
  `ef_metric = "power_hr"`.

- `"missing_hr_data"`: Stream lacked a `heartrate` / `heart_rate`
  column.

- `"insufficient_hr_data"`: Time-weighted HR coverage \<
  `min_hr_coverage`.

- `"insufficient_data_points"`: Fewer than 100 non-NA stream samples.

- `"insufficient_valid_data"`: Fewer than 100 samples survived the basic
  positivity filter (`velocity > 0` or `watts > 0`).

- `"insufficient_data_after_quality_filter"`:
  `quality_control = "filter"` removed enough out-of-range samples to
  drop the stream below 100 rows.

- `"poor_hr_quality"`: Activity-level path with out-of-range
  `average_heartrate` while `quality_control = "filter"`.

- `"poor_hr_quality_flagged"`: Activity-level path with out-of-range
  `average_heartrate` kept for inspection because
  `quality_control = "flag"`.

- `"too_short"`: Activity or steady-state window duration is below
  `min_steady_minutes`.

- `"non_steady"`: No rolling-CV window cleared `steady_cv_threshold`.

- `"insufficient_steady_duration"`: No *contiguous* steady-state block
  lasted at least `min_steady_minutes`.

- `"calculation_failed"`: Median ratio was non-positive or not finite.

## References

Allen, H., Coggan, A. R., & McGregor, S. (2019). *Training and Racing
with a Power Meter* (3rd ed.). VeloPress.

Coyle, E. F., & González-Alonso, J. (2001). Cardiovascular drift during
prolonged exercise: New perspectives. *Exercise and Sport Sciences
Reviews*, 29(2), 88-92.
[doi:10.1097/00003677-200104000-00009](https://doi.org/10.1097/00003677-200104000-00009)

## See also

[`plot_ef()`](https://docs.ropensci.org/Athlytics/reference/plot_ef.md)
for visualization with trend lines,
[`calculate_decoupling()`](https://docs.ropensci.org/Athlytics/reference/calculate_decoupling.md)
for within-activity efficiency analysis,
[`load_local_activities()`](https://docs.ropensci.org/Athlytics/reference/load_local_activities.md)
for data loading

## Examples

``` r
# Example using simulated data
data(sample_ef)
print(head(sample_ef))
#> # A tibble: 6 × 3
#>   date       activity_type ef_value
#>   <date>     <chr>            <dbl>
#> 1 2023-01-01 Run               1.16
#> 2 2023-01-01 Ride              1.86
#> 3 2023-01-04 Run               1.19
#> 4 2023-01-06 Run               1.21
#> 5 2023-01-06 Ride              1.94
#> 6 2023-01-08 Run               1.31

# Runnable example with dummy data:
end <- Sys.Date()
dates <- seq(end - 29, end, by = "day")
dummy_activities <- data.frame(
  date = dates,
  type = "Run",
  moving_time = rep(3600, length(dates)), # 1 hour
  distance = rep(10000, length(dates)), # 10 km
  average_heartrate = rep(140, length(dates)),
  average_watts = rep(200, length(dates)),
  weighted_average_watts = rep(210, length(dates)),
  filename = "",
  stringsAsFactors = FALSE
)

# Calculate EF (Speed/HR)
ef_result <- calculate_ef(
  activities_data = dummy_activities,
  activity_type = "Run",
  ef_metric = "speed_hr",
  end_date = end
)
print(head(ef_result))
#>         date activity_type   ef_value     status ef_metric_requested
#> 1 2026-08-01           Run 0.01984127 no_streams            speed_hr
#> 2 2026-08-02           Run 0.01984127 no_streams            speed_hr
#> 3 2026-08-03           Run 0.01984127 no_streams            speed_hr
#> 4 2026-08-04           Run 0.01984127 no_streams            speed_hr
#> 5 2026-08-05           Run 0.01984127 no_streams            speed_hr
#> 6 2026-08-06           Run 0.01984127 no_streams            speed_hr
#>   ef_metric_used quality_score hr_coverage steady_duration_minutes
#> 1       speed_hr            NA          NA                      NA
#> 2       speed_hr            NA          NA                      NA
#> 3       speed_hr            NA          NA                      NA
#> 4       speed_hr            NA          NA                      NA
#> 5       speed_hr            NA          NA                      NA
#> 6       speed_hr            NA          NA                      NA
#>   n_steady_blocks sampling_interval_seconds
#> 1              NA                        NA
#> 2              NA                        NA
#> 3              NA                        NA
#> 4              NA                        NA
#> 5              NA                        NA
#> 6              NA                        NA

if (FALSE) { # \dontrun{
# Example using local Strava export data
activities <- load_local_activities("strava_export_data/activities.csv")

# Calculate Speed/HR efficiency factor for Runs
ef_data_run <- calculate_ef(
  activities_data = activities,
  activity_type = "Run",
  ef_metric = "speed_hr"
)
print(tail(ef_data_run))

# Calculate Power/HR efficiency factor for Rides
ef_data_ride <- calculate_ef(
  activities_data = activities,
  activity_type = "Ride",
  ef_metric = "power_hr"
)
print(tail(ef_data_ride))
} # }
```
