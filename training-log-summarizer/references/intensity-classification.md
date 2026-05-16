# Intensity Classification Rules

Classify intensity conservatively and avoid false precision.

## Priority Order

1. `intensity_estimate` from `Compact Workout Facts`, if present
2. `hard_session` from `Compact Workout Facts`, if present
3. Explicit workout label/title (interval, race, TT, FTP)
4. Power-zone distribution (cycling)
5. Heart-rate zone distribution from `hr_zones`
6. Pace/speed relative to recent baseline
7. Perceived effort and notes
8. Unknown

## Labels

- `easy`
- `moderate`
- `hard`
- `unknown`

## Hard Session Signals

A session is hard when one or more are present:

- Interval workout markers
- Race/TT/FTP indicators
- Sustained high HR pattern
- High normalized power pattern
- Notes indicate hard effort

## Confidence

- `high`: direct indicators available (label, power/HR zones)
- `medium`: indirect signals available (notes + partial metrics)
- `low`: sparse or missing intensity signals

When uncertain, choose `unknown` and lower confidence.


## Heart-Rate Zone Handling

When compact workout facts include `hr_zones`, use zone percentages and seconds directly. Do not open full detail JSON just to compute intensity.

Suggested conservative mapping:

- Mostly Z1-Z2 with little Z3+ -> `easy`
- Substantial Z3 or some Z4/Z5 -> `moderate`
- Sustained Z4/Z5, race/test labels, or `hard_session: true` -> `hard`

If `intensity_estimate` is already present, use it as the primary label unless clearly contradicted by `hard_session: true` or explicit race/test labels.

## Normalized Load

When `training_load_points` exists, use it for load aggregation. Do not recompute it unless it is missing and enough compact fields are available.
