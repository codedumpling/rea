---
tags: [rea, reference, faq]
source:
  - README.md
  - docs/installation.md
  - docs/mcp-contracts.md
  - docs/cli.md
  - DISCLAIMER.md
  - SECURITY.md
---
# FAQ

## General

**What is REA, in one sentence?**
A local CLI and MCP server that lets your coding agent reverse engineer binaries, applications and runtime behaviour, returning findings with evidence and limitations. See [[What is REA]].

**Which agents can use REA?**
Any agent that supports local (stdio) MCP servers. `rea setup` configures 20+ clients automatically (Claude Code, Codex, Cursor, Gemini CLI, Grok Build, and more); others use manual registration. See [[Agent Setup]].

**Does REA upload my app?**
No. REA analyzes targets locally. But your agent receives the tool results, and the agent's model provider has its own data policy, so treat results as shared with that provider.

**Is REA free?**
REA-authored code is MIT-licensed. Commercial engines (Hopper, IDA, JEB) are separate software you must obtain and license yourself. See [[Legal and Responsible Use]].

## Setup

**Do I need Hopper, Ghidra or IDA?**
Only for deep native analysis (decompilation of executables). Static JavaScript/Electron and .NET inspection need no native engine. Setup can install Hopper after approval; Ghidra and IDA use your existing installations. See [[Providers Overview]].

**Do I need to start Hopper first?**
No, REA starts Hopper when an operation needs it. On macOS, a first-run dialog may ask you to pick demo mode or activate a licence: complete it in the same user session before your first CLI/MCP operation or unattended job. See [[Hopper]].

**What does installing the skill from skills.sh do?**
`npx skills add morluto/rea --skill reverse-engineer-anything` installs instructions only. It does not register the MCP server or install engines. Use `npx rea-agents setup` for both, then restart your agent. See [[Agent Skill]].

**I ran setup but my agent has no REA tools.**
Restart the agent. If still missing, `rea doctor --client <id> --json`. See [[Troubleshooting]].

**Does setup touch my other MCP servers or settings?**
It preserves unrelated entries and comments, writes backups (`.rea.backup`), uses atomic replacement, and writes nothing until you approve. Malformed config blocks the whole transaction. It never installs Node.js, npm, Homebrew, Java or Ghidra.

**Which Node.js versions are supported?**
22.x (>=22.19), 24.x (>=24.11), 26+. Not 23, 25 or prereleases. SQLite inspection needs Node 24.x >=24.15 or 26+.

**Does REA work on Windows?**
Partly. Setup, static JavaScript/.NET analysis and many readers work. Native analysis on Windows uses the experimental, read-only Windows Ghidra P0 boundary; Hopper is unavailable; some features (historical-source import, several Linux-only tool adapters) are refused. See [[Ghidra Variants]] and [[Targets Overview]].

## Using REA

**What code does REA return?**
Native analysis returns pseudocode and assembly. JavaScript/Electron analysis recovers modules and their relationships. Your agent uses these findings to write and test an implementation. See [[Showcases]].

**Is the pseudocode the original source?**
No. Provider pseudocode is never treated as original source, and decompilation may be incomplete or wrong. Verify consequential conclusions against the evidence. See [[Evidence Model]].

**Why does a tool say "unknown" instead of "none"?**
Because REA distinguishes absence from unexamined coverage. An empty call list with partial coverage is not proof of no calls.

**My JavaScript Evidence is hundreds of MB. What now?**
Save `--json` output to a file and project a `summary` or `page` view with `rea inspect-analysis-view`. In MCP, use `inspect_analysis_view` on the retained Evidence. See [[Evidence Export and Import]].

**The agent got `resource_constraint`.**
The result exceeded the 10 MiB stdio budget. The full Evidence is retained; follow `details.reported_limits.evidence_reference` with `inspect_analysis_view` or `export_evidence_bundle`. Clients with larger buffers can raise `REA_MCP_MAX_RESPONSE_BYTES`. See [[MCP Server]].

**My Ghidra query times out.**
The first Ghidra-backed query does import plus auto-analysis. Raise `REA_GHIDRA_STARTUP_TIMEOUT_MS` in the server environment and your client's tool timeout. See [[Ghidra]].

**How do I choose between Hopper and Ghidra when both are installed?**
`--provider ghidra` on the CLI, `provider_id: "ghidra"` on `open_binary`, or `REA_ANALYSIS_PROVIDER=ghidra` as a standing preference. REA never silently falls back to another provider.

**Can I reuse analysis between CLI runs?**
Yes, with snapshots: `rea analyze PATH --provider ghidra --snapshot /absolute/path/snap.json`. Exact repeat queries are served from the snapshot before any provider starts. See [[Sessions and Snapshots]].

**Does REA modify the binary?**
No executable bytes are modified. Ghidra uses ephemeral read-only projects; session metadata edits (names, comments) are discarded on close. IDA analysis is read-only. Hopper annotations apply to its document.

**Can text inside a target hijack my agent?**
REA marks target-derived results with `_meta.provenance.untrusted_content: true` and instructs agents never to follow directives from tool results. Still review agent plans, especially for runtime tools. See [[MCP Server]].

## Maintenance

**How do I update?**
`rea update` for a global npm/Bun install, then run the setup command it prints; or `npx rea-agents@latest setup`. Restart agents afterwards. See [[Updating and Uninstalling]].

**Why didn't updating the CLI update my agent?**
Agent registrations are pinned to the exact version that ran setup. Rerun setup and restart.

**Do minor versions break things?**
They can. The project uses `always-bump-minor` even for breaking releases. Check `CHANGELOG.md`'s breaking-changes section.

**How do I remove REA?**
`rea uninstall` (add `--purge-data` to remove `~/.rea` caches/state). Hopper, Node, Evidence files, captures and unrelated skills/servers are preserved.

## Bugs, security, legality

**What should I do if I hit a bug?**
Update, restart, retry. If it persists, open an issue with REA version, target type, steps and error output.

**How do I report a vulnerability?**
Privately, via **Report a vulnerability** in the repo's Security tab. Never in a public issue.

**Is reverse engineering with REA legal?**
It depends on your jurisdiction, licences and authorization. REA is for lawful research and software you own or are authorized to examine; the project will not help with DRM circumvention, cheats, auth bypass, etc. See [[Legal and Responsible Use]].

## Source
- https://github.com/morluto/rea/blob/main/README.md#faq
- https://github.com/morluto/rea/blob/main/docs/installation.md
- https://github.com/morluto/rea/blob/main/DISCLAIMER.md
