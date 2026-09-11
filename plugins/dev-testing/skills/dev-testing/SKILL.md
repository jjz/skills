---
name: dev-testing
description: Plan and write unit, integration, and end-to-end tests across languages — the test pyramid (L1 unit / L2 integration / L3 E2E), docker-compose-bound integration tests, per-language runners (Jest, Vitest, Bun test, pytest, Go, Rust, Solidity), Playwright E2E, and PDPO-safe synthetic test data. Use when choosing what to test for a change, writing or reviewing tests, or debugging a flaky/slow suite.
---

# Dev Testing — Unit, Integration & End-to-End

*[中文版本 / Chinese version →](../dev-testing-zh/SKILL.md)*

Authoritative sources (these change — re-check, don't rely on memory):
[Martin Fowler — Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html) ·
[Kent C. Dodds — Testing Trophy](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) ·
[Jest](https://jestjs.io/) ·
[Vitest](https://vitest.dev/) ·
[Bun test](https://bun.sh/docs/cli/test) ·
[pytest](https://docs.pytest.org/) ·
[Playwright](https://playwright.dev/) ·
[Testcontainers](https://testcontainers.com/) ·
[Go testing](https://pkg.go.dev/testing) ·
[rust test](https://doc.rust-lang.org/rustc/tests/index.html) ·
[Foundry](https://book.getfoundry.sh/) ·
[Hardhat](https://hardhat.org/)

A test suite is a design tool, not a checkbox. The three layers — unit, integration, end-to-end — answer different questions at different costs, and most suites fail because one layer is used to do another layer's job (slow "unit" tests that hit a database, "integration" tests that mock the very thing they exist to verify, or an E2E test per HTTP route). Go through every section below, not just the one that prompted the request.

## 1. The three levels (and how to choose)

| Level | Question it answers | Dependencies | Speed | Mocking | Typical share |
|-------|---------------------|--------------|-------|---------|---------------|
| **L1 Unit** | "Is this logic correct?" | none (in-process) | ms | mock all external I/O | many |
| **L2 Integration** | "Do the pieces actually fit?" | real DB/cache/broker, docker-compose | s | no mocking of the infra under test | fewer |
| **L3 E2E** | "Does the user journey work?" | full stack + browser | s–min | no faking on the UI path | fewest |

Choose by **what the change touches**, not by what is easy:

- Changed a pure function / validation / a single module's branching → **L1**.
- Changed an HTTP handler, an ORM query, a migration, a guard/middleware, or how the app wires to real dependencies → **L2**.
- Changed a full user flow, a critical UI, or cross-service integration → **L3**.
- **"Modify what, test what"**: prefer the test at the layer where the bug could actually live. Don't write an L2 test to cover logic that an L1 test already covers, and don't write an L3 test per API route.

## 2. Unit tests (L1)

- **In-process and fast**: no DB, no network, no filesystem, no clock unless injected. If a "unit" test needs an environment to pass, it's an integration test in disguise — move it up a layer.
- **Mock at the boundary, not the internals**: stub the *ports* (HTTP client, DB adapter, LLM client, clock, random source) the unit depends on, not private helpers. Mocking internals couples the test to the implementation and makes refactors red for the wrong reason.
- **Deterministic**: inject time and randomness instead of `Date.now()` / `Math.random()` inline; a test that only passes sometimes is worse than no test.
- **Arrange–Act–Assert** with one assertion per *behavior* (not per line). Name tests after behavior, not implementation: `returns error when balance is insufficient`, not `test_process_1`.
- **F.I.R.S.T.**: Fast, Independent (no test depends on another's side effects), Repeatable, Self-validating (assert, don't `console.log`), Timely.

## 3. Integration tests (L2) — docker-compose lifecycle

Integration tests verify *assembly*: schema/migrations against a real database, real cache/broker round-trips, guards and middleware on real HTTP. The one rule that matters:

> **Never mock the infrastructure the test exists to verify.** If the test is about the DB, it talks to a real DB. If you `jest.mock` the repository to make it "fast", it's not an integration test — it's a unit test with extra steps.

- **Bind compose to the test lifecycle** — start with `up`, always tear down with `stop`/`down`, even on failure:
  - `docker-compose up -d <service>` (only the services this suite needs) → run tests → `docker-compose stop`/`down`.
  - Use a shell `trap` / a global `beforeAll` + `afterAll` / a test-runner fixture so Ctrl-C and exceptions still clean up. Leaving the same ports occupied "just for tests" will collide with the next run.
- **Idempotent** `up -d`: re-running against an unchanged config must not spawn duplicate instances or race for ports.
- **Compose file lives with the package** under test (e.g. `docker-compose.test.yml`), versioned with the code so the DB/cache version in CI matches local and prod (or is declared compatible).
- **Healthcheck before connecting**: wait for the container's healthcheck (or a retry loop on the port) rather than sleeping a fixed delay — a fixed sleep is both flaky and slow.
- **Prefer the packaged command**: run the package's own `test` script that wraps `up → test → down` instead of hand-rolling the lifecycle each time.
- **CI without Docker**: either run only L1, or replace compose with **service containers** (GitHub Actions `services:`, GitLab `services:`) — but **pick one**, compose *or* service container, never both (double instances). Connection strings and ports must match the compose mapping.
- **Testcontainers** is the cleaner alternative when the runner supports it (Java, Go, Python, Node): it wraps the same compose/container lifecycle in code, so setup/teardown can't be skipped.

## 4. End-to-end tests (L3) — Playwright

E2E is a **few high-value user journeys**, not a re-test of every endpoint in a browser.

- **Drive a real browser** (Playwright) against the assembled stack. Favor user-facing locators (`getByRole`, `getByLabel`, `getByText`) over brittle CSS/XPath selectors, and prefer data-`testid` attributes over DOM positions.
- **Isolate state**: each scenario sets up its own data (or uses a known fixture) and cleans up after itself, so tests can run in any order and in parallel.
- **Distinguish flaky from broken**: retries are a safety net, not a fix — a test that needs retries to pass is hiding a real race; fix the root cause (wait for a condition, not a timeout) instead of raising the retry count.
- **Test through the public interface**: don't bypass auth, don't seed via private DB writes if the product path is a UI form — E2E should exercise what a user actually does, otherwise it proves nothing about the journey.
- **Keep it small and run it late**: E2E is the slowest, most expensive layer — gate it to the critical journeys, run it in CI after L1/L2 pass, and never let a green build substitute for actually checking status codes/behavior.

## 5. Per-language runners

The pyramid above is language-agnostic; the runner differs. Follow the convention already in the package rather than imposing one.

### TypeScript / JavaScript

- **Jest** — the default for long-lived Node/NestJS backends; `jest.config` + `ts-jest` for TS, `describe/beforeAll/it`, snapshot tests where appropriate.
- **Vitest** — the default when the package is Vite-based (Next/React frontends, Vite libs); Jest-compatible API, faster, ES-module-native.
- **Bun test** — Bun's built-in runner (`bun test`); Jest-compatible API but *not* Jest — don't assume every Jest plugin/glob works.
- **Pick one per package**: a monorepo may legitimately use Jest in one package and Vitest in another — match the package, don't migrate it casually, and don't cross-apply one package's runner config to another.
- Run through **`bun run <script>`** (or the package's declared package manager); the script is the source of truth for which runner and which config.

### Python

- **pytest** (run via the project's environment manager, e.g. `uv run pytest`): `fixture`s for shared setup/teardown, `@pytest.mark.parametrize` for table-driven cases, `tmp_path`/`monkeypatch` for isolation, `pytest-asyncio` for async.
- Prefer fixtures over `setUp`/`tearDown`; a fixture with `yield` gives you clean teardown. Keep fixtures scoped as narrow as possible (`function` over `session`).

### Go (reserved)

- Standard `testing` package: table-driven subtests (`t.Run`), `go test ./...`, `-race`, `-cover`. `testify` for assertions/suites/mocks. Integration tests use **Testcontainers-go** or compose + a `TestMain` that sets up/tears down once.

### Rust (reserved)

- `cargo test`: `#[test]`, `#[tokio::test]`/`#[rstest]` for async/parametrized. `assert!`/`assert_eq!`/`assert_matches!`. Integration tests live in `tests/` (public-crate API) vs `#[cfg(test)]` modules for unit tests. Containers via `testcontainers` crate.

### Solidity / EVM (reserved)

- **Foundry** (`forge test`) is the default for contract testing: Solidity test contracts, `vm.expectRevert`, fuzzing (`testFuzz_`), invariant testing, `forge snapshot` for gas. **Hardhat** for a JS/TS-native workflow (mocha + ethers + `hardhat test`).
- Always cover: revert paths (`vm.expectRevert`), access control (onlyOwner/roles), overflow/underflow (Solidity ≥0.8 checks by default), and **invariant tests** for financial contracts — an invariant violated only under a fuzz sequence is exactly where fund-loss bugs hide.

## 6. Test data & environment

- **Never use real personal data in fixtures.** Production PII (IDs, phone, address, KYC docs, wallet addresses tied to individuals) is regulated data — a test fixture with a real customer's data is a data-breach-in-waiting. Use synthetic/anonymized generators (faker/polygon), and scrub any fixture you copy from prod.
- **Factories over hand-written seeds**: build the minimum object a test needs with a factory/builder, overriding only the fields the scenario cares about. This keeps tests readable and stops unrelated assertions from coupling to a shared fixture's exact values.
- **Clean up side effects**: anything a test writes (rows, files, caches) should be removed in teardown or isolated per-run (unique IDs, a per-run DB schema, `tmp_path`).
- **Use the same config source** (env vars / `.env.test`) for connection strings and ports as the compose mapping — hardcoded ports drift and break silently when two suites run in parallel.

## 7. Best-practice summary

- **Correct > fast, fast > thorough**: an L1 test that runs in 10ms gets run on every save; an L2 suite that takes minutes gets run only in CI — bias toward the fast layer, and only reach for slower layers when the risk demands it.
- **Tests are independent and order-free**: no shared mutable state between tests, no test that requires a previous test to have run.
- **Coverage is a signal, not a target**: chase the *why* (a critical branch, an error path, a regression you just fixed — always add a regression test for a bug), not the *number*. 80% is a floor, not the goal; an uncovered critical path matters more than a covered getter.
- **Assertions on real behavior**: assert status/`errorCode`/observable output, not `console.log` or "it didn't throw". For business failures, keep a single source-of-truth contract (e.g. a `POINTS_ERRORS` map) that both `throw` and the test reference — never hardcode the literal `402` / `'INSUFFICIENT_BALANCE'` in the test.
- **Timeouts and error paths are first-class**: async/streaming code needs a timeout and the failure path asserted, not just the happy path. Streaming APIs should assert chunk boundaries and the end-of-stream marker.
- **CI gate**: `lint → build → test` before merge; keep the feedback fast by running L1 first, then L2, then the small E2E set. A failing test fails the build with a readable message, not a stack trace dump.

## 8. Checklist for a change

1. Classify the change: L1, L2, or L3 — pick the *right* layer, not the easy one.
2. L1: in-process, external I/O mocked, time/random injected, behavior-named.
3. L2: real dependencies via compose (or service container/Testcontainers), `up → test → down` with cleanup on failure, healthcheck before connect, no infra mocks.
4. L3: only critical journeys, browser-driven, state-isolated, user-facing locators, fix flakiness not retries.
5. Match the package's runner (Jest/Vitest/Bun test/pytest/Go/Rust/Foundry) — don't cross-apply config.
6. Test data: synthetic, factory-built, cleaned up, no real PII.
7. Add a regression test for any bug fixed; run `lint → build → test` (or `lint → test` when a dev server is already running) and gate on it in CI.
