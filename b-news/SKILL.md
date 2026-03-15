---
name: b-news
description: >
  Aggregate and summarize today's top tech news from curated sources.
  ALWAYS use this skill when the user says "tin tức hôm nay", "news hôm nay",
  "có gì mới hôm nay", "tech news", "b-news", "điểm tin", or asks what's
  happening in tech today. Trigger even if the user just says "hôm nay có gì mới không".
---

# b-news

Aggregates today's top tech news from 8 curated sources using Brave Search,
then groups stories by topic/category into a clean daily digest.

## Tools required

- `brave_web_search` — from `brave-search` MCP server (required)
- `firecrawl_scrape` — from `firecrawl` MCP server (optional, for detail)

If `brave-search` is unavailable, stop and tell the user:
"❌ brave-search MCP is not connected. Please check `/mcp`."

## Sources

| Source | Domain |
|---|---|
| 9to5Google | 9to5google.com |
| 9to5Linux | 9to5linux.com |
| 9to5Mac | 9to5mac.com |
| Engadget | engadget.com |
| How-To Geek | howtogeek.com |
| Android Central | androidcentral.com |
| TechCrunch | techcrunch.com |
| The Verge | theverge.com |

## Steps

### 1. Search headlines from all 8 sources

Run **4 parallel searches** (group sources to reduce tool calls):

```
Search 1: "site:9to5google.com OR site:9to5mac.com today"
Search 2: "site:theverge.com OR site:techcrunch.com today"
Search 3: "site:engadget.com OR site:androidcentral.com today"
Search 4: "site:howtogeek.com OR site:9to5linux.com today"
```

- Set `count: 5` per search
- Focus on results from today or the last 24 hours
- Collect all headlines + URLs + brief snippets

### 2. Categorize stories

Group collected stories into categories:

- **AI & Machine Learning** — LLMs, AI tools, model releases
- **Mobile & Devices** — phones, tablets, wearables
- **Software & Apps** — OS updates, app releases, browser news
- **Linux & Open Source** — distro releases, open source projects
- **Big Tech** — Google, Apple, Meta, Microsoft, Amazon news
- **Security & Privacy** — vulnerabilities, data breaches, policies
- **Other** — anything that doesn't fit above

Discard duplicate stories covering the same event — keep the best source.

### 3. Scrape for detail (optional)

Only scrape if the user asks to "đọc thêm về..." or "chi tiết về..." a specific story.
Use `firecrawl_scrape` on that specific URL only — do not bulk scrape.

## Output format

```
# 📰 Tech News — [Today's Date]

## 🤖 AI & Machine Learning
- **[Headline]** — [1 sentence summary] ([Source](URL))
- ...

## 📱 Mobile & Devices
- **[Headline]** — [1 sentence summary] ([Source](URL))
- ...

## 💻 Software & Apps
- ...

## 🐧 Linux & Open Source
- ...

## 🏢 Big Tech
- ...

## 🔒 Security & Privacy
- ...

## 📌 Other
- ...

---
*Sources: 9to5Google, 9to5Linux, 9to5Mac, Engadget, How-To Geek, Android Central, TechCrunch, The Verge*
```

## Rules

- Always include today's date in the header
- Max 5 stories per category — prioritize most impactful/interesting
- Each story: headline + 1 sentence summary + source link
- Skip opinion pieces, listicles, and sponsored content when possible
- If a category has no stories, omit it entirely
- Do not bulk scrape — use search snippets for summaries