---
tags: [rea, target, native, elf, crash]
source:
  - docs/binary-diagnostics.md
  - docs/recorded-crashes.md
  - third_party/pwntools/README.md
---
# ELF Layout and Recorded Crashes

Three target-free "offline binary diagnostics" tools inspect an explicit file without opening a [[Hopper]], [[Ghidra]] or [[IDA]] session:

| Tool (MCP / CLI) | Target | Engine |
| --- | --- | --- |
| `inspect_binary_layout` / `rea inspect-binary-layout` | ELF64 x86-64 EXEC/DYN/REL | Caller-supplied pwntools |
| `inspect_recorded_crash` / `rea inspect-recorded-crash` | Linux ELF64 x86-64 core file | Caller-supplied pwntools; optional GDB + pwndbg |
| `inspect_pe_resources` / `rea inspect-pe-resources` | PE32 / PE32+ resource tree | Built in (no Python, no Windows SDK) |

CTF work is one use; crash triage, compatibility checks and ordinary binary inspection use the same primitives.

## Bring your own engine (pwntools)

Initial real verification covers **Linux x64**. Point `REA_PWNTOOLS_PYTHON` at an absolute Python whose environment already contains unchanged **pwntools 4.15.0, pyelftools 0.33 and Unicorn 2.1.2**. REA never installs Python, packages or GDB, never edits init files, and never launches the selected object. See [[External Tools]].

## ELF layout

```sh
REA_PWNTOOLS_PYTHON=/absolute/isolated-env/bin/python \
  rea inspect-binary-layout ./selected.elf --json
```

```json
{ "name": "inspect_binary_layout", "arguments": { "path": "/artifacts/selected.elf" } }
```

Returns sections, segments, original symbols and relocations (with table + entry index), dynamic dependencies (`DT_NEEDED`), interpreter (`PT_INTERP`), GOT/PLT convenience maps, and static mitigation indicators (canary, PIE, NX, executable stack, RELRO).

> [!tip] Large results over MCP
> Pass `"detail": "summary"` to retain the complete layout Evidence in the session and get only a summary. Then page sections, symbols, mitigations or linkage with `inspect_analysis_view` using `normalized_result.parent_evidence_id`. On the CLI, pass saved Evidence JSON inline to `rea inspect-analysis-view`.

Interpretation rules worth remembering:
- Addresses/offsets are hex strings; a linked address is **not** a runtime address (runtime load base stays `null`).
- `DT_NEEDED`/`PT_INTERP` are reported names, not resolved paths. PLT inference may emulate instructions in Unicorn — outputs are static candidates.
- Mitigations are static inferences; an absent canary symbol does not prove every function is unprotected.

## Recorded crashes (core files)

```sh
export REA_PWNTOOLS_PYTHON=/absolute/existing/venv/bin/python
rea inspect-recorded-crash ./crash.core --json
```

Returns original program headers and recorded segments, every note (including unfamiliar owners), every `PRSTATUS` note as a separate thread with historical PID, signal and all 27 lossless uint64 registers, and signed `SIGINFO` (only SIGSEGV codes 1/2 establish the fault address initially).

Optional debugger mapping context:

```sh
export REA_PWNDBG_GDB=/absolute/existing/gdb
export REA_PWNDBG_GDBINIT=/absolute/unchanged/pwndbg/gdbinit.py
export REA_PWNDBG_VENV_PATH=/absolute/existing/pwndbg-venv
rea inspect-recorded-crash ./crash.core --debugger-context --json
```

MCP: `{ "path": "/artifacts/crash.core", "include_debugger_context": true }`. GDB starts without initfiles, auto-load, debuginfod or shared-library loading; no GDB command language is exposed. Evidence with maps is `derived`; without, `observed`.

> [!warning] Historical PIDs
> PIDs in a core are metadata only. They never select a live process or authorize attach.

## PE resources

```sh
rea inspect-pe-resources ./selected.exe --json
```

Returns the type/name/language resource tree with original offsets, per-payload SHA-256, and `RT_GROUP_ICON` → `RT_ICON` reference resolution. Options: `--max-file-bytes` (default 64 MiB, max 512 MiB) and `--max-entries` (default 16384, max 65536). No image decoding or resource loading.

## Worked example

> [!example] Triage a segfault in a CTF binary
> ```sh
> export REA_PWNTOOLS_PYTHON=/opt/pwn/bin/python
> rea inspect-binary-layout ./chall --json        # PIE? NX? RELRO? canary?
> rea inspect-recorded-crash ./core.1234 --json   # faulting thread, rip/rsp, SIGSEGV addr
> ```
> Then open `./chall` in [[Ghidra]] and inspect the function containing the faulting address ([[Native Binaries]]).

## Limits and side effects

- Fully offline: the selected file is read from an owned stable snapshot; originals are unchanged.
- Layout limits: 32 MiB input, 64 MiB reply, 1 MiB diagnostics, 30-second deadline; Python soft limits ≤ 3 GiB address space, 30 CPU s, 64 MiB file output. Large replies may need a larger MCP client `maxBufferSize` and `REA_MCP_MAX_RESPONSE_BYTES` (see [[Environment Variables]]).
- Crash limits: 32 MiB input, 64 MiB decoder reply, 1 MiB diagnostics per stage, 30 s per stage. These are not a sandbox for caller-supplied engine code.
- Core files from other ABIs return `unsupported_target`. Missing notes don't prove absence at the crash.
- GDB session control, pwndbg enrichment beyond maps, and other OS/architectures are not yet claimed (tracked by #969).

## Related
[[Targets Overview]] · [[Native Binaries]] · [[Go Binaries]] · [[External Tools]] · [[Evidence Model]]

## Source
- https://github.com/morluto/rea/blob/main/docs/binary-diagnostics.md
- https://github.com/morluto/rea/blob/main/docs/recorded-crashes.md
