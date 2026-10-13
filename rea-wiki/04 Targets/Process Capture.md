---
tags: [rea, target, process, runtime]
source:
  - docs/process-capture.md
  - src/process/
---
# Process Capture

Process Capture **runs one caller-selected command** in a native PTY and records bounded evidence about its terminal behavior, scheduled interactions, process lifetime and selected filesystem state. Its main use is comparing two direct runs of the same scenario — for example an authority build versus your reconstruction. It does not replay a previous execution or emulate dependencies.

## What REA returns

- Raw PTY output chunks (`data`, plus `raw_data` when normalization changed a chunk) and rendered terminal states (`lines`, `serialized_state`).
- Interaction dispatch outcomes (input, resize, signal events).
- Exit reason / settlement status and sampled process-tree observations.
- With `filesystem_observation_paths`: `files_before` / `files_after`, classified as created, deleted, modified or unchanged, with full-file SHA-256 when the `file_bytes` budget allows.
- Commitments for scenario, executable, comparison contract and normalization rules; truncation details and residual unknowns.

## Requirements and host support

| Host | Status |
| --- | --- |
| Linux | Supported (native PTY backend) |
| macOS | Supported; also needs Apple's Swift compiler via `xcrun` (REA compiles a narrow process-inspection helper into a private temp dir) |
| Windows | **Capture unavailable** (PTY adapter can't verify descendant cleanup). Use Linux REA inside WSL. Comparing existing captures works on Windows. |

A missing/incompatible PTY binary on a supported host → reinstall REA for that platform/arch/Node version with optional dependencies. See [[Troubleshooting]].

## CLI and MCP

| CLI | MCP |
| --- | --- |
| `rea capture-process <scenario.json>` | `capture_process_scenario` |
| `rea compare-process-captures <a.json> <b.json> [trace-spec.json]` | `compare_process_captures` |

Minimal scenario:

```json
{ "executable": "node", "arguments": ["-e", "console.log('ready')"], "timeout_ms": 5000 }
```

```sh
rea capture-process ./scenario.json --json > capture.json
rea compare-process-captures authority.json reconstruction.json
```

`--json` matters: the default terminal format is TOON. Write output to a file distinct from the scenario (shell redirection truncates first). Live status is streamed as `rea_progress` JSON lines on stderr (`phase`: `prepare`, `running`, `settling`, `cleanup`).

Scenario fields: `executable` (required), `arguments`, `working_directory`, `environment`, `filesystem_observation_paths`, `terminal` (`columns`, `rows`, `scrollback`), timed `events` (`input` with optional `sensitive: true`, `resize`, `signal`), `timeout_ms`, `idle_timeout_ms`, `settle_ms`, `limits` (e.g. `output_bytes`, `file_bytes`), `normalization` (literal `patterns`, `ports`). The exact contract is `processScenarioSchema`.

## Worked example

> [!example] Hash a report independently of noisy terminal output
> ```json
> {
>   "executable": "node",
>   "arguments": ["-e", "const fs=require('node:fs');fs.mkdirSync('reports',{recursive:true});fs.writeFileSync('reports/final.json',JSON.stringify({status:'complete',data:'X'.repeat(131072)}));console.log('x'.repeat(8192));"],
>   "working_directory": ".",
>   "filesystem_observation_paths": ["./reports"],
>   "limits": { "output_bytes": 1024, "file_bytes": 262144 }
> }
> ```
> The terminal output is truncated at 1 KiB, but the report's complete SHA-256 appears in `normalized_result.files_after` at `root_0:final.json`. REA never returns a prefix hash as a full-file digest.

> [!example] Prove a reconstruction behaves the same
> Capture the original binary and your rebuild with identical working directory, observation paths, terminal size, events and limits, then `rea compare-process-captures authority.json reconstruction.json`. Differing comparison contracts are rejected with the differing fields named. Add a trace-specification file when scheduling can reorder events.

## Limits and side effects

> [!warning] Not a sandbox
> The command runs with your user permissions and inherits the host environment (inherited values are not recorded). Observation paths select what REA snapshots — they do not restrict what the process can read or write.

- Filesystem effects are before/after snapshots, not a syscall trace; short-lived changes can be missed. With no paths selected, effects are unknown.
- Terminal dimensions must satisfy `columns × (rows + scrollback) ≤ 1,000,000`; oversized scenarios are rejected, never clamped.
- Sensitive inputs are sent to the process but persisted only as a byte-count placeholder; matching commitments don't reveal whether redacted inputs were equal.
- A complete file digest binds the bytes at that checkpoint — not the producer's notion of "finished."
- Residual unknowns or truncation prevent a complete-equivalence claim.

## Related
[[Native Binaries]] · [[JavaScript Runtime Observation]] · [[Evidence Model]] · [[Recipes]] · [[Troubleshooting]]

## Source
- https://github.com/morluto/rea/blob/main/docs/process-capture.md
