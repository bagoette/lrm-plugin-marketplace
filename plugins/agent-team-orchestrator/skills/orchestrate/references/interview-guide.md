# Requirements Interview Guide

Use this as a structured reference for the `requirements-analyst` agent. Don't ask every question — read the task context first and ask only what's genuinely unknown.

---

## Priority 1: Core Questions (always resolve if missing)

These three questions define the minimum viable requirements. Don't proceed without answers.

| Question | Why it matters |
|----------|---------------|
| What is the **end goal**? What should exist when this is done? | Defines scope and deliverable |
| Who are the **users or consumers** of the output? | Shapes interface, format, and quality bar |
| What are the **success criteria**? How will we know it's done correctly? | Prevents "done but wrong" outcomes |

---

## Priority 2: Scope and Integration (ask for complex or ambiguous tasks)

| Question | Why it matters |
|----------|---------------|
| What is explicitly **out of scope**? | Prevents scope creep |
| Does this need to **integrate with** existing systems? Which ones? | Surfaces hidden dependencies |
| What **tech stack, language, or framework** should be used — or must be avoided? | Prevents incompatible choices |
| What **must not change** (existing APIs, file formats, behavior)? | Protects backwards compatibility |

---

## Priority 3: Constraints (ask if not obvious from context)

| Question | Why it matters |
|----------|---------------|
| Any **performance or scale** requirements? (response time, file size, user load) | Affects architecture choices |
| Any **security or compliance** requirements? | Surfaces non-negotiables early |
| Are there **deadlines** that affect approach? (quick MVP vs. production-grade) | Shapes quality/speed tradeoff |

---

## Priority 4: Preferences (ask last)

| Question | Why it matters |
|----------|---------------|
| Any **style preferences** or conventions to follow? | Match existing codebase patterns |
| **Minimal change** (patch) vs. **clean approach** (refactor/rewrite)? | Frames implementation strategy |
| Any **past attempts** at this? What failed? | Avoids repeating mistakes |

---

## Batching Strategy

Ask in rounds of 2-4 questions maximum:

**Round 1**: Goal + users + success criteria (Priority 1)
**Round 2** (if needed): Scope + key constraints (Priority 2-3)
**Round 3** (if needed): Preferences and edge cases (Priority 4)

Stop asking when you have enough to write a complete requirements document. Mark genuine unknowns as **Open Questions** rather than guessing.

---

## Red Flags That Need Clarification

Stop and ask when you encounter:
- "Make it better" — better how? By what measure?
- "Add AI to it" — what should the AI do? What inputs/outputs?
- "Refactor this" — what problem is the refactor solving?
- "Like [other tool]" — which specific behavior of that tool?
- No mention of users — who will use this? Internal dev tool vs. end-user product shapes everything

---

## Confirming Requirements

End the interview with a confirmation step:
> "Here's what I have. Does this capture what you want, or are there corrections?"

Then output the structured requirements document. Don't proceed without confirmation.
