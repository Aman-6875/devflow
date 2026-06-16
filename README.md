# devflow

A collection of opinionated AI agent skills covering the full development
workflow — **plan → execute → test → PR → review**.

devflow is [APM](https://microsoft.github.io/apm/)-compatible, so it works
across Claude Code, Cursor, GitHub Copilot, OpenCode, Codex, Gemini, and
Windsurf.

---

## The Workflow

```
ticket (optional)
   │
   ▼
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────────┐     ┌──────────┐
│ planner  │ ──▶ │ executor │ ──▶ │  tester  │ ──▶ │  pr-creator  │ ──▶ │ reviewer │
└──────────┘     └──────────┘     └──────────┘     └──────────────┘     └──────────┘
   analyze         implement         write             open PR              audit
   & plan          per plan          tests             with context         the diff
```

Each step is a discrete skill. Invoke them individually — devflow never
auto-chains. You stay in control.

---

## Skills

| Skill | What it does |
|---|---|
| [`planner`](skills/planner) | Analyzes a ticket (or plain task description) and produces a step-by-step implementation plan |
| [`executor`](skills/executor) | Implements code following an approved plan, stack-aware and version-aware |
| [`tester`](skills/tester) | Writes test cases for code or a ticket, using the project's existing test framework |
| [`pr-creator`](skills/pr-creator) | Opens a pull request with a clean title, structured description, and ticket linkage |
| [`reviewer`](skills/reviewer) | Reviews a PR or local diff against SOLID, security, performance, and project conventions |

---

## Install

### Via APM (recommended)

Install the APM CLI:

```bash
# macOS / Linux
curl -sSL https://aka.ms/apm-unix | sh

# Windows
irm https://aka.ms/apm-windows | iex
```

In your project, declare the devflow skills you want in `apm.yml`:

```yaml
name: my-project
version: 1.0.0
dependencies:
  apm:
    - your-username/devflow/skills/planner
    - your-username/devflow/skills/executor
    - your-username/devflow/skills/tester
    - your-username/devflow/skills/pr-creator
    - your-username/devflow/skills/reviewer
```

Then:

```bash
apm install
apm compile -t claude    # or -t copilot, -t cursor, etc.
```

APM resolves and installs the skills; `apm compile` transforms them
into the right format for your target AI client.

### Manual install (Claude Code)

```bash
git clone https://github.com/your-username/devflow.git ~/devflow
ln -s ~/devflow/skills/* ~/.claude/skills/
```

---

## Usage

After install, invoke skills explicitly in your AI client:

```
/planner    — paste ticket details or describe the task
/executor   — after approving planner's output
/tester     — to add tests once code is written
/pr-creator — to open a PR once the branch is ready
/reviewer   — to review a PR or diff
```

Or rely on auto-trigger — each skill's description includes natural-language
triggers, so phrases like *"review this PR"* or *"write a plan for TICKET-123"*
will fire the right skill automatically.

---

## Ticket Sources

devflow is **ticket-source agnostic**. You bring the ticket — paste content,
share CLI output (`az boards`, `gh issue view`, `jira view`), share a URL with
a summary, or just describe the task. Skills never assume any specific ticket
system and never try to fetch tickets themselves.

---

## Design Principles

1. **Plan first, implement on approval** — no auto-chaining between skills.
2. **Trust the model on framework versions** — skills detect stack/version
   from project files and write idiomatic code for that exact version. No
   hand-maintained version reference files.
3. **Ticket-source agnostic** — bring your own ticket, in any format.
4. **Minimal dependencies** — each skill is a self-contained `SKILL.md`,
   with reference files only when a skill genuinely needs structured data.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for skill-authoring rules and PR
process.

---

## License

[MIT](LICENSE)
