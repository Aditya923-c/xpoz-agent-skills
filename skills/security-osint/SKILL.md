---
name: security-osint
version: 2026-02-23
description: Monitor social platforms for security threats, vulnerability discussions, and breach intelligence using Xpoz. Use when asked to "find CVE discussions", "security threat monitoring", "OSINT social media", "vulnerability intelligence", or "breach mentions on Twitter/Reddit".
---

# Security OSINT

## Overview

Monitor Twitter/X and Reddit for security-relevant discussions: CVE mentions, vulnerability disclosures, breach reports, exploit chatter, and threat actor activity. Get early warning signals from security researcher communities before formal advisories.

## When to Use

Activate when the user asks:
- "Find discussions about [CVE-XXXX-XXXXX] on Twitter"
- "Monitor social media for [vulnerability/product] security issues"
- "What are security researchers saying about [TOPIC]?"
- "OSINT check for [software/company] breaches"
- "Track threat intelligence on social media"
- "Early warning for [technology] vulnerabilities"

## Prerequisites

**Xpoz API Key** — get one free at [xpoz.ai/get-token](https://xpoz.ai/get-token?utm_source=github&utm_medium=agent-skills&utm_campaign=security-osint)

## Step-by-Step Instructions

### Step 1: Parse the Request

Extract:
- **Target**: CVE ID, product name, company, or threat type
- **Scope**: specific vulnerability vs broad monitoring
- **Platforms**: Twitter + Reddit (both recommended for security)
- **Time period**: default last 7 days (security moves fast)

Build targeted queries:
- Specific CVE: `"CVE-2026-1234" OR "CVE20261234"`
- Product vulnerability: `"Log4j" AND ("vulnerability" OR "exploit" OR "CVE" OR "RCE" OR "patch")`
- Breach monitoring: `"[Company]" AND ("breach" OR "leaked" OR "hacked" OR "compromised" OR "data leak")`
- Threat actor: `"[Actor name]" AND ("attack" OR "campaign" OR "APT" OR "malware")`

### Step 2: Fetch Security Discussions

#### Via MCP

**Twitter — security researcher community:**
```
Call getTwitterPostsByKeywords:
  query: "<security query>"
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount", "impressionCount"]
  startDate: "<7 days ago, YYYY-MM-DD>"
  endDate: "<today, YYYY-MM-DD>"
  language: "en"
```

**Twitter — find security researchers discussing this:**
```
Call getTwitterUsersByKeywords:
  query: "<security query>"
  fields: ["id", "username", "name", "description", "followersCount", "relevantTweetsCount", "verified"]
  startDate: "<7 days ago>"
```

**Reddit — detailed technical discussions:**
```
Call getRedditPostsByKeywords:
  query: "<security query>"
  fields: ["id", "title", "text", "authorUsername", "createdAtDate", "score", "numComments", "subreddit", "url"]
  startDate: "<7 days ago>"
```

**CRITICAL:** Each call returns an `operationId` — poll `checkOperationStatus` until "completed".

#### Via Python SDK

```python
from xpoz import XpozClient

client = XpozClient()

# Search Twitter for CVE discussions
twitter_posts = client.twitter.search_posts(
    '"CVE-2026-1234" OR "CVE20261234"',
    start_date="2026-02-16",
    end_date="2026-02-23",
    fields=["id", "text", "author_username", "created_at_date", "like_count", "retweet_count"]
)

# Find security researchers who posted about it
researchers = client.twitter.get_users_by_keywords(
    '"CVE-2026-1234"',
    start_date="2026-02-16",
    fields=["username", "name", "description", "followers_count", "relevant_tweets_count", "verified"]
)

# Search Reddit (r/netsec, r/cybersecurity, etc.)
reddit_posts = client.reddit.search_posts(
    '"CVE-2026-1234"',
    start_date="2026-02-16",
    fields=["id", "title", "text", "author_username", "score", "num_comments", "subreddit", "url"]
)

# For broad product monitoring
product_vulns = client.twitter.search_posts(
    '"Apache" AND ("vulnerability" OR "CVE" OR "exploit" OR "RCE" OR "0day")',
    start_date="2026-02-16",
    fields=["id", "text", "author_username", "created_at_date", "like_count"]
)

client.close()
```

#### Via TypeScript SDK

```typescript
import { XpozClient } from "xpoz";

const client = new XpozClient();
await client.connect();

const twitterPosts = await client.twitter.searchPosts('"CVE-2026-1234"', {
  startDate: "2026-02-16",
  endDate: "2026-02-23",
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount"],
});

const redditPosts = await client.reddit.searchPosts('"CVE-2026-1234"', {
  startDate: "2026-02-16",
  fields: ["id", "title", "text", "score", "numComments", "subreddit", "url"],
});

await client.close();
```

### Step 3: Analyze Security Intelligence

**Timeline Reconstruction:**
- Sort all posts chronologically
- Identify: first public mention, researcher disclosure, vendor response, patch availability
- Note the gap between social media chatter and formal advisory (often 24-72 hours)

**Source Credibility:**
- Verified security researchers (check bio for "security", "pentester", "CISO", "researcher")
- High-follower accounts in security community
- Reddit posts in security-specific subreddits (r/netsec, r/cybersecurity, r/blueteam, r/redteam)
- Cross-reference: same finding reported by multiple independent researchers = higher confidence

**Severity Assessment:**
From social signals, assess:
- **Volume**: how many people are discussing it
- **Velocity**: how fast mentions are growing
- **Expert engagement**: are known researchers amplifying it
- **Exploit availability**: mentions of PoC, exploit code, "in the wild"
- **Impact scope**: which products/versions affected based on discussion

**Categorize findings:**
| Category | Indicators |
|----------|-----------|
| 🔴 Critical | Active exploitation, PoC available, wide impact, high engagement |
| 🟠 High | Researcher disclosure, pre-patch, significant discussion |
| 🟡 Medium | Early chatter, limited PoC, vendor aware |
| 🟢 Low | Theoretical, low engagement, patched |

### Step 4: Generate Report

```
## Security OSINT Report: [TARGET]
**Period:** [date range] | **Sources:** Twitter, Reddit

### Executive Summary
[2-3 sentences: what was found, severity level, recommended action]

### Severity: 🔴 CRITICAL / 🟠 HIGH / 🟡 MEDIUM / 🟢 LOW

### Timeline
| Date | Event | Source |
|------|-------|--------|
| Feb 16 | First mention by @researcher | Twitter |
| Feb 17 | PoC published on GitHub (linked in tweet) | Twitter |
| Feb 18 | Discussion in r/netsec (456 upvotes) | Reddit |
| Feb 19 | Vendor advisory released | Twitter |

### Key Findings
1. **[Finding]**: [Details with source quotes]
2. **[Finding]**: [Details with source quotes]

### Notable Sources
| Researcher | Platform | Followers | Credibility |
|-----------|----------|-----------|-------------|
| @security_researcher | Twitter | 45K | ✅ Known researcher |
| u/netsec_analyst | Reddit | — | r/netsec regular |

### Social Signal Metrics
- **Twitter mentions:** [count]
- **Reddit posts:** [count] (top subreddits: r/netsec, r/cybersecurity)
- **First seen:** [date/time]
- **Peak activity:** [date/time]
- **Sentiment:** [Panic/Concern/Informational/Dismissive]

### Recommended Actions
1. [Immediate action if critical]
2. [Monitoring recommendation]
3. [Patching/mitigation guidance from community discussion]

### Raw Intelligence
[Top 10 most relevant posts with full text, author, and engagement metrics]
```

## Example Prompts

- "Find all Twitter and Reddit discussions about CVE-2026-1234"
- "Monitor social media for Apache Struts vulnerabilities this week"
- "OSINT check: any breach mentions for Acme Corp?"
- "What are security researchers saying about the new Chrome zero-day?"
- "Track ransomware group activity on Twitter in the last 30 days"

## Security-Relevant Subreddits

| Subreddit | Focus |
|-----------|-------|
| r/netsec | Technical security research |
| r/cybersecurity | General cybersecurity news |
| r/blueteam | Defensive security |
| r/redteam | Offensive security |
| r/malware | Malware analysis |
| r/ReverseEngineering | Binary analysis |
| r/AskNetsec | Security Q&A |

## Notes

- Social media often surfaces vulnerability details **24-72 hours before** formal CVE publication
- Cross-reference Twitter + Reddit for higher confidence — single-source findings need verification
- Free tier: 100K results/month at [xpoz.ai](https://xpoz.ai?utm_source=github&utm_medium=agent-skills&utm_campaign=security-osint)
- For ongoing monitoring, combine with scheduled runs (cron + this skill)
