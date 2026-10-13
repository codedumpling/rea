---
tags: [rea, development, testing]
source:
  - docs/testing.md
  - docs/client-verification.md
  - docs/reconstruction-readiness.md
  - package.json
  - vitest.config.ts
---
# Testing

REA's testing philosophy: **prefer real public CLI/MCP workflows, then production-boundary integration, then goldens captured from real producers.** Keep module tests only for semantics that stronger workflows cannot reliably reproduce. A simulated or injected provider never establishes real-provider behavior, so every engine claim needs its own `npm run verify:*` lane.

## Developer commands

| Command | Scope |
| --- | --- |
| `npm run test:local` | Dirty source tests via the import graph, no build. Explicit paths always run |
| `npm run test:focused -- PATH...` | Exact test files. Compiled boundaries build first; unmatched paths fail |
| `npm run test:changed` | Tests affected since the merge base with `origin/main` (`--base`, `--dry-run`) |
| `npm run test:fast` | All domain/service/adapter/composition/conformance/evaluation tests, no build |
| `npm run test:boundary` | Boundary, CLI boundary, MCP boundary, process boundary, process-global |
| `npm run test:mcp` | MCP boundary project |
| `npm run test:acceptance` | Complete compiled CLI and MCP journeys |
| `npm run test:python` | LLDB producer contracts with a fake LLDB API |
| `npm run test:watch` / `test:watch:all` | Watch modes |
| `npm run check:changed` / `check:pr` | Branch feedback / opt-in full local gate |
| `npm run docs:check` | Generated-doc validation and README onboarding consistency |

```bash
npm run test:local -- src/config.test.ts
npm run test:focused -- tests/acceptance/applications/runtime.test.ts
npm run test:changed -- --base origin/main
```

> [!warning] Empty selection ≠ verified
> Changed-test selection follows the import graph. It can miss runtime registration, bridges, generated data and shell entry points. If nothing is selected, nothing was tested. Choose boundary tests and real lanes explicitly.

`npm run test:prepare` generates the MCP test catalog (`.cache/mcp-tool-catalog.json`), the product catalog and managed evidence for the full suite. `npm run mcp-catalog:generate` refreshes the catalog on its own.

## Choose verification for the change

| Change | First check | Then |
| --- | --- | --- |
| Source semantics | `test:local -- PATH` | Boundary cases if it crosses a port |
| CLI/MCP workflow | `test:focused -- PATH` | Actual public calls |
| Docs / generated metadata | `docs:check` | `docs:build` for rendered pages |
| Engine adapter | Its real-provider lane | Host, version, target identity, cleanup |
| Client config / schema conversion | Client lanes (`docs/client-verification.md`) | Distinguish loopback models from live services |
| Agent investigation quality | `npm run verify:agent` | Review the transcript |

## Behavioral depths

| Depth | Location | Proves |
| --- | --- | --- |
| Module | `src/**/*.test.ts` | One domain/service/adapter through its narrow surface |
| Composition | `tests/composition/**` | Provider-neutral session and registry wiring |
| Boundary | `tests/boundary/**` | Exactly one production boundary (fs, process, network, browser, CLI, provider) |
| MCP boundary | `tests/boundary/mcp/**` | MCP transport and tool contracts |
| Acceptance | `tests/acceptance/**` | A compiled CLI or MCP journey |
| Conformance | `tests/conformance/**` | Shared provider contracts by declared capability |
| Evaluation | `tests/evaluation/**` | Evaluator parsing, scoring, reports |

`tests/process-global/**` holds cases that depend on process-global state (run in isolated forks). `tests/support/**` holds reusable fixtures, and `tests/fixtures/**` holds immutable artifacts and fakes (`fakeGhidra.mjs`, a fake CDP browser, and others).

**Pruning rules:** delete tests that mirror implementation details or duplicate a stronger claim, but name the replacement scenario first. Always keep coverage for malformed input, cancellation, cleanup, permissions, capacity and target identity. For `Result` values, throw the typed error before asserting; don't assert `.ok` first.

## Coverage, sharding, caching

- CI owns coverage. Thresholds live in `vitest.config.ts` and are never updated automatically.
- CI runs **six Vitest shards** with no retries. A merge job builds aggregate coverage, JUnit and timing reports. A failed or missing shard fails the required gate.
- Vitest projects use up to two workers. `process-boundary` runs later with serial files.
- Turbo caches deterministic builds and static checks, but `test:run` always executes, and **real-provider execution is never cached**.

## Real-toolchain lanes

Each lane must require only the tools needed for its claim, check prerequisites up front, and name any missing tool. A skipped lane never implies coverage.

| Area | Lanes |
| --- | --- |
| Hopper | `verify:hopper`, `verify:hopper:linux`, `verify:hopper:deadlines`, `verify:hopper:fat`, `verify:hopper:fat64` |
| Ghidra | `verify:ghidra`, `:seeds`, `:swift`, `:switch`, `:aarch64-jump-table`, `:cross-format`, `:riscv`, `:mips`, `:mips-shared`, `:mips-relocatable`, `:dos`, `:com`, `:raw`, `:nativeaot`, `:windows`, `:windows:package` |
| Windows native | `verify:windows-native`, `verify:package:windows` |
| IDA / JEB / Frida | `verify:ida`, `verify:jeb`, `verify:frida` |
| Android & mobile | `verify:android`, `verify:apktool`, `verify:adb`, `verify:flutter`, `verify:harmony` |
| Browser / JS | `verify:browser`, `verify:browser:*`, `verify:electron`, `verify:inspector`, `verify:javascript-runtime` |
| Other | `verify:managed`, `verify:firmware`, `verify:readiness`, `verify:package`, `verify:agent`, client lanes (`verify:claude-client`, `verify:gemini-client`, `verify:opencode-client`, …) |

The Ghidra lanes prove behavior on **Ghidra 12.1.4 + JDK 21**. For example, `verify:ghidra` needs a host C compiler and checks CLI/MCP parity, atomic annotation rollback, unchanged executable bytes, snapshot lifecycle, cancellation and cleanup. Variant prerequisites are covered in [[Ghidra Variants]].

## Reconstruction readiness

`npm run verify:readiness` runs a nine-stage conformance verifier: `discover-classify`, `diagnose-environment`, `acquire-authority`, `static-analysis`, `reactive-scenarios`, `compare-authority-candidate`, `preserve-contradictions`, `verify-reconstruction-closure`, `export-replay`. It runs over source-owned native, JavaScript CLI and Electron fixtures. It **fails closed**: any `fail`, `unknown`, `unsupported`, `truncated` or `skipped` result never aggregates to `pass`. Set `REA_READINESS_REPORT_PATH` to keep the full report. This is a repository verifier, not a public tool.

## Client and agent verification

The optional `verify:*-client` lanes configure disposable profiles through public `rea setup` and make real stdio MCP calls, against a deterministic **loopback model**. That means they do not prove live model-service compatibility. Executables are selected with variables such as `REA_VERIFY_QWEN_COMMAND`, `REA_VERIFY_GEMINI_COMMAND` and `REA_VERIFY_OPENCODE_COMMAND`. `verify:agent` evaluates scenarios through a real local Codex CLI (`REA_CODEX_CLI`, `REA_AGENT_EVAL_MODEL`, `REA_AGENT_EVAL_SCENARIOS`).

See also [[CI Workflows]], [[Contributing]], [[Readiness Checks]].

## Source
- https://github.com/morluto/rea/blob/main/docs/testing.md
- https://github.com/morluto/rea/blob/main/docs/client-verification.md
- https://github.com/morluto/rea/blob/main/docs/reconstruction-readiness.md
