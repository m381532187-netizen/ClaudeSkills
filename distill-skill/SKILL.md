---
name: distill-skill
description: Extracts a reusable Claude Code skill from work already done in the current session — scripts written, workflows executed, multi-step procedures completed. Crystallizes the pattern into a SKILL.md file and archives related scripts/outputs into the skill directory. Activates on /distill-skill or when user says "turn this into a skill", "save this as a skill", "make this reusable", "extract a skill from what we just did", "package this workflow", "把这个变成技能", "把我们刚做的存成技能", "帮我把这个流程做成skill", "提炼成技能", "保存为技能", "归档成技能".
---

# Distill Skill

A post-work crystallization tool. When activated, Claude scans the current conversation to find a completed workflow or behavior pattern, extracts its reusable structure (triggers, steps, rules, anti-patterns, scripts), and produces a self-contained skill bundle — SKILL.md plus any associated scripts and example outputs — organized under `~/.claude/skills/[name]/`.

The key difference from `/create-skill`: the information already exists in the session. This skill's job is **extraction and crystallization**, not design from blank.

**Nothing is written to disk until the user confirms.**

## When This Skill Activates

**Explicit:**
- `/distill-skill [optional-name]`

**Intent detection — activate when user says things like:**
- "Turn this into a skill" / "Save this as a skill"
- "Make this workflow reusable" / "Package this as a skill"
- "Extract a skill from what we just did"
- "I want to be able to do this again later"
- "Archive this workflow"
- "把这个变成技能" / "把我们刚做的存成技能"
- "帮我把这个流程做成skill" / "提炼成技能" / "保存为技能" / "归档成技能"

## Skills Root Path Resolution

**Always resolve the actual path before writing.** Do not assume `~` works in the Write tool.

1. Use `Glob` with pattern `**` and path `~/.claude/skills` to check if the directory is reachable
2. If Glob succeeds: use `~/.claude/skills/` as the base path
3. If Glob fails: run `Bash` → `echo ~` to get the home directory, then construct `[HOME]/.claude/skills/`
4. If skills directory doesn't exist: warn the user — "I couldn't find `~/.claude/skills/`. Is Claude Code installed? This directory is created automatically on first run."
5. Store the resolved absolute path as `SKILLS_ROOT` and use it for all file writes

**Platform notes:**
- Windows (bash): `~` → `/c/Users/[username]` → write to `C:/Users/[username]/.claude/skills/`
- macOS/Linux: `~` → `/home/[username]` or `/Users/[username]`

## Skill Bundle Directory Structure

Every skill created by this tool is a self-contained bundle:

```
[SKILLS_ROOT]/[name]/
├── SKILL.md            # Skill definition (always created)
├── scripts/            # Scripts from the session (if any)
│   ├── [script].py
│   └── [script].sh
└── examples/           # Sample outputs for reference (if any)
    └── [sample-output]
```

## Extraction Workflow

Execute in this exact order:

### Step 1 — Resolve skills root

Run the path resolution procedure above. Store `SKILLS_ROOT`.

### Step 2 — Scan the session

Read back through the current conversation and identify:

| What to find | How to recognize it |
|-------------|-------------------|
| **Completed work** | The end result or artifact produced |
| **Fixed-order steps** | Steps that happened in a deliberate sequence |
| **Judgment calls** | Decisions Claude made that could be rules |
| **User corrections** | Anything the user corrected → future anti-patterns |
| **Scripts written** | Files created via Write tool with code content |
| **Non-obvious workarounds** | Flags, env vars, path quirks discovered during the work |
| **Output files** | Results/reports that serve as useful reference examples |

If the conversation contains multiple distinct workflows, ask: "I see [N] workflows in this session: [list]. Which one should I distill?" before continuing.

### Step 3 — Show extraction summary (first confirmation gate)

Display a lightweight summary for user confirmation before writing anything:

```
## Extraction Summary

**Proposed skill name:** [name]
**Type:** Behavior / Workflow
**What it does:** [1 sentence]

**Extracted triggers:**
- Slash command: /[name]
- Intent phrases: [3-5 phrases inferred from the work context]
- Chinese variants: [2-3 phrases]

**Extracted steps:** (Workflow only)
1. [step]
2. [step]

**Extracted rules (NEVER):**
- Never [from user corrections or observed failures]

**Session artifacts to archive:**
- `scripts/[filename]` — [what it does] (core step)
- `examples/[filename]` — [what it shows] (reference)
- [skip] `[filename]` — one-off debug file, not archiving

Include all artifacts? (yes / adjust / skip all)

**Source:** [brief description of what work was distilled]

Proceed with full SKILL.md draft? (yes / adjust / cancel)
```

### Step 4 — Fill gaps (only what's missing)

After the extraction summary, ask only about information that cannot be inferred from the session:

- **Trigger phrases**: if the work was done without explicit invocation language, ask: "What would you say to trigger this in a future session? Give me 5–10 natural phrases."
- **Skill name**: if not provided and not obvious from context, ask
- **Scope boundary**: if the workflow edges are unclear, ask: "What should this skill explicitly NOT handle?"

Do NOT re-ask about steps, rules, or behavior directly observed in the session.

### Step 5 — Classify

| Type | Indicators in the session |
|------|--------------------------|
| **Behavior** | Claude changed response style, questioning pattern, or output structure. No files created, no APIs called. |
| **Workflow** | Files were written, APIs called, multi-step procedure with definite start and end. Scripts were produced. |

### Step 6 — Draft SKILL.md internally

Build the full SKILL.md using extracted content:

- `description` frontmatter: contains ALL trigger phrases (English + Chinese)
- "When This Skill Activates": explicit command + intent detection list
- Steps are numbered if Workflow
- Anti-patterns come from observed corrections and failures in the session
- **Examples come from the actual work done** — use real inputs/outputs from the conversation, generalized (see Extraction vs Transcription rules below)
- Technical Notes section: lists all `scripts/` files and their purpose
- Platform-specific workarounds (e.g., PYTHONUTF8=1) are embedded as rules, not one-off comments

### Step 7 — Quality checklist (silent)

Verify all 7 items before showing the preview. Fix failures silently:

- [ ] `description` frontmatter contains every phrase from "Intent detection"
- [ ] Intent detection phrases are user-phrasing ("run a YouTube search for X"), not technical descriptions ("execute yt-dlp with --dump-json")
- [ ] At least 3 Chinese trigger variants included
- [ ] Every Anti-Pattern starts with "Never..."
- [ ] At least one example (generalized from actual session work)
- [ ] **Workflow**: explicit confirmation gate before any file write or irreversible action
- [ ] **Behavior**: explicit activation and deactivation mechanism

### Step 8 — Preview and confirm (second confirmation gate)

Display the full proposed SKILL.md in a code block, then ask:

```
Write to [SKILLS_ROOT]/[name]/SKILL.md? (yes / edit / cancel)
```

### Step 9 — Write files

On "yes":
1. Check via Glob if `[SKILLS_ROOT]/[name]/` already exists — if so, warn before overwriting any file
2. Write `SKILL.md` to `[SKILLS_ROOT]/[name]/SKILL.md`
3. For each script in the archive list: write to `[SKILLS_ROOT]/[name]/scripts/[filename]`
   - Prepend a comment block at the top of each script file noting the skill it belongs to and any non-obvious flags/workarounds discovered during the session
4. For each example file in the archive list: write to `[SKILLS_ROOT]/[name]/examples/[filename]`
5. All paths must be resolved absolute paths (never bare `~`)

### Step 10 — Report

```
Skill bundle written to [SKILLS_ROOT]/[name]/

  [SKILLS_ROOT]/[name]/SKILL.md
  [SKILLS_ROOT]/[name]/scripts/[file] (if any)
  [SKILLS_ROOT]/[name]/examples/[file] (if any)

Distilled from: [source description]
Test it: /[name] [example input from the session]
Reload required: restart Claude Code or reload the window for the skill to be discovered.
```

## Extraction vs Transcription

The goal is not to record what happened — it is to extract what is **generalizable**.

| Transcription (wrong) | Distillation (right) |
|-----------------------|---------------------|
| "We searched YouTube for 'Claude Code Skills'" | "Search YouTube for the user's specified topic using yt-dlp" |
| "User said to use PYTHONUTF8=1" | "On Windows, prefix all CLI commands with `PYTHONUTF8=1` to prevent GBK encoding errors on Unicode output" |
| "Claude asked 3 clarifying questions" | "Ask 3–5 gap-filling questions before proceeding; skip any already answered in the brief" |
| "Notebook was named 'Claude Code知识库'" | "Create a notebook with the user's specified name" |
| "Used H:/Project/python/python.exe" | "Use the Python executable at the path specified in user's environment (check `which python` or ask if unknown)" |

Extract the **rule**, not the **instance**. Extract the **pattern**, not the **value**.

## Asset Identification Rules

When scanning the session for archivable assets:

**Archive into `scripts/`:**
- Files written via Write tool with code content (.py, .sh, .js, .ps1, etc.)
- The file is a core step in the workflow (not just a debugging aid)
- The script would be needed to re-run the workflow in a future session

**Archive into `examples/`:**
- Output files that represent a successful run (search results, generated reports, processed data)
- Content is useful as a reference template for the next run
- Content is not sensitive (no API keys, passwords, PII)

**Do not archive:**
- Temporary or one-off debug files
- Files with hardcoded secrets or credentials
- Files whose content is entirely specific to this one-time run (no generalizable value)
- Files the user explicitly said to skip

## Anti-Patterns (NEVER)

- Never re-interview the user about information already visible in the session
- Never transcribe specific values (filenames, URLs, usernames) as if they are reusable rules — generalize them
- Never skip the extraction summary — the user must confirm scope and artifact list before full draft
- Never include sensitive content from the session (API keys, passwords, personal data) in any skill file
- Never write files before the user confirms the full SKILL.md preview
- Never invent steps or rules that didn't appear in the session — flag as open questions instead
- Never use bare `~` in Write tool calls — always resolve to absolute path first

## Examples

### Example 1 — Distilling a YouTube research + NotebookLM workflow

**User** (after a research session): `把我们刚做的youtube搜索和notebooklm流程保存成技能`

**Claude scans session and shows:**

```
## Extraction Summary

**Proposed skill name:** yt-research
**Type:** Workflow
**What it does:** Searches YouTube for a topic, retrieves video info, creates a NotebookLM notebook with the results as sources, and generates a summary artifact.

**Extracted triggers:**
- Slash command: /yt-research [topic]
- Intent phrases: "research YouTube for X", "find videos about X and summarize them", "YouTube research on X"
- Chinese variants: "搜索Youtube关于X", "帮我研究X的youtube内容", "Youtube调研"

**Extracted steps:**
1. Run yt-dlp search for topic, collect top N results
2. For each video: retrieve transcript or metadata
3. Create NotebookLM notebook with video URLs as sources
4. Generate summary artifact

**Extracted rules (NEVER):**
- Never fetch more than 10 videos without asking (rate limit / cost)
- Never skip prefixing notebooklm commands with PYTHONUTF8=1 on Windows

**Session artifacts to archive:**
- `scripts/yt_search.py` — YouTube search script (core step)
- [skip] inline test output — not a reusable file

Include all artifacts? (yes / adjust / skip all)

**Source:** YouTube + NotebookLM research session in this conversation

Proceed with full SKILL.md draft? (yes / adjust / cancel)
```

---

### Example 2 — Windows workaround embedded as rule in script

When a Windows encoding fix was discovered during the session, the archived script file gets a header comment:

```python
#!/usr/bin/env python3
# Skill: yt-research | scripts/yt_search.py
# IMPORTANT (Windows): Run this script with PYTHONUTF8=1 prefix to prevent
# GBK codec errors on Unicode output (checkmarks, Chinese characters, etc.)
# Example: PYTHONUTF8=1 python scripts/yt_search.py "search topic"
```

---

### Example 3 — Behavior skill distillation

**User** (after a session where Claude was corrected multiple times about verbosity): `把这个会话里我们确定的回应风格存成技能`

**Claude:** This is a Behavior skill — it changes how Claude responds, produces no files. Extraction summary would show response style rules extracted from user corrections, no scripts to archive.

## Relationship to Other Skills

- **`/create-skill`**: use when you have an idea but haven't done the work yet — designs from blank via interview
- **`/distill-skill`**: use after work is complete — extracts from what already happened
- After distilling, consider `/grill-me` to stress-test whether the extracted rules are actually correct and complete
- The skill you create can be tested immediately; it becomes available after Claude Code reloads
