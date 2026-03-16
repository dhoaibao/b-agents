---
name: b-plan
description: >
  Create a detailed implementation plan for a feature or task, scan the codebase
  for context, and write the result to .claude/b-plan/PLAN.md.
  ALWAYS use this skill when the user says "lên plan", "plan for...", "tôi muốn build...",
  "implement feature...", "tạo plan cho...", "how should I implement...", or any request
  that involves planning before coding. Trigger even for medium-sized tasks where
  understanding the existing codebase is important.
---

# b-plan

Creates a detailed implementation plan by combining codebase scanning (jcodemunch),
library docs (context7), and structured reasoning (sequential-thinking).
Writes the result to `.claude/b-plan/PLAN.md` for Claude Code to reference during implementation.

## Tools required

- `jcodemunch` — scan codebase structure and symbols
- `context7` — fetch up-to-date library docs
- `sequential-thinking` — structured step-by-step reasoning

If `jcodemunch` is unavailable: "❌ jcodemunch MCP is not connected. Please check `/mcp`."

## Steps

### 1. Understand the task
- Clarify what needs to be built if the request is ambiguous
- Identify the libraries/frameworks involved

### 2. Scan the codebase (jcodemunch)
Run in this order — stop when you have enough context:

```
1. list_repos          → check if project is indexed
   - if not indexed: index_folder with current directory first
2. get_repo_outline    → understand overall structure
3. get_file_tree       → find relevant directories
4. search_symbols      → find existing patterns related to the task
5. get_symbol          → get full source of key symbols (only if needed)
```

Focus on:
- Existing patterns similar to what needs to be built
- Naming conventions (files, functions, classes, variables)
- Related files that will be affected
- Potential conflicts or duplicates

### 3. Fetch library docs (context7)
- Identify libraries the implementation will use
- Fetch relevant docs sections for APIs that will be called
- Note the correct version-specific usage

### 4. Plan with sequential-thinking
Use `sequential-thinking` to reason through:
- Break the task into ordered implementation steps
- Estimate complexity per step (S/M/L)
- Identify risks and dependencies between steps
- Determine what to build first

### 5. Write PLAN.md

Create `.claude/b-plan/PLAN.md` with the following structure:

```markdown
# Plan: [Task Name]
_Generated: [date]_

## Objective
[1-2 sentences describing what will be built and why]

## Codebase Context
### Relevant Files
- `path/to/file.ts` — [what it does, why it's relevant]
- ...

### Existing Patterns to Follow
- [pattern name]: [brief description + example file]
- ...

### Naming Conventions
- Files: [e.g. kebab-case]
- Functions: [e.g. camelCase]
- Components: [e.g. PascalCase]

### Potential Conflicts
- [file or symbol that might conflict, and why]

## Architecture
[Text description of how the feature fits into the existing system]
[Include data flow if relevant]

## Implementation Steps
- [ ] Step 1: [description] — complexity: S/M/L
- [ ] Step 2: [description] — complexity: S/M/L
- [ ] Step 3: [description] — complexity: S/M/L
...

## Library APIs to Use
- `[lib]`: [specific function/method] — [what it does]
- ...

## Risks & Notes
- [risk or important note]
- ...
```

## Rules

- Always index the project first if not already indexed in jcodemunch
- Scan codebase BEFORE writing any plan — never plan without context
- Keep PLAN.md concise — implementation steps should be actionable, not vague
- If `.claude/b-plan/` directory doesn't exist, create it before writing the file
- After writing PLAN.md, tell the user: "✅ Plan written to `.claude/b-plan/PLAN.md`. Ready to implement."
- Do NOT start implementing — planning and implementing are separate steps