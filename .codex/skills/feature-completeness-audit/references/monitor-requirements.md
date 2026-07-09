# Monitor Requirements Baseline

This file freezes the baseline for the feature-completeness audit. Do not fetch the original Feishu doc at runtime unless the user explicitly asks to refresh the baseline.

## Included Requirements

### P0

- `UR-1 Quick setup`
  - The user can register their own product and target competitors, then start monitoring quickly.
- `UR-3 Continuous, automated collection and interpretation`
  - The system periodically collects competitor signals, identifies meaningful changes, and interprets them without manual intervention.
- `UR-4 Structured intelligence consumption`
  - The user can browse, filter, archive, and inspect intelligence in a unified inbox or equivalent structured surface.
- `UR-5 Actionable recommendations`
  - Intelligence should explain not just what changed, but what the user should do and why.

### P1

- `UR-6 Proactive notification`
  - The system can proactively push important intelligence through channels such as email or Slack.

## Success Criteria

The audit must explicitly verify these outcomes:

1. After setup, the user can receive valuable intelligence on a recurring basis without manual operation.
2. Intelligence includes a complete `change -> intent -> recommendation` chain, not just raw data.
3. A feedback mechanism exists and can help improve intelligence quality over time.

## Scope Boundaries

Treat these as out of scope for this audit unless the user overrides the baseline:

- automatic discovery of new competitors across the whole web
- reverse monitoring or diagnostics for the user’s own product

## Evidence Expectations

- Marketing copy alone is not enough for a passing score.
- For P0/P1 pass decisions, prefer product surfaces, actual flows, and verifiable data interactions.
- If a feature is likely hidden behind auth, mark the relevant requirement as `blocked` unless the available evidence is still sufficient.
