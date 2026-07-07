---
name: ahmedclaudeskill
description: Use when Ahmed says "use ahmedclaudeskill" or "bootstrap claude arch" or "set up claude infra" or "generate claude files". Smart agent — scans a designated canonical project's .claude/ directory LIVE every time, builds a memory snapshot of the full arch, then reproduces it identically in the current (new) project. Zero baking, zero drift. Configurable canonical source. Copies everything: skills, agents, folders, data files, templates.
tools: [Read, Write, Bash]
---

# ahmedclaudeskill Agent v4

> Scans canonical project LIVE → reproduces identical .claude/ arch in new project.
> No baked-in content. No drift. Always reads real files from source.

---

## When to trigger

"use ahmedclaudeskill", "bootstrap claude arch", "set up claude infra",
"generate claude files", "give me the ahmed arch", "set up .claude/"

---

## Step 0 — Determine canonical source

Read `.claude/agents/ahmedclaudeskill.md` (this file) for the configured path.
The canonical source is configured here:

```
CANONICAL_SOURCE: ~/Desktop/enclaive/healthdetector
```

If Ahmed says "use X as canonical" or "scan from X" → use that path instead
for this run only (don't permanently change the config unless Ahmed says
"update canonical to X").

To update canonical permanently:
```bash
# Ahmed runs this when he wants to change the source project
sed -i 's|CANONICAL_SOURCE:.*|CANONICAL_SOURCE: ~/Desktop/enclaive/<new-project>|' \
  ~/.claude/agents/ahmedclaudeskill.md
```

---

## Step 1 — Scan canonical source LIVE

```bash
CANONICAL=~/Desktop/enclaive/healthdetector   # read from config above

# verify it exists and has .claude/
ls "$CANONICAL/.claude/" || echo "ERROR: canonical source not found"

# get full skill list
ls "$CANONICAL/.claude/skills/"

# get full agent list
ls "$CANONICAL/.claude/agents/" 2>/dev/null || echo "no agents yet"

# get full folder structure
find "$CANONICAL/.claude/" -type d | sort

# get all data file paths
find "$CANONICAL/.claude/docs/" -type f | sort
```

Build a live inventory:
- All skill names + SKILL.md paths
- All agent names + .md paths
- All doc files (cidata.md, infradata.md, etc)
- All folder names
- All empty placeholder files (github-ticket.md, comments.md, etc)

---

## Step 2 — Ask what to do

```
Found canonical source: ~/Desktop/enclaive/healthdetector
.claude/ contains:
  Skills: <N> (list names)
  Agents: <N> (list names)
  Docs: <N> files
  Folders: <N>

Target project: <current working directory>

What to do?
1. Full clone — copy EVERYTHING from canonical to this project
2. Selective — copy only specific skills/agents (tell me which)
3. Update — current project already has .claude/, only copy NEW things (diff mode)
4. Change canonical source — set a different project as canonical

Type 1/2/3/4:
```

---

## Step 3a — Full clone (option 1)

```bash
TARGET=$(pwd)
CANONICAL=~/Desktop/enclaive/healthdetector

echo "Cloning full .claude/ from $CANONICAL to $TARGET..."

# 1. Copy all skills verbatim
mkdir -p "$TARGET/.claude/skills"
cp -r "$CANONICAL/.claude/skills/"* "$TARGET/.claude/skills/"

# 2. Copy all agents verbatim
if [ -d "$CANONICAL/.claude/agents" ]; then
  mkdir -p "$TARGET/.claude/agents"
  cp -r "$CANONICAL/.claude/agents/"* "$TARGET/.claude/agents/"
fi

# 3. Recreate full folder structure
while IFS= read -r dir; do
  RELATIVE="${dir#$CANONICAL/}"
  mkdir -p "$TARGET/$RELATIVE"
done < <(find "$CANONICAL/.claude/" -type d | sort)

# 4. Copy doc templates (empty ones only — don't copy project-specific content)
EMPTY_TEMPLATES=(
  ".claude/docs/ci/cidata.md"
  ".claude/docs/cd/cddata.md"
  ".claude/docs/comments/comments.md"
  ".claude/docs/github/githubview.md"
  ".claude/skills/update-github-ticket/github-ticket.md"
)
for f in "${EMPTY_TEMPLATES[@]}"; do
  touch "$TARGET/$f"
done

# 5. Copy doc structure files that ARE templates (stack.md, deploy.md etc)
# but clear project-specific content, keeping only the section headers
for doc in stack.md deploy.md debug.md security.md localdev.md; do
  if [ -f "$CANONICAL/.claude/docs/$doc" ]; then
    # copy structure only — Ahmed fills project-specific content
    head -5 "$CANONICAL/.claude/docs/$doc" > "$TARGET/.claude/docs/$doc"
    echo "" >> "$TARGET/.claude/docs/$doc"
    echo "> Fill this with $TARGET project details." >> "$TARGET/.claude/docs/$doc"
  fi
done

echo ""
echo "✅ Clone complete"
echo "Skills: $(ls $TARGET/.claude/skills | wc -l)"
echo "Agents: $(ls $TARGET/.claude/agents 2>/dev/null | wc -l)"
echo "Folders: $(find $TARGET/.claude -type d | wc -l)"
```

---

## Step 3b — Selective clone (option 2)

Ahmed names which skills/agents to copy. Copy only those:

```bash
# example: copy only coach, implement-feature, reviewer
for item in "$@"; do
  if [ -d "$CANONICAL/.claude/skills/$item" ]; then
    cp -r "$CANONICAL/.claude/skills/$item" "$TARGET/.claude/skills/"
    echo "✅ skill: $item"
  elif [ -f "$CANONICAL/.claude/agents/$item.md" ]; then
    cp "$CANONICAL/.claude/agents/$item.md" "$TARGET/.claude/agents/"
    echo "✅ agent: $item"
  else
    echo "⚠️ not found: $item"
  fi
done
```

---

## Step 3c — Diff/update mode (option 3)

Current project already has .claude/. Only add what's missing:

```bash
TARGET=$(pwd)
CANONICAL=~/Desktop/enclaive/healthdetector

echo "Diff mode — checking for missing skills/agents..."

# skills in canonical but NOT in target
for skill in "$CANONICAL/.claude/skills/"*/; do
  name=$(basename "$skill")
  if [ ! -d "$TARGET/.claude/skills/$name" ]; then
    cp -r "$skill" "$TARGET/.claude/skills/"
    echo "➕ Added missing skill: $name"
  else
    echo "✓ Already exists: $name"
  fi
done

# agents in canonical but NOT in target
if [ -d "$CANONICAL/.claude/agents" ]; then
  for agent in "$CANONICAL/.claude/agents/"*.md; do
    name=$(basename "$agent")
    if [ ! -f "$TARGET/.claude/agents/$name" ]; then
      cp "$agent" "$TARGET/.claude/agents/"
      echo "➕ Added missing agent: $name"
    else
      echo "✓ Already exists: $name"
    fi
  done
fi

# folders in canonical but NOT in target
while IFS= read -r dir; do
  RELATIVE="${dir#$CANONICAL/}"
  if [ ! -d "$TARGET/$RELATIVE" ]; then
    mkdir -p "$TARGET/$RELATIVE"
    echo "➕ Created folder: $RELATIVE"
  fi
done < <(find "$CANONICAL/.claude/" -type d | sort)
```

---

## Step 4 — Ask project-specific questions (after clone)

After copying files, ask ONLY what's needed for CLAUDE.md generation:

```
.claude/ cloned successfully.

Now I'll generate CLAUDE.md for this project.
Need a few details:

1. What is this project? (1 sentence if README doesn't explain it)
2. Who's the approver? (name + communication style)
3. Is this an Enclaive project? (Y/N — affects enclaive-docs/enclaiveask inclusion)
4. Add .claude/ to .gitignore? (Y/N — recommended)
```

Read from real files first (README.md, .git/config, package.json, docker-compose.yml)
→ only ask what can't be detected.

---

## Step 5 — Generate CLAUDE.md for the NEW project

Generate `CLAUDE.md` at project root customized to the new project:

```markdown
# CLAUDE.md — <NEW_PROJECT> Router

> Read this first. Load ONLY the file your task needs.
> ⚠️ Active work = .claude/plan/mission.md
> Feature backlog = .claude/plan/proposedfeature.md

## What is <NEW_PROJECT>
<detected from README or asked>

## Architecture
<ASCII diagram from detected stack>

## Team
| Person | Role | Style |
|--------|------|-------|
| Ahmed | dev / owner | informal, short, direct |
| <approver> | reviewer | max 3-4 lines, paragraph, no bullets |

## Repos & environments
- Repo: <from .git/config>
- Deploy: <from docker-compose or asked>

## Workflow — start here
Run /coach first for any new task.

## Skills (<N> total)
<list all skills with trigger — copied from canonical, not hardcoded>

## Agents (<N> total)
<list all agents with trigger>

## Load the right file
| Task | File |
|------|------|
| Top priority | .claude/plan/mission.md |
| Current status | .claude/plan/tasks.md |
| Feature backlog | .claude/plan/proposedfeature.md |
| Stack | .claude/docs/stack.md |
| Deploy | .claude/docs/deploy.md |
| Debug | .claude/docs/debug.md |
| Security | .claude/docs/security.md |
| Local dev | .claude/docs/localdev.md |
| Infra/k8s | .claude/docs/infradata.md |
| CI state | .claude/docs/ci/cidata.md |
| Deploy state | .claude/docs/cd/cddata.md |
| Past decisions | .claude/docs/decisions/ |

## Critical rules
- Read files before editing
- Ahmed deploys manually — agent changes code only
- Ask <approver> before structural changes
- No secrets in commits (.env, keys, webhooks)
- <project-specific rules detected from files>

## Communication
- Ahmed: short, direct, informal, typos OK
- <approver>: max 3-4 lines, paragraph, no bullets

---
## Changelog
- <YYYY-MM-DD>: Bootstrapped from ahmedclaudeskill v4 (canonical: healthdetector)
```

---

## Step 6 — Final report

```
✅ ahmedclaudeskill complete for <project-name>

Cloned from: ~/Desktop/enclaive/healthdetector
Skills: <N> copied
Agents: <N> copied (reviewer, council, creative)
Folders: <N> created
CLAUDE.md: generated

.gitignore: .claude/ <added/skipped>

Next steps:
1. Paste ticket into .claude/skills/update-github-ticket/github-ticket.md
2. Say "overview the ticket" → auto-fills mission.md + proposedfeature.md
3. Say "/coach" → get your first skill path

Or start directly:
4. Say "i want to add <feature>" → build-feature
```

---

## Updating the canonical source

When healthdetector gets new skills/agents:
```
update canonical — I added a new skill called X
```

This agent re-reads healthdetector and notes the change in its own file header.
All future bootstraps will include skill X automatically.

---

## Rules

- **ALWAYS read canonical source live** — never use baked-in skill content.
  This is the core guarantee of v4. If canonical path doesn't exist → stop
  and tell Ahmed: "canonical source not found at <path>. Run this from
  a machine where healthdetector is available, or update canonical path."
- **Bash for copying, not Claude Code file writing** — use actual shell cp
  commands to ensure byte-identical copies, never try to reproduce file
  content by reading and re-writing it.
- **CLAUDE.md is generated per-project** — it's the ONLY file not copied
  from canonical. Everything else is verbatim copy.
- **Doc templates are cleared of project-specific content** — stack.md
  structure stays, healthdetector's actual service names don't.
- **Diff mode is safe** — never overwrites existing files in option 3.
  Only adds what's missing.
- **Self-referencing** — this agent copies itself to the new project, so
  future projects can also bootstrap from a canonical source.