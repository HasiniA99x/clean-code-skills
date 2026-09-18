---
name: code-cleanup
description: >-
  Safely analyze and clean existing codebases for maintainability, readability,
  consistency, security, and structural quality while preserving intended
  behavior. Prefer deletion and consolidation over new abstractions. Use when
  asked to clean up, refactor safely, reduce tech debt, audit AI/vibe-coded
  repos, or run code-cleanup in Analyze or Apply mode.
disable-model-invocation: true
---

# Code Cleanup

Reusable engineering skill for improving an existing codebase without creating
more unnecessary code. This is not a style guide and not a mandate to apply
SOLID, DRY, Clean Architecture, or design patterns. Those ideas are reasoning
tools only when they reduce a concrete problem in the current repository.

A mature codebase may require **zero** cleanup changes. Concluding that no
modification is justified is a successful outcome.

## When to use

- User asks to clean up, tidy, simplify, or reduce debt in an existing repo
- Repository was created or heavily modified with AI / vibe coding
- User requests Analyze or Apply mode for `code-cleanup`

## When not to use

- Greenfield feature work or intentional redesign
- Style-only reformatting with no maintainability problem
- Tasks that require changing business behavior or public contracts

## Core principles

Prefer:

- simplification over abstraction
- consolidation over creation
- deletion over duplication
- existing repository patterns over introducing new patterns
- small verified changes over broad rewrites
- evidence over assumptions
- reporting uncertainty instead of guessing

Understand the repository before modifying it.

## References (read when needed)

- [cleanup-checklist.md](references/cleanup-checklist.md) — technology-neutral audit checklist
- [vibe-code-smells.md](references/vibe-code-smells.md) — AI/vibe-code smell investigation guide

## Operating modes

### ANALYZE

Analyze the repository and produce findings and a cleanup plan.
**Do not modify source files.**

### APPLY

Run a mandatory human-in-the-loop session: present findings for individual
human decisions, apply at most one approved finding at a time, verify, then
request post-change human review before continuing.

**No finding may be modified automatically.**

If the user does not specify a mode, ask which mode to use. If they say
"cleanup" without mode, default to **ANALYZE** first, then offer Apply under
human-in-the-loop review.

---

## Workflow

Follow these stages in order. Do not skip Discover or Baseline before Audit.
In Analyze mode, stop after Plan (produce the Analyze report; do not Clean).
In Apply mode, after Analyze classification exists, follow
**Human-in-the-loop (Apply)** — one finding at a time with human approval.

### 1. DISCOVER

- Read repository instructions and documentation (`README`, `AGENTS.md`,
  `CONTRIBUTING`, `.cursor/rules`, etc.).
- Inspect repository structure.
- Identify architecture, modules, entry points, and boundaries.
- Identify dependencies.
- Identify tests, build, lint, typecheck, and static-analysis commands.
- Search for existing implementations before assuming functionality is missing.
- Determine dominant repository conventions.
- Do not infer architecture from a single file.

### 2. BASELINE

- Run available build/compile.
- Run existing tests.
- Run lint/typecheck/static analysis when configured.
- Record pre-existing failures.
- Never weaken tests or configuration just to obtain a green baseline.

### 3. AUDIT

Inspect for:

- correctness problems
- duplication
- dead code
- unnecessary files
- unnecessary abstractions
- excessive complexity
- inconsistent patterns
- architecture/responsibility issues
- dependency problems
- error-handling problems
- obvious security issues
- test weaknesses
- obvious performance problems
- stale or noisy documentation
- AI/vibe-code artifacts

Use [cleanup-checklist.md](references/cleanup-checklist.md). For AI-heavy
repos, also use [vibe-code-smells.md](references/vibe-code-smells.md).

### 4. CLASSIFY

**Finding types:** Correctness, Security, Architecture, Maintainability,
Duplication, Dead Code, Dependency, Performance, Testing, Documentation.

Every finding must include a **confidence** level independently from priority.

#### Confidence

Confidence describes certainty that the finding is real.
Priority describes impact if the finding is real.
Never use priority as a substitute for confidence.
Do not raise priority because a hypothetical consequence could be severe.

**HIGH CONFIDENCE**

Directly demonstrated through one or more of:

- execution
- failing/passing tests
- compiler/build output
- lint/static-analysis output
- reproducible runtime behavior
- clear code/configuration contradiction

**MEDIUM CONFIDENCE**

Strong static evidence exists, but runtime impact or intent has not been
directly demonstrated.

**LOW CONFIDENCE**

A potential issue exists, but important context, ownership, intent, runtime
behavior, or external usage is unknown.

Low-confidence findings should normally be report-only until verified.

#### Priority

Assign the lowest priority that is supported by evidence.
Do not increase priority based only on hypothetical consequences.
When runtime impact is unverified, explicitly state that uncertainty.
Documentation drift should normally be Low unless it directly causes
incorrect operation.
A code smell is not automatically a defect.

**CRITICAL**

Use only for confirmed issues such as:

- exploitable severe security vulnerability
- realistic data loss/corruption
- system cannot safely operate

**HIGH**

Use for:

- confirmed incorrect runtime behavior affecting an important supported path
- confirmed security weakness
- confirmed build/deployment failure
- important required behavior demonstrably broken

**MEDIUM**

Use for:

- maintainability problems with concrete engineering cost
- test/reliability weaknesses
- architectural or consistency problems with evidence they increase defect or
  maintenance risk
- incomplete implementation where impact is meaningful but not critical

**LOW**

Use for:

- localized maintainability improvements
- stale documentation
- minor cleanup
- small inconsistencies with limited operational impact

### 5. DECIDE

Before changing code, answer:

- Is there concrete evidence this is a problem?
- Is intended behavior sufficiently understood?
- Can observable behavior be preserved?
- Is this within cleanup scope?
- Can the change be small and coherent?
- Can the result be verified?

Place each item into exactly one bucket:

- **Actionable** — sufficient evidence and understanding to justify cleanup
- **Report-only** — real or potential problem that must not be changed yet
  (unclear intent/ownership, possible external consumers or public contracts,
  uncertain runtime impact, product/architecture decision required, or
  insufficient verification). Low-confidence findings normally land here.
- **Non-actionable observation** — stylistic preference, intentional pattern,
  documented placeholder, alternative valid design, speculative improvement,
  or smell without demonstrated engineering impact

If uncertain, **REPORT** the finding (report-only) instead of modifying the code.
Non-actionable observations must not become cleanup tasks unless new evidence
changes their classification.

### 6. PLAN

The cleanup plan must contain **only actionable findings**.
Report-only findings must not become automatic cleanup tasks.
Non-actionable observations must never appear in the cleanup plan.

Before editing, create a minimal cleanup plan. Prefer a concise table:

| ID | Problem | Files | Action | Benefit | Risk | Verification |

Reference finding IDs; do not restate the full analysis in the plan.

Keep the plan minimal. Prefer few high-confidence changes over many speculative ones.

### 7. CLEAN (Apply mode only)

Follow **Human-in-the-loop (Apply)**.

Apply source changes only after individual human **APPROVE** for that finding
ID. Apply at most one finding per approval cycle. Do not batch findings unless
the human explicitly asks to batch those specific finding IDs.

### 8. VERIFY (Apply mode only)

After each approved finding is applied, run that finding's verification plan,
inspect the git diff, confirm the change surface, present the post-change
report, and stop for human **ACCEPT** / **REVISE** / **REVERT** / **STOP**.

Also run broader build/tests/lint/typecheck/static analysis when they are
part of the finding's verification plan or needed to confirm no regression.

Inspect the diff for:

- no accidental files
- no unrelated modifications
- no unintended behavior changes
- no suppressed tests/checks
- no unnecessary dependencies

If the diff expands unexpectedly, stop and reassess (do not continue).

### 9. REPORT

#### Analyze mode

Use this structure. Avoid repeating the same evidence in multiple sections.
The Executive Summary should summarize rather than duplicate findings.
The Cleanup Plan should reference finding IDs rather than restating the full
analysis. Keep detailed evidence with the original finding.
Prefer concise evidence-backed findings over long lists of speculative smells.

**Title:** `# Code Cleanup Analysis`

**Executive Summary** — Briefly state repository health, highest-impact
findings, whether cleanup is justified, and important verification limitations.

**Baseline** — Show only checks actually executed. Never imply that an
unexecuted check passed.

Example baseline lines:

    Build                 PASS
    Backend tests         PASS
    Mobile tests          FAIL — 2 pre-existing failures
    Lint                  PASS with warnings
    Full pipeline          NOT RUN — tool unavailable

**Actionable Findings** — Problems with sufficient evidence and understanding
to justify a cleanup action. Each finding must contain: ID, Title, Type,
Priority, Confidence, Evidence, Impact, Decision, Proposed action,
Verification.

Example:

    F1 — Port configuration drift
    Type: Correctness
    Priority: Medium
    Confidence: High
    Evidence: ...
    Impact: ...
    Decision: Cleanup candidate
    Proposed action: ...
    Verification: ...

**Report-Only Findings** — Potential or real problems that should NOT
currently be automatically changed because intent/ownership is unclear,
external consumers or public contracts may be affected, runtime impact is
uncertain, architecture/product decisions are required, or verification is
insufficient. Each must state **Reason report-only**.

Example:

    R1 — Incomplete request-id propagation
    Type: Maintainability
    Priority: Medium
    Confidence: Medium
    Evidence: ...
    Reason report-only:
    Notification service does not consume the header, but intended observability
    requirements are not established.

**Non-Actionable Observations** — Stylistic preferences, intentional patterns,
documented placeholders, alternative valid designs, speculative improvements,
or smells without demonstrated engineering impact. These must NOT appear in
the cleanup plan.

Example:

    O1 — Duplicate DTO definitions
    Reason:
    Duplication exists, but no behavioral drift or maintenance problem was
    demonstrated. Consolidation would currently be architectural preference.

**Cleanup Plan** — Include ONLY actionable findings (finding IDs). Prefer:

    | ID | Problem | Files | Action | Benefit | Risk | Verification |

**Risks / Unverified Areas** — State commands not executed, runtime behavior
not verified, external consumers not verified, and ownership/product
decisions required.

**Modification Summary**

    Files Changed: None
    Files Created: None
    Files Deleted: None
    Dependencies Changed: None
    Behavior Changes: None

For Analyze mode, Modification Summary values should normally remain None.

#### Apply mode

When all findings have been reviewed or the human stops the session, report:

```markdown
## Resolved findings

## Skipped findings

## Deferred findings

## Remaining report-only findings

## New findings discovered during cleanup

## Files changed

## Files created

## Files deleted

## Dependencies changed

## Verification performed

## Verification not performed

## Behavior changes

## Remaining risks
```

Also maintain and include session progress totals (see Human-in-the-loop).
Do not treat skipped, deferred, or remaining report-only items as resolved.

---

## Human-in-the-loop (Apply)

Mandatory for all source modifications. Sequence:

ANALYZE → classify findings → human review → fix **ONE** finding → verify →
human review → continue to next finding

### Human-in-the-loop rule

No finding may be modified automatically.

Every finding must receive individual human approval before source changes
are made.

Approval for one finding does not authorize changes for another finding.

Do not batch multiple findings under one approval unless the human
explicitly asks to batch those specific finding IDs.

### Before applying a finding

Present:

- Finding ID and title
- Type
- Priority
- Confidence
- Evidence
- Why it should be changed
- Exact proposed remediation
- Expected files to change
- Expected files to create
- Expected files to delete
- Expected dependency changes
- Expected behavior changes
- Risk
- Verification plan

Then stop and request a human decision.

Supported decisions: **APPROVE**, **MODIFY APPROACH**, **INVESTIGATE**,
**SKIP**, **DEFER**, **STOP**.

### APPROVE

Apply ONLY the approved finding.

Do not perform opportunistic cleanup.

Do not modify unrelated code.

Do not fix another finding while touching the same file.

If another cleanup opportunity is discovered, record it as a new finding
and leave it unchanged.

### MODIFY APPROACH

Do not change source.

Update the proposed remediation according to human feedback and present it
again for approval.

### INVESTIGATE

Do not change source.

Gather additional evidence needed to resolve uncertainty.

After investigation:

- update confidence if justified
- update priority if justified
- reclassify Report-Only → Actionable only when evidence supports it
- explain why the classification changed

Modification still requires separate **APPROVE**.

### SKIP

Do not modify the finding.
Record it as skipped by human decision.

### DEFER

Do not modify the finding.
Keep it in the final report as deferred.

### STOP

Stop cleanup immediately.
Do not continue to another finding.

### Post-change human review

After applying ONE finding:

1. Run the verification defined for that finding.
2. Inspect the git diff.
3. Confirm the actual change surface.
4. Report:

```text
Finding:
Files changed:
Files created:
Files deleted:
Dependencies changed:
Verification executed:
Verification result:
Behavior changes:
Unexpected changes:
Remaining risk:
```

Then stop again.

Ask the human to choose: **ACCEPT**, **REVISE**, **REVERT**, **STOP**.

### ACCEPT

Mark the finding resolved.

Only then present the next unresolved finding.

### REVISE

Keep the finding active.

Explain the required adjustment and request approval before making
additional modifications if the adjustment expands the previously
approved change.

### REVERT

Revert ONLY changes introduced for that finding.

Do not revert pre-existing user changes or unrelated working-tree changes.

Verify the revert and report the result.

### Important change-scope rule

While fixing finding `F<n>`:

Allowed change surface = only changes necessary to resolve that finding using the
approved remediation.

Never use "while we're here", "for consistency", "related cleanup", or
"small improvement" as justification for expanding the change.

Any newly discovered issue must become a separate finding.

### Report-only findings

Report-Only findings must also participate in the human loop.

Present the finding and why it is report-only.

Allow: **INVESTIGATE**, **DEFER**, **SKIP**.

Do not offer **APPROVE** until sufficient evidence allows the finding to be
reclassified as Actionable.

Reclassification itself does not authorize modification.

### Non-actionable observations

Do not automatically modify them.

Allow the human to: **ACKNOWLEDGE**, **INVESTIGATE**, **PROMOTE FOR ANALYSIS**,
**SKIP**.

If promoted, analyze it as a new finding.
Do not directly modify code.

### Session progress

Maintain cleanup progress:

```text
Resolved:
Skipped:
Deferred:
Under investigation:
Pending actionable:
Pending report-only:
```

Never silently drop a finding.

---

## Hard constraints

The cleanup agent MUST NOT:

- introduce new features under the label of cleanup
- intentionally change business behavior
- change public contracts without explicit approval
- redesign architecture simply because another design looks cleaner
- introduce speculative abstractions
- create unnecessary interfaces, factories, adapters, wrappers or layers
- add dependencies for trivial functionality
- perform unrelated framework upgrades
- perform unrelated dependency upgrades
- mass rename code for stylistic reasons
- remove apparently unused code without checking indirect/dynamic usage
- suppress tests, lint rules, compiler errors or security checks
- rewrite working code solely because another implementation style is preferred

## Creation guard

Before creating ANY new source file:

1. Search for an existing owner of the responsibility.
2. Search for similar implementations.
3. Determine whether existing code can be consolidated.
4. Create the file only if it represents a clear responsibility that cannot
   reasonably belong to an existing module.

Treat unexpected growth in file count as a cleanup warning.

## Dependency guard

Before adding ANY dependency:

1. Check existing dependencies.
2. Check framework/platform capabilities.
3. Search existing repository utilities.
4. Add the dependency only when its benefit clearly justifies its maintenance
   and security cost.

Cleanup normally removes or consolidates dependencies; adding one is exceptional
and must be justified in the report.

## Repository search guard

When searching the repository:

- Prefer source-controlled application paths first.
- Exclude dependency, generated, cache, build-output, and VCS directories
  unless they are specifically relevant to the investigation.

Examples include:

- `.git/`
- `node_modules/`
- `.next/`
- `dist/`
- `build/`
- `bin/`
- `obj/`
- `coverage/`

Do not treat matches from generated or dependency directories as evidence
of application usage.

If a search command fails, its output must not be used as evidence.
Correct the command or use another verification method.

For `git log --follow`, inspect one path at a time because Git does not
support following multiple paths in a single invocation.

Do not perform broad repository-wide searches when a narrower source or
module-level search can answer the question.

## Principles are not mandates

Do not make subjective style preferences mandatory.

Do not blindly apply SOLID, DRY, design patterns, Clean Architecture, or other
principles. They may be useful reasoning tools, but applying them must reduce a
concrete problem in the current repository.

## Stop conditions

Stop and report (do not force changes) when:

- intended behavior is unclear
- cleanup would require public API or contract changes without approval
- verification cannot be performed and risk is non-trivial
- the only "improvement" is stylistic preference
- no finding has enough evidence to justify modification
