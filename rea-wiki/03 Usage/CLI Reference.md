---
tags: [rea, usage, cli]
source:
  - docs/cli.md
  - src/cliCommandNames.ts
  - src/cli/
  - src/cliEvidenceCommands.ts
  - src/cliOutput.ts
  - scripts/rea.mjs
  - src/cli/setupCommands.ts
  - src/cli/options.ts
---
# CLI Reference

The `rea` command runs the **same application workflows and Evidence contracts** as the [[MCP Server]], one process per invocation. Use it for a quick direct inspection, a script, or when no agent is connected. Because every invocation is a separate process, nothing is retained between calls. Save JSON output to a file if a later command needs it (see [[Sessions and Snapshots]] and [[Evidence Export and Import]]).

> [!info] Install or run ad hoc
> `npm install --global rea-agents` installs `rea`. Without a global install, prefix any command with `npx -y rea-agents@latest` (or `bunx rea-agents@latest`). See [[Installation]] and [[Quick Start]].

## How the dispatcher works

`scripts/rea.mjs` is the `bin` entry point. It routes on the argument vector:

| Invocation | What runs |
| --- | --- |
| `rea mcp` or `rea --mcp` (sole argument) | The long-lived stdio [[MCP Server]] (`dist/mcp/main.js`) |
| `rea mcp doctor [--json\|--format F]` | Starts the MCP server and checks that startup and `tools/list` work ("Validate MCP server startup and tool listing") |
| anything else | The one-shot CLI built on the Incur framework (`dist/cli-runtime/cli.js`) |

If the compiled runtime is missing (for example, in a source checkout that hasn't been built yet), the dispatcher prints a hint to run `npm ci && npm run build:cached` and exits `1`.

## Global output options

| Flag | Effect |
| --- | --- |
| *(default)* | TOON terminal format |
| `--json` | JSON. Indented on a terminal, **compact when piped or redirected** |
| `--format toon\|json\|yaml\|md\|jsonl` | Pick a format explicitly |
| `--filter-output EXPR` | Project part of the result (Incur filter) |
| `--full-output` | Show the full output envelope |
| `--token-count`, `--token-limit N`, `--token-offset N` | Token-window views. **Can't be combined with structured formats.** Use `--filter-output` or the command's own pagination instead |
| `--help` | Per-command arguments and options |

> [!tip] Save big results, then project them
> Real application Evidence can run to hundreds of MB (the docs cite a 334 MB Obsidian result that reduces to a ~10 KB summary view). Redirect to a file, then use `inspect-analysis-view`:
> ```bash
> rea analyze-javascript-application /absolute/path/to/app --json > app-evidence.json
> jq -c '{source: {kind: "inline", evidence: .}, view: {kind: "summary"}}' app-evidence.json > app-view.json
> rea inspect-analysis-view app-view.json
> ```

## Provider selection

Native ("deep") commands accept `--provider ID|auto`. Provider IDs are lowercase (`hopper`, `ghidra`, `ida`, …). Selection works like this:

1. An explicit `--provider` wins.
2. Otherwise `REA_ANALYSIS_PROVIDER` gives a standing preference (see [[Environment Variables]]).
3. Otherwise `auto` picks the **single** available provider that supports the target. If several qualify, you get an `ambiguous` error listing `details.candidate_ids`.

```bash
rea providers --json        # binary-session providers and operations
rea capabilities --json     # auxiliary capabilities
rea analyze /absolute/path/to/program --provider hopper
rea doctor --provider ghidra --json   # diagnose provider_unavailable
```

For headerless images, use `--target-format dos-com|raw-binary`. `raw-binary` also requires `--architecture` (`x86`, `x86_64`, `arm`, `arm64`) and `--entry-point 0xHEX`. See [[Providers Overview]], [[Ghidra]], [[Hopper]] and [[IDA]].

## Commands by area

The canonical list is `CLI_COMMANDS` in `src/cliCommandNames.ts`. The only alias is `compare-bundles` (for `compare`).

### Setup, readiness and maintenance
| Command | Purpose / key flags |
| --- | --- |
| `setup` | Configure agent MCP registrations and the bundled skill. Flags: `--client ID` (repeatable), `--all-detected`, `--skill-client ID`, `--skill`/`--skill=false`, `--dry-run`, `--yes`/`-y`, `--install-hopper`, `--accessible` |
| `doctor` | Readiness check. Flags: `--target PATH`, `--client ID`, `--provider hopper\|ghidra\|ida`, `--skill` |
| `update` | Update the installation and verify the installed release |
| `uninstall` | Remove REA-owned agent config and skill files. `--purge-data` also removes caches and state |
| `providers`, `capabilities` | Discovery (see above) |

See [[Agent Setup]], [[Readiness Checks]] and [[Updating and Uninstalling]].

### Native binaries (deep provider)
`analyze` / `inspect` (native overview), `function` (function dossier), `decompile` (pseudocode), `instructions` (raw instructions), `search` (`--kind procedures` for names), `xrefs`, `trace` (literal feature trace), `read-bytes` (`--length`), `address-to-file-offset`, `trace-native-values`, `inspect-native-instruction`, `resolve-native-call-targets`, `inspect-native-data-type` (`--address`), `inspect-native-load-image`, `inspect-native-api`, `inspect-native-dispatch-metadata`, `annotate-native-function` (`--name`, `--comment`), `trace-native-ui-action`.

All of these accept `--snapshot PATH` to reuse cached results ([[Sessions and Snapshots]]). See [[Native Binaries]].

### Apple / Mach-O
`inspect-macho`, `inspect-signature`, `list-architectures`, `inspect-plist`, `demangle-swift`, `trace-dylib-resolution` (`--root`, `--architecture`), `decode-interface-builder`, `inspect-keyed-archive`, `inspect-asset-catalog`, `observe-native-ui`, `observe-native-calls`, `capture-native-ui-scenario`, `project-apple-application-graph`. See [[Apple and Harmony Apps]].

### Artifacts, packages and offline formats
`inspect-artifact`, `extract-artifact` (`--integrity-policy record-and-continue`), `inspect-binary-layout`, `inspect-pe-resources`, `inspect-recorded-crash` (`--debugger-context`), `inspect-go-binary`, `inspect-wasm-artifact`, `inspect-evm-interface PATH raw|hex`, `inspect-sqlite-database` (`--table`, `--column`, `--row-limit`), `inspect-packet-capture` (`--fields`, `--display_filter`), `decode-signal-recording`, `inspect-firmware-regions`, `extract-firmware`, `inspect-analysis-view`, `inspect-evidence-bundle`. See [[Targets Overview]].

### .NET managed code
`inspect-managed-artifact`, `inspect-managed-members` (`--type` repeatable), `inspect-managed-native-boundaries`, `compare-managed-members`, `verify-managed-native-boundaries`, `import-managed-reconstruction`, `project-managed-application-graph`. See [[DotNET Assemblies]].

### Android, Flutter, JEB, adb
- Static APK (JADX): `inspect-android-package`, `search-android-classes`, `inspect-android-class`, `inspect-android-method` (`--overload-index`), `trace-android-references` (`--method-name`), `project-android-application-graph`
- apktool: `inspect-apktool-client`, `decode-android-resources` (`--locale`, `--no-strings`)
- Flutter: `identify-flutter-build`, `inspect-dart-aot` (`--abi`)
- JEB: `inspect-jeb-client`, `open-jeb-project`, `list-jeb-units`, `decompile-jeb-item`
- adb device: `inspect-adb-client`, `list-adb-devices`, `inspect-adb-device`, `list-adb-packages` (`--scope third_party`), `resolve-adb-packages`, `inspect-adb-package`, `pull-adb-package`, `pull-adb-file`, `push-adb-file`, `read-adb-logcat` (`--count`, `--pid`), `capture-adb-screen`, `list-adb-processes`, `list-adb-directory`, `list-adb-features`, `list-adb-services`, `inspect-adb-display`, `inspect-adb-window`, `read-adb-setting`, `collect-adb-bugreport`, `install-adb-package` (`--replace`), `uninstall-adb-package`, `start-adb-app`, `start-adb-activity` (`--data-uri`), `stop-adb-app`

Output-writing adb commands take `--output-directory`. See [[Android]], [[Flutter Apps]] and [[JEB]].

### JavaScript, Electron and application graphs
`analyze-javascript-application` (`--integrity-policy`, `--source-path` repeatable, `--artifact-format`), `recover-javascript-sources`, `trace-application-feature`, `trace-javascript-semantics`, `compare-application-versions`, `compare-source-to-bundle`, `compare-javascript-export-shapes`, `import-reference-source`, `reconcile-javascript-runtime`, `list-electron-targets`, `inspect-electron-page`, `capture-electron-scenario`, `list-javascript-runtime-targets`, `observe-javascript-runtime` (`--runtime-kind node`), `project-harmony-application-graph`.

Generic `rea analyze PATH` picks the JavaScript workflow for directories and `.asar` files when neither `--provider` nor `--snapshot` is given. See [[JavaScript and Electron]] and [[JavaScript Runtime Observation]].

### Browser and web
`list-browser-targets`, `inspect-web-page` (`--observation-ms`, `--include-*` capture switches), `analyze-web-bundle`, `observe-web-session`, `discover-webmcp-tools`, `compare-web-captures`, `capture-web-screenshot`, `compare-web-screenshots`, `capture-browser-scenario`, `export-web-scripts`, `observe-web-execution`, `inspect-web-event-listeners`, `trace-web-source-location`, `trace-web-module-imports`, `inspect-web-network-capture PATH har|mitmproxy|saz|qlog` (`--record N`). See [[Websites and Browser Observation]] and [[Network and Packet Captures]].

### Process capture and Frida
`capture-process`, `compare-process-captures`, `list-frida-devices`, `list-frida-processes`, `instrument-with-frida attach|spawn TARGET` (`--script`/`--script-file`, `--duration-ms` up to 60000, `--device-id` or remote options). See [[Process Capture]] and [[Frida]].

### Evidence and reconstruction
`evidence-import`, `evidence-export` (`--overwrite`), `compare` (alias `compare-bundles`), `build-reconstruction-obligation-ledger`, `evaluate-reconstruction-coverage`. See [[Evidence Export and Import]].

> [!info] MCP-only operations
> Some MCP tools have no dedicated CLI command, for example `open_binary`, `close_binary`, `binary_session`, `get_call_graph`, `batch_decompile`, `list_unknowns`/`record_unknown`, `compare_functions` and the Frida session tools. They are listed in `MCP_TOOLS_WITHOUT_DEDICATED_CLI`. Some CLI names also map to differently named tools: `analyze`/`inspect` map to `binary_overview`, `decompile` to `procedure_pseudo_code`, `function` to `analyze_function`, `trace` to `trace_feature`, `search` to `search_strings` + `search_procedures`, and `capture-process` to `capture_process_scenario`. See [[MCP Tool Catalog]].

### JSON-input commands
Workflow commands such as `trace-application-feature`, `compare-application-versions`, `capture-process`, `capture-browser-scenario`, `inspect-evidence-bundle` and `build-reconstruction-obligation-ledger` take **inline JSON or a path to a JSON file**. File inputs must be regular files (symlinks to regular files are accepted).

## Exit status

| Status | Meaning |
| --- | --- |
| `0` | Completed. The result may still contain partial Evidence, warnings or unresolved questions |
| `1` | Couldn't complete. Structured output names the cause (invalid input, permissions, cancellation, timeout, …) |
| `128 + N` | Ended by signal `N` (where the shell preserves it) |

A few special cases:
- `setup --dry-run` returns `planned` with `0`, and a cancelled setup also exits `0`.
- `setup` returns `1` for `needs_confirmation` or `needs_human`.
- `doctor` returns `1` only when required checks **in the selected scope** fail.

> [!warning] Preserve failure through pipes
> ```bash
> set -o pipefail
> rea inspect-artifact ./app.asar --json | jq . > inspection.json
> ```

CLI analysis translates SIGINT/SIGTERM into the same AbortSignal that providers use. Error shapes are covered in [[Errors and Exit Codes]].

## Source
- https://github.com/morluto/rea/blob/main/docs/cli.md
- https://github.com/morluto/rea/blob/main/src/cliCommandNames.ts
- https://github.com/morluto/rea/blob/main/scripts/rea.mjs
- https://github.com/morluto/rea/blob/main/src/cli/setupCommands.ts
