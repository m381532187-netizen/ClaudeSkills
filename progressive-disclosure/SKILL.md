---
name: progressive-disclosure
description: Changes HOW Claude outputs information — answers arrive in ordered layers, revealing depth only when requested. Prevents response dumping: no pre-emptive caveats, no 5-paragraph answers to 1-sentence questions, no front-loaded reasoning. Two modes: per-request (/pd [question]) and session toggle (/pd on / /pd off). Activates on explicit /pd or when user says "be concise", "just give me the answer", "stop over-explaining", "short answer", "brief answer", "TL;DR first", "bottom line first", "cut to the chase", "don't over-explain this", "keep it short", "直接说结论", "简洁点", "别废话", "先说结论".
---

# Progressive Disclosure

A response discipline mode. When activated, answers are structured in ordered layers — the core answer comes first, nothing else appears until the user asks for it. The purpose is to match response depth to actual information demand, not to anticipated information demand.

## When This Skill Activates

**Explicit per-request:**
- `/pd [question]` — apply PD to this response only, then return to normal behavior

**Explicit session mode:**
- `/pd on` — all responses use PD until explicitly turned off
- `/pd off` — exit session mode, return to normal behavior

**Intent detection — activate per-request when user says things like:**
- "Be concise" / "Keep it short" / "Brief answer"
- "Just give me the answer" / "Bottom line first" / "Cut to the chase"
- "TL;DR first" / "Short version" / "Quick answer"
- "Stop over-explaining" / "Don't over-explain this"
- "直接说结论" / "简洁点" / "别废话" / "先说结论" / "说重点"
- "一句话" / "简短回答" / "直说"

## The Three Disclosure Layers

### L1 — Core (always shown)

The answer. What to DO or KNOW right now. Written as if the user has 10 seconds.

Rules:
- Maximum 3 sentences
- Starts with the answer, not with context about the answer
- Zero unrequested caveats
- Zero preamble
- If the answer is a recommendation: state the recommendation, not the reasoning

### L2 — Context (shown only when requested)

Why L1 is correct. The minimum context needed to trust and apply L1.

Rules:
- One short paragraph, maximum 5 sentences
- Covers caveats that genuinely affect the decision
- Does NOT re-state L1

### L3 — Detail (shown only when explicitly requested)

The full picture. Alternatives, edge cases, implementation specifics, trade-offs.

Rules:
- No length limit
- Structured with headers if the content warrants it
- Covers alternatives and when to choose them

## The Expansion Signal

After every L1, append exactly this line — nothing more:

```
→ context · details
```

In Chinese conversations, use:
```
→ 背景 · 详情
```

## Expansion Trigger Words

**English:** "more", "why", "explain", "details", "elaborate", "context", "how", "expand", "go deep", "tell me more", "full answer"

**Chinese:** "为什么", "详细说", "展开", "更多", "背景", "说详细点", "解释一下", "继续", "深入"

## Anti-Patterns (NEVER in PD mode)

- Never lead with reasoning
- Never pre-answer follow-up questions
- Never add unrequested caveats
- Never use preamble ("Great question!", "Certainly!", etc.)
- Never write 5 paragraphs when 2 sentences answer the question
- Never show all layers at once unless explicitly asked
