# whats-next

> A Claude Code skill for persona-based strategic product analysis.

Tell it a GitHub repo, a project description, or paste your support emails — it identifies your top 5 users, surfaces the gaps they hit, and gives you three layers of recommendations: what to fix now, what to build next, and what would change what your project *is*.

## What it produces

Five personas, three layers, one diff.

```
# whats-next: finance-assistant
Signal: 17★ repo, 1 closed issue, Reddit (vibe-coder building own tool from scratch),
        HN (Era Context competitor launched), previous run 4 days ago

## The 5 Personas

### The FIRE Obsessive — Has a spreadsheet, wants Monte Carlo to be a living number
Evidence: Monte Carlo + FIRE mode built early — suggests this user drove initial features
Gap: portfolio values require manual entry; FIRE confidence goes stale between sessions
Demand: daily price sync for tickers already stored (Yahoo Finance, no auth needed)

### The Vibe-Coder — Spent 20 hours building own finance tool from scratch
Evidence: Reddit r/ClaudeAI post — user doesn't know finance-assistant exists, or
          hit the git clone + pip install + settings.json wall and quit
Gap: install journey is a developer workflow; this persona hits a wall at step 1
Demand: Claude Projects template — paste one file, start chatting in 2 minutes

[3 more personas...]

## Layer 1 — Close the Gaps
### 1. Hardcoded version string ★★★★★  [Effort: small]
skill.py line 132 prints "3.1.2" regardless of __version__ — embarrassing on --doctor

## Layer 2 — Build Next
### 1. Live portfolio price sync ★★★★★  [Effort: medium]
Source: FIRE Obsessive gap + Depth lens (what keeps satisfied users going deeper?)
Yahoo Finance API, no auth, current_value = units × price, 6h TTL, skips cash/real_estate

## Layer 3 — Elevation Moves
### The always-on financial layer
What it changes: from "a skill you invoke" to "financial context in every Claude conversation"
Competitor context: Era Context (MCP competitor) can't copy privacy-first ambient awareness
The bet: works only if hook latency < ~100ms and false-positive injection rate < 10%

## What changed since 2026-05-13
Still unaddressed: live portfolio prices (Layer 2 #1 from last run — still not built)
New gap: Claude Projects template surfaced after vibe-coder Reddit post found
Addressed: hardcoded version string — fixed in v3.2.0
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

**Phase 3 — Three-layer synthesis**
Layers 1 and 2 ranked by genuine overlap — gap→feature traces required, not just persona lists. Layer 3 elevation moves must pass an elevation test (changes who the project serves, not just what it does) and include a single falsifiable bet with measurability requirements.

**Phase 3.5 — Coverage check**
Before outputting, verifies every persona gap and lens finding landed somewhere. Gaps that don't make the layers appear in "What we set aside" with a reason — so nothing silently disappears.

**Phase 4 — Snapshot + diff + action offer**
Saves a snapshot to `~/.claude/skills/whats-next-workspace/snapshots/`. On subsequent runs, diffs against the previous snapshot: which gaps are new, which were addressed, and — most usefully — which are *still unaddressed* after multiple runs. The "Still unaddressed" list is the highest-signal output for repeat users. Offers to action the top recommendation: write a spec, find the relevant code, draft a user validation question, or start a competitor deep-dive.

## References

The skill bundles five reference files used at runtime:

| File | Purpose |
|------|---------|
| `references/skill-mode.md` | Alternate signal collection when input is a Claude Code skill |
| `references/external-signals.md` | HN/Reddit/web APIs, user signal injection from pasted content |
| `references/competitors.md` | Finding alternatives, comparing positioning, identifying moats |
| `references/actioning.md` | Spec kickoff, effort estimation, validation prompts |
| `references/memory.md` | Snapshot format, diff logic, time-series comparison |

## Why three layers

Most "what to build next" tools produce a feature list. The three-layer structure distinguishes:

- **Layer 1** — you're losing users *today* to friction that doesn't require new features to fix
- **Layer 2** — new capabilities ranked by how many distinct user types want them (3 quiet personas > 1 loud one)
- **Layer 3** — bets that require the project to become something different, not just better

The "Don't build yet" section gives a specific backfire mechanism for each item, not a prioritization judgment. "This feature works against itself because X" is a reason. "Not a priority" is not.

## Developed with

Built and iteratively improved using Claude Code's skill-creator workflow — 10 autonomous improvement iterations across signal collection, persona quality, lens additionality, and coverage checking.
