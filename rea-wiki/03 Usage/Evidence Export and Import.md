---
tags: [rea, usage, evidence]
source:
  - src/cliEvidenceCommands.ts
  - docs/cli.md
  - docs/mcp-contracts.md
  - docs/reconstruction-obligation-ledgers.md
  - docs/reconstruction-readiness.md
---
# Evidence Export and Import

Every REA result is an **Evidence record**. It keeps artifact identity, source locations, observations, inferences and unresolved findings (see [[Evidence Model]]). An **Evidence bundle** collects records with their artifacts, providers, environments, scenarios, captures and residual unknowns into one canonical, deterministic JSON document.

```json
{
  "artifacts": [], "providers": [], "environments": [],
  "scenarios": [], "captures": [], "unknowns": [], "records": []
}
```

## CLI commands (`src/cliEvidenceCommands.ts`)

| Command | Description | Args / flags |
| --- | --- | --- |
| `rea evidence-import PATH` | "Validate and import a bounded local Evidence bundle" | bundle JSON path |
| `rea evidence-export SOURCE OUTPUT` | "Validate and atomically export canonical Evidence JSON" | `--overwrite` to replace an existing output |
| `rea compare LEFT RIGHT` (alias `compare-bundles`) | "Compare two canonical Evidence bundles" | two bundle paths |
| `rea inspect-evidence-bundle JSON` | Summary/filtered projection of an exported bundle | `{"path", "detail":"summary", "filters":{…}}` |

```bash
rea evidence-import /absolute/path/to/evidence/bundle.json
rea evidence-export /absolute/path/to/evidence/bundle.json /absolute/path/to/evidence/canonical.json
rea compare /absolute/path/to/evidence/left.json /absolute/path/to/evidence/right.json
rea inspect-evidence-bundle '{"path":"/tmp/evidence.json","detail":"summary","filters":{"operation":"analyze_function"}}' --format json
```

> [!info] Exports never clobber silently
> An export keeps any existing destination unless you pass `--overwrite`. Writes are atomic: content is staged and published in one step. If staging cleanup fails *after* publication, the error carries `details.published: true`, `details.published_path` and `details.bytes`.

CLI failures return `{"error": "Analysis failed", …canonical projection}` (see [[Errors and Exit Codes]]).

## MCP tools

| Tool | Use |
| --- | --- |
| `export_evidence_bundle` | Stream the session's complete canonical bundle to a path. This is the escape hatch when a result is too big for one MCP frame |
| `import_evidence_bundle` | Validate every Evidence ID and manifest, then merge atomically into the session. After that, imported records can be referenced as `retained-evidence` |
| `get_evidence_bundle` | Return the bundle inline. Use `detail: "summary"` with `filters` to list IDs only |
| `compare_bundles` | Exact record membership, explicit observation pairs and unknown histories |

> [!tip] Move work between connections
> Retained references belong to one MCP connection, and `close_binary` clears them. To carry an investigation over: `export_evidence_bundle`, then (on a new connection) `import_evidence_bundle`, then reference the imported IDs. Separate CLI calls always use full saved records instead. See [[Sessions and Snapshots]].

Cancelling an export stops serialization and removes the staging file. The destination only changes when a complete export is published.

## Residual unknowns

Open questions are first-class, versioned records: `record_unknown` creates one, `update_unknown` appends a revision (stale revisions raise `revision_conflict`), `list_unknowns` pages current heads (`offset`/`limit`/`next_offset`), and `verify_unknown_resolution` re-validates against live bundled evidence. They travel in the bundle's `unknowns` array.

## Reconstruction obligation ledgers

When the goal is a **reconstruction** (re-implementing observed behaviour), `build_reconstruction_obligation_ledger` (CLI `rea build-reconstruction-obligation-ledger`) turns authenticated Evidence into a deterministic list of claims that must be proven.

Inputs:
- `evidence_bundle`: an exported bundle. Application Graph and process-capture records generate candidate obligations automatically.
- `reviewed_obligations`: hand-written claims, each with `obligation_id`, `obligation_version`, `application_layer`, `family`, `required_case_kinds` (e.g. `positive`, `negative`, `malformed`, `cancellation`, `teardown`), required original/fixture/verifier authority, `review_evidence_ids`, and so on.
- `manifest`: `bindings` (owner module/symbol plus `owner_sha256`, optional `parser_type`, `original_cases`, `fixtures`, `verifier`) and `contradictions`.

A smoke test that returns an `unknown` ledger with zero obligations:

```sh
rea build-reconstruction-obligation-ledger \
  '{"evidence_bundle":{"artifacts":[],"providers":[],"environments":[],"scenarios":[],"captures":[],"unknowns":[],"records":[]},
    "reviewed_obligations":[],"manifest":{"bindings":[],"contradictions":[]}}' --json
```

### Closure rules (conservative by design)
- An obligation closes only when **one** binding supplies a unique owner, any required parser/schema/domain type, **every** required case fixture, and a passing verifier with **comparable authority** that enumerates the obligation ID and whose result is in the bundle.
- A green unit test can't close a `packaged-process` obligation, and static-only Evidence can't close a runtime claim.
- Contradictions, duplicate owners, residual unknowns, missing dependencies or unavailable authority keep closure open or failed.
- `blocked` / `out-of-scope` dispositions are recorded but never counted as verified.
- The output is identified by `ledger_id` and `closure_digest`.

Related tools: `evaluate_reconstruction_coverage` (coverage against one named boundary) and `verify_reconstruction` (a finite typed spec against a bundle).

## Importing historical source

`rea import-reference-source /absolute/path/to/source` records hashes and metadata for an older source tree so you can compare it with a shipped bundle (`compare-source-to-bundle`). It honours `.gitignore` and default exclusions (`node_modules/`, `dist/`, `*.log`, …), records the enclosing Git HEAD, and marks paths matching `REA_REFERENCE_SECRET_PATTERNS_JSON` as configured-secret exclusions. It needs safe no-follow opens, so **native Windows returns `unsupported_host`**. Use WSL instead. See [[JavaScript and Electron]].

## Source
- https://github.com/morluto/rea/blob/main/src/cliEvidenceCommands.ts
- https://github.com/morluto/rea/blob/main/docs/cli.md#import-export-and-compare-evidence
- https://github.com/morluto/rea/blob/main/docs/reconstruction-obligation-ledgers.md
- https://github.com/morluto/rea/blob/main/docs/reconstruction-readiness.md
