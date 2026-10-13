---
tags: [rea, usage, recipes]
source:
  - docs/cli.md
  - docs/javascript-application-workflows.md
  - docs/browser-observation.md
  - docs/electron-observation.md
  - docs/android-analysis.md
  - docs/adb-device-analysis.md
  - docs/web-network-captures.md
  - docs/process-capture.md
  - docs/firmware-analysis.md
  - docs/frida-instrumentation.md
  - docs/managed-code-analysis.md
  - docs/javascript-recovery.md
  - .agents/skills/reverse-engineer-anything/SKILL.md
---
# Recipes

Practical end-to-end workflows. Each one shows the **agent prompt** (MCP route) and the equivalent **CLI** commands. All commands and flags here come from the repo docs. Replace the paths and IDs with your own. Tool details are in [[MCP Tool Catalog]] and [[CLI Reference]].

> [!tip] General pattern
> Route by target, then start summary-first, and only make a focused follow-up when a specific question is still unanswered. Cite Evidence IDs, and keep observations separate from inferences and unknowns. This is what the [[Agent Skill]] tells agents to do.

---

## 1. Understand a feature in an Electron app
**Prompt:** *"Using REA, explain how the search feature works in /Applications/Notes.app's app.asar. Cite Evidence IDs."*

MCP route: `analyze_javascript_application` with `detail: "summary"`, then `inspect_analysis_view` on `parent_evidence_id` (module pages, then `node_id` items), then `trace_application_feature` with a seed.

```bash
rea analyze-javascript-application /absolute/path/to/app.asar --json > app-evidence.json
jq -c '{source: {kind: "inline", evidence: .}, view: {kind: "summary"}}' app-evidence.json > app-view.json
rea inspect-analysis-view app-view.json
# trace a literal string through the graph
node --input-type=module -e '
import { readFileSync, writeFileSync } from "node:fs";
const application = JSON.parse(readFileSync("app-evidence.json", "utf8"));
writeFileSync("trace-input.json", JSON.stringify({
  application, seed: { kind: "string", value: "search-result" }, direction: "both"
}));'
rea trace-application-feature ./trace-input.json --json
```
To limit source analysis, use `--source-path package.json --source-path main.js`. If hashes don't match for `.asar.unpacked` natives, use `--integrity-policy record-and-continue`. See [[JavaScript and Electron]].

## 2. Decompile a native function with Ghidra
**Prompt:** *"Open /abs/game.exe with Ghidra, find the function that uses the string 'pan', decompile it, and list its callers."*

```bash
rea doctor --provider ghidra --json
rea search /absolute/path/to/program "pan" --provider ghidra --json
rea trace /absolute/path/to/program "pan" --provider ghidra --json
rea function /absolute/path/to/program 0x1000 --provider ghidra --json
rea decompile /absolute/path/to/program 0x1000 --provider ghidra --json
rea xrefs /absolute/path/to/program 0x1000 --provider ghidra --json
```
MCP: `open_binary` with `{"path": "...", "provider_id": "ghidra"}`, then `search_strings`, `trace_feature`, `analyze_function`, `procedure_callers`, and `close_binary` at the end. Allow a long first-call timeout (see [[Ghidra]]).

> [!tip] Repeated CLI runs
> Add `--snapshot /abs/analysis/program.json` to every call. Exact repeats then come from the snapshot without starting Ghidra ([[Sessions and Snapshots]]).

## 3. Trace a website's network calls (live browser)
Start a dedicated browser profile yourself. REA never launches or owns it:
```bash
google-chrome --remote-debugging-port=9222 --user-data-dir=/tmp/rea-browser-profile http://127.0.0.1:3000
rea list-browser-targets http://127.0.0.1:9222 --json
rea inspect-web-page http://127.0.0.1:9222 TARGET_ID --observation-ms 1000 --json
rea analyze-web-bundle http://127.0.0.1:9222 TARGET_ID --json        # routes, endpoint candidates
rea observe-web-session http://127.0.0.1:9222 TARGET_ID --observation-ms 10000 --json   # you drive the page
```
Observation is **passive**. Cookies, auth headers and raw bodies are deliberately left out. See [[Websites and Browser Observation]].

## 4. Map an API surface from a saved HAR
```bash
rea inspect-web-network-capture /absolute/session.har har --json > har-evidence.json
rea inspect-web-network-capture /absolute/session.har har --record 2 --record 0 --json
```
MCP: `inspect_web_network_capture` with `detail: "summary"`, then `inspect_analysis_view` on the parent with `view: {"kind":"api-surface","offset":0,"limit":50}`. That groups requests by method, origin and templated path (`{numeric}`, `{uuid}`, `{hex}`, `{iso-date}`). Templates are **inferences**. SAZ, mitmproxy and qlog are also supported. See [[Network and Packet Captures]].

## 5. Inspect an APK statically
```bash
export REA_JADX_MCP_JAR=/absolute/path/jadx-headless-mcp-0.7.1-all.jar   # Java 17+ required
rea inspect-android-package /path/Example.apk
rea search-android-classes /path/Example.apk MainActivity
rea inspect-android-class /path/Example.apk example.MainActivity
rea inspect-android-method /path/Example.apk example.MainActivity onCreate
rea trace-android-references /path/Example.apk example.MainActivity --method-name onCreate
rea decode-android-resources /targets/Example.apk          # apktool: manifest + strings
rea identify-flutter-build /targets/Example.apk            # if it's a Flutter app
```
No emulator is needed and the APK is never launched. See [[Android]] and [[Flutter Apps]].

## 6. Pull an installed app off a device, then analyse it
```bash
rea list-adb-devices
rea resolve-adb-packages emulator-5554 whatsapp
rea pull-adb-package emulator-5554 com.whatsapp --output-directory /tmp/acq
rea inspect-android-package /tmp/acq/com.whatsapp/base.apk
rea read-adb-logcat emulator-5554 --count 200
```
`install-`, `uninstall-`, `start-`, `stop-adb-*` and `push-adb-file` **change the device** (`mutatesDevice`).

## 7. Compare two versions of an application
**Prompt:** *"Compare /apps/v1/app.asar and /apps/v2/app.asar; what changed in settings sync? Report unknowns where coverage is partial."*

MCP: call `analyze_javascript_application` once per version, then `compare_application_versions` with `left`/`right` as `retained-evidence` references. For one export's return shape, use `compare_javascript_export_shapes`. Something is reported added or removed only when the opposite side has complete coverage. For packages in general, `open_binary` each one, run `inspect_artifact`, then `compare_artifacts`. The guided prompt `compare_application_versions` scripts this ([[Agent Prompts]]).

## 8. Inspect a .NET assembly
```bash
rea inspect-managed-artifact /absolute/path/Assembly.dll --json
rea inspect-managed-members /absolute/path/Assembly.dll --type Example.Parser --json
rea inspect-managed-native-boundaries /absolute/path/Assembly.dll --json
```
These read metadata and CIL without executing anything. NativeAOT method bodies need native (Ghidra) analysis instead. See [[DotNET Assemblies]].

## 9. Capture and compare a CLI program's behaviour
`scenario.json`:
```json
{ "executable": "node", "arguments": ["-e", "console.log('ready')"], "timeout_ms": 5000 }
```
```bash
rea capture-process ./scenario.json --json > capture.json
rea compare-process-captures authority.json reconstruction.json
```
Process capture **runs the target with your permissions**. It is not a sandbox. Use the `prepare_bounded_process_capture` prompt to design the experiment first. See [[Process Capture]].

## 10. Hook a running process with Frida
```bash
rea list-frida-devices --json
rea list-frida-processes --json
rea instrument-with-frida attach 12345 --script-file ./hook.js --duration-ms 5000 --json
```
For long-running work, use the MCP session tools: `start_frida_session`, `load_frida_script`, `resume_frida_session`, `frida_session_status`, `close_frida_session`. A spawned process starts paused, and REA never kills the target. See [[Frida]].

## 11. Look inside firmware
```bash
export REA_BINWALK_COMMAND=/absolute/path/to/binwalk
export REA_UNBLOB_COMMAND=/absolute/path/to/unblob
rea inspect-firmware-regions /path/to/firmware.bin
rea extract-firmware /path/to/firmware.bin /path/to/new-output
```
The output directory must not exist yet. For headerless embedded code, run Ghidra with `--target-format raw-binary --architecture arm --entry-point 0x8000000`. See [[Firmware]].

## 12. Recover readable source from minified web scripts
```bash
rea export-web-scripts /analysis/capture.json /analysis/exported-scripts --json
rea recover-javascript-sources /tmp/web-scripts/app.js /tmp/recovered-app --json   # Wakaru, Linux x64
rea analyze-javascript-application /tmp/recovered-app/modules --json
```
Recovered code is a derived representation and is never executed. See [[JavaScript and Electron]].

## 13. Quick Go / SQLite / ELF triage (no disassembler)
```bash
rea inspect-go-binary /absolute/path/to/program --json
rea inspect-sqlite-database ./application.sqlite --table settings --column key --column value --row-limit 20 --json
rea inspect-binary-layout ./selected.elf --json
rea inspect-pe-resources ./selected.exe --json
```
See [[Go Binaries]], [[SQLite Databases]] and [[ELF Layout and Recorded Crashes]].

---

> [!warning] When a result is too big
> If MCP returns `resource_constraint` with `details.resource: "transport"`, **don't re-run the producer**. Call `inspect_analysis_view` with `details.reported_limits.evidence_reference`, or `export_evidence_bundle` to a file. See [[Errors and Exit Codes]].

## Source
- https://github.com/morluto/rea/blob/main/docs/cli.md
- https://github.com/morluto/rea/blob/main/docs/javascript-application-workflows.md
- https://github.com/morluto/rea/blob/main/docs/browser-observation.md
- https://github.com/morluto/rea/blob/main/docs/android-analysis.md
- https://github.com/morluto/rea/blob/main/docs/process-capture.md
- https://github.com/morluto/rea/blob/main/docs/web-network-captures.md
