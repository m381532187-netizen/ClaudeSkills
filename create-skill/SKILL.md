---
name: create-skill
description: Guides Claude through designing and writing a new Claude Code skill (SKILL.md file) from scratch. All skill design knowledge is embedded — no web access needed. Activates on /create-skill or when user says "make a new skill", "create a skill", "build a skill for", "help me write a skill", "design a skill", "add a skill", "new skill", "新建一个技能", "帮我创建skill", "写个skill", "设计一个技能", "做一个技能".
---

# Create Skill

A meta-skill for building Claude Code skills. When activated, Claude interviews the user to understand the desired behavior, classifies the skill type (Behavior vs Workflow), drafts a complete SKILL.md, runs a silent quality check, shows a preview, and writes the file only after confirmation.

**All skill design knowledge is embedded here — no web access required.**

**Nothing is written to disk until the user confirms.**

## When This Skill Activates

**Explicit:**
- `/create-skill [name]`
- `/new-skill [name]`

**Intent detection — activate when user says things like:**
- "Make a new skill" / "Create a skill" / "Add a skill"
- "Build a skill for X" / "Help me write a skill"
- "Design a skill" / "I want a skill that..."
- "新建一个技能" / "帮我创建skill" / "写个skill"
- "设计一个技能" / "做一个技能"

## Skills Root Path Resolution

Skills live at `~/.claude/skills/[name]/SKILL.md`. On Windows the tilde expands to `C:\Users\[username]`, so the canonical path is `C:/Users/[username]/.claude/skills/`. **Always resolve the actual path before writing** — do not assume `~` works in the Write tool.

**Path resolution procedure (run once at start of Creation Workflow):**

1. Use `Glob` with pattern `**` and path `~/.claude/skills` to check if the directory is reachable
2. If Glob succeeds: use `~/.claude/skills/` as the base path
3. If Glob fails or returns empty: run `Bash` → `echo ~` to get the home directory, then construct `[HOME]/.claude/skills/`
4. If the skills directory doesn't exist at all: warn the user — "I couldn't find a `~/.claude/skills/` directory. Is Claude Code installed? The skills directory should be created automatically when Claude Code first runs."
5. Store the resolved absolute path and use it for all file writes (never use bare `~` in Write tool calls)

**Common resolved paths by platform:**
- Windows (bash): `~` → `/c/Users/[username]` → write to `C:/Users/[username]/.claude/skills/`
- macOS/Linux: `~` → `/home/[username]` or `/Users/[username]` → write to `~/.claude/skills/`

## SKILL.md Format Reference

Every skill lives at `[resolved-skills-root]/[name]/SKILL.md`. The structure is:

```
---
name: skill-name-in-kebab-case
description: One paragraph containing ALL trigger phrases. Format: "[Brief purpose]. Activates on /command or when user says [phrase 1], [phrase 2], [phrase N]."
---

# Skill Title

One-paragraph overview of what the skill does and why.

## When This Skill Activates

**Explicit:**
- `/command` or `/command [args]`

**Intent detection — activate when user says things like:**
- "English trigger phrase"
- "中文触发短语"

## [Core Framework or Workflow Name]

[Main content — framework for behavior skills, or step-by-step procedure for workflow skills]

## Anti-Patterns (NEVER)

- Never [specific prohibited action]
- Never [specific prohibited action]

## Examples

[At least one complete example: full user input + full Claude response]

## Relationship to Other Skills

[Optional: how this skill connects to others in the library]
```

**Critical rule:** Every phrase listed under "Intent detection" MUST also appear verbatim in the `description` frontmatter field. The frontmatter is what Claude Code uses to load the skill — if a phrase is missing from `description`, the skill will never activate for that phrase.

## Skill Type Classification

Classify the new skill BEFORE drafting. Ask yourself three questions:

1. Does activating this skill change HOW Claude thinks or responds (style, structure, questioning mode)?
2. Does it create files, GitHub issues, send messages, or call external APIs?
3. Does it need a "confirm before acting" step?

| Type | Criteria | Examples |
|------|---------|---------|
| **Behavior** | Changes HOW Claude responds. No external side effects. No file writes. No MCP calls. | grill-me, progressive-disclosure |
| **Workflow** | Multi-step procedure that produces artifacts or calls external tools. Has a definite start, steps, and end state. | prd-to-issues, write-a-prd |

If questions 2 or 3 are "yes" → Workflow skill.

## The Design Interview

After reading the user's initial idea, ask ALL Phase 1 questions in a single numbered list. Skip any question already answered in the brief.

### Phase 1 — Core Design (always ask, one message)

1. **Purpose**: In one sentence, what job does this skill do that Claude doesn't do by default?
2. **Type**: Does it change HOW Claude responds (Behavior), or does it run a multi-step procedure with artifacts or side effects (Workflow)?
3. **Triggers**: What slash command should invoke it? Give me 5–10 phrases a user might say naturally that should trigger it — including Chinese variants.
4. **Core logic**: Walk me through step-by-step what happens from trigger to completion.
5. **Hard limits**: What are the 3 most important things this skill must NEVER do?

### Phase 2 — Refinement (only if gaps remain after Phase 1)

Ask only about genuinely missing information:

- **For Behavior skills**: "What does Claude do differently in this mode vs. normal? Give me a before/after example."
- **For Workflow skills**: "What inputs does it need? What artifact does it produce? Which step requires user confirmation before any irreversible action?"
- **For both**: "What example would best show this skill working correctly?"

**Maximum two interview rounds.** After the second round, draft with remaining unknowns flagged as open questions inside the skill.

## Drafting Rules

### For Behavior Skills

- Lead overview with the mode's purpose in one sentence
- Define activation modes explicitly: per-request? session toggle (`/cmd on` / `/cmd off`)? always-on?
- Core framework section describes HOW Claude decides, not just WHAT Claude does (taxonomy, layers, decision rules)
- Include at least one complete before/after example
- Anti-Patterns section: minimum 5 "Never..." entries, each specific to this skill

### For Workflow Skills

- State "Nothing is [created/written/sent] until the user confirms" in the overview paragraph
- Include an **Input Resolution** section: what inputs are needed, how to handle each missing case
- Number the workflow steps explicitly: "Execute in this exact order"
- Show the **preview format** that appears before any destructive action
- Define the **completion report** format (what Claude outputs when done)
- Include error handling for the 2–3 most likely failure modes

## Quality Checklist

Before showing the draft, verify all 7 items silently. Fix any failures before displaying the preview — do not report the checklist to the user.

- [ ] `description` frontmatter contains every phrase from the "Intent detection" list verbatim
- [ ] Intent detection phrases are user-phrasing ("break this PRD into issues"), not technical descriptions ("parse PRD content")
- [ ] At least 3 Chinese language trigger variants are included
- [ ] Every Anti-Pattern entry starts with "Never..." (not "Avoid", "Try not to", "Don't")
- [ ] At least one complete example exists (full user input + full Claude response, not a summary)
- [ ] **Workflow skills**: there is an explicit confirmation gate before any file write, API call, or other irreversible action
- [ ] **Behavior skills**: there is an explicit activation mechanism and a way to turn it off

## Creation Workflow

Execute in this exact order:

1. **Resolve skills root** — run the path resolution procedure above; store the absolute path as `SKILLS_ROOT`
2. **Read intent** — absorb the user's description; extract the skill name (ask if not provided)
3. **Classify** — determine Behavior vs Workflow using the classification table above
4. **Phase 1 interview** — ask all 5 core questions in a single message (skip answered ones)
5. **Phase 2 interview** — only if critical gaps remain; otherwise proceed
6. **Draft internally** — compose the full SKILL.md content in memory
7. **Quality checklist** — verify all 7 items; fix failures silently
8. **Preview and confirm** — display the full proposed SKILL.md in a code block, then ask:

```
Write to [SKILLS_ROOT]/[name]/SKILL.md? (yes / edit / cancel)
```

9. **Write** — on "yes", check if `[SKILLS_ROOT]/[name]/SKILL.md` already exists via Glob (warn if overwriting), then write the file using the resolved absolute path
10. **Report** — output:

```
Skill written to [SKILLS_ROOT]/[name]/SKILL.md

Test it: /[name] [example input]
Reload required: restart Claude Code or reload the window for the skill to be discovered.
```

## Common Mistakes to Avoid When Drafting

**Trigger phrase mistakes (most common failure):**
- Trigger phrases appear in the skill body but NOT in `description` frontmatter → skill never loads for those phrases
- Using technical descriptions as trigger phrases: "parse PRD content" ✗ → "break this PRD into issues" ✓
- Using first-person trigger phrases: "I want to parse this" ✗ → "parse this into issues" ✓ (write as the user would say it)
- Choosing a name that conflicts with an existing skill — check `~/.claude/skills/` before deciding

**Behavior skill mistakes:**
- No explicit deactivation mechanism — user cannot turn the mode off
- Framework describes outputs rather than decision logic — skill is vague and inconsistently applied
- No example — impossible to verify the skill works as intended

**Workflow skill mistakes:**
- No confirmation gate — destructive actions happen without review
- No input resolution — skill crashes on missing inputs instead of asking for them
- Steps are described but not numbered — Claude reorders them differently each run
- No completion report — user doesn't know what was created or where

## Examples

### Example 1 — Brief already provides enough context (Behavior skill)

**User:** `/create-skill I want a skill that forces Claude to always give the confidence level of its answer before responding`

**Claude:** Classifying as Behavior skill. I have enough context for Phase 1 except for triggers and hard limits. Asking:

> 1. What slash command should invoke this? What phrases should trigger it naturally? (Give 5–10 phrases + Chinese variants)
> 2. Should confidence level always be shown, or only when toggled on? (per-response, session mode, or always-on?)
> 3. What are the 3 things this skill must NEVER do?

---

### Example 2 — Workflow skill from scratch

**User:** `帮我创建一个skill，让Claude在帮我写代码之前，先确认它理解了需求`

**Claude:** 分类为 Behavior skill（它改变 Claude 在写代码前的行为，但不产生外部产物）。

Phase 1 问题：

> 1. 这个技能一句话的工作是什么？
> 2. 触发命令是什么？用户会说哪些话触发它？（5-10个短语，含中文）
> 3. Claude 确认理解需求的方式是什么？复述需求？提问澄清？还是两者都有？
> 4. 从触发到开始写代码，每步发生什么？
> 5. 这个技能最重要的3条 NEVER 规则？

---

*(After receiving answers, Claude drafts internally, runs quality checklist silently, then shows the full SKILL.md preview.)*

---

### Example 3 — Preview format

After interview + internal drafting + quality check, Claude shows:

````
Here's the proposed skill:

```
---
name: confirm-understanding
description: Forces Claude to restate its understanding of a coding request and confirm with the user before writing any code. Activates on /confirm or when user says "make sure you understand", "confirm you got this", "repeat back the requirements", "先确认需求", "复述一下你的理解", "确认你理解了".
---

# Confirm Understanding

A pre-implementation gate. When activated, Claude restates what it understands the user wants — including scope, constraints, and expected output — then waits for explicit confirmation before writing any code.

...
```

Write to ~/.claude/skills/confirm-understanding/SKILL.md? (yes / edit / cancel)
````

## Relationship to Other Skills

- Use `/grill-me` BEFORE designing a skill to stress-test the concept: "Is this skill solving the right problem?"
- The skill you create can feed into `/write-a-prd` if you want to document it as a specification, or into `/prd-to-issues` if you want to track its implementation as GitHub issues
- `/create-skill` is the authoring tool; the skills it produces are the runtime tools
