---
tags: [rea, target, flutter, android, mobile]
source:
  - docs/flutter-analysis.md
  - src/flutter/
---
# Flutter Apps

REA identifies a Flutter app's build facts by **pure parsing of its APK** — no engine, no tool installation, no execution and no external lookups. This is the triage step for Flutter reversing: the Dart snapshot hash maps (via an external dataset such as reFlutter's) to a Dart SDK release, and the SDK version decides which snapshot parser applies.

## What REA returns

**`identify_flutter_build` / `rea identify-flutter-build`**
- `flutter_detected` (true when any `lib/<abi>/libapp.so` exists; a negative answer has complete coverage).
- `snapshot_hash` (bytes 20 after the `f5 f5 dc dc` magic); disagreeing values go to `snapshot_hash_candidates` with `snapshot_hash: null`.
- Per-ABI `libflutter` facts: size, SHA-256, GNU build IDs from declared note ranges (`build_id`, `build_id_candidates`), labeled `Dart VM version` strings (often `null` in release builds) and Android clang/LLVM toolchain lines.
- `elf_header_valid`, `target.sha256`.

**`inspect_dart_aot` / `rea inspect-dart-aot`** (one ABI's `libapp.so`)
- The four `_kDart*Snapshot*` sections via the ELF dynamic symbol table, with offsets, sizes and parsed data headers (complete structural coverage requires kind 3, FullAOT, in the Dart 2.14 layout).
- Snapshot hash from the VM and isolate headers.
- String-pool projection from the isolate data section: `package:` URIs (dependency inventory), `dart:` SDK URIs, and identifier-shaped `class_like_tokens` (up to 500 of each retained).

## Requirements

None — static APK parsing on any REA host.

## CLI and MCP

```sh
rea identify-flutter-build /targets/Example.apk
rea inspect-dart-aot /targets/Example.apk
rea inspect-dart-aot /targets/Example.apk --abi armeabi-v7a
```

MCP: `identify_flutter_build` (`path`), `inspect_dart_aot` (`path`, optional `abi`; default preference arm64-v8a → x86_64 → x86 → armeabi-v7a → others alphabetically).

> [!tip] Summary delivery
> `inspect_dart_aot` with `detail: "summary"` returns identity, headers and coverage without the string lists. Page them with `inspect_analysis_view`, view `{kind: "dart-aot", collection: "package_uris", offset: 0, limit: 64}`; collections are `package_uris`, `dart_uris`, `class_like_tokens`.

## Worked example

> [!example] Pin the Dart SDK and list dependencies
> ```sh
> rea identify-flutter-build ./app.apk          # snapshot_hash, engine build IDs per ABI
> rea inspect-dart-aot ./app.apk                # package: URIs → app & dependency Dart files
> ```
> Look up `snapshot_hash` in an external dataset to get the SDK version (REA does not do this), then use an SDK-specific snapshot parser, JEB's GUI Dart unit ([[JEB]]), or native analysis of `libapp.so` ([[Native Binaries]] with [[Ghidra]]). Use [[Android]] for the Java/Kotlin side and resources.

## Limits and side effects

- Owned read-only APK snapshot; the original is never modified. Cancellation is checked throughout.
- Bounds: APK ≤ 2 GiB; 768 MiB cumulative library extraction for identification; 256 MiB per library; 8 MiB ZIP metadata and ELF note metadata; up to 64 build IDs and 64 snapshot hashes; 32 ABIs.
- **No hash-to-SDK lookup.** The string pool is a byte-pattern projection, not deserialized cluster semantics — `class_like_tokens` include incidental matches and are not typed classes/functions.
- `libapp.so` without `libflutter.so` (or vice versa), unknown snapshot kinds, aliased symbols or conflicting build IDs yield `coverage: "partial"`. Payloads without exported snapshot symbols are refused.

> [!info] Verification
> `npm run verify:flutter -- --apk PATH [--plain-apk PATH]`, optionally with `--expectations scripts/fixtures/flutter-gallery-expectations.json` for Flutter Gallery 2.9.2. See [[Testing]].

## Related
[[Android]] · [[Apple and Harmony Apps]] · [[JEB]] · [[Native Binaries]] · [[Targets Overview]]

## Source
- https://github.com/morluto/rea/blob/main/docs/flutter-analysis.md
