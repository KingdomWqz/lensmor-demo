# Scoring Rubric

Score five dimensions from `0.0` to `1.0`. Total score is the sum, capped at `5.0`.
Intermediate scores are allowed when the evidence does not cleanly fit the anchor examples below.

## Bug-Aware Scoring Rule

- If a requirement is clearly implemented on primary product surfaces or verified APIs, but a secondary page, log view, or supporting surface has a real bug, do not automatically downgrade the requirement to `partial` or `missing`.
- In this case, prefer keeping the status as `implemented` and apply a modest score deduction, typically around `0.05` to `0.2`, depending on how much the bug reduces completeness, trust, or observability.
- Only downgrade to `partial` when the bug breaks the core requirement itself, prevents meaningful use, or leaves the key outcome unverified.

## Dimension 1: Quick Setup (`UR-1`)

- `1.0`: Clear flow to define monitoring targets, start monitoring, and stop/pause monitoring without destructive deletion
- `0.8`: Creation and start flow work, but stop/pause is missing or only achievable by deleting the target, which creates a likely risk that historical intelligence will no longer be accessible
- `0.75`: Strong setup surface, but one critical step is unclear or only partially evidenced
- `0.5`: Some setup/configuration UI exists, but it is incomplete or weakly evidenced
- `0.25`: Only marketing or static claims about easy setup
- `0.0`: No meaningful setup evidence

## Dimension 2: Continuous Collection and Interpretation (`UR-3`)

- `1.0`: Ongoing collection is evidenced by recurring updates, timestamps, feeds, scheduled behavior, or alert generation, plus analysis/interpretation
- `0.75`: Strong analysis output exists and automation is highly implied, but cadence or recurrence is not directly evidenced
- `0.5`: Some automated or analytical behavior is present, but the continuous collection loop is incomplete
- `0.25`: Static examples or isolated reports only
- `0.0`: No evidence

## Dimension 3: Structured Intelligence Consumption (`UR-4`)

- `1.0`: Structured inbox/feed/list with detail view and organization tools such as filters, archive, status, or equivalent
- `0.75`: Strong list and detail experience, but weak organization controls
- `0.5`: Basic list or feed exists without a complete inbox workflow
- `0.25`: Static sections describing an inbox without a real product surface
- `0.0`: No evidence

## Dimension 4: Actionable Recommendations (`UR-5`)

- `1.0`: Output clearly includes change, intent, and recommended action
- `0.75`: Change plus recommendation exists, but strategic intent is thin
- `0.5`: Change plus explanation exists, but recommendation is missing
- `0.25`: Vague “insights” or “AI analysis” without concrete decision support
- `0.0`: No evidence

## Dimension 5: Notifications and Feedback (`UR-6` + feedback success criterion)

- `1.0`: Proactive notification is evidenced and a feedback mechanism exists
- `0.75`: Proactive notification is evidenced, but no feedback mechanism is found
- `0.5`: Notification capability is partially evidenced and feedback exists
- `0.25`: Feedback exists, but proactive notification is not evidenced
- `0.0`: Neither is found

## Requirement Status Labels

- `implemented`: verified with product evidence
- `partial`: product evidence exists, but the requirement is incomplete
- `claimed`: only static or marketing claims
- `missing`: no supporting evidence found
- `blocked`: cannot verify because access is restricted or the flow is broken

## P0/P1 Gate Rules

- If any P0 dimension scores below `0.5`, set `p0_p1_passed` to `false`
- If Dimension 5 scores below `0.5`, set `p0_p1_passed` to `false`
- Missing feedback alone should not force failure if proactive notification is fully evidenced, but it should lower Dimension 5 to `0.75`

## Verdict Mapping

- `pass`: `p0_p1_passed=true` and total score `>= 4.0`
- `partial`: `p0_p1_passed=false` or total score between `2.0` and `3.99`
- `fail`: total score `< 2.0`
