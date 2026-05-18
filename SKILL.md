---
name: whats-next
description: Analyze one or more projects by identifying the top 5 user personas across the full spectrum from technical to non-technical, surfacing gaps each persona hits, and synthesizing what to build next — including breakthrough moves that elevate the project to a new level. Use this skill whenever the user asks "what should I build next", "what do my users want", "what's missing from this project", "who uses this and what do they need", or wants strategic product direction for a repo or portfolio. Trigger even if the user just says "analyze my repos" or "what should I add to X" — persona-based product thinking is almost always the right lens. Works with GitHub URLs, repo names, local project paths, plain project descriptions, or pasted user conversations (emails, Discord, reviews, support tickets).
---

# whats-next

Strategic product direction through persona-based analysis. You identify who actually uses a project — spanning the full range from power users to casual non-technical users — give each persona a voice, surface the gaps they hit, run five product lenses to catch what persona analysis misses, then verify the recommendations are complete before outputting.

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

**Low-signal mode:** Mark gaps as near-certain vs. hypothesis. Ask the user for support emails or feedback if signal quality is Low.

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

**Anti-overlap test:** Before finalizing, ask: "If I merged Persona A and Persona B into one card, would any layer item disappear?" If no — merge them and build a genuinely distinct persona instead. Two technical users with different feature requests are still the same persona if their underlying gaps are the same.

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

**Evidence requirement:** If a persona has no signal support (no issue, no commit message, no HN comment, no release title pointing to this user type), mark it explicitly as `[Hypothesis]` and lower its weight in layer prioritization.

**Pre-launch persona construction:** For WIP / unreleased apps with no user data, the non-technical and evaluator personas cannot be grounded in usage evidence. Instead, ground them in **launch audience characteristics**: What audience is the Product Hunt / launch channel submission targeting? Who shares this type of app on Twitter/Reddit? What are the top App Store reviews for the nearest comparable? Use these as evidence sources when in-app signal is unavailable.

---

## Phase 2.5 — Five product lenses (pre-synthesis)

Before writing the layers, run these five questions. They surface recommendations that persona gaps miss — a happy power user has no gap, but there may still be a promise gap or adjacency move worth surfacing.

Work through each fully — do not skip a lens. Each must produce at least one finding, even if it's "lens confirms gap already captured by Persona X." If a lens produces nothing, state why explicitly (it becomes "What we set aside" material).

1. **Depth** — What would make existing satisfied users use this *more*? What keeps them from going deeper?

2. **Breadth** — What's blocking people who've heard of this but haven't adopted? What would bring in the next 10x of users? **For pre-launch / WIP projects:** This lens almost always surfaces distribution blockers (no App Store listing, no landing page, no demo video) that persona analysis misses because personas assume users already found the product.

3. **Promise gap** — What does the README/description/tagline promise that the project doesn't fully deliver yet? Read the first sentence of the README literally — does the product actually do that?

4. **Adjacency** — What do users do immediately *before* or *after* this tool? Could this tool absorb those steps and become stickier?

5. **Category leadership** — What's the single thing keeping this from being the obvious, default choice in its category? What does the category leader have that this doesn't?

Lens findings feed into Layers 1, 2, and 3 alongside persona gaps. **All 5 lens labels must appear somewhere in the output** — either in a layer item or in "What we set aside." If a lens label is missing from the output, the coverage check fails.

**This working section is internal — do not reproduce it in the output.** Only the findings appear, labeled in the layer items where they land.

---

## Phase 3 — Three-layer output

Synthesize persona gaps + lens findings into three layers:

### Layer 1 — Close the gaps
Blockers and friction points **actively costing users today** — not underexposed features, not missing features. These are things a current user hits and loses trust or stops. Ask: "Would removing this make a user who already found the product stay?" If yes, it's Layer 1. If it requires a new user to show up first, it's Layer 2.

### Layer 2 — Build next
Ranked by genuine overlap across personas AND lenses. **Show your work:** trace each item to its source (persona name + gap, or lens name + finding). Only count a persona if their specific gap implies demand for this specific feature.

**Explain the ranking:** After each item's ★ rating, note in one phrase what drove it: "3 personas + 2 lenses" or "1 persona, but the only feature blocking Product Hunt launch." A rating without a basis is unverifiable.

### Layer 3 — Elevation moves
2–3 bets that change what the project *is*. Aim for moves from different angles:
- One that changes **who** the project primarily serves
- One that changes **how** value is delivered (pull → push, sync → async, individual → ambient)

**Elevation test:** Would you describe the project differently after this move? If not, it's Layer 2.

**Competitor context required:** Name what competitors do or don't do. "Build X in a way structurally impossible for competitor Y to copy" is an elevation move. "Build X, which competitor Y already does" is not.

**Falsifiable bet:** One make-or-break condition. Not a list — one thing. Should be wrong-able: "This works only if X" not "This works if it's good enough."

**Measurability check:** Before writing the bet, ask: does this project currently have the instrumentation to measure the condition? If not, append: "Requires: [what needs to be tracked before this bet can be evaluated]." A bet that can't be measured is unfalsifiable in practice, even if it sounds specific.

### Don't build yet
Specific backfire mechanism per item — not prioritization. "Building Y would require solving Z first, and Z is a harder problem than Y appears" is a reason. "Not a priority" is not.

---

## Phase 3.5 — Coverage check (quality gate)

Before outputting, verify every box is checked. If any fails, fix it before proceeding:

- [ ] **Every persona's gap** appears in at least one layer, OR is explicitly in "Don't build yet" with a reason
- [ ] **All 5 lens labels appear** in the output — either in a layer item or in "What we set aside." Count them: Depth, Breadth, Promise gap, Adjacency, Category leadership. Missing label = fix before proceeding.
- [ ] **Every lens finding** that isn't already covered by a persona gap appears somewhere, OR is explicitly set aside with a reason
- [ ] **Layer 1 exists** — at least one fix (no project has zero friction points today)
- [ ] **Layer 2 has ≥2 items** with verified overlap traces
- [ ] **Layer 3 has at least one move** that changes who the project primarily serves, not just what it does
- [ ] **"Don't build yet" has ≥1 item** with a backfire mechanism (no project has zero tempting-but-wrong moves)
- [ ] **Effort signal exists** somewhere — even rough (small / medium / large) on the top Layer 2 item
- [ ] **Sequencing noted** if any Layer 1 fix must happen before a Layer 2 item is reachable, OR if any Layer 2 item must happen before a Layer 3 move is viable. In particular: if a Layer 3 move makes a brand/positioning claim (e.g. "privacy-native"), verify that Layer 1 fixes any behavior that contradicts that claim — brand before behavior is a credibility trap.
- [ ] **Effort differentials explained** — if two Layer 2 items have similar overlap scores but different effort levels, the lower-effort item should be recommended first, or the reasoning for the different ordering must be explicit

If a gap or lens finding is excluded, note why in a brief `## What we set aside` section at the end — this prevents the coverage check from silently passing by omission.

---

## Phase 4 — Diff + save snapshot + offer to action

**If a previous snapshot was loaded in Phase 1 Source D**, produce a `## What changed since [DATE]` section *before* saving (see `references/memory.md` for the exact format). This is the most valuable part of a repeat run — skip it only if there is genuinely no previous snapshot.

Save to `~/.claude/skills/whats-next-workspace/snapshots/<slug>-<YYYY-MM-DD>.md`. See `references/memory.md`.

Offer to action the top pick immediately — don't wait to be asked:
> "Which of these do you want to pursue? I can: write a spec, find the relevant code and estimate effort, draft a validation question for your users, or run a competitor deep-dive."

---

## Output template

```
# whats-next: [Project Name]
> Mode: Skill-as-project (skill-mode.md loaded)   ← include only when in skill-mode; omit for standard repos

**Signal sources:** [repo data, N HN mentions, competitor scan, previous run DATE / first run]
**Signal quality:** High / Medium / Low
**Mode:** Standard / Skill-as-project

---

## The 5 Personas
[5 persona cards]

---

## Layer 1 — Close the Gaps
### 1. [Fix] ★★★★★
**Who's blocked:** [personas]
**Effort:** small / medium / large
**Mechanism:** ...

## Layer 2 — Build Next
### 1. [Feature] ★★★★★
**Source:** [Persona X gap] + [lens finding]
**Effort:** small / medium / large
**Why it wins:** ...

## Layer 3 — Elevation Moves
### [Move name]
**What it changes:** ...
**Competitor context:** ...
**The bet:** [single falsifiable condition]

## Don't build yet
- [Thing] — [specific backfire mechanism]

## What we set aside
- [Gap or lens finding] — [why it didn't make the layers]

## What changed since [DATE]
**Personas:** [new / dropped / confidence changes]
**Top gaps:** [appeared / disappeared / changed priority]
**Recommendations:** [moved up/down in layers, newly added]
**Signal changes:** [new external mentions, issues, competitor moves]
**Still unaddressed from last run:** [Layer 1/2 items from previous run that weren't built]

*(Omit this section on first run. Include on every subsequent run.)*

---
*Snapshot saved. Run again to see what changed.*
```

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
