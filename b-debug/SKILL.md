---
name: b-debug
description: >
  Systematically debug an error or unexpected behavior using codebase analysis,
  web search, and structured root cause analysis.
  ALWAYS use this skill when the user pastes an error message, stack trace, or says
  "debug", "tại sao bị lỗi", "lỗi này là gì", "fix this error", "không hiểu lỗi này",
  "why is this failing", or describes unexpected behavior in their code.
---

# b-debug

Systematic debugging workflow: search the error online, trace it through the codebase,
reason through root cause, and propose a fix with explanation.

## Tools required

- `jcodemunch` — trace call chain and find related symbols
- `brave_web_search` — search error message online
- `sequential-thinking` — structured root cause analysis

If `jcodemunch` is unavailable: "❌ jcodemunch MCP is not connected. Please check `/mcp`."
If `brave-search` is unavailable: skip Step 1, continue with Steps 2–4.

## Steps

### 1. Search the error online (brave_web_search)
- Search the exact error message or a cleaned-up version
- Look for: known bug reports, GitHub issues, Stack Overflow answers
- Note if this is a known issue with a version-specific fix
- Skip if the error is clearly project-specific (custom error messages, business logic)

### 2. Trace in codebase (jcodemunch)
```
list_repos         → ensure project is indexed (index_folder if not)
search_symbols     → find symbols mentioned in the stack trace
get_symbol         → get full source of directly involved functions
search_text        → search for the exact error string if thrown manually
```

Follow the call chain:
- Where is the error thrown or triggered?
- What calls that function?
- What data is passed in?

### 3. Root cause analysis (sequential-thinking)
Use `sequential-thinking` to reason step by step:
1. What is the error exactly?
2. Where does it originate?
3. What condition triggers it?
4. What is the actual vs expected behavior?
5. What is the root cause?

### 4. Propose fix

Output a clear diagnosis and fix:

```
## Debug Report

**Error:** `[error message]`

**Root Cause:**
[1-3 sentences explaining WHY this happens]

**Location:**
- `path/to/file.ts` → `FunctionName` (line ~N)

**Fix:**
[Code snippet or clear description of what to change]

**Why this fix works:**
[Brief explanation]

**Related (if found online):**
- [link to relevant issue/answer if applicable]
```

## Rules

- Always search online first — many errors are known bugs or common mistakes
- Never guess the root cause without tracing through the code
- If the stack trace points to a library (not user code), check context7 for known issues
- If multiple root causes are possible, list them ranked by likelihood
- Do NOT silently apply the fix — always explain it first and wait for user confirmation
- If the bug requires a larger refactor, say so explicitly rather than proposing a band-aid fix