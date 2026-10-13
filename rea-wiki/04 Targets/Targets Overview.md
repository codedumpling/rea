---
tags: [rea, target, overview]
source:
  - README.md
  - docs/index.md
---
# Targets Overview

REA groups what it can analyze into **target families**. Each family has its own tools (CLI command + MCP tool with the same name, hyphens vs. underscores), its own requirements, and its own boundary between *static* inspection (reads bytes, never runs the target) and *runtime* capture (launches, attaches to, or drives a target with your user permissions).

> [!info] Baseline requirement
> Every family needs Node.js 22.x (>=22.19), 24.x (>=24.11), or 26+, plus npm. Everything else in the table below is *additional* and target-specific. See [[Installation]] and [[Readiness Checks]].

## What you can analyze

| Target | What REA returns | Requirements | Note |
| --- | --- | --- | --- |
| Native binaries | Pseudocode, assembly, strings, symbols, calls and references | Hopper, Ghidra or IDA | [[Native Binaries]] |
| Offline ELF layout | Sections, segments, original symbols/relocations and static mitigation candidates | Caller-supplied pwntools on Linux x64 | [[ELF Layout and Recorded Crashes]] |
| Go binaries | Embedded compiler version, modules, replacements and build settings | Built-in static ELF/PE/thin Mach-O inspection; no Go installation | [[Go Binaries]] |
| WASM artifact | Validation, sections, exact import/export WAT forms and decoded WAT | Caller-supplied WABT 1.0.42 | [[WASM Artifacts]] |
| EVM bytecode | Dispatch selectors, byte offsets, inferred arguments and mutability | Local raw/hex carrier on Linux x64 | [[EVM Bytecode]] |
| SQLite databases | Tables, columns, indexes, views, triggers and selected ordinary-table rows | Local database/WAL snapshot; Node.js 24.x >=24.15 or 26+ | [[SQLite Databases]] |
| Recorded Linux crashes | Raw notes, every recorded thread's registers/signals, optional mapping candidates | Caller-supplied pwntools on Linux x64; optional GDB/pwndbg | [[ELF Layout and Recorded Crashes]] |
| JavaScript / Electron | Modules, imports, source maps, routes, IPC and native add-on relationships | Node.js and npm | [[JavaScript and Electron]], [[JavaScript Runtime Observation]] |
| Websites | Page structure, scripts, network observations and requested screenshots | A Chrome-family browser | [[Websites and Browser Observation]] |
| Saved network captures | Requests, responses, exposed payloads and source locations | HAR or Fiddler Everywhere SAZ; mitmdump on Linux for native mitmproxy | [[Network and Packet Captures]] |
| .NET assemblies | Metadata, CIL instructions, declared native dependencies and build comparisons | None (static only) | [[DotNET Assemblies]] |
| Android APKs | Manifest declarations, classes, decompiled methods and references | Headless JADX and a full JDK on Linux/macOS/Windows x64 | [[Android]] |
| Android devices | Devices, packages, processes, logs, dumps, screen captures, APK/file transfers | Caller-supplied adb (emulator or hardware) | [[Android]] |
| JEB projects | Engine-backed project units, artifact digests and type/method pseudo-code | Caller-started JEB client serving MCP, selected with `REA_JEB_MCP_URL` | [[JEB]], [[Android]] |
| Android resources | Decoded manifest, string tables, locales and version facts | Caller-supplied Apktool launcher | [[Android]] |
| Flutter apps | Dart snapshot hash, per-ABI library digests and engine build provenance | None (static APK parsing) | [[Flutter Apps]] |
| Firmware | Regions, extraction results and native-analysis handoffs | Binwalk / Unblob on Linux | [[Firmware]] |
| Recorded packets | Selected fields, repeated values, filters and decode-as rules | Linux: TShark 4.x + prlimit | [[Network and Packet Captures]] |
| Recorded logic signals | Decoder annotations, sample ranges and diagnostics | Linux: sigrok-cli 0.7.2 + prlimit | [[Signal Recordings]] |
| Packages and resources | File inventories, digests, plists, Apple bundle anatomy and extracted resources | Built in | [[JavaScript and Electron]], [[Apple and Harmony Apps]] |
| Process behavior | Terminal output, interactions, exit and filesystem observations, run comparisons | Linux/macOS with a native PTY | [[Process Capture]] |

## Static vs. runtime

> [!warning] Runtime capture has side effects
> Static JavaScript and .NET inspection read the supplied files without running the application. Runtime capture runs or interacts with the selected target using **your** user permissions; it is not a sandbox. Each target note has a "Limits and side effects" section.

| Mode | Families / tools |
| --- | --- |
| Pure static (reads bytes only) | Go, WASM, EVM, SQLite, ELF layout, PE resources, recorded crashes, .NET, Flutter, Android APK (JADX), Apple/Harmony inventory, JS/Electron static analysis, saved HAR/SAZ, packet & signal recordings, firmware |
| Passive attach (never drives the target) | `inspect_web_page`, `observe_web_session`, `inspect_electron_page`, `observe_javascript_runtime` |
| Instrumenting attach | `observe_web_execution` (precise V8 coverage changes JIT/timing) |
| Owned/active runtime | `capture_browser_scenario`, `capture_electron_scenario`, `capture_process_scenario`, `observe_native_calls`, `capture_native_ui_scenario`, device-mutating ADB tools, [[Frida]] |

## Common patterns across families

- **Caller-supplied engines.** REA never installs third-party tools (pwntools, WABT, TShark, sigrok, Binwalk, Unblob, Wakaru, JADX, Apktool, adb, mitmdump). You point REA at them with environment variables; see [[External Tools]] and [[Environment Variables]].
- **Evidence everywhere.** Every successful call returns an Evidence record with artifact path, SHA-256, limitations and coverage; see [[Evidence Model]].
- **Summary + retained views.** Many MCP tools accept `detail: "summary"`, retain the complete Evidence in the session and return a `normalized_result.parent_evidence_id` that you page with `inspect_analysis_view`. CLI output is always complete. See [[MCP Tool Catalog]] and [[Sessions and Snapshots]].
- **Absence is not proof.** Empty results, `null` fields and missing events mean "not observed in this scope," never "does not exist."

## Choosing a starting point

1. Native executable? → [[Native Binaries]] (then pick a provider: [[Hopper]], [[Ghidra]], [[IDA]]). Before that, [[Go Binaries]] or [[ELF Layout and Recorded Crashes]] can answer quick metadata questions without a provider.
2. Electron app / `.asar` / JS bundle? → [[JavaScript and Electron]].
3. Live website? → [[Websites and Browser Observation]].
4. APK? → [[Android]], and [[Flutter Apps]] if it ships `libapp.so`.
5. `.app`, IPA, DMG, `.hap`? → [[Apple and Harmony Apps]].
6. Firmware blob? → [[Firmware]], then hand ELF children to [[Ghidra]].

See also [[Recipes]], [[CLI Reference]], [[Providers Overview]].

## Source
- https://github.com/morluto/rea/blob/main/README.md
- https://github.com/morluto/rea/blob/main/docs/index.md
