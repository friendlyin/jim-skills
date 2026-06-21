---
name: make-prompt
description: Shape rough notes into a clean prompt or SKILL.md, or review an existing one against Anthropic's best practices.
disable-model-invocation: true
---

# Make a Prompt or Skill

Use this when the user wants to shape rough notes into a proper prompt or skill, or review one they already have.

## What you do

Pick the mode from the user's request:

1. **Notes → prompt.** They give scrappy notes; you return a clean, ready-to-use prompt.
2. **Notes → skill.** Same, but the output is a `SKILL.md` with YAML frontmatter.
3. **Review existing.** They paste a prompt or skill; you flag concrete issues with reasons, then offer a rewrite.

If anything important is missing, ask follow-ups before writing. See "When to ask" below.

## Universal rules (prompts and skills)

Golden rule: a colleague with no context should be able to follow it. If they'd be confused, so will LLM.

- Be concise. Assume LLM already knows basics.
- Be specific about output, format, and constraints. Vague briefs produce vague results.
- Tell LLM what to do, not what not to do. "Write flowing prose" beats "don't use bullets."
- Use examples selectively, not by default. Add 1 to 2 short positive examples only when instructions alone may leave ambiguity about format, tone, or edge-case handling. For rigid or high-stakes formats where consistency matters, use up to 3 to 5 examples only if they are clearly relevant, diverse, and measurably improve reliability. Keep examples minimal and pattern-focused so the model learns the structure without anchoring on unnecessary content. Wrap examples in `<example>` tags so they don't blur with instructions.
- Use XML tags when the prompt mixes instructions, context, examples, and inputs. Names should be consistent and descriptive (`<instructions>`, `<context>`, `<input>`).
- Steps in order? Use numbered list.
- Add the why. "Use a warm tone because users are nervous first-timers" beats "Use a warm tone."
- Match prompt style to the desired output style.
- Don't shout (e.g. "CRITICAL: ..."). Aggressive language now causes overtriggering.
- Put longform data (documents, inputs) at the top: above query, instructions, and examples. This can significantly improve performance.

## Prompt-specific

- Decide upfront: system prompt (persistent role and behavior) or task prompt (one-off). A role can be set in a single sentence.
- For tasks over a long document, ask LLM to quote relevant passages first, then answer. Cuts hallucinations.

## Skill-specific

A skill is a `SKILL.md` with YAML frontmatter. The frontmatter is what LLM uses to decide whether the skill applies, so it matters more than the body.

### Frontmatter (meta-settings)

```yaml
---
name: <name>
description: <description>
---
```

- **`name`** (≤64 chars, lowercase + numbers + hyphens only, no reserved words `anthropic` or `claude`). Prefer **action oriented**: `process-pdfs`, `analyze-spreadsheets`. Avoid vague, generic names.
- **`description`** (≤1024 chars, third person). Must include **both what the skill does AND when to use it**, with concrete trigger words. This is the most important field. LLM picks the skill from this alone.
  - Good: "Generate commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes."
  - Bad: "Helps with documents."
- Other useful frontmatter (Claude Code skills): `disable-model-invocation: true` to make the skill manual-only (user invokes with `/name`, LLM never auto-triggers), `argument-hint: [example]` for autocomplete UX, `allowed-tools: Read Grep` to pre-approve tools.

### Body

- Keep under **500 lines** for optimal performance.
- Keep the skill as a single `SKILL.md` file. If examples are needed, include them in the file itself and keep them compact.
- Set a **degree of freedom** that matches the task:
  - *High* (list instructions) when many approaches work.
  - *Medium* (template scripts with parameters) when there's a preferred pattern.
  - *Low* (exact commands, "do not modify") when consistency is critical.
- Give **one default**, with an escape hatch if needed. Don't present multiple approaches unless necessary.
- **Consistent terminology.** Pick one term and use it throughout.
- Always **forward slashes** in paths, even on Windows.
- For MCP tools, always use **`ServerName:tool_name`**.
- Multi-step workflow? Give a **checklist** LLM can copy and tick off.
- Quality-critical? Build a **feedback loop**. Common pattern: Run validator → fix errors → repeat.

## When to ask follow-up questions

Don't try to write a great prompt or skill from a vague brief. Ask questions to specify requirements. Pick the ones that would most change the output, and skip anything you can reasonably infer.

**General:**
- What does a good answer look like?
- Desired length, tone, format?
- Edge cases or things to refuse?
- Pure instructions, or does it run scripts/tools?
- One happy path, or branches?

**For skills:**
- What's the trigger? When should and shouldn't this skill activate? (Or should it be manual-only via `disable-model-invocation`?)
- Bundled reference files needed, or fits in one `SKILL.md`?

## Reviewing an existing prompt or skill

1. Run through the **Universal rules** and the relevant specific section. Flag concrete violations with pointers to the line or section.
2. Note what's **missing**.
3. Suggest improvements, and wait for the user to validate.
4. Implement improvements only if asked.

Keep the review short. Don't echo the original back to the user.

## Where to save new skills

Save every new skill to `~/.agents/skills/<skill-name>/SKILL.md`. After saving, create a symlink in `~/.claude/skills/` so tools that scan that directory (Claude Code, Conductor, etc.) can find it.

```bash
mkdir -p ~/.agents/skills/<skill-name>
# ... write the SKILL.md ...
ln -s "../../.agents/skills/<skill-name>" ~/.claude/skills/<skill-name>
```

The symlink path must be relative (`../../.agents/skills/<name>`), not absolute, so it survives home-directory moves.

Do not write skill files directly into `~/.claude/skills/` — that directory holds only symlinks.

---
