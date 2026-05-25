# Jim's Claude Skills

A collection of Claude Code skills I use regularly and refine over time. Install any of them to get the same workflows in your own projects.

## How to install

**Install all skills globally (available in every project):**

```bash
git clone https://github.com/friendlyin/jim-skills.git /tmp/jim-skills \
  && mkdir -p ~/.claude/skills \
  && cp -r /tmp/jim-skills/make-prompt /tmp/jim-skills/make-requirements-with-user ~/.claude/skills/ \
  && rm -rf /tmp/jim-skills
```

**Install a single skill globally:**

```bash
# make-prompt
mkdir -p ~/.claude/skills/make-prompt \
  && curl -o ~/.claude/skills/make-prompt/SKILL.md \
     https://raw.githubusercontent.com/friendlyin/jim-skills/main/make-prompt/SKILL.md

# make-requirements-with-user
mkdir -p ~/.claude/skills/make-requirements-with-user \
  && curl -o ~/.claude/skills/make-requirements-with-user/SKILL.md \
     https://raw.githubusercontent.com/friendlyin/jim-skills/main/make-requirements-with-user/SKILL.md
```

**Install into a single project only:**

Replace `~/.claude/skills` with `.claude/skills` inside your project directory.

Once installed, restart Claude Code (or open a new session) and invoke any skill by typing `/skill-name` in the chat.

## Skills

### [`/make-requirements-with-user`](./make-requirements-with-user/SKILL.md)

Turns raw notes into a complete, structured requirements document through an iterative review cycle. Useful before writing any code: Claude reviews the codebase, drafts clarifying questions with recommendations, and refines the spec through back-and-forth until it's ready for development.

### [`/make-prompt`](./make-prompt/SKILL.md)

Shapes rough notes into a clean, ready-to-use prompt or `SKILL.md` file, or reviews an existing one against Anthropic's prompt engineering best practices. Covers both one-off task prompts and reusable Claude Code skills.
