---
tags: [rea, provider, ghidra]
source:
  - docs/installation.md
  - docs/ghidra-provider.md
  - docs/native-investigation.md
  - src/ghidra/
  - bridge/ghidra/
  - src/config/environment.ts
---
# Ghidra

Provider ID: `ghidra`. REA drives an **existing** Ghidra installation in headless mode. Each session imports one target into an **ephemeral** Ghidra database through `analyzeHeadless` and talks to a packaged Java bridge (`bridge/ghidra/ReaGhidraBridge.java`, a `HeadlessScript`). Executable bytes are never modified. REA never controls the GUI and never opens or saves your own Ghidra projects.

## Requirements

- **Ghidra 12.1.x** (verified with 12.1.4).
- The **64-bit full JDK** inside the range the installation declares (`application.java.min`/`application.java.max`). For current 12.1 releases that means JDK 21 or newer, with no maximum. `javac` must be present.
- The **native decompiler** for your host, in `Ghidra/Features/Decompiler/os/<platform>/` or `build/os/<platform>/`. REA checks the executable's own header, so a binary built for the wrong platform is reported as incompatible. Linux ARM64 uses `linux_arm_64`. Official archives may lack it for some hosts, in which case you build it yourself.
- **Hosts:** Linux x64/arm64, macOS x64/arm64, plus experimental Windows x64 P0 (see [[Ghidra Variants]]).
- Python and PyGhidra are **not** required.

```bash
export GHIDRA_INSTALL_DIR=/absolute/path/to/ghidra_12.1.4_PUBLIC
export JAVA_HOME=/absolute/path/to/jdk-21   # optional if java/javac are on PATH
rea doctor --json
rea setup                                    # can save verified Ghidra/JDK paths into agent registrations
```

Doctor validates the platform and architecture, the Ghidra 12.1.x version, `support/analyzeHeadless` (or `.bat`), the Java major range, JDK bitness and `javac`. Setup **never** downloads, installs or upgrades Ghidra or Java.

## What it gives you

- **25 read-only operations:** 13 inventory/name/search operations and 12 function-analysis operations. They cover metadata, decompilation, assembly, resolved calls, typed references, xrefs, function dossiers, instructions, recovered data types, load mappings, loaded memory bytes and file offsets.
- **Session-only edits** (Linux/macOS only): `annotate_native_function` (atomic name and/or entry-comment edit with a refreshed dossier), `set_address_name` and `set_addresses_names`. Edits are discarded on close. Windows P0 is read-only.
- Bounded **p-code value flow** in dossiers: up to 3,000 high-p-code ops, 64 inputs per op and 12,000 def-use edges, with truncation reported. `trace_native_values` composes these flows across resolved calls.
- Independent **load-image attestation** for DOS MZ, explicit COM and raw-binary imports. PE gets measurements but an `unsupported` attestation status.

Ghidra's semantics are specific:
- Addresses are lowercase `0x` hex. Non-default spaces use the form `<space>:0x<hex>`.
- Memory-block ends are exclusive.
- String search covers only Ghidra-defined string `Data`, so an empty result does not prove a string is absent.
- Callers and callees include only resolved functions. Targetless computed flow stays unknown.
- Inventories and searches return the complete matching set, with no paging.
- Pseudocode is Ghidra's output, never original source, and is never compared as equal to Hopper's text.

## Lifecycle

- `open_binary` validates the target-provider binding only. **The first Ghidra-backed query** triggers import and auto-analysis, and the bridge socket appears only once default auto-analysis has finished.
- Startup deadline: **330,000 ms** by default. Override with `REA_GHIDRA_STARTUP_TIMEOUT_MS` (integer from 1 to 2,147,483,647). Your MCP client's per-call timeout is separate and can expire first. See [[MCP Server]].
- REA passes `-readOnly` and `-deleteProject`, uses isolated home/cache/config/temp paths, and loads the bridge with `-scriptPath`. On Linux/macOS the transport is an owner-only Unix socket. If the socket path would exceed the host limit, REA uses a separate owner-only directory under `/tmp`. On Windows it is authenticated IPv4 loopback.
- One persistent `DecompInterface` per Program, with a FIFO serial queue and no fixed queue length or response ceiling. Cancellation removes queued work.
- On non-Windows hosts, an import repair clears incorrect no-return flags on `malloc`, `__tls_get_addr`, `memcpy`, `memset`, `strlen` and `__errno_location`.

## Configuration environment variables

| Variable | Purpose |
| --- | --- |
| `GHIDRA_INSTALL_DIR` | Absolute path to the Ghidra installation (required) |
| `JAVA_HOME` | Absolute JDK home (optional if Java is on `PATH`) |
| `REA_GHIDRA_STARTUP_TIMEOUT_MS` | Startup deadline (default 330000) |
| `REA_GHIDRA_LANGUAGE_ID` | Overrides the detected language, e.g. `x86:LE:32:default`. Passed as `-processor` |
| `REA_GHIDRA_COMPILER_SPEC_ID` | Compiler spec, e.g. `borlandcpp`. Requires a language. Passed as `-cspec` |
| `REA_GHIDRA_SEED_FILE` | Absolute TSV of `address<TAB>kind[<TAB>name]`, where kind is `function`, `code` or `label` |
| `REA_GHIDRA_DISABLED_ANALYZERS` | Comma-separated exact analyzer names to skip (max 32, each ≤120 chars) |
| `GHIDRA_HEADLESS_MAXMEM` → `GHIDRA_MAXMEM` → `2G` | JVM `-Xmx`; the first non-empty value wins |
| `REA_GHIDRA_NATIVEAOT_JAR` | Optional NativeAOT annotation JAR (see [[Ghidra Variants]]) |

Language, compiler spec, seed digest and disabled analyzers are committed to the **analysis profile**. If the profile and settings later disagree, opening the session fails, so snapshots cannot replay under different settings. DOS imports reject a language override.

> [!example] Unstick a slow analyzer
> `export REA_GHIDRA_DISABLED_ANALYZERS="Objective-C Message Analyzer"` is useful when auto-analysis stalls on a Swift/Objective-C arm64 Mach-O. The profile records it as `ghidra-default+disabled:Objective-C Message Analyzer`.

> [!warning] Java option injection is cleared
> REA clears inherited `_JAVA_OPTIONS`, `JAVA_TOOL_OPTIONS`, `JDK_JAVA_OPTIONS` and `GHIDRA_JAVA_OPTIONS`, and owns `GHIDRA_HEADLESS_JAVA_OPTIONS`. Use the heap settings above. The Linux/macOS launch sets `-XX:ParallelGCThreads=2 -XX:CICompilerCount=2`. REA does not pass `-max-cpu`. If you need to limit CPU, use host affinity controls.

## Troubleshooting

- **`native_decompiler` missing on macOS:** build it from Ghidra's source. On x64: `cd "$GHIDRA_INSTALL_DIR/Ghidra/Features/Decompiler/src/decompile/cpp" && make ghidra_opt`, then install it as `os/mac_x86_64/decompile`. On Apple Silicon: `make ghidra_opt ARCH_TYPE="-arch arm64" OSDIR=mac_arm_64`, then install to `os/mac_arm_64/decompile`. If your toolchain rejects old flags, add `ADDITIONAL_FLAGS="-w"`.
- **Client timed out on the first query:** raise the client timeout or `REA_GHIDRA_STARTUP_TIMEOUT_MS`, then close and reopen. See [[Troubleshooting]].
- **Unexpected failures:** `details.diagnostics.failure_cause` preserves the internal cause chain, with bridge tokens redacted.
- **Memory:** a 512 MiB heap works for small targets. Large targets need more. Check the effective `-Xmx` on the owned JVM itself (on Linux, `/proc/<pid>/cmdline`), not in the parent shell.

## Verification

`npm run verify:ghidra` (host-native ELF/Mach-O plus a DWARF type object), `verify:ghidra:cross-format` (needs `clang`, LLD and `lld-link`; select them with `REA_CLANG`, `REA_LLD`, `REA_LLD_LINK`), `verify:ghidra:seeds`, `verify:ghidra:swift`, `verify:ghidra:switch`, `verify:ghidra:aarch64-jump-table`. Variant lanes are listed in [[Ghidra Variants]], and the whole matrix is in [[Testing]].

See also [[Providers Overview]], [[Native Binaries]], [[Environment Variables]].

## Source
- https://github.com/morluto/rea/blob/main/docs/ghidra-provider.md
- https://github.com/morluto/rea/blob/main/docs/installation.md#ghidra
