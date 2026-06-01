---
name: whats-next
description: Analyze one or more projects by identifying the top 5 user personas across the full spectrum from technical to non-technical, surfacing gaps each persona hits, and synthesizing what to build next — including breakthrough moves that elevate the project to a new level. Use this skill whenever the user asks "what should I build next", "what do my users want", "what's missing from this project", "who uses this and what do they need", or wants strategic product direction for a repo or portfolio. Trigger even if the user just says "analyze my repos" or "what should I add to X" — persona-based product thinking is almost always the right lens. Works with GitHub URLs, repo names, local project paths, plain project descriptions, or pasted user conversations (emails, Discord, reviews, support tickets).
---

# whats-next

Strategic product direction through persona-based analysis. You identify who actually uses a project — spanning the full range from power users to casual non-technical users — give each persona a voice, surface the gaps they hit, and run five product lenses to catch what persona analysis misses. You then produce **one comprehensive list of recommendations, ranked best-first against the builder's actual goal**, each carrying its own analysis. The persona/lens machinery is the engine; the ranked recommendation list is the output.

Two things make this more than a generic "what should I build" answer: it ranks every recommendation by **impact toward a confirmed goal ÷ effort** (so the #1 pick is the best move *for this builder*, not the most exciting one), and it leads with the best recommendation and its reasoning rather than burying it under persona cards.

After analysis, you offer to action the top pick.

## When this skill applies

- "What should I build next for [project]?"
- "What do my users want?" / "What's missing?"
- "Analyze my repos" / "Give me product direction on Y"
- Any pasted user conversations, support emails, reviews, Discord logs

## Input gathering

Accept whatever the user gives you: GitHub repo names / URLs / local paths / plain descriptions / pasted user conversations / combinations.

If the user gives you nothing specific, ask: "Which project or projects? Or should I look at your GitHub profile as a whole?"

**If the user pastes raw user conversations:** treat as primary signal — outranks everything else. See `references/external-signals.md` → "User signal injection."

### Confirmation gate — mandatory before any analysis

Before collecting any signal, confirm two things: **what** you're analyzing and **what the user is optimizing for**. The goal is what makes "best recommendation" mean something — without it, "best" silently defaults to "most persona-overlap," which is a proxy, not necessarily best *for this builder*.

Infer a provisional goal from the input and offer it for correction:
- A pre-launch / WIP app → launch conversion
- A library with issues piling up → reducing maintenance load
- A growing tool with adoption signal → retention or revenue
- A profile-wide scan → portfolio coherence
- Don't know? Offer the two most likely and let the user pick.

Format:
> **Analyzing:** `<repo / skill path / description>`
> **Optimizing for:** `<inferred goal>` — revenue / adoption / retention / launch conversion / cutting maintenance load / learning / portfolio coherence. Correct me if it's something else.
> **Mode:** Standard / Skill-as-project / Portfolio
> I'll start once you confirm.

Wait for confirmation. If the user corrects the goal, **re-rank everything against the corrected goal** — it changes what counts as "best." The goal is not decoration; it is the ranking key in Phase 3.

---

## Phase 1 — Signal collection

**First: detect project type. This is a mandatory gate — run it before any other command.**

**Step 1a — Filesystem check (local paths only):**
```bash
ls <given-path>/SKILL.md 2>/dev/null
```
If the file exists → this is a skill. Stop. Do not continue to Step 1b or signal collection.

**Step 1b — Name/description check:**
If the input mentions "a skill I built", "a Claude Code skill", a skill name matching the available skills list, or a path that describes a skill — this is a skill. Stop.

**If either check triggers:**
Read `references/skill-mode.md` in full. Do not run `gh` commands. Do not collect signals. Do not form personas. Follow skill-mode.md's signal collection path entirely.

**Required output gate — must appear before Phase 2 begins:**
```
> Mode: Skill-as-project (skill-mode.md loaded)
```
If this line is absent from your output, you have not followed the detection path. Do not proceed to persona identification without it.

If neither check triggers, proceed with standard signal collection:

For everything else, collect from four sources in parallel:

**Source A — Repo data:**
```bash
gh repo view <owner/repo> --json name,description,stargazerCount,forkCount,topics,primaryLanguage,readme
gh issue list --repo <owner/repo> --limit 50 --json title,labels,comments,state
gh pr list --repo <owner/repo> --limit 20 --json title,body,state
gh release list --repo <owner/repo> --limit 10
```
CHANGELOG bugs are especially valuable — a locale bug means real users in that locale; a 50MB import fix means someone hit it.

**Source B — External conversations:** What people say outside the repo (HN, Reddit, web). See `references/external-signals.md`. These are the users who didn't care enough to file an issue — often the most representative.

**Source C — Competitor landscape:** What alternatives exist and how users compare them. Required before writing elevation moves. See `references/competitors.md`.

**Source D — Memory:** Check `~/.claude/skills/whats-next-workspace/snapshots/` for a previous run. If found, **read the full snapshot file** — it is the baseline for the diff. Note the date in the signal header: "previous run from DATE (N days ago)". See `references/memory.md` for diff format.

**Low-signal mode:** When signal quality is Low, do not just label and proceed. Execute the Low Signal Protocol:

1. **Label the quality** — `Signal quality: Low` in the signal header.
2. **Mark personas** — tag all inferred personas as `[Hypothesis]`.
3. **Generate a validation block** — append this section at the END of the output:

```
## Low Signal — Gather Before Re-Running

Signal quality is Low. The analysis above is hypothesis-based. To get higher-signal output, collect the following before re-running:

**3 validation questions to ask your users / community:**
1. [Domain-specific — e.g. for dev tools: "What made you try this instead of [most obvious alternative]?"]
2. [Gap-targeting — "What's the first thing you'd remove or change if you had 30 minutes?"]
3. [Adoption — "What almost stopped you from trying this?"]

**Signals to gather:**
- [ ] Any GitHub issues (even 1 is more than none)
- [ ] Any HN / Reddit / Discord mentions of the project name or category
- [ ] Any support emails or DMs from users
- [ ] A published GitHub repo (if local-only today)
```

Tailor the 3 questions to the project domain. Don't generate generic questions.

---

## Phase 2 — Persona identification

Always span **most technical → least technical**. The non-technical user is the most overlooked.

Archetypes (adapt — don't force):
1. Power user / developer
2. Practitioner — serious use, not a developer
3. Evaluator — hasn't committed yet
4. Casual / non-technical user
5. Team deployer — installs it for others

Ask: **who is the least technical person who could plausibly benefit?** That persona belongs in the set.

**Avoid overlap.** Each persona must drive at least one different recommendation. **Weight by signal volume** — 40 HN comments > 1 GitHub issue; reflect this in confidence.

**Anti-overlap test:** Before finalizing, ask: "If I merged Persona A and Persona B into one card, would any recommendation disappear?" If no — merge them and build a genuinely distinct persona instead. Two technical users with different feature requests are still the same persona if their underlying gaps are the same.

**Specificity floor:** Each persona's "Their context" must include at least one concrete lifestyle detail (not just a job title). Bad: "a developer who wants AI in their workflow." Good: "a developer who has Claude open in a browser tab all day and copies message text into it 10+ times before noon."

For each persona:
```
### [Persona Name] — [One-line role]
**Evidence:** [source + what it shows — must cite a specific signal, not "plausible" or "assumed"]
**Their context:** [goal + constraints + one concrete daily-life detail]
**Gap they hit:** [specific blocker costing them today — not a wish list item]
**What they'd demand next:** [concrete feature]
```

Gap ≠ demand. Gap is what's wrong now. Demand is what they want built. They're often at different levels of specificity.

**Evidence requirement:** If a persona has no signal support (no issue, no commit message, no HN comment, no release title pointing to this user type), mark it explicitly as `[Hypothesis]` and lower its weight in the ranking.

**Pre-launch persona construction:** For WIP / unreleased apps with no user data, the non-technical and evaluator personas cannot be grounded in usage evidence. Instead, ground them in **launch audience characteristics**: What audience is the Product Hunt / launch channel submission targeting? Who shares this type of app on Twitter/Reddit? What are the top App Store reviews for the nearest comparable? Use these as evidence sources when in-app signal is unavailable.

---

## Phase 2.5 — Five product lenses (pre-synthesis)

Before writing the recommendations, run these five questions. They surface recommendations that persona gaps miss — a happy power user has no gap, but there may still be a promise gap or adjacency move worth surfacing.

Work through each fully — do not skip a lens. Each must produce at least one finding, even if it's "lens confirms gap already captured by Persona X." If a lens produces nothing, state why explicitly (it becomes "What we set aside" material).

1. **Depth** — What would make existing satisfied users use this *more*? What keeps them from going deeper?

2. **Breadth** — What's blocking people who've heard of this but haven't adopted? What would bring in the next 10x of users? **For pre-launch / WIP projects:** This lens almost always surfaces distribution blockers (no App Store listing, no landing page, no demo video) that persona analysis misses because personas assume users already found the product.

3. **Promise gap** — What does the README/description/tagline promise that the project doesn't fully deliver yet? Read the first sentence of the README literally — does the product actually do that?

4. **Adjacency** — What do users do immediately *before* or *after* this tool? Could this tool absorb those steps and become stickier?

5. **Category leadership** — What's the single thing keeping this from being the obvious, default choice in its category? What does the category leader have that this doesn't?

Lens findings feed into the ranked recommendations alongside persona gaps. **All 5 lens labels must appear somewhere in the output** — either in a recommendation or in "What we set aside." If a lens label is missing from the output, the coverage check fails.

**This working section is internal — do not reproduce it in the output.** Only the findings appear, labeled in the recommendations where they land.

---

## Phase 3 — Rank and synthesize

Turn persona gaps + lens findings into **one comprehensive list of recommendations, ranked best-first**. The rank — not the layer — is the organizing principle of the output. The user wants the single best move at the top *with its reasoning attached*, then the next best, all the way down. Don't group by type and make the reader hunt; interleave and rank.

Each recommendation carries a **type tag** so the reader knows what kind of move it is:

- **Gap** — friction *actively costing users today*. A current user hits this and loses trust or stops. Test: "Would removing this make a user who already found the product stay?" If it needs a *new* user to show up first, it's a Build, not a Gap.
- **Build** — a new capability. Driven by genuine overlap across personas AND lenses.
- **Elevation** — a bet that changes what the project *is*, not just makes it better. Test: would you describe the project differently after this move?

### The ranking key
Rank across all three types **together**, by **impact toward the confirmed goal ÷ effort**. This is the heart of the upgrade:

- A cheap Gap fix that directly serves the goal **outranks** an expensive Build, even though "gaps" sound less exciting than "features." Don't rank by type.
- "Impact" means impact toward *this builder's goal* — not generic value. A feature 3 personas want but that doesn't move the stated goal ranks below a fix only 1 persona wants that unblocks the goal directly.
- Within an impact tie, lower effort wins.
- State the ranking logic once at the top of the list: "Ranked by impact toward [goal] ÷ effort."

### Each recommendation carries its analysis inline
The analysis travels **with** the recommendation — that is what "best recommendations with analysis first" means. Don't separate the list from the reasoning. For each:

- **Header line:** `[Name] · [Gap/Build/Elevation] · Impact ★★★★★ · Effort S/M/L`
- **Why it ranks here** — its impact toward the *goal* specifically. This is the sentence that justifies its position.
- **Source trace** — which persona gap(s) and/or lens finding(s) drive it. Only count a persona if their specific gap implies demand for this specific item. A rating without a basis is unverifiable.
- **Mechanism** — concretely what to build or change.
- **What would change my mind** — the one assumption that, if false, drops this item down or off the list. One line. This lets the builder, who knows things you don't, override intelligently.
- **Sequencing** — if it depends on another item shipping first, say so.

### Elevation discipline (applies to Elevation-tagged items)
Include **at least 2** Elevation moves from different angles: one that changes **who** the project serves, one that changes **how** value is delivered (pull → push, sync → async, individual → ambient).

**Elevation moves are ranked into the single best-first list like everything else** — they carry the `Elevation` tag inline and take their position by impact-toward-goal ÷ effort. Do **not** append them as a separate "Elevation moves" section at the bottom: that recreates the old Layer 3, breaks the "one ranked list" the user asked for, and pushes content past the table that must come last. If a big elevation bet ranks low because it's high-effort and long-horizon, that's honest — it's a direction, not your next action — and the reader still sees it in the list and the table.

- **Competitor context required:** name what competitors do or don't do. "Build X in a way structurally impossible for competitor Y to copy" is elevation. "Build X, which Y already does" is not.
- **Falsifiable bet:** the "what would change my mind" line for an Elevation item must be a single make-or-break condition, wrong-able: "Works only if X," not "works if it's good enough."
- **Measurability check:** if the project can't currently measure that bet, append "Requires: [what to instrument]." A bet that can't be measured is unfalsifiable in practice, however specific it sounds.

### Don't build yet
Specific backfire mechanism per item — not prioritization. "Building Y would require solving Z first, and Z is a harder problem than Y appears" is a reason. "Not a priority" is not.

---

## Phase 3.5 — Coverage check (quality gate)

Before outputting, verify every box is checked. If any fails, fix it before proceeding:

- [ ] **The #1 recommendation is genuinely the best move toward the stated goal** — not the most exciting, not the biggest. Re-check: does a cheaper item further down actually serve the goal better? If so, it should be #1. This is the single most important check.
- [ ] **Ranking is by goal-impact ÷ effort, not by type** — verify Gaps, Builds, and Elevation moves are interleaved by genuine rank, not grouped with all Gaps first.
- [ ] **Every recommendation carries its inline analysis** — type tag, impact, effort, why-it-ranks-here, source trace, mechanism, "what would change my mind."
- [ ] **Every persona's gap** appears as a recommendation, OR is explicitly in "Don't build yet" / "What we set aside" with a reason
- [ ] **All 5 lens labels appear** in the output — in a recommendation or in "What we set aside." Count them: Depth, Breadth, Promise gap, Adjacency, Category leadership. Missing label = fix before proceeding.
- [ ] **Every lens finding** not already covered by a persona gap appears somewhere, OR is explicitly set aside with a reason
- [ ] **At least one Gap-type recommendation exists** (no project has zero friction points today)
- [ ] **At least 2 Build-type recommendations** with verified overlap traces
- [ ] **At least one Elevation move** that changes who the project primarily serves, not just what it does — **ranked inline in the list, NOT in a separate trailing section**
- [ ] **Nothing follows the "At a glance" table** except the snapshot footer — no trailing "Elevation moves" block
- [ ] **"Don't build yet" has ≥1 item** with a backfire mechanism (no project has zero tempting-but-wrong moves)
- [ ] **Effort signal (S/M/L) on every recommendation**, not just the top one
- [ ] **Sequencing noted** where one item must ship before another is reachable. In particular: if an Elevation move makes a brand/positioning claim (e.g. "privacy-native"), verify that a Gap fix addresses any behavior that contradicts that claim — brand before behavior is a credibility trap.
- [ ] **Goal stated in the header** — the output's "Optimizing for:" line matches what the user confirmed in the gate

If a gap or lens finding is excluded, note why in a brief `## What we set aside` section at the end — this prevents the coverage check from silently passing by omission.

---

## Phase 4 — Diff + save snapshot + offer to action

**If a previous snapshot was loaded in Phase 1 Source D**, produce a `## What changed since [DATE]` section (see `references/memory.md` for the exact format). This is the most valuable part of a repeat run — skip it only if there is genuinely no previous snapshot.

Save to `~/.claude/skills/whats-next-workspace/snapshots/<slug>-<YYYY-MM-DD>.md`. See `references/memory.md`.

**Output order matters.** The ranked recommendations lead. The numbered "At a glance" table is the **last** section before the snapshot footer — it's the scannable index, and it comes after the detail, not before it. See the template.

Offer to action the top pick immediately — don't wait to be asked:
> "Want me to take #1 further? I can: write a spec, find the relevant code and estimate real effort, draft a validation question for your users, or run a competitor deep-dive."

---

## Output template

```
# whats-next: [Project Name]
> Mode: Skill-as-project (skill-mode.md loaded)   ← include only when in skill-mode; omit for standard repos

**Optimizing for:** [confirmed goal]
**Signal sources:** [repo data, N HN mentions, competitor scan, previous run DATE / first run]
**Signal quality:** High / Medium / Low

---

## Recommendations — best first
*Ranked by impact toward [goal] ÷ effort.*

### 1. [Name] · [Gap / Build / Elevation] · Impact ★★★★★ · Effort M
**Why it ranks #1 (toward [goal]):** [the sentence that earns the top spot]
**Source:** [Persona X gap] + [lens finding]
**Mechanism:** [concretely what to build/change]
**What would change my mind:** [the one assumption that, if false, drops it]
**Sequencing:** [if it depends on another item — else omit]

### 2. [Name] · [Build] · Impact ★★★★☆ · Effort S
**Why it ranks here (toward [goal]):** ...
**Source:** ...
**Mechanism:** ...
**What would change my mind:** ...

### 3. … [continue — comprehensive, every recommendation, ranked best-first]

[Elevation items use the same header; their "What would change my mind" line IS the falsifiable bet, plus competitor context and a Requires-to-measure note if needed.]

---

## Don't build yet
- [Thing] — [specific backfire mechanism]

## What we set aside
- [Gap or lens finding] — [why it didn't make the list]

---

## Personas behind these
*The analytical basis for the ranking above — who these recommendations serve.*
[5 persona cards: Evidence / Their context / Gap they hit / What they'd demand next]

---

## What changed since [DATE]
*(Repeat runs only — omit on first run.)*
**Top recommendations:** [moved up/down, newly added, dropped since last run]
**Still unaddressed from last run:** [items recommended before but not yet built — the most important line]
**Signal changes:** [new external mentions, issues, competitor moves]
**Personas:** [new / dropped / confidence changes]

---

## At a glance
*The scannable index — same order as the recommendations above. Detail is in the sections above.*

| # | What | Type | Impact | Effort | Description |
|---|------|------|--------|--------|-------------|
| 1 | [short name] | Gap / Build / Elev | High / Med / Low | S / M / L | [5–8 word why — the problem it solves] |
| 2 | … | | | | |

Numbered best-first, identical order to the recommendations. Impact = impact toward the goal. Effort = S (days) / M (weeks) / L (months).

---
*Snapshot saved. Run again to see what changed.*
```

**Nothing appears after the "At a glance" table except the snapshot footer.** If you find yourself writing an "Elevation moves" block below the table, stop — those items belong *in* the ranked recommendations above, interleaved by rank. The table is the last content section, every run.

---

## Portfolio mode

- Prefer cross-tool personas; identify portfolio gaps and sequencing
- Apply the five lenses across the portfolio as a whole, not per project
- Elevation moves can span the portfolio: unified config, shared auth, meta-orchestration

---

## References

- `references/skill-mode.md` — **read first if input is a Claude Code skill**
- `references/external-signals.md` — HN/Reddit/web APIs, user signal injection
- `references/competitors.md` — finding alternatives, positioning, moats
- `references/actioning.md` — spec kickoff, code-finding, effort estimation, validation
- `references/memory.md` — snapshot format, diff logic, time-series comparison
