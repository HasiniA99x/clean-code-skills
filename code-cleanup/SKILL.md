---
name: code-cleanup
description: >-
  Safely analyze and clean existing codebases for maintainability, readability,
  consistency, security, and structural quality while preserving intended
  behavior. Detect used-but-unnecessary and speculative AI-generated complexity.
  Prefer deletion and consolidation over new abstractions. Use when asked to
  clean up, refactor safely, reduce tech debt, audit AI/vibe-coded repos, or
  run code-cleanup in Analyze or Apply mode.
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

**Used does not mean necessary. Working does not mean justified.**
Evaluate whether complexity supports a demonstrated current requirement,
contract, invariant, integration, operational need, or established repository
boundary. Prefer the simplest implementation that preserves those
demonstrated responsibilities.

Do not interpret this as permission to aggressively delete working code.
Absence of immediately visible evidence is NOT proof that something is
unnecessary.

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
- **unjustified complexity** (used and working, but more machinery than
  demonstrated responsibility requires)
- speculative functionality
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

When meaningful complexity is encountered, do not stop after determining that
the code is referenced. Perform **Necessity analysis** (below).

### 4. CLASSIFY

**Finding types:** Correctness, Security, Architecture, Maintainability,
Duplication, Dead Code, Dependency, Performance, Testing, Documentation,
**Unjustified Complexity**.

**Unjustified Complexity** means code that is used and functional but introduces
more machinery, flexibility, states, indirection, infrastructure, or capability
than the demonstrated responsibility requires (for example speculative
abstractions, single-use generics, wrappers that only forward, hypothetical
provider/plugin systems, unused configuration, premature caching/scalability,
redundant resilience, compatibility layers without a demonstrated need, or
"nice-to-have" AI-generated capability).

Distinguish:

- **Dead code** — not used
- **Unjustified complexity** — used, but more machinery than demonstrated
  responsibility requires
- **Speculative functionality** — working capability with no demonstrated
  current or committed requirement

Suspected unjustified complexity must **not** automatically become Actionable.
Use the existing buckets:

- **Actionable** — clearly unnecessary; required behavior understood;
  simplification small; capability loss understood; verification possible
- **Report-only** — strong evidence of unnecessary complexity, but ownership,
  external consumers, runtime usage, future commitment, or behavioral impact
  is uncertain
- **Non-actionable** — justified by a demonstrated responsibility or
  established boundary, or the concern is merely architectural/style preference

When uncertain, prefer Report-Only.

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
- unjustified complexity with evidenced unnecessary machinery and understood
  capability loss
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

## Necessity analysis

When meaningful complexity is encountered, ask:

1. What current responsibility does this code serve?
2. What current requirement, contract, invariant, integration, operational
   need, or established repository convention requires that responsibility?
3. Is the amount of complexity proportional to that responsibility?
4. Could the same required behavior be implemented materially more simply
   using existing repository patterns?
5. What concrete capability would be lost if this code were removed,
   consolidated, specialized, or simplified?
6. Is that lost capability currently required?
7. Is it part of an explicitly committed near-term requirement?
8. Is the complexity protecting an external/public contract?
9. Is it an intentional architecture boundary?
10. Is it required for testing, security, infrastructure isolation,
    dependency inversion, plugin loading, or runtime configuration?
11. Does repository history/documentation provide evidence for why it exists?
12. Is another framework/library/platform layer already providing the same
    capability?

### Capability justification

For suspected unjustified complexity, explicitly identify:

```text
Current required capability:
Additional capability introduced by the complexity:
Evidence that the additional capability is required:
Simpler alternative:
Capability lost by simplification:
Evidence that the lost capability is acceptable:
Uncertainty:
```

Do not allow findings such as "This factory looks unnecessary." Require the
capability justification fields above.

### Weak justifications

These statements alone are NOT sufficient justification for keeping complexity:

- "might be useful later" / "future-proof" / "just in case" / "nice to have"
- "more flexible" / "more scalable" / "more robust"
- "clean architecture" / "follows SOLID" / "follows DRY" / "best practice"
- "supports future providers" / "allows future extension"

Require repository-specific evidence. These phrases are also NOT proof that
the code should be removed. Investigate first.

### Justification hierarchy

Prefer evidence in roughly this order:

1. Current executable behavior / runtime usage
2. Public or external contracts
3. Current product/business requirements
4. Security or correctness invariants
5. Deployment/infrastructure requirements
6. Tests demonstrating required behavior
7. Runtime configuration
8. Established repository architecture/boundaries
9. Repository documentation
10. Explicitly committed near-term requirements

Weak evidence: TODO without context, speculative comments, generic best
practices, architectural preference, hypothetical future use.

### Preserve intentional boundaries

Do NOT automatically remove an abstraction merely because it has one
implementation, few callers, a simple implementation, or because direct calls
would use fewer lines.

A single implementation may still be a justified boundary for external/public
contracts, established DI, testing seams, security boundaries, infrastructure
isolation, plugins, dynamic loading, framework conventions, separate
ownership, or committed near-term requirements.

Require evidence before simplification.

### Prefer simplification, not replacement architecture

When an Unjustified Complexity finding is approved, prefer in order:

remove → consolidate → specialize → simplify → reuse existing repository pattern

Avoid replacing an old abstraction with a new "better" abstraction.
Cleanup must produce less conceptual machinery, not merely different machinery.

### Complexity delta check

For every approved Unjustified Complexity cleanup, compare before/after:

- files involved
- abstractions involved
- dependencies involved
- configuration/options involved
- meaningful branches/states involved

Do not require every numeric measure to decrease. Ask: did conceptual
complexity decrease while required capability remained? If the cleanup
introduces equal or greater conceptual complexity, **STOP** and reassess.

### Do not over-correct

Objective: minimum **justified** complexity for demonstrated responsibilities—
not minimum lines of code.

Do not:

- delete code merely because its requirement is not immediately obvious
- remove extension points merely because only one implementation exists
- simplify public contracts without explicit approval
- remove operational resilience without understanding failure requirements
- remove security checks because they appear redundant
- remove scalability mechanisms without understanding actual deployment
- replace working architecture with a preferred architecture
- treat line count as the measure of simplicity

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
  Suspected unjustified complexity with uncertain requirement ownership also
  lands here.
- **Non-actionable observation** — stylistic preference, intentional pattern,
  documented placeholder, alternative valid design, speculative improvement,
  smell without demonstrated engineering impact, or complexity justified by a
  demonstrated responsibility/boundary

If uncertain, **REPORT** the finding (report-only) instead of modifying the code.
Non-actionable observations must not become cleanup tasks unless new evidence
changes their classification.

For Unjustified Complexity, also confirm capability loss is understood and
verification can show required behavior remains.

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
Keep the final report concise and in simple English. Structured technical
detail is fine; do not repeat the same evidence in multiple sections.

---

## Human-in-the-loop (Apply)

Mandatory for all source modifications. Sequence:

ANALYZE → classify findings → human review → fix **ONE** finding → verify →
human review → continue to next finding

Cleanup logic, safety rules, priority, confidence, necessity analysis, and
approval requirements are unchanged. This section also defines how to talk to
the human in APPLY chat.

### Human-in-the-loop rule

No finding may be modified automatically.

Every finding must receive individual human approval before source changes
are made.

Approval for one finding does not authorize changes for another finding.

Do not batch multiple findings under one approval unless the human
explicitly asks to batch those specific finding IDs.

### Conversational UX (APPLY chat)

Deep analysis internally. Small decisions externally.

The human should never need to read a full audit report just to answer the
next cleanup question.

#### Simple English

All human-facing APPLY chat must use simple English:

- short sentences
- common words
- concrete explanations
- small paragraphs
- at most 3 short bullets when useful

Avoid formal audit language when a simpler phrase exists.
Avoid tables in interactive chat.

Chat labels (prefer in interactive chat):

- Unjustified Complexity → Extra complexity
- Actionable → Safe to fix
- Report-Only → Needs more information
- Non-Actionable → Leave it alone
- Current required capability → What the app needs today
- Additional capability introduced → What this extra code adds
- Proposed remediation → Suggested change

Show formal classification only when useful or the user asks.

#### Chunked chat

Never present the entire investigation or finding report at once.

Reveal information in small chunks:

FIND → explain ONE important point → ask ONE question → STOP →
human answers → explain the next relevant point → ask ONE question → STOP

Continue until enough information exists for the current decision.

#### Response size

For normal interactive messages:

- prefer 2-4 short sentences
- use at most 3 short bullets when useful
- then show the decision
- avoid tables
- avoid long technical summaries
- avoid repeating session progress after every interaction

Aim for roughly **5-8 short lines** before decision options.

Detailed information stays available on request:

- Show details
- Show evidence
- Show code
- Show affected files
- Show technical reasoning

#### Do not show internal report fields by default

Do not automatically show:

- Type, Priority, Confidence
- Capability justification
- Classification tables
- Full evidence or full uncertainty analysis
- Session progress
- Cleanup plan
- Detailed verification plan

Keep these internally. Show them only when necessary for the immediate
decision, or when the human asks.

#### One question / one decision at a time

When human information is needed:

1. Ask **exactly one** question **or** present **exactly one** decision.
2. STOP.
3. Wait for the answer before continuing.

Do not ask multiple requirement questions in one message.

Do not mix decisions from different stages in one prompt
(for example do not combine investigate / approve / skip / defer / evidence /
next finding when those belong to different stages).

Ask only what is relevant **now**.

#### Do not ask what the repository can answer

Before asking the human, investigate the repository (code, tests, config, docs,
history, patterns).

Ask the human only when the answer cannot be established with enough
confidence from the repository.

#### Progressive disclosure

Use this interaction pattern:

FIND → EXPLAIN ONE POINT → ASK ONE QUESTION IF NEEDED → WAIT →
(next point / INVESTIGATE conversationally) → SUGGEST SIMPLE CHANGE →
ASK FOR APPROVAL → WAIT → APPLY → VERIFY → SHOW SIMPLE RESULT →
WAIT FOR ACCEPTANCE

#### Investigation can end early

Do not continue investigating once enough information exists for the current
decision.

Example:

```text
I cannot prove whether another provider is required.

Without that information, I don't recommend changing this code.

Leave it for now?

1. Yes
2. Investigate further
3. Show details
```

Then STOP.

#### Handle "Not sure"

"Not sure" is a valid answer.

If the user is not sure:

- investigate further when possible
- do not pressure the user to approve
- do not assume the feature is unnecessary
- keep it as Needs more information (Report-Only) when uncertainty remains

### Simple finding presentation

Default format — one point, then one decision:

```text
F<n> — <simple title>

<2-4 short sentences on ONE important point.>

<one question or one decision>

1. ...
2. ...
3. ...
```

Example:

```text
F4 — Extra provider setup

The app sends push notifications through Expo.

I also found an interface and factory for switching providers, but I could
not find another provider in use.

Is another push provider planned?

1. Yes
2. No
3. Not sure
4. Show evidence
```

Then STOP.

### Before applying a finding

Internally prepare (and keep for the final report / on request):

- Finding ID and title
- Type, Priority, Confidence
- Evidence, Impact
- Why it should be changed
- Exact proposed remediation
- Expected files to change / create / delete
- Expected dependency and behavior changes
- Risk
- Verification plan
- For Unjustified Complexity: capability justification fields

In chat, do **not** dump that list by default.

Once enough information is known, use a simple approval prompt:

```text
F<n> — Suggested cleanup

<2-4 short sentences: what you propose and that behavior should stay the same.>

What do you want to do?

1. Apply this change
2. Change the approach
3. Skip it
4. Show details
```

Then STOP.

Map choices to existing decisions:

| Chat choice | Internal decision |
|-------------|-------------------|
| Apply this change | APPROVE |
| Change the approach | MODIFY APPROACH |
| Skip it | SKIP |
| Show details | reveal formal fields; then re-ask |
| (also support) Defer / Stop / Investigate | DEFER / STOP / INVESTIGATE |

Do not modify anything until the user chooses **Apply this change** (APPROVE).

Do not automatically simplify or delete working-but-unnecessary code.
**APPROVE** is still required before modification.

### APPROVE

Apply ONLY the approved finding.

Do not perform opportunistic cleanup.

Do not modify unrelated code.

Do not fix another finding while touching the same file.

If another cleanup opportunity is discovered, record it as a new finding
and leave it unchanged.

For Unjustified Complexity: follow remove → consolidate → specialize →
simplify → reuse existing pattern. After applying, run the **complexity
delta check**. If conceptual complexity did not decrease, STOP and reassess.

### MODIFY APPROACH

Do not change source.

Update the proposed remediation according to human feedback and present it
again for approval (simple chat form).

### INVESTIGATE

Do not change source.

Gather additional evidence needed to resolve uncertainty.

INVESTIGATE must also be conversational. Do **not** investigate everything and
then dump the complete result.

After investigation work:

1. Identify the **most important conclusion** first.
2. Explain that one point in 2-4 short sentences.
3. Ask **one** follow-up decision.
4. STOP.
5. Continue chunk by chunk until enough information exists for a decision—
   or until investigation can end early.

Example first message after investigation:

```text
NS-R2 — Push adapter setup

I checked how it is used.

Email looks fine, so I would leave it alone.

Push is different: both the generic PushAdapter and ExpoPushAdapter
are used directly.

Want to know why?

1. Yes
2. Skip this finding
3. Show technical details
```

Then STOP.

If the human chooses Yes, explain the next point only:

```text
The generic adapter handles sending.

But Expo is still used directly for circuit checks and receipt polling.

So the abstraction does not fully hide Expo.

Want me to check whether this can safely be simplified?

1. Yes
2. Leave it for now
3. Show code evidence
```

Then STOP.

Internally, after investigation:

- update confidence if justified
- update priority if justified
- reclassify Report-Only → Actionable only when evidence supports it
- keep the classification change explanation for the final report / Show details

Do not dump those updates in chat unless the human asks or they are required
for the immediate decision.

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
4. Keep the formal result for the final cleanup report.

In chat, show a **simple** result by default (not the full formal template):

```text
F<n> fixed.

Changed:
- <files changed or deleted>

Checked:
- <short verification results>
- No unexpected files were created (or list surprises)

Behavior:
<expected behavior change, or none>

Keep this change?

1. Accept
2. Revise
3. Revert
4. Show diff/details
5. Stop
```

Then STOP.

Map: Accept → ACCEPT, Revise → REVISE, Revert → REVERT, Stop → STOP.
Show diff/details reveals formal fields (files created/deleted, dependencies,
verification executed/result, unexpected changes, remaining risk, complexity
delta when relevant), then re-ask Keep this change?

### ACCEPT

Mark the finding resolved.

Only then present the next unresolved finding.

### REVISE

Keep the finding active.

Explain the required adjustment simply and request approval before making
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

In chat, explain simply that this **Needs more information** before any change.
Use the simple finding presentation. Ask at most one question if needed.

Allow: **INVESTIGATE**, **DEFER**, **SKIP** (worded simply for the user).

Do not offer **APPROVE** / Apply until sufficient evidence allows the finding
to be reclassified as Actionable (Safe to fix).

Reclassification itself does not authorize modification.

### Non-actionable observations

Do not automatically modify them.

In chat, explain simply that you recommend **Leave it alone**, unless the user
wants more investigation.

Allow the human to: **ACKNOWLEDGE**, **INVESTIGATE**, **PROMOTE FOR ANALYSIS**,
**SKIP** (worded simply).

If promoted, analyze it as a new finding.
Do not directly modify code.

### Session progress

Maintain cleanup progress **internally**:

```text
Resolved:
Skipped:
Deferred:
Under investigation:
Pending actionable:
Pending report-only:
```

Never silently drop a finding.

Do **not** print session progress after every finding interaction.

Show progress only:

- when the human asks
- when switching major phases
- when the human stops
- in the final report

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
