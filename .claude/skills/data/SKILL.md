---
name: data
description: Use when Ahmed says "/data", "add to data", "data: <paste>", "save this for context", "remember this", "ingest ticket X", "save teammate note", "save doc", "save this image ref", or wants to persist any external context (tickets, team notes from Seb/Ion/Nicu/Ahmed B, referenced docs, image paths, ideas, config snippets) that ALL agents should later know. Classifies input into 1 of 6 fixed sections in .claude/docs/data.md, appends dated entry, auto-tidies when file exceeds 2000 lines. Read-only for agents — write-only through this skill. Never overwrites, always appends. Never deletes user content (only condenses oldest entries to one-liners at tidy threshold).
---

# data Skill — single source of truth for external context

> ONE file: `.claude/docs/data.md`
> ONE writer: this skill
> Many readers: every agent, every skill
> Six fixed sections. Auto-tidy at 2000 lines.

---

## When to trigger

Ahmed says:
- "/data" / "add to data" / "data: <paste>"
- "save this for context" / "remember this"
- "ingest ticket <ref>" / "save the ticket <ref>"
- "save teammate note" / "seb said ..." / "ion told me ..."
- "save this doc" / "reference this"
- "save image ref" / "add screenshot ref"
- "add this idea" / "note this decision"
- "save this config" / "note this API"

---

## The file

Path: `.claude/docs/data.md`

Six fixed sections (never rename, never add new):

| # | Section | Icon | For |
|---|---|---|---|
| 1 | Tickets | 🎫 | GitHub issues, Linear tasks, JIRA tickets |
| 2 | Team notes | 👥 | Instructions/notes from Seb, Ion, Nicu, Ahmed B, Alexa, Julia |
| 3 | Referenced docs | 📚 | Pasted external docs, RFCs, spec extracts |
| 4 | Images / screenshots | 🖼️ | Diagram paths + what they show |
| 5 | Ideas & decisions | 💡 | Not-yet-features + team decisions |
| 6 | Config / API notes | 🔧 | Model IDs, endpoints, env vars, API quirks |

---

## Steps

### Step 1 — Classify the input

Look at the pasted content. Pick 1 of 6 sections:

| Signal | Section |
|---|---|
| GitHub URL, "issue #", "PR #", ticket ref, acceptance criteria language | 🎫 Tickets |
| Person name (Seb/Ion/Nicu/Ahmed B/Alexa/Julia/Sebastian) + instruction/opinion | 👥 Team notes |
| URL to external docs, RFC number, "https://docs...", pasted API reference | 📚 Referenced docs |
| Image path, "screenshot", `.png` / `.jpg`, "see attached", diagram reference | 🖼️ Images |
| "I want to ...", "we should ...", "decided to ...", "brainstorm ...", not tied to a ticket | 💡 Ideas & decisions |
| Model IDs (claude-*, gpt-*), env var, endpoint URL, config key, API quirk | 🔧 Config / API |

**If ambiguous → ask ONE question:**

```
Which section?
[1] 🎫 Tickets
[2] 👥 Team notes
[3] 📚 Referenced docs
[4] 🖼️ Images
[5] 💡 Ideas & decisions
[6] 🔧 Config / API notes
```

Wait for number.

### Step 2 — Extract metadata

From the content, derive:
- **Date:** today's system date (YYYY-MM-DD)
- **Ref / person / source:** the primary identifier
  - Tickets → ticket number or title
  - Team → person's name
  - Docs → source (e.g. "enclaive.cloud/docs")
  - Images → filename or slug
  - Ideas → auto slug from first sentence
  - Config → target (e.g. "Groq", "OLLAMA_URL")
- **One-line summary:** ≤ 80 chars, human-readable

### Step 3 — Append the entry

Format:

```markdown
### YYYY-MM-DD — <ref/person/source> — <one-line summary>

<the actual content, preserved verbatim — code blocks kept, quotes kept>

<optional: `Cited by: <where Ahmed found it>` if given>
```

Append UNDER the matching section heading, ABOVE any `_none yet_` placeholder (which you delete on first real entry).

Preserve chronological order — newest at the top of each section.

### Step 4 — Update the top-of-file timestamp

Change `_Last updated: <date>_` to today's date.

### Step 5 — Check tidy threshold

```bash
wc -l .claude/docs/data.md
```

If > 2000 lines → run auto-tidy (Step 6).
Else → done.

### Step 6 — Auto-tidy (only when > 2000 lines)

For each of the 6 sections:
1. Keep the last 10 entries verbatim
2. For older entries: replace with a one-line summary format:
   ```
   ### <date> — <ref> — <summary> _(condensed)_
   ```
3. Preserve the condensed summaries chronologically at the bottom of the section

**Never delete a user entry outright. Only compress old ones.**

After tidy, add a comment at the top of the file:

```markdown
_Tidied YYYY-MM-DD — <N> old entries condensed_
```

### Step 7 — Report

```
✅ Added to 📚 Referenced docs
File: .claude/docs/data.md
Section: <name>
Entry: <date> — <ref> — <summary>
File size: <N> lines (tidy at 2000)
```

---

## Rules

- **Never overwrite.** Always append. `str_replace` on the placeholder `_none yet_` for first entry.
- **Never rename sections.** Six is the schema. If Ahmed wants a new section, tell him — don't invent.
- **Never delete entries.** Auto-tidy compresses, never removes.
- **Preserve content verbatim.** If Ahmed pasted code, keep the code block. If he pasted a quote, keep the quote.
- **One entry per invocation.** If Ahmed pastes 3 things, ask if they should be 3 entries or 1.
- **Never post to external systems.** No GitHub API calls, no Slack, no email. Local file only.
- **Never load or interpret images.** If Ahmed adds an image ref, store the path + description. Don't OCR or analyze.

---

## Boundary vs adjacent tools

| Tool | Does what | data does what |
|---|---|---|
| `update-github-ticket` | Tracks progress vs current ticket, writes ticket-update.md | Stores ticket text/context for agents to read |
| `clean-comments` | Tidies comments.md (progress log) | Different file — data.md self-tidies |
| `enclaivetech` / `owudocs` | Fetch external docs live via WebFetch | Stores pasted external content locally |
| `enclaiveask` | Answers teammate questions in plain language | Stores teammate notes for that answer to draw from |
| `previous-step` | Session snapshot + commit | Not related — data.md persists between sessions |

No overlap with any existing tool.

---

## Consumers (read-only)

Every agent + skill can read `data.md`. Fastest path: grep the section anchor they care about:

| Consumer | Reads section(s) |
|---|---|
| reviewer | 🎫 Tickets · 👥 Team notes |
| planner | 💡 Ideas · 🎫 Tickets · 👥 Team notes |
| implementer | 🎫 Tickets · 👥 Team notes (Seb approvals) |
| infra | 👥 Team notes (Ion cluster changes) · 🔧 Config |
| improve / improveclaude | 💡 Ideas |
| baby-explain | 📚 Referenced docs |
| enclaiveask | 👥 Team notes |
| coach | Full skim to route right |

Every agent should treat `.claude/docs/data.md` as always-relevant context. CLAUDE.md router points to it.

---

## Example — ingesting a Seb approval

Ahmed: "data: seb approved the vllm migration, said we should keep ollama fallback for 2 weeks"

```
Mode: DATA · classify

Section: 👥 Team notes (person = Seb + instruction)

Adding:

### 2026-07-07 — Seb — vLLM migration approved, keep Ollama fallback 2 weeks

seb approved the vllm migration, said we should keep ollama fallback for 2 weeks

File: .claude/docs/data.md now at 47 lines.
```

---

## Example — ingesting a ticket

Ahmed: "/data #47 https://github.com/enclaive/garnet/issues/47 — add prometheus metrics for PII"

```
Mode: DATA · classify

Section: 🎫 Tickets (URL + issue #)

Ref: #47
Summary: add prometheus metrics for PII

Adding:

### 2026-07-07 — #47 — add prometheus metrics for PII

https://github.com/enclaive/garnet/issues/47

<full ticket body if Ahmed pastes it>

File: .claude/docs/data.md now at 53 lines.
```

---

## Example — ambiguous input

Ahmed: "data: OLLAMA_URL should be http://ollama:11434, not localhost"

```
Mode: DATA · classify

Ambiguous — could be config (🔧) or a team decision (💡).

Which section?
[5] 💡 Ideas & decisions
[6] 🔧 Config / API notes
```

Ahmed picks. Skill adds under chosen section.
