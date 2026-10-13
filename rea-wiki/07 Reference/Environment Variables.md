---
tags: [rea, reference, configuration]
source:
  - src/config/environment.ts
  - src/config/parseConfig.ts
  - src/config/mcpInputSchemaProfile.ts
  - src/config/mcpResponseBudget.ts
  - src/config/providerStartupTimeout.ts
  - src/config/homeDirectory.ts
  - docs/installation.md
  - docs/mcp-contracts.md
  - docs/cli.md
---
# Environment Variables

REA is configured almost entirely through environment variables. Core settings are parsed and validated with Zod in `src/config/environment.ts` at the composition root; tool-specific variables are read by the adapter that uses them.

> [!warning] Set them where REA runs
> Variables must be present in the environment **of the process that starts REA**. For MCP that is the agent's server entry (`env` object in the client config), not your terminal. Changing a shell variable does not reconfigure a running server or JVM; restart the agent. MCP budget/profile settings are captured at server startup.

> [!info] Validation
> Invalid values fail configuration validation with a `ConfigurationError` naming the setting and constraint (never the rejected value). Setup and doctor report a malformed REA `env` map before writing files. Many path variables must be **absolute**.

## Core configuration (`src/config/environment.ts`)

| Variable | Purpose | Default / constraint |
| --- | --- | --- |
| `REA_ANALYSIS_PROVIDER` | Standing deep-provider preference: `auto` or a provider ID such as `hopper`, `ghidra`, `ida` (any lowercase ID; validated against available providers at selection). CLI `--provider` and MCP `open_binary.provider_id` override it. | `auto` |
| `REA_MCP_MAX_RESPONSE_BYTES` | Raise the MCP response budget for clients with a larger receive buffer (set both sides to the same value). 1024 bytes reserved for the envelope. | Safe decimal integer >= 10485760 (10 MiB) |
| `REA_MCP_INPUT_SCHEMA_PROFILE` | `full` or `compact`. Compact advertises each tool's input schema within 13,312 bytes (no nested descriptions/examples) for size-capped clients (Moonshot/Kimi, Copilot `gpt-4.1`, Claude Code without tool search). Server-side validation is unchanged. | `full` |
| `REA_IDA_MCP_CONFIG` | Absolute path to the upstream IDA MCP JSON registration. See [[IDA]]. | absolute |
| `GHIDRA_INSTALL_DIR` | Existing Ghidra 12.1.x installation. See [[Ghidra]]. | absolute |
| `JAVA_HOME` | JDK for Ghidra (optional if `java`/`javac` on PATH). | absolute |
| `REA_GHIDRA_STARTUP_TIMEOUT_MS` | Deadline for Ghidra import, analysis, bridge and health readiness. | 330000; integer 1..2147483647 |
| `REA_HOPPER_STARTUP_TIMEOUT_MS` | Deadline for Hopper launch, initial analysis and bridge readiness. | 120000; integer 1..2147483647 |
| `REA_GHIDRA_LANGUAGE_ID` | Force Ghidra import language (e.g. `x86:LE:32:default`); passed as `-processor`, checked against `.ldefs`. DOS imports reject overrides. | optional |
| `REA_GHIDRA_COMPILER_SPEC_ID` | Compiler spec (e.g. `borlandcpp`), passed as `-cspec`. | Requires `REA_GHIDRA_LANGUAGE_ID` |
| `REA_GHIDRA_DISABLED_ANALYZERS` | Comma-separated exact Ghidra analyzer names to disable before auto-analysis (recorded in `analyzer_preset`). | <=32 names, each <=120 chars |
| `REA_GHIDRA_SEED_FILE` | TSV of `address<TAB>kind[<TAB>name]` seeds (`function`, `code`, `label`) applied before auto-analysis. | absolute |
| `REA_GHIDRA_NATIVEAOT_JAR` | Locally built NativeAOT Ghidra extension JAR. See [[Ghidra Variants]]. | absolute |
| `REA_ILSPY_CMD_PATH` | Absolute `ilspycmd` used as a non-canonical managed-code reconstruction oracle. See [[DotNET Assemblies]]. | absolute |
| `HOPPER_LAUNCHER_PATH` | Override the Hopper launcher. Defaults: macOS `/Applications/Hopper Disassembler.app/Contents/MacOS/hopper`; Linux `/opt/hopper/bin/Hopper`, then `~/.local/share/rea/hopper/bin/Hopper`. | optional |
| `HOPPER_TARGET_PATH` | Parsed into config (`hopperTargetPath`); not described in user docs. | optional |
| `HOPPER_TARGET_KIND` | `executable` or `database`. | `executable` |
| `HOPPER_LOADER_ARGS_JSON` | JSON array of explicit Hopper loader arguments; overrides derived loader selection. | optional |
| `REA_LOG_LEVEL` | `trace`, `debug`, `info`, `warn`, `error`, `fatal`, `silent`. | `info` |
| `REA_REFERENCE_SECRET_PATTERNS_JSON` | JSON array of ignore patterns marking paths as sensitive during `rea import-reference-source` (reported as configured-secret exclusions). | `[]` |

## Ghidra JVM controls (consumed by REA's Ghidra launch)

| Variable | Purpose |
| --- | --- |
| `GHIDRA_HEADLESS_MAXMEM` | JVM `-Xmx` for headless Ghidra (first nonempty of this, then `GHIDRA_MAXMEM`, then `2G`) |
| `GHIDRA_MAXMEM` | Fallback heap setting |

REA **clears** inherited `_JAVA_OPTIONS`, `JAVA_TOOL_OPTIONS`, `JDK_JAVA_OPTIONS` and `GHIDRA_JAVA_OPTIONS`, and owns `GHIDRA_HEADLESS_JAVA_OPTIONS`; flags injected there are unsupported.

## Target-specific tools (read by their adapters)

| Variable | Purpose | Read in | Guide |
| --- | --- | --- | --- |
| `REA_JEB_MCP_URL` | JEB MCP endpoint; loopback `http(s)` only. Default `http://127.0.0.1:8425/mcp`. | `src/composition/jeb.ts` | [[JEB]] |
| `REA_JADX_MCP_JAR` | Path to `jadx-headless-mcp` JAR for APK analysis | `src/android/JadxConfiguration.ts` | [[Android]] |
| `REA_JADX_HEAP_MIB` | Explicit JADX JVM heap (MiB) | `src/android/JadxConfiguration.ts` | [[Android]] |
| `REA_JADX_ACTIVE_PROCESSOR_COUNT` | Explicit JVM active processor count | `src/android/JadxConfiguration.ts` | [[Android]] |
| `REA_ADB_PATH` | Absolute `adb`; otherwise resolved from PATH | `src/adb/AdbConfiguration.ts` | [[Android]] |
| `REA_APKTOOL_COMMAND` | Absolute Apktool launcher; otherwise `apktool` on PATH | `src/apktool/ApktoolConfiguration.ts` | [[Android]] |
| `REA_PWNTOOLS_PYTHON` | Python with pwntools 4.15.0 / pyelftools 0.33 / Unicorn 2.1.2 | `src/native/pwntools/PwntoolsDecoder.ts` | [[ELF Layout and Recorded Crashes]] |
| `REA_PWNDBG_GDB`, `REA_PWNDBG_GDBINIT`, `REA_PWNDBG_VENV_PATH` | Optional GDB/pwndbg mapping context for recorded crashes | `src/native/pwndbg/PwndbgCoreContext.ts` | [[ELF Layout and Recorded Crashes]] |
| `REA_WABT_BIN_DIRECTORY` | Directory with `wasm-validate`, `wasm-objdump`, `wasm2wat` (WABT 1.0.42) | `src/application/CapabilityInventory.ts` | [[WASM Artifacts]] |
| `REA_EVM_PRLIMIT_COMMAND` | Alternate absolute `prlimit` for EVM analysis | `src/evm/EvmoleInterfaceProvider.ts` | [[EVM Bytecode]] |
| `REA_WAKARU_COMMAND` | Absolute Wakaru executable for JavaScript source recovery | `src/javascript/recovery/WakaruCommand.ts` | [[JavaScript and Electron]] |
| `REA_JAVASCRIPT_PRLIMIT_COMMAND` | Alternate absolute `prlimit` for Wakaru (default `/usr/bin/prlimit`) | `src/javascript/recovery/WakaruCommand.ts` | [[JavaScript and Electron]] |
| `REA_BROWSER_EXECUTABLE` | Absolute Chromium-family browser for native module resolution (and verification lanes) | `src/browser/modules/NativeModuleResolver.ts` | [[Websites and Browser Observation]] |
| `REA_BROWSER_NO_SANDBOX` | `true` passes Chromium's no-sandbox launch option when the host requires it | `src/browser/modules/NativeModuleResolver.ts` | [[Websites and Browser Observation]] |
| `REA_MITMDUMP_COMMAND` | Absolute `mitmdump` for native mitmproxy capture files (Linux) | `src/browser/history/MitmproxyCaptureAdapter.ts` | [[Network and Packet Captures]] |
| `REA_TSHARK_COMMAND`, `REA_TSHARK_PRLIMIT_COMMAND` | TShark 4.x and its `prlimit` (Linux) | `src/captures/TsharkCaptureProvider.ts` | [[Network and Packet Captures]] |
| `REA_SIGROK_COMMAND`, `REA_SIGROK_PRLIMIT_COMMAND` | sigrok-cli 0.7.2 and its `prlimit` (Linux) | `src/signals/SigrokRecordingProvider.ts` | [[Signal Recordings]] |
| `REA_BINWALK_COMMAND`, `REA_UNBLOB_COMMAND`, `REA_FIRMWARE_PRLIMIT_COMMAND` | Binwalk 3.1.x, Unblob 26.6.x, and their `prlimit` | `src/firmware/FirmwareCommand.ts` | [[Firmware]] |
| `REA_DARWIN_SWIFTC_PATH` | Absolute `swiftc` for compiling REA's macOS process helper without `xcrun` lookup (pair with `SDKROOT`) | `src/process/DarwinProcessRunTokenReader.ts` | [[Process Capture]] |

## Client/host variables REA honours
- `NO_COLOR=1` disables colour in setup prompts.
- `HOME` / `USERPROFILE` (and `HOMEDRIVE`+`HOMEPATH` on Windows) select the home directory (`src/config/homeDirectory.ts`).
- Client config roots such as `CODEX_HOME`, `CLAUDE_CONFIG_DIR`, `GEMINI_CLI_HOME`, `QWEN_HOME`, `PI_CODING_AGENT_DIR`, `HERMES_HOME`, `QODER_CONFIG_DIR`, `OPENCODE_CONFIG_DIR`, `GROK_HOME`, `DSH_HOME` are followed by setup/doctor/uninstall. See [[Agent Setup]].
- Bun update paths: `BUN_INSTALL_GLOBAL_DIR`, `BUN_INSTALL`, `XDG_CACHE_HOME`.

## Development and verification variables
Many `REA_VERIFY_*`, `REA_AGENT_EVAL_*`, `REA_CLANG`, `REA_LLD_LINK`, `REA_CODEX_CLI`, `REA_WEB_SOURCE_MAP_COMPILER` etc. variables configure optional verification lanes only (e.g. `REA_VERIFY_CLAUDE_COMMAND`, `REA_VERIFY_RUNTIME_ROOT`, `REA_AGENT_EVAL_SCENARIOS`). They are documented in `docs/testing.md` and `docs/client-verification.md`; see [[Testing]]. Internal variables such as `REA_PROCESS_RUN_ID` are set by REA itself for owned child processes and are not user settings.

## Example: MCP entry with environment
```json
{
  "mcpServers": {
    "rea": {
      "command": "npx",
      "args": ["-y", "rea-agents@6.4.0", "mcp"],
      "env": {
        "GHIDRA_INSTALL_DIR": "/absolute/path/to/ghidra_12.1.4_PUBLIC",
        "REA_ANALYSIS_PROVIDER": "ghidra",
        "REA_GHIDRA_STARTUP_TIMEOUT_MS": "900000",
        "REA_MCP_INPUT_SCHEMA_PROFILE": "compact"
      }
    }
  }
}
```

## Source
- https://github.com/morluto/rea/blob/main/src/config/environment.ts
- https://github.com/morluto/rea/blob/main/src/config/parseConfig.ts
- https://github.com/morluto/rea/blob/main/docs/installation.md
- https://github.com/morluto/rea/blob/main/docs/mcp-contracts.md
