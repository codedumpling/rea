---
tags: [rea, target, javascript, electron, runtime]
source:
  - docs/javascript-runtime-observation.md
  - docs/electron-observation.md
  - docs/javascript-runtime-reconciliation.md
  - src/inspector/
---
# JavaScript Runtime Observation (Node and Electron)

Three runtime surfaces complement the static [[JavaScript and Electron]] analysis. They have deliberately different authorities:

| Surface | Tools | Authority |
| --- | --- | --- |
| Node/Electron **V8 Inspector** | `list_javascript_runtime_targets`, `observe_javascript_runtime` | Passive attach to a caller-owned target |
| Electron **`file://` page** via CDP | `list_electron_targets`, `inspect_electron_page` | Passive attach, no JS evaluation |
| Active **Electron scenario** | `capture_electron_scenario` | REA launches and owns Electron through Playwright |

`reconcile_javascript_runtime` then joins any of them with static Evidence.

## Passive V8 Inspector observation

Start your app with Inspector on loopback (REA does not launch it):

```bash
node --inspect=127.0.0.1:9229 ./app.mjs
rea list-javascript-runtime-targets http://127.0.0.1:9229 --json
rea observe-javascript-runtime http://127.0.0.1:9229 TARGET_ID --runtime-kind node --json
```

Runtime kinds (caller-declared, recorded as `caller-declared-unverified`): `node`, `electron-main`, `electron-preload`, `electron-renderer`.

**Returns:** `Debugger.scriptParsed` script locations, a bounded invalid-script count, and execution-context lifecycle within the capture window.

> [!info] Exactly two protocol commands
> The provider sends only `Runtime.enable` and `Debugger.enable`. It never evaluates, reads source, sets breakpoints, pauses/resumes or calls `Runtime.runIfWaitingForDebugger`. Use `--inspect`, not `--inspect-brk` — REA will not resume a paused target.

## Passive Electron page inspection

```bash
rea list-electron-targets http://127.0.0.1:9223 --json
rea inspect-electron-page http://127.0.0.1:9223 TARGET_ID --observation-ms 100 --json
rea inspect-electron-page http://127.0.0.1:9223 TARGET_ID --include-script-sources --json
```

**Returns:** canonical local paths (including members inside `app.asar`), frame and DOM structure, resource metadata, stable script/resource identities, worker/service-worker/shared-worker inventory, completeness, and — only with `include_script_sources` — content-addressed script artifacts. Only literal-port loopback HTTP endpoints and local hostless `file://` URLs are accepted.

## Active Electron scenarios

```bash
rea capture-electron-scenario scenario.json --json
```

The request supplies the Electron executable, the app entry point (root defaults to its parent), arguments and actions: click/wait, window-targeted renderer reload/crash, and synthetic `open-url` / `second-instance` deep links. Results include an app/window/WebContents timeline with preload, session, navigation, shell, permission, popup, download, protocol, native-addon, process and **IPC channel + value-shape metadata** (payload values are never retained). External shell/navigation, permission, download, popup, updater and OS-integration effects are **blocked and recorded**.

## Reconciliation

```bash
rea reconcile-javascript-runtime reconciliation-input.json --json
```

Input: `static_layers` (exactly one `application` role, optional `cache`/`assets`) each with an `analysis` Evidence record and optional `runtime_mappings` (`file-root` or `url-prefix`), plus `runtime_observations` (complete Evidence from `inspect_web_page`, `inspect_electron_page`, `observe_javascript_runtime` or `capture_electron_scenario`). Output: a combined graph with match / ambiguity / mismatch / unknown classifications. No filesystem, browser or Electron I/O.

## Worked example

> [!example] Did the preload script actually load?
> 1. Static: `rea analyze-javascript-application /abs/app.asar --json > static.json` ([[JavaScript and Electron]]).
> 2. Launch the app yourself so it serves CDP on a literal loopback port (Chromium's standard remote-debugging switch; not an REA flag), then `rea inspect-electron-page http://127.0.0.1:9223 TARGET_ID --include-script-sources --json > page.json`.
> 3. Build a reconciliation input with `static.json` as the application layer and `page.json` as a runtime observation; run `reconcile-javascript-runtime`.
> 4. Captured script bytes give the strongest identity; V8 Inspector matches are location-only and weaker.

## Limits and side effects

- **Attach is inherently incomplete**: scripts collected before attach may be missing; absence never proves a script doesn't run. `scriptParsed` proves presence, not import edges, execution order or causality. IPC messages, EventEmitter emissions and unload events are not exposed by Inspector.
- Selecting an endpoint exposes **all** targets it serves and their local paths/origins. Targets explicitly marked `attached: true` are rejected; Node's `/json/list` omits attachment state, so it is `null` (unknown).
- V8 capture: 8 MiB aggregate metadata budget, 16 MiB per CDP message (`payload_limit`).
- Electron page inspection never navigates, clicks, invokes IPC, closes targets or terminates the app. ASAR headers limited to 16 MiB.
- **Active scenarios launch the target with normal filesystem and network privileges** — ownership is not a sandbox. 60-second startup deadline; CLI SIGINT/SIGTERM awaits owned-process cleanup. Scenario Evidence is target-only/partial for reconciliation.

> [!info] Verification
> `REA_ELECTRON_EXECUTABLE=/absolute/path/to/electron npm run verify:electron`; `npm run verify:javascript-runtime -- --application <dir> --runtime-evidence <json> ...`. See [[Testing]].

## Related
[[JavaScript and Electron]] · [[Websites and Browser Observation]] · [[Process Capture]] · [[Frida]] · [[Evidence Model]]

## Source
- https://github.com/morluto/rea/blob/main/docs/javascript-runtime-observation.md
- https://github.com/morluto/rea/blob/main/docs/electron-observation.md
- https://github.com/morluto/rea/blob/main/docs/javascript-runtime-reconciliation.md
