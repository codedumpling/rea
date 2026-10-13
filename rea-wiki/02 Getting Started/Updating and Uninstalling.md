---
tags: [rea, getting-started, maintenance]
source:
  - README.md
  - docs/installation.md
  - docs/migration-6.2.md
  - src/cli/setupCommands.ts
  - CHANGELOG.md
---
# Updating and Uninstalling

REA changes quickly and new releases frequently include bug fixes, so the README's first advice for any bug is **update first**. Updating has two independent parts: the **CLI installation** and the **agent registrations**, which are pinned to the exact version that performed setup.

## Update a global CLI

For a CLI installed with `npm install --global` or `bun add --global`:
```bash
rea update
```
- Updates only the npm prefix or Bun global directory that owns the running CLI, installing the exact resolved release and checking the new executable's version before reporting success.
- Uses npm's (or Bun's) configured registry.
- Bun global updates need **Bun 1.2.17+** (for `bun info`). With an older or unparseable Bun version, the updater stops and tells you to upgrade Bun manually; it never upgrades Bun itself. Bun commands run inside the owning Bun global directory (`BUN_INSTALL_GLOBAL_DIR`, else `$BUN_INSTALL/install/global`, else `$XDG_CACHE_HOME/.bun/install/global`, else `~/.bun/install/global`), never in your current project.
- It does **not** reopen onboarding. It returns a maintenance plan that selects only your existing REA registrations and installed skill. **Run the printed scoped setup command** to refresh them, then restart affected agents.

> [!warning] Source checkouts and package-runner copies
> `rea update` cannot update a source checkout or an `npx`/`bunx` cache. Update those through the mechanism that owns them.

## Update via package runner

```bash
npx rea-agents@latest setup
```
Review the setup plan (it migrates unversioned or older managed registrations) and restart your agents. For one-off CLI commands use `npx rea-agents@latest <command>`; with Bun, `bunx rea-agents@latest`.

> [!info] Why restart?
> Installing newer instructions does not update a running server or its registration. The agent must reconnect to launch the newly pinned version.

Verify afterwards:
```bash
npm view rea-agents dist-tags.latest
rea doctor --client claude_code --json
```
Or ask the agent to call `binary_session` and compare its reported package version.

## Uninstall

```bash
rea uninstall
rea uninstall --purge-data
```

| Removed by `rea uninstall` | Preserved |
| --- | --- |
| REA-owned agent registrations (across all supported clients; no `--client` flag) | Hopper, Node.js |
| REA skill files in REA's managed shared and Claude Code personal skill locations | Evidence files and captures |
| With `--purge-data`: REA's cache and state under `~/.rea` | Unrelated skills and other MCP servers |

Safety behaviour:
- A malformed, unreadable or unsafely located client config stops the operation **before** anything is removed.
- A client that fails mid-update stops the remaining removals.
- A purge path that is a symbolic link is retained and reported, not followed.
- Grok Bot's account connector is not removed (it was added manually in chat).

To remove the global package itself afterwards, use your package manager (e.g. `npm uninstall --global rea-agents`).

## Upgrading across breaking releases

The repository uses `always-bump-minor` versioning, including for breaking releases: **a minor version bump does not imply backward compatibility.** Check the `⚠ BREAKING CHANGES` section of `CHANGELOG.md` before upgrading custom consumers, and refresh cached MCP tool definitions.

### 6.1 → 6.2 highlights (docs/migration-6.2.md)
- Evidence-producing tools return the complete Evidence record directly in `structuredContent`; the old `{ result, evidence_id, evidence }` wrapper is gone. Read `normalized_result` and `evidence_id`.
- Errors have `isError: true` with the diagnostic as JSON text and no `structuredContent`.
- Every Evidence record requires `analysis_profile` (object or `null`); old records cannot be patched by inserting `null` because it changes identity. Regenerate.
- `binary_session` dropped `provider`/`providers` aliases; use `analysis_provider_binding`, `analysis_provider_candidates`, `capabilities`.
- Snapshots require `workflow_entries` with profile-bound Evidence; save new snapshots to a **new path**, e.g.
  ```bash
  rea analyze /absolute/path/to/program --provider ghidra --snapshot /absolute/path/to/analysis/program-6.2.json
  ```
- Artifact inventory per-path facts moved to `occurrences[]` (`artifact_kind`, `artifact_format`, `executable`).
- Legacy v3 process captures are no longer converted on import; recapture.
- Function comparison statuses are `unchanged`, `changed` or `unknown` (no `truncated`).

### 6.4.0 breaking changes (CHANGELOG)
Include `network.websocket_connections` replacing `network.websocket_events`, saved reference-graph exclusions requiring `pattern`, terminal-capture buffer limits, artifact graph edges reading child IDs via `occurrence_id`, and semantic-graph identity changes. Earlier Evidence may no longer validate against the new schemas; regenerate rather than edit.

> [!tip] Keep originals
> Keep old saved artifacts until replacements and consuming workflows are checked. Fresh analysis may establish different facts; it does not rewrite what an older capture proved.

## Related
- [[Installation]] · [[Agent Setup]] · [[Readiness Checks]] · [[Releasing]] · [[Roadmap]]

## Source
- https://github.com/morluto/rea/blob/main/docs/installation.md#diagnose-update-and-remove
- https://github.com/morluto/rea/blob/main/docs/migration-6.2.md
- https://github.com/morluto/rea/blob/main/CHANGELOG.md
