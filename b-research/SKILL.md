---
name: b-research
description: >
  Deep research by combining Brave Search + Firecrawl scraping to get full page content.
  ALWAYS use this skill when the user asks to "research", "tìm hiểu sâu", "deep dive",
  "so sánh", "compare", "tổng hợp thông tin về", or any query that needs more than
  search snippets — such as evaluating tools, understanding concepts deeply, comparing
  options, or producing a comprehensive report. Prefer this over b-search when the user
  wants depth, not just a quick answer.
---

# b-research

Deep research workflow: search for relevant URLs via Brave Search, scrape full content
via Firecrawl, then synthesize into a comprehensive report with citations.

## When to use

- User asks to research, compare, or deeply understand a topic
- User wants a report or summary of multiple sources
- A quick search snippet is not enough — full page content is needed
- User says: "tìm hiểu", "research", "deep dive", "so sánh", "tổng hợp", "viết report về"

## Tools required

- `brave_web_search` — from `brave-search` MCP server
- `firecrawl_scrape` — from `firecrawl` MCP server

If either tool is unavailable, stop and tell the user:
- brave-search missing: "❌ brave-search MCP is not connected. Please check `/mcp`."
- firecrawl missing: "❌ firecrawl MCP is not connected. Please check `/mcp`."

Do NOT fall back to built-in web search or training data.

## Steps

### 1. Search
- Use `brave_web_search` with a focused English query (unless topic is Vietnamese-specific)
- Fetch 5–8 results
- Pick the **3–5 most relevant URLs** — prioritize official docs, authoritative blogs, recent articles

### 2. Scrape
- Call `firecrawl_scrape` on each selected URL
- Use `formats: ["markdown"]` to get clean content
- If a page fails to scrape, skip it and note it in the report

### 3. Synthesize
- Read the scraped content carefully
- Answer the user's specific question based on actual content — not training data
- Produce a structured report (see output format below)
- If sources conflict, note the disagreement explicitly

## Output format

```
## [Topic / Research Question]

### Summary
[2–4 sentence direct answer to the user's question]

### Key Findings
- **[Finding 1]**: ...
- **[Finding 2]**: ...
- **[Finding 3]**: ...

### [Optional: Comparison Table or Deep Dive Section]
...

### Sources
- [Page Title](URL) — [one line on what this source contributed]
- [Page Title](URL) — ...
```

## Rules

- Always scrape — never rely on search snippets alone
- Max 5 URLs to scrape per research session (to avoid excessive tool calls)
- Cite every claim with its source URL
- If the user asks a specific question, answer it directly in the Summary before going into detail
- Keep the report focused — omit irrelevant scraped content
