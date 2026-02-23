---
name: security-osint
version: 2026-02-23
description: Monitor social platforms for security threats, vulnerability discussions, and breach intelligence using Xpoz. Use when asked to "find CVE discussions", "security threat monitoring", "OSINT social media", "vulnerability intelligence", "breach mentions", or "threat intel from Twitter/Reddit".
---

# Security OSINT

## Overview

Monitor Twitter/X and Reddit for security-related discussions — CVE mentions, zero-day chatter, breach reports, exploit code sharing, and emerging threats. Provides early warning intelligence often 24-48 hours before formal advisories.

## When to Use

Activate when the user asks:
- "Find discussions about [CVE-XXXX-XXXXX] on Twitter"
- "What's the security community saying about [VULNERABILITY]?"
- "Monitor social media for [SOFTWARE] vulnerabilities"
- "OSINT research on [THREAT ACTOR/CAMPAIGN]"
- "Are there breach reports about [COMPANY] on social media?"
- "Threat intelligence from Twitter and Reddit"

## Prerequisites

**Xpoz API Key** — get one free at [xpoz.ai/get-token](https://xpoz.ai/get-token?utm_source=github&utm_medium=agent-skills&utm_campaign=security-osint)

## Step-by-Step Instructions

### Step 1: Parse the Request

Extract:
- **Target**: CVE ID, software name, company, threat actor, or general topic
- **Scope**: specific vulnerability vs broad monitoring
- **Platforms** (default: Twitter + Reddit — where security researchers are most active)
- **Time period** (default: last 7 days; use 24-48h for breaking threats)

Build targeted queries:

**For a specific CVE:**
```
"CVE-2026-1234" OR "CVE202612345"
```

**For a software vulnerability:**
```
("[SOFTWARE]" AND ("vulnerability" OR "vuln" OR "exploit" OR "RCE" OR "zero-day" OR "0day" OR "CVE"))
```

**For breach monitoring:**
```
("[COMPANY]" AND ("breach" OR "hacked" OR "leak" OR "data leak" OR "compromised" OR "ransomware"))
```

**For threat actor tracking:**
```
("[THREAT ACTOR]" OR "[KNOWN ALIASES]") AND ("attack" OR "campaign" OR "APT" OR "malware")
```

### Step 2: Fetch Security Discussions

#### Via MCP

**Twitter (security researchers are very active here):**
```
Call getTwitterPostsByKeywords:
  query: "<security query>"
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount", "impressionCount"]
  startDate: "<7 days ago, YYYY-MM-DD>"
  endDate: "<today, YYYY-MM-DD>"
  language: "en"
```

**Find security researchers discussing the topic:**
```
Call getTwitterUsersByKeywords:
  query: "<security query>"
  fields: ["id", "username", "name", "description", "followersCount", "relevantTweetsCount", "relevantTweetsLikesSum", "verified"]
  startDate: "<7 days ago>"
```

**Reddit (r/netsec, r/cybersecurity, r/hacking, etc.):**
```
Call getRedditPostsByKeywords:
  query: "<security query>"
  fields: ["id", "title", "text", "authorUsername", "createdAtDate", "score", "numComments", "subreddit", "url"]
  startDate: "<7 days ago>"
```

**CRITICAL:** Poll `checkOperationStatus` with each `operationId` until "completed".

#### Via Python SDK

```python
from xpoz import XpozClient

client = XpozClient()

cve_id = "CVE-2026-1234"
query = f'"{cve_id}"'

# Twitter - security researcher chatter
twitter_posts = client.twitter.search_posts(
    query,
    start_date="2026-02-16",
    end_date="2026-02-23",
    fields=["id", "text", "author_username", "created_at_date", "like_count", "retweet_count", "impression_count"]
)

# Find researchers discussing it
researchers = client.twitter.get_users_by_keywords(
    query,
    start_date="2026-02-16",
    fields=["username", "name", "description", "followers_count", "relevant_tweets_count", "verified"]
)

# Reddit - deeper technical discussions
reddit_posts = client.reddit.search_posts(
    query,
    start_date="2026-02-16",
    fields=["id", "title", "text", "author_username", "created_at_date", "score", "num_comments", "subreddit", "url"]
)

print(f"Twitter: {twitter_posts.pagination.total_rows} posts")
print(f"Reddit: {reddit_posts.pagination.total_rows} posts")
print(f"Researchers: {researchers.pagination.total_rows} users")

# Export for analysis
twitter_csv = twitter_posts.export_csv()
reddit_csv = reddit_posts.export_csv()

client.close()
```

#### Via TypeScript SDK

```typescript
import { XpozClient } from "xpoz";

const client = new XpozClient();
await client.connect();

const cveId = "CVE-2026-1234";

const twitterPosts = await client.twitter.searchPosts(`"${cveId}"`, {
  startDate: "2026-02-16",
  endDate: "2026-02-23",
  fields: ["id", "text", "authorUsername", "createdAtDate", "likeCount", "retweetCount"],
});

const redditPosts = await client.reddit.searchPosts(`"${cveId}"`, {
  startDate: "2026-02-16",
  fields: ["id", "title", "text", "score", "numComments", "subreddit"],
});

await client.close();
```

### Step 3: Analyze Threat Intelligence

**Timeline Reconstruction:**
- Sort all posts chronologically
- Identify the first public mention (potential disclosure date)
- Track how discussion evolved (initial report → PoC → exploitation → patches)

**Severity Assessment (from social signals):**
| Signal | Indicates |
|--------|-----------|
| High engagement + rapid spread | Critical/actively exploited |
| Security researchers sharing PoC code | Weaponization in progress |
| Vendor accounts responding | Acknowledged, patch likely coming |
| Low volume, technical-only discussion | Early stage or low severity |
| Mentions of "in the wild" / "actively exploited" | Immediate action needed |

**Source Credibility:**
- Verified security researchers (check bio for "security", "pentest", "CISO", "CVE")
- High follower count in security niche
- Posted from known security company accounts
- Cross-referenced across multiple independent sources

**Key Information to Extract:**
- Affected software/versions
- Attack vector (remote/local, authentication required?)
- Exploit availability (PoC published?)
- Patch status (fixed? workaround available?)
- Active exploitation reports
- IoCs (indicators of compromise) shared

### Step 4: Generate Report

```
## Security Intelligence: [TARGET]
**Period:** [date range] | **Sources:** Twitter ([X] posts), Reddit ([X] posts)

### ⚠️ Threat Summary
**Severity:** Critical / High / Medium / Low
**Status:** [Active exploitation / PoC available / Discussion only / Patched]
**First seen:** [date of earliest social mention]

### Timeline
| Date | Source | Event |
|------|--------|-------|
| Feb 16 | Twitter @researcher | First public mention of vulnerability |
| Feb 17 | Reddit r/netsec | Technical analysis posted |
| Feb 18 | Twitter @vendor | Patch announced |

### Technical Details (from social sources)
- **Affected:** [software, versions]
- **Vector:** [remote/local, auth requirements]
- **Impact:** [RCE, data leak, DoS, etc.]
- **Exploit:** [PoC available? Where?]
- **Patch:** [Available? Version? Workaround?]

### Key Voices
| Researcher | Followers | Posts | Credibility |
|-----------|-----------|-------|-------------|
| @security_expert | 50K | 3 | High (CISO at [company]) |
| ... | ... | ... | ... |

### Notable Posts
> "Actual quote from security researcher" — @username (❤️ X, 🔁 X)

### Subreddit Activity
| Subreddit | Posts | Top Thread |
|-----------|-------|-----------|
| r/netsec | X | "Title..." (⬆️ X, 💬 X) |
| r/cybersecurity | X | "Title..." |

### Recommended Actions
1. [Immediate: patch/mitigate if affected]
2. [Monitor: watch for exploitation reports]
3. [Investigate: check logs for IoCs]
```

## Example Prompts

- "What's the security community saying about CVE-2026-1234?"
- "Monitor Twitter for Log4Shell discussions in the last 48 hours"
- "OSINT: find breach reports about [Company] on social media"
- "Are there any zero-day discussions about Chrome this week?"
- "Track discussions about the latest ransomware campaign on Twitter and Reddit"
- "Find security researchers talking about MCP server vulnerabilities"

## Notes

- Security discussions on Twitter often precede formal CVE publication by 24-48 hours
- Reddit r/netsec and r/cybersecurity provide technical depth; Twitter provides speed
- Use `relevantTweetsCount` from user search to find who's most actively discussing a threat
- Be careful with sensitive information — don't amplify exploit code or IoCs unnecessarily
- Free tier: 100K results/month at [xpoz.ai](https://xpoz.ai?utm_source=github&utm_medium=agent-skills&utm_campaign=security-osint)
