---
name: make-requirements-with-user
description: Turn raw user notes into a complete, developer-ready requirements folder through iterative clarification. Produces one or more focused requirement files plus a QA test-case file per feature. Use when the user explicitly invokes this skill to scope a feature, bug fix, or development iteration before any code is written.
disable-model-invocation: true
---

# Make Requirements with User

A multi-cycle workflow: codebase review → draft with recommendations → user feedback → revise → finalise. Most non-trivial tasks need 2-3 cycles. The requirements folder is the source of truth; chat history is not a substitute.

**One folder per feature.** Everything for a feature lives under `prd/[slug]/`. During clarification there is a single working file, `prd/[slug]/_working.md`. At finalisation that becomes one or more clean requirement files plus a `qa-test-cases.md`, all in the same folder. Never leave a `-draft` companion and never scatter one feature across sibling folders.

**One item, one place.** Every question or decision appears exactly once, in the section it belongs to, and that single entry is where the user comments and where its status lives. Never restate a question in a second list, a summary, or a separate "open items" section — duplication wastes the user's reading time and makes it unclear which copy is authoritative. Track status *on the item itself* with a `Status:` line, not by copying it elsewhere.

## Phase checklist

1. Receive raw input.
2. Review the codebase silently — including how each behaviour can be forced/observed, for the QA file.
3. Create `prd/[slug]/_working.md` with a DRAFT banner: a **Captured requirements** part (what the user already specified, restated) plus **Open items** (Sections A and B) for the genuine decisions. Each open item carries an inline `**Status:**` line.
4. Post a short chat message and stop.
5. Iterate on user feedback in the same working file — update each item's status in place; append any newly-surfaced item to A or B.
6. When every item is resolved and the user approves, finalise: replace the working file with clean requirement file(s) + `qa-test-cases.md` in the folder, then delete `_working.md`.

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
- Note **testability hooks** for the QA file: feature flags / remote-config values that force a state, mock endpoints, server handlers to call with mock data, and seedable accounts or fixtures. This lets the QA cases be concrete.

This review grounds your questions and your QA cases. It stays **internal** — it does not become a prescriptive codebase map in the finalised documents.

## Phase 3 — The draft

Write to `prd/[slug]/_working.md` and open it with a banner: `Status: **DRAFT — cycle N.** Not yet finalised.` The document is the source of truth: every decision lives here in full detail, not summarised away because "we covered it in chat". A reader joining at cycle 3 should understand every decision from the file alone.

The draft has **two parts with different jobs**, and keeping them apart is what makes the user's review fast:

1. **Captured requirements** — your structured restatement of everything the user *already specified*, including grammar-polished copy. The user skims this to catch anything you got wrong; they do **not** answer it item by item.
2. **Open items** (Sections A and B) — only the genuine decisions that need the user's input. This is where the user spends their time, so it stays short and high-value.

### Captured requirements

Restate what the user gave you — organised, clarified, grouped — as **statements, not questions**. Fold your grammar/clarity/copy polish in here (see "Copy polish" below). This part is skim-only: it never carries `Status: open` and never blocks finalisation. If you got something wrong, the user edits it inline.

Never turn a captured requirement into an open item just to "confirm" it. A restatement the user could only answer with "yes, that's what I said" wastes their time — it belongs here as a captured statement, not in Section A. This is the single most important rule of the draft: **echoing the user's own requirement back as a question is the failure mode to avoid.**

Group captured requirements the way the finalised files will split (by concern), so the eventual file boundaries are visible early.

### What counts as an open item

An item belongs in Section A/B **only if the user's answer could change the spec.** When it genuinely could, ask — losing something important from the plan is worse than one extra question, so when in doubt, raise it. The one thing you must never raise is an item whose only content restates what the user already decided.

Kinds of question that usually earn their place (illustrative, **not** a closed list — raise any other consequential question you spot, even if it fits none of these):

- **Gap** — something needed to implement that the user didn't specify (a missing option, an unstated default, an unlisted trigger point).
- **Ambiguity / contradiction** — the input is unclear or conflicts with itself.
- **Edge case / risk** — an unhandled "what happens when…".
- **Conflict** — contradicts existing code behaviour or a previously finalised requirement.
- **Deviation** — your recommendation differs from what the user asked, so they should weigh in.

### Copy polish

When the user supplies UI strings, fix grammar/clarity/length silently and show the result in **Captured requirements**, inviting inline edits. Do not create an open item that asks the user to "confirm my polished copy" — present it as captured and let them tweak it if they want.

### Section A — Open questions (with recommendations)

Ask about product behaviour, UX, and scope. Skip low-level technical choices (where to put a field, what to name a variable) — make those silently. Each Section A item is self-contained: question, recommendation, its own status — the user answers it in one place and you update its status there.

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

After the user answers, the item flips in place (no copy made elsewhere):

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

There is **no Section C / "open items" list.** An item's openness lives on the item itself, via its `**Status:**` line. To see what still blocks finalisation, scan for `Status: open`. Re-listing those questions in a second section is exactly the duplication this workflow forbids.

- A **new** question that surfaces mid-cycle is appended to Section A (or B if it's a technical trade-off) as a normal item with `Status: open`.
- An item the user has answered flips to `Status: resolved — <decision>` in place.
- The draft is **ready to finalise when no item is still `Status: open`** (and the user has approved).

## Phase 4 — Notify and wait

Post this in chat, then stop:

```
Draft at prd/[slug]/_working.md, in two parts:
- Captured requirements — my restatement of what you already specified (incl. polished copy). Skim it; edit inline if I got anything wrong. No item-by-item reply needed.
- Open items (Sections A/B) — the decisions I actually need from you, each with my recommendation and a Status line. This is where your time goes.
Reply with "approve all", per-item answers/overrides, or inline edits. Items marked "Status: open" are what's blocking finalisation.
```

## Phase 5 — Iterate

Update the same `prd/[slug]/_working.md` file in place. Show what changed in chat and stop again. Keep iterating until the user explicitly approves finalisation and no item is still `Status: open`.

Update each answered item's `**Status:**` line in place; never copy a question into a tracking list. If a new question surfaces, append it to Section A or B as a normal `Status: open` item.

When the user paraphrases a refinement in chat, translate it into the concrete spec inside the draft — exact wording, example pairs, observable behaviour. The chat paraphrase is not the spec.

If the user says "developer decides" on any item, record it as: *developer to implement using the simplest approach that satisfies the surrounding requirements*.

## Phase 6 — Finalise

Reached when no item is still `Status: open` and the user has approved. Finalise inside `prd/[slug]/`, then delete `_working.md`.

### Step 1 — Decide the split

Group the resolved work into concerns. **Split test:** a concern gets its own file when it could be built, reviewed, and shipped independently of the others *without reading them*. Tightly-coupled work stays in one file.

- **Single concern** → one file: `prd/[slug]/requirements.md`.
- **Multiple independent concerns** → one file each, numbered by suggested build order: `prd/[slug]/01-[concern].md`, `prd/[slug]/02-[concern].md`, … plus `prd/[slug]/index.md` (summary, list of parts, shared constraints).

A folder with a single requirements file is fine and expected for coupled work.

### Step 2 — Write the requirement file(s)

**The requirement files are the only developer-facing artifact — there is no decision log.** Build the requirement bodies from the **Captured requirements** plus the resolved decisions from Sections A/B. Everything a developer needs must be in the body. Do not demote the Section A/B questions into a log; **delete the questions themselves** at finalisation. Preserve intent as a **one-line `Why:`** on any non-obvious requirement — enough for a developer agent to understand what we're trying to achieve, not the full debate.

**Self-sufficiency sweep (required before flipping any banner):** re-read each requirement file and confirm every behaviour-affecting decision is stated in the body, not merely implied by a resolved question you just deleted. If something load-bearing lived only in the Q&A, write it into the requirements now.

**Requirements describe behaviour, not the call graph.** State observable behaviour, constraints, and acceptance criteria. Do **not** prescribe where new code goes, internal call sequences, or which private helper to invoke — the developer owns implementation. Reference an existing file/function/table **only when a requirement is about that specific entity** (e.g. "rename the string `createStory.stopAndReview`"), never to dictate placement of new code. There is **no `Codebase touchpoints` section**.

The requirement files include only actionable work. Things decided as "do nothing", "deferred", or "out of scope" get a one-line mention in Scope → Out of scope so they read as deliberate.

Requirement file template:

```
# Requirements: [Concern name]
Date: YYYY-MM-DD
Status: Ready for development
Part of: [Feature name] (see index.md)   ← omit this line if single-file

## Summary
[2-4 sentences. Do not restate this inside the requirements below.]

## Scope
### In scope
### Out of scope

## Functional requirements
[Numbered, specific, observable, testable behaviours. One-line `Why:` on non-obvious
ones. No paths for new code, no internal call sequences.]

## Items requiring investigation before fix
[Only if bugs with unverified causes: symptom, suspected cause, investigation step,
then the fix only after the cause is confirmed.]

## Non-functional requirements
[Performance, accessibility, security, compatibility — only if relevant.]

## Data & state
[Models, fields, schema changes — only if relevant.]

## Edge cases & error handling

## Open decisions
[Empty if Phase 6 was reached cleanly. Record any remaining unresolved decision
explicitly rather than leaving it implicit.]
```

`index.md` template (multi-file only):

```
# [Feature name]
Date: YYYY-MM-DD
Status: Ready for development

## Summary
[2-4 sentence overview of the whole feature.]

## Parts
- 01-[concern].md — one line on what it covers
- 02-[concern].md — one line on what it covers

## Shared constraints
[Limits, flags, cross-cutting rules referenced by more than one part.]

## QA
See qa-test-cases.md.
```

### Step 3 — Write `qa-test-cases.md`

One file per feature folder, addressed to a **browser-capable QA agent that validates the feature after development** — not the developer. Group cases under a heading per functionality block, matching the requirement file/section they validate, so the mapping to requirements is explicit (e.g. `## 01 — Demo-first funnel`).

Every functional requirement should map to at least one case. If a requirement can't be expressed as something observable, it is under-specified or belongs in non-functional requirements.

Each case:

<example>
### QA-1.1 — Anonymous user hits the demo interaction cap

**Validates:** 01-demo-first-funnel.md FR 2.4

**Setup:** Set remote-config `onboarding_variant_web = demo_first`. Start from a clean browser (no session). Use the mock interviewer handler that returns a canned response per turn.

**Steps:**
1. Open the app, tap "Get started".
2. Submit 3 user responses in the demo interview.
3. Attempt a 4th response.

**Expected result:** After the 3rd interviewer response, the templated closing line (`interviewer.demoLimitReached`) is shown; both the voice-record control and the text-input button are gone or disabled; no further `respond` network call can be made.

**Pass/fail:** Pass only if a 4th `respond` request cannot be issued from the UI and the closing line is visible.
</example>

Fields per case: **ID & title**, **Validates** (file + requirement number), **Setup / preconditions** (flags, remote-config values, mock data, server handlers, seeded accounts), **Steps** (concrete, browser-driven), **Expected result** (visible text, element present/absent, network call made or blocked), **Pass/fail** (an objective criterion an agent can evaluate without judgment).

### Step 4 — Finish

Flip the banner to `Status: Ready for development` in every requirement file (and `index.md`). Delete `_working.md`. Then post:

```
Requirements finalised at prd/[slug]/. Files: [list]. QA cases in qa-test-cases.md. Ready to plan the implementation when you are.
```

## Operating rules

- **One folder per feature.** Everything lives in `prd/[slug]/`: `_working.md` during cycles; requirement file(s) + `qa-test-cases.md` after finalisation. Never create a `-draft` companion or scatter a feature across folders.
- **No decision log.** The requirement bodies are complete on their own; keep only a one-line `Why:` on non-obvious requirements, and run the self-sufficiency sweep before finalising. Delete the Q&A at finalisation — do not demote it.
- **No `Codebase touchpoints` section.** Requirements describe behaviour, constraints, and acceptance — not the call graph or where new code goes.
- **Split only when independent.** Multiple files only when concerns are independently shippable; otherwise one requirements file.
- **QA maps to requirements.** Every functional requirement is covered by at least one grouped case in `qa-test-cases.md`.
- **Two-part draft; no confirmation questions.** What the user already specified goes in **Captured requirements** as statements they can skim. Sections A/B hold only decisions whose answer could change the spec. Never create an item whose only purpose is to confirm a restatement — that is the workflow's primary failure mode. When genuinely unsure whether something needs input, ask; when it's just your understanding of their requirement, capture it.
- **No duplication.** Every question or decision appears exactly once, tracked by its own `Status` line. Don't restate scope inside the functional requirements.
- Ground every question in code you read in Phase 2.
- Output goes to the feature folder and short chat messages only — no interactive elicitation tools.
- No implementation code during this workflow.
- Prefer specificity over completeness theatre.
- If raw input is very short, note that in the draft and still produce a first pass.
- Make your applied principle visible on every recommendation so the user can audit the reasoning.
