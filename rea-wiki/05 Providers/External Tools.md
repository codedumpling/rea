---
tags: [rea, provider, external-tools]
source:
  - README.md
  - docs/binary-diagnostics.md
  - docs/recorded-crashes.md
  - docs/wasm-artifacts.md
  - docs/packet-captures.md
  - docs/signal-recordings.md
  - docs/firmware-analysis.md
  - docs/android-analysis.md
  - docs/apktool-resource-analysis.md
  - docs/adb-device-analysis.md
  - docs/web-network-captures.md
  - docs/javascript-recovery.md
  - docs/managed-code-analysis.md
  - docs/evm-bytecode.md
  - docs/process-capture.md
  - bridge/
  - third_party/
---
# External Tools

Beyond the deep providers ([[Hopper]], [[Ghidra]], [[IDA]]) and the auxiliary engines ([[JEB]], [[Frida]]), many REA capabilities wrap a **caller-supplied** third-party program. The pattern is the same for all of them:

- **You install the tool. REA never installs, upgrades or patches it**, and never changes user init files.
- You point REA at it, usually with an **absolute path** in an environment variable. Some tools fall back to `PATH`.
- REA runs the tool as an **owned process**: private config/cache directories, deadlines, bounded output, resource limits (often through util-linux `prlimit`) and verified cleanup. See `src/process/`.
- REA checks the version banner and records the exact executable path, digest and version in Evidence. Releases inside an accepted range that were not explicitly verified are noted as a limitation.
- A missing tool is reported as unavailable with remediation. It never causes a silent fallback.

Upstream source pins for several tools live as submodules under `third_party/` (`binwalk`, `unblob`, `wabt`, `wakaru`, `pwntools`, `pwndbg`, `jadx-headless-mcp`, `evmole`, `ghidra-nativeaot`, `web-network-captures`).

## Tool table

| Tool | Used for | Configure with | Version / host |
| --- | --- | --- | --- |
| **pwntools** (+ pyelftools, Unicorn) | `inspect_binary_layout`, `inspect_recorded_crash` | `REA_PWNTOOLS_PYTHON` (absolute Python with the packages installed) | pwntools 4.15.0, pyelftools 0.33, Unicorn 2.1.2; Linux x64 ELF64 x86-64 verified |
| **pwndbg** + GDB | Optional core-file context enrichment for recorded crashes | `REA_PWNDBG_GDB`, `REA_PWNDBG_GDBINIT`, `REA_PWNDBG_VENV_PATH` | GDB 13.1, pwndbg 2026.09.15; Linux x64 |
| **WABT** | `inspect-wasm-artifact` validation and disassembly | `REA_WABT_BIN_DIRECTORY` (directory of WABT executables) | WABT 1.0.42 |
| **TShark** | Recorded packet captures | `REA_TSHARK_COMMAND`, `REA_TSHARK_PRLIMIT_COMMAND` | TShark 4.x on **Linux** with `prlimit` |
| **sigrok-cli** | Recorded logic-signal decoding | `REA_SIGROK_COMMAND`, `REA_SIGROK_PRLIMIT_COMMAND` | sigrok-cli 0.7.2 + libsigrokdecode on **Linux** |
| **Binwalk** | Firmware region inspection | `REA_BINWALK_COMMAND`, `REA_FIRMWARE_PRLIMIT_COMMAND` | Binwalk 3.1.x (verified 3.1.0); Linux x64 |
| **Unblob** | Explicit firmware extraction | `REA_UNBLOB_COMMAND`, `REA_FIRMWARE_PRLIMIT_COMMAND` | Unblob 26.6.x (verified 26.6.4); needs format extractors on `PATH` |
| **JADX** (jadx-headless-mcp) | Static APK analysis | `REA_JADX_MCP_JAR`, optional `REA_JADX_HEAP_MIB`, `REA_JADX_ACTIVE_PROCESSOR_COUNT` | jadx-headless-mcp 0.7.1 + full JDK 17+ |
| **Apktool** | Android resource-table decoding | `REA_APKTOOL_COMMAND` | Apktool 2.7.0+ plus a Java runtime |
| **adb** | Android device/emulator acquisition | `REA_ADB_PATH` (otherwise `PATH`) | Android platform-tools (verified 34.0.5) |
| **mitmproxy** (`mitmdump`) | Native `.mitm` web network capture decoding | `REA_MITMDUMP_COMMAND` | mitmdump 12.2.3 on **Linux** |
| **Wakaru** | JavaScript bundle recovery | `REA_WAKARU_COMMAND`, `REA_JAVASCRIPT_PRLIMIT_COMMAND` | Wakaru ^1.13.0 (verified 1.14.0) on Linux x64 |
| **ilspycmd** | Optional managed reconstruction inference and a real-ILSpy verification oracle | `REA_ILSPY_CMD_PATH` | Any runnable `ilspycmd`; result stays non-canonical |
| **Chrome-family browser** | Browser observation over CDP; verification lanes | Attach to a user-owned endpoint; `REA_BROWSER_EXECUTABLE` for verifiers | Linux x64 / Chrome 153 verified for runtime lane |
| **prlimit** (EVM worker) | Resource limit for the bundled EVMole worker | `REA_EVM_PRLIMIT_COMMAND` | util-linux |
| **swiftc** + SDK | Compiling REA's macOS process-capture helper without an `xcrun` lookup | `REA_DARWIN_SWIFTC_PATH`, `SDKROOT` | macOS |

> [!info] Bundled vs. caller-supplied
> Some engines ship inside the package and need no configuration. EVMole 0.9.3 (offline EVM interface inference) runs in a bounded worker. Portable PE resource inspection (`inspect-pe-resources`) needs no external engine, Python or Windows SDK.

## pwntools in detail (offline binary diagnostics)

`inspect_binary_layout` / `rea inspect-binary-layout` reports section and segment layout, symbols, relocations, static linkage names and mitigation indicators (canary, PIE, NX, RELRO). It runs **without** an active Hopper, Ghidra or IDA target.

```sh
REA_PWNTOOLS_PYTHON=/absolute/isolated-env/bin/python \
  rea inspect-binary-layout ./selected.elf --json
```

- Python runs in isolated mode with an owned cache. The selected object is **never launched**. It is parsed from an owned stable snapshot.
- Limits: 32 MiB input, 64 MiB reply, 1 MiB diagnostics, a 30 s deadline, and Python soft limits of 3 GiB virtual memory, 30 CPU seconds and 64 MiB file output. If a limit is exceeded the call fails with no partial success.
- Mitigation results are **static inferences**. For example, an absent canary symbol does not prove that every function is unprotected.
- Large results may require a bigger MCP client receive buffer and `REA_MCP_MAX_RESPONSE_BYTES`. See [[MCP Server]].
- Over MCP, `"detail": "summary"` keeps the full Evidence in the session and returns only a summary. Use `inspect_analysis_view` afterwards for one section, symbol or facet.

The same Python environment powers [[ELF Layout and Recorded Crashes|recorded crash inspection]]. Bridge scripts live in `bridge/pwntools/` and `bridge/pwndbg/`.

## Troubleshooting pattern

1. Run the capability's readiness tool if it has one. Examples: `rea inspect-apktool-client`, `rea inspect-adb-client`, `inspect_jeb_client`. Otherwise run `rea doctor --json`.
2. Confirm that the configured path is **absolute** and points to an executable regular file. Directory paths are rejected.
3. Check the reported version against the table above.
4. On non-Linux hosts, remember that TShark, sigrok, mitmdump, Wakaru and the firmware tools are Linux-only lanes.

See also [[Firmware]], [[Android]], [[Network and Packet Captures]], [[Signal Recordings]], [[WASM Artifacts]], [[Environment Variables]].

## Source
- https://github.com/morluto/rea/blob/main/docs/binary-diagnostics.md
- https://github.com/morluto/rea/blob/main/README.md
- https://github.com/morluto/rea/blob/main/docs/firmware-analysis.md
- https://github.com/morluto/rea/blob/main/docs/packet-captures.md
