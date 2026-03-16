---
name: b-tech-compare
description: >
  Compare 2-3 technology options for a specific problem and give a clear recommendation.
  ALWAYS use this skill when the user says "so sánh X vs Y", "nên dùng X hay Y",
  "X vs Y vs Z", "which is better for...", "should I use X or Y", "trade-offs giữa...",
  "lựa chọn nào tốt hơn cho...", or any question comparing technical options before
  making an architectural or library decision.
---

# b-tech-compare

Compares 2–3 tech options for a specific use case using real benchmarks, official docs,
and structured trade-off analysis. Ends with a clear recommendation.

## Tools required

- `brave_web_search` — find benchmarks, reviews, real-world usage
- `context7` — fetch official docs for each option
- `sequential-thinking` — structured trade-off reasoning

If `brave-search` is unavailable: "❌ brave-search MCP is not connected. Please check `/mcp`."

## Steps

### 1. Clarify the context
Before researching, identify:
- What problem needs to be solved?
- What constraints exist? (scale, team familiarity, existing stack, budget)
- What matters most? (performance, DX, cost, ecosystem, simplicity)

If unclear, ask one focused question before proceeding.

### 2. Research each option (brave_web_search)

For each technology, search:
```
"[tech] vs [other tech] [use case] 2025 OR 2026"
"[tech] benchmark [use case]"
"[tech] pros cons production"
```

Look for:
- Real-world performance benchmarks
- Known limitations or gotchas
- Community sentiment and adoption trends
- Recent updates or deprecations

### 3. Fetch official docs (context7)
- Get the key concepts and API surface for each option
- Note any version-specific considerations
- Verify features claimed in articles are actually in the current version

### 4. Analyze trade-offs (sequential-thinking)
Use `sequential-thinking` to reason through:
- How does each option fit the specific use case?
- What are the real trade-offs given the constraints?
- Which trade-offs matter most for this situation?

### 5. Output comparison

```
## Tech Comparison: [Option A] vs [Option B] (vs [Option C])
**Use case:** [specific problem being solved]

### Comparison Table
| Criteria | [Option A] | [Option B] | [Option C] |
|---|---|---|---|
| Performance | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| DX / Ease of use | ... | ... | ... |
| Ecosystem | ... | ... | ... |
| Production maturity | ... | ... | ... |
| [relevant criteria] | ... | ... | ... |

### [Option A]
**Strengths:** ...
**Weaknesses:** ...
**Best for:** ...

### [Option B]
**Strengths:** ...
**Weaknesses:** ...
**Best for:** ...

### [Option C] (if applicable)
...

---

## ✅ Recommendation: [Option X]

**Why:** [2-3 sentences explaining why this option wins for the specific use case and constraints]

**Caveat:** [Any condition where a different option would be better]

**Next step:** Run `b-plan` to start implementing with [Option X].

---
*Sources: [URLs of key articles/benchmarks used]*
```

## Rules

- Always tie the recommendation to the **specific use case** — never recommend in the abstract
- If the answer is genuinely "it depends", say so clearly and explain what it depends on
- Cite sources for benchmark claims — do not state performance numbers without a source
- If one option is clearly dominant, say so directly — do not hedge unnecessarily
- Max 3 options — if user asks for more, suggest narrowing down first
- End with a clear next step (usually `b-plan`)