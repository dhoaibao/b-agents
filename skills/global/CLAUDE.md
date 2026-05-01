# b-skills — Claude Code Global Rules

> Short rules enforced every turn.

---

## Tool Priority — MANDATORY

When an MCP is connected, use it before native fallbacks.

- Before symbol-aware work, call `check_onboarding_performed`; if false, call `onboarding` once.
- **Code symbols / structural edits** → `serena:*` first. Prefer: symbol discovery / overview → references → narrow reads → symbol-aware edits.
- Use native `Read` / `Edit` / `Bash` directly only for file listing/discovery, exact-string search, non-code prose, small manifests, or when the user names a small file. Do not bypass Serena for broad code exploration when it can answer.
- **Library / framework / SDK docs** → `context7:*` first. Resolve the library ID before querying docs. If Context7 is unavailable, scrape the official docs; if that fails, use `/b-research`. Never fill library-specific gaps from training knowledge alone.
- **Web search** → `brave-search` first; fall back to `firecrawl_search`, then `WebFetch` only as a last resort.
- **Known URLs / page extraction** → `firecrawl_scrape` first. If scrape misses JS-rendered content, use `firecrawl_map` before broader fallback.
- **Complex reasoning** → `sequential-thinking` for multi-hypothesis debugging, architecture, vague decomposition, or real trade-off analysis. If unavailable, use numbered hypotheses with evidence and confirmed/rejected status.
- If a required MCP is unavailable, say so explicitly and follow the skill's documented fallback. If the skill says graceful degradation is not possible, stop and tell the user to check `/mcp` instead of silently switching strategies.

---

## Coding Principles

- **Think before coding**: state assumptions, surface trade-offs, and ask when unclear. If multiple interpretations exist, present them instead of picking silently. If a simpler approach exists, say so.
- **Keep solutions minimal**: add only what was asked. No speculative features, no single-use abstractions, no unrequested configurability, and no impossible-case handling.
- **Make surgical changes**: touch only what is needed, match the existing style, and do not clean up unrelated code, comments, or formatting. Remove only imports, variables, or functions that your change made unused.
- **Define success before acting**: turn tasks into verifiable goals and state a brief step → verify plan for multi-step work. Do not stop at "implemented"; stop at verified.

---

## Session Hygiene

- After compaction, re-read the active plan if one exists, re-check Serena onboarding if project context seems lost, and prefer focused reads and diff inspection over pasting large files into chat.

---

## Git Safety

Never run autonomously: `git push`, `git pull`, `git commit`, `git reset --hard`, `git revert`, `git clean -f`, `git branch -D`.

Never auto-rollback with `git checkout -- .`; offer it to the user instead.

---

## Sensitive File Safety

Never read, search, print, diff, edit, upload, summarize, or commit files that likely contain secrets without explicit user permission.

Treat at least these as sensitive:
- `.env*`, `*.env`, `.envrc`, `.npmrc`, `.pypirc`, `.netrc`
- `credentials.json`, `settings.local.json`, `secrets.yml`, `secrets.yaml`, `*.tfvars`, `terraform.tfstate*`
- private keys and cert material: `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa`, `id_ed25519`, `.ssh/*`, `.gnupg/*`
- cloud / cluster / deploy auth: `.aws/*`, `.config/gcloud/*`, `kubeconfig`, `.kube/config`
- any file whose name suggests secrets, tokens, credentials, private keys, or service-account data

Do not recursively grep, glob, or scan inside sensitive locations without explicit user permission.

If unsure whether a file is sensitive, stop and ask first.
