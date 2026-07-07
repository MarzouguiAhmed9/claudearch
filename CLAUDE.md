# CLAUDE.md — Garnet Router

> Read this first. Load ONLY the file your task needs.
> ⚠️ **Active work = `.claude/plan/mission2-k8s.md` (k8s migration). Read it before any task.**
> Current status / what's in progress = `.claude/plan/tasks.md`.

---

## What is Garnet

Privacy-preserving chatbot. Intercepts messages → pseudonymizes PII → sends to LLM → restores real values.

```
User sends:  "Max Mustermann at Enclaive"
LLM sees:    "PERSON_cc75010d at ORGANIZATION_bd3a68a5"
User sees:   "Max Mustermann at Enclaive"   ← restored
```

## Architecture

```
Browser → Caddy (TLS) → Open WebUI (Svelte fork)
                              ↓
                   Privacy Proxy (FastAPI)
                   ↓        ↓        ↓
                Ollama   OpenAI   Anthropic/Groq/Gemini
```

---

## Team

| Person | Role | Style |
|--------|------|-------|
| Ahmed | Intern | Informal, typos OK, overview-first |
| Sebastian (Seb) | Supervisor / owner | Max 3-4 lines, 1 ask, no fluff |
| Ion | Infra / cVM / DNS / Harbor admin | — |
| Nicu | Harbor registry / OCI artifacts | — |
| Ahmed Bouzid | Senior / k8s admin | Cluster admin perms |

Approve-before-acting (Seb): structural proxy changes, new services, `ResponseMessage.svelte`, streaming depseudo, public key distribution, vLLM migration.

---

## Repos & environments

| Repo | Branch | Purpose | Local |
|------|--------|---------|-------|
| `enclaive/garnet` | `garnet-privacy-proxy` | monorepo: proxy + webui + pipeline | `~/Desktop/garnet` |
| `enclaive/garnet` | `feature-button-i` | older OWU feature branch | `~/Desktop/garnet` |
| `enclaive/garnet_helm` | `main` | Helm chart (pending) | TBD |

- **cVM (live):** `root@65.108.38.50`, Fedora, `/opt/garnet/`, Docker Compose
- **k8s (target):** `garnet.enclaive.cloud`, namespace `garnet`, ArgoCD + Helmfile
- **Harbor:** `harbor.enclaive.cloud/garnetdemo/` — `privacy-proxy`, `garnet-webui`

---

## Workflow — SDD

Every feature follows: **Spec → Review → Design → Review → Tasks → Build → Validate.**
Files live in `.claude/features/<YYYY-MM>-<name>/` (01-spec.md … 07-validate.md).

Run `/coach` first for any new task — it reads tasks.md and recommends the right skill path.

| Skill | Trigger | Role |
|-------|---------|------|
| `coach` | `/coach` | Pick skill/agent path for any task. Run FIRST. |
| `data` | `/data`, "add to data", "save this", "ingest ticket/note/doc/image" | Single source of truth for external context (tickets, team notes, docs, images, ideas, config). Writes to `.claude/docs/data.md`. All agents read it. |
| `git` | "/git", "/gitwork", "/previous-step", "/updategithubticket", "commit", "open PR", "create issue", "post progress" | 6 modes: VIEW/SNAPSHOT/COMMIT/PR/ISSUE/POST. Absorbs old gitwork+previous-step+update-github-ticket. Reads tickets from `data.md` 🎫. Uses GitHub MCP + gh CLI. 3-choice confirm on any push. |
| `baby-explain` | "explain easily / baby mode / i don't get X" | Reads existing feature/plan/spec → plain-language → `.claude/explanations/` |
| `update-project` | "update project / sync docs" | Sync CLAUDE.md + docs against reality |
| `improve` | "improve / improve garnet / brilliant ideas" | Godmode audit of **garnet project** (security/perf/UX/DX) → `.claude/docs/improvements/` (Opus + max) |
| `improveclaude` | "improve claude / improve arch / audit .claude" | Godmode audit of **Claude arch** (skills/agents/hooks/MCPs/plugins). Invokes claude-code-setup + WebSearch → `.claude/improve/` |
| `enclaiveask` | `/enclaiveask <question>` | Plain-language answer for teammates |
| `clean-comments` | "clean comments" | Tidy comments.md |
| `readme` | "generate readme" | Write README from real project files |
| `network-map` | "network map / show architecture" | Live ASCII service + data-flow map |
| `enclaivetech` | mentions vHSM/Nitride/Vault/Buckypaper | Enclaive product reference |
| `web` | `/web`, `/fetch`, paste URL, "search for X", "docs of X", "owu docs", "crawl X" | 5 modes: FETCH/SEARCH/CRAWL/DOC/OWU. Uses Firecrawl+Exa+context7 MCPs. Absorbs owudocs. Offers save to `data.md` 📚. |

## Agents (isolated subagent context)

For infra, code review, deep planning — use these instead of skills:

| Agent | Trigger | Role |
|-------|---------|------|
| `planner` | "plan/spec/design/brainstorm X" · "i want to add X" · "task list for X" · "overview X" · "sdd flow" | 8 modes (BRAINSTORM/QUICK/SPEC/DESIGN/TASKS/SDD-FULL/EXTRACT/REVIEW-PLAN). Absorbs build-feature+make-plan+sdd-flow+extract-plan. Writes to `features/<date>-<slug>/`. Uses context7 MCP + superpowers:brainstorming/writing-plans. Hands off to implementer. |
| `implementer` | "implement/code/build X" · "/implementer <feature>" | 7 modes (SETUP/TDD/EXECUTE/PARALLEL/VERIFY/REVIEW/PR). Reads planner's `05-tasks.md`. Mandates superpowers:TDD + verification-before-completion + executing-plans. Uses context7 + code-modernization. Hands off to reviewer then infra. Never deploys. |
| `infra` | ANY infra topic: CI/CD, Docker, k8s, Helm, ArgoCD, Harbor, manifests, deploy, releases, migrations, "review manifests" | 6 modes (ANSWER/BUILD/REVIEW/DIAGNOSE/DEPLOY/MIGRATE). Paste-mode when no live cluster. Uses kubernetes/helm/argocd MCPs. Never deploys without 3-choice confirm. |
| `reviewer` | `/reviewer` · "review" · "vibe check" · "write tests for X" · "debug X" · "security check" · "review PR" · "over-engineered?" | THE CHECK pillar. 7 modes: FULL (5-phase full scan + specialists), VIBE (44 vibe-code defects), TEST (per-feature test gen with python-testing MCP), DEBUG (root cause), SECURITY (semgrep MCP + /security-review), PR (gh + /review), COMPLEXITY (ponytail-audit). |
| `council` | `/council <topic>` | 3 lenses (Dev / Security / Logic) → synthesis DO/DON'T/DEFER |
| `creative` | `/creative` | Senior creative engineer, ranked ideas across 10 dimensions |
| `logic` | `/logic` | Brainstorm all scenarios, check coverage vs code |
| `ahmedclaudeskill` | "use ahmedclaudeskill" / "bootstrap claude arch" | Copies canonical .claude/ live into new project |

## Load the right file

| Task | File |
|------|------|
| **External context (tickets, team notes, docs, images, ideas)** | **`.claude/docs/data.md` ← always-relevant, all agents read this** |
| **k8s migration / Mission 2** | **`.claude/plan/mission2-k8s.md` ← read first** |
| Current work / in progress | `.claude/plan/tasks.md` |
| Proxy / pseudonymizer / FastAPI / main.py | `.claude/docs/proxy.md` |
| Svelte / WebUI / frontend | `.claude/docs/webui.md` |
| Deploy / push images / CI-CD / cVM commands | `.claude/docs/deploy.md` |
| Logs / something broken (read-only diagnosis) | `.claude/docs/debug.md` |
| Security issues / hardening | `.claude/docs/security.md` |
| RAG / chunking / retrieval / embeddings / KB | `.claude/docs/rag.md` |
| File upload / sensitive counter | `.claude/docs/fileupload.md` |
| Image generation / stream=False / response_format | `.claude/docs/image.md` |
| Local dev / jump host / SSH tunnel / test locally | `.claude/docs/localdev.md` |

`spec/` = what to build (empty). `skills/` = 22 reusable skills (see Workflow table above). `archive/` = scratch, do not load unless asked.

---

## Critical rules (never violate)

- NEVER remove `body["stream"] = False` (non-streaming / Ollama paths)
- NEVER expose proxy port on `0.0.0.0`
- NEVER log request headers (API key leak)
- NEVER commit `cosign.key`, `.env`, or secrets
- ALWAYS read files before editing; verify fix in running container before marking done
- ALWAYS ask Seb before structural changes
- Ahmed deploys manually — agents change code only

## Communication

- **Ahmed**: informal, abbreviations OK
- **Seb**: max 3-4 lines, one ask, paragraph tone, no bullets
- **GitHub comments**: factual, lowercase, no fluff

## Agent prompt pattern

Split every prompt: (1) code changes only — describe problem + location, no prescribed fix unless asked; (2) deploy/verify commands kept separate for Ahmed to run.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
