# clean-code-skills

Reusable AI coding skill that **cleans existing code**. It removes waste, simplifies bulky code, and fixes real quality, performance, and security issues — without changing how the app is supposed to behave.

This repo is skill docs only. There is no CLI or app to run. Any coding agent that can follow a `SKILL.md` file can use it (Cursor, Claude Code, GitHub Copilot agent, Windsurf, Continue, and similar tools).

## How to use

### 1. Add the skill

Copy the `code-cleanup/` folder into the skills location your tool uses.

**This project only** (typical paths):

- `.cursor/skills/code-cleanup/`
- `.claude/skills/code-cleanup/`
- `.github/skills/code-cleanup/`
- `.agents/skills/code-cleanup/`

**All your projects** (typical paths):

- `~/.cursor/skills/code-cleanup/`
- `~/.claude/skills/code-cleanup/`

If your tool does not auto-load skills, open [`code-cleanup/SKILL.md`](code-cleanup/SKILL.md) in the chat or paste:

```text
Follow the code-cleanup skill at <path-to>/code-cleanup/SKILL.md
```

Then open the **repo you want cleaned** (not this skills repo) and start an agent chat.

### 2. Ask it to clean

In the target repo, say one of:

```text
Run code-cleanup on this repository.
```

```text
Clean this repo. Start with the notification service.
```

```text
Simplify the vibe-coded parts of apps/api.
```

That starts **CLEAN** (default). It does **not** write a full audit first.

It will:

1. Understand the repo
2. Find **one** worthwhile cleanup
3. Show a short summary and ask you **one** question
4. Wait for your answer
5. Fix only that item (after you say Yes)
6. Verify, then ask you to keep or revert
7. Repeat until nothing else is clearly worth changing

You approve every source change. One cleanup at a time.

### 3. How to answer in chat

Each message is short: what it found, what it wants to do, then one question.

Example:

```text
F4 — Extra provider setup

What I found:
- The app sends push through Expo only
- Extra factory/interface exists for switching providers

What I want to do:
- Remove the unused switch setup and keep Expo

Apply this cleanup?

1. Yes
2. Show why
3. Change approach
4. Skip
5. Stop
```

Reply with the number or the same words (`Yes`, `Skip`, …). Use **Show why** if you want more detail.

After a fix it asks **Keep this cleanup?** (`Accept` / `Revise` / `Revert` / `Show diff` / `Stop`).

### 4. Other modes (only if you ask)

| Mode | When to use | Example |
|------|-------------|--------|
| **CLEAN** (default) | Fix the code | `Run code-cleanup on this repository.` |
| **ANALYZE** | Report only, no edits | `Run code-cleanup in Analyze mode.` |
| **REVIEW** | Check a branch/PR/agent diff | `Run code-cleanup in Review mode against the parent branch.` |

You can also limit scope: `Clean only apps/notification-service.`

## What’s in this repo

```
code-cleanup/
├── SKILL.md                 # agent instructions
└── references/
    ├── cleanup-checklist.md
    ├── vibe-code-smells.md
    └── tooling-hints.md
```

You normally only invoke the skill. The agent reads the references when needed.

## What it will not do

- Change product behavior or public APIs without your approval
- Rewrite the architecture because another design looks nicer
- Add files, layers, or dependencies unless truly needed
- Commit after every fix (commits only if you ask)
- Keep going when remaining issues are taste, guesses, or unsafe to change

## Benchmark

Evaluate this skill against a seeded NestJS app:

- Target repo: [vibe-cleanup-benchmark](https://github.com/HasiniA99x/vibe-cleanup-benchmark)
- Human gold set, scoring rules, and a reference-run score: [`docs/vibe-cleanup-benchmark-eval.md`](docs/vibe-cleanup-benchmark-eval.md)

Keep the gold set **outside** the benchmark checkout when the cleaner runs (blind evaluation).

## User experience feedback

Session notes from people who have run the skill: [`docs/user-experience-feedback.md`](docs/user-experience-feedback.md).
