# Jim's Agent Skills

A collection of agentic skills I use regularly and refine over time. They are written in `SKILL.md` format and can be used with Claude Code, Claude Cowork, Codex, or other agent setups that read skills from `.agents/skills`.

## Install

Install skills into `~/.agents/skills`.

### Install all skills globally

```bash
mkdir -p \
  ~/.agents/skills/make-prompt \
  ~/.agents/skills/make-requirements-with-user \
  ~/.agents/skills/extract-feedback \
  ~/.agents/skills/make-iteration \
  ~/.agents/skills/make-textbook \
  ~/.agents/skills/product-map \
  ~/.agents/skills/product-manager && \
curl -o ~/.agents/skills/make-prompt/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/make-prompt/SKILL.md && \
curl -o ~/.agents/skills/make-requirements-with-user/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/make-requirements-with-user/SKILL.md && \
curl -o ~/.agents/skills/extract-feedback/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/extract-feedback/SKILL.md && \
curl -o ~/.agents/skills/make-iteration/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/make-iteration/SKILL.md && \
curl -o ~/.agents/skills/make-textbook/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/make-textbook/SKILL.md && \
curl -o ~/.agents/skills/product-map/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/product-map/SKILL.md && \
curl -o ~/.agents/skills/product-manager/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/product-manager/SKILL.md
```

### Install into a single project

Run the same pattern from your project root, but use `.agents/skills` instead of the home-directory path:

```bash
mkdir -p .agents/skills/make-prompt && \
curl -o .agents/skills/make-prompt/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/make-prompt/SKILL.md
```

After installation, restart your agent session if it does not pick up new skills automatically. In Claude Code, manual skills can be invoked with `/skill-name`. Other agent tools may expose installed skills through a different discovery or invocation flow.

## Skills

### [`make-prompt`](./make-prompt/SKILL.md)

Shapes rough notes into a clean prompt or `SKILL.md`, or reviews an existing prompt or skill against Anthropic's prompt-writing best practices. Useful when you want to turn scratch notes into a reusable instruction set for Claude Code, Claude Cowork, Codex, or another agent environment.

Install:

```bash
SKILL=make-prompt
mkdir -p ~/.agents/skills/"$SKILL" && \
curl -o ~/.agents/skills/"$SKILL"/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/$SKILL/SKILL.md
```

### [`make-requirements-with-user`](./make-requirements-with-user/SKILL.md)

Turns rough notes into a developer-ready requirements folder through an iterative clarification loop. It produces one or more focused requirement files plus a QA test-case file for the feature, so the output is ready for both implementation and post-build validation.

Install:

```bash
SKILL=make-requirements-with-user
mkdir -p ~/.agents/skills/"$SKILL" && \
curl -o ~/.agents/skills/"$SKILL"/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/$SKILL/SKILL.md
```

### [`extract-feedback`](./extract-feedback/SKILL.md)

Runs through your conversation with an agent and extracts reusable principles from the feedback you had to give in the second or third iteration. It is especially useful when small iterations keep missing the result you want and you want to turn that repeated feedback into something durable, including improvements to `make-iteration`.

Install:

```bash
SKILL=extract-feedback
mkdir -p ~/.agents/skills/"$SKILL" && \
curl -o ~/.agents/skills/"$SKILL"/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/$SKILL/SKILL.md
```

### [`make-iteration`](./make-iteration/SKILL.md)

This is the skill I use for small iterations where I describe what I want directly in chat and do not need a larger planning or requirements-writing workflow. It is a compact collection of things I often end up repeating as feedback in second or third passes, so the goal is simply to reduce how many iterations it takes to get to a good result. It is still quite subjective and reflects the way I work today, which is why `extract-feedback` is useful alongside it.

Install:

```bash
SKILL=make-iteration
mkdir -p ~/.agents/skills/"$SKILL" && \
curl -o ~/.agents/skills/"$SKILL"/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/$SKILL/SKILL.md
```

### [`make-textbook`](./make-textbook/SKILL.md)

Turns a document, web page, or codebase into a structured textbook that teaches the topic as if you were a student working through it from the top down. I created it because I wanted to go deep on areas like software architecture, using a codebase and supporting documents as source material, and I wanted something much more analytical and sequential than a summary. It asks about the reader's current level first, then generates a textbook meant to actually teach the material rather than just describe it.

Install:

```bash
SKILL=make-textbook
mkdir -p ~/.agents/skills/"$SKILL" && \
curl -o ~/.agents/skills/"$SKILL"/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/$SKILL/SKILL.md
```

### [`product-map`](./product-map/SKILL.md)

Creates a product description for a codebase from the code itself, input documents, or freeform chat input. The output is a high-level product map that covers the value proposition, product principles, use cases, first-session flows, and also legacy components that still exist but should not drive future development. The point is to create shared product context that other agents can use later for things like QA work or product-level requirements.

Install:

```bash
SKILL=product-map
mkdir -p ~/.agents/skills/"$SKILL" && \
curl -o ~/.agents/skills/"$SKILL"/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/$SKILL/SKILL.md
```

### [`product-manager`](./product-manager/SKILL.md)

Takes a rough idea, feature request, or bug report and turns it into product requirements rather than technical requirements. It focuses on UX behavior, scope, and edge cases, and it is meant to build on top of the principles captured by `product-map`. This version is still early and, honestly, quite junior right now: I have only done a couple of iterations with it so far. The expectation is that it will improve over time as I keep giving it feedback and updating the skill.

Install:

```bash
SKILL=product-manager
mkdir -p ~/.agents/skills/"$SKILL" && \
curl -o ~/.agents/skills/"$SKILL"/SKILL.md \
  https://raw.githubusercontent.com/friendlyin/jim-skills/main/$SKILL/SKILL.md
```
