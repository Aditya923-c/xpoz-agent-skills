---
name: security-osint
version: 2026-02-23
description: Monitor social media for security threats, vulnerability discussions, CVE mentions, and breach reports using Xpoz. Use when asked to "find CVE discussions", "security threat monitoring", "OSINT social media", "vulnerability chatter", or "breach mentions on Twitter".
---

# Security OSINT

## Overview

Monitor Twitter/X and Reddit for security-relevant discussions: CVE mentions, vulnerability disclosures, breach reports, exploit chatter, and threat intelligence. Social platforms often surface security issues hours or days before formal advisories.

## When to Use

Activate when the user asks:
- "Find discussions about [CVE-XXXX-XXXXX] on Twitter"
- "What's the social chatter around [vulnerability/product]?"
- "Monitor Twitter for [security topic]"
- "OSINT search for [threat/breach]"
- "Are people discussing [zero-day/exploit] on social media?"
- "Security threat intelligence from Reddit"

## Prerequisites

**Xpoz API Key** — get one free at [xpoz.ai/get-token](https://xpoz.ai/get-token?utm_source=github&utm_medium=agent-skills&utm_campaign=security-osint)

## Step-by-Step Instructions

### Step 1: Parse the Request

Extract:
- **Target**: CVE ID, product name, vulnerability type, or threat actor
- **Platforms** (default: Twitter + Reddit — both are critical for security)
- **Time period** (default: last 7 days; use 24-48h for active incidents)
- **Urgency**: is this an active incident or general monitoring?

Build targeted queries:
- CVE: `"CVE-2026-1234" OR "CVE20261234"`
- Product vulnerability: `"Log4j" AND ("vulnerability" OR "exploit" OR "RCE" OR "CVE")`
- Breach: `"[company]" AND ("breach" OR "hacked" OR "leak" OR "compromised")`
- Threat actor: `"[name]" AND ("APT" OR "ransomware" OR "attack" OR "campaign")`

### Step 2: Fetch Security Discussions

#### Via MCP

**Twitter:**
```
Call getTwitterPostsByKeywords:
  query: "<security query>"
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount", "impressionCount"]
  startDate: "<7 days ago, YYYY-MM-DD>"
  endDate: "<today, YYYY-MM-DD>"
```

**Security researchers who posted about it:**
```
Call getTwitterUsersByKeywords:
  query: "<security query>"
  fields: ["id", "username", "name", "description", "followersCount", "relevantTweetsCount", "verified"]
  startDate: "<7 days ago>"
```

**Reddit (security subreddits are goldmines):**
```
Call getRedditPostsByKeywords:
  query: "<security query>"
  fields: ["id", "title", "text", "authorUsername", "createdAtDate", "score", "numComments", "subreddit", "url"]
  startDate: "<7 days ago>"
```

**CRITICAL:** Call `checkOperationStatus` with each returned `operationId` and poll until "completed".

#### Via Python SDK

```python
from xpoz import XpozClient

client = XpozClient()

# Search for CVE discussions on Twitter
twitter_results = client.twitter.search_posts(
    '"CVE-2026-1234" OR "CVE20261234"',
    start_date="2026-02-16",
    end_date="2026-02-23",
    fields=["id", "text", "author_username", "created_at_date", "like_count", "retweet_count"]
)

# Find security researchers discussing it
researchers = client.twitter.get_users_by_keywords(
    '"CVE-2026-1234"',
    start_date="2026-02-16",
    fields=["username", "name", "description", "followers_count", "relevant_tweets_count", "verified"]
)

# Reddit security communities
reddit_results = client.reddit.search_posts(
    '"CVE-2026-1234"',
    start_date="2026-02-16",
    fields=["id", "title", "text", "author_username", "score", "num_comments", "subreddit", "url"]
)

# For broader threat monitoring
threat_results = client.twitter.search_posts(
    '"Log4j" AND ("exploit" OR "RCE" OR "vulnerability" OR "patch")',
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

const twitterResults = await client.twitter.searchPosts('"CVE-2026-1234"', {
  startDate: "2026-02-16",
  endDate: "2026-02-23",
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount"],
});

const redditResults = await client.reddit.searchPosts('"CVE-2026-1234"', {
  startDate: "2026-02-16",
  fields: ["id", "title", "text", "authorUsername", "score", "numComments", "subreddit"],
});

await client.close();
```

### Step 3: Analyze for Intelligence

**Timeline Reconstruction:**
- Sort all posts chronologically
- Identify first mention (earliest social signal)
- Track how discussion evolved: discovery → analysis → exploit → patch
- Compare to official advisory dates (NVD, vendor advisories)

**Source Credibility:**
- High credibility: verified accounts, known security researchers, high followers in security niche
- Medium: active security community members, moderate engagement
- Low: new accounts, no security background, potentially FUD

**Severity Assessment from Social Signals:**
| Signal | Indicates |
|--------|-----------|
| Rapid spread (many RTs in hours) | High severity, active exploitation |
| Security researchers posting PoC | Exploit available, patch urgently |
| Vendor accounts responding | Acknowledged, fix in progress |
| Reddit threads in r/netsec with high score | Community-validated severity |
| "In the wild" / "actively exploited" mentions | Critical — immediate action |
| Mostly questions, few details | Early stage, unclear severity |

**Entity Extraction:**
- Affected products/versions
- CVE IDs mentioned
- Patch/mitigation links shared
- PoC/exploit references
- Threat actor attributions

### Step 4: Generate Report

```
## Security OSINT Report: [TARGET]
**Period:** [date range] | **Sources:** Twitter, Reddit | **Assessment:** [Critical/High/Medium/Low]

### Timeline
| Time | Source | Event |
|------|--------|-------|
| Feb 16, 09:00 | @researcher | First public mention of vulnerability |
| Feb 16, 14:00 | r/netsec | Detailed analysis posted |
| Feb 17, 08:00 | @vendor | Official advisory released |
| ... | ... | ... |

### Social Signal Summary
- **Twitter mentions:** X posts by Y unique authors
- **Reddit threads:** X posts across Y subreddits
- **First social mention:** [date/time] ([hours/days] before official advisory)
- **Key subreddits:** r/netsec, r/cybersecurity, r/sysadmin

### Key Findings
1. **[Finding]** — [evidence from posts]
2. **[Finding]** — [evidence from posts]

### Notable Researchers / Sources
| Author | Platform | Followers | Role | Key Contribution |
|--------|----------|-----------|------|-----------------|
| @researcher | Twitter | 50K | Security Researcher | First PoC analysis |
| u/user | Reddit | — | Sysadmin | Mitigation workaround |

### Relevant Posts (by impact)
| Platform | Author | Text | Engagement | Date |
|----------|--------|------|------------|------|
| Twitter | @user | "..." | ❤️ 2.3K 🔁 890 | Feb 16 |
| Reddit | u/user | "..." | ⬆️ 456, 💬 89 | Feb 16 |

### Extracted Intelligence
- **Affected:** [products, versions]
- **CVEs:** [list]
- **Exploitability:** [based on social signals]
- **Patches/Mitigations:** [links shared in posts]
- **Threat Actors:** [if attributed]

### Recommendations
[Actionable next steps based on findings]
```

## Example Prompts

- "Find all Twitter and Reddit discussions about CVE-2026-1234"
- "What's the security community saying about the latest Chrome zero-day?"
- "Monitor social media for mentions of ransomware attacks on healthcare"
- "OSINT search: what are people saying about the [Company] data breach?"
- "Find security researchers discussing supply chain attacks this week"

## Notes

- Social media often surfaces vulnerabilities 24-72 hours before official NVD publication
- Reddit's r/netsec, r/cybersecurity, r/sysadmin are high-signal sources
- Twitter's security research community is very active — look for users with "security", "pentester", "CISO", "infosec" in their bios
- Free tier: 100K results/month at [xpoz.ai](https://xpoz.ai?utm_source=github&utm_medium=agent-skills&utm_campaign=security-osint)
- For ongoing monitoring, run this skill periodically and compare results
