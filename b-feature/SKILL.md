---
name: b-feature
description: >
  Full-cycle feature development orchestrator. Runs the complete workflow:
  plan → fetch docs → research (if needed) → implement → self-review.
  ALWAYS use this skill when the user prefixes their request with "b-feature:",
  or describes building something non-trivial from scratch, integrating a new
  service, implementing a significant new capability, or making changes that
  span multiple files and layers. Use when you want the full pipeline run
  automatically without manually invoking each skill. Best triggered explicitly:
  "b-feature: [description]" to guarantee activation.
---

# b-feature

Orchestrator for complete feature development. Chains b-plan → b-docs → b-research
→ implement → b-analyze in a single workflow, skipping steps that don't apply.
Each step gates the next — never skip ahead.

## When to use

- Building a new feature from scratch
- Integrating a third-party service or SDK
- Changes that span multiple files, layers, or services
- Any task where you'd normally invoke 2+ skills manually
- Explicitly triggered with: `b-feature: [description]`

**Not needed for**: bug fixes (use `b-debug`), quick edits to a single function,
or one-off questions. Those are faster without the full pipeline.

## Skills required

This skill orchestrates the full b-skill suite. All must be available:

| Skill | MCP | Role |
|---|---|---|
| `b-plan` | sequential-thinking | Decompose and sequence the work |
| `b-docs` | context7 | Fetch live library/SDK docs |
| `b-research` | brave-search + firecrawl | Research tools, patterns, comparisons |
| `b-analyze` | jcodemunch | Understand existing code + self-review |

If a required MCP is missing, note it and skip that step — do not abort the entire pipeline.

---

## Pipeline

```
[Input] → PLAN → UNDERSTAND → GATHER → IMPLEMENT → REVIEW → [Output]
```

Each phase maps to one or more skills. Phases run in order. Some are conditional.

---

### Phase 1 — PLAN `(b-plan)`

Always run. No exceptions.

Invoke `b-plan` to decompose the feature into ordered steps, surface dependencies,
and identify unknowns. Do not proceed to Phase 2 until the plan is confirmed.

**Gate**: user confirms the plan, or explicitly says "proceed" / "looks good".
If the plan reveals the task is actually a bug fix → stop and switch to `b-debug`.

---

### Phase 2 — UNDERSTAND `(b-analyze)` *(conditional)*

Run if: the feature modifies or extends existing code.
Skip if: this is purely greenfield with no existing code to understand.

Invoke `b-analyze` on the relevant existing modules before writing any new code.
Goal: understand the current structure so the new code integrates cleanly,
follows existing patterns, and doesn't break hidden dependencies.

**Output fed into Phase 4**: findings from b-analyze inform implementation decisions.

---

### Phase 3 — GATHER *(conditional)*

Run the appropriate sub-skill based on what the plan flagged as unknowns:

**3a — `b-docs`** *(run if any library/SDK is involved)*
Fetch live docs for every external library the feature will use.
Do not implement library calls without this step.

**3b — `b-research`** *(run if plan flagged a tool/approach decision)*
Research open questions: which library to choose, best practices, known pitfalls.
Only run if genuinely needed — don't research things already known.

Both 3a and 3b can run in the same phase. 3a is far more common.

---

### Phase 4 — IMPLEMENT

Now write the code, informed by:
- The ordered steps from Phase 1 (b-plan)
- The existing structure from Phase 2 (b-analyze)
- The accurate API surface from Phase 3 (b-docs / b-research)

Execute plan steps one at a time. After each step:
- Confirm the step is complete and matches the "done when" criteria from the plan
- If something unexpected surfaces → pause, re-evaluate remaining steps, continue

Do not implement all steps in one pass without checkpoints.

---

### Phase 5 — REVIEW `(b-analyze)`

Always run. After implementation is complete, invoke `b-analyze` on the newly
written code as a self-review:

- Does the new code follow the patterns found in Phase 2?
- Any complexity hotspots introduced?
- Any duplication with existing code?
- Anything that should be flagged before shipping?

If findings are 🔴 High → fix before presenting to user.
If findings are 🟡 Medium or 🟢 Low → present alongside the implementation as known follow-ups.

---

## Output format

Present a progress header at each phase transition so the user can follow along:

```
── b-feature: [task name] ──────────────────

▶ Phase 1 — Plan
[b-plan output]

▶ Phase 2 — Understand existing code      [SKIP if greenfield]
[b-analyze findings — brief summary]

▶ Phase 3 — Gather
  3a. Docs: [library names fetched]        [SKIP if no libraries]
  3b. Research: [topic]                    [SKIP if not needed]

▶ Phase 4 — Implement
[code, step by step]

▶ Phase 5 — Self-review
[b-analyze findings on new code]
[🔴 fixed before presenting / 🟡🟢 noted as follow-ups]

── Done ─────────────────────────────────
```

---

## Rules

- **Never skip Phase 1** — a plan, even a short one, prevents wrong-direction work
- **Never skip Phase 5** — shipping unreviewed code defeats the purpose of the pipeline
- Phase order is fixed: Plan → Understand → Gather → Implement → Review. Never reorder.
- If a MCP tool fails mid-pipeline, note it, skip that phase, continue — do not abort
- Keep phase summaries concise — the user wants progress, not a wall of text between phases
- If the task grows significantly during implementation, pause and revise the plan before continuing
- b-feature is for complex tasks. If the task turns out to be simple (one file, one function), say so and offer to proceed without the full pipeline.