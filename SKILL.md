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
gh repo view <owner/repo> --json name,description,stargazerCount,forkCount,repositoryTopics,primaryLanguage
gh issue list --repo <owner/repo> --limit 50 --json title,labels,comments,state
gh pr list --repo <owner/repo> --limit 20 --json title,body,state
gh release list --repo <owner/repo> --limit 10
```
(The README field isn't reliable via `--json readme`; fetch it separately if needed: `gh repo view <owner/repo> | head -40` or read the raw file.) CHANGELOG bugs are especially valuable — a locale bug means real users in that locale; a 50MB import fix means someone hit it.

**Description-only input (no repo, not a skill):** Sources A–D assume a GitHub repo. When the input is a plain project description (a private/pre-launch app, an idea), you have no repo signal — don't pretend otherwise. Collect what you can: Source C (competitors — always available) and Source B (category conversations on HN/Reddit). Then lean on the pre-launch persona construction in Phase 2 and mark signal quality honestly. A description-only run is legitimately Low or Medium signal; say so.

**Source B — External conversations:** What people say outside the repo (HN, Reddit, web). See `references/external-signals.md`. These are the users who didn't care enough to file an issue — often the most representative.

**Source C — Competitor landscape:** What alternatives exist and how users compare them. Required before writing elevation moves. See `references/competitors.md`.

**Source D — Memory:** Check `~/.claude/skills/whats-next-workspace/snapshots/` for a previous run. If found, **read the full snapshot file** — it is the baseline for the diff. Note the date in the signal header: "previous run from DATE (N days ago)". See `references/memory.md` for diff format.

**Low-signal mode:** When signal quality is Low, do not just label and proceed. Execute the Low Signal Protocol:

1. **Label the quality** — `Signal quality: Low` in the signal header.
2. **Mark personas** — tag all inferred personas as `[Hypothesis]`.
3. **Generate a validation block** — place this section immediately **before** the "At a glance" table (the table stays strictly last):

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

Ask: **who is the least technical person who could plausibly benefit?** That persona belongs in the set. If that persona turns out to be "dead" — their only gap is unsolvable within the project (e.g. "can't install it without a developer" when the project can't change that) — don't silently backfill the slot with another technical persona and quietly raise the floor. Either reach down to the lowest persona who *does* have a solvable gap, or state plainly that the category has no viable non-technical user. A silently all-technical persona set on a tool that could serve normal people is a miss.

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

## Phase 2.6 — Domain coverage sweep (anti-omission)

The five lenses are *angles*; they can all fire and still miss an entire *domain* that no persona happened to raise. A finance tool with no Security item, an auth library with no threat-model item, a CLI with no install/distribution item — that's a silent hole, and silent holes are how "comprehensive" lies. The sweep is the difference between "what occurred to me" and "what I verified I didn't skip."

Before ranking, walk this checklist. For each domain, either surface a recommendation (it enters the Phase 3 ranking) or consciously decide it doesn't apply. This is fast — most domains resolve in one line.

| Domain | The question to ask |
|--------|---------------------|
| **Trust (perception)** | Does the user understand and believe where their data goes? Is the handling *disclosed*? |
| **Security (technical)** | Is data-at-rest, credentials, and key handling actually *protected*? (Distinct from trust — a tool can disclose clearly and still encrypt nothing.) |
| **Data & input** | Import/export breadth, format coverage, migration, silent data loss / mis-parse |
| **Reliability** | Failure modes, error recovery, what breaks under edge input or scale |
| **Performance** | Anything slow enough to make a user quit or distrust the result |
| **Accessibility / reach** | Who's locked out — by language, disability, platform, or skill level |
| **Distribution / onboarding** | How a brand-new user gets from zero to first value |
| **Monetization / sustainability** | If revenue or sustainability is the goal: is there *any* surface for it today? |
| **Compliance / legal** | Regulated domain (finance, health, kids, EU data)? Any obligation unmet? |

**Output discipline:** Domains that surface a real issue become ranked recommendations, tagged `(from: X domain)` like a lens. A domain that is **category-adjacent to the project's core function** (data-integrity for a tool that parses files; security for anything holding money/credentials/PII; distribution for a CLI) must be **explicitly dispositioned** — either a recommendation or a line in "What we set aside" with a reason. Only domains genuinely *orthogonal* to the project appear nowhere; don't clutter the output with "N/A" lines for those. The test: if a reader who knows the category would expect a domain to be addressed, silence on it is a defect — name it in "What we set aside."

Bias check: do not force a domain that doesn't fit just to fill the table, and do not skip one because no persona raised it. The whole point is to catch the domain no persona raised.

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
- **Sequencing** — required field. Write the dependency ("after #2 — needs the landing page first") or "none." Forcing the word "none" makes you actually check, instead of leaving a real dependency buried in prose.

### Elevation discipline (applies to Elevation-tagged items)
Include **at least 2** Elevation moves from different angles: one that changes **who** the project serves, one that changes **how** value is delivered (pull → push, sync → async, individual → ambient).

**Elevation moves are ranked inline like everything else** — they carry the `Elevation` tag and take their position by impact ÷ effort, never a separate trailing section. A big elevation bet that ranks low because it's high-effort and long-horizon is honest — it's a direction, not your next action.

Elevation items carry the **same fields as every other recommendation** (including Mechanism — what concretely to build), plus three more:
- **Competitor context required:** name what competitors do or don't do. "Build X in a way structurally impossible for competitor Y to copy" is elevation. "Build X, which Y already does" is not.
- **Falsifiable bet:** the "what would change my mind" line for an Elevation item must be a single make-or-break condition, wrong-able: "Works only if X," not "works if it's good enough."
- **Measurability check:** if the project can't currently measure that bet, append "Requires: [what to instrument]." A bet that can't be measured is unfalsifiable in practice, however specific it sounds.

### Don't build yet
Specific backfire mechanism per item — not prioritization. "Building Y would require solving Z first, and Z is a harder problem than Y appears" is a reason. "Not a priority" is not.

---

## Phase 3.5 — Coverage check (quality gate)

Before outputting, verify every box is checked. If any fails, fix it before proceeding:

- [ ] **#1 is genuinely the best move toward the goal** — not the most exciting or biggest. Does a cheaper item lower down serve the goal better? Then it's #1. The single most important check.
- [ ] **Ranked by goal-impact ÷ effort, and the ratings discriminate** — types interleaved (not grouped); if more than half the recs share the top ★ tier the ranking isn't sorting, so re-spread or let effort break ties. The At-a-glance table carries the same ★, not collapsed High/Med/Low.
- [ ] **Every rec carries all six fields — Elevations included** — type · impact · effort header, plus Source, Mechanism, "what would change my mind," and Sequencing. Elevations are *not* exempt from Source / Mechanism / Sequencing (the most common silent omission). Elevations add Competitor context + a falsifiable, measurable bet.
- [ ] **Nothing accounted for by silence** — every persona gap, all 5 lenses (per the lens-coverage line), and every domain category-adjacent to the project's core either drives a rec or is named in "What we set aside" with a reason.
- [ ] **No phantom citations** — every persona named in a Source line has a card in "Personas behind these"; every lens named actually appears.
- [ ] **No self-narration** — the output never references its own machinery. Banned: naming a reference file or line ("per skill-mode.md line 110"), "per the coverage check," "gate skipped," "the Adjacency lens is accounted for," a lens/split "firing," or naming the Trust-vs-Security split or domain sweep as a mechanic. State the *finding*, never that a rule produced it. Persona swaps are invisible — never "this replaces the dead persona." A reader shouldn't be able to tell the skill has phases.
- [ ] **Type minimums:** ≥1 Gap, ≥2 Builds with overlap traces, ≥1 Elevation that changes *who* or *how* — ranked inline, never a trailing section.
- [ ] **"Don't build yet" has ≥1 item** with a backfire mechanism (not a priority judgment).
- [ ] **Brand before behavior:** if an Elevation makes a positioning claim ("privacy-native"), a Gap fixes any current behavior that contradicts it — else the claim is a credibility trap.
- [ ] **Structure:** goal in the header matches the gate; recommendations lead; the At-a-glance table is last with nothing after it but the footer.

---

## Phase 4 — Diff + save snapshot + offer to action

**If a previous snapshot was loaded in Phase 1 Source D**, produce a `## What changed since [DATE]` section (see `references/memory.md` for the exact format). This is the most valuable part of a repeat run — skip it only if there is genuinely no previous snapshot.

Save to `~/.claude/skills/whats-next-workspace/snapshots/<slug>-<YYYY-MM-DD>.md`. See `references/memory.md`.

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
**Source:** [Persona X gap] + [lens/domain finding]
**Mechanism:** [concretely what to build/change]
**What would change my mind:** [the one assumption that, if false, drops it]
**Sequencing:** [terse — "after #3" or "none". Don't restate the rationale.]

### 2. [Name] · [Build] · Impact ★★★★☆ · Effort S
**Why it ranks here (toward [goal]):** ...
**Source:** ...
**Mechanism:** ...
**What would change my mind:** ...
**Sequencing:** none

### 3. [Name] · [Elevation] · Impact ★★★☆☆ · Effort L
**Why it ranks here (toward [goal]):** ...
**Source:** [persona/lens/domain — REQUIRED, same as any rec]
**Mechanism:** [concretely what to build — REQUIRED, elevations are not exempt]
**Competitor context:** [what rivals do/don't do that makes this structurally yours]
**What would change my mind:** [the falsifiable bet — "works only if X." Add "Requires: [what to instrument]" if unmeasurable today.]
**Sequencing:** [terse]

### … [continue — comprehensive, every recommendation, ranked best-first. Every rec — Gap, Build, AND Elevation — carries all six fields: Source, Mechanism, Sequencing are never dropped.]

---

## Don't build yet
- [Thing] — [specific backfire mechanism]

## What we set aside
- [Gap or lens finding] — [why it didn't make the list]

*Lens coverage: Depth → [#N or set-aside] · Breadth → [#N] · Promise gap → [#N] · Adjacency → [#N] · Category leadership → [#N].* ← one terse line; every lens maps to a rec number or "set-aside." This makes a dropped lens impossible to miss.

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
| 1 | [short name] | Gap / Build / Elev | ★★★★★ | S / M / L | [5–8 word why — the problem it solves] |
| 2 | … | | ★★★★☆ | | |

Numbered best-first, identical order to the recommendations. **Impact column carries the same ★ rating as the recommendation** — not a collapsed High/Med/Low, which buckets 4★ and 5★ together and makes the index stop sorting. Effort = S (days) / M (weeks) / L (months).

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
