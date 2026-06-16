# Contributing to devflow

Thanks for your interest. This guide covers how to add a new skill or
improve an existing one.

---

## Skill Authoring Rules

### 1. One folder per skill under `skills/`

```
skills/
└── your-skill/
    ├── SKILL.md          ← required
    └── (optional supporting files)
```

The folder name **is** the skill name (and the slash-command users will
type). Keep it short, lowercase, kebab-case if multi-word.

### 2. SKILL.md frontmatter is mandatory

```yaml
---
name: your-skill
description: >
  One paragraph. Include both the skill's purpose AND natural-language
  trigger phrases so the AI client can auto-invoke it. Be specific.
version: 0.1.0
tags: [category, category]
---
```

`description` is the most important field. The AI uses it to decide when
to fire the skill. Write it with trigger keywords baked in.

### 3. SKILL.md body structure

Every SKILL.md must have these sections, in order:

```markdown
# Skill Name

## When to Use
Clear triggers. When should this skill fire? When should it NOT fire?

## Process
Numbered, deterministic steps. The AI executes these in order.

## Output Format
What the skill produces. Be precise.

## Anti-patterns
What this skill must NOT do. Common failure modes to avoid.
```

### 4. Trust the model

Do not hand-maintain framework-version-specific reference files. Tell the
skill to *detect* the stack and version and *write idiomatic code for that
version*. Modern LLMs know the differences.

### 5. Keep supporting files minimal

Only add reference files (checklists, templates, examples) when the skill
genuinely needs **structured data the model would otherwise hallucinate**.
A PR-review checklist is fine. A "Laravel 11 anonymous migration syntax
guide" is not — the model already knows that.

### 6. No auto-chaining

A devflow skill does one thing and stops. It does not invoke other
devflow skills. The user decides what runs next.

### 7. Ticket-source agnostic

If your skill consumes ticket info, accept whatever the user provides
(paste, CLI output, URL with summary, plain text). Never assume a
specific ticket system. Never instruct the user to install MCP servers
or CLIs.

---

## PR Process

1. Fork and branch (`feat/skill-<name>` or `fix/<short-description>`).
2. Add or modify the skill following the rules above.
3. Run any validation scripts in `.github/workflows/` locally first.
4. Open a PR with a clear description of:
   - What the skill does (or what changed)
   - Why it belongs in devflow
   - Example invocation and expected output

---

## Style

- Markdown only. No HTML in SKILL.md unless absolutely required.
- Keep instructions concise — every line should pull weight.
- Examples are welcome but go in an `examples/` subfolder, not in
  SKILL.md itself.
