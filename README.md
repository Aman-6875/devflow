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
| [`bug-finder`](skills/bug-finder) | Debugs a reported bug as a senior engineer — traces the code path, ranks hypotheses, finds the root cause, and proposes a minimal fix |

---

## Install

There are two ways to install devflow. **APM is recommended** because
it works across every supported AI client with one config file.

### Option 1 — Via APM (recommended)

#### Step 1. Install the APM CLI (one-time, per machine)

```bash
# macOS / Linux
curl -sSL https://aka.ms/apm-unix | sh

# Windows (PowerShell)
irm https://aka.ms/apm-windows | iex
```

Verify the install:

```bash
apm --version
```

#### Step 2. Create an `apm.yml` in your project root

```yaml
name: my-project
version: 1.0.0

# Which AI client(s) should APM deploy skills to.
# Pick one or more from the table below.
targets:
  - claude

dependencies:
  apm:
    # Pin to a tag for reproducible installs. Drop the #v0.1.0 to track main.
    - Aman-6875/devflow/skills/planner#v0.1.0
    - Aman-6875/devflow/skills/executor#v0.1.0
    - Aman-6875/devflow/skills/tester#v0.1.0
    - Aman-6875/devflow/skills/pr-creator#v0.1.0
    - Aman-6875/devflow/skills/reviewer#v0.1.0
    - Aman-6875/devflow/skills/bug-finder#v0.1.0
```

You can drop any skill you don't want — pick à la carte.

**Supported targets:**

| AI Client          | `targets:` value |
|--------------------|------------------|
| Claude Code        | `claude`         |
| GitHub Copilot     | `copilot`        |
| Cursor             | `cursor`         |
| OpenCode           | `opencode`       |
| Codex              | `codex`          |
| Gemini             | `gemini`         |
| Windsurf           | `windsurf`       |

#### Step 3. Install

```bash
apm install
```

APM resolves the skills, downloads them, and integrates them into your
chosen target client's expected location (`.claude/skills/`,
`.github/copilot-instructions.md`, `.cursor/`, etc.). It also writes
`apm.lock.yaml` for reproducibility and adds `apm_modules/` to your
`.gitignore`.

#### Step 4. Restart your AI client

Restart your editor / AI client so it picks up the new skills. Then
verify:

- Type `/planner` and confirm the skill is available
- Or describe a task naturally and watch the right skill auto-trigger

#### Updating later

```bash
apm update              # pull the latest within your version constraints
apm install <pkg>       # add a new skill
```

---

### Option 2 — Manual install (Claude Code only)

For a quick try without APM:

```bash
git clone https://github.com/Aman-6875/devflow.git ~/devflow
mkdir -p ~/.claude/skills
ln -s ~/devflow/skills/* ~/.claude/skills/
```

Restart Claude Code and the skills will be available globally on your
machine. Use this for local testing; prefer **Option 1** for any
shared / team / CI workflow.

---

### Troubleshooting

- **`apm: command not found` after install** — re-open the shell, or
  add the install location to your `PATH` (the installer prints the
  exact path).
- **`apm install` fails to resolve** — confirm the repo path is
  correct (`Aman-6875/devflow/skills/<skill-name>`) and that you have
  network access to GitHub.
- **`No harness detected` error** — your `apm.yml` is missing the
  `targets:` field. Add it (see Step 2) or pass `--target <client>`
  on the command line.
- **Skills not triggering in your AI client** — restart the client
  after `apm install`. Confirm the deployed files exist (e.g. for
  Claude Code: `ls .claude/skills/`).
- **`unpinned` warning** — for reproducible installs, pin each
  dependency with `#v0.1.0` (or any tag/sha) as shown in Step 2.

---

## Usage

After install, invoke skills explicitly in your AI client:

```
/planner    — paste ticket details or describe the task
/executor   — after approving planner's output
/tester     — to add tests once code is written
/pr-creator — to open a PR once the branch is ready
/reviewer   — to review a PR or diff
/bug-finder — to debug a reported bug / error
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
