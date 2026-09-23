# Cleanup Checklist

Technology-neutral audit checklist for the `code-cleanup` skill. Use across
TypeScript/JavaScript, C#, Python, Java, and similar application codebases.

Language-specific notes below are examples only—not mandatory rules.

A checked item means "investigated with evidence," not "must change." A mature
repository may warrant no changes.

---

## How to use

1. Complete Discover and Baseline first.
2. Work through each section; record findings with type and priority.
3. Mark items as: **OK**, **Finding**, **Uncertain (report only)**, or **N/A**.
4. Prefer evidence (call sites, tests, runtime paths, ownership) over intuition.

---

## Correctness

- [ ] Observable behavior matches documented or tested intent where intent is known
- [ ] Edge cases that the domain requires are handled (not every imaginable edge)
- [ ] Null/empty/missing-input paths are intentional, not accidental omissions
- [ ] Concurrency, ordering, and idempotency assumptions are consistent with usage
- [ ] Numeric, date/time, and encoding handling matches domain needs
- [ ] Partial failures leave the system in a safe, understandable state
- [ ] Regressions are not hidden by weakened or skipped checks

**Examples (not rules):** off-by-one in pagination; incorrect status codes; silent
swallow of exceptions that should surface.

---

## Duplication

- [ ] Near-identical logic exists in multiple places with the same responsibility
- [ ] Copy-paste variants have drifted and cause inconsistent behavior
- [ ] Shared constants/config are redefined instead of reused from one source
- [ ] Parallel "helper" modules solve the same problem differently

**Safe cleanup signal:** Same inputs → same outputs, clear single owner after merge.

**Do not force DRY:** Similar shapes that serve different domains or evolve
independently may correctly remain separate.

---

## Complexity

- [ ] Functions/methods do many unrelated things in one body
- [ ] Nesting, branching, or state combinations obscure the happy path
- [ ] Conditionals encode policies that belong closer to the decision owner
- [ ] "Clever" indirection makes the main flow harder to follow than a direct version
- [ ] Large files mix unrelated concerns without a clear module boundary

**Prefer:** Extract only when it clarifies an existing responsibility—not to
reach a line-count target.

---

## Necessity / complexity

Use when code is referenced and working, but may still be unjustified.

Canonical rules: **Necessity analysis** in `SKILL.md` (capability justification,
weak justifications, justification hierarchy, intentional boundaries,
do-not-over-correct). Do not restate those criteria here.

Checklist only:

- [ ] Is this complexity required by current behavior?
- [ ] What capability disappears if removed, consolidated, or specialized?
- [ ] Is that capability actually required (current or committed near-term)?
- [ ] Is the repository already solving this problem elsewhere?
- [ ] Is this abstraction protecting a real boundary (contract, DI, test seam,
      security, plugin, framework convention)?
- [ ] Is this functionality speculative / "nice to have" / future-proofing?
- [ ] Is future-proofing supported by an actual commitment?
- [ ] Could existing repository patterns express the same required behavior more
      simply?
- [ ] Would simplification reduce conceptual machinery (not merely replace it)?

Prefer Report-Only when uncertain. Optional mechanical evidence:
[tooling-hints.md](tooling-hints.md).

---

## Architecture / responsibility

- [ ] Module boundaries match how the repository actually organizes work
- [ ] A unit owns one coherent responsibility relative to local conventions
- [ ] Cross-layer calls skip established boundaries without reason
- [ ] UI, domain, persistence, and integration concerns are mixed in ways that
      already cause defects or change friction (not merely "impure")
- [ ] Feature logic lives far from related tests or callers without benefit

**Do not:** Redesign toward a preferred architecture (e.g. Clean Architecture)
unless the current structure causes a concrete, evidenced problem and the
change stays small and verifiable.

---

## Dead code

- [ ] Unreachable code paths after conditionals or returns
- [ ] Unused exports, types, classes, or modules with no references
- [ ] Feature flags permanently off / abandoned branches
- [ ] Commented-out code blocks left "just in case"
- [ ] Orphan files not referenced by build, entry points, or tooling

**Before deleting:** Search static references, dynamic/reflective usage, stringly
routed names, DI registration, config-driven loading, CLI entry points, and
serialization contracts.

**If usage might be dynamic:** Report as uncertain; do not delete automatically.

---

## Dependencies

- [ ] Declared dependencies are used
- [ ] Multiple libraries overlap for the same capability
- [ ] Trivial utilities could use language/stdlib/framework instead
- [ ] Versions or packages are inconsistent across the repo without reason
- [ ] Transitive risk (abandoned, overly broad permissions, known issues) is obvious

**Do not:** Perform unrelated framework or dependency upgrades as "cleanup."

---

## Error handling

- [ ] Errors are empty-caught, logged-and-ignored, or converted to success
- [ ] Error types/messages lose information needed by callers or operators
- [ ] Retries lack bounds, backoff, or idempotency where required
- [ ] Failures are inconsistent: some paths throw, others return null/sentinel
- [ ] Resource cleanup (handles, connections, files) is missing on failure paths

**Prefer:** Align with existing repository error patterns before inventing new ones.

---

## Security

- [ ] Trust boundaries: user/input data reaches queries, commands, HTML, shells, or paths unsafely
- [ ] Secrets in source, logs, tests, or committed config
- [ ] AuthZ checks missing on sensitive operations (when auth exists in the system)
- [ ] Insecure defaults (open CORS, debug flags, verbose errors in production paths)
- [ ] Unsafe deserialization, SSRF-prone URL fetch, path traversal in file APIs
- [ ] Dependency or script install hooks that execute untrusted code (when visible)

Report security findings with evidence. Prefer minimal, behavior-preserving fixes.
Do not suppress security checks to green the baseline.

---

## Testing

- [ ] Critical paths lack any automated coverage where the repo otherwise tests
- [ ] Tests assert mocks/implementation details instead of behavior
- [ ] Tests are tautological (assert constants, always-pass stubs)
- [ ] Flaky timing/order assumptions without synchronization
- [ ] Snapshot or golden files that hide regressions without human-readable intent
- [ ] Test helpers duplicate production bugs or bypass the system under test

**Do not:** Delete or weaken failing tests to obtain a green baseline.
**Do not:** Mass-rewrite a working suite for stylistic preference.

---

## Performance

- [ ] Obvious hot-path N+1 or repeated work with clear call-site evidence
- [ ] Unbounded allocations, unbounded caches, or missing pagination where required
- [ ] Synchronous blocking on clearly async/IO-bound workflows (when that is the local pattern)
- [ ] Unnecessary work on every request that could be scoped or cached per existing patterns

Only act on **obvious** problems with evidence. Do not micro-optimize without
measurement unless the defect is glaring and localized.

---

## Documentation

- [ ] README/setup instructions contradict the actual build and run flow
- [ ] Comments restate code or narrate AI "thought process"
- [ ] Docs describe removed features or obsolete architecture
- [ ] Public API docs omit breaking behavioral constraints that callers need
- [ ] Generated or duplicated docs drift from source of truth

Prefer updating or deleting stale docs over adding new documentation layers.

---

## Cross-cutting gates (before any Apply change)

- [ ] Concrete evidence of a problem exists
- [ ] Intended behavior is sufficiently understood
- [ ] Observable behavior can be preserved
- [ ] Change is within cleanup scope
- [ ] Change is small and coherent
- [ ] Result can be verified (build/tests/diff review as available)
- [ ] Creation guard satisfied if adding a file
- [ ] Dependency guard satisfied if adding a dependency

If any gate fails → **report only**, do not modify.
