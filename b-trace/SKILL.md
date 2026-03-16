---
name: b-trace
description: >
  Trace data flow or execution path through the codebase to understand how something works.
  ALWAYS use this skill when the user asks "dữ liệu này đi qua đâu", "function X được gọi từ đâu",
  "trace flow của...", "luồng xử lý của...", "where does X come from", "what calls X",
  "how does data flow through...", or wants to understand an execution path without debugging an error.
  Use b-debug instead if there is an actual error or bug.
---

# b-trace

Traces data flow or execution path through the codebase using jcodemunch symbol navigation
and sequential-thinking to follow the chain step by step.

## Tools required

- `jcodemunch` — navigate symbols and references
- `sequential-thinking` — follow the chain step by step

If `jcodemunch` is unavailable: "❌ jcodemunch MCP is not connected. Please check `/mcp`."

## When to use vs b-debug

| | b-trace | b-debug |
|---|---|---|
| Goal | Understand a flow | Fix an error |
| Input | Function / data / feature name | Error message / stack trace |
| Output | Flow diagram (text) | Root cause + fix |

## Steps

### 1. Identify the starting point
- What symbol, data, or event does the user want to trace?
- Determine direction: **forward** (where does X go?) or **backward** (what calls X? where does X come from?)

### 2. Index check
```
list_repos → if not indexed: index_folder with current directory
```

### 3. Trace the chain (jcodemunch)

**Forward trace** (following data/execution forward):
```
get_symbol         → get source of starting point
search_text        → find where the output/return value is used
search_symbols     → find downstream consumers
```

**Backward trace** (finding callers/sources):
```
search_symbols     → find the symbol
search_text        → find all places it is called or referenced
get_symbol         → get source of each caller
```

Repeat until the chain is complete or reaches an entry point (API handler, event listener, user action).

### 4. Reason through the chain (sequential-thinking)
Use `sequential-thinking` to:
- Order the steps in the correct execution sequence
- Identify any branching paths (if/else, error handling)
- Note where data is transformed

### 5. Output flow diagram

```
## Trace: [What was traced]

**Direction:** Forward / Backward
**Starting point:** `SymbolName` in `path/to/file.ts`

**Flow:**
1. `path/to/file.ts` → `FunctionA`
   - [what happens here, what data looks like]
   ↓
2. `path/to/other.ts` → `FunctionB`
   - [transformation or logic]
   ↓
3. `path/to/api.ts` → `handlerC`
   - [final destination or output]

**Branches:**
- If [condition] → [alternate path]

**Summary:**
[1-2 sentences describing the full flow in plain language]
```

## Rules

- Always determine direction (forward/backward) before starting
- Follow the actual code — do not infer or assume paths without verifying with jcodemunch
- If the chain is too long (>8 steps), summarize intermediate steps and note it
- If a branch leads to a dead end or external lib, note it and stop that branch
- Do NOT suggest fixes — this skill is for understanding only