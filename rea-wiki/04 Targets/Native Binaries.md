---
tags: [rea, target, native]
source:
  - docs/native-investigation.md
  - docs/raw-binary.md
  - docs/apple-application-analysis.md
  - README.md
---
# Native Binaries

Native executables and libraries (Mach-O, ELF, PE, and with Ghidra also headerless raw images and DOS programs) are analyzed through a **deep-analysis provider**: [[Hopper]], [[Ghidra]] or [[IDA]]. REA opens the target in the provider and exposes functions, instructions, pseudocode, strings, references and types as Evidence.

## What REA returns

- Function inventory, function dossiers, assembly and pseudocode.
- Strings (with exact bytes, encoding and termination), symbols, calls and cross-references.
- Ghidra-specific exact-object primitives: decoded instruction facts, static call-target resolution, database data types, value def-use traces, jump tables.
- Static Apple metadata: Mach-O load commands, Objective-C/Swift dispatch metadata, Interface Builder, keyed archives, asset catalogs, dylib resolution.
- Optional macOS runtime observation: selected-window UI capture and LLDB call-entry tracing.

## Requirements and host support

| Provider | Hosts | Notes |
| --- | --- | --- |
| [[Hopper]] | macOS 12+, Ubuntu 24.04+, Fedora 41+, Nobara 44+, 64-bit Arch Linux, CachyOS, Omarchy | Setup can optionally install Hopper with approval |
| [[Ghidra]] | Linux x64/arm64, macOS x64/arm64; experimental Windows x64 P0 | Ghidra 12.1.x + the 64-bit full JDK it declares + matching native decompiler |
| [[IDA]] | Via an upstream MCP registration | Set `REA_IDA_MCP_CONFIG`, select with `--provider ida` |

> [!warning] Windows Ghidra P0
> Experimental Windows support admits native x86/x86-64 PE applications and DLLs on local NTFS only, and is **read-only** (no annotation). Raw-binary targets are unsupported there. See [[Ghidra Variants]].

Ghidra has no automatic fallback: REA never switches to Hopper on its own. Specialized Ghidra lanes (DOS, MIPS, RISC-V, PSP, NativeAOT) are covered in [[Ghidra Variants]].

## CLI and MCP surface

Start with an inventory, then select an exact name or address:

```bash
rea inspect /absolute/path/to/program --provider ghidra --json
rea search /absolute/path/to/program entry --kind procedures --provider ghidra --json
rea function /absolute/path/to/program PROCEDURE --provider ghidra --json
```

| CLI command | MCP tool | Purpose |
| --- | --- | --- |
| `rea inspect` | `open_binary` | Open target, overview |
| `rea search` (`--kind strings\|procedures`) | `search_strings`, `search_procedures`, `list_procedures`, `list_strings` | Inventory and search |
| `rea function` | `analyze_function` | Function dossier |
| `rea decompile` | `procedure_pseudo_code`, `batch_decompile` (MCP only) | Pseudocode |
| `rea instructions` | `procedure_assembly`, `read_function_instructions` | Assembly |
| `rea xrefs` | `xrefs`, `procedure_references`, `procedure_callers`, `procedure_callees` | References |
| `rea trace` | `trace_feature` | Bounded feature trace from a query |
| `rea inspect-native-instruction` | `inspect_native_instruction` | Bytes, tokens, typed refs, `incoming_references` |
| `rea resolve-native-call-targets` | `resolve_native_call_targets` | Direct / indirect / ambiguous / unresolved calls |
| `rea inspect-native-data-type` | `inspect_native_data_type` | Struct/union/enum layouts (`--type` or `--address`) |
| `rea trace-native-values` | `trace_native_values` | High p-code def-use across resolved calls (Ghidra) |
| `rea trace-native-ui-action` | `trace_native_ui_action` | UI wiring → implementation → bounded call refs |
| `rea inspect-native-dispatch-metadata` | `inspect_native_dispatch_metadata` | ObjC classes/methods/ivars/protocols, Swift conformances/vtables |
| `rea annotate-native-function` | `annotate_native_function` | Rename + entry comments in the ephemeral DB |
| — | `set_address_name`, `set_addresses_names` | Cheap batch naming of globals/helpers |
| `rea read-bytes`, `rea address-to-file-offset` | `read_bytes`, `address_to_file_offset` | Raw bytes, original-file coordinates |
| `rea inspect-native-load-image` | `inspect_native_load_image` | Independent load-image verification |
| `rea inspect-macho`, `rea inspect-signature`, `rea inspect-plist` | `inspect_macho`, `inspect_signature`, `inspect_plist` | Mach-O, code signing, plists |
| `rea trace-dylib-resolution` | `trace_dylib_resolution` | dyld path expansion per process root |
| `rea decode-interface-builder`, `rea inspect-keyed-archive`, `rea inspect-asset-catalog` | same names (snake_case) | Nib/storyboard, NSKeyedArchiver, `Assets.car` |
| `rea observe-native-calls` | `observe_native_calls` | LLDB entry breakpoints on a launched Mach-O |
| `rea observe-native-ui`, `rea capture-native-ui-scenario` | `observe_native_ui`, `capture_native_ui_scenario` | Selected-window capture and AX actions |

> [!tip] Names that start with a dash
> `function`, `instructions`, `decompile` accept `--procedure=<name-or-address>`; `xrefs` accepts `--address=`; `search` accepts `--pattern=`; `trace` accepts `--query=`. Use the equals form for Objective-C names: `rea function ./app '--procedure=-[REAWidget delegate]' --provider hopper --format json`.

## Worked example

> [!example] From inventory to call observation
> ```bash
> # 1. Find candidates
> rea search /Applications/Example.app/Contents/MacOS/Example open --kind procedures --provider ghidra --json
> # 2. Function dossier and instruction facts
> rea function /path/Example --procedure=_open_wrapper --provider ghidra --json
> rea inspect-native-instruction /path/Example 0x100003f20 --provider ghidra
> rea resolve-native-call-targets /path/Example 0x100003f44 --provider ghidra
> # 3. Confirm at runtime (macOS, owned process under LLDB)
> rea observe-native-calls /Applications/Example.app '{"breakpoints":[{"kind":"function","name":"open","module":"libsystem_kernel.dylib"}],"duration_ms":5000,"backtrace_frames":4}' --json
> ```
> The static steps produce *inferences* about references; step 3 produces *observations* of call entries.

## Headerless raw binaries (Ghidra only)

Supply CPU family and the address where file offset 0 is mapped; REA never infers this from a `.bin` suffix:

```bash
rea inspect /absolute/path/to/ecu.bin --target-format raw-binary --architecture arm --entry-point 0x8000000 --provider ghidra --json
```

MCP: `open_binary` with `format: "raw-binary"`, `architecture` (`x86`, `x86_64`, `arm`, `arm64`), `entry_point`, `provider_id: "ghidra"`. ARM/AArch64 entries must be 4-byte aligned. Hopper and IDA refuse raw targets. Big-endian, Thumb-only and Cortex-M variants are unsupported; reset vectors and MMIO are not emulated.

## Limits and side effects

- **Static results are inferences.** Empty incoming references do not prove an address is unused at runtime; call resolution does not classify ObjC/Swift/vtable dispatch from names; jump tables describe the decompiler's recovered switch.
- **Ephemeral databases.** Ghidra annotations live in an ephemeral database; they never change executable bytes. After metadata edits, snapshot save/import is unavailable in that session — export Evidence with `export_evidence_bundle` and reopen. See [[Sessions and Snapshots]].
- **Artifact identity.** If the source changes before Ghidra's private copy is taken, queries report `artifact_changed`.
- **Bounds.** Ghidra serializes at most 20,000 fields per type (check `total_fields`/`truncated`); value traces are bounded to 8 MiB node payloads / 32 MiB results; Hopper regex has a 5-second deadline.
- **`observe_native_calls`** launches the target as a new owned process: it may change files, show UI or use the network. Only entries are observed (no return values, stack/FP args, inlined or `objc_direct` calls). Hardened-runtime targets without `com.apple.security.get-task-allow` fail with `debugger-attach-denied`.
- **`capture_native_ui_scenario`** needs Screen Recording/Accessibility permission (macOS 14+), uses no global event injection, and leaves app state as-is; UI actions may change application data.

## Related

- Quick metadata without a provider: [[Go Binaries]], [[ELF Layout and Recorded Crashes]].
- Handoffs into native analysis: [[Firmware]], [[DotNET Assemblies]] (NativeAOT), [[JavaScript and Electron]] (native add-ons).
- Runtime hooks: [[Frida]].
- [[Providers Overview]], [[MCP Tool Catalog]], [[Evidence Model]].

## Source
- https://github.com/morluto/rea/blob/main/docs/native-investigation.md
- https://github.com/morluto/rea/blob/main/docs/raw-binary.md
- https://github.com/morluto/rea/blob/main/docs/apple-application-analysis.md
