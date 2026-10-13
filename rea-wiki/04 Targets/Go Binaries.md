---
tags: [rea, target, native, go]
source:
  - docs/go-binaries.md
  - src/go/
---
# Go Binaries

`inspect_go_binary` / `rea inspect-go-binary` answers one question: **which Go compiler version, modules, replacements and build settings are embedded in this executable or library?** It is a built-in TypeScript reader — no Go installation, no active binary target, and no [[Hopper]], [[Ghidra]] or [[IDA]].

## What REA returns

The inline Evidence contains artifact path, SHA-256, byte length, container format, architecture, word size and byte order. A recognized `build_info` record contains:

- `go_version` (embedded UTF-8 text, not forced into semver; `null` if not UTF-8) plus `go_version_bytes_base64`.
- Exact file offsets and byte lengths for the header, compiler string and module string.
- `module_text` and `module_bytes_base64` (original bytes incl. Go framing).
- `module.path`, `module.main` (path, version, checksum, optional replacement), `module.dependencies` (original order).
- `module.settings` — ordered key/value entries, duplicates kept (e.g. `GOOS`, `GOARCH`, `vcs.revision` when present).
- `module.unparsed_lines`, `module.unparsed_line_bytes_base64`, `module.complete` — so malformed or unfamiliar records never silently vanish.

## Requirements and host support

| Item | Value |
| --- | --- |
| External tools | None |
| Containers | ELF, PE, thin Mach-O (inline or historical pointer build-info encoding) |
| Not supported | Fat Mach-O as a selected architecture; scanning arbitrary bytes for metadata |
| Host | Any host that runs REA |

## CLI and MCP

```bash
rea inspect-go-binary /absolute/path/to/application --json
```

```json
{
  "name": "inspect_go_binary",
  "arguments": { "path": "/artifacts/application" }
}
```

The CLI resolves relative paths against the working directory; MCP requires an absolute path.

## Worked example

> [!example] Supply-chain triage of a stripped Go service
> 1. `rea inspect-go-binary ./svc --json` → read `build_info.go_version` and `module.main.version`.
> 2. Walk `module.dependencies` for versions and replacements of interest; check `module.settings` for `vcs.revision` and `GOOS`/`GOARCH`.
> 3. If `module.complete` is false, read `module.unparsed_lines` before drawing conclusions.
> 4. For function-level questions, move to [[Native Binaries]] with a provider.

## Limits and side effects

- **Purely static.** The file is never modified; no temp files, no subprocesses, no network.
- `build_info: null` means no recognized record was found — **not** that the binary isn't Go (stripping, old builds, removal, obfuscation).
- Malformed supported containers → input-format failure; unsupported containers → separate unsupported-target failure. Symlinks and non-regular files are rejected.
- Bounds: 256 MiB file, 1 MiB aggregate embedded strings, 16 MiB structural tables, 8 MiB normalized report. Oversized results fail without partial success. MCP also applies its response budget (see [[MCP Server]]).
- Embedded metadata is evidence of file content only; it does not authenticate a compiler, dependency, commit or supply chain, and says nothing about runtime behavior.

> [!info] Verification
> `npm run verify:go:binary` compares REA's output with Go's own `debug/buildinfo` on compiler-built fixtures through CLI and MCP (requires Go; normal use does not). See [[Testing]].

## Related
[[Targets Overview]] · [[Native Binaries]] · [[ELF Layout and Recorded Crashes]] · [[Evidence Model]]

## Source
- https://github.com/morluto/rea/blob/main/docs/go-binaries.md
