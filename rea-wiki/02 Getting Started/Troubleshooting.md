---
tags: [rea, getting-started, troubleshooting]
source:
  - docs/installation.md
  - docs/client-configuration.md
  - docs/mcp-contracts.md
  - docs/cli.md
  - README.md
  - .agents/skills/reverse-engineer-anything/references/connection-and-recovery.md
---
# Troubleshooting

Start every bug hunt the way the README does: **update first** ([[Updating and Uninstalling]]), restart the agent, retry. If it persists, diagnose by the symptom you actually see; registration, client approval, shell PATH and engine availability each have different remedies.

## Symptom → remedy

| Symptom | Meaning | Next step |
| --- | --- | --- |
| No REA tools in the agent session | Not registered, or the agent was not restarted | `rea doctor --client <id> --json`; restart the agent. A skill install alone does not register MCP. |
| Registration missing/malformed/stale | Config needs repair | `rea setup --client <id> --dry-run --json`, review, then apply that scope |
| Registration aligned but no tools | Client didn't reconnect or failed to launch the server | Restart/reconnect; inspect the client's launch error. Doctor cannot see the live connection. |
| `MCP tool call requires approval, but approval policy is never` | Codex rejected the call before REA ran | Client policy, not a REA problem. Use the client's approval flow or preapprove specific tools ([[Agent Setup#Codex approval policy]]). |
| `provider_unavailable` / unavailable tool entry | The engine for that operation isn't ready | Read the diagnostic or `binary_session` → `tool_availability`; `rea doctor --provider <id> --json` |
| `capability_unavailable` with `selection_reason: ambiguous` | Several providers support the target | Choose from `details.candidate_ids` via `--provider` / `provider_id` / `REA_ANALYSIS_PROVIDER` |
| `rea: command not found` in a child shell | PATH or Node issue, not a broken MCP registration | `command -v rea`, `command -v node`, `node --version`, `rea --version`; add the install dir (often `~/.local/bin`) to PATH |
| Setup reports `needs_confirmation`, exit 1 | `--yes` without approval scope | Add `--client <id>` or `--all-detected` |
| Setup blocked by malformed config | REA refuses to rewrite broken files | Fix the named file (e.g. Pi and ZCode require strict JSON: no comments, trailing commas or BOM) and rerun |
| `resource_constraint` with `details.resource: "transport"` | Result too large for the 10 MiB stdio frame | Use `inspect_analysis_view` with `details.reported_limits.evidence_reference`, or `export_evidence_bundle`. See [[MCP Server]]. |
| `cleanup_incomplete` | REA couldn't verify a resource was released | Address the reported resource, then retry `close_binary` on the **same** connection |

## Large tool catalog / context thrashing

Some clients send every tool's schema with every request:
- **Claude Code** with a non-first-party `ANTHROPIC_BASE_URL` (e.g. DeepSeek's Anthropic-compatible endpoint) turns tool search off. The docs measured ~496 KB of tool definitions with the default profile, ~290 KB with `compact`, which can make autocompact thrash. Fixes: make Claude Code assume the real context window (`CLAUDE_CODE_MAX_CONTEXT_TOKENS`, or a model ID with `[1m]`), try `ENABLE_TOOL_SEARCH=true` if the endpoint supports `tool_reference`, or set the compact profile.
- **GitHub Copilot CLI** 1.0.95 with `gpt-4.1` metadata can block chat with `compaction_static_context_blocked`.
- **Moonshot/Kimi** clients reject any single tool schema over ~13.7–15 KB.

Compact profile: add `"REA_MCP_INPUT_SCHEMA_PROFILE": "compact"` to the `env` object of the `rea` MCP entry, then restart. Setup preserves this override.

## Native provider problems

### Ghidra
- **First query times out in the agent.** `open_binary` doesn't wait for import; the first Ghidra-backed query does (default deadline 330,000 ms). Raise `REA_GHIDRA_STARTUP_TIMEOUT_MS` in the **server's** environment, and make sure the client's own tool-call timeout is long enough. A client can give up before Ghidra finishes. See [[Ghidra]].
- **`native_decompiler` missing** (common on macOS): the archive may lack the host decompiler binary; build it per the installation guide (`make ghidra_opt`, with `ARCH_TYPE="-arch arm64" OSDIR=mac_arm_64` on Apple Silicon).
- **Out of memory / constrained host:** `GHIDRA_HEADLESS_MAXMEM=512M` (or `GHIDRA_MAXMEM`; default `2G`). REA clears `_JAVA_OPTIONS`, `JAVA_TOOL_OPTIONS`, `JDK_JAVA_OPTIONS` and `GHIDRA_JAVA_OPTIONS`, so don't inject flags there.
- **Auto-analysis stalls in one analyzer:** `REA_GHIDRA_DISABLED_ANALYZERS="Objective-C Message Analyzer"`.
- **Wrong language detected** (e.g. Borland PE): `REA_GHIDRA_LANGUAGE_ID` (+ optional `REA_GHIDRA_COMPILER_SPEC_ID`).
- **GUI client can't find Java:** setup records the observed JDK home; otherwise add `JAVA_HOME` to the registration `env`.

### Hopper
- `Cannot launch Hopper! Connection is invalid.` or waiting for bridge readiness → check Hopper for a pending demo, licence or loader dialog; complete it and retry. On macOS, complete the first-run dialog in the same user session before any CLI/MCP or unattended job.
- macOS denies Apple Events → allow the app that starts REA under **System Settings → Privacy & Security → Automation**.
- Linux launcher won't start → `ldd /absolute/path/to/Hopper | grep 'not found'` and install missing libraries.
- A second session reports an owning run ID → another REA session holds the target (macOS) or the single Linux Hopper application; switch targets through that session or close it.
- Large binaries → raise `REA_HOPPER_STARTUP_TIMEOUT_MS` (default 120,000 ms).
- In CI: `rea doctor --provider hopper --json` distinguishes private-display dependencies, unsupported dialogs, ownership conflicts and early exits.

See [[Hopper]] for lease recovery and retained-resource guidance.

## Large JavaScript results

Complete application Evidence can be hundreds of MB. Save `--json` output to a file and use `rea inspect-analysis-view` with a `summary` or `page` view rather than printing it. With pipes, use `set -o pipefail` so a downstream `jq` doesn't hide REA's exit status.

## Windows notes
- Historical-source import (`rea import-reference-source`) returns `unsupported_host` on native Windows; use WSL.
- Windows Ghidra is the experimental read-only P0 boundary; Hopper is unavailable. See [[Ghidra Variants]].
- Setup wraps runner commands in `cmd.exe /d /c` so clients can start `npx`/`bunx` shims.

## Reporting a bug
Open an issue at https://github.com/morluto/rea/issues with your REA version, target type, steps to reproduce and error output. **Security issues go through the private advisory flow**, not public issues; see [[Legal and Responsible Use]].

## Related
- [[Readiness Checks]] · [[Errors and Exit Codes]] · [[FAQ]] · [[Environment Variables]]

## Source
- https://github.com/morluto/rea/blob/main/docs/installation.md
- https://github.com/morluto/rea/blob/main/docs/client-configuration.md
- https://github.com/morluto/rea/blob/main/docs/mcp-contracts.md#ghidra-first-query-deadlines-and-recovery
- https://github.com/morluto/rea/blob/main/.agents/skills/reverse-engineer-anything/references/connection-and-recovery.md
