# Intensity Classification Rules

Classify intensity conservatively and avoid false precision.

## Priority Order

1. Explicit workout label/title (interval, race, TT, FTP)
2. Power-zone distribution (cycling)
3. Heart-rate zone distribution
4. Pace/speed relative to recent baseline
5. Perceived effort and notes
6. Unknown

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
