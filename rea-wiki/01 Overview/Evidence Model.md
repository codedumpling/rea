---
tags: [rea, overview, evidence]
source:
  - AGENTS.md
  - docs/mcp-contracts.md
  - docs/cli.md
  - docs/migration-6.2.md
  - src/domain/evidence.ts
  - src/domain/residualUnknown.ts
  - src/application/investigation/EvidenceLedger.ts
---
# Evidence Model

REA's core promise is that every conclusion comes with the evidence behind it. The repository rules (AGENTS.md) require that REA **distinguish observations, derivations, inferences and unknowns** and **preserve Evidence provenance and session semantics through both adapters** (CLI and MCP).

## The four kinds of claim

| Kind | Meaning | How REA represents it |
| --- | --- | --- |
| **Observation** | Something an engine or reader directly saw in the artifact or runtime (a string at an address, an import, a network request). | Evidence with `confidence: "observed"` |
| **Derivation** | A fact computed deterministically from observations (a call path assembled from observed edges, a file offset mapped from an address). | Evidence with `confidence: "derived"` |
| **Inference** | A plausible interpretation that is not directly proven (a guessed argument type, an inferred interface, a reconstruction hint). | Evidence with `confidence: "inferred"`, often `authority: "analyst-inference"` |
| **Unknown** | A question the available evidence cannot settle. | A **residual unknown** record, plus `limitations`, `coverage: unknown`, `goal_status: unknown`, etc. |

> [!warning] Unknown is not "absent"
> An empty call list with partial or unknown coverage is **not** proof that a function has no calls. Unresolved identity or applicability must stay unknown rather than being treated as absent, foreign, or aligned. Read `relationship_coverage`, `limitations` and `residual_unknowns` before concluding a negative.

## Anatomy of an Evidence record

Defined in `src/domain/evidence.ts` (strict Zod schema). Key fields:

| Field | Purpose |
| --- | --- |
| `evidence_id` | `ev_`-prefixed digest computed from the record's canonical content (semantic identity). Tampering breaks the ID check. |
| `subject` | Name, SHA-256 digest, format (e.g. `elf`, `pe`, `mach-o`, `asar`, `apk`, `javascript`, `dos-mz`...), architecture and local path of what was analyzed, or `null` with a limitation explaining why. |
| `provider` | `id`, `name`, `version` of the engine that produced it. |
| `analysis_profile` | The provider's committed analysis profile, or `null`. Always present since 6.2 and part of identity. |
| `operation`, `parameters` | The tool and arguments that produced the result. |
| `raw_result` | The provider's own representation (e.g. an upstream report), or `null`. |
| `normalized_result` | REA's operation-specific result. This is what you usually read. |
| `confidence` | `observed`, `derived` or `inferred`. |
| `authority` | `shipped-artifact`, `controlled-replay`, `historical-reference`, `external-service`, or `analyst-inference`. |
| `environment` | Execution environment (platform, architecture, isolation: `none`/`process`/`container`/`virtual-machine`) for runtime evidence, else `null`. |
| `limitations` | Human-readable caveats. |
| `relationship_coverage` | How completely relationships (calls, references) were examined. |
| `locations` | Addresses, address ranges, artifact paths, file offsets or offset ranges. |
| `evidence_links` | IDs of other Evidence this record builds on. |

### Authority, in plain terms
- **shipped-artifact**: read from the actual distributed binary/package.
- **controlled-replay**: produced by running the target under REA's control (process capture, browser scenario).
- **historical-reference**: from an imported older source tree (`rea import-reference-source`).
- **external-service**: from a live external service.
- **analyst-inference**: an interpretation, not a direct observation.

## The EvidenceLedger

Each MCP connection owns a session-scoped **EvidenceLedger** (`src/application/investigation/EvidenceLedger.ts`): a set of immutable Evidence records plus residual-unknown revisions and heads.

- Recording is **idempotent**: the same record twice is a duplicate; a conflicting record with the same ID is rejected as an integrity error.
- Bundle import validates the entire bundle before merging atomically.
- The ledger is what `get_evidence_bundle`, `export_evidence_bundle` and `inspect_analysis_view` read from. Retained references belong to **one connection**; separate CLI calls consume full saved records instead.

See [[Evidence Export and Import]] for bundles and [[Sessions and Snapshots]] for how snapshots retain Evidence across runs.

## Residual unknowns

Defined in `src/domain/residualUnknown.ts`. An unknown has a stable `unk_` ID derived from its question, domain and scope, and is revised over time (each revision has a digest chained to the previous one).

| Field | Values / meaning |
| --- | --- |
| `question`, `domain` | What is unknown and in which area |
| `status` | `open`, `investigating`, `blocked`, `contradicted`, `resolved` |
| `severity` | `low`, `medium`, `high`, `critical` |
| `supporting_evidence_ids` / `contradicting_evidence_ids` | Evidence for and against |
| `required_authority`, `required_confidence`, `required_environment` | What evidence would be needed to settle it |
| `recommended_probes` | Suggested operations with rationale |
| `relationships` | `depends-on`, `related-to`, `contradicts` other unknowns |
| `resolution` | Only when resolved: `verified` (needs at least one Evidence ID), `withdrawn` or `out-of-scope`, with rationale |

Rules enforced by the schema: only resolved unknowns carry a resolution; `contradicted` requires contradicting evidence. Tools: `record_unknown`, `update_unknown`, `list_unknowns` (paged with `offset`/`limit`/`next_offset`). The `audit_residual_unknowns` prompt walks an agent through reviewing them ([[Agent Prompts]]).

## Provenance and redaction rules

- Caller-selected inputs and local evidence (output, URLs, paths, digests, metadata) are preserved.
- Transport authentication credentials and **explicitly marked** sensitive values are redacted; REA does not infer secrecy from names or text patterns.
- The ambient environment is not persisted merely because a child process inherits it.
- Target-derived text is flagged with `_meta.provenance.untrusted_content: true` so hosts can treat it as data, not instructions.

## Reading Evidence in practice

```javascript
// MCP, 6.2+
if (reply.isError) { /* parse JSON diagnostic from text content */ }
const evidence = reply.structuredContent;
const value = evidence.normalized_result;
const id = evidence.evidence_id;
```

On the CLI, `--json` output *is* the Evidence record. Save it, then project a view:

```bash
rea analyze-javascript-application /absolute/path/to/app --json > app-evidence.json
jq -c '{source: {kind: "inline", evidence: .}, view: {kind: "summary"}}' app-evidence.json > app-view.json
rea inspect-analysis-view app-view.json
```

> [!tip] Pass Evidence straight to comparisons
> `analyze_function` Evidence can be passed directly to `compare_functions`, and `inspect_artifact` Evidence to `compare_artifacts`.

## Related
- [[How REA Works]]
- [[Glossary]]
- [[MCP Server]] for result envelopes and response budgets
- [[Errors and Exit Codes]]

## Source
- https://github.com/morluto/rea/blob/main/AGENTS.md
- https://github.com/morluto/rea/blob/main/docs/mcp-contracts.md
- https://github.com/morluto/rea/blob/main/src/domain/evidence.ts
- https://github.com/morluto/rea/blob/main/src/domain/residualUnknown.ts
- https://github.com/morluto/rea/blob/main/src/application/investigation/EvidenceLedger.ts
