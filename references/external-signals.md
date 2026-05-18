# External Signal Sources

Repo issues are the least honest signal — only users who care enough to file them. Search external sources for what people say when they're not talking to the maintainer.

## Hacker News (Algolia API — no auth required)

Search for project mentions:
```bash
# By project name
curl "https://hn.algolia.com/api/v1/search?query=<project-name>&tags=comment&hitsPerPage=30"

# By repo URL
curl "https://hn.algolia.com/api/v1/search?query=github.com%2F<owner>%2F<repo>&hitsPerPage=20"

# Show HN posts specifically
curl "https://hn.algolia.com/api/v1/search?query=<project-name>&tags=show_hn&hitsPerPage=10"
```

Extract: `hits[].comment_text` or `hits[].story_text`. Look for recurring themes, comparisons to alternatives, specific feature complaints. A comment that says "would use this if it had X" is more valuable than a filed issue requesting X.

## Reddit

```bash
curl "https://www.reddit.com/search.json?q=<project-name>&sort=relevance&limit=25" \
  -H "User-Agent: whats-next-skill/1.0"
```

Also search subreddits likely to discuss it:
- `/r/programming`, `/r/MachineLearning`, `/r/SideProject`, `/r/devops`
- Domain-specific subs (e.g., `/r/personalfinance` for a finance tool)

Extract: post titles, top-level comments, upvote patterns (high upvote = resonant pain).

## Web search

Use WebSearch or `mcp__plugin_context-mode_context-mode__ctx_fetch_and_index` for:
- `"<project-name>" review`
- `"<project-name>" alternative`
- `"<project-name>" vs`
- `site:dev.to OR site:medium.com "<project-name>"`

Blog posts and tutorials reveal what the author had to figure out the hard way — which is often a gap the project doesn't document well.

## What to extract

For each external source, note:
- **Recurring themes** — what do multiple independent people mention?
- **Comparisons** — what do they compare the project to? What does the alternative do better?
- **Abandonment signals** — "used to use X but switched to Y because..."
- **Adoption blockers** — "would use if..." patterns
- **Unexpected use cases** — people using the tool for something the README doesn't address

## Interpreting volume

External signal volume is a proxy for persona size:
- 1 GitHub issue → one vocal user
- 5 HN comments making the same point → likely dozens of silent users with the same pain
- 40 Reddit mentions → significant audience, weight this persona higher

When presenting personas, note the signal volume: "Evidence: 12 HN comments and a 300-upvote Reddit thread all asking for the same feature."

## User signal injection

If the user pastes raw conversations (support emails, Discord messages, app store reviews, user interview notes), these are **primary signal** — they outrank GitHub issues and external search.

**How to handle pasted content:**
1. Treat each message/review as a data point
2. Cluster by theme: what problem is each person actually describing?
3. Use direct quotes in persona evidence: `**Evidence:** "I love this but every time I come back after a week I have no idea where I left off" (support email, recurring theme in 4/10 emails reviewed)`
4. Note the source and volume: how many messages, what time period, what channel
5. Validate inferred personas against the pasted content — if a persona you'd normally include has no representation in the real user conversations, flag it as low-confidence

**What to ask the user if they have conversations:**
> "Do you have any support emails, Discord messages, app store reviews, or user feedback I can analyze? Real user voice will make this more accurate than inference from the repo alone."

Prompt this if signal quality from GitHub alone is Low or Medium.
