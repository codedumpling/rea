---
tags: [rea, getting-started, doctor]
source:
  - docs/installation.md
  - docs/cli.md
  - docs/mcp-contracts.md
  - src/cli/setupCommands.ts
  - src/application/DoctorProviderIds.ts
  - src/mcpDoctor.ts
---
# Readiness Checks

`rea doctor` is REA's diagnostic. It is **strictly read-only**: it reads host prerequisites and agent configuration and never changes them. Run it when something is not working, or before an unattended job, not before every investigation.

> [!tip] Don't over-diagnose
> Without options, doctor audits *everything* (every detected registration, the skill, and all optional engines). Its overall `healthy` can be `false` while the workflow you actually need works fine. Scope the check to your task.

## Pick the check for your task

| Task | Readiness check |
| --- | --- |
| Static JavaScript/Electron analysis | Just run `rea analyze-javascript-application PATH --json` |
| One analysis engine | `rea doctor --provider ghidra --json` (or `hopper`, `ida`) |
| One agent registration | `rea doctor --client codex --json` (any client ID from [[Agent Setup]]) |
| Installed workflow instructions | `rea doctor --skill --json` |
| A specific target | add `--target /absolute/path/to/app` |
| The MCP server itself starts and lists tools | `rea mcp doctor --json` |

## Doctor options

Verified in `src/cli/setupCommands.ts`:

| Option | Meaning |
| --- | --- |
| `--client <id>` | Agent registration whose readiness is required; repeatable |
| `--provider <id>` | Deep provider whose readiness is required: `hopper`, `ghidra`, `ida`; repeatable |
| `--skill` | Require the installed REA skill identity to be aligned |
| `--target <path>` | Add a check for an app path |
| `--json` | Structured report (full health diagnostics and canonical tool-catalog details) |

## Reading the report

- A scoped report has `scope.mode: "explicit"`. Its `scope_checks` decide `healthy` **and the exit status**.
- Other checks are listed under `informational_checks`; `environment_healthy` summarizes the full audit.
- `--target` alone adds a target check but the report stays audit-wide unless a scope option is also given.
- Exit status: `1` when required checks in the selected scope fail; unavailable optional providers remain informational. See [[Errors and Exit Codes]].

### Registration states
Doctor inspects supported JSON/TOML/YAML client registrations and reports each command vector as **aligned**, **stale**, **missing** or **invalid**. It cannot see the live agent connection: an aligned registration with no tools in your session means **restart/reconnect** the agent.

### Skill readiness
`doctor --skill --json` verifies installed copies against the bundled instructions and references. Read `identity.skill.state`, `installed_version` and `installed_tool_count`. (The obsolete `installed_catalog_digest` was removed; catalog identity is at `identity.catalog`.)

### Provider readiness
- **Ghidra**: platform/architecture, Ghidra 12.1.x version, `support/analyzeHeadless` (or `.bat`), the install's Java major range, 64-bit JDK, `javac` presence, and the native decompiler for your host (checked by reading the executable header, so a wrong-architecture build is reported as incompatible). Look for `native_decompiler` / `ghidra-native_decompiler`.
- **Hopper**: launcher path (override with `HOPPER_LAUNCHER_PATH`), on Linux the private-display strategy and host/effective `/tmp/.X11-unix` mount facts.
- **IDA**: the `REA_IDA_MCP_CONFIG` registration.

Shell `PATH` changes do not affect an already-running MCP server; the server must be launched with the right environment.

## Doctor inside setup

`rea setup --json` includes a compact `.doctor` projection; `doctor.scope.clients` lists required registrations and `doctor.scope.skill_clients` lists the skill destinations checked. Setup and doctor both report a malformed REA `env` map or invalid runtime setting **before** setup writes files; fix the named setting in the client config.

## In-session readiness (MCP)

Doctor checks files. From inside a connected agent, the authoritative live view is:
- `binary_session` with `{}` → `result.tool_availability` (grouped per-tool availability, reason, remediation), `server_identity`, provider candidates and binding.
- Optional inputs `expected_package_version`, `expected_catalog_digest`, `expected_server_path` compare the live session with your expectations.
- `tools/list` always returns the full catalog, including unavailable tools, so presence in the list does not mean callable.

CLI equivalents for discovery:
```bash
rea providers --json
rea capabilities --json
```

## Provider selection errors

If a native target has several available providers you get `capability_unavailable` with `details.selection_reason`:
- `ambiguous` → choose from `details.candidate_ids` with `--provider` / `provider_id`.
- `provider_unavailable` → repair the selected engine; run `rea doctor --provider ID --json`.

## Related
- [[Troubleshooting]] · [[Agent Setup]] · [[Providers Overview]] · [[CLI Reference]]

## Source
- https://github.com/morluto/rea/blob/main/docs/installation.md#check-readiness-for-your-task
- https://github.com/morluto/rea/blob/main/docs/mcp-contracts.md#identity-and-discovery
- https://github.com/morluto/rea/blob/main/src/cli/setupCommands.ts
