---
name: reddit-research
version: 2026-02-23
description: Search and analyze Reddit discussions for market research, product feedback, and community insights using Xpoz. Use when asked to "search Reddit", "what does Reddit think about X", "Reddit feedback on X", "subreddit analysis", or "Reddit market research".
---

# Reddit Research

## Overview

Search and analyze Reddit discussions across all subreddits. Extract community opinions, identify pain points, discover product feedback, and understand market sentiment — all without Reddit API keys.

## When to Use

Activate when the user asks:
- "What does Reddit think about [PRODUCT]?"
- "Search Reddit for [TOPIC]"
- "What are people saying about [BRAND] on Reddit?"
- "Reddit feedback on [TOOL/SERVICE]"
- "Find Reddit discussions about [TOPIC]"
- "Market research on Reddit for [INDUSTRY]"

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

Get a free API key at [xpoz.ai/get-token](https://xpoz.ai/get-token?utm_source=github&utm_medium=agent-skills&utm_campaign=reddit-research) — 100K results/month free, no credit card required.

### Auth Errors
| Problem | Solution |
|---------|----------|
| MCP: "Unauthorized" | Run `mcporter config login xpoz --reset` |
| SDK: `AuthenticationError` | Verify API key at [xpoz.ai/settings](https://xpoz.ai/settings) |
| mcporter not found | Included with OpenClaw — ensure OpenClaw is installed |


## Step-by-Step Instructions

### Step 1: Parse the Request

Extract:
- **Topic/product/brand** to research
- **Specific questions** the user wants answered
- **Time period** (default: last 30 days)
- **Subreddit filter** (if user specifies one)

Build the query:
- Product name + common alternatives: `"Cursor" OR "Cursor IDE" OR "cursor.sh"`
- Include comparison terms: `"Cursor vs" OR "Cursor alternative"`
- For feedback: `"Cursor" AND ("love" OR "hate" OR "switched" OR "review")`

### Step 2: Fetch Reddit Posts

#### Via MCP

```
Call getRedditPostsByKeywords:
  query: "<expanded query>"
  fields: ["id", "title", "text", "authorUsername", "createdAtDate", "score", "numComments", "subreddit", "url"]
  startDate: "<30 days ago, YYYY-MM-DD>"
  endDate: "<today, YYYY-MM-DD>"
```

**CRITICAL:** Call `checkOperationStatus` with the returned `operationId` and poll until "completed" (up to 8 retries, ~5 seconds apart).

**For users who posted about the topic:**
```
Call getRedditUsersByKeywords:
  query: "<query>"
  fields: ["id", "username", "relevantPostsCount"]
  startDate: "<30 days ago>"
```

#### Via Python SDK

```python
from xpoz import XpozClient

client = XpozClient()

# Search Reddit posts
results = client.reddit.search_posts(
    '"Cursor" OR "Cursor IDE"',
    start_date="2026-01-24",
    end_date="2026-02-23",
    fields=["id", "title", "text", "author_username", "created_at_date", "score", "num_comments", "subreddit", "url"]
)

# Collect all pages
all_posts = results.data
while results.has_next_page():
    results = results.next_page()
    all_posts.extend(results.data)

print(f"Found {len(all_posts)} Reddit posts")

# Export to CSV for deeper analysis
csv_url = results.export_csv()

client.close()
```

#### Via TypeScript SDK

```typescript
import { XpozClient } from "xpoz";

const client = new XpozClient();
await client.connect();

const results = await client.reddit.searchPosts('"Cursor" OR "Cursor IDE"', {
  startDate: "2026-01-24",
  endDate: "2026-02-23",
  fields: ["id", "title", "text", "authorUsername", "createdAtDate", "score", "numComments", "subreddit", "url"],
});

console.log(`Found ${results.pagination.totalRows} posts`);
const csvUrl = await results.exportCsv();

await client.close();
```

### Step 3: Analyze the Data

**Subreddit Distribution:**
- Group posts by subreddit
- Identify where the most discussion happens
- Note subreddit context (r/programming = developers, r/productivity = end users, etc.)

**Sentiment Analysis:**
- Reddit uses upvotes/downvotes as built-in sentiment (high score = community agrees)
- Posts with high `numComments` indicate controversial or engaging topics
- Score/comments ratio: high score + few comments = consensus; low score + many comments = debate

**Theme Extraction:**
Identify recurring themes:
- **Pain points**: complaints, frustrations, feature requests
- **Praise**: what users love, competitive advantages
- **Comparisons**: how the product compares to alternatives
- **Use cases**: how people actually use the product
- **Questions**: common confusion points or information gaps

**Tip:** Reddit posts often contain more nuanced, detailed opinions than Twitter. Prioritize posts with high `score` and `numComments` for quality insights.

### Step 4: Generate Report

```
## Reddit Research: [TOPIC]
**Period:** [date range] | **Posts analyzed:** [count]

### Overview
[2-3 sentence summary of what Reddit thinks]

### Subreddit Distribution
| Subreddit | Posts | Avg Score | Top Theme |
|-----------|-------|-----------|-----------|
| r/programming | X | X | Performance concerns |
| r/productivity | X | X | Workflow improvements |
| ... | ... | ... | ... |

### Key Themes

#### 1. 👍 What People Love
- [Theme with supporting quotes]
- [Theme with supporting quotes]

#### 2. 👎 Pain Points & Complaints
- [Theme with supporting quotes]
- [Theme with supporting quotes]

#### 3. 🔄 Comparisons & Alternatives
- [Product vs Competitor: community consensus]
- [Common alternatives mentioned]

#### 4. 💡 Feature Requests & Suggestions
- [Most requested features]
- [Creative use cases discovered]

### Top Posts (by engagement)
| Score | Comments | Subreddit | Title |
|-------|----------|-----------|-------|
| 1.2K | 234 | r/programming | "Title..." |
| ... | ... | ... | ... |

### Notable Quotes
> "Actual Reddit quote with context" — u/username in r/subreddit (⬆️ 456)

### Actionable Insights
[3-5 bullet points of what to do with this information]
```

## Example Prompts

- "What does Reddit think about Cursor IDE?"
- "Search Reddit for people complaining about Zapier pricing"
- "Reddit market research: what tools are indie hackers using for automation?"
- "Find Reddit posts comparing Claude vs GPT-4"
- "What's r/machinelearning saying about open-source LLMs?"

## Notes

- Reddit data includes post titles and body text — titles alone often reveal sentiment
- High `num_comments` posts are goldmines for qualitative research
- Free tier: 100K results/month at [xpoz.ai](https://xpoz.ai?utm_source=github&utm_medium=agent-skills&utm_campaign=reddit-research)
- For CSV export, use `export_csv()` / `exportCsv()` to download complete datasets
