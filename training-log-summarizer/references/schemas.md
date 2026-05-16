
## Compact Daily Workout Input

Preferred daily input blocks are delimited by:

```text
<!-- OPENCLAW_FIT_WORKOUT_START source_file="..." -->
...
<!-- OPENCLAW_FIT_WORKOUT_END source_file="..." -->
```

Inside each block, parse the `Compact Workout Facts` YAML:

```yaml
memory_type: daily_workout
source: fit-parser
source_file: string
detail_file: string
date: YYYY-MM-DD
start_time: ISO-8601 string | null
sport: run | running | cycling | bike | strength | walk | other | unknown
sub_sport: string | null
duration_minutes: float | null
distance_km: float | null
avg_speed_kmh: float | null
max_speed_kmh: float | null
avg_hr_bpm: float | null
max_hr_bpm: float | null
avg_power_w: float | null
max_power_w: float | null
avg_cadence_rpm: float | null
max_cadence_rpm: float | null
elevation_gain_m: float | null
elevation_loss_m: float | null
calories_kcal: float | null
intensity_estimate: easy | moderate | hard | race_or_test | unknown
hard_session: bool
training_load_points: int | null
hr_zones:
  z1:
    range: string | null
    seconds: int | null
    time: string | null
    percent: float | null
```

`detail_file` points to the high-resolution workout archive. Do not open it during normal summary generation.

# Schemas

## Weekly Training Summary

```yaml
summary_type: weekly_training_summary
week_start: YYYY-MM-DD
week_end: YYYY-MM-DD
run:
  sessions: int
  total_distance_km: float | null
  total_duration_minutes: int | null
  longest_run_km: float | null
  longest_run_minutes: int | null
  hard_sessions: int
bike:
  sessions: int
  total_distance_km: float | null
  total_duration_minutes: int | null
  longest_ride_km: float | null
  longest_ride_minutes: int | null
  hard_sessions: int
combined:
  total_sessions: int
  total_duration_minutes: int | null
  total_load_points: int | null
  active_days: int
  rest_days: int
  hard_days: int
intensity_distribution:
  easy_percent: int | null
  moderate_percent: int | null
  hard_percent: int | null
  confidence: low | medium | high
flags: string[]
notes: string[]
data_quality:
  confidence: low | medium | high
  missing_fields: string[]
```

## Rolling Training Summary

```yaml
summary_type: rolling_training_summary
period_days: 28 | 56 | 84
period_start: YYYY-MM-DD
period_end: YYYY-MM-DD
run:
  weekly_average_distance_km: float | null
  weekly_average_duration_minutes: int | null
  peak_week_distance_km: float | null
  longest_run_km: float | null
  sessions: int
bike:
  weekly_average_duration_minutes: int | null
  peak_week_duration_minutes: int | null
  longest_ride_minutes: int | null
  sessions: int
combined:
  average_sessions_per_week: float | null
  average_training_minutes_per_week: float | null
  average_load_points_per_week: float | null
  total_load_points: int | null
  hard_sessions_per_week_average: float | null
trend:
  volume_direction: increasing | stable | decreasing
  intensity_direction: increasing | stable | decreasing | unknown
  consistency: low | medium | high
flags: string[]
coach_state_inputs:
  recent_load_7d_points: int | null
  recent_load_28d_points: int | null
  recent_load_56d_points: int | null
  recent_load_84d_points: int | null
  recent_load_7d_minutes: int | null
  recent_load_28d_minutes: int | null
  hard_sessions_last_7d: int | null
  hard_sessions_last_28d: int | null
  longest_run_last_28d_km: float | null
  longest_ride_last_28d_minutes: int | null
  fatigue_or_pain_mentions: string[]
data_quality:
  confidence: low | medium | high
  missing_fields: string[]
```
