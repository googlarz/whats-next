# whats-next

> A Claude Code skill for persona-based strategic product analysis.

Tell it a GitHub repo, a project description, or paste your support emails — it identifies your top 5 users, surfaces the gaps they hit, and gives you three layers of recommendations: what to fix now, what to build next, and what would change what your project *is*.

## What it produces

```
## The 5 Personas
[5 users spanning technical → non-technical, each with evidence, context, gap, demand]

## Layer 1 — Close the Gaps
[Friction points costing you users today, with effort estimates]

## Layer 2 — Build Next
[Ranked by persona overlap, with gap→feature traces]

## Layer 3 — Elevation Moves
[1-3 bets that change what the project is, with falsifiable conditions]

## Don't Build Yet
[Tempting moves with specific backfire mechanisms]

## What We Set Aside
[Gaps that didn't make the layers, and why]
```

## Install

```bash
# Add to your Claude Code skills
git clone https://github.com/googlarz/whats-next ~/.claude/skills/whats-next
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

**Phase 4 — Snapshot + action offer**
Saves a snapshot for time-series comparison. Offers to action the top recommendation: write a spec, find the relevant code, draft a user validation question, or start a competitor deep-dive.

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
