---
name: make-iteration
description: Improve task execution by applying a compact set of decision-making principles before implementation. Use when the user asks to use make iteration, wants a more deliberate approach after failed attempts, or wants work guided by diagnosis-first thinking, protection of known-good behavior, responsive design based on real constraints, reuse of proven patterns, and narrow root-cause fixes.
disable-model-invocation: true
---

# Make Iteration

Use this skill to improve the quality of execution on a new task by slowing down the decision-making just enough to avoid avoidable mistakes.

## Goal

Produce a more deliberate solution by applying a small set of principles before and during implementation.

## Principles

### Diagnosis and scope

1. Diagnose before implementing.
   Identify the real failure mode first. If the issue is unclear or prior attempts failed, compare plausible approaches before changing code.

2. Prefer narrow root-cause fixes.
   Make the smallest change that removes the underlying trigger, and use each round of feedback to tighten scope further.

### Protect working behavior

3. Preserve known-good behavior.
   Treat any working context, especially mobile or another already-approved mode, as a hard constraint while fixing the broken one.

4. Match verification depth to change risk.
   Use lightweight validation for low-risk copy or styling changes, and reserve heavier testing for behavior, state, or multi-step flows.

### Work with the real UI constraint

5. Design for the real constraint.
   In responsive or stateful work, base decisions on the limiting factor such as height, width, device behavior, async timing, stale local state, or perceived layout stability, not just the most obvious label or proxy.

6. Avoid unnecessary intermediate states.
   If the final destination is already known, open or render that state directly instead of showing a temporary precursor flow that adds friction or visual churn.

7. Keep loading and final layouts structurally aligned.
   Loading states should reserve the same core structure and footprint as the resolved UI so the interface does not blink, jump, or resize when data arrives.

8. Reuse proven patterns.
   If an existing layout, flow, or behavior already works under similar constraints, use it as the fallback or template instead of inventing a new intermediate pattern.

### Match the product intent

9. Lock copy before implementation when meaning is still being shaped.
   If the user is still refining wording or product framing, iterate on concise options first and implement only after the message is approved.

10. Describe the real user value, not internal mechanics.
   Prefer copy and interactions that reflect what users are actually invited into or trying to do, rather than internal terms, technical roles, or system-centric labels.

11. Confirm transient actions in place.
   For actions like copy, save, or share, give immediate lightweight feedback in the control itself when possible.

## Output style

Be concise, structured, and decision-oriented. Prefer short options with tradeoffs when analysis is needed, and short implementation rationale when action is clear.
