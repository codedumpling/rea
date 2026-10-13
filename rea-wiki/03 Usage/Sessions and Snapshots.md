---
tags: [rea, usage, sessions]
source:
  - src/application/binary/BinarySession.ts
  - src/application/binary/AnalysisSnapshotCache.ts
  - src/application/binary/AnalysisSnapshotFiles.ts
  - src/contracts/sessionLifecycleInputs.ts
  - docs/cli.md
  - docs/mcp-contracts.md
---
# Sessions and Snapshots

## The BinarySession

`BinarySession` (`src/application/binary/BinarySession.ts`) **owns the single active target** that the CLI and MCP adapters share. Its key properties:

- **One active target at a time.** Opening another target first drains calls against the current one, then tears down its provider client. Transitions are serialized. If a switch fails, REA recreates the previous target instead of keeping a client whose resources were already shut down.
- **Immutable provider binding.** A successful deep open binds one provider, with its concrete version, selection source and complete analysis profile (`analysis_provider_binding`). A runtime failure **never** silently swaps in another provider.
- **Lazy engines.** Opening selects a target and binding. The engine (for example Ghidra import and auto-analysis) starts on the first query that needs it.
- **Run identity.** Each successful transition allocates `analysis_run.run_id` before any provider starts. `process_lineage` moves from `not_observed` to `snapshots` once dynamic providers start, recording launcher PID, parent PID, process group and descendants at bounded checks.
- **Fast path.** Re-opening the same target reuses the retained client **only if it's actually alive** (checked against child health, not the composite's).

In MCP, the session lives for the whole server process (see [[MCP Server]]). In the CLI, each command opens a session and closes it on exit.

### MCP lifecycle tools
| Tool | Notes |
| --- | --- |
| `open_binary` | `path` (absolute), optional `provider_id` (`auto` or an ID), `snapshot_path` (absolute), `format` (`dos-com`/`raw-binary`), `architecture` and `entry_point` (both required with `raw-binary`) |
| `binary_session` | `{}` → target, binding, `tool_availability`, `analysis_activity`, server identity. Never starts analysis |
| `close_binary` | Optional `snapshot_path` and `overwrite` (default false). Drains work, saves, closes the provider and **clears retained Evidence and unknowns** |

```json
{ "path": "/absolute/path/to/program", "provider_id": "ghidra" }
```

> [!warning] Close clears the ledger
> Evidence retained on a connection (for `retained-evidence` references) belongs to that connection only. Opening another target **keeps** retained records, but `close_binary` clears them even if no binary is active. Export a bundle first if you'll need it later. See [[Evidence Export and Import]].

`analysis_activity` reports work still in progress. A client timeout can stop waiting while the provider keeps analysing. `cleanup_incomplete` names owned resources whose shutdown couldn't be verified.

## Retained Evidence (per connection)

Every Evidence-producing MCP call is recorded in the session's ledger. That's what makes these work:
- `{"kind":"retained-evidence","evidence_id":"ev_<64 hex>"}` inputs to `trace_application_feature`, `compare_application_versions`, `inspect_analysis_view` and others
- `detail: "summary"` on `analyze_javascript_application`, `inspect_binary_layout`, `inspect_artifact`, `inspect_wasm_artifact`, `inspect_managed_artifact` and `inspect_evm_interface`. The complete record is retained and you get a summary view whose `normalized_result.parent_evidence_id` points at it
- Recovery from an oversized reply through `details.reported_limits.evidence_reference`

A missing reference reports `details.reason: "missing"`. REA can't tell whether it was never recorded, was cleared, or belongs to another connection. Use `get_evidence_bundle` with `detail: "summary"` and filters (`evidence_id`, `operation`, `target_sha256`, `analysis_profile_digest`, `procedure_address`) to find IDs without pulling payloads.

## Analysis snapshots

A **snapshot** is a local JSON file of successful analysis results that later queries can reuse. REA replays an exact cached result when the **target bytes, operation, parameters, provider and settings** all match.

### What gets cached
- Immutable queries only. Stateful operations (`current_address`, `current_procedure`, `current_document`, `goto_address`, `list_documents`, health) are never cached.
- Document-scoped operations are cached only when an explicit `document` is supplied. Cursor-default operations (`address_name`, `comment`, `inline_comment`, `next_address`, `prev_address`, `xrefs`) are cached only with an explicit `address`.
- Live external state such as IDA results isn't replayed.
- There's a ceiling of 10 000 duplicated result bindings (`SNAPSHOT_CACHE_ENTRY_CEILING`).
- The file holds `target`, `binding`, `entries` (provider primitives), workflow entries and an `evidence_bundle`.

### CLI
```bash
rea analyze /absolute/path/to/program --provider ghidra --snapshot /absolute/path/to/analysis/program.json
# Repeat the same query: exact hit is read BEFORE any provider process starts
rea analyze /absolute/path/to/program --provider ghidra --snapshot /absolute/path/to/analysis/program.json
```

### MCP
- `open_binary` with `snapshot_path` imports a snapshot atomically for its matching target. A provider may still start before a cached result comes back.
- `close_binary` with `snapshot_path` (and optionally `overwrite: true`) saves before releasing resources. **If the save fails, the session stays open** so you can fix the output problem.
- The save receipt reports `primitive_entries`, `workflow_entries` and `evidence_records` separately. Zero primitive entries can still come with retained workflow results.

> [!info] A snapshot isn't a provider database
> Snapshots cache *observations*. New or live queries can still need the engine. Snapshot files are local and use **owner-only permissions**.

> [!warning] Writes invalidate the cache
> A metadata write (rename, comment, bookmark, `annotate_native_function`) invalidates the active snapshot cache, **even if the reply fails**, because the provider may already have applied it. Close and reopen the target before importing or saving another snapshot.

Snapshots keep eligible target-scoped question histories (residual unknowns) in full. If dependencies can't be kept, the whole history is excluded rather than reverted to an older revision.

## Recovering a stuck cold start
1. Keep useful inline Evidence, or `export_evidence_bundle`.
2. `close_binary` and check for `cleanup_incomplete`.
3. `open_binary` the same path with the same `provider_id`, then retry with a longer client timeout.

Re-issuing `open_binary` without closing can reuse the current client, so it **isn't** a fresh start. See [[Ghidra]] and [[Troubleshooting]].

## Source
- https://github.com/morluto/rea/blob/main/src/application/binary/BinarySession.ts
- https://github.com/morluto/rea/blob/main/src/application/binary/AnalysisSnapshotCache.ts
- https://github.com/morluto/rea/blob/main/docs/cli.md#save-and-reuse-analysis-snapshots
- https://github.com/morluto/rea/blob/main/docs/mcp-contracts.md#retained-application-evidence-inputs
