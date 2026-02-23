---
name: competitive-intel
version: 2026-02-23
description: Compare brands and products across social media — share of voice, sentiment, positioning, and audience overlap using Xpoz. Use when asked to "compare brands", "competitive analysis", "share of voice", "brand vs brand", or "competitive intelligence".
---

# Competitive Intelligence

## Overview

Compare multiple brands or products side by side across Twitter/X, Reddit, and Instagram. Measure share of voice, compare sentiment, identify positioning differences, and discover competitive advantages from real social conversations.

## When to Use

Activate when the user asks:
- "Compare [BRAND A] vs [BRAND B] on social media"
- "Share of voice: [BRAND] vs competitors"
- "Competitive analysis for [PRODUCT]"
- "How does [BRAND A] sentiment compare to [BRAND B]?"
- "What are people saying about [BRAND] vs [COMPETITOR]?"

## Setup & Authentication

Before fetching data, verify Xpoz access is configured. Try them in order:

### Check 1: MCP Server (preferred)
If you have MCP tool access, check if `xpoz` tools are available (e.g., `getTwitterPostsByKeywords`). If yes, authentication is already handled — skip to Step 1.

### Check 2: SDK via Environment Variable
```bash
echo $XPOZ_API_KEY
```
If set, the SDK will authenticate automatically:
```python
from xpoz import XpozClient
client = XpozClient()  # auto-reads XPOZ_API_KEY
```

### Check 3: Nothing Configured
If neither MCP nor env var is available, ask the user to:
1. Get a free API key at [xpoz.ai/get-token](https://xpoz.ai/get-token?utm_source=github&utm_medium=agent-skills&utm_campaign=competitive-intel) (100K results/month free)
2. Then EITHER:
   - **Set env var:** `export XPOZ_API_KEY=<their-key>` and use the SDK
   - **Configure MCP:** Add to `~/.claude.json` (or agent's MCP config):
     ```json
     {
       "mcpServers": {
         "xpoz": {
           "url": "https://mcp.xpoz.ai/mcp",
           "transport": "http-stream",
           "headers": { "Authorization": "Bearer <their-key>" }
         }
       }
     }
     ```
   - **Pass directly:** `XpozClient("<their-key>")` in Python or `new XpozClient({ apiKey: "<their-key>" })` in TypeScript

### Auth Errors
If you get `AuthenticationError` or 401 responses:
- Verify the API key is valid at [xpoz.ai/settings](https://xpoz.ai/settings)
- Check the key hasn't expired
- Ensure no extra whitespace in the key


## Step-by-Step Instructions

### Step 1: Parse the Request

Extract:
- **Primary brand** and **competitors** (2-5 brands total)
- **Platforms** (default: Twitter + Reddit)
- **Time period** (default: last 7 days)
- **Industry context** for better analysis

Build expanded queries for each brand:
- `"Slack"` → `"Slack" NOT "cut some slack" NOT "slack off"`
- `"Discord"` → `"Discord" NOT "sow discord" NOT "discord between"`
- For stocks: include ticker symbols

### Step 2: Fetch Data for Each Brand

Run parallel searches — one per brand, per platform.

#### Via MCP

For each brand, call:

**Twitter posts:**
```
Call getTwitterPostsByKeywords:
  query: "<brand query>"
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount", "impressionCount"]
  startDate: "<7 days ago>"
  endDate: "<today>"
  language: "en"
```

**Twitter users discussing the brand:**
```
Call getTwitterUsersByKeywords:
  query: "<brand query>"
  fields: ["id", "username", "name", "followersCount", "relevantTweetsCount", "relevantTweetsLikesSum"]
  startDate: "<7 days ago>"
```

**Reddit (for each brand):**
```
Call getRedditPostsByKeywords:
  query: "<brand query>"
  fields: ["id", "title", "text", "score", "numComments", "subreddit", "createdAtDate"]
  startDate: "<7 days ago>"
```

**CRITICAL:** Each call returns an `operationId` — poll `checkOperationStatus` until "completed".

**Tip:** Launch all brand searches in sequence, collect all operationIds, then poll them. This is faster than waiting for each one.

#### Via Python SDK

```python
from xpoz import XpozClient

client = XpozClient()

brands = {
    "Slack": '"Slack" NOT "cut some slack"',
    "Discord": '"Discord" NOT "sow discord"',
    "Teams": '"Microsoft Teams" OR "MS Teams"',
}

brand_data = {}

for brand_name, query in brands.items():
    # Twitter posts
    twitter = client.twitter.search_posts(
        query,
        start_date="2026-02-16",
        end_date="2026-02-23",
        language="en",
        fields=["id", "text", "author_username", "like_count", "retweet_count", "impression_count", "created_at_date"]
    )

    # Twitter users (for influencer overlap analysis)
    users = client.twitter.get_users_by_keywords(
        query,
        start_date="2026-02-16",
        fields=["username", "followers_count", "relevant_tweets_count", "relevant_tweets_likes_sum"]
    )

    # Reddit posts
    reddit = client.reddit.search_posts(
        query,
        start_date="2026-02-16",
        fields=["id", "title", "text", "score", "num_comments", "subreddit", "created_at_date"]
    )

    brand_data[brand_name] = {
        "twitter_posts": twitter,
        "twitter_users": users,
        "reddit_posts": reddit,
        "tweet_count": twitter.pagination.total_rows,
        "reddit_count": reddit.pagination.total_rows,
    }

client.close()
```

#### Via TypeScript SDK

```typescript
import { XpozClient } from "xpoz";

const client = new XpozClient();
await client.connect();

const brands: Record<string, string> = {
  Slack: '"Slack" NOT "cut some slack"',
  Discord: '"Discord" NOT "sow discord"',
  Teams: '"Microsoft Teams" OR "MS Teams"',
};

const brandData: Record<string, any> = {};

for (const [name, query] of Object.entries(brands)) {
  const twitter = await client.twitter.searchPosts(query, {
    startDate: "2026-02-16",
    endDate: "2026-02-23",
    language: "en",
    fields: ["id", "text", "authorUsername", "likeCount", "retweetCount", "createdAtDate"],
  });

  const reddit = await client.reddit.searchPosts(query, {
    startDate: "2026-02-16",
    fields: ["id", "title", "text", "score", "numComments", "subreddit"],
  });

  brandData[name] = { twitter, reddit };
}

await client.close();
```

### Step 3: Analyze and Compare

**Share of Voice (SOV):**
```
SOV for Brand A = (Brand A mentions) / (Total mentions across all brands) × 100
```
Calculate separately for Twitter and Reddit.

**Sentiment Comparison:**
For each brand, classify posts into positive/neutral/negative (see social-sentiment-analyzer skill for classification method) and compare:
- Overall sentiment score (0-100)
- Positive/negative ratio
- Sentiment trend over the time period

**Engagement Comparison:**
- Average likes per post
- Average comments/replies per post
- Total impressions (Twitter)
- Total Reddit score

**Audience Overlap:**
- Find users who posted about multiple brands (common usernames across datasets)
- These users are particularly valuable for understanding switching behavior

**Positioning Analysis:**
- What attributes does each brand's audience associate with it?
- What are the unique strengths/weaknesses mentioned for each?
- Common comparison contexts ("I switched from X to Y because...")

### Step 4: Generate Report

```
## Competitive Intelligence: [BRAND] vs Competitors
**Period:** [date range] | **Platforms:** Twitter, Reddit

### Share of Voice
| Brand | Twitter Posts | Reddit Posts | Total | SOV |
|-------|-------------|-------------|-------|-----|
| Slack | 1,234 | 456 | 1,690 | 42% |
| Discord | 890 | 678 | 1,568 | 39% |
| Teams | 456 | 321 | 777 | 19% |

### Sentiment Comparison
| Brand | Score | Positive | Neutral | Negative | Trend |
|-------|-------|----------|---------|----------|-------|
| Slack | 62 | 38% | 42% | 20% | → Stable |
| Discord | 71 | 48% | 35% | 17% | ↑ Improving |
| Teams | 45 | 22% | 45% | 33% | ↓ Declining |

### Engagement Comparison
| Brand | Avg Likes (Twitter) | Avg Score (Reddit) | Avg Comments |
|-------|--------------------|--------------------|--------------|
| ... | ... | ... | ... |

### Key Findings

#### [Brand A] Strengths
- [What people praise, with example quotes]

#### [Brand A] Weaknesses
- [What people complain about, with example quotes]

#### [Brand B] Strengths / Weaknesses
...

### Competitive Positioning Map
- **[Brand A]:** Positioned as [description]
- **[Brand B]:** Positioned as [description]
- **Switching signals:** [users switching from X to Y, with reasons]

### Audience Overlap
[X users posted about multiple brands — analysis of their preferences]

### Recommendations
[3-5 actionable insights based on the competitive landscape]
```

## Example Prompts

- "Compare Tesla vs Rivian vs Lucid on Twitter sentiment"
- "Share of voice: Figma vs Sketch vs Adobe XD"
- "Competitive analysis for Notion vs Obsidian vs Roam Research on Reddit"
- "How does Claude sentiment compare to ChatGPT and Gemini?"

## Notes

- Expand brand names carefully to avoid false positives (common words need exclusions)
- Reddit provides qualitative depth; Twitter provides quantitative breadth
- Free tier: 100K results/month at [xpoz.ai](https://xpoz.ai?utm_source=github&utm_medium=agent-skills&utm_campaign=competitive-intel)
- For large comparisons (5+ brands), use CSV exports and analyze locally with pandas/Excel
