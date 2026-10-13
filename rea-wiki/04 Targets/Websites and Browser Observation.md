---
tags: [rea, target, browser, web]
source:
  - docs/browser-observation.md
  - docs/browser-scenario-contract.md
  - docs/web-runtime.md
  - docs/website-script-export.md
  - docs/website-module-trace.md
  - docs/web-source-location.md
  - src/browser/
---
# Websites and Browser Observation

REA inspects websites through a **Chrome-family browser** over the Chrome DevTools Protocol (CDP). There are two kinds of authority, kept in separate providers:

- **Attach-only (passive)** — CDP page capture against a browser *you* started; REA never launches, drives, evaluates or mutates it.
- **Owned (active)** — Playwright scenarios where REA launches a browser (or connects to one), runs declared actions and captures step snapshots.

Saved HAR/SAZ/mitmproxy traffic is covered in [[Network and Packet Captures]].

## What REA returns

Page structure (DOM shape, accessibility tree), scripts and resources, safe response/DOM metadata, attach-window network/console/WebSocket metadata, workers, quota, optional storage key names and redacted fingerprints, optional script sources, screenshots, static bundle analysis (chunks, routes, endpoints, vendor fingerprints, WebMCP declarations, optional source maps), V8 coverage, and event listeners.

## Requirements

- A Chrome-family browser you install. For passive tools, start it with a dedicated profile and a **literal loopback** debugging port:

```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/rea-browser-profile \
  http://127.0.0.1:3000
```

- On Linux with a locked keyring, add `--password-store=basic` to the dedicated profile.
- `localhost`, LAN addresses, HTTPS endpoints, credentials, paths and implicit ports are not valid CDP endpoints.
- `trace_web_module_imports` additionally needs `REA_BROWSER_EXECUTABLE` (Chromium supporting `import.meta.resolve`).

## Tool map

| MCP tool / CLI command | Kind | Purpose |
| --- | --- | --- |
| `list_browser_targets` / `list-browser-targets` | passive | Discover page targets (optional exact-origin filter) |
| `inspect_web_page` / `inspect-web-page` | passive | DOM, a11y, scripts, resources, network/console/WebSocket metadata |
| `analyze_web_bundle` / `analyze-web-bundle` | static | Parse selected scripts: chunk edges, routes, endpoints, vendors, WebMCP, source maps |
| `observe_web_session` / `observe-web-session` | passive | Observation window for an external user action: reloads, SPA navigation, redirects |
| `discover_webmcp_tools` / `discover-webmcp-tools` | passive | WebMCP registrations + full `input_schema` (never invokes) |
| `capture_web_screenshot` / `capture-web-screenshot` | passive | Visible-viewport PNG |
| `compare_web_captures`, `compare_web_screenshots` | offline | Compare captures / PNG pixels |
| `observe_web_execution` / `observe-web-execution` | instrumenting | Precise V8 coverage + request initiators |
| `inspect_web_event_listeners` / `inspect-web-event-listeners` | passive | Listeners registered on a CSS-selected node |
| `capture_browser_scenario` / `capture-browser-scenario` | **owned** | Declarative Playwright scenario |
| `export_web_scripts` / `export-web-scripts` | offline | Write captured sources for static analysis |
| `trace_web_module_imports` / `trace-web-module-imports` | offline* | Resolve native ES imports of one exported script |
| `trace_web_source_location` / `trace-web-source-location` | offline | Map a generated position through a local source map |

\* runs REA's resolver stub in an owned browser; application code is never executed.

## Passive capture

```bash
rea list-browser-targets http://127.0.0.1:9222 --json
rea inspect-web-page http://127.0.0.1:9222 TARGET_ID --observation-ms 1000 --json
rea inspect-web-page http://127.0.0.1:9222 TARGET_ID \
  --include-accessibility-text --include-console-text \
  --include-json-body-shapes --include-websocket-shapes \
  --include-script-sources --include-storage-keys --include-storage-fingerprints --json
rea analyze-web-bundle http://127.0.0.1:9222 TARGET_ID --json
rea observe-web-session http://127.0.0.1:9222 TARGET_ID --observation-ms 10000 --json
rea capture-web-screenshot http://127.0.0.1:9222 TARGET_ID --json
```

MCP input: `{"cdp_endpoint": "http://127.0.0.1:9222", "target_id": "..."}`. Call `list_browser_targets` first; target IDs are browser-instance-specific. Default inspection window is 500 ms; default scope is the page's current origin (add `allowed_origins` to widen).

## Owned scenarios

```json
{
  "browser": { "mode": "launch", "executable_path": "/absolute/path/to/chromium" },
  "start_url": { "url": "http://127.0.0.1:3000" },
  "actions": [
    { "step_id": "search", "action": "click", "locator": { "kind": "css", "selector": "#search" }, "timeout_ms": 10000 },
    { "step_id": "results-ready", "action": "wait_for", "locator": { "kind": "css", "selector": "#results[data-state='ready']" }, "state": "visible", "timeout_ms": 10000 }
  ],
  "capture": { "at_end": ["dom", "accessibility", "url"] }
}
```

```bash
rea capture-browser-scenario ./browser-search.json --json > browser-search-capture.json
```

`browser.mode` is `launch` (provider-owned temp profile, deleted on cleanup; `headless` defaults to `true`) or `connect` (explicit-port loopback CDP; REA only disconnects). Captures: `dom`, `accessibility`, `screenshot`, `url`, `history`, `storage`, at end or `after_each_step`; `events` such as `console`, `page-errors`, `network`; `capture.network` with `request_body`, `response_body`, `header_values` (all default `false`). Secrets are declared and redacted; service workers are blocked unless `environment.service_workers` is `allow`.

> [!warning] `not_requested` means "did not look"
> A URL-only capture can be `complete` yet unable to answer a question about rendered content. Request the artifact you need and wait for a page-specific ready condition.

## Runtime source attribution

```bash
rea observe-web-execution http://127.0.0.1:9222 TARGET_ID --observation-ms 10000 --json
rea inspect-web-event-listeners http://127.0.0.1:9222 TARGET_ID '#submit' --json
```

Wait for the armed progress event (`rea_progress` on stderr / MCP progress) before triggering the behavior.

## Worked example: capture → export → analyze → map

> [!example]
> ```sh
> rea inspect-web-page http://127.0.0.1:9222 TARGET_ID --include-script-sources --json > /analysis/capture.json
> rea export-web-scripts /analysis/capture.json /analysis/exported-scripts --json
> rea analyze-javascript-application /analysis/exported-scripts/files --json
> REA_BROWSER_EXECUTABLE=/abs/chrome rea trace-web-module-imports /analysis/exported-scripts/manifest.json 0 --json
> rea trace-web-source-location /analysis/exported-scripts/manifest.json 0 \
>   /analysis/app.js.map 'https://example.test/maps/app.js.map' 1 25 --json
> ```
> Then continue in [[JavaScript and Electron]] (tracing, comparisons) or [[JavaScript Runtime Observation]] (reconciliation with `inspect_web_page` Evidence). Bundles can also be un-minified with `recover_javascript_sources`.

## Limits and side effects

- **Data minimization:** DOM text/attribute values are not retained; URL userinfo is stripped (query values and fragments kept); cookies/authorization headers never retained; storage values always redacted; JSON/WebSocket captures keep value-free shapes only. Selected accessibility text, console text, script sources and query values **may contain secrets** and become Evidence.
- Network/console coverage starts at attach (`prior_activity_available` is always `false`).
- Source maps are fetched only by `analyze_web_bundle` with `fetch_source_maps`, same-origin, no credentials (30 s, 64 MiB).
- 64 MiB per CDP WebSocket message (`payload_limit`). Main-frame navigation during capture → `target_changed`.
- Never exposed: generic CDP commands, `Runtime.evaluate`, WebMCP invocation, navigation, input, downloads, page/browser closure.
- **`observe_web_execution` resets coverage counters and disables optimized execution** while armed — it changes timing/JIT behavior. Real coverage: Linux x64 / Chrome 153 / Node 24.
- Scenarios **launch and drive** a browser; the page can make external requests. This is not a sandbox — use a dedicated profile and only origins you intend to inspect.
- `compare_web_screenshots` admits both PNGs against a 256 MiB estimated working-memory budget.

> [!info] Verification
> `npm run verify:browser`, `npm run verify:browser:scripts`, `npm run verify:browser:runtime` (set `REA_BROWSER_EXECUTABLE`). See [[Testing]].

## Related
[[Network and Packet Captures]] · [[JavaScript and Electron]] · [[JavaScript Runtime Observation]] · [[Evidence Model]] · [[Legal and Responsible Use]]

## Source
- https://github.com/morluto/rea/blob/main/docs/browser-observation.md
- https://github.com/morluto/rea/blob/main/docs/browser-scenario-contract.md
- https://github.com/morluto/rea/blob/main/docs/web-runtime.md
- https://github.com/morluto/rea/blob/main/docs/website-script-export.md
- https://github.com/morluto/rea/blob/main/docs/website-module-trace.md
- https://github.com/morluto/rea/blob/main/docs/web-source-location.md
