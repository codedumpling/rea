---
tags: [rea, target, dotnet, managed]
source:
  - docs/managed-code-analysis.md
  - docs/ghidra-nativeaot.md
  - src/dotnet/
---
# .NET Assemblies (managed code)

REA inspects .NET PE/CLI artifacts **without loading or executing** them. It identifies an assembly, inspects metadata and CIL (Common Intermediate Language), compares members across builds, and connects declared native calls (P/Invoke) to supplied native-analysis evidence. The parser is REA-owned TypeScript shipped with the package — no .NET runtime, SDK or ILSpy is required.

## What REA returns

- Artifact identity: path, byte length, SHA-256, PE/CLI classification, assembly and module identities (MVID), target frameworks, metadata coverage.
- Members: types, fields, methods, build-local metadata tokens, signatures, CIL bodies, decoded call edges and field accesses.
- Declared native boundaries and native implementation indicators.
- Cross-build member comparisons with token remapping through observed structure.

## Requirements

None beyond Node.js — static only. The parser inspects Windows-produced PE/CLI bytes on REA's Linux and macOS hosts with the same result. Optional: `REA_ILSPY_CMD_PATH` (absolute `ilspycmd`) for the real ILSpy oracle used by `verify:managed` and for importing C# reconstruction inference.

## The seven tools

| Task | MCP tool | CLI |
| --- | --- | --- |
| Identify artifact and managed deployment form | `inspect_managed_artifact` | `rea inspect-managed-artifact` |
| Types, signatures, method bodies, calls, field access | `inspect_managed_members` | `rea inspect-managed-members` |
| Declared native calls / implementation indicators | `inspect_managed_native_boundaries` | `rea inspect-managed-native-boundaries` |
| Compare members across builds, map tokens | `compare_managed_members` | `rea compare-managed-members` |
| Check P/Invoke declarations against native export/function evidence | `verify_managed_native_boundaries` | `rea verify-managed-native-boundaries` |
| Import decompiled code against verified member identities | `import_managed_reconstruction` | `rea import-managed-reconstruction` |
| Add managed findings to an application graph | `project_managed_application_graph` | `rea project-managed-application-graph` |

```bash
rea inspect-managed-artifact /absolute/path/Assembly.dll --json
rea inspect-managed-members /absolute/path/Assembly.dll --json
rea inspect-managed-members /absolute/path/Assembly.dll --type Example.Parser
```

MCP focused selection: `{"path":"/absolute/path/Assembly.dll","type_names":["Example.Parser"]}` (names match reported `full_name` exactly).

> [!tip] Summary first
> `inspect_managed_artifact` with `{"path": "...", "detail": "summary"}` retains complete Evidence and returns an overview. Page rows with `inspect_analysis_view`, view `{"kind":"managed","collection":"attributes","offset":0,"limit":16}` — collections are `references`, `resources`, `attributes`.

## Worked example

> [!example] What changed between two builds, and which native DLLs does it call?
> 1. `rea inspect-managed-artifact old/App.dll --json` and `new/App.dll` — confirm both are PE/CLI and note MVIDs.
> 2. `rea compare-managed-members` on the two paths — members are paired by exact CIL/signature identity, then exact type+name+raw signature, then structural body shape; ambiguity stays ambiguous.
> 3. `rea inspect-managed-native-boundaries new/App.dll` → declared P/Invoke modules/entry points.
> 4. Analyze the native DLL with [[Ghidra]] ([[Native Binaries]]), then `verify_managed_native_boundaries` with that Evidence. Name/module matches establish declaration links, not executed calls.

## NativeAOT, ReadyToRun, IL2CPP

- **NativeAOT** strips CIL bodies; the image (PE, ELF or Mach-O, including shared libraries) is a native target for [[Hopper]]/[[Ghidra]]. REA has a built-in read-only parser for x64 PE/RTR 9.1 metadata and an optional Linux headless Ghidra adapter requiring a locally built `REA_GHIDRA_NATIVEAOT_JAR`. See [[Ghidra Variants]].
- The static tools do not unpack single-file hosts, decode IL2CPP metadata, or infer NativeAOT identity from an ordinary PE. Metadata tokens are not native addresses.

## Limits and side effects

- Input files ≤ 128 MiB; metadata/CIL/projection representation budget 64 MiB. Overflow → `resource-limit` with partial/unavailable coverage, never fake empty observations.
- Tokens are build-local: cross-observation matching needs the artifact SHA-256 and MVID. A caller cannot carry `0x06001234` to a new build.
- `.NET Standard` assemblies keep `runtime_family: unknown` unless markers (e.g. ReadyToRun) identify it.
- Absence of declared imports does not exclude dynamic resolution, generated code or native helpers.
- No CLR execution: runtime behavior stays unknown. [[Process Capture]] can record a run's I/O but not which managed methods executed.
- Imported ILSpy output is reconstruction **inference**, non-canonical.

> [!info] Verification
> `npm run verify:managed` — semantic-fact conformance on byte-built fixtures; optional ILSpy oracle. See [[Testing]].

## Related
[[Native Binaries]] · [[Ghidra Variants]] · [[JavaScript and Electron]] (application graph) · [[Evidence Model]]

## Source
- https://github.com/morluto/rea/blob/main/docs/managed-code-analysis.md
- https://github.com/morluto/rea/blob/main/docs/ghidra-nativeaot.md
