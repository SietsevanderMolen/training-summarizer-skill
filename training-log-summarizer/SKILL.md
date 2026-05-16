---
name: training-log-summarizer
description: summarize parsed running and cycling workout memories into weekly and rolling training summaries for downstream coaching. use when asked to aggregate recent exercise history, create weekly run/bike summaries, produce 4/8/12-week training rollups, detect basic training-pattern flags, prepare compact coach-readable training state from daily workout memories, or summarize openclaw compact workout blocks marked with OPENCLAW_FIT_WORKOUT_START and OPENCLAW_FIT_WORKOUT_END.
---

# Training Log Summarizer

## Purpose

Convert parsed daily workout memories into compact, factual summaries for downstream coaching.

This skill records what happened and highlights training patterns. It does not prescribe what to do next.

## Use This Skill When

- Asked to summarize weekly or rolling training from parsed daily activity memories
- Asked to produce coach-readable state inputs
- Asked to compare recent volume, consistency, or intensity trends

## Do Not Use This Skill For

- Raw `.fit` parsing or file ingestion
- Training plan prescriptions
- Medical diagnosis or injury advice
- Inferring thresholds or zones that are not provided

## Input Expectations

Input is already-parsed daily activity memory records. Schema may vary by source.

### Preferred OpenClaw Compact Workout Blocks

The current preferred input format is generated inside removable marker blocks in daily memory files:

```text
<!-- OPENCLAW_FIT_WORKOUT_START source_file="..." -->
...
<!-- OPENCLAW_FIT_WORKOUT_END source_file="..." -->
```

During normal weekly and rolling summarization:

- Read only content inside `OPENCLAW_FIT_WORKOUT_START` / `OPENCLAW_FIT_WORKOUT_END` blocks.
- Use the `Compact Workout Facts` YAML block as the primary source of truth.
- Use the `Heart Rate Zones` section only as a fallback or human-readable check.
- Do not open `workout_detail_ref` JSON files unless a specific workout is being debugged or required fields are missing.
- Ignore prior summary files, coach state files, Telegram logs, and files in `/workout_details/`.

Expected compact shape:

```yaml
memory_type: daily_workout
source: fit-parser
source_file: 2026-03-02_22-04-00_zw_2085213439378079792.fit
detail_file: /home/user/.openclaw/workspace/workout_details/2026-03-02_22-04-00_zw_2085213439378079792.json
date: 2026-03-02
start_time: 2026-03-02T21:04:00+00:00
sport: cycling
sub_sport: virtual_activity
duration_minutes: 60.85
distance_km: 25.1
avg_hr_bpm: 133
max_hr_bpm: 144
avg_power_w: 103
max_power_w: 243
avg_cadence_rpm: 74
max_cadence_rpm: 86
elevation_gain_m: 130
elevation_loss_m: 124
calories_kcal: 361
intensity_estimate: easy
hard_session: false
training_load_points: 61
hr_zones:
  z1:
    range: "55-65%"
    seconds: 492
    time: "8:12"
    percent: 19.5
```

### Legacy Parsed Daily Activity Records

Older daily memories may use a generic activity-list shape:

```yaml
date: 2026-05-14
activities:
  - type: run
    source: zwift
    duration_minutes: 42
    distance_km: 7.8
    avg_heart_rate: 148
    max_heart_rate: 174
    avg_pace: "5:23/km"
    elevation_m: 62
    perceived_effort: null
    notes: "felt good"
```

Treat missing fields as unknown. Do not invent values.

## Workflow

1. Read relevant daily memory files for the requested period.
2. Extract only generated workout blocks between `OPENCLAW_FIT_WORKOUT_START` and `OPENCLAW_FIT_WORKOUT_END`.
3. Parse each block's `Compact Workout Facts` YAML as the primary source.
4. Normalize activities into categories: `run`, `bike`, `strength`, `walk`, `other`.
5. Build weekly aggregation (ISO week: Monday-Sunday).
6. Build rolling summaries when requested (4/8/12 weeks = 28/56/84 days).
7. Use `training_load_points` for normalized load aggregation when present; also keep duration-minute totals.
8. Classify intensity conservatively from `intensity_estimate`, `hard_session`, and `hr_zones`.
9. Detect factual flags from available data.
10. Output structured YAML or Markdown with explicit date boundaries.
11. Preserve uncertainty, missing-data notes, and confidence.

## Source Selection and Exclusions

For normal scheduled summaries, never scan the entire memory directory as undifferentiated text. Select only date-based daily memory files in the requested period, then extract marked workout blocks from those files.

Exclude these inputs unless explicitly requested:

- `training_summary_*.md`
- `coach_state*.md`
- Telegram or notification logs
- files outside the requested date range
- `/home/user/.openclaw/workspace/workout_details/*.json`

The `/workout_details/` archive contains high-resolution parser JSON and may be large. Open those files only for specific workout debugging, missing required compact fields, or explicit user requests.


## Training Load Aggregation

When `training_load_points` is available in compact workout facts, aggregate it for weekly and rolling summaries.

Include both:

- duration-based totals, such as `total_duration_minutes`
- normalized load totals, such as `total_load_points`

For rolling summaries, include coach-readable load inputs:

```yaml
coach_state_inputs:
  recent_load_7d_points: null
  recent_load_28d_points: null
  recent_load_56d_points: null
  recent_load_84d_points: null
  recent_load_7d_minutes: null
  recent_load_28d_minutes: null
  hard_sessions_last_7d: null
  hard_sessions_last_28d: null
```

If load points are missing, keep `*_load_points` fields as `null`, aggregate minutes normally, and add `training_load_points` to `data_quality.missing_fields`.

## Normalization Rules

- Treat Zwift run-like activities as `run`.
- Treat Zwift ride-like activities as `bike`.
- If activity type is ambiguous, map to `other` and mention in notes if relevant.
- A day is an `active_day` if at least one activity exists.
- A `rest_day` has zero activities.

## Intensity Classification

Use signals in this priority order:

1. `intensity_estimate` from `Compact Workout Facts`, if present
2. `hard_session` from `Compact Workout Facts`, if present
3. Explicit workout label/title
4. Power zones (cycling)
5. Heart-rate zones from `hr_zones`
6. Pace/speed vs known baseline
7. Perceived effort or notes
8. Fallback: `unknown`

Allowed labels:

- `easy`
- `moderate`
- `hard`
- `unknown`

Do not overclassify. If evidence is weak, keep intensity as `unknown` and lower confidence.

## Hard Session Detection

Mark a session as hard if one or more are true:

- interval workout detected
- race/time trial/FTP test
- high HR sustained for substantial duration
- high normalized power or high power-zone distribution
- notes indicate hard effort
- workout title implies intensity

## Required Output Types

Produce one or more of:

1. Weekly training summary
2. Rolling 4-week summary
3. Rolling 8-week summary
4. Rolling 12-week summary
5. Coach-readable compact state input

### Weekly summary schema

```yaml
summary_type: weekly_training_summary
week_start: YYYY-MM-DD
week_end: YYYY-MM-DD
run:
  sessions: 0
  total_distance_km: 0.0
  total_duration_minutes: 0
  longest_run_km: null
  longest_run_minutes: null
  hard_sessions: 0
bike:
  sessions: 0
  total_distance_km: 0.0
  total_duration_minutes: 0
  longest_ride_km: null
  longest_ride_minutes: null
  hard_sessions: 0
combined:
  total_sessions: 0
  total_duration_minutes: 0
  total_load_points: null
  active_days: 0
  rest_days: 7
  hard_days: 0
intensity_distribution:
  easy_percent: null
  moderate_percent: null
  hard_percent: null
  confidence: low
flags: []
notes: []
```

### Rolling summary schema

```yaml
summary_type: rolling_training_summary
period_days: 28
period_start: YYYY-MM-DD
period_end: YYYY-MM-DD
run:
  weekly_average_distance_km: null
  weekly_average_duration_minutes: null
  peak_week_distance_km: null
  longest_run_km: null
  sessions: 0
bike:
  weekly_average_duration_minutes: null
  peak_week_duration_minutes: null
  longest_ride_minutes: null
  sessions: 0
combined:
  average_sessions_per_week: null
  average_training_minutes_per_week: null
  hard_sessions_per_week_average: null
trend:
  volume_direction: stable
  intensity_direction: stable
  consistency: medium
flags: []
coach_state_inputs:
  recent_load_7d_points: null
  recent_load_28d_points: null
  recent_load_56d_points: null
  recent_load_84d_points: null
  recent_load_7d_minutes: null
  recent_load_28d_minutes: null
  hard_sessions_last_7d: null
  hard_sessions_last_28d: null
  longest_run_last_28d_km: null
  longest_ride_last_28d_minutes: null
  fatigue_or_pain_mentions: []
```

## Flag Detection

Use factual flags only. Never prescribe actions.

Potential flags:

- `two_hard_days_in_48_hours`
- `three_or_more_hard_sessions_this_week`
- `running_volume_up_more_than_10_percent`
- `bike_volume_up_more_than_20_percent`
- `no_full_rest_day_last_9_days`
- `longest_run_in_6_weeks`
- `longest_ride_in_6_weeks`
- `sudden_duration_spike`
- `pain_or_injury_mentioned`
- `illness_mentioned`
- `low_data_confidence`
- `missing_days`
- `first_activity_after_break`

If flags are not computable due to missing data, omit that flag and mention uncertainty.

## Missing Data and Confidence

- Keep important unknown values as `null`.
- Omit nonessential fields if absent.
- Add `low_data_confidence` flag when data is sparse/incomplete.
- Include missing-data context:

```yaml
data_quality:
  confidence: low
  missing_fields:
    - heart_rate
    - power
    - perceived_effort
```

Confidence guidance:

- `high`: most activities include clear duration + type and at least one intensity signal.
- `medium`: core volume fields are present but intensity signals are partial.
- `low`: substantial missing days/fields, sparse sessions, or conflicting signals.

## Guardrails

Do not:

- prescribe training plans
- diagnose injuries
- provide medical advice
- state certainty on safety/unsafety
- infer FTP/threshold/max HR/zones unless explicitly provided

Use cautious language:

- "Data suggests..."
- "Based on available activity summaries..."
- "Confidence: low/medium/high..."

## Output Style

- Prefer YAML for machine readability.
- Markdown is acceptable if structured and compact.
- Always include explicit date boundaries (`week_start`, `week_end`, `period_start`, `period_end`).
- Keep summaries concise and factual.

## Prompt Examples Supported

- Summarize last week’s training.
- Create a rolling 4-week training summary for the coach.
- Update my weekly training summary from this week’s daily workout memories.
- Prepare coach-readable training state from the last 12 weeks.
- Compare this week’s running volume to the previous four weeks.
