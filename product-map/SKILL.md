---
name: product-map
description: Generates and maintains product/product.md — a living document covering value props, product principles, use-cases (grouped by feature scope with descriptions), and first-session flows. Reflects what the product does today, not aspirations. Use when the user wants to create or update their product documentation based on the codebase. Manual-only — never auto-triggered.
disable-model-invocation: true
allowed-tools: Read Bash Glob Grep Write Edit
---

# Product Map

Creates and maintains `product/product.md` — a single source of truth for what the product does today. Used by agents and team members when making product decisions.

## Output document structure

`product/product.md` always follows this layout:

```
---
last_commit: <git SHA>
---

# Product Map

## Value Props
## Product Principles
## Use-Cases
## First-Session Flows
## Legacy
```

The `last_commit` frontmatter tracks which git state the document reflects. Do not remove or manually edit it.

### Section definitions

**Value Props** — outcomes users get from this product, stated from the user's perspective. Not feature descriptions. "Ship with less review overhead" beats "Has an automated review tool."

**Product Principles** — short, opinionated rules that guide product decisions. Each principle must pass this test: a PM should be able to cite it when accepting or rejecting a feature. These are stable and protected — see "Protecting principles" below.

**Use-Cases** — grouped by feature scope (`###` subheadings). Each use-case is a bullet with a one-line description of what the user can do and why it matters. A feature counts as user-accessible only when there is a clear gateway to it: a navigation link, menu item, button/CTA, exposed CLI command, or documented API endpoint. Code-only features with no user-facing entry point do not belong here — they go in Legacy.

```
### [Feature Scope Name]
- **Use-case name** — what the user can do and why it matters.
```

**First-Session Flows** — distinct onboarding or first-use journeys available in the product. For each flow:
- Who it's for (persona or entry condition)
- Steps the user goes through
- End state: what the user has achieved by the end of the session

**Legacy** — features and use-cases that exist in the codebase but are not user-accessible (no UI/CLI gateway). Same grouping format as Use-Cases. This section exists so the team is aware of what's in the code but unreachable, and can make explicit decisions about whether to surface, clean up, or leave these features.

```
### [Feature Scope Name]
- **Feature name** — what it does, and why it's not currently accessible to users.
```

---

## Step 1 — Determine mode

Check `product/product.md`:

| State | Mode |
|---|---|
| File doesn't exist | **Init** |
| File exists, no `## Open Questions` section | **Update** |
| File exists with `## Open Questions` section | **Finalize** |

---

## Step 2A — Init mode

### Check for git

Run `git rev-parse HEAD`. If it fails (no repo or no commits), ask the user:

> "This project has no git history, so I won't be able to track changes for future updates. Should I proceed with a full codebase scan, skipping version tracking?"

Wait for confirmation before continuing.

### Scan the codebase

Read only high-signal artifacts. Stop scanning a category once you have enough signal — do not read exhaustively.

Priority order:
1. `README.md`, `README.rst`, `docs/` — user-facing descriptions
2. `package.json` / `pyproject.toml` / `Cargo.toml` — name, description, scripts/commands
3. Route files, nav configs, sitemap — reveals available screens and flows
4. Top-level page or screen components (`src/pages/`, `src/screens/`, `app/`, etc.)
5. API route definitions (`routes/`, `src/api/`, `server/`, etc.)
6. CLI entry points or command definitions

Skip: test files, lock files, generated files, `node_modules/`, build output, config-only files.

### Verify accessibility

Before placing any feature in Use-Cases, confirm it has a user-facing gateway. For each candidate feature, look for:
- A navigation link, menu item, or route that is reachable from the app's main entry point
- A button or CTA that triggers it
- An exposed and documented CLI command
- A public API endpoint referenced from UI or docs

If a feature exists in code but you cannot find a clear gateway, do not add it to Use-Cases. Instead:
- Add it to the **Legacy** section if you are confident it has no gateway.
- If you are uncertain (gateway may exist but you couldn't find it), flag it in **Open Questions** tagged `[Accessibility]` and ask the user.

### Write the draft

Create `product/` if it doesn't exist. Write a full draft of all five sections based on what you found. Set `last_commit` in frontmatter to HEAD (omit if no git).

**On Product Principles during init:** Derive 3–5 starter principles from visible design patterns in the codebase (e.g., "progressive disclosure", "mobile-first", "single-step actions"). Keep them high-level — not implementation detail. Mark them clearly as starters that the user should refine. In chat, tell the user: "I've drafted initial Product Principles from codebase patterns — please review and adjust these, they're meant to be yours."

Then proceed to **Step 3**.

---

## Step 2B — Update mode

### Get what changed

1. Read `last_commit` from the doc's frontmatter.
2. Run: `git diff <last_commit>..HEAD --name-only`
3. Filter to product-relevant files: routes, pages, screens, API definitions, README, docs. If nothing relevant changed, tell the user and stop.
4. Run: `git diff <last_commit>..HEAD -- <relevant files>` to read the actual diff.

### Apply surgical updates

Determine which sections are affected:
- New screens / routes / commands → verify accessibility first (same rules as Init), then add to Use-Cases or Legacy accordingly
- Removed gateway (nav link, button, route removed) → move the use-case from Use-Cases to Legacy; do not delete it
- Feature fully deleted from codebase → remove from Use-Cases or Legacy entirely
- Changed onboarding or entry flows → First-Session Flows
- README / docs changes → Value Props

When a diff shows a gateway being added to a feature already in Legacy, move it from Legacy to Use-Cases.

If a change makes accessibility ambiguous (e.g., route exists but it's unclear if it's linked), flag it as an `[Accessibility]` question rather than guessing.

Edit only affected sections. Do not rewrite content that wasn't touched.

**Product Principles are protected.** Do not modify the Principles section during an update, even if the codebase changed. If you believe a principle is now outdated, flag it in chat and ask for guidance before touching it.

Update `last_commit` in frontmatter to HEAD.

Then proceed to **Step 3**.

---

## Step 2C — Finalize mode

1. Read all questions and answers from `## Open Questions`.
2. Remove the `## Open Questions` section entirely from the document.
3. Apply answers to the relevant sections.
4. Update `last_commit` in frontmatter to HEAD.
5. If any answer implies a change to Product Principles, stop and ask the user explicitly before applying — do not modify Principles autonomously.

Then proceed to **Step 4**.

---

## Step 3 — Question handling

Only ask questions when there is genuine uncertainty that would materially affect the document. Good triggers: a feature is behind a feature flag (is it user-facing?), two competing patterns (which is canonical?), something ambiguous enough that a wrong call would mislead future agents.

Do not ask for completeness or validation.

**If no questions are needed:** save/write the document as-is, then proceed to Step 4.

**If questions are needed:** append this section to the end of `product/product.md`:

```markdown
## Open Questions

_Answer inline below each question, then run `/product-map` again to finalize._

- [ ] **[Section]** Question text here.
  > Your answer:

```

Tag each question with the section it affects: `[Value Props]`, `[Use-Cases]`, `[Flows]`, `[Principles]`, `[Legacy]`, `[Accessibility]`.

Tell the user in chat: "I've added some questions to the end of `product/product.md`. Answer them there and run `/product-map` again when done."

Stop here — do not proceed to Step 4 until the next invocation.

---

## Step 4 — Gap analysis (end of session)

After the document is in its final state for this session (no pending Open Questions), scan for gaps:

- Are there Value Props that no Use-Case covers?
- Are there First-Session Flows implied by the Value Props but not documented?

If gaps exist, mention them once in chat — a concise list of what's implied but not yet in the product. Frame it as: "Based on the value props, these capabilities seem implied but I didn't find them in the codebase — worth discussing whether to build or document them:"

Do not add aspirational items to the document itself.

---

## Protecting principles

Product Principles are the most stable section and should not be changed without the user's intent. Specifically:
- Never modify Principles autonomously during an update run.
- If a codebase change implies a principle needs updating, raise it in chat after the update is done.
- If an Open Questions answer implies a Principles change, confirm with the user before applying.
- When the user explicitly asks to update Principles, proceed — but propose the change and wait for confirmation before writing.

---

## File conventions

- Output: `product/product.md` (create `product/` if missing)
- YAML frontmatter is always the first thing in the file
- `## Open Questions` is always the last section and is temporary — never let it persist across sessions
