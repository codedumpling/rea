---
tags: [rea, provider, ida]
source:
  - docs/ida-provider.md
  - docs/testing.md
  - src/ida/
  - src/config/environment.ts
---
# IDA

Provider ID: `ida`. REA does **not** drive IDA directly. It adapts an **existing IDA MCP server** into REA's native analysis tools. REA never installs, upgrades, activates or licenses IDA, and does not bundle an IDA server. **IDA analysis is read-only**: REA never saves or closes an attached GUI database.

Supported upstreams:
- [mrexodia/ida-pro-mcp](https://github.com/mrexodia/ida-pro-mcp) for IDA Pro, in a legacy attached GUI profile or a modern headless database-supervisor profile.
- The optional [ida-free-mcp native plugin](https://github.com/0xshlomil/ida-free-mcp) for IDA Pro and IDA Free (`"protocol": "native"`, experimental).

## Configure

1. Write the upstream registration into its own JSON file, with exactly one lifecycle mode.
2. Point `REA_IDA_MCP_CONFIG` at that file. The path must be absolute.
3. Select `ida` once, through `REA_ANALYSIS_PROVIDER`, `--provider ida` or `open_binary.provider_id`.

Nothing starts during discovery. `rea doctor --provider ida --json` validates the registration without launching IDA.

### Attached GUI profile (legacy 1.4.0 tools)

```json
{ "command": "python",
  "args": ["/absolute/path/to/ida_pro_mcp/server.py", "--ida-rpc", "http://127.0.0.1:13337"] }
```

- Open the **original input binary** in REA, not an `.idb`/`.i64`. Its SHA-256 must match IDA's recorded input hash. REA re-checks identity and image base around each request.
- Results are **live**, so edits made in the GUI show up and nothing is replayed from snapshots. Some operations move GUI focus (`effects.may_show_ui: true`).
- This profile can span WSL and a Windows GUI, because it binds the recorded hash rather than a path.

### Native plugin profile

```json
{ "mode": "attached", "protocol": "native", "url": "http://127.0.0.1:13337/mcp" }
```

> [!warning] Listener exposure
> Native MCP 1.1.2 binds `0.0.0.0` and has no listen-address setting. Firewall it, or use a loopback-only build. REA's loopback URL check does not change the plugin's own listener. Use 1.1.3 or a build that includes the navigation fix if you need function switching to work reliably.

IDA Free's cloud decompiler needs network access and may ask for consent. Direct callers are unavailable in this profile.

### Headless profile (database supervisor)

```json
{ "command": "idalib-mcp", "args": ["--stdio", "--max-workers", "1"], "mode": "headless" }
```

- Requires upstream's supervisor API (`idb_open`/`idb_list`/`idb_close`). Legacy 1.4.0 `idalib-mcp` will not work. Add `"env": { "IDADIR": "..." }` if your installation needs it. A local Streamable HTTP supervisor (`"url": "http://127.0.0.1:8745/mcp"`) is also accepted.
- REA copies the binary into a **private owner-only workspace**, verifies its digest, requests an owned worker with `force_headless`, and on close releases it with `save: false` and confirms it is gone. If release cannot be verified, the workspace is kept and cleanup is reported as incomplete.
- REA and the supervisor must share a filesystem and path syntax: if one runs on Windows, the other must too.
- Optional keys: `workspaceRoot` (an existing absolute temp parent) and `timeoutMs` (default 300000). Stdio registrations default `IDA_MCP_MAX_WORKERS` to `1`.

## Connect an agent

Keep the upstream registration in its own file. Add only the path to the REA entry:

```json
{ "mcpServers": { "rea": { "command": "npx", "args": ["-y", "rea-agents@<version-with-ida>", "mcp"],
  "env": { "REA_ANALYSIS_PROVIDER": "ida", "REA_IDA_MCP_CONFIG": "/absolute/path/to/ida-mcp.json" } } } }
```

Setting `REA_IDA_MCP_CONFIG` before `rea setup --client <client>` makes setup include the file reference in its plan. Setup does not copy upstream credentials.

## Coverage

| Facet | Available |
| --- | --- |
| Procedure/string inventory, literal and regex search, procedure resolution | yes |
| Pseudocode, assembly, raw instructions, callees, incoming xrefs, dossier | yes |
| Direct callers | legacy profile only |
| Body ranges, typed refs, CFG, locals, comment kinds, DB revision | no (reported as **unknown**) |
| `binary_overview`, mutation, debugging, arbitrary Python | out of scope |

A freshly opened headless worker can report `status: "busy"`. REA polls every 100 ms until `timeoutMs`, and only then returns a retryable `provider_timeout`. IDA rejects MIPS/RISC-V ELF, analysis databases as input, and raw binaries; use [[Ghidra]] for those.

## Verification

```bash
npm run verify:ida -- --target /absolute/path/to/program --procedure main
```

Native mode also needs `--second-procedure` and `--second-marker`. Optional oracles: `--expected-string`, `--expected-xref`, `--package-root`, `--report`. Verified so far: legacy GUI and headless IDA 9.3 on Windows x64, headless IDA Pro 9.1 on macOS arm64, and native mode with IDA Pro 9.3 on Windows plus IDA Free 9.4 on Linux (with a locally patched plugin). Linux headless and the modern attached GUI tools remain unverified.

See also [[Providers Overview]], [[Agent Setup]], [[Environment Variables]].

## Source
- https://github.com/morluto/rea/blob/main/docs/ida-provider.md
- https://github.com/morluto/rea/blob/main/docs/testing.md#ida-mcp-adapter
