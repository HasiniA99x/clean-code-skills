# clean-code-skills

Experimental **skill definitions** for improving existing codebases—especially
AI/vibe-coded ones—by removing waste and simplifying code. Not an application,
CLI, or cleanup engine. Not primarily an audit-report tool.

## What `code-cleanup` does

**Mission:** improve the codebase. Analysis exists only to make cleanup safe.

**Target:** minimum justified complexity + required behavior preserved + better
quality + less unnecessary work + safer code.

Default mode **CLEAN**:

```text
Understand → Baseline → Find next cleanup → Understand why → Fixability Gate
→ Propose → Human approves → Fix → Verify → Accept → Next
```

Until nothing else is clearly worth changing safely.

Optional (explicit only):

- **ANALYZE** — full/module findings report (no edits)
- **REVIEW** — diff/branch/PR scope-creep check

Prefer REMOVE → CONSOLIDATE → SPECIALIZE → SIMPLIFY → REUSE.
**Used ≠ necessary.** Do not increase conceptual complexity as "cleanup."

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
| `code-cleanup/SKILL.md` | Mission, CLEAN workflow, safety, HITL, verify |
| `references/cleanup-checklist.md` | Search checklist + Analyze classification detail |
| `references/vibe-code-smells.md` | AI/vibe smell symptoms |
| `references/tooling-hints.md` | Optional mechanical tools (+ optional Jev note) |

## Modes

| Mode | When |
|------|------|
| **CLEAN** (default; `APPLY` alias) | "clean / cleanup / improve / simplify / remove vibe code" |
| **ANALYZE** | Explicit audit/report request |
| **REVIEW** | Explicit diff/branch/PR/agent-change review |

## Example prompts

- `Run code-cleanup on this repository.` → CLEAN
- `Run code-cleanup in Analyze mode.`
- `Run code-cleanup in Review mode against the parent branch diff.`

## Testing the skill

Try: clean repo (expect few/no changes), known-debt repo, vibe-coded repo.
Measure: accepted fixes, wasted human asks, net files/deps/abstractions removed,
change-budget stops, regressions, Keep/Questionable/Revert ratings.

## Design stance

- Fix the code; don't audit for its own sake
- One cleanup at a time; human approval mandatory
- Uncertainty → leave alone and continue
- Optional Jev may gate structured safety checks only — not required
- No subagents, CLI, or cleanup product in this repo
