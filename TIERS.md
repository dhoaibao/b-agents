# Agent Model Tiers

Classification of 11 b-agents into 3 tiers based on task complexity and model capability needed.

---

## Tier 1 — Premium (deep reasoning / critical decisions)

Agents where wrong output causes cascade failures across the pipeline. Low call frequency, high cost acceptable.

| Agent | Why Tier 1 |
|---|---|
| `b-plan` | Wrong decomposition → entire pipeline fails. Needs deep reasoning for feasibility, trade-offs, impact analysis |
| `b-debug` | Wrong root cause → wrong fix → bug persists. Hypothesis ranking needs critical thinking |
| `b-review` | Missed logic bug or security vuln ships to production |
| `b-execute-plan` | Orchestrator — wrong routing cascades. Complex state bridging between subagents |

---

## Tier 2 — Standard (balanced quality + cost)

Agents producing findings or reports, not final decisions. Good quality needed but not critical.

| Agent | Why Tier 2 |
|---|---|
| `b-analyze` | Deep analysis but outputs findings, not changes. Needs large context + pattern recognition |
| `b-research` | Tool-use heavy, output is a reference report |
| `b-tdd` | Coding discipline matters, but tests catch mistakes in the RGR loop |
| `b-observe` | Static audit — produces findings report, not direct fixes |

---

## Tier 3 — Economy (high-volume / simple execution)

Agents that run frequently with simple, deterministic, easy-to-verify tasks. Cost matters most.

| Agent | Why Tier 3 |
|---|---|
| `b-gate` | Pure execution — runs bash commands sequentially, zero reasoning |
| `b-docs` | Simple tool chain: `resolve-library-id` → `query-docs` → extract |
| `b-commit` | Reads diff → formats text from template |

---
