---
name: extract-feedback
description: Extract reusable decision-making principles from feedback given to an AI agent across one thread or conversation, especially where the first iteration missed the right result and later iterations improved through user feedback. Use when the user asks to analyze a thread, extract principles from in-chat feedback, compare them against the make-iteration skill, identify already-covered-but-missed principles, propose improvements to that skill, and optionally update make-iteration after user approval.
disable-model-invocation: true
---

# Extract Feedback

Use this skill to learn from iterative user feedback and turn it into reusable principles.

## Goal

Scan a specific thread or the current conversation, find places where the user corrected, narrowed, or redirected the work after a weak first attempt, and convert that feedback into a compact list of reusable principles.

This skill also checks whether the same principle already exists in `~/.agents/skills/make-iteration/SKILL.md` but was not applied in practice.

## Inputs

Use one of these sources:

1. A specific thread the user names or provides.
2. The current conversation if no separate thread is provided.
3. A pasted transcript if thread access is unavailable.

Always read `~/.agents/skills/make-iteration/SKILL.md` before extracting principles.

## Default workflow

1. Read the conversation source.
   Find iteration points where:
   - the first result was wrong, weak, or incomplete;
   - the user gave corrective feedback;
   - later iterations improved because of that feedback.

2. Extract feedback candidates.
   Focus on reusable lessons, not task facts.

3. Normalize the principles.
   Apply these rules:
   - Generalize, do not retell.
   - Merge overlaps aggressively.
   - Prefer a few strong principles over a long exhaustive list.
   - Organize by logic, not chronology.
   - Keep the abstraction level consistent.
   - Preserve domain-relevant lessons in generalized form.

4. Compare against `make-iteration`.
   For each extracted principle, decide whether it is:
   - `new`: not present in `make-iteration`;
   - `covered`: already present in `make-iteration` in substance;
   - `partially-covered`: related principle exists, but wording or scope is too weak.

5. Flag execution gaps.
   If a principle was already covered or partially covered, but the user still had to repeat that feedback, call that out explicitly as an application gap.

6. Suggest improvements to `make-iteration`.
   Only when useful:
   - propose sharper wording;
   - propose better grouping;
   - propose a new category if enough principles cluster around one theme.

7. Present the candidate list to the user.
   Do not update `make-iteration` yet.

8. After user approval or refinement, update `make-iteration`.
   Edit the skill so the new principle set stays deduplicated, concise, and well-grouped.

## What counts as feedback worth extracting

Extract only feedback that changes how future work should be done.

Good candidates:

- corrections to diagnosis approach;
- constraints that should have been protected from regression;
- better ways to scope a fix;
- lessons about responsive behavior, platform behavior, or state timing;
- instructions about how to present options, tradeoffs, or iterations;
- repeated signals that an existing principle is too vague to be reliably applied.

Ignore:

- one-off preferences with no reuse value;
- purely emotional reactions without an actionable lesson;
- bug-specific facts that do not generalize.

## Output format before approval

Return a short structured response with these sections:

- `New principles`
- `Already covered in make-iteration but missed`
- `Suggested improvements to make-iteration`
- `Grouping suggestions` (only if useful)

For each item, keep it short:

- principle text;
- why it matters in one sentence at most;
- status: `new`, `covered`, or `partially-covered`.

## Output format after approval

1. State what changed in `make-iteration`.
2. Keep the summary short.
3. Mention any regrouping or wording upgrades.

## Rules for updating make-iteration

- Do not append blindly.
- Deduplicate against existing principles.
- Rewrite overlapping items into one stronger principle.
- If a category becomes crowded, group it under a clear heading instead of keeping one flat list.
- Preserve readability over completeness.
- Prefer strong default wording over long explanatory text.

## Output style

Be concise, analytical, and reusable. Optimize for future guidance, not for summarizing the past conversation.
