---
tags: [rea, target, evm]
source:
  - docs/evm-bytecode.md
  - src/evm/
  - third_party/evmole/README.md
---
# EVM Bytecode

`inspect_evm_interface` / `rea inspect-evm-interface` infers interface candidates (function selectors, argument types, state mutability) from one local EVM bytecode file. It runs fully offline using the bundled, unchanged **EVMole 0.9.3** and needs no active native target. Contract investigation, compatibility checks, audits and CTFs all use the same primitive.

## What REA returns

- **Carrier SHA-256** (exact original file, including `0x` prefix, casing, whitespace) and **decoded-byte SHA-256** (the actual EVM bytes). Neither is the Ethereum Keccak code hash.
- Full bytecode, upstream function byte offsets (into the decoded bytecode) and the producer's raw representation, inline by default.
- Per-function selector, dispatch info, argument string and state mutability — all **inferences**.
- Upstream terminal metadata in `raw_result`.

## Requirements and host support

| Item | Value |
| --- | --- |
| Verified host | Linux x64, flat legacy bytecode |
| Engine | Bundled EVMole 0.9.3 (no install) |
| System tool | util-linux `prlimit` (normally `/usr/bin/prlimit`); override with `REA_EVM_PRLIMIT_COMMAND` |
| Unsupported | EOF-style `EF00` containers → `unsupported_target` |

## CLI and MCP

The second positional argument is the required encoding (`hex` or `raw`):

```sh
rea inspect-evm-interface ./runtime.hex hex --json
rea inspect-evm-interface ./runtime.bin raw --json
```

```json
{
  "name": "inspect_evm_interface",
  "arguments": { "path": "/artifacts/runtime.hex", "encoding": "hex" }
}
```

Hex may have an optional `0x` prefix and outer ASCII whitespace; odd length, interior separators, BOM and invalid UTF-8 are rejected. Empty bytecode is valid with unknown coverage.

### Summary delivery and views (MCP)

On a retaining server, add `"detail": "summary"` to get every inferred function with identity and uncertainty but without bytecode text or raw stdout. Page the complete parent with `inspect_analysis_view` (source kind `retained-evidence`, view kind `evm`): page `functions` by row order, or `bytecode_hex`, `stdout`, `stderr` by UTF-16 code units. Portable historical Evidence can be viewed with source kind `inline`. See [[MCP Tool Catalog]].

## Worked example

> [!example] Identify a deployed contract's interface from saved runtime bytecode
> ```sh
> rea inspect-evm-interface ./runtime.hex hex --json > evm.json
> ```
> Read the selectors and mutability candidates, then look up signatures yourself — REA does **not** look up function names/signatures, and recovered types are not a verified source ABI.

## Limits and side effects

- **No network or chain access**: no RPC/explorer lookup, wallet action, transaction broadcast or target execution.
- Empty candidate list does not prove there is no callable interface; optimized code may yield different mutability candidates; runtime/init-code identity, deployment authenticity and hardfork remain unknown.
- Bounds: 4 MiB input, 16 MiB reply, 1 MiB diagnostics; owned worker with 256 MiB JS heap, 3 GiB virtual address space, 30 CPU s, 30-second deadline. Limits fail without partial success.
- Source/metadata, proxy resolution, storage/CFG and fork execution are separate future increments (#972).

## Related
[[Targets Overview]] · [[External Tools]] · [[Evidence Model]] · [[Sessions and Snapshots]]

## Source
- https://github.com/morluto/rea/blob/main/docs/evm-bytecode.md
- https://github.com/morluto/rea/blob/main/third_party/evmole/README.md
