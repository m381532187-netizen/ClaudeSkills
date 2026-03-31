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
```

### Install a single skill

```bash
cp -r ClaudeSkills/<skill-name> ~/.claude/skills/
```

### Windows (PowerShell)

```powershell
$skills = @("grill-me", "notebooklm", "prd-to-issues", "progressive-disclosure", "write-a-prd", "yt-search")
foreach ($s in $skills) {
    Copy-Item -Recurse "ClaudeSkills\$s" "$env:USERPROFILE\.claude\skills\"
}
```

---

## Skill Relationships

```
/grill-me  →  /write-a-prd  →  /prd-to-issues
                                     ↓
                              GitHub Issues

/pd  (can be used alongside any skill for concise output)
/yt-search  (standalone research tool)
/notebooklm  (standalone knowledge management tool)
```

**Recommended workflow:**
1. `/grill-me` — stress-test your idea before writing specs
2. `/write-a-prd` — produce a structured requirements document
3. `/prd-to-issues` — decompose into GitHub Epics and Tasks

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

---

## Related Projects

- [ClineSkills](https://github.com/m381532187-netizen) — Curated Cline AI rules and workflows
- [Claude Code](https://claude.ai/code) — The CLI this skill collection is built for
- [notebooklm-py](https://github.com/teng-lin/notebooklm-py) — The library powering the NotebookLM skill
