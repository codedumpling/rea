---
tags: [rea, provider, jeb, android]
source:
  - docs/jeb-analysis.md
  - docs/testing.md
  - src/jeb/
  - src/cliCommandNames.ts
---
# JEB

[JEB](https://www.pnfsoftware.com/) is a commercial decompiler, strongest on Android and Java targets. REA reaches it through **JEB's own MCP interface** and exposes project, unit and decompilation inspection through the CLI and MCP. JEB is **bring-your-own**: REA never installs or launches it, and never opens its project databases directly.

JEB is an **auxiliary capability** with its own four tools. It is not a deep `open_binary` provider and is not part of [[Providers Overview|provider selection]].

## Connect

Start a JEB client that serves MCP, then point REA at it:

- Default endpoint: `http://127.0.0.1:8425/mcp`
- Override it with `REA_JEB_MCP_URL`. It must be an absolute `http`/`https` URL whose literal host is `127.0.0.1` or `[::1]`. Redirects, remote hosts, credentials, query strings and fragments are **rejected**, not silently dropped.
- **GUI:** start the MCP server from the *File* menu, or through the VIBRE assistant.
- **Headless:** use JEB's programmatic route (`JebMcpServerInstance` from the JEB API) with the JEB client on your classpath. JEB 5.48's `-c` headless mode does not accept `--mcp` directly.

Verified against **JEB 5.48.0** (Linux x64, licensed build) serving `jeb-mcp-server` 1.3.0 over streamable HTTP.

## Operations

| MCP tool | CLI command | Purpose |
| --- | --- | --- |
| `inspect_jeb_client` | `inspect-jeb-client` | Engine version, GUI or headless mode, startup time |
| `open_jeb_project` | `open-jeb-project` | Open or create a project from an artifact or `.jdb2` on the engine host. Returns input SHA-256 digests and top-level units |
| `list_jeb_units` | `list-jeb-units` | Page through units by stable path and type (`filter`, `parent-unit-path`, `index`, `count`) |
| `decompile_jeb_item` | `decompile-jeb-item` | Decompile one type or method to engine pseudo-code |

## Guarantees and semantics

- All four tools are **read-only with respect to the target file**. Only `open_jeb_project` changes engine-side project state.
- Evidence records the endpoint, the parameters you chose and the digests the engine reported.
- Renames, comments, `save_project` and `close_project` are intentionally **not** exposed.
- Paths given to `open_jeb_project` are resolved by the JEB process. The artifact does not need to exist on the REA host.
- The engine caps a `list_units` page at 100 entries. A full page is reported as `coverage: "partial"`.
- If `decompile_jeb_item` omits `unit_path`, the engine picks the first code unit. REA reports `unit_path: null` with a limitation rather than guessing which unit was used.
- Engine refusals (`success: false`) come back as unsupported-target errors carrying the engine's own message.
- An unreachable endpoint is a provider-availability failure with recovery guidance. REA does not retry and does not spawn JEB.

> [!warning] Headless JEB cannot open projects
> JEB 5.48.0 headless MCP instances do not advertise `open_project`. Select the target when the engine starts (`--infile` on the programmatic launcher, or open it in the GUI). REA passes the engine's refusal through to you.

## Verification

`npm run verify:jeb` needs a JEB client you started yourself, serving MCP at `REA_JEB_MCP_URL`, with a project already open. It checks client inspection, unit-listing coverage and method decompilation, and records the engine's exact response to `open_jeb_project`. Mock transport tests do not establish compatibility with a real engine. See [[Testing]].

See also [[Android]], [[External Tools]].

## Source
- https://github.com/morluto/rea/blob/main/docs/jeb-analysis.md
