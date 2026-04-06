# b-agents — Agent reference

Detailed contract reference for the b-agents suite. For install and overview, see [README.md](README.md).

## Agent reference

### b-plan

Plans non-trivial work before coding. Produces ordered steps, dependencies, risks, unknowns, and execution-ready handoff in `.opencode/b-plans/*.md`.

**Core behavior**
- Uses `sequential-thinking` to decompose work and surface risks.
- For existing-code tasks, uses jcodemunch preflight plus targeted structure reads so plans reference real files/symbols.
- Includes conditional **Step 0 feasibility gate** only when scope/decision is still uncertain.
- Adds architecture, impact, and deploy-safety checkpoints when relevant.

**Good triggers**
```text
b-plan: add rate limiting to the API
plan: design the notification system
how should I approach refactoring the auth module?
```

**Output**
- Writes a plan file to `.opencode/b-plans/[task-slug].md` in the current project root.
- Includes ordered checkbox steps, risks, unknowns, optional `## Feasibility`, plus `## Last Gate Failure` and `## Review Feedback` sections for execution.
- Plan files are always in English.

**Key rules**
- Never execute in the same session as planning.
- Step 0 only confirms feasibility / blockers / success criteria; it does not replace `b-docs` or `b-research`.
- If an issue/ticket is provided, write `**Issue**: ...` in the plan header for downstream `b-review` enrichment.

---

### b-execute-plan

Primary orchestrator for the execution pipeline: conditional `b-analyze` Step 0 → `b-tdd` → `b-gate` → `b-review` → `b-commit`.

**Core behavior**
- Reads `.opencode/b-plans/*.md`, parses checkbox state, resumes automatically from first pending step.
- Uses `@b-tdd` as default route for implementation-like steps, including common `fix` wording.
- Auto-advances on success; pauses only on failure, truly vague routing, manual steps, or `NEEDS FIXES`.
- Writes `## Context`, `## Last Gate Failure`, and `## Review Feedback` into the plan file to bridge state between subagents.

**Step 0 policy**
- Greenfield plans skip Step 0.
- Existing-code plans ask first and run `@b-analyze` only when scope is ambiguous, unfamiliar, multi-file/multi-layer, high-blast-radius, or `## Context` is missing/stale/mismatched.
- Small, local, well-scoped changes may skip Step 0.

**Good triggers**
```text
execute plan
run plan: implement-b-execute-plan.md
/b-execute-plan
```

**Output**
- Progress view with completed/pending steps and current routed subagent.
- On final success, suggests next actions with explicit agent names.

**Key rules**
- Never auto-invoke `@b-analyze`; Step 0 is ask-first.
- `@b-gate` failure shortcut may offer `@b-debug`.
- `@b-observe` is optional follow-up only when `@b-review` explicitly surfaces observability uncertainty.
- Rollback is offered to the user; never auto-executed.

---

### b-docs

Fetches live, version-accurate library documentation before writing integration code.

**Core behavior**
- Uses Context7 first for API signatures, config, auth, and deprecations.
- Detects project version from manifests/lockfiles when possible.
- If Context7 has no index, applies a scope gate:
  - simple lookup + known docs URL → direct `firecrawl_scrape`
  - complex query or insufficient scrape → actively invoke `b-research`
  - simple lookup + no obvious official docs URL → stop instead of over-escalating

**Good triggers**
```text
b-docs: sendgrid send email with attachments
how do I configure retries in bullmq?
tra cứu cách dùng thư viện Prisma
```

**Output**
- Accurate current-version docs summary for the requested API area.

**Key rule**
- Never rely on training memory for library APIs when Context7 is available.

---

### b-research

Deep multi-source research with search, scrape, filtering, and synthesis.

**Core behavior**
- Routes NEWS queries to `brave_news_search`; other queries to `brave_web_search`; HOWTO/API queries hit Context7 first.
- Picks only the best sources, not the most sources.
- Uses strict pre-scrape and post-scrape filtering; stops if evidence is too weak.
- For docs sites, can use `firecrawl_crawl` for multi-page coverage.
- Uses a single Explore subagent for context isolation when **≥4 URLs** need scraping.

**Good triggers**
```text
b-research: compare bullmq vs bee-queue for job queues
research: best practices for webhook signature verification
deep dive into Redis Streams
```

**Output**
- Structured report with summary, findings, optional comparison, and cited sources.

**Key limits**
- Default scrape cap: 3 URLs per session; 5 for COMPARE queries.
- Never fill factual gaps from training data when sources do not support them.

---

### b-tdd

Implements code through Red-Green-Refactor discipline.

**Core behavior**
- Detects the test stack automatically.
- Enforces the Iron Law: no production code before a failing test.
- Supports:
  - `[plan-file]:[N]` → single-step mode
  - `[plan-file]` → first pending step
  - no args → iterate-all mode
- Refreshes index after refactor when jcodemunch is available.

**Good triggers**
```text
b-tdd
test first
viết test trước
```

**Output**
- RGR checkpoint log.
- Single-step completion in plan mode; full handoff to `b-gate` in iterate-all mode.

**Key rules**
- Do not mark plan checkboxes in single-step mode; `b-execute-plan` owns them.
- Do not silently ignore regressions.

---

### b-gate

Runs the mandatory quality gate after implementation.

**Core behavior**
- Ordered checks: lint → typecheck → tests → coverage → security → clean-code → integration/e2e (soft block).
- Stops on hard failures; warns on soft failures.
- Runs only checks supported by the detected stack/config.

**Good triggers**
```text
b-gate
check quality
kiểm tra chất lượng
```

**Output**
- Gate report with ✅ / ❌ / ⚠️ per check and a specific failing action.

**Key rules**
- Never pass with unresolved hard failures.
- Re-run the failed check before running the full gate again.

---

### b-review

Human-judgment pre-PR review: correctness, requirements, edge cases, tests, and minimum observability on new entry points.

**Core behavior**
- Reads git diff and builds a requirements baseline from plan file, `$ARGUMENTS`, or user clarification.
- Uses jcodemunch to prioritize review depth by changed symbols and blast radius.
- Always checks **injection vectors**, even on very small diffs.
- Runs a conditional observability check only for newly added endpoints/handlers/jobs/consumers.

**Small-change fast path**
- If diff is ≤50 lines AND ≤2 files:
  - accept any non-empty requirements baseline
  - skip vague-response enforcement
  - skip observability check
  - skip expanded security checklist
  - still check **injection vectors**

**Good triggers**
```text
b-review
review before PR
kiểm tra logic trước khi push
```

**Output**
- Logic findings
- Requirements coverage table
- Edge-case / test adequacy findings
- Reviewer question
- `READY FOR PR` or `NEEDS FIXES`

**Handoff**
- READY FOR PR → run `b-commit`
- If observability uncertainty remains on new entry points/jobs → suggest `b-observe: [scope]`
- NEEDS FIXES → fix blockers, re-run `b-gate` if code changed, then `b-review` again

---

### b-commit

Generates commit message and PR description text only.

**Core behavior**
- Reads diff and recent context.
- Uses conventional-commit style subject.
- Detects mixed-concern diffs and outputs split suggestions instead of one misleading unified message.

**Good triggers**
```text
b-commit
viết commit message
PR description
```

**Output**
- Ready-to-copy commit message block
- Ready-to-copy PR description block

**Key rule**
- Never stages, commits, pushes, or creates PRs.

---

### b-analyze

Deep structural code analysis before modification or as a standalone review.

**Core behavior**
- Uses jcodemunch to map repo structure, dependencies, complexity, duplication, dead code, hotspots, and OOP hierarchy.
- Deep mode produces prioritized findings; quick mode produces structure only.
- Uses `sequentialthinking` in deep mode to rank the highest-ROI actions.

**Modes**
- `quick` → structure map only
- `deep` / no args → full analysis

**Good triggers**
```text
b-analyze: review the service layer before refactoring
analyze this file for code smells
```

**Output**
- Structure overview
- Severity-ranked findings
- Metrics snapshot
- Recommended next steps

**Handoff**
- If a bug is uncovered rather than a design/code-quality issue, recommend `b-debug`.
- If refactor work is needed, recommend `b-plan`.

---

### b-observe

Static observability audit for logging, metrics, tracing, and swallowed errors.

**Core behavior**
- Uses jcodemunch to find instrumentation libraries, map instrumented vs uninstrumented files, inspect error handlers, and enumerate service boundaries.
- Uses `sequentialthinking` to rank remediation by incident impact.
- Audits statically only; no runtime profiling or APM setup.

**Good triggers**
```text
b-observe: the payment service
observability audit
find missing logs in background workers
tracing coverage audit
```

**Output**
- Missing logs / metrics / tracing findings
- Metrics snapshot
- Ordered remediation list

**Distinctions**
- vs `b-analyze`: observability completeness, not structural quality
- vs `b-debug`: prevents silent failures; does not trace an active bug

---

### b-debug

Hypothesis-driven debugging with full-loop execution by default.

**Core behavior**
- Uses jcodemunch to map execution path, references, blast radius, and suspicious symbols.
- Uses `sequential-thinking` to rank hypotheses.
- If static evidence is insufficient, can run a bounded dynamic verification loop with temporary targeted logs.
- For library-related issues, looks up external evidence and verifies API behavior through docs.

**Default contract**
- `trace → confirm root cause → fix → verify`
- Diagnosis-only is allowed only when the caller explicitly requests it or a safe minimal fix is not possible from available evidence.

**Good triggers**
```text
b-debug: webhook not triggering despite correct URL registration
b-debug: intermittent 500 on /api/send with no error in logs
why is this callback not running?
```

**Output**
- Symptoms summary
- Execution path map
- Ranked hypotheses
- Confirmed root cause
- Minimal fix
- Verification result or exact verification steps

**Key rules**
- Never patch before root cause is explicitly confirmed.
- After fixing, refresh changed-file index when jcodemunch is available.

---

## Usage patterns

### Standard feature flow
```text
1. b-plan: [task]
2. b-docs: [library]              (if needed)
3. b-tdd
4. b-gate
5. b-review: [task]
6. b-commit: [task]
```

If existing code must be understood first, use `b-analyze` directly or let `b-execute-plan` decide Step 0 conditionally.

### Debug flow
```text
b-debug: [symptom + expected behavior]
```

### Before touching unfamiliar code
```text
b-analyze: [module or layer]
```

### Library choice / comparison
```text
b-research: compare [A] vs [B] for [use case]
```

### Known library, API uncertain
```text
b-docs: [library] — [feature]
```

### Full manual pipeline
```text
1. b-plan: [task]
2. b-analyze: [module]            (optional, or let b-execute-plan decide Step 0 later)
3. b-docs: [library]              (if needed)
4. b-tdd
5. b-gate
6. b-review: [task]
7. b-commit: [task]
```

---

## Trigger tips

- Prefix with agent name: `b-plan: ...`, `b-debug: ...`, `b-review`, etc.
- Use explicit intent words: `plan`, `debug`, `analyze`, `review`, `gate`, `research`.
- Mention complexity when relevant: multi-file, unfamiliar module, unclear root cause, integration, shared module.

---

## Agent interaction map

```text
b-plan ───────────────► plan file in .opencode/b-plans/
       ├─ unknown library/API ─────────────► b-docs
       └─ research / compare decision ─────► b-research

b-execute-plan ───────► Step 0 (conditional, ask-first) ─► b-analyze
                 └────► implementation step routing ─────► b-tdd / b-gate / b-review / b-commit / manual

b-gate ───────────────► pass ────────────────────────────► b-review
       └──────────────► fail shortcut ───────────────────► b-debug (optional)

b-review ─────────────► READY FOR PR ────────────────────► b-commit
         └────────────► observability uncertainty ───────► b-observe (optional follow-up)

b-analyze ────────────► bug found ───────────────────────► b-debug
         └────────────► refactor recommended ────────────► b-plan

b-observe ────────────► remediation needed ──────────────► b-plan
```

---

## Personal / daily agent reference
