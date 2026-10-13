---
tags: [rea, target, wasm]
source:
  - docs/wasm-artifacts.md
  - src/wasm/WabtArtifactProvider.ts
  - third_party/wabt/README.md
---
# WASM Artifacts

`inspect_wasm_artifact` / `rea inspect-wasm-artifact` inspects explicitly selected local WebAssembly bytes using caller-supplied **WABT 1.0.42**. It needs no active binary target, installs nothing, fetches nothing and never executes WASM.

> [!warning] Not an `open_binary` target
> A recognized WASM v1 header passed to `open_binary` returns `target_unavailable` (category `unsupported_target`, constraint `wasm_requires_static_inspection`) with a suggested valid `inspect_wasm_artifact` call.

## What REA returns

- Selected byte SHA-256 and size.
- Validation result (`wasm-validate --enable-all`).
- Section payload ranges and full header/detail output (`wasm-objdump -h -x`), including module-name preambles.
- Exact import/export WAT forms (escaped names preserved).
- Decoded WAT (`wasm2wat --enable-all`) and its UTF-8 SHA-256.
- Optional JavaScript glue analysis: `.wasm` string literals and literal templates with source coordinates, resolved to local path candidates or basename candidates.
- Tool profile: upstream release identity, resolved paths, executable digests, banners and arguments.

## Requirements

| Item | Value |
| --- | --- |
| Engine | WABT 1.0.42: `wasm-validate`, `wasm-objdump`, `wasm2wat` (`.exe` on Windows) |
| Configuration | `REA_WABT_BIN_DIRECTORY` = absolute directory holding those executables |
| Verified host | macOS arm64 (other hosts need their own verification) |

Check `binary_session` for configured capability availability; missing WABT is reported as an unavailable capability, not as missing native-engine support. See [[External Tools]].

## CLI and MCP

```bash
REA_WABT_BIN_DIRECTORY=/absolute/wabt/bin rea inspect-wasm-artifact \
  /artifacts/module.wasm --glue_paths /artifacts/glue.js \
  --candidate_paths /artifacts/other/module.wasm --format json
```

```json
{
  "name": "inspect_wasm_artifact",
  "arguments": {
    "path": "/artifacts/module.wasm",
    "glue_paths": ["/artifacts/glue.js"],
    "candidate_paths": ["/artifacts/other/module.wasm"]
  }
}
```

MCP records the Evidence for later `get_evidence_bundle` readback.

## Worked example

> [!example] Which module does this web app load?
> 1. Export scripts from a browser capture ([[Websites and Browser Observation]]) and locate the Emscripten/wasm-bindgen glue JS.
> 2. Run `inspect-wasm-artifact` with the `.wasm` as `path`, the glue as `--glue_paths`, and other local `.wasm` files as `--candidate_paths`.
> 3. Read the import list (what the host must provide) and export list (what JS calls), then the glue's literal associations.

## Limits and side effects

- Each file: regular file only, no symlinks, ≤ 8 MiB; bytes are copied read-only into a private workspace and re-hashed before/after every tool.
- Each subprocess: 30-second deadline and 32 MiB combined output; truncation fails.
- **WAT is a decoded representation**, not original source, round-trip byte identity, or proof of runtime behavior.
- Glue associations are literal string matches — not proven data flow or runtime loading. Computed strings and dynamic imports need further evidence.
- Caller-supplied tools are trusted; digest recording is not publisher authentication or an OS sandbox.

## Related
[[Targets Overview]] · [[JavaScript and Electron]] · [[External Tools]] · [[Testing]]

## Source
- https://github.com/morluto/rea/blob/main/docs/wasm-artifacts.md
- https://github.com/morluto/rea/blob/main/third_party/wabt/README.md
