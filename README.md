# unzuck

Take back your feeds. Scans 7 platforms, kills the noise, keeps the signal.

**unzuck** is a Claude Code plugin that scans your social media feeds, scores content against your interests, deep-dives on the best items, and generates a beautiful interactive HTML dashboard — your daily content digest, on autopilot.

![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-7C3AED)

<img width="1512" height="782" alt="Screenshot 2026-03-25 at 00 36 01" src="https://github.com/user-attachments/assets/3d1ea8f7-58ad-4dd9-8015-9c9b3ddaa16c" />

## What it does

1. **Scans 7 platforms in parallel** — YouTube, Twitter/X, Reddit, Hacker News, LinkedIn, Facebook, Instagram
2. **Scores every item** against your personal interest profile (weighted topics, negative topics, engagement signals)
3. **Deep-dives on the top items** — fetches full articles, transcripts, threads, and synthesizes detailed summaries that teach you the core insight without clicking through
4. **Generates an interactive dashboard** — filterable by platform, tags, and score, with dark/light theme, search, and embedded media
5. **Evolves your profile over time** — detects emerging interests, flags stale topics, suggests new subreddits

## Install

```bash
/plugin marketplace add donttalkaboutit/unzuck
/plugin install unzuck@unzuck
```

## Usage

```
/unzuck
```

Or say any of these to Claude:

- "unzuck my feeds"
- "curate my feeds"
- "what's new on my feeds"
- "daily digest"
- "scan my feeds"

### First run

On your first run, unzuck walks you through a ~2 minute onboarding:

1. **Choose where to save digests** (e.g., `~/Documents/content-digests/`)
2. **Pick which platforms to scan** and how many items per platform
3. **Tell it your interests** — or let it bootstrap by scanning your feeds first
4. **Review your profile** and suppress topics you never want to see

After setup, every subsequent run is fully autonomous — no confirmations, no interruptions, just a finished dashboard.

### Bootstrap mode

Don't want to list interests manually? Say "bootstrap from my feeds" — unzuck will scan your YouTube topic chips, Twitter timeline, Reddit subscriptions, and more to infer your interest profile automatically.

## Requirements

- [Claude Code](https://claude.ai/code) with the [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome/) extension
- Logged into the platforms you want scanned (in your Chrome browser)
- Hacker News and Reddit work without login

## How it works

### Architecture

```
/unzuck (SKILL.md)
    │
    ├── Step 0: Load/bootstrap interest profile
    ├── Step 1: Pre-flight — check which platforms are logged in
    ├── Step 2: Parallel scan — one platform-scanner agent per platform
    ├── Step 2.5: Dedup (cross-platform + cross-session via history.json)
    ├── Step 3: Score all items (0-100) against interest profile
    ├── Step 4: Parallel deep dives — one deep-dive-agent per top item
    ├── Step 5: Generate HTML dashboard from template
    ├── Step 5.5: Profile refinement — suggest emerging/stale topics
    └── Step 6: Save history, present results
```

### Agents

| Agent | Purpose |
|-------|---------|
| `platform-scanner` | Navigates a single platform via Chrome, scrolls, extracts items with rich descriptions and engagement data |
| `deep-dive-agent` | Fetches full content (articles, transcripts, threads), enriches from supplementary sources, produces teach-me-level summaries |

### Scoring

Items are scored 0-100 based on:

- **Topic relevance** — weighted match against your interest profile
- **Recency** — +5 for content from the last 24 hours
- **Engagement** — +3 for viral content (300+ HN points, 100K+ YouTube views, etc.)
- **Content quality** — +5 for rich, detailed descriptions
- **Negative topics** — configurable penalty (default -30) for topics you want suppressed
- **History** — -20 for items you've already seen in previous runs

### Dashboard

The generated HTML dashboard includes:

- Platform filters and tag cloud
- Score slider to set minimum threshold
- Search across titles and descriptions
- Dark/light theme toggle
- Embedded YouTube players and Instagram embeds
- Expandable deep-dive sections with key takeaways and timestamped insights
- Keyboard navigation (arrow keys, enter to expand, `b` to bookmark)

### Interest profile

Your profile lives in `interests.json` and includes:

```json
{
  "topics": [
    { "name": "AI/ML", "weight": 0.9, "keywords": ["LLM", "transformer"] }
  ],
  "negativeTopics": [
    { "name": "Crypto trading", "keywords": ["altcoin"], "penalty": -30 }
  ],
  "platformOverrides": {
    "hackernews": { "maxItems": 25 },
    "youtube": { "maxItems": 10 },
    "facebook": { "maxItems": 0 }
  }
}
```

- **Weights**: 0.8-1.0 = core interest, 0.5-0.7 = solid, 0.2-0.4 = casual
- **Negative topics**: push matching items down or off the dashboard
- **Platform overrides**: control item count per platform (0 = skip)

The profile evolves automatically — unzuck tracks topic match rates across runs and suggests additions, weight adjustments, and new subreddits.

## Supported platforms

| Platform | Login required | Default items |
|----------|---------------|---------------|
| Hacker News | No | 25 |
| Reddit | No (public) / Yes (personalized) | 15 |
| YouTube | Yes (Google) | 10 |
| Twitter/X | Yes | 10 |
| LinkedIn | Yes | 5 |
| Facebook | Yes + opt-in | Off |
| Instagram | Yes + opt-in | Off |

## Files generated

| File | Purpose |
|------|---------|
| `interests.json` | Your interest profile (persistent) |
| `history.json` | Seen URLs for cross-session dedup (persistent) |
| `items-scored.json` | Scored items for current run (overwritten) |
| `content-digest-YYYY-MM-DD.html` | Daily dashboard output |

## License

MIT
