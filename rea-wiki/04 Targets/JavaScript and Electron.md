---
tags: [rea, target, javascript, electron]
source:
  - docs/javascript-artifact-reconstruction.md
  - docs/javascript-application-graph.md
  - docs/javascript-application-workflows.md
  - docs/javascript-recovery.md
  - src/javascript/
  - src/artifacts/
---
# JavaScript and Electron (static)

REA reads a local Electron/JavaScript **directory or `.asar`** without executing any of its code and returns a **JavaScript Application Graph**: packages, ASAR entries, Electron main/preload/renderer code, modules, source maps, BrowserWindows, `contextBridge` APIs, IPC channels/handlers, workers, endpoints, storage and native add-ons. No [[Hopper]]/[[Ghidra]], browser or Electron process is required.

For live Electron/Node processes see [[JavaScript Runtime Observation]]; for websites see [[Websites and Browser Observation]].

## What REA returns

- Canonical local path, root artifact digest, artifact manifest and graph commitments.
- The graph itself (nodes + evidence-bearing edges: `contains`, `loads`, `imports`, `maps_to`, `exposes`, `sends`, `invokes`, `handles`, `calls`, `persists_to`, `observed_as`, `changed_from`).
- Static Electron summary: BrowserWindow options and `webPreferences`, preload paths, `contextBridge.exposeInMainWorld`/`exposeInIsolatedWorld` keys, IPC operations with conservative pairing, sender/origin validation *candidates*, utility processes, `.node` binding requests.
- Webpack/Rspack chunk registrations and module factories, imports/requires/dynamic imports, workers, routes, endpoints (incl. tRPC procedures/links and named GraphQL operations), storage, vendor markers, local source maps.
- A separate **semantic relation graph** (def/use, closures, calls, Promise flow, EventEmitter, timers, `child_process`, config defaults, request/response) for `trace_javascript_semantics`.

## Requirements

Node.js and npm only. Optional: **Wakaru ^1.13.0 on Linux x64** for un-bundling (`REA_WAKARU_COMMAND`, plus util-linux `prlimit` / `REA_JAVASCRIPT_PRLIMIT_COMMAND`). See [[External Tools]].

## CLI and MCP

```bash
rea analyze /absolute/path/to/apps/app.asar --json
rea analyze-javascript-application /absolute/path/to/apps/app.asar \
  --integrity-policy record-and-continue --json
rea analyze /absolute/path/to/app.asar --source-path package.json --source-path main.js --json
```

Generic `rea analyze` picks this static provider for directories and `.asar` paths when neither `--provider` nor `--snapshot` is given. Progress lines (`rea_progress`) go to stderr.

MCP `analyze_javascript_application`:

```json
{
  "input_path": "/absolute/path/to/app.asar",
  "format": "auto",
  "integrity_policy": "fail",
  "source_paths": ["package.json", "main.js"],
  "detail": "summary"
}
```

`format`: `auto` | `asar` | `directory`. `integrity_policy`: `fail` (default) | `record-and-continue`.

### Follow-up workflows (consume Evidence; never reread the artifact)

| MCP tool / CLI command | Purpose |
| --- | --- |
| `inspect_analysis_view` / `inspect-analysis-view` | Summary, module page or one module from retained Evidence |
| `trace_application_feature` / `trace-application-feature` | Reachable subgraph from a literal seed (node ID, route, string, API, IPC channel, module, native export) |
| `trace_javascript_semantics` / `trace-javascript-semantics` | Backward provenance / forward influence / callers / ownership on the semantic graph |
| `compare_application_versions` / `compare-application-versions` | Pair entities across two graph versions by tiered identity |
| `compare_source_to_bundle` / `compare-source-to-bundle` | Historical source tree vs. current bundle |
| `compare_javascript_export_shapes` / `compare-javascript-export-shapes` | Execution-free return-shape diff of one export |
| `recover_javascript_sources` / `recover-javascript-sources` | Un-bundle with Wakaru into readable modules |
| `reconcile_javascript_runtime` / `reconcile-javascript-runtime` | Join with runtime captures (see [[JavaScript Runtime Observation]]) |

Related artifact tools: `rea inspect-artifact` / `rea extract-artifact` (MCP `inspect_artifact`, `inventory_artifact`, `extract_artifact`) inventory and extract packages and resources with digests.

## Worked example

> [!example] Trace a string through an Electron app
> ```bash
> rea analyze-javascript-application /abs/app.asar --json > application-evidence.json
> node --input-type=module -e '
> import { readFileSync, writeFileSync } from "node:fs";
> const application = JSON.parse(readFileSync("application-evidence.json", "utf8"));
> writeFileSync("trace-input.json", JSON.stringify({
>   application, seed: { kind: "string", value: "search-result" }, direction: "both"
> }));'
> rea trace-application-feature ./trace-input.json --json
> ```
> Over MCP, analyze with `detail: "summary"` and pass `{"kind":"retained-evidence","evidence_id":"<parent_evidence_id>"}` as `application` instead of the full record. Native handoffs in the trace bind native add-on digests and recommend provider follow-up (`requires-provider-analysis`) — REA never starts a provider implicitly.

> [!example] Un-bundle minified chunks first
> ```sh
> rea recover-javascript-sources /tmp/web-scripts/main.js /tmp/recovered-chunks \
>   --additional-path /tmp/web-scripts/runtime.js --json
> rea analyze-javascript-application /tmp/recovered-chunks/modules --json
> ```
> Output directory must be absent; up to 16 additional files; `extraction_mode` `structural` (default) | `heuristic` | `inspection`; `rewrite_level` `minimal` | `standard` | `aggressive`.

## Limits and side effects

- **Never executes code**: no `eval`, `Function`, `vm`, DOM, bundle `push` handlers or bootstrap. Directory readers don't follow symlinks; native add-ons are metadata only.
- Imports, loads, IPC pairings and calls are **static inferences** — syntax does not prove runtime execution. Malformed files produce `unknown`/`unavailable` scopes, not evidence of absence.
- ASAR integrity mismatches fail by default; `record-and-continue` keeps both hashes, marks bytes `observed-untrusted`, and makes coverage partial. Missing `.asar.unpacked` bytes stay `unavailable`.
- `source_paths` reduces parsing but makes coverage partial (unselected files are `outside-analysis-scope`). `detail: "summary"` reduces delivery only.
- Timeouts/cancellation return partial Evidence via a `partial_observation` retained reference.
- Version comparison reports `added`/`removed` only when the opposite graph has complete coverage; otherwise `unknown`.
- Wakaru: 1 GiB address space, 120 s, 64 MiB input, 128 MiB/10000 entries output; existing output is never overwritten.

> [!info] Verification
> `npm run verify:application-workflows -- --left <dir|asar> --right <dir|asar>` reconstructs, compares and traces two operator-provided versions without printing source text.

## Related
[[JavaScript Runtime Observation]] · [[Websites and Browser Observation]] · [[WASM Artifacts]] · [[Apple and Harmony Apps]] · [[Native Binaries]] · [[Evidence Model]] · [[MCP Tool Catalog]]

## Source
- https://github.com/morluto/rea/blob/main/docs/javascript-artifact-reconstruction.md
- https://github.com/morluto/rea/blob/main/docs/javascript-application-graph.md
- https://github.com/morluto/rea/blob/main/docs/javascript-application-workflows.md
- https://github.com/morluto/rea/blob/main/docs/javascript-recovery.md
