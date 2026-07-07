# claudearch

Ahmed's Claude Code architecture — **14 skills · 8 agents · 9 MCPs · 11 plugins**.
Portable across projects. Verified functional. Zero overlaps.

---

## Quick start (new project)

```bash
cd ~/Desktop/your-new-project
git clone https://github.com/MarzouguiAhmed9/claudearch
# in Claude Code:
/setup
```

The `setup` agent copies `.claude/` + `CLAUDE.md` into your new project, adapts project-specific fields, and verifies every skill/agent/MCP registers.

---

## Design principles

1. **Mode-based mega-agents.** Instead of 40 tiny skills, 4 big agents (planner, implementer, reviewer, infra) each with 6-8 modes. Fewer routing conflicts.
2. **Single sources of truth.** `data.md` = external context. `git` skill = git ops. `data.md` 🎫 = tickets. No duplication.
3. **Read-first, act-second.** Every agent has phase 0 = read state. Never fabricates.
4. **Paste-mode fallback.** When MCPs can't reach live cluster/CI, agents ask Ahmed to paste output. Never guesses.
5. **Ponytail applies.** Shortest working diff. No unrequested abstractions.

---

## The 5 pillars

```
    THINK          BUILD         CHECK        KNOW         SYNC
    (6 tools)      (2 agents)    (1 agent)    (5 tools)    (9 tools)
    ├─ planner    ├─ implementer  reviewer   ├─ web        ├─ coach
    ├─ council    └─ infra                    ├─ data       ├─ git
    ├─ creative                               ├─ graphify   ├─ update-project
    ├─ improve                                ├─ enclaivetech ├─ readme
    ├─ improveclaude                          ├─ enclaiveask ├─ network-map
    └─ logic                                  └─ baby-explain ├─ clean-comments
                                                              ├─ setup
                                                              └─ (built-ins)
```

---

## Agents (8)

Every agent lives in `.claude/agents/<name>.md`. Isolated context window (doesn't pollute main chat).

### `planner` — turn idea into ready-to-build plan
**Modes:** BRAINSTORM · QUICK · SPEC · DESIGN · TASKS · SDD-FULL · EXTRACT · REVIEW-PLAN
**Uses:** superpowers:brainstorming (mandatory), superpowers:writing-plans, feature-dev plugin, context7 MCP
**Writes:** `.claude/features/YYYY-MM-DD-<slug>/{00-idea, 01-spec, 03-design, 05-tasks}.md`
**Handoff:** ends with `STATUS: ready-for-impl` → implementer picks up

### `implementer` — turn plan into working code
**Modes:** SETUP · TDD · EXECUTE · PARALLEL · VERIFY · REVIEW · PR
**Uses:** superpowers:test-driven-development, executing-plans, verification-before-completion, using-git-worktrees, code-modernization plugin, context7 MCP
**Reads:** `05-tasks.md`
**Writes:** `06-build.md`
**Handoff:** dispatches `reviewer` at REVIEW mode, then `/git PR` mode. Never deploys — hands to `infra`.

### `reviewer` — the CHECK pillar (absorbs test + debug + antivibecode)
**Modes:** FULL · VIBE · TEST · DEBUG · SECURITY · PR · COMPLEXITY
- **FULL** — 5-phase full scan, dispatches 4 specialists (pr-review-toolkit), reads security-guidance findings, runs 6-pair cross-boundary contract check
- **VIBE** — 44 vibe-code defects across 7 categories
- **TEST** — python-testing MCP fuzz + mutation testing, framework-detected
- **DEBUG** — root cause from debug.md + SSH paste-mode
- **SECURITY** — semgrep MCP + /security-review built-in
- **PR** — gh CLI + /review built-in
- **COMPLEXITY** — dispatches ponytail-audit/review/debt

**Uses:** pr-review-toolkit specialists, security-guidance hooks, semgrep MCP, python-testing MCP, ponytail plugin
**Writes:** `.claude/agents/reviewer/report-YYYY-MM-DD-<mode>.md`

### `infra` — everything CI/CD/k8s/Helm/ArgoCD/Docker
**Modes:** ANSWER · BUILD · REVIEW · DIAGNOSE · DEPLOY · MIGRATE
- **ANSWER** — factual queries from `.claude/docs/infra/state.md`
- **BUILD** — writes Dockerfile, docker-compose, k8s manifests, Helm charts, GitHub Actions workflows
- **REVIEW** — called by reviewer agent for manifest audit
- **DIAGNOSE** — pod crashloop, CI failure, deploy stuck; falls back to paste-mode
- **DEPLOY** — 3-choice confirm (YES / DRY-RUN / NO) before any push
- **MIGRATE** — Docker Compose → k8s planning

**Uses:** kubernetes-mcp-server, mcp-helm, mcp-for-argocd, github MCP, gh CLI
**Writes:** `.claude/docs/infra/state.md` (single infra state file)

### `council` — 3-lens debate
Runs Developer / Security / Logic perspectives on the same input, synthesizes DO / DON'T / DEFER.
**Writes:** `.claude/docs/decisions/*.md`

### `creative` — blue-sky improvement ideas
Reads whole project, generates ranked ideas across 10 dimensions (perf, UX, security, arch, DX, reliability, observability, business).
**Writes:** `.claude/docs/creative/ideas-*.md`

### `logic` — scenario coverage vs requirements
Reads ticket from `data.md` 🎫, brainstorms ALL scenarios (happy path, edges, errors, races), checks source handles each. Asks Ahmed to paste live output for uncertain ones.

### `setup` — bootstrap into new project
Reads `./claudearch/` clone, copies `.claude/` + `CLAUDE.md` into current project, adapts fields, verifies. See Quick start above.

---

## Skills (14)

Every skill lives in `.claude/skills/<name>/SKILL.md`. User-invoked via slash commands or natural language.

### Core routing / meta

| Skill | Trigger | Role | Writes |
|---|---|---|---|
| `coach` | `/coach` | Run FIRST. 3 phases: SELF-CHECK (verify skills/agents/MCPs healthy) → ASK (one question about the task) → ROUTE (recommend skill+agent path with model+effort) | chat only |
| `graphify` | Any codebase question | Persistent knowledge graph. `graphify query <q>`, `graphify explain <concept>`, `graphify path <A> <B>` | `graphify-out/` |

### Context storage

| Skill | Trigger | Role | Writes |
|---|---|---|---|
| `data` | `/data`, "save this", "ingest X", "remember this" | **Single source of truth** for external context (tickets, team notes, docs, images, ideas, config). Classifies into 6 fixed sections. Auto-tidies at 2000 lines. | `.claude/docs/data.md` |

### Git / GitHub

| Skill | Trigger | Role | Writes |
|---|---|---|---|
| `git` | "git status", "commit", "open PR", "create issue", "post progress" | 6 modes: VIEW · SNAPSHOT · COMMIT · PR · ISSUE · POST. Reads tickets from `data.md` 🎫 (never duplicates). 3-choice confirm on any push. | `.claude/skills/git/snapshots/*.md`, `.claude/skills/git/last-post.md` |

### Web / fetch

| Skill | Trigger | Role | Writes |
|---|---|---|---|
| `web` | `/web`, "/fetch", paste URL, "search X", "docs of X", "owu docs" | 5 modes: FETCH · SEARCH · CRAWL · DOC · OWU. Uses Firecrawl + Exa + context7 MCPs. Offers to save results to `data.md` 📚 via data skill. | chat + offers save to `data.md` |

### Audits / improvements

| Skill | Trigger | Role | Writes |
|---|---|---|---|
| `improve` | "/improve", "improve <project>", "brilliant ideas" | Godmode project audit (Opus + max thinking) across security, performance, correctness, DX, UX. | `.claude/docs/improvements/YYYY-MM-DD-improve.md` |
| `improveclaude` | "/improveclaude", "improve claude", "audit .claude" | Godmode Claude arch audit. **Mandatory:** invokes `claude-code-setup:claude-automation-recommender` + scans installed marketplaces + WebSearch fresh plugins/MCPs. | `.claude/improve/YYYY-MM-DD-improveclaude.md` |

### Documentation

| Skill | Trigger | Role | Writes |
|---|---|---|---|
| `update-project` | "update project", "sync docs" | Diffs `.claude/` against actual code reality, makes surgical str_replace updates. Never overwrites. | edits CLAUDE.md + docs/* |
| `readme` | "generate readme" | Reads CLAUDE.md + docs + docker-compose → developer-facing README. | `README.md` at project root |
| `network-map` | "network map" | Reads docker-compose + proxy code → ASCII service + data-flow map. | `.claude/skills/network-map/network-map.md` |
| `clean-comments` | "clean comments" | Tidies `comments.md` progress log — removes dupes, keeps last 10. | `.claude/docs/comments/comments.md` |

### Explain / reference

| Skill | Trigger | Role | Writes |
|---|---|---|---|
| `baby-explain` | "explain easily", "ELI5", "baby mode" | Reads any feature/plan/spec file → plain-language explanation. | `.claude/explanations/<name>.md` |
| `enclaiveask` | `/enclaiveask <question>` | Answers a teammate question in plain human language, copy-paste ready. No jargon. | chat only |
| `enclaivetech` | vHSM/Vault/Nitride/Buckypaper mentions | Enclaive product reference — always grounded in official docs, never general Vault knowledge. | chat only |

---

## MCPs (9)

Registered in `~/.claude.json` per-project. Fall back gracefully when disconnected.

| MCP | Used by | Falls back to |
|---|---|---|
| **kubernetes** (containers/kubernetes-mcp-server) | infra ANSWER/DIAGNOSE/DEPLOY/REVIEW | `kubectl` CLI → paste-mode |
| **helm** (zekker6/mcp-helm) | infra BUILD/REVIEW | `helm` CLI → paste-mode |
| **argocd** (argoproj-labs/mcp-for-argocd) | infra DEPLOY | `argocd` CLI → paste-mode |
| **semgrep** (semgrep/mcp) | reviewer SECURITY, VIBE | `semgrep` CLI → skip |
| **python-testing** (jazzberry-ai/python-testing-mcp) | reviewer TEST | hand-written pytest |
| **context7** | planner DESIGN, implementer, web DOC | WebFetch |
| **firecrawl** | web FETCH/CRAWL | WebFetch |
| **exa** | web SEARCH, improveclaude | WebSearch |
| **playwright** | reviewer FULL (Svelte E2E if wired) | skip |
| **github** | git PR/ISSUE/POST, infra CI diagnosis | `gh` CLI → paste-mode |

---

## Plugins (11)

| Plugin | Provides | Used by |
|---|---|---|
| **superpowers** | 14 skills: brainstorming, TDD, verification-before-completion, executing-plans, using-git-worktrees, subagent-driven-development, etc. | planner, implementer |
| **ponytail** | ponytail (lazy mode), ponytail-audit, ponytail-review, ponytail-debt, ponytail-help, ponytail-gain | reviewer COMPLEXITY, all coding |
| **pr-review-toolkit** | 6 specialist subagents: silent-failure-hunter, pr-test-analyzer, type-design-analyzer, code-simplifier, comment-analyzer, code-reviewer | reviewer FULL/VIBE/PR |
| **security-guidance** | Passive hooks on Edit/Write/Stop: pattern-based warnings + LLM diff review for 25+ vuln classes | reviewer FULL/SECURITY (reads state) |
| **feature-dev** | Feature planning workflow templates | planner |
| **claude-md-management** | Keep CLAUDE.md aligned across changes | planner |
| **code-modernization** | Modernize legacy patterns (dict → dataclass, old async idioms) | implementer refactors |
| **context7** | Live library docs skill (also exposed as MCP) | planner, implementer, web DOC |
| **claude-code-setup** | Automation recommender skill | improveclaude (mandatory) |
| **skill-creator** | Create/modify/eval skills | when adding new skills |
| **superpowers-marketplace** | Marketplace registration for superpowers | — |

---

## Hooks (in `.claude/settings.json`)

Two `PreToolUse` hooks enforce graphify usage before raw source reads/grep:

- **Bash matcher** — if command contains `grep|rg|find|fd|ack|ag` AND `graphify-out/graph.json` exists → inject "MUST run graphify query first" context.
- **Read|Glob matcher** — if reading `.py/.js/.ts/.tsx/.svelte/.go/.rs` etc AND `graphify-out/graph.json` exists → same enforcement.

Both are advisory (add context, don't block). They keep agents oriented on the code graph instead of grepping blind.

---

## File map — what each tool writes

```
.claude/
├── docs/
│   ├── data.md              ← data skill (SINGLE source of truth)
│   ├── comments/
│   │   └── comments.md      ← clean-comments tidies
│   ├── improvements/
│   │   └── YYYY-MM-DD-improve.md  ← improve skill
│   ├── infra/
│   │   └── state.md         ← infra agent state
│   ├── decisions/           ← council agent
│   └── creative/            ← creative agent
├── improve/
│   └── YYYY-MM-DD-improveclaude.md  ← improveclaude skill
├── features/YYYY-MM-DD-<slug>/
│   ├── 00-idea.md           ← planner BRAINSTORM
│   ├── 01-spec.md           ← planner SPEC
│   ├── 03-design.md         ← planner DESIGN
│   ├── 05-tasks.md          ← planner TASKS (handoff to implementer)
│   ├── 06-build.md          ← implementer log
│   └── 07-validate.md       ← reviewer post-impl
├── explanations/
│   └── <name>.md            ← baby-explain
├── skills/
│   ├── git/
│   │   ├── snapshots/*.md   ← git SNAPSHOT mode
│   │   └── last-post.md     ← git POST mode
│   └── network-map/
│       └── network-map.md   ← network-map skill
├── agents/
│   └── reviewer/
│       └── report-YYYY-MM-DD-<mode>.md  ← reviewer all modes
├── plan/
│   └── tasks.md             ← project-wide task log
└── settings.json            ← hooks
```

---

## How the tools work together — full feature lifecycle

```
1. Ahmed has an idea
   ↓
2. /coach (SELF-CHECK → ASK → ROUTE)
   → recommends planner BRAINSTORM
   ↓
3. planner BRAINSTORM
   → uses superpowers:brainstorming
   → writes 00-idea.md
   ↓
4. planner SPEC → DESIGN (Opus + max) → TASKS
   → uses context7 MCP for library decisions
   → writes 01-spec.md, 03-design.md, 05-tasks.md
   → STATUS: ready-for-impl
   ↓
5. implementer SETUP
   → uses superpowers:using-git-worktrees
   → reads 05-tasks.md
   → writes 06-build.md
   ↓
6. implementer EXECUTE / TDD (per task)
   → uses superpowers:test-driven-development
   → uses context7 MCP for API signatures
   → uses code-modernization plugin for refactors
   → after each task: superpowers:verification-before-completion
   ↓
7. implementer REVIEW mode
   → dispatches reviewer FULL
   ↓
8. reviewer FULL
   → Phase 0.5: dispatches 4 specialists in parallel
     - pr-review-toolkit:silent-failure-hunter
     - pr-review-toolkit:pr-test-analyzer
     - pr-review-toolkit:type-design-analyzer
     - infra REVIEW mode (helm lint, kubectl dry-run)
   → Phase 1: own scan with graphify
   → Phase 1.5: reads security-guidance findings
   → Phase 1.6: cross-boundary contract (env/port/volume/secret/image/health)
   → Phase 2: generates tests via python-testing MCP
   → writes report-YYYY-MM-DD-full.md
   ↓
9. implementer PR mode → hands off to /git PR
   ↓
10. /git PR
    → uses github MCP + gh CLI
    → drafts body from 05-tasks.md + reviewer report
    → 3-choice confirm
    → opens PR
    ↓
11. infra DEPLOY mode
    → 3-choice confirm (YES / DRY-RUN / NO)
    → uses kubernetes/helm/argocd MCPs or paste-mode
    → updates .claude/docs/infra/state.md
    ↓
12. /data (throughout — save any teammate feedback, decisions, docs)
    → appends to data.md
```

---

## Trigger cheatsheet

| Say | Fires |
|---|---|
| `/coach` | coach (health check + route) |
| `/setup` | setup (bootstrap into new project) |
| "plan X" / "spec X" / "design X" / "task list for X" / "overview X" | planner (mode by phrase) |
| "implement X" / "code it" / "build it" | implementer |
| "/reviewer" / "review X" / "vibe check" / "write tests" / "debug X" / "security check" / "review PR N" / "over-engineered?" | reviewer (mode by phrase) |
| "git status" / "commit" / "open PR" / "create issue" / "post progress" / "/updategithubticket" | git (mode by phrase) |
| "/web" / paste URL / "search for X" / "docs of X" / "owu docs" | web (mode by phrase) |
| "/data" / "save this" / "ingest X" | data |
| "improve garnet" / "brilliant ideas" | improve |
| "improve claude" / "audit .claude" | improveclaude |
| "explain easily" / "ELI5" | baby-explain |
| "/enclaiveask <q>" | enclaiveask |
| vHSM / Vault / Nitride / Buckypaper mentions | enclaivetech |
| "sync docs" / "update project" | update-project |
| "generate readme" | readme |
| "network map" | network-map |
| any infra topic (Docker, k8s, Helm, ArgoCD, Harbor, CI, deploy) | infra (mode by phrase) |
| "/council <topic>" | council |
| "/creative" | creative |
| "/logic" | logic |

---

## Setup — full walkthrough

Real end-to-end example. Say you're starting a new project called **`api-gateway`** — a Go + Gin backend deployed to k8s.

### Step 1 — Create + clone

```bash
mkdir -p ~/Desktop/api-gateway && cd ~/Desktop/api-gateway
git init
git clone https://github.com/MarzouguiAhmed9/claudearch
```

Directory now:
```
~/Desktop/api-gateway/
├── .git/
└── claudearch/         ← the cloned repo (source)
    ├── .claude/
    ├── CLAUDE.md
    └── README.md
```

### Step 2 — Open Claude Code + run setup

```bash
claude    # or open your Claude Code IDE in this folder
```

Then type:
```
/setup
```

### Step 3 — What the agent does

**Phase 1 — Existing `.claude/` check**
```
Mode: SETUP
No existing .claude/ detected — CLEAN install.
Proceeding.
```

**Phase 2 — Ask project identity**
```
New project name (short slug):
Primary stack:
Git remote URL:
Deploy target:
```

You answer:
```
api-gateway
go + gin
git@github.com:me/api-gateway.git
k8s
```

**Phase 3 — Copy**
```
cp -r ./claudearch/.claude ./
cp ./claudearch/CLAUDE.md ./
✓ 14 skills copied
✓ 8 agents copied
```

**Phase 4 — Adapt `CLAUDE.md`**
```
Adapted CLAUDE.md:
  - 12 "garnet" → "api-gateway"
  - 3 path refs updated
  - 4 garnet-specific sections commented out:
      • Team table (Ahmed/Seb/Ion/Nicu/Ahmed B)
      • FastAPI + Svelte + Presidio stack
      • cVM 65.108.38.50
      • Harbor harbor.enclaive.cloud/garnetdemo/
  - Backup saved: CLAUDE.md.bak

Review commented-out sections and either restore or delete.
```

**Phase 5 — Verify**
```
Skills:   14/14 ✓
Agents:   8/8 ✓
MCPs:     3 connected / 9 configured

Missing MCPs (run to install):
  claude mcp add kubernetes -- npx -y kubernetes-mcp-server@latest
  claude mcp add helm -- npx -y @zekker6/mcp-helm@latest
  claude mcp add argocd -- npx -y argocd-mcp@latest
  claude mcp add semgrep -- npx -y semgrep-mcp@latest
  claude mcp add python-testing -- npx -y @jazzberry-ai/python-testing-mcp@latest
  claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
```

Setup does NOT auto-install — copy/paste the commands you want.

**Phase 6 — Report**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Setup complete: api-gateway
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Skills:   14/14 ✓
Agents:   8/8 ✓
MCPs:     3 configured, 6 pending

Next:
  1. Review CLAUDE.md commented sections
  2. Delete or restore team table, adapt to your team
  3. Run: /coach
  4. Optional: graphify update .
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 4 — Clean up + first task

Remove the cloned source directory (no longer needed):
```bash
rm -rf ./claudearch
```

Verify + get first suggested task:
```
/coach
```

Coach runs SELF-CHECK, then asks what you want to build. Say `I want to add a rate limiter` and you get:
```
Route:
1. planner QUICK — Blueprint the rate limiter feature → 05-tasks.md
2. implementer EXECUTE — Read the tasks, write Go code, run tests

Start with: "plan a rate limiter feature"
```

### Step 5 — Now you have the full arch

Every skill, agent, MCP, plugin from garnet — working in your new project.

---

### Common gotchas

| Symptom | Fix |
|---|---|
| `/setup` says "clone first" | You forgot to `git clone https://github.com/MarzouguiAhmed9/claudearch` inside the project dir |
| `/coach` reports "Skills: 0/14" | You ran `/coach` from a parent dir — `cd` into the project first |
| MCPs all show ✗ Failed | Normal on first probe — they auto-install on first real use OR you skipped the `claude mcp add` commands |
| CLAUDE.md still says "garnet" everywhere | You ran `/setup` on an existing `.claude/` and picked SKIP — re-run with OVERWRITE, or manually edit |
| Reviewer / infra agents refer to `.claude/docs/data.md` 🎫 but yours is empty | Normal — add first ticket via `/data ingest ticket #1 <URL>` |

---

## Adapting for your project

After `/setup`, review these garnet-specific items in `CLAUDE.md`:

- **Team table** — Ahmed / Seb / Ion / Nicu / Ahmed Bouzid → replace with your team
- **Repo table** — enclaive/garnet URLs → your repo
- **Rules section** — "NEVER remove `body['stream'] = False`" etc are garnet-proxy-specific → remove
- **Stack references** — FastAPI + Svelte + Presidio + litellm → your stack
- **Deployment** — cVM 65.108.38.50 / Harbor → your deploy target

`setup` agent auto-comments most of these; Ahmed reviews the diff.

---

## License

Personal architecture snapshot. Fork freely, adapt to your team.
