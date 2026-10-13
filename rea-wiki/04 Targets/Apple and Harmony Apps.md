---
tags: [rea, target, apple, ios, macos, harmonyos]
source:
  - docs/apple-application-analysis.md
  - docs/harmony-analysis.md
  - docs/native-investigation.md
  - src/artifacts/
---
# Apple and Harmony Apps

Both families start from an **authenticated artifact inventory** (`rea inspect-artifact`, MCP `inspect_artifact` / `inventory_artifact` Evidence) and project it into an execution-free application graph. They report what is *in* the package — paths, hashes, formats, bundle anatomy, runtime-family hints — not what runs.

| Family | Inputs | Projection tool (MCP / CLI) | External tools |
| --- | --- | --- | --- |
| Apple | IPA, macOS `.app` directory, ZIP, DMG | `project_apple_application_graph` / `rea project-apple-application-graph` | None (DMG children need the native macOS mounting adapter) |
| HarmonyOS | `.hap`, `.hsp`, `.app` App Pack | `project_harmony_application_graph` / `rea project-harmony-application-graph` | None |

## Apple applications

### What REA returns

- Every component per inventory category, with SHA-256 and content-derived format; per-occurrence `artifact_kind`, `artifact_format`, executable bit.
- **Application roots**: each `Payload/<Name>.app` in an IPA; the bundle itself for a `.app`; each outermost `<Name>.app` in a directory/ZIP/DMG.
- `platforms` (`macos` for roots with `Contents/`, `ios` for shallow roots with `Info.plist`).
- **macOS bundle anatomy** (`bundles`): `layout` (`macos-deep`, `versioned-framework`, `shallow`), `info_plist_path`, `executable_candidates`, `signing_paths`, and path-convention roles (`app-extension`, `xpc-service`, `framework`, `system-extension`, `driver-extension`, `plug-in`, `resource-bundle`, `login-item`, `helper-application`, …), plus `privileged_helpers`, `launchd_plists`, `helpers`, `symlinks`.
- Framework hints from complete path segments (`React.framework`, `hermes.framework`, `Flutter.framework`, `UnityFramework.framework`) and path-based JavaScript/native **bridge hypotheses** (2 MiB candidate budget, `bridge_candidate_coverage`).

```sh
rea inspect-artifact /path/MyApp.ipa --format json > inventory.json
rea project-apple-application-graph '{"inventory_evidence":[<inventory_artifact Evidence>]}'
```

### Going deeper on Apple binaries

The projection does not read plists or signatures. Follow up with the native-static tools (see [[Native Binaries]]):

| Need | Tool (MCP / CLI) |
| --- | --- |
| `CFBundleExecutable`, identifiers, URL schemes, `SMPrivilegedExecutables` | `inspect_plist` / `rea inspect-plist` |
| Code-signing claims, debugger-attach facet | `inspect_signature` / `rea inspect-signature` |
| Which file each dylib load reaches (`@rpath`, `@loader_path`, `@executable_path`) | `trace_dylib_resolution` / `rea trace-dylib-resolution` |
| Mach-O slices and load commands | `inspect_macho` / `rea inspect-macho`, `list_architectures` / `rea list-architectures` |
| Nibs/storyboards, keyed archives, `Assets.car` | `decode_interface_builder`, `inspect_keyed_archive`, `inspect_asset_catalog` |
| ObjC/Swift dispatch metadata | `inspect_native_dispatch_metadata` |

```sh
rea trace-dylib-resolution MyApp.app --json
rea trace-dylib-resolution MyApp.app --root Contents/MacOS/MyApp --architecture arm64 --json
```

`trace_dylib_resolution` parses Mach-O/FAT load commands in TypeScript (works off macOS, nothing executed) and labels every edge `resolved`, `conditional`, `unresolved` or `undetermined`, with derived findings such as `required-load-unresolved` and `earlier-rpath-candidate-absent`. Absolute system install names are `outside-target` (dyld shared cache is not checked).

## HarmonyOS packages

No SDK, toolchain or engine required. Recognized roots: `.hap` (application module), `.hsp` (runtime shared package), `.app` (App Pack, verified ZIP magic only). `.har` is **not** classified by suffix because it also names HTTP Archive JSON.

```sh
rea inspect-artifact /path/to/Example.hap --format json > inspection.json
node --input-type=module -e 'import { readFileSync } from "node:fs"; console.log(JSON.stringify({ inventory_evidence: [JSON.parse(readFileSync("inspection.json", "utf8"))] }));' > projection-input.json
rea project-harmony-application-graph projection-input.json --format json
```

Reports component paths/IDs/hashes (manifests, Ark bytecode, resources, native libraries, JavaScript, signing), a packaging-model hint (Stage only from `module.json`, FA only from `config.json`, otherwise `unknown`), App Pack child paths (not recursed), runtime-family hints (`ark`, `native`, `javascript`) and path-based N-API bridge hypotheses to `libs/<abi>` libraries.

## Worked example

> [!example] Map a macOS app's helpers and broken rpaths
> 1. `rea inspect-artifact /Applications/Example.app --format json > inv.json`
> 2. Project with `project-apple-application-graph` → list `xpc-service`, `login-item` and `privileged_helpers`.
> 3. `rea inspect-plist` on each helper's `info_plist_path`; `rea inspect-signature` on its executable candidate.
> 4. `rea trace-dylib-resolution /Applications/Example.app --json` → look for `required-load-unresolved` and `conditional` edges.
> 5. Open the interesting executable in [[Hopper]] or [[Ghidra]].

## Limits and side effects

- Execution-free: no plist/CMS semantics in the projection, no observed runtime calls. Runtime families and bridges are **path hints**, not observed loading or bindings. Generic framework paths don't establish Swift vs. Objective-C.
- Partial inventories (e.g. a DMG with only its root identity) yield `partial` coverage: absence is unknown. APK/ASAR inventories are rejected as input errors.
- AppleDouble (`._*`, `__MACOSX/`) and components outside every app root are excluded from anatomy (counted in a limitation).
- HarmonyOS: path-presence only. Manifest semantics, Ark bytecode, resource resolution, signing verification and native boundaries are out of scope (#1431); encrypted entries make coverage partial; unauthenticated inventories fail closed.

> [!info] Verification
> `verify:macos-bundle` compares the dylib parser with `otool -l` and `DYLD_PRINT_LIBRARIES`. HarmonyOS: `npm run fixtures:harmony` then `REA_HARMONY_TEST_HAP=... npm run verify:harmony`. See [[Testing]].

## Related
[[Native Binaries]] · [[Android]] · [[Flutter Apps]] · [[JavaScript and Electron]] · [[Evidence Model]]

## Source
- https://github.com/morluto/rea/blob/main/docs/apple-application-analysis.md
- https://github.com/morluto/rea/blob/main/docs/harmony-analysis.md
- https://github.com/morluto/rea/blob/main/docs/native-investigation.md
