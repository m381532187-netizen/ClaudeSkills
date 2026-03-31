---
name: prd-to-issues
description: Decomposes a PRD into a GitHub issue hierarchy (Epics → Tasks) using the GitHub MCP. Shows a preview before creating anything. Activates on /prd-to-issues or when user says "break this PRD into issues", "create GitHub issues from this PRD", "decompose into tickets", "turn this PRD into GitHub issues", "parse PRD to issues", "把PRD拆成Issues", "把需求文档拆成Issues", "从PRD创建Issues", "分解成任务".
---

# PRD to Issues

A GitHub issue creation workflow. When activated, Claude reads a PRD (pasted text or file), decomposes it into a structured Epic → Task hierarchy, shows a scannable preview for confirmation, then uses the GitHub MCP to create all issues and link Tasks as sub-issues under their parent Epics.

**Nothing is created until the user confirms.**

## When This Skill Activates

**Explicit:**
- `/prd-to-issues [owner/repo]`

**Intent detection — activate when user says things like:**
- "Break this PRD into issues"
- "Create GitHub issues from this PRD"
- "Decompose this into tickets"
- "Turn this PRD into GitHub issues"
- "Make GitHub issues from this spec"
- "把PRD拆成Issues"
- "把需求文档拆成Issues"
- "从PRD创建Issues"
- "分解成任务"
- "帮我建Issues"

## Input Resolution

Before decomposing, resolve two inputs:

**1. The PRD content:**
- If the user pasted text in the conversation: use that text directly.
- If the user provided a file path: read the file with the Read tool.
- If neither: ask "Please paste your PRD or provide a file path."

**2. The target repository:**
- If provided as `/prd-to-issues owner/repo`: use it.
- If not provided: ask "Which GitHub repository should these issues be created in? (format: owner/repo)"
- Call `mcp__github__get_me` to confirm authentication is working before proceeding.

## Decomposition Rules

### What is an Epic?

An Epic is a major feature area or user-facing capability that:
- Is too large for a single PR — requires multiple tasks
- Represents 5–20 days of engineering effort
- Has a clear user-facing outcome that can be stated in 1–2 sentences
- Is named as a noun phrase: "User Authentication System", "Payment Integration", "Notification Service"

Target **3–10 Epics per PRD**. If you find only 1–2, the PRD is probably a single Epic. If you find more than 10, consolidate related areas.

### What is a Task?

A Task is a concrete, independently implementable unit of work that:
- Is completable by one engineer in 1–5 days
- Produces something testable or demonstrable at completion
- Is scoped to approximately one PR
- Starts with an action verb: "Implement JWT token generation", "Add login form validation", "Write migration for users table"

Target **2–8 Tasks per Epic**. Never create a single-task Epic — collapse it into the Task itself (no Epic needed).

### Size estimation:

| Label | Duration | Examples |
|-------|----------|---------|
| `size/S` | ≤ 2h | Config change, minor UI tweak, single endpoint |
| `size/M` | 2–8h | Small feature, form + validation, DB schema + migration |
| `size/L` | 1–3d | Full CRUD feature, authentication flow, integration |
| `size/XL` | > 3d | Epic-level, major subsystem, complex integration |

Epics are always `size/XL`. Tasks use `size/S` through `size/L`.

## Issue Templates

### Epic body

```
## Overview
[1-2 sentences describing what this Epic delivers and why it matters to the user.]

## Goals
- [ ] [User-facing goal 1 — written from the user's perspective]
- [ ] [User-facing goal 2]

## Out of Scope
- [What this Epic explicitly does NOT include]

## Tasks
Tracked as sub-issues.
```

### Task body

```
## Context
Part of Epic: [Epic title]. [1 sentence explaining why this Task exists within the Epic.]

## What to Build
[Specific description of what needs to be implemented.]

## Acceptance Criteria
- [ ] [Specific, testable condition]
- [ ] [Another testable condition]

## Out of Scope
[What NOT to include in this Task.]

## Dependencies
[Other task titles or external requirements. Write "None" if standalone.]
```

## The Preview Format

Present this before any issue is created:

```
## PRD → Issues Preview (owner/repo)

### Epic 1: [Title]
Labels: epic, size/XL, area/[detected]
  └── Task 1.1: [Title] — size/L, area/backend
  └── Task 1.2: [Title] — size/M, area/api

### Epic 2: [Title]
Labels: epic, size/XL, area/[detected]
  └── Task 2.1: [Title] — size/L, area/frontend

**Total: N Epics, N Tasks — Create in owner/repo? (yes / edit N / skip N / cancel)**
```

## Creation Workflow

Execute in this exact order once the user confirms:

1. **Create all Epics** via `mcp__github__issue_write` with `method: "create"`
2. **Create all Tasks** (grouped by Epic)
3. **Link Tasks as sub-issues** via `mcp__github__sub_issue_write`
4. **Report completion** with issue numbers

## Anti-Patterns (NEVER)

- Never create issues without preview confirmation
- Never make an Epic with only one Task
- Never skip the duplicate check (`search_issues` before preview)
- Never forget sub-issue linking
- Never leave "Out of Scope" sections empty
