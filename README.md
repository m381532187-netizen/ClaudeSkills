# ClaudeSkills

A curated collection of [Claude Code](https://claude.ai/code) skills — reusable slash commands that extend Claude's capabilities for AI-assisted development workflows.

## What are Skills?

Skills are Markdown files (with optional helper scripts) that give Claude structured instructions for specific tasks. Install them into `~/.claude/skills/<skill-name>/SKILL.md` and Claude Code will automatically detect and activate them.

## Skills in this Collection

### `/grill-me` — Critical Design Review

> Forces critical questioning of a design, plan, or approach before any implementation. Like a senior engineer peer review.

**Activates on:** `/grill-me`, or when you say things like "challenge my approach", "poke holes in this", "stress test this idea".

**What it does:** Asks 3–5 pointed, specific questions before doing anything else. No implementation, no suggestions — just the hard questions that expose gaps in your design.

**Best for:** Architecture decisions, technical proposals, complex refactors.

---

### `/notebooklm` — Google NotebookLM Automation

> Complete programmatic access to Google NotebookLM, including features not in the web UI.

**Activates on:** `/notebooklm`, or intent like "create a podcast about X", "generate a quiz from my research".

**What it does:** Create notebooks, add sources (URLs, YouTube, PDFs, audio, video, images), generate all artifact types (podcasts, videos, slide decks, quizzes, flashcards, mind maps), and download results in multiple formats.

**Requires:** `pip install notebooklm-py` and `notebooklm login`.

---

### `/prd-to-issues` — PRD → GitHub Issues

> Decomposes a PRD into a structured Epic → Task hierarchy and creates GitHub issues via the GitHub MCP.

**Activates on:** `/prd-to-issues owner/repo`, or "break this PRD into issues", "把PRD拆成Issues".

**What it does:**
1. Reads a PRD (pasted text or file)
2. Decomposes it into Epics (major features) and Tasks (implementable units)
3. Shows a scannable preview for confirmation
4. Creates all issues in GitHub and links Tasks as sub-issues under Epics

**Nothing is created until you confirm.**

---

### `/pd` — Progressive Disclosure

> Changes HOW Claude outputs information — answers arrive in ordered layers, revealing depth only when requested.

**Activates on:** `/pd [question]`, `/pd on`, `/pd off`, or when you say "be concise", "just give me the answer", "直接说结论".

**What it does:** Structures responses in three layers:
- **L1 (Core):** The answer in ≤3 sentences — always shown
- **L2 (Context):** Why L1 is correct — shown on request
- **L3 (Detail):** Full depth, alternatives, edge cases — shown on request

**Best for:** When you want answers without surrounding noise.

---

### `/write-a-prd` — Interview-Driven PRD Authoring

> Interviews you to fill gaps, then produces a structured 8-section PRD saved as a Markdown file.

**Activates on:** `/write-a-prd`, `/prd`, or "help me spec this out", "帮我写PRD".

**What it does:**
1. Collects your initial brief
2. Identifies gaps (who, why now, success metrics, scope boundary, technical constraints, dependencies)
3. Asks 3–5 targeted questions (max two rounds)
4. Produces a complete PRD saved to `docs/[feature-name]-prd.md`

The output feeds directly into `/prd-to-issues`.

---

### `yt-search` — YouTube Search

> Search YouTube and return structured video results with views, subscriber counts, and engagement ratios.

**Activates on:** `/yt-search` or intent like "search YouTube for X".

**What it does:** Uses `yt-dlp` to search YouTube and returns structured results including channel subscriber counts, view counts, views/subscribers ratio, duration, and upload date. Supports date filtering (`--months N`) and result count control (`--count N`).

**Requires:** `pip install yt-dlp`.

---

### `/create-skill` — Skill Authoring from Scratch

> Guides Claude through designing and writing a new skill from scratch. All skill design knowledge is embedded — no web access needed.

**Activates on:** `/create-skill [name]`, or "make a new skill", "帮我创建skill", "设计一个技能".

**What it does:**
1. Interviews you to capture purpose, triggers, core logic, and hard limits (max two rounds)
2. Classifies the skill as Behavior (changes how Claude responds) or Workflow (multi-step with artifacts)
3. Runs a 7-item quality checklist silently before showing a preview
4. Resolves the correct `~/.claude/skills/` path automatically — works even without web access
5. Writes the file only after you confirm

**Best for:** Building a new skill when you have an idea but haven't done the work yet.

---

### `/distill-skill` — Skill Extraction from Completed Work

> Extracts a reusable skill from a workflow you just completed — scripts written, steps taken, corrections made. Produces a self-contained skill bundle including the SKILL.md and any associated scripts.

**Activates on:** `/distill-skill [name]`, or "turn this into a skill", "把我们刚做的存成技能", "提炼成技能".

**What it does:**
1. Scans the current conversation to identify completed workflows, scripts written, and user corrections
2. Shows an extraction summary (skill name, steps, NEVER rules, artifacts to archive) for lightweight confirmation
3. Drafts a full SKILL.md generalized from real session content — extracts *rules*, not *instances*
4. Archives scripts into `scripts/` and sample outputs into `examples/` under the skill directory
5. Resolves `~/.claude/skills/` path automatically; writes only after full SKILL.md preview is confirmed

**Skill bundle structure:**
```
~/.claude/skills/[name]/
├── SKILL.md
├── scripts/   ← scripts from the session
└── examples/  ← sample outputs for reference
```

**Best for:** After completing a multi-step task — lock in the workflow so you never have to rediscover it.

---

## Installation

### Install all skills

```bash
# Clone this repo
git clone https://github.com/m381532187-netizen/ClaudeSkills.git

# Copy skills to Claude's skills directory
cp -r ClaudeSkills/grill-me ~/.claude/skills/
cp -r ClaudeSkills/notebooklm ~/.claude/skills/
cp -r ClaudeSkills/prd-to-issues ~/.claude/skills/
cp -r ClaudeSkills/progressive-disclosure ~/.claude/skills/
cp -r ClaudeSkills/write-a-prd ~/.claude/skills/
cp -r ClaudeSkills/yt-search ~/.claude/skills/
cp -r ClaudeSkills/create-skill ~/.claude/skills/
cp -r ClaudeSkills/distill-skill ~/.claude/skills/
```

### Install a single skill

```bash
cp -r ClaudeSkills/<skill-name> ~/.claude/skills/
```

### Windows (PowerShell)

```powershell
$skills = @("grill-me", "notebooklm", "prd-to-issues", "progressive-disclosure", "write-a-prd", "yt-search", "create-skill", "distill-skill")
foreach ($s in $skills) {
    Copy-Item -Recurse "ClaudeSkills\$s" "$env:USERPROFILE\.claude\skills\"
}
```

---

## Skill Relationships

```
                    ┌─────────────────────────────────────┐
                    │         Skill Authoring              │
                    │  /create-skill   /distill-skill      │
                    │  (from idea)     (from session work) │
                    └──────────────┬──────────────────────-┘
                                   │ produces
                                   ▼
/grill-me  →  /write-a-prd  →  /prd-to-issues
                                     ↓
                              GitHub Issues

/pd          (use alongside any skill for concise output)
/yt-search   (standalone research tool)
/notebooklm  (standalone knowledge management tool)
```

**Recommended workflow:**
1. `/grill-me` — stress-test your idea before writing specs
2. `/write-a-prd` — produce a structured requirements document
3. `/prd-to-issues` — decompose into GitHub Epics and Tasks
4. `/create-skill` or `/distill-skill` — capture new workflows as reusable skills

---

## Requirements

| Skill | Requirements |
|-------|--------------|
| grill-me | None |
| notebooklm | `pip install notebooklm-py`, Google account |
| prd-to-issues | GitHub MCP server configured |
| progressive-disclosure | None |
| write-a-prd | None (GitHub MCP optional for `--github` flag) |
| yt-search | `pip install yt-dlp` |
| create-skill | None (all knowledge embedded) |
| distill-skill | None (all knowledge embedded) |

---

## Related Projects

- [ClineSkills](https://github.com/m381532187-netizen) — Curated Cline AI rules and workflows
- [Claude Code](https://claude.ai/code) — The CLI this skill collection is built for
- [notebooklm-py](https://github.com/teng-lin/notebooklm-py) — The library powering the NotebookLM skill
