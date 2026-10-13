---
tags: [rea, provider, hopper]
source:
  - docs/installation.md
  - docs/native-investigation.md
  - src/hopper/HopperProvider.ts
  - src/hopper/HopperProviderCapabilities.ts
  - src/config/environment.ts
  - src/main/startup.ts
  - bridge/hopper_bridge.py
---
# Hopper

**Hopper Disassembler** is a separate commercial product with its own license. Its free demo has vendor-defined limits, and a paid license is optional. Provider ID: `hopper`. It is the only deep provider that `rea setup` can install, and only with your explicit approval. It is also the only provider with **GUI authority**: cursor navigation, bookmarks, and names and comments in a live document.

## What it gives you

The private bridge implements 36 operations (`HOPPER_OPERATIONS`), including:

- **Inventory and search:** `list_documents`, `list_segments`, `list_procedures`, `list_names`, `list_strings`, `search_procedures`, `search_strings`
- **Function analysis:** `procedure_info`, `procedure_assembly`, `procedure_pseudo_code`, `procedure_callers`, `procedure_callees`, `procedure_references`, `read_function_instructions`, `xrefs`, `analyze_function`
- **Memory and mapping:** `read_bytes`, `address_to_file_offset`, `resolve_containing_procedure`
- **GUI navigation:** `current_address`, `current_procedure`, `current_document`, `goto_address`, `next_address`, `prev_address`
- **Mutations:** `set_address_name`, `set_addresses_names`, `set_comment`, `set_inline_comment`, `set_bookmark`, `unset_bookmark`

Mutating operations change the Hopper document, not the executable. All Hopper operations declare `launchesProcess: true` and `mayShowUi: true`. See [[MCP Tool Catalog]].

## Platform support

- **Hosts:** macOS 12+, Ubuntu 24.04+, Fedora 41+, Nobara 44+, 64-bit Arch Linux, CachyOS, Omarchy. Any other platform (including Windows) reports `unsupported_host`.
- **Targets:** executables and Hopper databases (`.hop`). The adapter **refuses** MIPS and RISC-V ELF, headerless raw binaries and DOS MZ/COM. Use [[Ghidra]] for those.

## Install and discovery

| Host | What approved setup does |
| --- | --- |
| macOS | Downloads the official DMG, checks its size and digest, validates the bundle, and atomically installs it to `~/Applications/Hopper Disassembler.app`. It then opens Hopper so you can choose demo mode or activate a license. No Homebrew, no admin rights. |
| Linux | Verifies the official `.deb`, `.rpm` or Arch package, then calls the native package manager. The demo runs on a private **Xvfb** display, so your desktop is not needed. |

Unattended install requires `rea setup --yes --install-hopper` (plus scope flags). REA reuses any Hopper installation it detects and leaves Hopper in place when you uninstall REA.

**Launcher discovery:**
- macOS: `/Applications/Hopper Disassembler.app/Contents/MacOS/hopper`
- Linux: executable `/opt/hopper/bin/Hopper`, then `~/.local/share/rea/hopper/bin/Hopper`
- Override with `HOPPER_LAUNCHER_PATH`. It must point to a regular executable file, not an `.app` directory.

```bash
export HOPPER_LAUNCHER_PATH=/absolute/path/to/Hopper
rea doctor --provider hopper --json
```

## Configuration environment variables

| Variable | Meaning |
| --- | --- |
| `HOPPER_LAUNCHER_PATH` | Overrides the launcher path |
| `REA_HOPPER_STARTUP_TIMEOUT_MS` | Startup deadline (launch, initial analysis, bridge readiness). Default 120,000 ms; decimal integer 1 to 2,147,483,647 |
| `HOPPER_LOADER_ARGS_JSON` | JSON array of explicit Hopper loader arguments (overrides automatic loader selection) |
| `HOPPER_TARGET_PATH` / `HOPPER_TARGET_KIND` | Optional target the MCP server opens at startup; kind is `executable` (default) or `database` |
| `REA_ANALYSIS_PROVIDER=hopper` | Standing provider preference (see [[Providers Overview]]) |

## Lifecycle and ownership

- **One document per target and loader profile.** While a live REA session owns a target, a second session gets back the owning run ID instead of opening a duplicate document. Hopper's launcher cannot attach to an existing document, so REA never reuses or focuses a document you opened by hand.
- **Linux single application.** The Linux demo forwards any extra launch into its existing application. REA therefore reserves one Hopper application per user. A competing session receives the owner's session ID. Switch targets through that MCP session, or close it first.
- Requests are **bound to the active target's native document**, even if GUI focus changes. Use `open_binary` to switch targets.
- Hopper **serializes** requests. Successful decompilation is cached until a relevant rename or comment invalidates it.
- **FAT Mach-O:** REA reads the container header. For FAT64 it prepares a private thin image, checking SHA-256 while copying. Offsets still refer to the original container. An ambiguous subtype requires you to supply a thin image yourself; REA does not guess.
- **Closing** closes the bound document, shuts down the bridge and removes the socket directory. The Hopper app and unrelated documents stay open. If cleanup cannot be confirmed, the result is `cleanup_incomplete`. Retry `close_binary` on the same connection.

> [!info] Bridge internals
> `bridge/hopper_bridge.py` runs on Hopper's dedicated Python thread, launched through Hopper's `--python` option. The bootstrap injects a socket path and a random token, and every request is authenticated. All Hopper API calls stay on that one thread, because dispatching from a worker can deadlock Hopper.

## Troubleshooting

> [!warning] Pending dialogs
> If startup reports `Cannot launch Hopper! Connection is invalid.` or hangs waiting for the bridge, Hopper probably has a pending demo, license or loader dialog. Finish it in the same user session and retry. On macOS, complete Hopper's first-run UI before any unattended job.

- **macOS Apple Events denied:** allow the app that starts REA under *System Settings → Privacy & Security → Automation*.
- **Linux launcher won't start:** `ldd /absolute/path/to/Hopper | grep 'not found'`, install the missing libraries, then re-run the scoped doctor. Demo sessions need Xvfb, Python 3, X11 and XTEST.
- **Immutable `/tmp/.X11-unix`:** REA falls back to a user and mount namespace with a private tmpfs, without `sudo`. Doctor reports the chosen strategy.
- **Interrupted lease recovery:** stop REA sessions before deleting the reservation directory that the diagnostic names.
- **CI:** run `rea doctor --provider hopper --json` on the failing runner. Never copy license secrets into logs or kill unrelated Hopper processes.

## Verification

`npm run verify:hopper` (macOS), `npm run verify:hopper:linux` (Linux demo runner), plus `verify:hopper:deadlines`, `verify:hopper:fat`, `verify:hopper:fat64`. See [[Testing]] and [[CI Workflows]].

See also [[Native Binaries]], [[Troubleshooting]], [[Environment Variables]].

## Source
- https://github.com/morluto/rea/blob/main/docs/installation.md#hopper
- https://github.com/morluto/rea/blob/main/docs/native-investigation.md
- https://github.com/morluto/rea/blob/main/src/hopper/HopperProviderCapabilities.ts
