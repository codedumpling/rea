---
tags: [rea, reference, index]
source:
  - docs/
  - docs/index.md
---
# Doc Index

Every file in the repository's `docs/` directory, with a one-line description and the vault note that covers it. Links go to GitHub `main`. Root-level documents are listed at the end.

## Getting started and usage

| File | Description | Vault note |
| --- | --- | --- |
| [index.md](https://github.com/morluto/rea/blob/main/docs/index.md) | Docs site home page: one-command start and a task-to-guide table | [[Home]], [[What is REA]] |
| [installation.md](https://github.com/morluto/rea/blob/main/docs/installation.md) | Setup transaction, readiness checks, update/uninstall, Hopper and Ghidra configuration, MCP Registry, package runners | [[Installation]], [[Agent Setup]], [[Readiness Checks]], [[Updating and Uninstalling]] |
| [client-configuration.md](https://github.com/morluto/rea/blob/main/docs/client-configuration.md) | Per-client config paths, profiles, trust policies and schema-size workarounds | [[Agent Setup]], [[Troubleshooting]] |
| [cli.md](https://github.com/morluto/rea/blob/main/docs/cli.md) | CLI usage, provider selection, snapshots, Evidence import/export, output formats and exit status | [[CLI Reference]], [[Sessions and Snapshots]], [[Evidence Export and Import]] |
| [mcp-contracts.md](https://github.com/morluto/rea/blob/main/docs/mcp-contracts.md) | MCP result envelopes, retained Evidence, discovery, cancellation, Ghidra deadlines, schema advertisement, generated catalog | [[MCP Server]], [[MCP Tool Catalog]], [[Evidence Model]] |
| [mcp-prompts.md](https://github.com/morluto/rea/blob/main/docs/mcp-prompts.md) | The six guided MCP prompts and session-aware completion | [[Agent Prompts]] |
| [migration-6.2.md](https://github.com/morluto/rea/blob/main/docs/migration-6.2.md) | Breaking-change guide for upgrading custom consumers from 6.1 to 6.2 | [[Updating and Uninstalling]] |
| [roadmap.md](https://github.com/morluto/rea/blob/main/docs/roadmap.md) | Priorities, trackers and admission criteria for new capabilities | [[Roadmap]] |
| [reconstruction-obligation-ledgers.md](https://github.com/morluto/rea/blob/main/docs/reconstruction-obligation-ledgers.md) | `build_reconstruction_obligation_ledger`: turning Evidence into reconstruction obligations | [[Evidence Export and Import]], [[Recipes]] |
| [user-visible-errors.csv](https://github.com/morluto/rea/blob/main/docs/user-visible-errors.csv) | Catalog of user-visible error codes/messages | [[Errors and Exit Codes]] |
| [error-contract.schema.json](https://github.com/morluto/rea/blob/main/docs/error-contract.schema.json) | JSON Schema for the error diagnostic contract | [[Errors and Exit Codes]] |

## Native binaries and providers

| File | Description | Vault note |
| --- | --- | --- |
| [native-investigation.md](https://github.com/morluto/rea/blob/main/docs/native-investigation.md) | Discovering functions, instructions, pseudocode, references and types with a native provider | [[Native Binaries]] |
| [ghidra-provider.md](https://github.com/morluto/rea/blob/main/docs/ghidra-provider.md) | Ghidra provider semantics: ephemeral database, operations, guarantees | [[Ghidra]] |
| [windows-ghidra-p0.md](https://github.com/morluto/rea/blob/main/docs/windows-ghidra-p0.md) | Experimental read-only Windows x64 Ghidra boundary | [[Ghidra Variants]] |
| [ghidra-dos.md](https://github.com/morluto/rea/blob/main/docs/ghidra-dos.md) | DOS MZ and COM analysis through Ghidra | [[Ghidra Variants]] |
| [raw-binary.md](https://github.com/morluto/rea/blob/main/docs/raw-binary.md) | Headerless raw-binary imports with caller-supplied CPU family | [[Ghidra Variants]] |
| [ghidra-mips.md](https://github.com/morluto/rea/blob/main/docs/ghidra-mips.md) | Bounded MIPS ELF analysis through Ghidra | [[Ghidra Variants]] |
| [ghidra-psp.md](https://github.com/morluto/rea/blob/main/docs/ghidra-psp.md) | PSP static ELF specialization of MIPS support | [[Ghidra Variants]] |
| [ghidra-riscv.md](https://github.com/morluto/rea/blob/main/docs/ghidra-riscv.md) | RISC-V ELF analysis through Ghidra | [[Ghidra Variants]] |
| [ghidra-nativeaot.md](https://github.com/morluto/rea/blob/main/docs/ghidra-nativeaot.md) | Optional NativeAOT metadata recovery (x64 PE, .NET 8 RTR) | [[Ghidra Variants]], [[DotNET Assemblies]] |
| [ida-provider.md](https://github.com/morluto/rea/blob/main/docs/ida-provider.md) | Adapting an IDA MCP server (ida-pro-mcp or native bridge) into REA | [[IDA]] |
| [jeb-analysis.md](https://github.com/morluto/rea/blob/main/docs/jeb-analysis.md) | JEB project, unit and decompilation inspection over JEB's MCP interface | [[JEB]] |
| [frida-instrumentation.md](https://github.com/morluto/rea/blob/main/docs/frida-instrumentation.md) | Frida device/process discovery and caller-authored instrumentation | [[Frida]] |
| [go-binaries.md](https://github.com/morluto/rea/blob/main/docs/go-binaries.md) | Embedded Go build metadata from ELF/PE/Mach-O | [[Go Binaries]] |
| [binary-diagnostics.md](https://github.com/morluto/rea/blob/main/docs/binary-diagnostics.md) | Offline ELF layout via caller-supplied pwntools | [[ELF Layout and Recorded Crashes]] |
| [recorded-crashes.md](https://github.com/morluto/rea/blob/main/docs/recorded-crashes.md) | Recorded Linux crash (core) evidence, optional GDB/pwndbg context | [[ELF Layout and Recorded Crashes]] |
| [wasm-artifacts.md](https://github.com/morluto/rea/blob/main/docs/wasm-artifacts.md) | Offline WebAssembly inspection with WABT | [[WASM Artifacts]] |
| [evm-bytecode.md](https://github.com/morluto/rea/blob/main/docs/evm-bytecode.md) | Offline EVM interface inference with EVMole | [[EVM Bytecode]] |
| [sqlite-databases.md](https://github.com/morluto/rea/blob/main/docs/sqlite-databases.md) | SQLite database/WAL snapshot inspection | [[SQLite Databases]] |
| [firmware-analysis.md](https://github.com/morluto/rea/blob/main/docs/firmware-analysis.md) | Firmware region inspection and extraction with Binwalk/Unblob | [[Firmware]] |

## JavaScript, Electron and web

| File | Description | Vault note |
| --- | --- | --- |
| [javascript-artifact-reconstruction.md](https://github.com/morluto/rea/blob/main/docs/javascript-artifact-reconstruction.md) | Static inspection of an Electron/JS directory or ASAR | [[JavaScript and Electron]] |
| [javascript-application-workflows.md](https://github.com/morluto/rea/blob/main/docs/javascript-application-workflows.md) | Feature traces, views and version comparisons over saved Evidence | [[JavaScript and Electron]] |
| [javascript-application-graph.md](https://github.com/morluto/rea/blob/main/docs/javascript-application-graph.md) | The provider-neutral JavaScript Application Graph model | [[JavaScript and Electron]] |
| [javascript-recovery.md](https://github.com/morluto/rea/blob/main/docs/javascript-recovery.md) | Readable source recovery with caller-supplied Wakaru | [[JavaScript and Electron]] |
| [javascript-runtime-observation.md](https://github.com/morluto/rea/blob/main/docs/javascript-runtime-observation.md) | Passive Node/Electron V8 Inspector observation | [[JavaScript Runtime Observation]] |
| [javascript-runtime-reconciliation.md](https://github.com/morluto/rea/blob/main/docs/javascript-runtime-reconciliation.md) | Combining static graphs with runtime observations | [[JavaScript Runtime Observation]] |
| [electron-observation.md](https://github.com/morluto/rea/blob/main/docs/electron-observation.md) | Attaching to Electron `file://` renderer pages via CDP | [[JavaScript Runtime Observation]] |
| [browser-observation.md](https://github.com/morluto/rea/blob/main/docs/browser-observation.md) | Website observation through CDP | [[Websites and Browser Observation]] |
| [browser-scenario-contract.md](https://github.com/morluto/rea/blob/main/docs/browser-scenario-contract.md) | `capture_browser_scenario` input contract | [[Websites and Browser Observation]] |
| [web-runtime.md](https://github.com/morluto/rea/blob/main/docs/web-runtime.md) | Runtime source attribution with CDP and V8 coverage | [[Websites and Browser Observation]] |
| [web-source-location.md](https://github.com/morluto/rea/blob/main/docs/web-source-location.md) | Mapping a position in a captured script to its source | [[Websites and Browser Observation]] |
| [website-module-trace.md](https://github.com/morluto/rea/blob/main/docs/website-module-trace.md) | Tracing a captured script's ES module imports | [[Websites and Browser Observation]] |
| [website-script-export.md](https://github.com/morluto/rea/blob/main/docs/website-script-export.md) | Exporting captured scripts into static JavaScript analysis | [[Websites and Browser Observation]] |
| [web-network-captures.md](https://github.com/morluto/rea/blob/main/docs/web-network-captures.md) | HAR, Fiddler SAZ and mitmproxy capture inspection | [[Network and Packet Captures]] |
| [packet-captures.md](https://github.com/morluto/rea/blob/main/docs/packet-captures.md) | Recorded packet capture fields via TShark (Linux) | [[Network and Packet Captures]] |

## Mobile, managed, packages and other targets

| File | Description | Vault note |
| --- | --- | --- |
| [managed-code-analysis.md](https://github.com/morluto/rea/blob/main/docs/managed-code-analysis.md) | Static .NET PE/CLI inspection | [[DotNET Assemblies]] |
| [android-analysis.md](https://github.com/morluto/rea/blob/main/docs/android-analysis.md) | Static APK analysis via headless JADX | [[Android]] |
| [apktool-resource-analysis.md](https://github.com/morluto/rea/blob/main/docs/apktool-resource-analysis.md) | Decoding resources and manifest with Apktool | [[Android]] |
| [adb-device-analysis.md](https://github.com/morluto/rea/blob/main/docs/adb-device-analysis.md) | Device inspection and acquisition via adb | [[Android]] |
| [flutter-analysis.md](https://github.com/morluto/rea/blob/main/docs/flutter-analysis.md) | Flutter build identification from APKs | [[Flutter Apps]] |
| [apple-application-analysis.md](https://github.com/morluto/rea/blob/main/docs/apple-application-analysis.md) | Apple application inventory projection | [[Apple and Harmony Apps]] |
| [harmony-analysis.md](https://github.com/morluto/rea/blob/main/docs/harmony-analysis.md) | Static HarmonyOS package inventory | [[Apple and Harmony Apps]] |
| [process-capture.md](https://github.com/morluto/rea/blob/main/docs/process-capture.md) | Running a command and recording terminal, exit and filesystem evidence | [[Process Capture]] |
| [signal-recordings.md](https://github.com/morluto/rea/blob/main/docs/signal-recordings.md) | Logic-signal decoding with sigrok-cli (Linux) | [[Signal Recordings]] |

## Development

| File | Description | Vault note |
| --- | --- | --- |
| [architecture.mermaid](https://github.com/morluto/rea/blob/main/docs/architecture.mermaid) | Layered architecture diagram | [[Architecture]], [[Codebase Map]] |
| [testing.md](https://github.com/morluto/rea/blob/main/docs/testing.md) | Test strategy and real-toolchain verification lanes | [[Testing]] |
| [client-verification.md](https://github.com/morluto/rea/blob/main/docs/client-verification.md) | Optional native-client and agent-evaluation lanes | [[Testing]] |
| [reconstruction-readiness.md](https://github.com/morluto/rea/blob/main/docs/reconstruction-readiness.md) | Development conformance verifier for reconstruction readiness | [[Testing]] |
| [tool-design.md](https://github.com/morluto/rea/blob/main/docs/tool-design.md) | How to design MCP tools around the analyst question | [[Tool Design Principles]] |
| [releasing.md](https://github.com/morluto/rea/blob/main/docs/releasing.md) | Release Please workflow and release conventions | [[Releasing]] |
| [releases/6.4.0.contributions.json](https://github.com/morluto/rea/blob/main/docs/releases/6.4.0.contributions.json) | Full contribution record (commits and PRs) for 6.4.0 | [[Releasing]] |
| `assets/` (`discord.svg`, `rea-hopper-analysis.png`, `rea-showcases.png`) | Images used by the README and docs site | [[Showcases]] |

## Root-level documents

| File | Description | Vault note |
| --- | --- | --- |
| [README.md](https://github.com/morluto/rea/blob/main/README.md) (+ 18 translations) | Product overview, quick start, target table, showcases, FAQ | [[What is REA]], [[Quick Start]] |
| [AGENTS.md](https://github.com/morluto/rea/blob/main/AGENTS.md) | Repository guidelines for contributors and coding agents | [[Architecture]], [[Evidence Model]], [[Contributing]] |
| [CONTRIBUTING.md](https://github.com/morluto/rea/blob/main/CONTRIBUTING.md) | Development setup, checks and contribution scope | [[Contributing]] |
| [CHANGELOG.md](https://github.com/morluto/rea/blob/main/CHANGELOG.md) | Release notes, including breaking changes | [[Updating and Uninstalling]], [[Releasing]] |
| [DISCLAIMER.md](https://github.com/morluto/rea/blob/main/DISCLAIMER.md) | Responsible use and liability | [[Legal and Responsible Use]] |
| [SECURITY.md](https://github.com/morluto/rea/blob/main/SECURITY.md) | Vulnerability reporting and security boundary | [[Legal and Responsible Use]] |
| [THIRD_PARTY_NOTICES.md](https://github.com/morluto/rea/blob/main/THIRD_PARTY_NOTICES.md) | Dependency, engine and commercial-tool licensing | [[Legal and Responsible Use]] |
| [LICENSE](https://github.com/morluto/rea/blob/main/LICENSE) | MIT licence | [[Legal and Responsible Use]] |
| [.agents/skills/reverse-engineer-anything/SKILL.md](https://github.com/morluto/rea/blob/main/.agents/skills/reverse-engineer-anything/SKILL.md) | The bundled agent workflow skill | [[Agent Skill]] |

> [!tip] Generated catalog
> `docs/public/product-catalog.json` is generated by `npm run docs:generate` and is not checked in. For a running server, `binary_session` is authoritative. See [[MCP Tool Catalog]].

## Source
- https://github.com/morluto/rea/tree/main/docs
- https://github.com/morluto/rea/blob/main/docs/index.md
