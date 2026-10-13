---
tags: [rea, getting-started, agents, mcp]
source:
  - docs/installation.md
  - docs/client-configuration.md
  - src/cli/setupCommands.ts
  - .agents/skills/reverse-engineer-anything/SKILL.md
  - .agents/skills/reverse-engineer-anything/references/connection-and-recovery.md
---
# Agent Setup

`rea setup` connects coding agents to REA's local MCP server and installs the matching workflow skill. It is a **reviewed transaction**: it shows exactly what it will change and writes nothing until you approve.

## Supported agents

| Client | `--client` value | | Client | `--client` value |
| --- | --- | --- | --- | --- |
| Claude Code | `claude_code` | | Qwen Code | `qwen_code` |
| Claude Desktop | `claude_desktop` | | VS Code | `vscode` |
| Codex | `codex` | | Grok Build | `grok_build` |
| Cursor | `cursor` | | OMP | `omp` |
| Gemini CLI | `gemini_cli` | | Pi | `pi` |
| Windsurf | `windsurf` | | Hermes | `hermes` |
| Devin | `devin` | | Qoder | `qoder` |
| OpenCode | `opencode` | | Trae | `trae` |
| Antigravity | `antigravity` | | ZCode | `zcode` |
| GitHub Copilot CLI | `copilot_cli` | | DeepSeek Harness | `dsh` (main only, not in npm 6.4.0) |
| Command Code | `commandcode` | | | |

**Grok Bot** (`grok_bot`) is a special case: it runs connectors on its hosted computer, so setup does not write a file. Ask the Grok Bot chat to add a custom MCP server named `rea` that runs `npx -y rea-agents@<version> mcp`. `rea doctor --client grok_bot` reports this manual step and `rea uninstall` does not remove it.

Any other client that supports local stdio MCP servers can use [[#Manual MCP registration]].

## Interactive setup

```bash
npx rea-agents setup
```

1. **Select agents.** Existing REA registrations are preselected. Newly detected clients are listed but **unselected**: detection is context, not permission. You can also select clients with no detected config.
2. **Review the plan.** It lists any existing/proposed Hopper installation or verified Ghidra installation, each agent config path, each skill destination, and external software, network origins, integrity evidence and package-manager commands.
3. **Approve.** Hopper installation needs its own explicit approval.
4. **Restart** the agents named in the completion message.

Safety properties:
- Malformed or unsafe existing config blocks the **whole** transaction before any write.
- Writes preserve unrelated entries and comments, create backups (an existing `.rea.backup` is retained, not overwritten), use atomic replacement, and verify the result.
- Existing valid REA environment overrides (e.g. `REA_MCP_INPUT_SCHEMA_PROFILE`, `REA_MCP_MAX_RESPONSE_BYTES`) are kept; detected provider settings replace only matching keys.
- Declining or Ctrl-C makes no changes and returns status `cancelled` (exit 0).

## Setup flags

Verified in `src/cli/setupCommands.ts`:

| Flag | Effect |
| --- | --- |
| `--client <id>` | Configure this agent; repeatable; skips the interactive selection |
| `--all-detected` | Configure every detected supported agent |
| `--skill=false` | Omit the bundled skill (it is installed by default with integrations) |
| `--skill-client <id>` | Refresh/verify an existing client's skill destination independently of MCP registration; repeatable |
| `--dry-run` | Read-only plan, status `planned`, exit 0 |
| `--yes` / `-y` | Approve user-owned setup actions without prompting |
| `--install-hopper` | Also approve Hopper installation with `--yes` |
| `--accessible` | Sequential, vertically rendered yes/no prompts |
| `--json` | Structured result including a compact `.doctor` readiness projection |

Examples:
```bash
rea setup --client codex --client cursor --skill=false --dry-run
rea setup --yes --all-detected --install-hopper --json
rea setup --skill-client claude_code --yes --json
```

> [!warning] `--yes` without a scope
> Without `--client` or `--all-detected`, `--yes` applies only to **existing REA-owned registrations**; it does not select all detected clients. An unapproved actual apply reports `needs_confirmation` and exits 1.

Prompts and progress go to stderr so stdout stays clean for structured output. `NO_COLOR=1` disables colour. Interactive setup requires stdin, stdout and stderr to all be terminals; otherwise setup stays non-interactive.

## What gets written

- An MCP server entry pinned to the exact REA version: `npx -y rea-agents@<version> mcp` (or `bunx rea-agents@<version> mcp` when setup ran through `bunx`). On Windows the runner command is wrapped in `cmd.exe /d /c`. Global installs may register Node launching REA's entry script directly.
- `startup_timeout_sec = 30` for Codex and Grok Build.
- The bundled skill in each client's personal skill directory (mixed selections plan each distinct destination). Selecting the skill without a client uses `~/.agents/skills`.

### Client-specific notes (from client-configuration.md)
| Client | Where/how |
| --- | --- |
| Claude Code | `mcpServers.rea` in `~/.claude.json` (`$CLAUDE_CONFIG_DIR/.claude.json` if set) |
| Codex | Config under `CODEX_HOME` (unset/empty means `~/.codex`) |
| Devin | `devin/mcp_config.json`, honouring `XDG_CONFIG_HOME` on non-Windows |
| Gemini CLI | `$GEMINI_CLI_HOME/.gemini/settings.json` when set; workspace/system `mcpServers.rea` replaces the user entry and setup stops on conflicts; never edits workspace/admin settings or grants trust |
| Qwen Code | `mcpServers` in `~/.qwen/settings.json` or `$QWEN_HOME/settings.json`; skill in `~/.agents/skills` |
| Antigravity | Skill in `~/.gemini/config/skills` |
| Pi | `mcpServers.rea` (`type: "stdio"`) in `~/.pi/agent/mcp.json`, strict JSON (no comments/trailing commas/BOM); follows `PI_CODING_AGENT_DIR`; run `/reload` in an existing Pi session |
| OMP | `~/.omp/agent/mcp.json` or profile path via `OMP_PROFILE`/`PI_PROFILE`; removes `rea` from `disabledServers` |
| Hermes | `mcp_servers.rea` in `config.yaml` under `HERMES_HOME` (default `~/.hermes`, `%LOCALAPPDATA%\hermes` on Windows); follows `active_profile` |
| Qoder | `mcpServers.rea` in `~/.qoder/settings.json` (`QODER_CONFIG_DIR`); skill in `~/.qoder/skills` |
| Trae | `mcp.json` in Trae's VS Code-style user dir; skill in `~/.trae/skills` (loading unverified) |
| ZCode | nested `mcp.servers` in `~/.zcode/cli/config.json`; skill in `~/.zcode/skills` |
| OpenCode | V1 `mcp.rea` (or V2 `mcp.servers` if already used); honours `OPENCODE_CONFIG_DIR` and `OPENCODE_CONFIG` |
| Grok Build | `[mcp_servers.rea]` in `$GROK_HOME/config.toml` or `~/.grok/config.toml` |
| DeepSeek Harness | Cordis YAML patch `$DSH_HOME/cordis.patch.yml` (entry id `rea-mcp`); tools appear as `mcp__rea__<tool_name>` |

> [!tip] Large tool catalogs
> Some clients forward every tool schema with every request. Claude Code does this when `ANTHROPIC_BASE_URL` points to a non-first-party host (tool search turns off); the docs measured about 496 KB of definitions with the default profile vs about 290 KB with `compact`. Add `"REA_MCP_INPUT_SCHEMA_PROFILE": "compact"` to the `env` of the `rea` entry and restart. The same workaround applies to GitHub Copilot CLI with `gpt-4.1` metadata and to Moonshot/Kimi clients. See [[Troubleshooting]] and [[Environment Variables]].

## Manual MCP registration

For clients setup does not support (or if you prefer to edit config yourself), register the stdio server with one exact version:

```json
{
  "mcpServers": {
    "rea": {
      "command": "npx",
      "args": ["-y", "rea-agents@6.4.0", "mcp"]
    }
  }
}
```

Add provider settings in an `env` object if needed (e.g. `GHIDRA_INSTALL_DIR`, `JAVA_HOME`, `REA_ANALYSIS_PROVIDER`). Persistent registrations should use one exact package version; rerun current setup later to refresh older registrations, then restart the client. Registry-aware clients can instead search for `io.github.morluto/rea`.

## Skill-only installation

```bash
npx skills add morluto/rea --skill reverse-engineer-anything
```

This installs **instructions only** (the skill and its bundled references), not the MCP registration or any analysis engine. If tools are missing afterwards:

```bash
rea doctor --client codex --json          # inspect the registration (use your client ID)
rea setup --client codex --dry-run --json # plan the repair, then approve that scope
```

An aligned registration whose tools are absent from the active session needs a restart/reconnection; `doctor` checks files and prerequisites, not the live agent connection. The skills.sh route may install newer instructions than your installed server supports, so follow the server's actual schemas. Static JavaScript CLI analysis works even while MCP is unavailable. See [[Agent Skill]].

## Codex approval policy

Codex can require approval for REA tools that create owned resources or launch helpers; this is client policy, not a REA readiness issue. An operator can preapprove specific tools in a disposable profile:

```toml
[mcp_servers.rea.tools.open_binary]
approval_mode = "approve"

[mcp_servers.rea.tools.close_binary]
approval_mode = "approve"
```

Only approve operations whose effects you accept.

## Related
- [[Installation]] · [[Readiness Checks]] · [[Updating and Uninstalling]] · [[Troubleshooting]] · [[MCP Server]]

## Source
- https://github.com/morluto/rea/blob/main/docs/installation.md
- https://github.com/morluto/rea/blob/main/docs/client-configuration.md
- https://github.com/morluto/rea/blob/main/.agents/skills/reverse-engineer-anything/references/connection-and-recovery.md
