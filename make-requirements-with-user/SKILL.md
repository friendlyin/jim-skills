---
name: make-requirements-with-user
description: Turn raw user notes into a complete, structured requirements document through iterative clarification. Use when the user explicitly invokes this skill to scope a feature, bug fix, or development iteration before any code is written.
disable-model-invocation: true
---

# Make Requirements with User

A multi-cycle workflow: codebase review → draft with recommendations → user feedback → revise → finalise. Most non-trivial tasks need 2-3 cycles. Treat the draft as the source of truth; chat history is not a substitute.

## Phase checklist

1. Receive raw input.
2. Review the codebase silently.
3. Write `prd/[slug]-draft.md` with Sections A/B/C.
4. Post a short chat message and stop.
5. Iterate on user feedback in the same draft file.
6. When the user approves and Section C is empty, write `prd/[slug].md` and clear Section C / draft markers from the file (keep the discussion log in the draft for reference).

## Decision principles

Apply these in order. For every recommendation you write, end with one short `Why:` line naming the principle(s) used.

1. **Simplicity** — pick the smallest solution that solves the actual problem.
2. **User experience** — favour clear, predictable, uncluttered behaviour.
3. **Hard-coded logic beats prompt complexity** — reach for prompt edits only when the task genuinely needs LLM judgment.
4. **Investigate before deciding** — if a root cause is unverified, recommend an investigation step, not a fix.
5. **Preserve user value** — when simplifying, keep anything the end user produced or cares about.

## Handling assumptions

Label every claim as one of:

- **Confirmed** — grounded in code you read or facts the user gave.
- **Hypothesis** — plausible but unverified. Mark it ("likely cause:", "hypothesis:") and pair with an investigation step.
- **Open** — not enough info to even hypothesise. List as needing input.

A hypothesis must never become a finalised functional requirement that prescribes a code change. It belongs under "Items requiring investigation" with an investigation step first.

## Phase 2 — Codebase review

Before drafting, silently:

- Locate existing features, data models, and code paths relevant to the request.
- Identify what already exists vs. what's new.
- For any reported bug, find the suspect code and form an initial hypothesis (do not commit to it).

Questions in the draft must be grounded in what you actually found, not generic.

## Phase 3 — The draft

Write to `prd/[feature-slug]-draft.md`. The draft is the source of truth: every decision lives here in full detail, not summarised away because "we covered it in chat". A reader joining at cycle 3 should understand every decision from the file alone.

### Section A — Clarifying questions (with recommendations)

Ask about product behaviour, UX, and scope. Skip low-level technical choices (where to put a field, what to name a variable) — make those silently.

Format each item as:

<example>
### 1. Empty-state copy on the dashboard

**Question:** When a user has no projects yet, should the dashboard show a CTA to create one, or a link to the tutorial?

**Recommendation:** Show the CTA. The tutorial link can sit underneath as secondary text.

*Why:* UX — the primary action on an empty dashboard should be the one that resolves the empty state.
</example>

<example>
### 2. Root cause of duplicate notifications

**Question:** Are duplicate notifications caused by the retry loop in `notifier.ts`, or by the upstream webhook firing twice?

**Recommendation:** Investigate first. Add logging at the webhook entry point and the retry decision, reproduce once, then choose between deduping at intake or fixing the retry guard.

*Why:* Investigate before deciding — the symptom fits both causes; a fix aimed at the wrong one will regress.
</example>

If the user has already answered something in the raw notes, incorporate it directly instead of re-asking.

### Section B — Implementation considerations

Only high-consequence technical trade-offs the user should weigh in on: architecture, irreversible data decisions, security. Skip anything obvious or low-risk. Never duplicate a Section A topic. Typical length: 0-3 items.

<example>
### 1. Storage for user-uploaded transcripts

Context: transcripts can reach 5MB and are queried by substring.
Options: A) Postgres `text` + GIN trigram index. B) Object storage + separate search index.
Recommendation: A. Volume is small and search-by-substring fits Postgres well; B adds two systems for no current benefit.
Why: Simplicity.
</example>

### Section C — Open items for next cycle (optional)

For things that genuinely block finalisation and need user input. Each item must pass this test: *can you write a one-sentence question whose answer would change the spec?*

Items that belong here:

- Open disagreements.
- New questions that surfaced during this cycle.
- Bugs whose cause is still unverified.
- Trade-offs only the user can weigh.

If the user has already given clear guidance, the item is resolved — write it up in the resolved section in full detail.

<example>
### 1. Should past transcripts be backfilled?

**Question:** When we ship the new field, do we backfill existing rows or leave them null?
**Recommendation:** Backfill in a single migration; the table is small (~10k rows).
*Why:* Simplicity + preserving user value.
**Status: open — awaiting feedback**
</example>

Section C being empty is the signal that the draft is ready to finalise.

## Phase 4 — Notify and wait

Post this in chat, then stop:

```
Draft requirements at [path]. Each question has my recommended answer. Reply with:
- "approve all" to accept every recommendation,
- per-item approvals or overrides,
- or inline edits in the file.
If any items remain in Section C, please address those.
```

## Phase 5 — Iterate

Update the same `-draft.md` file in place. Show what changed in chat and stop again. Keep iterating until the user explicitly approves finalisation and Section C is empty.

When the user paraphrases a refinement in chat, translate it into the concrete spec inside the draft — exact wording, file paths, function names, example pairs. The chat paraphrase is not the spec.

If the user says "developer decides" on any item, record it as: *developer to implement using the simplest approach that satisfies the surrounding requirements*.

## Phase 6 — Finalise

Write the final document to a **new file** at `prd/[slug].md`. Keep the `-draft.md` file alongside it as the discussion log; don't delete it.

The final document is developer-facing. Include only actionable work. Omit anything decided as "do nothing", "deferred", "out of scope", rejected alternatives, and background commentary — those stay in the draft.

Template:

```
# Requirements: [Feature name]
Date: YYYY-MM-DD
Status: Ready for development

## Summary
[2-4 sentence overview]

## Scope
### In scope
### Out of scope

## Functional requirements
[Numbered, specific, testable behaviours.]

## Items requiring investigation before fix
[Bugs with unverified causes. For each: symptom, suspected cause, investigation step,
then the fix only after the cause is confirmed. Anything with a confirmed cause
goes under Functional requirements instead.]

## Non-functional requirements
[Performance, accessibility, security, compatibility — only if relevant.]

## Data & state
[Models, fields, schema changes — only if relevant.]

## Edge cases & error handling

## Open decisions
[Should be empty if Phase 6 was reached cleanly. Record any remaining
unresolved decision explicitly rather than leaving it implicit.]

## Implementation notes
[Agreed approach for any Section B items.]
```

After writing the file, post:

```
Requirements finalised at [path]. Ready to plan the implementation when you are.
```

## Operating rules

- Ground every question in code you read in Phase 2.
- Output goes to the draft file and short chat messages only — no interactive elicitation tools.
- No implementation code during this workflow.
- Prefer specificity over completeness theatre.
- If raw input is very short, note that in the draft and still produce a first pass.
- Make your applied principle visible on every recommendation so the user can audit the reasoning.
