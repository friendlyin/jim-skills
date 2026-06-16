---
name: make-requirements-with-user
description: Turn raw user notes into a complete, structured requirements document through iterative clarification. Use when the user explicitly invokes this skill to scope a feature, bug fix, or development iteration before any code is written.
disable-model-invocation: true
---

# Make Requirements with User

A multi-cycle workflow: codebase review → draft with recommendations → user feedback → revise → finalise. Most non-trivial tasks need 2-3 cycles. The document is the source of truth; chat history is not a substitute.

**One file, start to finish.** There is a single document per feature: `prd/[slug].md`. You do not create a separate `-draft` file and you never end up with two files for one feature. The same file is the working draft during cycles and the finalised spec at the end — finalising restructures it in place, it does not spawn a second file. Two files for one feature means an ambiguous source of truth and clutter in `prd/`; never do it.

**One item, one place.** Every question or decision appears exactly once, in the section it belongs to, and that single entry is where the user comments and where its status lives. Never restate a question in a second list, a summary, or a separate "open items" section — duplication wastes the user's reading time and makes it unclear which copy is authoritative. When something needs tracking across cycles, track it *on the item itself* with a status line, not by copying it elsewhere.

## Phase checklist

1. Receive raw input.
2. Review the codebase silently.
3. Write `prd/[slug].md` with a DRAFT banner and Sections A and B. Each item carries an inline `**Status:**` line.
4. Post a short chat message and stop.
5. Iterate on user feedback in the same file — update each item's status in place; append any newly-surfaced item to A or B.
6. When every item is resolved, finalise **in the same file**: prepend the clean developer-facing spec, demote the A/B Q&A into a "Decision log & rationale" section below, flip the banner.

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

Write to `prd/[feature-slug].md` (the one and only file for this feature) and open it with a banner: `Status: **DRAFT — cycle N.** Not yet finalised.` The document is the source of truth: every decision lives here in full detail, not summarised away because "we covered it in chat". A reader joining at cycle 3 should understand every decision from the file alone.

Each item in A and B is **self-contained and actionable**: it holds the question, your recommendation, and its own status. The user reads and answers it in one place; you update its status there. This single entry is the source of truth for that decision through every cycle — there is no separate open-items list to keep in sync.

### Section A — Clarifying questions (with recommendations)

Ask about product behaviour, UX, and scope. Skip low-level technical choices (where to put a field, what to name a variable) — make those silently.

Format each item as below. The `**Status:**` line starts as `open` and you flip it to `resolved` (with the decision) once the user answers — in place, on the same item.

<example>
### 1. Empty-state copy on the dashboard

**Question:** When a user has no projects yet, should the dashboard show a CTA to create one, or a link to the tutorial?

**Recommendation:** Show the CTA. The tutorial link can sit underneath as secondary text.

*Why:* UX — the primary action on an empty dashboard should be the one that resolves the empty state.

**Status:** open — awaiting feedback.
</example>

<example>
### 2. Root cause of duplicate notifications

**Question:** Are duplicate notifications caused by the retry loop in `notifier.ts`, or by the upstream webhook firing twice?

**Recommendation:** Investigate first. Add logging at the webhook entry point and the retry decision, reproduce once, then choose between deduping at intake or fixing the retry guard.

*Why:* Investigate before deciding — the symptom fits both causes; a fix aimed at the wrong one will regress.

**Status:** open — awaiting feedback.
</example>

After the user answers, the item becomes (same place, no copy made elsewhere):

<example>
### 1. Empty-state copy on the dashboard

**Question:** … **Recommendation:** … *Why:* …

**Status:** resolved — show the CTA, tutorial link as secondary text (user approved 2026-06-11).
</example>

If the user has already answered something in the raw notes, incorporate it directly instead of re-asking — write the item straight to `resolved`.

### Section B — Implementation considerations

Only high-consequence technical trade-offs the user should weigh in on: architecture, irreversible data decisions, security. Skip anything obvious or low-risk. Never duplicate a Section A topic. Typical length: 0-3 items. Each B item carries its own `**Status:**` line too.

<example>
### 1. Storage for user-uploaded transcripts

Context: transcripts can reach 5MB and are queried by substring.
Options: A) Postgres `text` + GIN trigram index. B) Object storage + separate search index.
Recommendation: A. Volume is small and search-by-substring fits Postgres well; B adds two systems for no current benefit.
Why: Simplicity.

**Status:** open — awaiting feedback.
</example>

### Tracking open vs resolved — no separate list

There is **no Section C / "open items" list.** An item's openness lives on the item itself, via its `**Status:**` line. To see what still blocks finalisation, scan for `Status: open`. Re-listing those questions in a second section is exactly the duplication this workflow forbids: it forces the user to read the same question twice and creates two places that can disagree.

- A **new** question that surfaces mid-cycle is appended to Section A (or B if it's a technical trade-off) as a normal item with `Status: open` — not collected into a separate bucket.
- An item the user has answered flips to `Status: resolved — <decision>` in place.
- The draft is **ready to finalise when no item is still `Status: open`** (and the user has approved). That replaces the old "Section C is empty" signal.

## Phase 4 — Notify and wait

Post this in chat, then stop:

```
Draft requirements at [path]. Each item has my recommended answer and a Status line. Reply with:
- "approve all" to accept every recommendation,
- per-item approvals or overrides,
- or inline edits in the file.
Items still marked "Status: open" are the ones needing your input.
```

## Phase 5 — Iterate

Update the same `prd/[slug].md` file in place. Show what changed in chat and stop again. Keep iterating until the user explicitly approves finalisation and no item is still `Status: open`.

Update each answered item's `**Status:**` line in place; never copy a question into a tracking list. If a new question surfaces, append it to Section A or B as a normal `Status: open` item.

When the user paraphrases a refinement in chat, translate it into the concrete spec inside the draft — exact wording, file paths, function names, example pairs. The chat paraphrase is not the spec.

If the user says "developer decides" on any item, record it as: *developer to implement using the simplest approach that satisfies the surrounding requirements*.

## Phase 6 — Finalise

Reached when no item is still `Status: open` and the user has approved. Finalise **in the same `prd/[slug].md` file** — do not create a second file. Restructure it into two parts:

1. **The spec** (top of file) — clean, developer-facing, only actionable work.
2. **`## Decision log & rationale (historical)`** (appended below, after a `---`) — the Section A/B Q&A with each item's recommendation, `Why:`, the user's answer, and rejected alternatives. This preserves the reasoning and provenance without cluttering the spec.

Flip the banner to `Status: Ready for development`.

**The spec must be self-sufficient.** A developer must be able to implement from the spec section alone, without scrolling into the decision log. So promote the reusable essentials *up* into the spec:

- A short **Codebase touchpoints** list — the real files/functions/tables the work touches (from your Phase 2 review). This is the part of the old "context" block developers actually need; don't leave it stranded in the log.
- A one-line rationale on any **non-obvious** decision (so a developer who disagrees sees why before re-opening it). Deep reasoning and rejected alternatives stay in the log.

The spec includes only actionable work. Things decided as "do nothing", "deferred", or "out of scope" get a one-line mention in Scope → Out of scope (so they read as deliberate), with any detail living in the log.

Spec template:

```
# Requirements: [Feature name]
Date: YYYY-MM-DD
Status: Ready for development

## Summary
[2-4 sentence overview]

## Scope
### In scope
### Out of scope

## Codebase touchpoints
[Key files / functions / tables the implementation touches, from Phase 2 review.
Each with a few words on its role. Lets a developer start without re-discovering the map.]

## Functional requirements
[Numbered, specific, testable behaviours. One-line rationale inline on non-obvious ones.]

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

---

## Decision log & rationale (historical)
[The Section A/B items: question, recommendation, Why, user's answer, rejected
alternatives. Kept for provenance — not needed to implement the spec above.]
```

After restructuring the file, post:

```
Requirements finalised at [path]. Ready to plan the implementation when you are.
```

## Operating rules

- **One file per feature.** Everything lives in `prd/[slug].md` — working draft and finalised spec are the same file at different stages. Never create a `-draft` companion or any second file for the same feature.
- **No duplication.** Every question or decision appears exactly once. Never re-list items in a separate open-items section, a summary, or a recap — the single entry, with its `Status` line, is the source of truth. If you catch yourself writing the same question twice, delete one.
- **Spec is self-sufficient.** After finalisation, the spec section must be implementable without reading the decision log; promote codebase touchpoints and one-line rationale up into it.
- Ground every question in code you read in Phase 2.
- Output goes to the feature document and short chat messages only — no interactive elicitation tools.
- No implementation code during this workflow.
- Prefer specificity over completeness theatre.
- If raw input is very short, note that in the draft and still produce a first pass.
- Make your applied principle visible on every recommendation so the user can audit the reasoning.
