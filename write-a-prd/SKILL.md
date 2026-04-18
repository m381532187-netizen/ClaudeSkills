---
name: write-a-prd
description: Interview-driven PRD authoring workflow. Claude interviews the user to fill gaps, optionally explores the codebase, then produces a structured PRD saved as a local Markdown file ready for /prd-to-issues decomposition. Activates on /write-a-prd or /prd or when user says "write a PRD", "create a PRD", "draft a requirements doc", "help me spec this out", "turn this into a PRD", "write up a spec for", "write a product requirements document", "帮我写PRD", "写需求文档", "帮我整理需求", "起草规格文档", "写个PRD".
---

# Write a PRD

An interview-driven PRD authoring workflow. When activated, Claude collects an initial brief from the user, identifies information gaps, asks 3–5 targeted questions (maximum two rounds), optionally explores the current codebase for context, then produces a structured 8-section PRD saved to `docs/[feature-name]-prd.md`. The resulting file is designed to feed directly into `/prd-to-issues` for GitHub issue decomposition.

**Nothing is written to disk until the full PRD draft is complete.**

## When This Skill Activates

**Explicit:**
- `/write-a-prd`
- `/prd`

**Intent detection — activate when user says things like:**
- "Write a PRD for X"
- "Create a PRD"
- "Draft a requirements doc"
- "Help me spec this out"
- "Turn this into a PRD"
- "Write up a spec for X"
- "帮我写PRD" / "写需求文档" / "帮我整理需求" / "起草规格文档" / "写个PRD"

## The Interview Framework

After reading the brief, check each gap category. Only ask about categories with genuine gaps.

| Category | What is missing | Example question |
|----------|----------------|------------------|
| **Who** | Users not identified | "Who are the primary users of this?" |
| **Why now** | No trigger or priority context | "What's driving this now?" |
| **Success** | No measurable definition of done | "How will you know this shipped successfully?" |
| **Scope boundary** | What is NOT included is unclear | "What is explicitly out of scope?" |
| **Technical constraints** | No platform/performance constraints | "Are there technical constraints to flag?" |
| **Dependencies** | Blocking/blocked work not mentioned | "Does this block or depend on other work?" |

### Interview Rules

- **Always use the `AskUserQuestion` tool to ask gap questions** — do not ask via plain text. One `AskUserQuestion` call per interview round, batching all questions for that round.
- Never ask more than 5 questions in one round
- Never ask a question already answered in the brief
- Maximum two interview rounds before drafting
- Flag remaining unknowns as Open Questions in the PRD

## The 8 PRD Sections

1. **Overview** — 2–3 sentence executive summary
2. **Problem Statement** — what is broken/missing today and why it matters now
3. **Goals and Success Metrics** — table of goals and measurable success signals
4. **User Stories** — 5–10 stories in "As a [role], I want [action], so that [benefit]" format
5. **Implementation Scope** — feature areas (each becomes an Epic in `/prd-to-issues`)
6. **Out of Scope** — minimum 3 explicit exclusions
7. **Technical Notes** — architecture decisions, constraints, affected areas, migration notes, open risks
8. **Open Questions** — unresolved decisions with owner and needed-by date

## Output

Save to `docs/[feature-name]-prd.md` in the current working directory.

After saving, output exactly:
```
PRD saved to docs/[feature-name]-prd.md

Run `/prd-to-issues owner/repo` to decompose into GitHub issues.
```

If invoked with `--github`: also create a GitHub Issue via `mcp__github__issue_write`.

## Anti-Patterns (NEVER)

- Never ask gap questions via plain text — always use `AskUserQuestion`
- Never write the PRD before asking gap questions
- Never include file paths, class names, or function signatures in the PRD body
- Never leave Out of Scope empty or thin
- Never proceed past two interview rounds
- Never ask a question already answered in the brief
