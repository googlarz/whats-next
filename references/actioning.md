# Actioning — From Analysis to Execution

After presenting the whats-next output, always make a concrete offer. Don't end with "here's what you should build" — end with a specific next step the user can take in the next 5 minutes.

## The offer

Present this after the analysis:

> "Which of these do you want to pursue? I can:
> 1. **Write a spec** — requirements, acceptance criteria, edge cases
> 2. **Find the code** — locate relevant files, estimate implementation effort
> 3. **Draft a validation prompt** — a question to post to your users to test whether this gap is real before building
> 4. **Run a competitor deep-dive** — understand whether a competitor already solves this and how
> 5. **Start building** — kick off the implementation directly"

Make the offer specific to the top recommendation. Don't list all 5 options generically — tailor it:
> "For 'session delta digest': I can write the spec (the DB query is against existing tables, so effort is probably low), or draft a Discord message asking your users if they'd find this useful."

## What each action does

### Write a spec
Produce a concise spec document covering:
- Problem statement (the gap, in user terms)
- Proposed solution (what to build, at design level)
- Acceptance criteria (how you'll know it's done)
- Out of scope (what this doesn't cover)
- Open questions (what needs a decision before building)

Keep it under 1 page. The goal is alignment, not documentation.

### Find the code
Use code navigation tools to:
- Locate the files most likely to be modified
- Identify the data structures involved
- Find existing patterns to follow
- Estimate lines-of-code delta (rough: <50, 50-200, 200+)
- Flag any risky dependencies or potential regressions

Output: "This touches `X.py` (main logic), `Y.py` (DB schema), and `Z.md` (docs). Estimated effort: small/medium/large."

### Draft a validation prompt
A short, specific question to post to your actual users to test whether the identified gap is real before spending time building.

Good validation prompts are:
- Specific enough to get a yes/no answer (not "what do you think of X?")
- Framed around the user's pain, not your proposed solution ("do you find yourself re-orienting each time you open this?" not "would you use a delta digest feature?")
- Short enough to post in Discord / tweet / email without editing

Output: ready-to-post text + recommended channel (Discord, GitHub Discussions, Twitter, email list).

### Competitor deep-dive
Pick the top competitor and produce a focused analysis:
- What their users love (from their reviews / issues)
- What their users hate (same source)
- How their positioning overlaps with the analyzed project
- Whether the top recommendation is already solved by the competitor
- If yes: what would differentiation look like?

### Start building
If the user picks this, hand off to the appropriate skill:
- `/agent-skills:spec-driven-development` for spec-first approach
- `/agent-skills:incremental-implementation` for slice-by-slice build
- `/agent-skills:test-driven-development` if the feature needs test coverage first

Brief the next skill with: the gap being addressed, the persona who hits it, the proposed solution, and any constraints identified during analysis.

## Tone

The offer should feel like a colleague asking "what do you want to do next?" not a menu of services. Read the conversation — if the user seems ready to build, lead with "Start building." If they seem uncertain the gap is real, lead with "Draft a validation prompt."
