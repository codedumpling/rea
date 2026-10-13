---
tags: [rea, development, architecture]
source:
  - AGENTS.md
  - docs/architecture.mermaid
  - CONTRIBUTING.md
  - scripts/lib/module-boundaries.mjs
  - package.json
  - src/
  - tests/
---
# Codebase Map

REA is one TypeScript package (`rea-agents`, ESM, Node `^22.19.0 || ^24.11.0 || >=26.0.0`; the dev toolchain is pinned by `.nvmrc` = 24.18.0 and `npm@11.16.0`). Both binaries, `rea` and `rea-agents`, point to `scripts/rea.mjs`, which dispatches either to the MCP server (`rea mcp`) or to a one-shot CLI command. For the conceptual picture, see [[Architecture]].

## Layering rule

> [!info] Dependencies flow inward
> **domain → contracts → provider adapters → application → CLI/MCP adapters.** Domain semantics and contracts sit at the center. Engine protocols and provider-specific code must stay out of the domain and application layers. (AGENTS.md)

Read the arrow as "inner to outer". Outer layers may import inner layers, never the reverse. `npm run verify:module-boundaries`, which runs as part of `npm run lint`, enforces resolved imports, including type imports and re-exports. It reports these boundary violations:

| Boundary | Rule |
| --- | --- |
| `pure-layer` | `src/domain/` and `src/contracts/` may import only each other, never other `src/` layers or `src/cli.*`/`src/main.*` |
| `application-composition` | `src/application/` must not import `src/composition/` |
| `provider-construction` | `application/` and `server/` must not construct providers directly (`ghidra`, `hopper`, `ida`, `sqlite`, `evm`, `go`, `wasm`, `captures`, `signals`, `android`, `firmware`, `inspector`, …). Construction belongs in `src/composition/` |
| `artifact-acquisition` | artifact readers must not import application, composition, server, cli or main |
| `process-capture` | `src/process/capture/` must not import outer layers |

Production factories in `src/composition/` build fresh providers **without starting engines or acquiring targets**. When an optional adapter fails, its successful peers must still be preserved.

## `src/` subdirectories

| Directory | Layer | Role |
| --- | --- | --- |
| `domain/` | Domain | Pure, side-effect-free modules: Evidence, comparisons, application graphs, the `Result` type and the tagged error algebra. Includes per-family subfolders (`android`, `apple`, `evm`, `javascript`, `managed`, `native`, `wasm`, …) |
| `contracts/` | Contracts | Caller-visible Zod schemas for every tool, `providerSelection.ts`, `errorSchemas.ts` (the generated error catalog), tool effects and examples |
| `config/` | Config | Zod-validated environment parsing (`environment.ts`), provider settings, startup timeouts, MCP response budget |
| `process/` | Provider foundation | Owned process groups, private roots, deadlines, bounded reads, cleanup. All supervision should reuse this |
| `windows/` | Provider foundation | Windows host adapters: native package loader, owned Job Objects, private runtimes |
| `filesystem/` | Provider foundation | Owned file/directory handles, stable hashing, staged output |
| `hopper/` | Provider | Hopper launch, Unix-socket protocol, target leases, private Xvfb diagnostics. See [[Hopper]] |
| `ghidra/` | Provider | Ghidra discovery, analysis profiles, Unix/Windows transport, snapshots, MIPS/RISC-V/PSP/raw profiles. See [[Ghidra]] |
| `ida/` | Provider | Adapts the upstream IDA MCP contract: attached GUI or owned headless database. See [[IDA]] |
| `jeb/` | Provider (aux) | JEB MCP endpoint and connection. See [[JEB]] |
| `frida/` | Provider (aux) | `FridaInstrumentationManager`, optional bindings. See [[Frida]] |
| `native/` | Provider | Apple Mach-O/ObjC/Swift metadata, LLDB tracer, macOS native UI, `pe/`, `parsers/`, `pwntools/`, `pwndbg/` |
| `dotnet/` | Provider | Managed PE/CLI static metadata (members, IL). See [[DotNET Assemblies]] |
| `android/`, `apktool/`, `adb/` | Provider | JADX headless adapter, Apktool resource decoding, adb device acquisition. See [[Android]] |
| `flutter/` | Provider | Dart ELF and Flutter build identification |
| `go/` | Provider | Go buildinfo/module metadata from ELF/Mach-O/PE |
| `wasm/` | Provider | WABT-backed WASM inspection |
| `evm/` | Provider | EVMole worker and EVM carrier parsing |
| `sqlite/` | Provider | SQLite database inspection in a worker |
| `captures/` | Provider | TShark packet-capture provider |
| `signals/` | Provider | sigrok recording provider |
| `firmware/` | Provider | Binwalk/Unblob adapters |
| `artifacts/` | Provider | Artifact inventory, extraction, Apple bundles, JavaScript artifact reading (ASAR/directories) |
| `javascript/` | Provider | `analysis/`, `recovery/` (Wakaru), `sourceMaps/` (pinned map codec) |
| `browser/` | Provider | CDP/Playwright capture: `assets/`, `execution/`, `history/`, `modules/`, `network/` |
| `inspector/` | Provider | Passive V8 Inspector discovery and capture |
| `reference/` | Provider | Bounded filesystem reader for reference sources |
| `application/` | Application | Shared CLI/MCP workflows. `binary/` holds `AnalysisProviderRegistry`, `SessionProviderRouter` and `BinarySession`; `investigation/` holds InvestigationRecords and the EvidenceLedger; plus one folder per family |
| `composition/` | Composition | Typed factories wiring providers into sessions (`binary.ts`, `jeb.ts`, `firmware.ts`, …) |
| `server/` | Adapter (MCP) | MCP request translation: `createServer`, `register*Tools`, availability policy |
| `cli/` + `src/cli*.ts` | Adapter (CLI) | One-shot command implementations, output, JSON input, logging, command names |
| `main/` + `src/main.ts` | Adapter (MCP entry) | stdio server startup, transport, shutdown, process lifetime |
| `evaluation/` | Tooling | Agent release evaluation (Codex transcript scoring) |

Top-level files in `src/` include `cli.ts`, `main.ts`, `doctorRuntime.ts`, `mcpDoctor.ts`, `logger.ts`, `identity.ts`, `serverIdentity.ts` and the generated `generatedPackageMetadata.ts`.

## Top-level directories

| Path | Role |
| --- | --- |
| `src/` | Runtime source (above). Colocated module tests are `*.test.ts`; shared builders are `*.fixture.ts` |
| `tests/` | Vitest suites by depth: `composition/`, `boundary/` (incl. `boundary/mcp/`), `acceptance/`, `conformance/` (C/PSP/IDA fixtures, readiness apps), `evaluation/`, `process-global/`, `support/`, `fixtures/`. See [[Testing]] |
| `bridge/` | Code that runs inside or beside engines: `hopper_bridge.py`, `ghidra/*.java` (bridge, raw/COM preparers, seeds, MIPS soft-float, no-return fix), `nativeaot/`, `android/ReaJadxBridge.java`, `mitmproxy/capture.py`, `native/` (Swift UI, LLDB tracer), `process/`, `pwntools/`, `pwndbg/` |
| `scripts/` | `rea.mjs` dispatcher, build/generation scripts, `verify-*.mjs` real-toolchain lanes, `verify/`, `ci/` planners, `lib/`, `fixtures/`, `release-notes.mjs`, `website.py` |
| `native/windows/` | Node-API source for Windows controls (NTFS admission, DACLs, Job Objects) |
| `docs/` | User guides (also the VitePress site), `architecture.mermaid`, `releases/` contribution inventories, error schema |
| `website/` | Static rea.tools site. See [[Website]] |
| `experiments/` | `analysis-efficiency/`: measured token/time studies (#2016) and their scripts |
| `third_party/` | Pinned upstream submodules (binwalk, unblob, wabt, wakaru, pwntools, pwndbg, evmole, jadx-headless-mcp, ghidra-nativeaot, …) |
| `licenses/` | Third-party license texts and the upstream sources inventory |
| `.agents/skills/` | Authored skills. See [[Repo Agent Skills]] |
| `.github/workflows/` | CI and release. See [[CI Workflows]] |
| `.husky/` | Pre-commit (format and lint staged files) and pre-push (`check:fast`) hooks |

Root config files: `turbo.json` (task graph and cache), `vitest.config.ts`, `tsconfig*.json`, `knip.json`, `.oxlintrc.json`, `.oxfmtrc.json`, `.jscpd.json`, `release-please-config.json`, `.release-please-manifest.json`, `server.json` (MCP Registry metadata), `install.sh` (curl wrapper).

> [!warning] Ignored outputs
> `docs/public/product-catalog.json`, `docs/verification/managed-conformance-*.json` and `skills/` are generated and must not be committed. Edit the source contracts and `.agents/skills/reverse-engineer-anything/` instead. Never commit `dist/`, `node_modules/`, binaries, provider project files or credentials.

See also [[Contributing]], [[Tool Design Principles]], [[Providers Overview]].

## Source
- https://github.com/morluto/rea/blob/main/docs/architecture.mermaid
- https://github.com/morluto/rea/blob/main/AGENTS.md
- https://github.com/morluto/rea/blob/main/scripts/lib/module-boundaries.mjs
