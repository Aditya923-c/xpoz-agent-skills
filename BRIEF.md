# Build Brief: xpoz-agent-skills

## Goal
Create a GitHub repo with 6 agent skills in SKILL.md format for SkillsMP discovery.
Each skill uses Xpoz to give AI coding agents social media intelligence capabilities.

## SKILL.md Format (SkillsMP standard)
Each skill lives in `skills/<skill-name>/SKILL.md`. The file is what the AI agent reads when it activates the skill. It should contain:
1. A clear description comment at the top (// comment style)
2. When to use this skill
3. Step-by-step instructions the agent should follow
4. Integration options: MCP config OR SDK code (Python/TypeScript)
5. Example prompts and expected outputs
6. Links to xpoz.ai with UTM tags

## Integration Options (BOTH must be documented in each skill)

### Option A: MCP (for agents with MCP support like Claude Code)
```json
{
  "mcpServers": {
    "xpoz": {
      "url": "https://mcp.xpoz.ai/mcp",
      "transport": "http-stream",
      "headers": { "Authorization": "Bearer YOUR_XPOZ_API_KEY" }
    }
  }
}
```
Key MCP tools: getTwitterPostsByKeywords, getTwitterUsersByKeywords, searchTwitterUsers, getTwitterUser, getTwitterPostsByAuthor, getInstagramPostsByKeywords, getInstagramUsersByKeywords, getRedditPostsByKeywords, checkOperationStatus

### Option B: Python SDK
```bash
pip install xpoz
```
```python
from xpoz import XpozClient
client = XpozClient("YOUR_API_KEY")  # or set XPOZ_API_KEY env var
# client.twitter.search_posts(), client.twitter.get_users_by_keywords(), etc.
# client.instagram.search_posts(), client.reddit.search_posts()
# All paginated results support .export_csv()
client.close()
```

### Option C: TypeScript SDK
```bash
npm install xpoz
```
```typescript
import { XpozClient } from "xpoz";
const client = new XpozClient({ apiKey: "YOUR_API_KEY" });
await client.connect();
// client.twitter.searchPosts(), client.twitter.getUsersByKeywords(), etc.
await client.close();
```

## The 6 Skills to Build

### 1. social-sentiment-analyzer
Analyze brand/topic sentiment across Twitter, Reddit, Instagram.
- Search posts by keyword across platforms
- Classify sentiment (positive/neutral/negative)
- Extract recurring themes/narratives
- Generate summary report
- Use case: "Analyze sentiment around {brand} on Twitter and Reddit"

### 2. twitter-data-export
Export Twitter data to CSV for analysis. Simple, practical utility.
- Search tweets by keyword, author, date range
- Export full results to CSV (up to 500K rows)
- Download and analyze locally
- Use case: "Export all tweets about {topic} from last 30 days to CSV"

### 3. influencer-discovery
Find and evaluate influencers by niche, engagement, content.
- Search users by keywords across platforms
- Analyze follower counts, engagement rates
- Classify by tier (mega/macro/micro/nano)
- Rank by relevance and authenticity
- Use case: "Find top AI influencers on Twitter with high engagement"

### 4. reddit-research
Search and analyze Reddit discussions for market research.
- Search Reddit posts by keyword
- Analyze subreddit distribution
- Extract common opinions and pain points
- Use case: "What are people saying about {product} on Reddit?"

### 5. competitive-intel
Monitor competitor mentions and sentiment across social platforms.
- Compare multiple brands side by side
- Track share of voice
- Identify competitive positioning
- Use case: "Compare {brand} vs {competitor} on Twitter sentiment"

### 6. security-osint
Monitor social platforms for security threats and vulnerability discussions.
- Search for CVE mentions, breach discussions
- Track security community chatter
- Early warning for emerging threats
- Use case: "Find discussions about {CVE} or {vulnerability} on Twitter and Reddit"

## Repo Structure
```
xpoz-agent-skills/
├── README.md                    # Repo overview with skill index
├── LICENSE                      # MIT
├── skills/
│   ├── social-sentiment-analyzer/
│   │   └── SKILL.md
│   ├── twitter-data-export/
│   │   └── SKILL.md
│   ├── influencer-discovery/
│   │   └── SKILL.md
│   ├── reddit-research/
│   │   └── SKILL.md
│   ├── competitive-intel/
│   │   └── SKILL.md
│   └── security-osint/
│       └── SKILL.md
```

## README.md Should Include
- Brief intro: "Agent skills for social media intelligence, powered by Xpoz"
- Table of skills with descriptions
- Quick setup (MCP config + SDK install)
- Link to xpoz.ai
- GitHub topics to add: skill-md, agent-skills, claude-code, codex-cli, mcp, social-media-api, twitter-api, reddit-api

## Key Principles
1. Skills should be ACTIONABLE — the agent reads SKILL.md and can immediately execute
2. Include concrete example prompts users can try
3. Both MCP and SDK paths must work
4. Each skill should be self-contained (no cross-dependencies)
5. Professional tone, clear formatting
6. UTM tags on all xpoz.ai links: utm_source=github&utm_medium=agent-skills&utm_campaign={skill-name}

## API Key
Get free at https://xpoz.ai/get-token (100K results/month free)

## DO NOT
- Make up fake API responses or data
- Include actual API keys
- Over-promise capabilities
- Use "Twitter API alternative" as primary positioning
