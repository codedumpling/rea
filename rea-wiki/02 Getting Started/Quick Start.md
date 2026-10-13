---
tags: [rea, getting-started]
source:
  - README.md
  - docs/index.md
  - docs/installation.md
  - docs/cli.md
---
# Quick Start

Get from zero to a first REA result in a few minutes. You need **Node.js 22.x (>=22.19), 24.x (>=24.11) or 26+** with npm. Node 23, 25 and prereleases are unsupported.

## Option A: with your agent (recommended)

### 1. Run setup
```bash
npx rea-agents setup
```
- Pick your agents in the multi-select (existing REA registrations are preselected; newly detected clients are not).
- Review the plan: exact config paths, skill destinations, backups, and any optional Hopper install.
- Approve. npm's package-download prompt and REA's setup approval are separate.

> [!info] What setup changes
> It adds REA's MCP server entry (pinned to the exact version, e.g. `npx -y rea-agents@6.4.0 mcp`) to each selected client config, installs the bundled `reverse-engineer-anything` skill, and keeps `.rea.backup` copies. Nothing is written until you approve. See [[Agent Setup]].

### 2. Restart your agent
Restart every agent named in the completion message. Tools do not appear in an already-running session.

### 3. Ask a question
```text
Understand how search works in the Notes app, show me the evidence, and build a
similar feature for my project.
```
Replace the app and feature with your own target. The agent routes the target, calls REA tools, and answers with Evidence IDs and limitations.

## Option B: from the terminal (no agent, no native engine)

Static JavaScript/Electron analysis needs nothing beyond Node.js:
```bash
npx -y rea-agents@latest analyze-javascript-application /absolute/path/to/app --json
```
Use an extracted app directory or an `.asar`. On Windows use a path such as `"D:/apps/example"`. The result includes modules, imports, Electron boundaries and their evidence.

Real apps can produce huge Evidence (hundreds of MB). Save it and read a summary instead:
```bash
rea analyze-javascript-application /absolute/path/to/app --json > app-evidence.json
jq -c '{source: {kind: "inline", evidence: .}, view: {kind: "summary"}}' app-evidence.json > app-view.json
rea inspect-analysis-view app-view.json
```

To have `rea` on your PATH for regular use:
```bash
npm install --global rea-agents
rea --help
```

## Option C: native binaries

Native decompilation needs a provider:
- **Hopper** (macOS/Linux): setup can install it after separate approval. See [[Hopper]].
- **Ghidra** 12.1.x + JDK 21+: bring your own, then export `GHIDRA_INSTALL_DIR` (and optionally `JAVA_HOME`). See [[Ghidra]].
- **IDA**: via an IDA MCP server and `REA_IDA_MCP_CONFIG`. See [[IDA]].

```bash
export GHIDRA_INSTALL_DIR=/absolute/path/to/ghidra_12.1.4_PUBLIC
rea doctor --provider ghidra --json
rea analyze /absolute/path/to/program --provider ghidra
```

## Check that it worked

| What to verify | Command |
| --- | --- |
| Your agent's registration | `rea doctor --client claude_code --json` (use your client ID) |
| The installed skill | `rea doctor --skill --json` |
| One analysis engine | `rea doctor --provider ghidra --json` |

See [[Readiness Checks]]. If tools are still missing after a restart, see [[Troubleshooting]].

## Next steps
- Pick your target family: [[Targets Overview]]
- Learn the commands: [[CLI Reference]], [[MCP Tool Catalog]]
- Understand the results: [[Evidence Model]]
- Keep REA current: [[Updating and Uninstalling]]

## Source
- https://github.com/morluto/rea/blob/main/README.md#quick-start
- https://github.com/morluto/rea/blob/main/docs/installation.md
- https://github.com/morluto/rea/blob/main/docs/cli.md
