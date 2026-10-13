---
tags: [rea, provider, overview]
source:
  - README.md
  - docs/cli.md
  - docs/installation.md
  - docs/mcp-contracts.md
  - src/application/binary/AnalysisProviderRegistry.ts
  - src/contracts/providerSelection.ts
  - src/config/environment.ts
  - docs/architecture.mermaid
---
# Providers Overview

REA does not ship its own disassembler or decompiler. **Deep native analysis** (functions, pseudocode, assembly, strings, xrefs, CFG, annotations) is delegated to an external engine through a **provider adapter**. Every other capability that relies on a third-party program — JADX, TShark, Binwalk, WABT and others — uses a separate "bring your own tool" adapter (see [[External Tools]]).

Every engine is **bring-your-own**: REA never installs, upgrades, activates or licenses Ghidra, Java, IDA, JEB or Frida server. The only exception is Hopper: `rea setup` can install it, but only after you approve that step separately (see [[Installation]]).

## The three deep providers

| Provider ID | Engine | Hosts | Mutation | Note |
| --- | --- | --- | --- | --- |
| `hopper` | Hopper Disassembler (commercial; free demo) | macOS 12+, supported Linux distros | Names, comments, bookmarks in the live Hopper document | [[Hopper]] |
| `ghidra` | Ghidra 12.1.x headless + JDK 21+ | Linux x64/arm64, macOS x64/arm64, experimental Windows x64 P0 | Session-only names/comments (discarded on close); Windows P0 is read-only | [[Ghidra]], [[Ghidra Variants]] |
| `ida` | IDA Pro / IDA Free through an upstream IDA MCP server | Wherever the upstream server runs | None: REA's IDA analysis is read-only | [[IDA]] |

[[JEB]] (Android/Java decompilation through JEB's own MCP) and [[Frida]] (dynamic instrumentation) are **auxiliary capabilities** with their own tools. They are not deep `open_binary` providers and are not part of provider selection.

## Provider selection: deterministic, with no fallback

Provider selection lives in `AnalysisProviderRegistry` (`src/application/binary/`). Its source comments describe it as *"sorted deep candidates, deterministic selection, no fallback"*.

1. **Discovery never starts an engine.** For each configured provider, the registry checks availability (for example: is the launcher present, is `GHIDRA_INSTALL_DIR` valid, is `REA_IDA_MCP_CONFIG` set) and target support (format, architecture, kind). Candidates are sorted by provider ID, so registration order never changes the result.
2. **Selector precedence** is: an explicit request (`--provider ID` on the CLI, or `provider_id` on `open_binary`), then the `REA_ANALYSIS_PROVIDER` environment variable, then `auto` (the default).
3. **`auto`** binds the single usable candidate. If more than one provider can handle the target, selection fails with `ambiguous` and lists `details.candidate_ids`. REA does not pick one for you.
4. **An explicit or environment selector** binds exactly that provider. If it is unknown, unsupported for the target or unavailable, you get an error. REA never substitutes another engine.
5. Once bound, the provider **stays bound to the session** until an explicit switch or close. If the provider fails at runtime, REA returns the original reason and still does not switch engines automatically.

```bash
rea providers --json        # configured providers and supported operations
rea capabilities --json     # auxiliary capabilities
rea analyze /abs/program --provider ghidra
```

```json
{"name":"open_binary","arguments":{"path":"/abs/program","provider_id":"ghidra"}}
```

Failure reasons you may see in `capability_unavailable` errors include `ambiguous`, `provider_unavailable`, `target_unsupported`, `unknown_provider` and `invalid_options`. Rejection codes for individual candidates include `not_configured`, `executable_missing`, `runtime_missing`, `unsupported_host`, `unsupported_version`, `target_format_unsupported`, `architecture_unsupported`, `target_role_unsupported`, `managed_target_unsupported` and `version_unresolved`. See [[Errors and Exit Codes]].

> [!tip] Diagnose one engine
> `rea doctor --provider ghidra --json` (or `hopper`, `ida`) runs only that provider's checks. Doctor is strictly read-only. See [[Readiness Checks]].

## Which provider can open which target

| Target | Hopper | Ghidra | IDA |
| --- | --- | --- | --- |
| Mach-O / ELF / PE executables | yes | yes | yes |
| Hopper database (`.hop`) | yes | no | no (open the original input instead) |
| MIPS / RISC-V ELF | no | yes, admitted lanes only ([[Ghidra Variants]]) | no |
| DOS MZ / COM | no | yes | no |
| Headerless raw binary | no | yes, with caller-supplied architecture and entry point | no |
| Windows-native host | no | x86/x86-64 PE only (P0) | yes, if the upstream server runs there |

## Shared guarantees

- **Executable bytes are never modified.** Ghidra works on a private snapshot in an ephemeral project, and IDA headless uses a digest-verified private copy.
- **Profile identity.** Each binding commits an analysis profile, which records things like language, seeds, disabled analyzers and the extension JAR digest. Snapshots replay only under a matching profile. See [[Sessions and Snapshots]].
- **Effects are declared.** Capabilities advertise `mutatesArtifact`, `launchesProcess`, `mayShowUi` and similar flags, and `binary_session` reports current tool availability. See [[MCP Server]].
- **Ownership of cleanup.** If cleanup fails, the result is `cleanup_incomplete` and REA keeps ownership of the resources. It never kills unrelated processes.
- **Startup deadlines** are configurable: `REA_GHIDRA_STARTUP_TIMEOUT_MS` (default 330,000 ms) and `REA_HOPPER_STARTUP_TIMEOUT_MS` (default 120,000 ms). Your MCP client's own request timeout must allow at least as long.

See also [[Architecture]], [[Native Binaries]], [[Environment Variables]].

## Source
- https://github.com/morluto/rea/blob/main/docs/cli.md
- https://github.com/morluto/rea/blob/main/docs/installation.md
- https://github.com/morluto/rea/blob/main/src/application/binary/AnalysisProviderRegistry.ts
