# Skill-as-Project Mode

When the subject being analyzed is a Claude Code skill (not a software repo), the standard `gh` data collection fails — skills have no GitHub issues, no releases, no star counts unless explicitly published. This reference defines an alternate signal collection path.

## Detection — when to enter skill mode

Enter skill mode when any of these are true:
- Input is a path containing `SKILL.md` (e.g. `~/.claude/skills/foo/SKILL.md`, `./my-skill/`)
- Input is a skill name matching the available skills list (e.g. "the whats-next skill", "my fashion skill")
- Input describes "a Claude Code skill", "a Claude skill", "a skill I built"
- The project path exists locally and contains `SKILL.md` but no standard repo indicators (no `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc.)

When in doubt: check for SKILL.md at the path first, then decide.

## Signal collection in skill mode

Replace the four standard sources with these:

### Source A: Skill files (replaces repo data)
Read all skill files:
```bash
cat <skill-dir>/SKILL.md
ls <skill-dir>/references/ 2>/dev/null && cat <skill-dir>/references/*.md
ls <skill-dir>/evals/ 2>/dev/null && cat <skill-dir>/evals/evals.json
ls <skill-dir>/scripts/ 2>/dev/null
```

Extract from SKILL.md:
- **Description field** — who it's for, what it does, when to trigger
- **Trigger phrases** — what user intents it serves ("when the user asks X")
- **References used** — complexity and scope of the skill
- **Evals** — what test cases exist reveal what the author worried about
- **Scripts** — bundled scripts reveal what was too complex to inline

This is the equivalent of README + CHANGELOG for a skill.

**If local SKILL.md is absent but a GitHub repo exists:** fall back to the repo's README as the primary signal source. Extract the same fields from the README description and usage sections. Note this in the signal quality header: "SKILL.md not available locally — signals extracted from GitHub README."

### Source B: Ecosystem mentions (replaces external conversations)
Search for the skill by name and category:

```bash
# HN — skill name + "claude code skill" category
curl -s "https://hn.algolia.com/api/v1/search?query=<skill-name>+claude+skill&hitsPerPage=10"
curl -s "https://hn.algolia.com/api/v1/search?query=<skill-category>+claude+code&hitsPerPage=10"

# Reddit
curl -s "https://www.reddit.com/search.json?q=<skill-name>+claude&limit=10" \
  -H "User-Agent: whats-next/1.0"
```

Also search for community lists that might include this skill:
- awesome-claude-skills repositories
- Claude Code community forums/Discord threads

**If the skill has a GitHub repo:** run standard repo data collection on it too — some skills are published as standalone repos (e.g. googlarz/context-handoff).

### Source C: Category competitor skills (replaces competitor landscape)
Skills compete with other skills and with Claude's native capabilities. Find alternatives:

1. **Skills in the same category** — search the available skills list for skills with overlapping trigger phrases or descriptions. If analyzing a "code review" skill, all other code-review-adjacent skills are competitors.

2. **Installed MCPs** — if the system has MCP tools in the same domain (e.g., analyzing a health skill when a health-related MCP is also connected), those MCPs are direct capability competitors. Name them if they're visible in the system context.

3. **Claude's native capability** — the most important competitor for any skill is doing the task without a skill at all. What does Claude do by default when asked this question, with no skill loaded? That's the baseline the skill must beat.

4. **Other AI tools that do this** — web search for `"<category> AI tool"` to find non-Claude alternatives. A persona might prefer a dedicated SaaS over a Claude skill.

Evaluate each alternative on:
- What it does that this skill doesn't
- What this skill does that it can't
- Which user types prefer which approach

### Source D: Memory (same as standard mode)
Check `~/.claude/skills/whats-next-workspace/snapshots/` for previous analysis.

## Inferring users without analytics

Skills have no star counts, download metrics, or issue reporters. Infer user size and type from:

**Publication status:**
- Local only → users = the author + maybe close collaborators. Be explicit: "this skill has no known external users yet."
- Listed in awesome-claude-skills → wider distribution, unknown count
- Has its own GitHub repo → check stars, issues, forks
- Featured in a Show HN / blog post → audience depends on engagement (fetch and check)

**Skill category signals:**
- Developer tools → users are probably technical (engineers, indie devs)
- Productivity/scheduling → users span technical and non-technical
- Finance/health/lifestyle → deliberately broad, likely includes non-technical users
- Domain-specific (legal, medical, etc.) → practitioners in that domain

**Trigger phrase complexity:**
- Simple triggers ("when user asks about X") → broader user base, lower bar to discover
- Complex triggers ("when importing from CSV with specific format") → narrow power-user audience

**Eval coverage:**
- Evals present and varied → author thought carefully about edge cases → skill has been used in varied contexts
- No evals → likely early stage or single-author use

## Persona guidance for skills

Because skill users are Claude Code users (or Claude users), the technical floor is higher than a general software project. But the spectrum still applies:

- **Most technical:** skill author / plugin developer — might fork and extend it
- **Mid:** power Claude Code user — installs many skills, uses them daily, notices rough edges
- **Casual:** occasional Claude Code user — invokes skills when needed but doesn't maintain a curated stack
- **Non-technical floor:** Claude.ai users who encounter skill-like behavior through Cowork or shared projects

**When to include the non-technical persona:** Ask whether a non-developer who uses Claude through a browser (not Claude Code) could plausibly benefit from this skill's *domain* — not just the skill itself. If yes (e.g. health, finance, productivity), include them with the gap being "can't install skills directly." If the skill's domain is inherently technical (code review, CLI tools, shell scripts), the non-technical persona doesn't fit — replace it with a different archetype on the technical spectrum, such as: an internal team member who has Claude Code but avoids skill configuration, or a new Claude Code adopter who hasn't built skill literacy yet. Don't force a persona that the evidence doesn't support.

## Output adjustments in skill mode

In the signal quality header, note:
```
**Signal sources:** SKILL.md + references (N files), evals (N test cases), ecosystem search (N mentions found / not found), [GitHub repo if exists], no previous snapshot / previous snapshot from DATE
**Mode:** Skill-as-project
```

In the persona evidence field, cite skill-specific signals:
- "Evidence: description field targets X, trigger phrases assume Y, eval test cases reveal Z"
- "Evidence: skill has no external repo, inferred as single-author use"
- "Evidence: 3 HN comments mention this skill category positively"

In Layer 3 elevation moves, always include publication as a candidate:
- If unpublished: "Publish to GitHub + awesome-claude-skills" is almost always an elevation move — it changes the project from personal tool to community resource
- If published: the elevation move should be about distribution, integration, or category expansion

**Layer 3 test for skills:** "Publish" is the baseline elevation. For a second or third elevation move, ask: does this change what the skill *is*, not just what it *does*? Examples of genuine skill-level elevation: packaging as an MCP server (changes interface), defining an open data format (becomes a protocol), enabling proactive triggers (changes from reactive to ambient). Extending existing capabilities (even significantly) is Layer 2, not Layer 3.

**Gap vs. demand in skill mode:** The gap is what breaks or frustrates the persona *today* with the current skill. The demand is what they would ask to be built. These should differ in specificity level: gap = "skill tries gh commands on a local path and fails silently", demand = "skill-mode detection with confirmed Mode: Skill-as-project in header". If gap and demand are close paraphrases of each other, the gap isn't specific enough.
