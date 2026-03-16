---
name: b-context
description: >
  Quickly scan the codebase to build context before implementing a small task or hotfix.
  Use this skill when the user says "check codebase", "xem codebase", "context trước khi fix",
  "hotfix", "quick fix", "sửa nhanh", or starts implementing a small task WITHOUT a b-plan.
  Do NOT use when b-plan has already been run — use PLAN.md context instead.
---

# b-context

Quick codebase scan using jcodemunch to build just enough context for a small task or hotfix.
Outputs a concise summary directly in the conversation — no files written.

## Tools required

- `jcodemunch` — scan codebase structure and symbols
- `context7` — fetch library docs (optional, only if needed)

If `jcodemunch` is unavailable: "❌ jcodemunch MCP is not connected. Please check `/mcp`."

## When to use vs b-plan

| | b-context | b-plan |
|---|---|---|
| Task size | Small / hotfix | Medium / large |
| Output | Conversation only | `.claude/b-plan/PLAN.md` |
| Includes steps | No | Yes |
| Includes architecture | No | Yes |

If the task turns out to be larger than expected mid-scan, suggest running `b-plan` instead.

## Steps

### 1. Index check
```
list_repos → if not indexed: index_folder with current directory
```

### 2. Quick scan (always run)
```
get_repo_outline   → overall structure
get_file_tree      → find relevant directories/files
```

### 3. Targeted lookup (only what's relevant to the task)
```
search_symbols     → find symbols related to the task
get_symbol         → full source of directly affected symbols only
```

Stop as soon as you have enough context — do not over-scan.

### 4. Output summary in conversation

```
## Codebase Context

**Relevant files:**
- `path/to/file.ts` — [what it does]
- ...

**Related symbols:**
- `FunctionName` in `path/to/file.ts` — [what it does]
- ...

**Patterns to follow:**
- [naming / structure pattern observed]

**Watch out for:**
- [potential conflict or side effect]

Ready to implement. 🚀
```

## Rules

- Keep the scan minimal — this is for quick tasks, not deep analysis
- Do NOT write any files
- Do NOT create an implementation plan — just surface context
- If task seems complex, say: "This looks like a medium/large task — consider running `b-plan` instead."
- After outputting context, stop and wait for the user to proceed