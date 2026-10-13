---
tags: [rea, overview, glossary]
source:
  - AGENTS.md
  - docs/cli.md
  - docs/mcp-contracts.md
  - docs/installation.md
  - src/domain/evidence.ts
  - src/domain/residualUnknown.ts
---
# Glossary

Terms used throughout REA's docs and this vault, alphabetized.

**Analysis profile** — The committed description of how a provider analyzed a target (provider identity and version, loader, language, analyzer preset such as `ghidra-default+disabled:...`, seed digest...). Recorded in every Evidence record as `analysis_profile` (object or `null`) and part of its identity. See [[Evidence Model]].

**`analysis_provider_binding` / `analysis_provider_candidates`** — Fields of `binary_session`. Candidates are the possible deep providers; the binding is the one concretely selected for the current target. Not interchangeable. See [[Sessions and Snapshots]].

**ASAR** — Electron's archive format (`app.asar`). REA reads it directly for static JavaScript analysis. See [[JavaScript and Electron]].

**Authority** — Where an Evidence record's claim comes from: `shipped-artifact`, `controlled-replay`, `historical-reference`, `external-service` or `analyst-inference`.

**Auxiliary capability** — A non-deep-provider operation (artifact inventory, format readers, etc.) routed alongside the session's deep provider. Listed by `rea capabilities --json`.

**Binary session** — The MCP state created by `open_binary`: one target bound to one immutable deep provider until switched or closed with `close_binary`. Inspect with `binary_session`.

**BYO (bring your own)** — Engines REA uses but never installs: Ghidra, IDA, JEB, JADX, Binwalk, Unblob, pwntools, WABT, TShark, sigrok, Apktool, adb, etc. Hopper is the one exception setup may install, with approval. See [[External Tools]].

**CDP** — Chrome DevTools Protocol, used to observe websites and Electron renderers. See [[Websites and Browser Observation]].

**`cleanup_incomplete`** — A result/diagnostic meaning REA could not verify shutdown or removal of some owned resource (provider process, document, temp dir). REA retains ownership so you can retry `close_binary`. See [[Troubleshooting]].

**Client / agent** — The MCP host (Claude Code, Codex, Cursor, Gemini CLI, ...). Each has a `--client` ID such as `claude_code` or `codex`. See [[Agent Setup]].

**Compact profile** — `REA_MCP_INPUT_SCHEMA_PROFILE=compact`: advertises smaller input schemas (each within 13,312 bytes) for size-capped model APIs; server-side validation is unchanged. See [[Environment Variables]].

**Confidence** — An Evidence record's `observed`, `derived` or `inferred` level.

**Coverage** — How completely something was examined (e.g. `relationship_coverage`). Partial/unknown coverage means a missing item is not proof of absence.

**Deep provider** — A native analysis engine selected per session: `hopper`, `ghidra`, `ida` (and JEB for its own tools). See [[Providers Overview]].

**Doctor** — `rea doctor`, the strictly read-only readiness diagnostic. See [[Readiness Checks]].

**Dry run** — `rea setup --dry-run`: returns the setup plan with status `planned`, exit 0, without writing anything.

**Evidence** — A strict, provider-neutral record of one successful observation, with an `ev_` ID. See [[Evidence Model]].

**Evidence bundle** — A collection of Evidence records and unknowns, exported/imported as JSON (`export_evidence_bundle`, `rea evidence-export`, `rea evidence-import`). See [[Evidence Export and Import]].

**EvidenceLedger** — The per-connection, idempotent store of Evidence and unknown revisions inside the MCP server.

**Lease / reservation** — Ownership guard preventing two REA sessions from driving the same Hopper target (or, on Linux, the same Hopper application). A competing session receives the owning run ID. See [[Hopper]].

**Limitations** — The `limitations` array on Evidence: caveats a reader must account for.

**MCP** — Model Context Protocol. REA serves its tools over stdio via `rea-agents mcp`. See [[MCP Server]].

**`normalized_result` / `raw_result`** — REA's operation result vs. the provider's own original representation (or `null`).

**P0 (Windows Ghidra P0)** — The experimental, read-only Windows x64 Ghidra support boundary. See [[Ghidra Variants]].

**Package runner** — `npx` or `bunx`. Setup pins registrations like `npx -y rea-agents@<version> mcp`.

**Pinned registration** — An MCP registration fixed to the exact REA version that performed setup. Updating the CLI does not update it; rerun setup. See [[Updating and Uninstalling]].

**Provider** — Any engine adapter (deep or auxiliary). `rea providers --json` lists them.

**`provider_id`** — The `open_binary` argument that selects a deep provider (`auto` or a concrete ID); same parser as CLI `--provider` and `REA_ANALYSIS_PROVIDER`.

**Residual unknown** — A durable `unk_` record for an unresolved question with status, severity, evidence for/against and recommended probes. Managed via `record_unknown`, `update_unknown`, `list_unknowns`.

**`resource_constraint`** — Error code returned when a result cannot fit the transport budget; it carries a reference to the retained Evidence. See [[MCP Server]].

**Retained Evidence reference** — `{"kind":"retained-evidence","evidence_id":"ev_..."}`: a pointer to Evidence held by the current MCP connection, accepted by tools such as `inspect_analysis_view`.

**Run ID** — `binary_session.analysis_run.run_id`, allocated before any provider starts for each target transition.

**Setup plan** — The reviewed list of exact paths, changes, backups and external software that `rea setup` proposes before writing.

**Skill** — The bundled `reverse-engineer-anything` agent workflow instructions installed by setup (or via skills.sh). See [[Agent Skill]].

**Snapshot** — A local file that caches successful, exact analysis results (target bytes + operation + parameters + provider + settings) for reuse via `--snapshot` / `snapshot_path`. See [[Sessions and Snapshots]].

**TOON** — The default compact terminal output format of the CLI; use `--json` for JSON.

**`tool_availability`** — The grouped per-tool availability, reason and remediation reported by `binary_session`. `tools/list` always returns the complete catalog, including currently unavailable tools.

**Untrusted content marker** — `_meta.provenance.untrusted_content: true` on results that may contain target-derived text.

## Source
- https://github.com/morluto/rea/blob/main/docs/mcp-contracts.md
- https://github.com/morluto/rea/blob/main/docs/cli.md
- https://github.com/morluto/rea/blob/main/docs/installation.md
- https://github.com/morluto/rea/blob/main/src/domain/evidence.ts
