---
tags: [rea, moc]
source: [README.md, docs/index.md]
---
# REA — Reverse Engineer Anything

> [!info] What this vault is
> A navigable wiki for [morluto/rea](https://github.com/morluto/rea) (npm: `rea-agents`, v6.4.0, MIT). REA is **one CLI and one MCP server** that lets you, or your coding agent, inspect native binaries, JavaScript/Electron apps, websites, .NET assemblies, Android APKs, firmware and more. It runs **locally** and returns findings together with the evidence and limits behind each one.

```bash
npx rea-agents setup
```

## Start here
- [[What is REA]]: the pitch, the scope, and what it isn't
- [[Quick Start]]: from zero to a first result in five minutes
- [[How REA Works]]: agent → MCP → analysis tools → evidence
- [[Evidence Model]]: observations, derivations, inferences and unknowns

## Map of content

### 01 Overview
[[What is REA]] · [[How REA Works]] · [[Architecture]] · [[Evidence Model]] · [[Glossary]] · [[Showcases]]

### 02 Getting Started
[[Quick Start]] · [[Installation]] · [[Agent Setup]] · [[Updating and Uninstalling]] · [[Readiness Checks]] · [[Troubleshooting]]

### 03 Usage
[[CLI Reference]] · [[MCP Server]] · [[MCP Tool Catalog]] · [[Agent Prompts]] · [[Sessions and Snapshots]] · [[Evidence Export and Import]] · [[Recipes]] · [[Errors and Exit Codes]] · [[Agent Skill]]

### 04 Targets — *what can I analyze?*
[[Targets Overview]]
- Native: [[Native Binaries]] · [[Go Binaries]] · [[ELF Layout and Recorded Crashes]] · [[WASM Artifacts]] · [[EVM Bytecode]]
- Apps: [[JavaScript and Electron]] · [[JavaScript Runtime Observation]] · [[DotNET Assemblies]] · [[Android]] · [[Apple and Harmony Apps]] · [[Flutter Apps]]
- Web and runtime: [[Websites and Browser Observation]] · [[Network and Packet Captures]] · [[Process Capture]]
- Data and hardware: [[SQLite Databases]] · [[Firmware]] · [[Signal Recordings]]

### 05 Providers — *the engines underneath*
[[Providers Overview]] · [[Hopper]] · [[Ghidra]] · [[Ghidra Variants]] · [[IDA]] · [[JEB]] · [[Frida]] · [[External Tools]]

### 06 Development — *hacking on REA itself*
[[Codebase Map]] · [[Contributing]] · [[Testing]] · [[Tool Design Principles]] · [[Releasing]] · [[CI Workflows]] · [[Website]] · [[Repo Agent Skills]]

### 07 Reference
[[Environment Variables]] · [[FAQ]] · [[Legal and Responsible Use]] · [[Roadmap]] · [[Doc Index]]

## Which note do I need?
| I want to… | Go to |
| --- | --- |
| Hook REA into Claude Code, Codex, Cursor and similar agents | [[Agent Setup]] |
| Understand how a feature in a desktop app works | [[Recipes]], [[JavaScript and Electron]], [[Native Binaries]] |
| Decompile a function | [[Native Binaries]], then [[Ghidra]] / [[Hopper]] / [[IDA]] |
| See what a website does on the network | [[Websites and Browser Observation]], [[Network and Packet Captures]] |
| Script REA from a terminal or CI | [[CLI Reference]], [[Errors and Exit Codes]] |
| Know which tools my agent can call | [[MCP Tool Catalog]] |
| Fix "provider not found" or startup timeouts | [[Readiness Checks]], [[Troubleshooting]], [[Environment Variables]] |
| Contribute a fix | [[Contributing]], [[Codebase Map]], [[Testing]] |

> [!warning] Responsible use
> REA is for lawful research, interoperability, preservation, and analysis of software you own or are authorized to examine. Commercial engines (Hopper, IDA, JEB) need your own legitimate licenses. See [[Legal and Responsible Use]].

## Links
- Website: https://rea.tools/ · Guides: https://rea.tools/guides/ · Showcases: https://rea.tools/showcase/
- npm: https://www.npmjs.com/package/rea-agents
- Discord: https://discord.gg/GkcryMnJDM
