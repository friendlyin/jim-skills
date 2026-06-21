---
name: make-textbook
description: Create a structured, analytical learning textbook that teaches the concepts behind a source document, web page, or codebase. Use when the user wants to deeply understand a complex topic or system before working on it — when they ask to "make a textbook", "create a study guide / learning manual / explainer", "teach me this codebase/document", or want a top-down, plain-language-but-rigorous guide with definitions, diagrams, alternatives, and good-vs-bad examples. The skill interviews the user for their learning goal and current level, proposes a table of contents for approval, then writes the textbook chapter by chapter in a consistent house style.
disable-model-invocation: true
---

# Make a Textbook

Turn a source (a document, web page, or part of a codebase) into a **practical, analytical textbook** that teaches the concepts a learner needs in order to understand and act on that material.

This is a *teaching* document, not a summary and not a reference manual. Its defining qualities: top-down, plain-language but intellectually demanding, driven by *mechanisms* (it explains how things work and why, not just what they are), and always honest about alternatives and trade-offs.

## Inputs

The user supplies one or more sources to build the textbook around: a document (file path), a web page (URL), or part of a codebase (paths/directories). Ingest the source completely before doing anything else, so your questions and outline are grounded in what's actually there.

## Workflow

Follow these steps in order. Copy this checklist and tick items off as you go.

1. **Ingest the source.** Read the document / fetch the page / explore the code. Identify the concepts a learner would need to understand it, and which are advanced enough to deserve their own treatment.
2. **Interview the learner.** Use the AskUserQuestion tool (not prose questions) to establish the four things that set the right level of abstraction — see "The interview" below.
3. **Set the abstraction level.** From the answers, decide how high to start and how deep to go. Calibrate analogies and what to explain-vs-assume to the learner's actual background.
4. **Propose the table of contents in chat.** Present the full Part → Chapter hierarchy with a one-line description of each, in reading order. Do not write the textbook yet. Adjust until the user approves.
5. **Write the textbook**, chapter by chapter, following the house style and chapter shape below. For a long textbook, write and save in parts rather than one pass.
6. **Add the appendices.** A **Glossary** of every defined term with the chapter where it's explained, and a **Source map** linking each chapter to the part of the source it explains.
7. **Run the term-coverage review** (required final pass). Re-read for any technical term used without a definition or cross-reference, and fix each one.

## The interview

Ask with the AskUserQuestion tool. Prioritize the questions that most change the output; infer the rest. Cover:

- **Learning goal** — what the user wants to *be able to do* after reading. This sets what the textbook must build toward.
- **Current level** — novice / some exposure / strong in this domain. This sets the starting abstraction and how much to derive from first principles.
- **Existing background** — adjacent skills they already have, so you can calibrate analogies, reuse what they know, and skip basics they've mastered. Pitching too low bores; too high loses.
- **Scope & format** — how comprehensive (whole topic vs. hard parts only), rough length, and reading medium (which constrains formatting; see Output).

If the source's purpose or the right depth is genuinely ambiguous after ingesting it, ask one or two clarifying questions alongside the interview rather than guessing.

## Output

- **Format:** a single Markdown (`.md`) file by default, named for the topic, written into the user's working folder.
- **Reading-medium safety (default on):** assume the file may be exported to PDF and read on a small device. So: no tables wider than four columns (prefer lists or prose), and draw diagrams as monospace ASCII inside code fences so they survive export. Override only if the user says otherwise.
- **Artifacts:** include real, simplified code/SQL/JSON/folder-trees where they make a concept concrete, annotated in plain language so a non-coder can follow them.

## House style

The voice is for a capable, analytically-minded learner who is not yet an expert: simple words, demanding content, no condescension and no padding. Every sentence should carry information the reader can act on or reason with. Apply these principles throughout:

- **Top-down.** Move from the largest idea to the most concrete — across the book, and within each chapter. Establish *why a thing matters* before *how it works in detail*.
- **Mechanism over assertion.** This is the central rule. Do not claim something is good, fast, reliable, or powerful — explain the specific mechanism that produces the property, and name the alternative it beats. Replace adjectives with causes. Avoid empty intensifiers ("extraordinarily", "seamlessly", "best-in-class"); if you reach for one, you owe the reader a mechanism instead.
- **Define every hard term on first use**, in a short line beginning "In plain words:". If a term gets fuller treatment in a later chapter, still gloss it briefly now and add "(see Chapter N)".
- **Lists over dense paragraphs** whenever you are enumerating reasons, properties, steps, or options — the reader will return to find one specific point.
- **Show alternatives and trade-offs.** For each significant recommendation, name the real competitors (not strawmen), explain why the chosen option wins *for this situation*, and state the honest cost of the winner.
- **Contrast good and bad decisions** where the topic allows, ideally by following a plausible wrong choice until it breaks. Mistakes teach faster than rules.
- **Cross-reference** related chapters with "(see Chapter N)" so the reader can navigate the web of ideas and each definition lives in exactly one place.

Use examples and artifacts to make abstract points concrete, but sparingly and only where they genuinely aid understanding — enough to illustrate, not so many that they become a template the reader feels bound to.

To make "mechanism over assertion" concrete: instead of "this database is extremely reliable and capable," write what actually causes that — how it handles simultaneous access, what guarantees it gives on a crash, what a weaker alternative gives up — so the reader gains a transferable insight rather than an opinion.

## Chapter shape

Group chapters into **Parts** by theme (foundational ideas first, concrete subsystems next, transferable meta-skills last). Give each chapter, in this order:

1. **Plain-words orientation** — one short, jargon-free paragraph on the idea and why it matters.
2. **Terms introduced here** — a short list of the new terms this chapter introduces, each with a one-line definition (note where any are covered more fully).
3. **Analytical sections** — mechanism-driven explanation; lists for multi-part reasoning; hard terms defined in flow; an annotated artifact or diagram where it helps; the alternatives and trade-offs; a good-vs-bad contrast where the topic allows.
4. **A closing link or two** to neighboring chapters, so the reader sees the connections.

Not every chapter needs code or a good-vs-bad block, but every chapter needs the orientation, the terms list, mechanism-driven explanation, and at least one alternative or contrast where the subject allows.

## Degree of freedom

The workflow steps and the house-style principles are fixed — follow them closely, because consistency across chapters is what makes a textbook usable. Within a chapter you have full freedom in which examples, analogies, and diagrams best teach the specific concept.
