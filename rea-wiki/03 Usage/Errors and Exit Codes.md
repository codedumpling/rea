---
tags: [rea, usage, errors, reference]
source:
  - src/contracts/errorSchemas.ts
  - docs/error-contract.schema.json
  - docs/user-visible-errors.csv
  - docs/cli.md
  - docs/mcp-contracts.md
  - src/main/messages.ts
  - src/contracts/providerSelection.ts
---
# Errors and Exit Codes

REA has **one error contract** for both the CLI and MCP. It is the Zod schema `analysisErrorProjectionSchema` in `src/contracts/errorSchemas.ts`, published as JSON Schema in `docs/error-contract.schema.json`. The schema is a `oneOf`, with one branch per `code`.

## Error shape

```json
{
  "code": "provider_unavailable",
  "category": "unavailable",
  "message": "…plain-language message…",
  "retryable": false,
  "remediation": { "action": "…what to do next…" },
  "details": { "…": "code-specific JSON" }
}
```

`code`, `category`, `message`, `retryable` and `remediation.action` are required, and no other top-level fields are allowed. `details` is a free-form JSON object, except for `artifact_integrity_mismatch`, which requires `logical_path`, `declared_sha256`, `calculated_sha256` and `unpacked`.

### Where it shows up
- **MCP:** the tool result has `isError: true`, and the text `content` holds `{ "error": <projection> }`. There's **no** `structuredContent`, because output schemas describe successes only. Errors from tools that return target content also carry the `_meta.provenance.untrusted_content` marker.
- **CLI:** an envelope with `error` (a label such as `"Analysis failed"`) spread over the projection, optionally with `input_path` and `input_reason` (`invalid-json` | `read-failed` | `too-large`). The process exits `1`.

## Codes

| Code | Typical meaning / what to do |
| --- | --- |
| `invalid_request` | Bad arguments. Fix the named field |
| `access_denied` | Host or OS permission denied (e.g. Accessibility, Screen Recording) |
| `artifact_changed` | Target bytes changed since they were captured. Reopen it |
| `unreadable_output` | Provider returned something unparseable. Run `rea doctor` |
| `capability_unavailable` | Operation isn't available for this target, provider or server (e.g. `detail: "summary"` without session retention) |
| `unsupported_target` | Format, architecture or role not supported |
| `provider_unavailable` | Selected engine missing or broken. Run `rea doctor --provider ID --json` |
| `provider_timeout` | Engine deadline expired (see `REA_*_STARTUP_TIMEOUT_MS`) |
| `resource_constraint` | Budget exceeded. With `details.resource: "transport"` the result didn't fit an MCP frame (see below) |
| `cancelled` | Caller cancelled. Look for `details.partial_observation` |
| `artifact_integrity_mismatch` | Declared vs calculated hash differ. Consider `integrity_policy=record-and-continue` |
| `artifact_operation_failed` | Inventory or extraction failed |
| `evidence_integrity_mismatch` | Evidence invalid or changed. Recreate or re-import it |
| `truncated` | Output truncated |
| `process_capture_failed` | Process capture couldn't complete |
| `cleanup_incomplete` | Owned resources couldn't be verified as released. Address them before retrying |
| `revision_conflict` | Stale `update_unknown` revision. Re-read, then retry |
| `configuration_invalid` | Bad environment/config. Run `rea doctor` and fix the setting |
| `target_unavailable` | Target missing or unreadable |
| `execution_failure` | Generic execution failure |

**Categories** (a closed enum): `invalid_input`, `unsupported_provider`, `unsupported_target`, `integrity_mismatch`, `truncated`, `cancelled`, `timeout`, `resource_constraint`, `unavailable`, `execution_failure`.

> [!info] Provider selection details
> If deep-provider selection fails, the details carry stable reasons. `ambiguous` lists `details.candidate_ids`. Per-provider rejection codes include `not_configured`, `executable_missing`, `runtime_missing`, `unsupported_host`, `unsupported_version`, `target_kind_unsupported`, `target_format_unsupported`, `architecture_unsupported`, `target_role_unsupported`, `managed_target_unsupported`, `open_options_invalid`, `version_unresolved`. See [[Providers Overview]].

## Important recovery patterns

> [!example] Oversized MCP result
> `resource_constraint` + `details.resource: "transport"` means the analysis **succeeded and is retained**. Use `details.reported_limits.evidence_reference` with `inspect_analysis_view`, or call `export_evidence_bundle`. For big `analyze_function` results, `details.reported_limits.continuation` is a ready-made `inspect_analysis_view` call for the first pseudocode page.

> [!example] Atomic write with failed cleanup
> If an Evidence bundle or snapshot was published but staging cleanup failed, you get `details.published: true`, `details.published_path` and `details.bytes`. The file *is* complete.

> [!example] Partial observations
> Cancelled or failed native call tracing, process capture and JS analysis keep what they gathered in `details.partial_observation`. For JavaScript analysis this is a retained Evidence reference you can pass straight to `inspect_analysis_view`.

- **No target open:** "No app is open. Ask the user which app to investigate, then call open_binary with its local path."
- **Missing retained reference:** `details.reason: "missing"` plus the exact ID. Re-run the producer, or import a bundle.
- **Directory passed to `open_binary`:** the admission reason names `analyze_javascript_application` and `rea analyze` as alternatives.

## User-visible message catalogue

`docs/user-visible-errors.csv` is a harm-ranked audit of every user-facing message. Each row gives the location, trigger, old copy, risk, replacement copy, implementation status and verification. The design rules it applies:
- Don't leak provider internals or paths. Use plain language plus a concrete recovery step, which is often "Run `rea doctor`…".
- Say whether repeating the request will help ("repeating the unchanged request is not expected to succeed").
- Setup and uninstall messages state whether anything changed, and point to `.rea.backup` files.
- Install messages give exact Node ranges: 22.x (>=22.19), 24.x (>=24.11), or 26+.

MCP transport messages are listed in [[MCP Server]].

## Exit codes (CLI)

| Status | Meaning |
| --- | --- |
| `0` | Completed. May still contain partial Evidence, warnings or unknowns |
| `1` | Couldn't complete (structured error on output) |
| `128 + N` | Terminated by signal `N` |

Special cases: `setup --dry-run` (`planned`) and a cancelled setup exit `0`. `setup` exits `1` for `needs_confirmation`/`needs_human`. `doctor` exits `1` only when required checks in its scope fail. The MCP server process exits `1` if configuration is invalid or the initial `HOPPER_TARGET_PATH` fails to open. Use `set -o pipefail` when piping into `jq`. See [[CLI Reference]] and [[Troubleshooting]].

## Source
- https://github.com/morluto/rea/blob/main/src/contracts/errorSchemas.ts
- https://github.com/morluto/rea/blob/main/docs/error-contract.schema.json
- https://github.com/morluto/rea/blob/main/docs/user-visible-errors.csv
- https://github.com/morluto/rea/blob/main/docs/cli.md#output-and-exit-status
