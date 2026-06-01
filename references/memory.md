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

**Top recommendations:** [what moved up/down in the ranking, what's newly #1, what dropped off]
**Still unaddressed from last run:** [recommendations from the previous run that weren't built]
**Signal changes:** [new external mentions found, issues filed, competitor moves]
**Personas:** [new personas added / old ones dropped / confidence changes]
**Goal:** [note if the builder's optimization goal changed since last run — this re-ranks everything]
```

The "still unaddressed" section is the most valuable part — it shows whether recommendations are being acted on or accumulating. If the same recommendation appears 3 runs in a row unbuilt, it's either not actually a priority or it's harder to build than it looked — say so, and consider dropping its rank.

## Snapshot format

The snapshot file should be the full analysis output — no trimming. Future runs load the full content for diffing.

Add a metadata header using structured status tracking:
```markdown
---
project: <repo-slug>
date: <YYYY-MM-DD>
goal: <the confirmed optimization goal — e.g. launch conversion, retention>
signal_quality: High/Medium/Low
top_recommendation: <the #1 ranked item>
personas: [list of persona names]
recommendations:
  - item: "short label"
    type: Gap            # Gap / Build / Elevation
    rank: 1              # position in the best-first list
    status: open         # open / addressed / deferred
    addressed_date: null # fill with YYYY-MM-DD when addressed
---

[full analysis output]
```

**On each run:** When producing the diff section, compare the new ranked list against the previous snapshot's metadata. For any item where `status: addressed`, note it as closed with the `addressed_date`. For any item `status: open` appearing in the prior snapshot, include it in "Still unaddressed." Note rank movement — an item that was #4 and is now #1 (or vice versa) is a signal worth calling out, especially if the goal changed.

**Updating status:** When an item is completed between runs, update the previous snapshot's metadata before saving the new one — this is what makes the "Still unaddressed" list accurate rather than cumulative noise.

## Retention

Keep all snapshots — don't overwrite. The history is the value. A project with 6 monthly snapshots tells a richer story than one with only the latest.

## Time-series insight (3+ snapshots)

When 3 or more snapshots exist, look for patterns:
- **Consistent gaps** — appear every run → genuinely hard to build or genuinely deprioritized
- **Resolved gaps** — appeared, then disappeared → presumably built
- **Drifting personas** — persona composition shifting → user base evolving
- **Elevation move convergence** — same move recommended repeatedly → strong signal it's the right bet

Summarize trend lines in the diff section when 3+ snapshots are available.
