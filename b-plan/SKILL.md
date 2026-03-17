---
name: b-plan
description: >
  Decompose any non-trivial task into ordered steps, dependencies, and risks before
  writing any code. ALWAYS use this skill when the user says "plan", "lên kế hoạch",
  "thiết kế", "design", "how should I approach", "nên làm thế nào", "before I implement",
  "trước khi code", or describes a task that touches more than 2 files or involves
  multiple moving parts. Also use when the user is about to implement something complex
  and hasn't explicitly asked for a plan — a plan first saves more time than it costs.
  Never jump straight to implementation on complex tasks without running this skill first.
---

# b-plan

Think before coding. Use sequential-thinking to decompose tasks into ordered steps,
surface dependencies, identify risks, and produce a clear execution plan — before any
implementation begins.

## When to use

- Task involves more than 2 files or multiple layers (API, DB, service, UI)
- Task has unclear scope or multiple valid approaches
- User is about to implement something non-trivial and hasn't thought through the order
- Refactoring, architecture changes, or new feature integration
- User says: "plan", "thiết kế", "how should I approach X", "lên kế hoạch", "nên bắt đầu từ đâu"

## Tools required

- `sequential_thinking` — from `sequential-thinking` MCP server

If sequential-thinking is unavailable: reason through the plan inline step by step,
making the thinking explicit in the response. Do not skip planning — just do it without the tool.

---

## Steps

### Step 1 — Clarify scope

Before planning, confirm:

- **What is the end state?** What does "done" look like exactly?
- **What already exists?** Is this greenfield or modifying existing code?
- **What are the constraints?** Deadlines, must-not-break areas, tech stack limits?

If the task description is ambiguous, ask one focused question to clarify — not multiple.
Once scope is clear, proceed.

---

### Step 2 — Decompose with sequential-thinking

Use `sequential-thinking` to break the task into atomic steps:

- Each step should be independently executable and verifiable
- Steps should be ordered by dependency — a step cannot start until its prerequisites are done
- Aim for 4–8 steps for most tasks; more than 10 suggests the task should be split into sub-tasks
- Each step should answer: *what to do*, *why at this point*, and *how to verify it's done*

Think through:
- **Happy path**: the sequence of steps assuming everything works
- **Dependencies**: which steps block others, which can run in parallel
- **Risks**: where things are most likely to go wrong, and what the fallback is

---

### Step 3 — Identify unknowns

Flag anything that needs to be resolved before or during execution:

- **Docs needed**: library/API behavior that needs verification → mark as `b-docs` call
- **Research needed**: tool or approach comparison → mark as `b-research` call
- **Decisions needed**: choices that depend on user preference or business logic
- **Assumptions**: things the plan assumes to be true — state them explicitly

An unresolved unknown is a risk. Surface it now, not halfway through implementation.

---

### Step 4 — Output the plan

Present the plan clearly. Get confirmation from the user before proceeding to implementation.

If the user says "looks good" or equivalent → begin execution step by step, checking off
each step as it's completed.

If the user requests changes → revise the plan before starting.

---

## Output format

```
### Plan: [task name]

**Scope**: [one sentence — what this plan covers]
**End state**: [what "done" looks like]

**Steps**

1. [Step name]
   - What: ...
   - Why now: ...
   - Done when: ...

2. [Step name]
   - What: ...
   - Why now: ...
   - Done when: ...

...

**Dependencies**
- Step 3 requires Step 1 to be complete
- Steps 4 and 5 can run in parallel
- ...

**Risks**
- [Risk]: [mitigation or fallback]
- ...

**Unknowns** *(resolve before starting)*
- Need b-docs: [library] — [what to verify]
- Need decision: [question for user]
- Assuming: [assumption that may not hold]

---
Ready to proceed? Or would you like to adjust anything?
```

---

## Rules

- Never start implementing before the plan is confirmed by the user
- Steps must be ordered by dependency — wrong order causes cascading failures
- Keep steps atomic — one clear action per step, not "implement the whole service layer"
- If a step requires a b-docs or b-research call, mark it explicitly — don't fold it silently into implementation
- Surface risks and assumptions proactively — a wrong assumption found at Step 1 is free; found at Step 7 it costs a rewrite
- If the task turns out to require 10+ steps, split it into phases and plan one phase at a time
- After execution begins, check off completed steps and re-evaluate remaining steps if something unexpected happened