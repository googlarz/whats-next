# whats-next

> A Claude Code skill for persona-based strategic product analysis.

Tell it a GitHub repo, a project description, or paste your support emails — it confirms what you're optimizing for, identifies your top 5 users, surfaces the gaps they hit, and hands you **one comprehensive list of recommendations ranked best-first against your actual goal** — each with its reasoning attached, and a scannable numbered table at the end.

## What it produces

A goal, a ranked recommendation list with inline analysis, the personas behind it, a diff, and a table — in that order.

```
# whats-next: finance-assistant
Optimizing for: retention (keep existing users coming back)
Signal: 17★ repo, 1 closed issue, Reddit (vibe-coder building own tool), HN (Era Context
        competitor launched), previous run 4 days ago

## Recommendations — best first
Ranked by impact toward retention ÷ effort.

### 1. Live portfolio price sync · Build · Impact ★★★★★ · Effort M
Why it ranks #1 (toward retention): FIRE confidence goes stale between sessions — the
   number users come back for is wrong until they hand-edit. Fixing this is the single
   biggest pull back into the app.
Source: FIRE Obsessive gap + Depth lens (what keeps satisfied users going deeper?)
Mechanism: Yahoo Finance API, no auth, current_value = units × price, 6h TTL.
What would change my mind: if users treat it as a one-shot planner, not a recurring check.

### 2. Hardcoded version string · Gap · Impact ★★★☆☆ · Effort S
Why it ranks here: trust ding on --doctor, cheap to fix, but doesn't itself drive returns.
Mechanism: skill.py line 132 → read __version__ instead of the literal "3.1.2".
What would change my mind: nothing — it's just correct. Ships in an afternoon.

### 3. The always-on financial layer · Elevation · Impact ★★★★☆ · Effort L
What it changes: from "a skill you invoke" to "financial context in every conversation."
Competitor context: Era Context (MCP competitor) can't copy privacy-first ambient awareness.
What would change my mind (the bet): works only if hook latency < ~100ms and
   false-positive injection rate < 10%. Requires: latency + injection instrumentation.

[more recommendations, ranked...]

## Personas behind these
[5 persona cards — the analytical basis for the ranking above]

## What changed since 2026-05-13
Still unaddressed: live portfolio prices (was #1 last run — still not built)
Addressed: nothing since last run

## At a glance
| # | What | Type | Impact | Effort | Description |
|---|------|------|--------|--------|-------------|
| 1 | Live price sync | Build | High | M | Stale FIRE number is what pulls users back |
| 2 | Version string fix | Gap | Low | S | Trust ding on --doctor, afternoon fix |
| 3 | Always-on layer | Elev | High | L | From invoked tool to ambient context |
```

## Install

```bash
git clone https://github.com/googlarz/whats-next ~/.claude/skills/whats-next
```

Then add to `~/.claude/settings.json`:

```json
{
  "skills": [
    { "name": "whats-next", "path": "~/.claude/skills/whats-next" }
  ]
}
```

## Usage

Just describe your project:

```
what should I build next for googlarz/finance-assistant?
```

```
analyze my MCP server portfolio — proton-mail-bridge-client, signal-mcp, suunto-mcp
```

```
whats next for my whats-next skill
```

```
[paste support emails / Discord messages / app store reviews]
```

It also handles Claude Code skills as inputs — no GitHub repo needed.

## Use without Claude Code

If you use Claude.ai (browser) rather than Claude Code, paste the skill's core instructions directly into a [Claude Project](https://claude.ai/projects):

1. Create a new Project at [claude.ai](https://claude.ai).
2. Open Project Settings → System Prompt.
3. Paste the contents of [`SKILL.md`](./SKILL.md) into the system prompt.
4. In any conversation in that Project, type your request: `what should I build next for [your project]?`

The core persona analysis, five lenses, and goal-ranked recommendations work without Claude Code. What you lose without Claude Code: snapshot memory across sessions (no run-to-run diff), live GitHub data collection, and skill-mode detection for analyzing other Claude Code skills as input.

## How it works

**Phase 1 — Signal collection**
Pulls from four sources: GitHub repo data (issues, PRs, releases, CHANGELOG), external conversations (HN Algolia API, Reddit, web search), competitor landscape, and previous run snapshots for time-series diff.

For Claude Code skills as input, switches to skill-mode: reads SKILL.md directly, searches the ecosystem for mentions, identifies competing skills and MCPs.

**Phase 2 — Persona identification**
Builds 5 personas spanning the full technical spectrum. Each has evidence from actual signal (not archetypes), a gap (what's wrong now), and a demand (what they'd want built). Anti-overlap enforced — each persona must drive at least one different recommendation.

**Phase 2.5 — Five product lenses**
Before synthesizing, runs 5 structured questions that persona gaps miss:
1. Depth — what would make satisfied users go deeper?
2. Breadth — what's blocking adoption?
3. Promise gap — what does the README promise but not fully deliver?
4. Adjacency — what could this tool absorb from adjacent steps?
5. Category leadership — what's keeping this from being the obvious choice?

Lens findings are labeled `(from: X lens)` in the output.

**Phase 3 — Rank and synthesize**
Folds persona gaps and lens findings into one comprehensive list, ranked best-first by **impact toward the confirmed goal ÷ effort**. Recommendations are interleaved by genuine rank, not grouped by type — a cheap fix that serves the goal outranks an expensive feature. Each carries a type tag (Gap / Build / Elevation), its source trace, a mechanism, and a "what would change my mind" line so you can override intelligently. Elevation moves additionally require competitor context and a falsifiable, measurable bet.

**Phase 3.5 — Coverage check**
Before outputting, verifies the #1 pick is genuinely the best move toward the goal (not the most exciting), that ranking isn't grouped by type, and that every persona gap and lens finding landed somewhere. Anything excluded appears in "What we set aside" with a reason — so nothing silently disappears.

**Phase 4 — Snapshot + diff + action offer**
Saves a snapshot to `~/.claude/skills/whats-next-workspace/snapshots/`. On subsequent runs, diffs against the previous snapshot: which recommendations are new, which were addressed, how the ranking moved, and — most usefully — which are *still unaddressed* after multiple runs. The "Still unaddressed" list is the highest-signal output for repeat users. Offers to action the top recommendation: write a spec, find the relevant code, draft a user validation question, or start a competitor deep-dive.

## References

The skill bundles five reference files used at runtime:

| File | Purpose |
|------|---------|
| `references/skill-mode.md` | Alternate signal collection when input is a Claude Code skill |
| `references/external-signals.md` | HN/Reddit/web APIs, user signal injection from pasted content |
| `references/competitors.md` | Finding alternatives, comparing positioning, identifying moats |
| `references/actioning.md` | Spec kickoff, effort estimation, validation prompts |
| `references/memory.md` | Snapshot format, diff logic, time-series comparison |

## Why goal-ranked, not just a list

Most "what to build next" tools produce a flat feature list, implicitly ranked by "how many people want it." But "best" is meaningless without an objective — the right next move for a builder chasing revenue differs from one chasing adoption or cutting maintenance load. So the skill confirms your goal up front and ranks **every** recommendation by impact toward *that* goal ÷ effort.

Each recommendation still carries a type, so you know what kind of move it is:

- **Gap** — you're losing users *today* to friction that doesn't require new features to fix
- **Build** — a new capability, driven by genuine overlap across distinct user types (3 quiet personas > 1 loud one)
- **Elevation** — a bet that requires the project to become something different, not just better

But the *ranking* cuts across all three: a cheap Gap that unblocks your goal beats an exciting Build that doesn't. The "Don't build yet" section gives a specific backfire mechanism for each excluded item, not a prioritization judgment. "This feature works against itself because X" is a reason. "Not a priority" is not.

## Developed with

Built and iteratively improved using Claude Code's skill-creator workflow — 10 autonomous improvement iterations across signal collection, persona quality, lens additionality, and coverage checking.
