---
name: twitter-data-export
version: 2026-02-23
description: Export Twitter/X data to CSV for analysis using Xpoz. Search by keywords, author, date range, and download complete datasets (up to 500K rows). Use when asked to "export tweets", "download Twitter data", "get tweets as CSV", "Twitter dataset", or "bulk tweet download".
---

# Twitter Data Export

## Overview

Search and export Twitter/X data to CSV files for analysis. Supports keyword search, author-based search, date filtering, and bulk exports up to 500K rows — no Twitter API keys required.

## When to Use

Activate when the user asks:
- "Export tweets about [TOPIC] to CSV"
- "Download all tweets from @[USER]"
- "Get Twitter data for [KEYWORD] from last month"
- "I need a dataset of tweets about [TOPIC]"
- "Bulk download tweets matching [QUERY]"
- "Twitter data export"

## Prerequisites

**Xpoz API Key** — get one free at [xpoz.ai/get-token](https://xpoz.ai/get-token?utm_source=github&utm_medium=agent-skills&utm_campaign=twitter-data-export) (100K results/month free)

## Step-by-Step Instructions

### Step 1: Parse the Request

Extract:
- **Query**: keywords, hashtags, or phrases to search
- **Author** (optional): specific Twitter username
- **Date range** (default: last 30 days)
- **Fields** the user cares about (default: full export)

Build the query using boolean operators:
- Exact phrase: `"machine learning"`
- OR: `"AI" OR "artificial intelligence"`
- AND: `"Tesla" AND "earnings"`
- NOT: `"crypto" NOT "scam"`
- Combined: `("deep learning" OR "neural network") AND python`

### Step 2: Search and Export

#### Via MCP

**Search by keywords:**
```
Call getTwitterPostsByKeywords:
  query: "<query>"
  fields: ["id", "text", "authorUsername", "authorId", "createdAtDate", "likeCount", "retweetCount", "quoteCount", "impressionCount", "language"]
  startDate: "<YYYY-MM-DD>"
  endDate: "<YYYY-MM-DD>"
  language: "en" (optional)
```

**Search by author:**
```
Call getTwitterPostsByAuthor:
  identifier: "<username>"
  identifierType: "username"
  fields: ["id", "text", "createdAtDate", "likeCount", "retweetCount", "quoteCount", "impressionCount"]
  startDate: "<YYYY-MM-DD>"
  endDate: "<YYYY-MM-DD>"
```

**CRITICAL: Async Pattern** — calls return an `operationId`. Call `checkOperationStatus` with that ID and poll until "completed" (up to 8 retries, ~5 seconds apart).

**CSV Export (two options):**
1. Pass `responseType="csv"` in the original call to get a CSV download directly
2. Or use the `dataDumpExportOperationId` from the response — call `checkOperationStatus` with it to get an S3 download URL for the complete dataset

#### Via Python SDK

```python
from xpoz import XpozClient

client = XpozClient()  # Uses XPOZ_API_KEY env var

# Search by keywords
results = client.twitter.search_posts(
    '"artificial intelligence" AND ethics',
    start_date="2026-01-01",
    end_date="2026-02-23",
    language="en",
    fields=["id", "text", "author_username", "created_at_date", "like_count", "retweet_count", "impression_count"]
)

print(f"Found {results.pagination.total_rows:,} tweets")

# Export entire result set to CSV (up to 500K rows)
csv_url = results.export_csv()
print(f"Download CSV: {csv_url}")

# Or search by author
author_results = client.twitter.get_posts_by_author(
    "elonmusk",
    start_date="2026-01-01",
    fields=["id", "text", "created_at_date", "like_count", "retweet_count"]
)
author_csv = author_results.export_csv()

client.close()
```

**Download and analyze locally:**
```python
import pandas as pd
import subprocess

# Download the CSV
subprocess.run(["curl", "-L", "-o", "tweets.csv", csv_url])

# Load and analyze
df = pd.read_csv("tweets.csv")
print(f"Total tweets: {len(df)}")
print(f"Date range: {df['created_at_date'].min()} to {df['created_at_date'].max()}")
print(f"Average likes: {df['like_count'].mean():.1f}")
print(f"Top authors:\n{df['author_username'].value_counts().head(10)}")
```

#### Via TypeScript SDK

```typescript
import { XpozClient } from "xpoz";

const client = new XpozClient();
await client.connect();

const results = await client.twitter.searchPosts('"artificial intelligence" AND ethics', {
  startDate: "2026-01-01",
  endDate: "2026-02-23",
  language: "en",
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount"],
});

console.log(`Found ${results.pagination.totalRows.toLocaleString()} tweets`);

// Export to CSV
const csvUrl = await results.exportCsv();
console.log(`Download: ${csvUrl}`);

await client.close();
```

### Step 3: Present Results

After export, provide the user with:

1. **Summary stats**: total rows, date range, top authors, avg engagement
2. **CSV download link** (from `export_csv()` / `exportCsv()`)
3. **Sample data**: show first 5-10 rows as a table
4. **Suggested analysis**: what they might want to do with the data

```
## Export Complete: [QUERY]

**Rows exported:** 12,456
**Period:** Jan 1 – Feb 23, 2026
**Download:** [CSV link]

### Sample Data
| Date | Author | Text (truncated) | Likes | RTs |
|------|--------|-------------------|-------|-----|
| ... | ... | ... | ... | ... |

### Quick Stats
- Avg likes per tweet: 45.2
- Most active author: @user (234 tweets)
- Peak day: Feb 14, 2026 (1,203 tweets)

### Suggested Next Steps
- Load into pandas/Excel for deeper analysis
- Filter by engagement (like_count > 100) for high-impact posts
- Group by date for trend analysis
```

## Available Fields

| Field | Description |
|-------|-------------|
| `id` | Tweet ID |
| `text` | Full tweet text |
| `authorUsername` | Author's username |
| `authorId` | Author's numeric ID |
| `createdAtDate` | Post date (YYYY-MM-DD) |
| `likeCount` | Number of likes |
| `retweetCount` | Number of retweets |
| `quoteCount` | Number of quote tweets |
| `impressionCount` | Number of impressions |
| `replyCount` | Number of replies |
| `language` | Detected language |
| `isRetweet` | Whether it's a retweet |
| `isReply` | Whether it's a reply |

## Example Prompts

- "Export all tweets mentioning 'Claude Code' from the last 2 weeks to CSV"
- "Download @OpenAI's tweets from January 2026"
- "Get a dataset of tweets about 'MCP server' OR 'model context protocol'"
- "Export tweets about the Super Bowl with more than 100 likes"

## Notes

- Maximum export size: ~500K rows per CSV
- Date range: up to 60-day rolling windows
- Free tier: 100K results/month at [xpoz.ai](https://xpoz.ai?utm_source=github&utm_medium=agent-skills&utm_campaign=twitter-data-export)
- Pro: $20/month for 1M results
- No Twitter API keys needed — Xpoz handles all data access
