---
name: b-plan
description: >
  Think before coding. Decompose non-trivial tasks into ordered steps, evaluate approaches,
  surface risks, and produce an execution-ready plan file. ALWAYS invoke when the user says
  "plan", "thiết kế", "how should I approach", "lên kế hoạch", "nên bắt đầu từ đâu",
  or the task spans more than 2 files or has unclear scope.
  Unlike b-debug (fix broken) or b-research (lookup info), b-plan owns the decision of
  what to build and in what order.
effort: high
---

# b-plan

$ARGUMENTS

Think before coding. Lock scope, evaluate approaches, decompose into ordered steps,
surface risks and unknowns, then produce a clear plan file before any implementation.

If `$ARGUMENTS` is provided, treat it as the task description — skip asking "what do you want to build?" in Step 1 and proceed directly with the stated task. Ask only for missing context (constraints, greenfield vs existing, issue URL).

## When to use

- Task involves more than 2 files or multiple layers (API, DB, service, UI).
- Task has unclear scope or multiple valid approaches — need a decision.
- User is about to implement something non-trivial and hasn't thought through the order.
- Refactoring, architecture changes, or new feature integration.
- User says: "plan", "thiết kế", "how should I approach X", "lên kế hoạch", "nên bắt đầu từ đâu".

## When NOT to use

- Simple single-file edit or ≤2-step task → do it directly.
- Something is broken → use **b-debug**.
- Quick fact or library lookup → use **b-research**.

## Tools required

- `sequentialthinking` — from `sequential-thinking` MCP server (required for Steps 3–4: approach evaluation and decomposition).
- `activate_project`, `check_onboarding_performed`, `onboarding`, `find_file`, `list_dir`, `find_symbol`, `get_symbols_overview`, `find_referencing_symbols`, `search_for_pattern`, `read_file`, `rename_symbol` — from `serena` MCP server *(required for modify-existing-code tasks; optional for pure greenfield)*.
- `resolve-library-id`, `query-docs` — from `context7` MCP server *(optional, for inline library verification in Step 5 — simple lookups only)*.
- `brave_web_search` — from `brave-search` MCP server *(optional, for tool/approach comparison in Step 5 — simple lookups only)*.
- `firecrawl_scrape` — from `firecrawl` MCP server *(optional, for scraping Issue/ticket URL in Step 1)*.

If sequential-thinking is unavailable: reason through plans and trade-offs inline with explicit numbered steps. Format fallback as: `Goal → Constraints → Options → Decision → Ordered steps → Open questions`.
If Serena is unavailable: use Glob/Read to inspect key files. Note: "⚠️ Serena unavailable — cross-file tracking incomplete."
If context7 or brave-search is unavailable: delegate to /b-research.
If firecrawl is unavailable: store the Issue URL as a plain reference without scraping.

Graceful degradation: ✅ Possible — core planning works without MCPs using inline reasoning and Glob/Read.

## Steps

### Step 1 — Scope lock

Confirm what is being built before scanning any code.

**If the task is clearly scoped** (user already described the full feature, no ambiguity):
- Restate the scope in one sentence and ask the user to confirm.
- If confirmed, move directly to Issue URL and greenfield/existing check below.

**If the task has unclear scope or the user hasn't fully thought it through**:
- Ask the three scope questions:
  - **What is the end state?** What does "done" look like exactly?
  - **What are the hard constraints?** Performance, compatibility, deadlines, must-not-break areas.
  - **What does success look like?** 2–4 concrete, verifiable criteria.
- Ask once. If still unclear, ask one focused follow-up. Don't loop.

**Unknown-ask rule** *(enforced throughout all steps)*: Any requirement or decision that cannot be determined from the task description or the codebase — e.g. behavioral choice, priority, integration contract, naming convention — must be asked to the user immediately. Never self-infer or assume. Surface unknowns as they are discovered; batch them per step if multiple arise at once.

**Decision accumulation** *(running record throughout all steps)*: Each time a user answer, a codebase finding, or an approach choice settles a behavioral or design question, immediately record it as a numbered confirmed decision. These compile into `## Confirmed decisions` in the plan. Format each entry as a single, unambiguous, implementation-actionable statement — no hedging, no "consider", no "may". Example: `"Realtime update must update the existing VoiceCall matched by VendorCallKey; if soft-deleted, insert a new row instead."`

**Feasibility check** *(run inline when scope is non-trivial — not a separate step)*:
- Does the current architecture support this? Use Serena symbol/file discovery (`list_dir`, `find_file`, `find_symbol`, `find_referencing_symbols`) or Glob/Read if unavailable.
- Any blockers? (Missing infrastructure, incompatible dependencies, architectural gaps.)
- Effort estimate: S (hours) / M (1–2 days) / L (3–5 days) / XL (1–2 weeks) / XXL (weeks+).
- If blockers found: state clearly. If no workaround exists, do not proceed until resolved.
- If XL–XXL AND unfamiliar pattern or unverified library: stop and run /b-research first.

**Issue/ticket** *(optional)*:
- Ask once: "Issue/ticket URL or ID? (Leave blank to skip.)"
- If a URL is provided: call `firecrawl_scrape` with `formats: ["markdown"], onlyMainContent: true`. Trim to 800 words and use as **requirements context** for Steps 3–5. If scrape returns <200 characters or 403: store the URL as a plain reference.
- If a ticket ID (not a URL): store as-is; no fetch.

**Greenfield vs existing**:
- Is this a new module/service, or modifying existing code?
- If existing code → proceed to Step 2. If greenfield → skip Step 2.

---

### Step 2 — Scan existing code *(existing-code tasks only)*

Use Serena to understand what already exists before planning. Follow this exact order — do not skip to `read_file`:

1. **Activate project** — call `activate_project`. If `check_onboarding_performed` returns false, call `onboarding` first.
2. **Discover symbols** — call `find_symbol` on the main symbol or module name involved in the change. Use `search_for_pattern` for exact strings (config keys, error messages, repeated patterns).
3. **Inspect structure** — call `get_symbols_overview` on each relevant file to see which symbols are worth reading.
4. **Trace references** — call `find_referencing_symbols` on the key symbol(s) to confirm which callers and dependents are affected.
5. **Read narrowly** — only if the above still leaves ambiguity: call `read_file` on the exact symbol body or file section on the proposed execution path.

**Goal**: reference real paths and symbols. A plan that references wrong file names or non-existent functions fails at execution. Never paste full file contents into the plan — only the names and line references that matter.

---

### Step 3 — Evaluate approaches *(conditional)*

Run if the task has a structural decision: new module vs extending existing, sync vs async, REST vs event-driven, library A vs B.

1. List 2–3 viable approaches with key trade-offs (complexity, performance, coupling, reversibility).
2. Use `sequentialthinking` to evaluate them systematically against the current constraints.
3. Make the reasoning useful for execution: return the chosen approach, alternatives rejected, the assumption that could flip the decision, and the first implementation step.
4. Pick one and document in `## Decision` (see plan file format below).
5. Add the approach choice and all structural trade-offs settled here to the running confirmed decisions list.

Skip this step if the approach is already obvious or decided — do not invent choices where there are none.

---

### Step 4 — Decompose

Use `sequentialthinking` to break the chosen approach into atomic, ordered steps:

- Each step: independently executable, independently verifiable.
- Ordered by dependency — not by what's easiest.
- Usually 4–8 steps. Split into phases if >10.
- Each step answers: *what*, *why now*, *done when* — and must include:
  - **Exact file paths** and **symbol names** involved (e.g. `src/auth/middleware.ts:validateToken()`).
  - **Current state** of anything being changed (what exists today, what interface/behavior will change).
  - **Concrete done-when** that a fresh agent can verify independently (test command, observable output, specific assertion).
  - Any **API signatures**, **config keys**, or **contract details** needed to implement without further lookup.
  - **`Exact [X]:`** sub-bullets for any implementation choices that must be locked in to prevent implementor drift — e.g. `Exact insertion points:`, `Exact helper responsibilities:`, `Exact fields to create:`, `Concrete implementation choice:`, `Build rules:`. Add these whenever the step would otherwise leave a structural decision open.
- **Handoff standard: 90%+** — if a fresh agent with zero prior context would need to ask a follow-up question to implement the step, the step is not detailed enough. Add the missing detail now.
- Ask for output in this shape: `Goal`, `Constraints`, `Ordered steps`, `Dependencies`, `Open questions`, `First action`.

**Impact checkpoint** *(modify-existing-code only)*:
- `find_referencing_symbols` on the main symbol/module being changed.
- `rename_symbol` only when the plan explicitly includes a rename of an exported/public symbol; call out broad references as migration risk.
- Wide downstream impact → split into smaller phases or add rollback steps.

**Deploy safety** — annotate any step that matches:
- New routes/endpoints → `⚠️ consider feature flag`
- DB schema changes → `⚠️ deploy order: [before / after] app deploy`
- New external service calls → `⚠️ verify availability in target environment`

**Planned touch points** — after decomposing all steps, compile `## Planned touch points` for the plan file: one bullet per file/class that will change, with the exact path and what is added/changed/removed at method or field level. A fresh agent must be able to read this section and know every artifact to touch before opening a single file.

**Mapping/contract table** *(conditional — only when the task involves field mapping, data transformation, or protocol contracts)*: produce a `## Mapping outline` section listing every source → target mapping with repo field names, types, and any normalization notes. Prevents implementors from guessing names or semantics.

---

### Step 5 — Identify unknowns

Flag anything unresolved before handing off the plan:

- **Docs needed**: library/API behavior not yet verified.
- **Research needed**: tool or approach comparison still open.
- **Decisions needed**: choices that require user input.
- **Assumptions**: things the plan assumes but hasn't confirmed.

**Classify each unknown before acting:**

| Type | Action |
|---|---|
| **User decision** — behavioral choice, priority, integration contract, naming, or anything the codebase can't answer | ⛔ Stop. Ask the user immediately. Do NOT write the plan until resolved. |
| **Tech lookup** — library API behavior, yes/no capability, 2-option comparison | Resolve inline: `query-docs` (context7) or `brave_web_search`. Append `→ Confirmed: [finding]`. |
| **Complex research** — multi-source or open-ended comparison | Delegate to /b-research. Mark as `Unknown — needs /b-research: [topic]`. Do NOT block the plan on this. |

**Clarification gate** — before proceeding to Step 6, batch all outstanding user-decision unknowns into a single message and wait for answers. Only write the plan after every user-decision unknown is resolved. A plan with unresolved user decisions is not a complete plan.

---

### Step 6 — Write plan

Write to `.claude/b-plans/[task-slug].md` in the **current project root only**.

- `task-slug` = kebab-case, e.g. `add-retry-logic`, `refactor-auth-module`.
- Create `.claude/b-plans/` if it doesn't exist.
- Show the exact saved path after writing.

Present a short summary (scope + step count) and ask for confirmation. Update and re-confirm if the user requests changes.

---

## Plan file format

Always English, regardless of the user's query language.

```markdown
# Plan: [task name]

**Scope**: [one sentence]
**End state**: [what "done" looks like]
**Created**: [date]
**Issue**: [URL, ticket ID, or omit entirely]

## Feasibility *(only if assessed in Step 1)*
**Effort**: [S/M/L/XL/XXL]
**Blockers**: [none / description]
**Assumptions confirmed**: [list]

## Decision *(only if multiple approaches were evaluated)*
**Chosen approach**: [what was selected]
**Alternatives rejected**: [option — reason]; [option — reason]
**Why**: [1–2 sentence rationale]

## Confirmed decisions
1. [Unambiguous, implementation-actionable statement of a behavioral/design/product decision made during planning.]
2. ...

## Mapping outline *(only if task involves field mapping, data transformation, or protocol contracts)*
- `[source field / repo property]` → `[target field]` — [normalization notes]
...

## Planned touch points
- `[exact/path/to/File.ext]` — [what is added / changed / removed, at method or field level]
...

---

## Steps

- [ ] 1. [Step name]
  - What: ... *(exact file path + symbol name)*
  - Current state: ... *(what exists today that will change)*
  - Why now: ...
  - Done when: ... *(verifiable by a fresh agent — test command, output, assertion)*
  - Exact [X]: ... *(optional — lock in implementation choices, field lists, insertion points, build rules that must not be left open)*

- [ ] 2. [Step name]
  ...

## Dependencies
- Step 3 requires Step 1 to be complete
- Steps 4 and 5 can run in parallel

## Risks
- [Risk]: [mitigation or fallback]

## Unknowns *(resolve before starting)*
- Need /b-research: [topic] — [what to verify]
- Need decision: [question for user]
- Assuming: [assumption that may not hold]
```

---

## Rules

- Always write to `.claude/b-plans/` — never leave the plan only in chat.
- Always write plan files in English.
- Do not implement in the same session as planning.
- Steps must be ordered by dependency — wrong order causes cascading failures.
- Keep steps atomic — one clear action per step.
- Surface risks and assumptions proactively.
- Split into phases if 10+ steps.
- Never trigger destructive git commands.
- **Never self-infer ambiguous requirements** — if a decision requires user input, ask immediately during planning. A plan built on silent assumptions is not a complete plan.
- **Handoff standard: 90%+** — the final plan must be self-contained enough that a fresh agent with zero prior context can implement every step correctly without asking clarifying questions. Include exact paths, symbol names, current state, and verifiable done-when criteria in every step.
