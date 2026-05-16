# Flag Definitions

Flags are factual observations, not recommendations.

## Activity Pattern Flags

- `two_hard_days_in_48_hours`: at least two hard sessions occur within any 48-hour window.
- `three_or_more_hard_sessions_this_week`: weekly hard session count >= 3.
- `running_volume_up_more_than_10_percent`: current week run volume > 110% of prior baseline.
- `bike_volume_up_more_than_20_percent`: current week bike volume > 120% of prior baseline.
- `no_full_rest_day_last_9_days`: zero no-activity days in trailing 9 days.
- `longest_run_in_6_weeks`: longest run distance this period exceeds prior 6-week max.
- `longest_ride_in_6_weeks`: longest ride duration/distance this period exceeds prior 6-week max.
- `sudden_duration_spike`: abrupt total-duration jump vs recent baseline.
- `first_activity_after_break`: first session after multi-day inactivity break.

## Notes-Derived Flags

- `pain_or_injury_mentioned`: notes contain pain/injury indicators.
- `illness_mentioned`: notes contain sickness indicators.

## Data Quality Flags

- `low_data_confidence`: critical fields missing or sparse activity context.
- `missing_days`: one or more days absent from expected date sequence.

## Computation Guidance

- Prefer direct calculations from available records.
- If insufficient data exists for a flag, do not force it; note uncertainty instead.
- Always keep period boundaries explicit.


## Compact Block Guidance

When evaluating flags from OpenClaw daily memory, use only generated workout blocks delimited by:

```text
<!-- OPENCLAW_FIT_WORKOUT_START
<!-- OPENCLAW_FIT_WORKOUT_END
```

Use `Compact Workout Facts` YAML as the primary source. In particular:

- Use `hard_session` for hard-session counts when present.
- Use `training_load_points` for sudden load/duration spike context when present.
- Use `date`, `duration_minutes`, `distance_km`, and `sport` for volume and consistency flags.
- Use `hr_zones` for intensity context.
- Do not open `workout_detail_ref` files during routine flag detection.
