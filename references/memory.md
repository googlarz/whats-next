# Memory — Snapshots and Time-Series Comparison

A one-shot analysis resets every time. Snapshots make analysis compound — each run builds on the last.

## Snapshot storage

Save every completed analysis to:
```
~/.claude/skills/whats-next-workspace/snapshots/<repo-slug>-<YYYY-MM-DD>.md
```

For portfolio analyses, use a combined slug:
```
~/.claude/skills/whats-next-workspace/snapshots/portfolio-proton-signal-suunto-2026-05-17.md
```

## On each run: check for previous snapshot

Before starting analysis, check:
```bash
ls ~/.claude/skills/whats-next-workspace/snapshots/ | grep <repo-slug>
```

If a previous snapshot exists, load it. Note the date at the top of the output:
```
**Signal sources:** repo data, 12 HN mentions, competitor scan, previous run from 2026-04-17 (30 days ago)
```

## Diff logic

When a previous snapshot exists, after completing the new analysis, produce a diff section:

```
## What changed since [DATE]

**Personas:** [new personas added / old ones dropped / confidence changes]
**Top gaps:** [gaps that appeared, disappeared, or changed priority]
**Recommendations:** [what moved up/down in Layer 1-2, what's new in Layer 3]
**Signal changes:** [new external mentions found, issues filed, competitor moves]

**Still unaddressed from last run:** [Layer 1/2 items from previous run that weren't built]
```

The "still unaddressed" section is the most valuable part — it shows whether recommendations are being acted on or accumulating. If the same gap appears 3 runs in a row, it's either not actually a priority or it's harder to build than it looked.

## Snapshot format

The snapshot file should be the full analysis output — no trimming. Future runs load the full content for diffing.

Add a metadata header:
```markdown
---
project: <repo-slug>
date: <YYYY-MM-DD>
signal_quality: High/Medium/Low
top_recommendation: <Layer 2 item 1>
personas: [list of persona names]
layer1_items: [short label for each Layer 1 fix, used for "still unaddressed" diff]
layer2_items: [short label for each Layer 2 feature, used for "still unaddressed" diff]
---

[full analysis output]
```

## Retention

Keep all snapshots — don't overwrite. The history is the value. A project with 6 monthly snapshots tells a richer story than one with only the latest.

## Time-series insight (3+ snapshots)

When 3 or more snapshots exist, look for patterns:
- **Consistent gaps** — appear every run → genuinely hard to build or genuinely deprioritized
- **Resolved gaps** — appeared, then disappeared → presumably built
- **Drifting personas** — persona composition shifting → user base evolving
- **Elevation move convergence** — same move recommended repeatedly → strong signal it's the right bet

Summarize trend lines in the diff section when 3+ snapshots are available.
