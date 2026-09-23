# clean-code-skills

Experimental repository for reusable AI coding skills focused on improving
**existing** codebases—especially ones created or heavily modified with AI /
vibe coding—without turning cleanup into feature work or speculative redesign.

This repo contains **skill definitions only**. It does not ship an application,
CLI, API, or cleanup engine.

## What `code-cleanup` does

`code-cleanup` is an agent skill that guides a coding agent to:

1. **Understand** a target repository (structure, conventions, commands, tests)
2. **Baseline** build/test/lint health without weakening checks
3. **Audit** for concrete maintainability, correctness, security, structural
   issues, and AI-generated accidental or speculative complexity
4. **Classify and decide** what is safe to change vs report-only
5. Optionally **review** a branch or worktree diff for scope creep before merge
6. Optionally **apply** small, verified cleanups that preserve intended behavior
   under mandatory human-in-the-loop review (one finding at a time)
7. **Report** what changed, what remains, and what is uncertain

The skill strongly prefers simplification, consolidation, and deletion over
creating new files, abstractions, or dependencies. **Used does not mean
necessary**—referenced, compiling code may still be unjustified complexity.

A clean or mature repository may correctly receive **zero** code changes.

## Repository structure

```
clean-code-skills/
├── README.md
└── code-cleanup/
    ├── SKILL.md
    └── references/
        ├── cleanup-checklist.md
        ├── vibe-code-smells.md
        └── tooling-hints.md
```

| Path | Role |
|------|------|
| `code-cleanup/SKILL.md` | Skill entrypoint: principles, modes, workflow, hard constraints, guards, report format |
| `code-cleanup/references/cleanup-checklist.md` | Technology-neutral audit checklist |
| `code-cleanup/references/vibe-code-smells.md` | Investigation guide for AI/vibe-code smells |
| `code-cleanup/references/tooling-hints.md` | Optional per-language tools for HIGH-confidence mechanical evidence |

## Analyze vs Review vs Apply

| Mode | Behavior |
|------|----------|
| **ANALYZE** | Discover → Baseline → Audit → Classify → Decide → Plan → Report (full repo or scoped module). **Does not modify source files.** |
| **REVIEW** | Diff-scoped audit (`<parent>...HEAD` or worktree) for Intended / Unrelated / Scope creep / Speculative addition. Remediations (including per-hunk revert) only via Human-in-the-loop. |
| **APPLY** | Human-guided: dedicated branch; present one finding; wait for approval; apply only that finding; verify; one commit per ACCEPT; REVERT via `git revert`; wait for ACCEPT/REVISE/REVERT; then continue. **No automatic modification.** |

If mode is unspecified, prefer **ANALYZE** first, then Apply under human-in-the-loop review. For “check this agent diff before merge,” prefer **REVIEW**.

Details live in [`code-cleanup/SKILL.md`](code-cleanup/SKILL.md).

## How to use the skill

Point a coding agent at this skill (for example by installing or referencing
`code-cleanup/SKILL.md` in your agent skills path) and run it against a **target
repository** that is not this skills repo.

Example prompts:

- `Run code-cleanup in Analyze mode on this repository.`
- `Run code-cleanup in Review mode against the parent branch diff.`
- `Run code-cleanup in Apply mode. Start with finding F1 only.`

## How to test the skill

Evaluate the skill against multiple real repositories—not against this
definition repo alone.

### Recommended target repositories

1. **Reasonably clean repository** — Expect few or no Apply changes; watch for false positives and unnecessary "improvements."
2. **Repository with known technical debt** — Expect useful findings; compare agent output to known debt.
3. **AI / vibe-coded repository** — Expect file proliferation, duplicate solutions, speculative abstractions, weak tests, and similar smells from `vibe-code-smells.md`.

### What to measure

| Metric | Why it matters |
|--------|----------------|
| True-positive findings | Skill catches real problems |
| False positives | Skill invents problems or over-flags style |
| Missed problems | Known issues the skill should have seen |
| Unnecessary files created | Creation guard effectiveness |
| Unnecessary abstractions introduced | Anti-abstraction bias effectiveness |
| Unrelated files changed | Scope control / diff discipline |
| Dependencies added | Dependency guard effectiveness |
| Test/build regressions | Verification discipline |
| Human rating of applied changes: **Keep** / **Questionable** / **Revert** | End quality of Apply mode |

### Suggested test loop

1. Snapshot or branch the target repo.
2. Run **ANALYZE**; score findings against a human review.
3. Run **APPLY** on a bounded subset (or full Apply if appropriate).
4. Inspect the diff and verification results.
5. Rate each change Keep / Questionable / Revert.
6. Record metrics; refine `SKILL.md` or references only when patterns repeat.
7. Optionally run **REVIEW** on an agent-produced branch before merge.

## Design stance

- Evidence over assumptions; uncertainty is reported, not guessed away.
- Used does not mean necessary; working does not mean justified.
- Existing repository patterns beat imported "best practices."
- Principles such as SOLID or DRY are optional reasoning aids, not mandatory checklists.
- Cleanup must not become a vehicle for new features, contract changes, or broad rewrites.
