---
name: social-sentiment-analyzer
version: 2026-02-23
description: Analyze brand or topic sentiment across Twitter, Reddit, and Instagram using Xpoz. Classifies posts as positive/neutral/negative, extracts recurring themes, and generates a sentiment report. Use when asked for "sentiment analysis", "what are people saying about X", "brand sentiment", or "social media opinion on X".
---

# Social Sentiment Analyzer

## Overview

Analyze public sentiment for any brand, product, or topic across Twitter/X, Reddit, and Instagram. Fetches real posts, classifies sentiment, extracts themes, and produces a structured report.

## When to Use

Activate when the user asks:
- "What's the sentiment around [TOPIC]?"
- "Analyze sentiment for [BRAND] on Twitter"
- "What are people saying about [PRODUCT] on social media?"
- "Is the reaction to [EVENT] positive or negative?"
- "Social media opinion on [TOPIC]"

## Setup & Authentication

Before fetching data, verify Xpoz access is configured. Try these in order:

### Check 1: MCP Server (preferred for AI agents)

If you have MCP tool access, test if Xpoz is already authenticated:

```bash
mcporter call xpoz.checkAccessKeyStatus
```

- If `hasAccessKey: true` → **Xpoz is ready.** Skip to Step 1.
- If it fails or the server isn't configured → set it up:

**Add the server:**
```bash
mcporter config add xpoz https://mcp.xpoz.ai/mcp --auth oauth
```

**Authenticate (local machine with browser):**
```bash
mcporter config login xpoz
```
This opens the browser for Google sign-in. Tell the user:
> "A browser window should open — just sign in with your Google account and click Authorize."

**Authenticate (remote/headless server):**
If there's no browser, generate an OAuth URL and send it to the user:
```bash
mcporter config login xpoz
```
Copy the authorization URL from the output, send it to the user, and wait for them to paste back the authorization code.

**Verify:**
```bash
mcporter call xpoz.checkAccessKeyStatus
```

For **Claude Code** users, Xpoz can also be added to `~/.claude.json`:
```json
{
  "mcpServers": {
    "xpoz": {
      "url": "https://mcp.xpoz.ai/mcp",
      "transport": "http-stream"
    }
  }
}
```
Authentication is handled via OAuth on first use — no API keys needed in the config.

### Check 2: SDK (for coding tasks)

Install the SDK and set your API key:

**Python:**
```bash
pip install xpoz
export XPOZ_API_KEY=your-token-here  # Get at https://xpoz.ai/get-token
```
```python
from xpoz import XpozClient
client = XpozClient()  # auto-reads XPOZ_API_KEY
# Or pass directly: XpozClient("your-token-here")
```

**TypeScript:**
```bash
npm install xpoz
export XPOZ_API_KEY=your-token-here
```
```typescript
import { XpozClient } from "xpoz";
const client = new XpozClient(); // auto-reads XPOZ_API_KEY
await client.connect();
// Or pass directly: new XpozClient({ apiKey: "your-token-here" })
```

Get a free API key at [xpoz.ai/get-token](https://xpoz.ai/get-token?utm_source=github&utm_medium=agent-skills&utm_campaign=social-sentiment-analyzer) — 100K results/month free, no credit card required.

### Auth Errors
| Problem | Solution |
|---------|----------|
| MCP: "Unauthorized" | Run `mcporter config login xpoz --reset` |
| SDK: `AuthenticationError` | Verify API key at [xpoz.ai/settings](https://xpoz.ai/settings) |
| mcporter not found | Included with OpenClaw — ensure OpenClaw is installed |


## Step-by-Step Instructions

### Step 1: Parse the Request

Extract from the user's message:
- **Topic/brand** to analyze
- **Platforms** to search (default: Twitter + Reddit; add Instagram if relevant)
- **Time period** (default: last 7 days)
- **Language** filter (default: English)

Expand the query for better coverage:
- Publicly traded companies → include ticker symbol: `"Tesla" OR "$TSLA"`
- Products → include common abbreviations: `"ChatGPT" OR "GPT-4"`
- Events → include hashtags: `"CES 2026" OR "#CES2026"`

### Step 2: Fetch Posts

#### Via MCP (if xpoz MCP server is configured)

**Twitter:**
```
Call getTwitterPostsByKeywords:
  query: "<expanded query>"
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount", "impressionCount"]
  startDate: "<7 days ago, YYYY-MM-DD>"
  endDate: "<today, YYYY-MM-DD>"
  language: "en"
```

**Reddit:**
```
Call getRedditPostsByKeywords:
  query: "<expanded query>"
  fields: ["id", "title", "text", "authorUsername", "createdAtDate", "score", "numComments", "subreddit"]
  startDate: "<7 days ago>"
  endDate: "<today>"
```

**Instagram (if requested):**
```
Call getInstagramPostsByKeywords:
  query: "<expanded query>"
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "commentCount"]
  startDate: "<7 days ago>"
  endDate: "<today>"
```

**CRITICAL: Async Pattern** — Each call returns an `operationId`. You MUST call `checkOperationStatus` with that ID and poll until status is "completed" (up to 8 retries, ~5 seconds apart).

#### Via Python SDK

```python
from xpoz import XpozClient

client = XpozClient()  # Uses XPOZ_API_KEY env var

# Twitter
twitter_results = client.twitter.search_posts(
    '"Tesla" OR "$TSLA"',
    start_date="2026-02-16",
    end_date="2026-02-23",
    language="en",
    fields=["id", "text", "author_username", "created_at_date", "like_count", "retweet_count"]
)

# Reddit
reddit_results = client.reddit.search_posts(
    '"Tesla" OR "$TSLA"',
    start_date="2026-02-16",
    end_date="2026-02-23",
    fields=["id", "title", "text", "author_username", "created_at_date", "score", "num_comments", "subreddit"]
)

# Collect all posts
twitter_posts = twitter_results.data
reddit_posts = reddit_results.data

# Fetch additional pages if needed
while twitter_results.has_next_page():
    twitter_results = twitter_results.next_page()
    twitter_posts.extend(twitter_results.data)

client.close()
```

#### Via TypeScript SDK

```typescript
import { XpozClient } from "xpoz";

const client = new XpozClient();
await client.connect();

const twitterResults = await client.twitter.searchPosts('"Tesla" OR "$TSLA"', {
  startDate: "2026-02-16",
  endDate: "2026-02-23",
  language: "en",
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount"],
});

const redditResults = await client.reddit.searchPosts('"Tesla" OR "$TSLA"', {
  startDate: "2026-02-16",
  endDate: "2026-02-23",
  fields: ["id", "title", "text", "authorUsername", "createdAtDate", "score", "numComments", "subreddit"],
});

await client.close();
```

### Step 3: Classify Sentiment

For each post, classify into one of 5 levels:

| Level | Indicators |
|-------|-----------|
| **Positive** | "love", "amazing", "bullish", "great", "best", 🚀🔥💪, strong praise |
| **Leaning Positive** | "looking good", "solid", "promising", measured optimism |
| **Neutral** | Questions, factual statements, news without opinion, balanced takes |
| **Leaning Negative** | "worried", "not sure", "concerned", "some issues", cautious criticism |
| **Negative** | "terrible", "worst", "avoid", "bearish", 📉💀, strong criticism |

**Tips:**
- Sarcasm detection: "Great, another outage" → Negative
- Retweets/quotes with no commentary → Neutral
- Engagement-weighted: high-engagement posts carry more signal

### Step 4: Extract Themes

Identify 5-8 recurring themes from the posts. For each theme:
- **Title**: 3-5 word label
- **Sentiment**: overall lean of posts in this theme
- **Key quotes**: 2-3 representative posts
- **Volume**: approximate % of total posts

### Step 5: Generate Report

Present results in this structure:

```
## Sentiment Report: [TOPIC]
**Period:** [start] to [end] | **Posts analyzed:** [count]

### Overall Sentiment
Score: [0-100, where 50=neutral, 100=max positive]
- Positive: X%
- Neutral: X%
- Negative: X%

### Platform Breakdown
| Platform | Posts | Sentiment Score | Top Theme |
|----------|-------|----------------|-----------|
| Twitter  | X     | X              | ...       |
| Reddit   | X     | X              | ...       |

### Key Themes
1. **[Theme Title]** (Positive/Neutral/Negative)
   [2-3 sentence explanation with example quotes]

2. **[Theme Title]** ...

### Notable Posts
[Top 5 highest-engagement posts with text, author, and metrics]

### Summary
[2-3 paragraph executive summary with actionable insights]
```

## Example Prompts

- "Analyze sentiment around NVIDIA this week on Twitter and Reddit"
- "What's the social media reaction to the new iPhone?"
- "How are people feeling about Cursor IDE on Reddit?"
- "Sentiment analysis for Bitcoin in the last 30 days"

## Notes

- Free tier: 100,000 results/month at [xpoz.ai](https://xpoz.ai?utm_source=github&utm_medium=agent-skills&utm_campaign=social-sentiment-analyzer)
- For large datasets, use CSV export (`export_csv()` / `exportCsv()`) and analyze locally
- Reddit tends to have longer, more nuanced opinions; Twitter has higher volume but shorter takes
