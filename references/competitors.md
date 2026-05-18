# Competitor Landscape Analysis

Every elevation move needs competitor context — you can't identify a moat without knowing what you're competing against.

## Step 1: Find alternatives

Search for what users actually compare the project to:

```bash
# Web search queries to run
"<project-name> alternative"
"<project-name> vs"
"best <category> tools"
"open source <category>"
```

Also look in HN and Reddit results from external signal collection — comparison threads are gold. Users who say "I switched from X to Y because..." are giving you the full competitive map.

For GitHub projects, check:
- The README "alternatives" or "comparison" section if present
- Issues where users mention other tools
- "Topics" tags — other repos with the same tags are often competitors

## Step 2: Evaluate each competitor on these axes

For each significant competitor found:

| Axis | Question |
|------|----------|
| **Positioning** | Who do they say they're for? How do they describe themselves? |
| **User complaints** | What do their users complain about? (Check their issues, reviews) |
| **Strengths** | What do their users love? What keeps people from leaving? |
| **Distribution** | How do people find and install them? What's their growth vector? |
| **Structural advantages** | What can they do that the analyzed project can't, by design? |
| **Structural weaknesses** | What can't they do, by design? (Privacy-first vs. cloud; CLI vs. GUI) |

## Step 3: Find the moat opportunities

After mapping competitors, identify:

**Differentiating gaps** — things users of competitor tools complain about that the analyzed project already does better, or could do better with a focused move.

**Structural impossibilities** — things the analyzed project could do that competitors structurally can't (e.g., a privacy-first tool can offer local encryption in a way a cloud-first competitor can't without rebuilding their architecture).

**Underserved segments** — user types who appear in competitor reviews but aren't well-served by any existing tool.

## Step 4: Apply to Layer 3

Every elevation move must pass this check:
- Does it play to a structural advantage (not just being better at the same thing)?
- Would the category leader have to abandon their core positioning to copy it?
- Is there a user segment who currently uses a competitor but would switch for this?

If yes to any of these, it's a real elevation move. If not, it's Layer 2.

## Example competitor analysis format

```
### Competitors found: [Tool A], [Tool B], [Tool C]

**Tool A** (closest alternative)
- Positioning: [their tagline / README framing]
- Their users complain about: [top issues / review themes]
- Structural weakness: [what they can't do by design]
- Users who'd switch: [persona description]

**Gap the analyzed project could own:**
[Specific differentiator + why competitor can't easily copy it]
```

## When no competitors exist

If the project is genuinely novel (no direct alternatives found), note this explicitly — it's both an opportunity and a risk. Analyze adjacent tools in related categories instead, and ask: what do users currently do instead of this tool? That workaround is the real competitor.
