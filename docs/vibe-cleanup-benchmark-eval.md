# Vibe Cleanup Benchmark — Evaluation

Human evaluation sheet for [vibe-cleanup-benchmark](https://github.com/HasiniA99x/vibe-cleanup-benchmark).

**Keep this file outside the benchmark repository when running the cleaner.**
It is the answer key. Do not give it to the cleanup agent before the run.

Skill under test: [`code-cleanup`](https://github.com/HasiniA99x/clean-code-skills/tree/main/code-cleanup) (`SKILL.md`).

## How to run a blind evaluation

1. Clone a fresh copy of [vibe-cleanup-benchmark](https://github.com/HasiniA99x/vibe-cleanup-benchmark) (seeded / dirty state).
2. Copy [`code-cleanup/`](https://github.com/HasiniA99x/clean-code-skills/tree/main/code-cleanup) into that repo’s skills path (for Cursor: `.cursor/skills/code-cleanup/`).
3. Open the **benchmark** repo (not this skills repo) and start an agent chat.
4. Prompt: `Run code-cleanup on this repository.`
5. Approve or reject each proposed cleanup as a human evaluator.
6. After the session ends, score against the gold set below **without** having shared this sheet with the agent.

## Gold set

| ID | Seeded case | Expected decision | Why |
|----|-------------|-------------------|-----|
| B01 | `lodash` declared but unused | FIX | No app/tooling usage; remove dependency |
| B02 | `src/common/string-normalizer.ts` has no references | FIX | Dead helper file |
| B03 | Title length validation repeated in DTO/controller/service | FIX | HTTP-boundary DTO validation is the intended owner |
| B04 | `AiClient` + `AiClientFactory` for one provider with no multi-provider requirement | FIX | Used but unjustified extensibility |
| B05 | `ClientWrapper` only forwards `summarize()` | FIX | Adds indirection without responsibility |
| B06 | 500 ms delay inside AI summarization | FIX | Product notes explicitly say no AI delay is required |
| B07 | `getById()` calls repository `findById()` twice | FIX | Duplicate work with no capability gain |
| B08 | Profile and preferences remote stand-ins are awaited sequentially | FIX | Independent I/O can safely run together; preserve each 40 ms stand-in |
| B09 | AI token is written to logs | FIX | Secret exposure |
| B10 | Body `userId` can override trusted `x-user-id` identity | FIX | Authorization/identity boundary violation |
| B11 | `Clock` interface/token has one implementation | LEAVE | Intentional deterministic-test boundary |
| B12 | Email retry loop adds complexity | LEAVE | Explicit operational requirement: up to 3 attempts |
| B13 | DTO/class-validator input validation | LEAVE | Required HTTP trust-boundary validation |
| B14 | `ArticleFormatter` is relatively long | LEAVE | One coherent formatting responsibility; no evidence that splitting helps |
| B15 | SMS placeholder/config path | ASK | Pilot requested it, but product has not decided whether it will ship |

## Suggested scoring

| Metric | Definition |
|--------|------------|
| Precision | correct FIX proposals / all FIX proposals |
| Recall | correct FIX proposals / 10 expected FIX cases |
| Restraint | correct LEAVE decisions / 4 expected LEAVE cases |
| Escalation accuracy | correct ASK decisions / 1 expected ASK case |
| Keep rate | accepted applied fixes / applied fixes |
| Revert rate | reverted applied fixes / applied fixes |
| Scope leak rate | applied fixes that changed outside the approved surface / applied fixes |

## Reference run score (2026-10-01)

Agent session using [`code-cleanup`](https://github.com/HasiniA99x/clean-code-skills/tree/main/code-cleanup) CLEAN mode against the seeded benchmark.

| Metric | Score | Notes |
|--------|-------|-------|
| Precision | 10/10 | All FIX proposals matched gold FIX cases (B01–B10) |
| Recall | 10/10 | Every expected FIX case was proposed and applied |
| Restraint | 4/4 | Clock, email retries, DTO validation, and ArticleFormatter left alone |
| Escalation accuracy | 0/1 | SMS (B15) was left alone at session end; never surfaced as ASK |
| Keep rate | 10/10 | F1–F10 all accepted; none reverted |
| Revert rate | 0/10 | — |
| Scope leak rate | 0/10 | One npm formatting drift on `package.json` was narrowed before accept |

### Mapping (reference run)

| Gold ID | Chat finding | Result |
|---------|--------------|--------|
| B01 | F3 — unused lodash | Fixed |
| B02 | F2 — unused string-normalizer | Fixed |
| B03 | F6 + F7 — duplicate title checks (service, then controller); DTO kept | Fixed |
| B04 | F1 — AI provider switch layers | Fixed |
| B05 | F1 — ClientWrapper removed with factory/interface | Fixed |
| B06 | F5 — artificial AI delay | Fixed |
| B07 | F4 — duplicate `findById` | Fixed |
| B08 | F10 — `Promise.all` for independent calls | Fixed |
| B09 | F9 — token logging removed | Fixed |
| B10 | F8 — always use authenticated user id | Fixed |
| B11–B14 | Left alone | Correct LEAVE |
| B15 | Left alone without asking | Missed ASK |
