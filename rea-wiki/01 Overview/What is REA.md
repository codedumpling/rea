---
tags: [rea, overview]
source:
  - README.md
  - AGENTS.md
  - docs/index.md
  - package.json
---
# What is REA

**REA (Reverse Engineer Anything)** is a local-only reverse-engineering toolkit that exposes one set of analysis workflows through two adapters: an **MCP server** that coding agents call, and a **CLI** (`rea`) you can use directly from a terminal or script. It is published on npm as `rea-agents` (current release 6.4.0), and in the official MCP Registry as `io.github.morluto/rea`.

The project's one-line pitch: *see a feature you like, understand how it works, down to the binary level.* You point your agent at an application you do not have the source for, ask how a feature works, and REA gives the agent the decompiled code, module graphs, references and runtime observations it needs, along with the evidence and limitations behind each conclusion.

> [!info] Key facts
> - Package: `rea-agents` (binaries `rea` and `rea-agents`, both `scripts/rea.mjs`)
> - License: MIT for REA-authored code; see [[Legal and Responsible Use]]
> - Runtime: Node.js 22.x (>=22.19), 24.x (>=24.11) or 26+, plus npm
> - Entry point: `npx rea-agents setup`
> - Website and guides: https://rea.tools/

## What problem it solves

Reverse engineering usually means juggling a disassembler, a decompiler, a browser's DevTools, archive extractors and a notebook of findings. REA wraps those engines behind consistent, schema-validated tools so that an AI agent (or a script) can:

1. Inspect a shipped artifact without its source: native binaries, Electron/JavaScript apps, .NET assemblies, Android APKs, firmware, websites and more. See [[Targets Overview]].
2. Trace the code relevant to a question (call graphs, references, IPC paths, module imports).
3. Return findings as **Evidence** records that separate what was observed from what was inferred, and record what remains unknown. See [[Evidence Model]].
4. Let the agent explain the behaviour, or write and test a reimplementation in your own project.

## What it is not

- **Not a sandbox.** Opening an untrusted binary delegates parsing to the selected local provider with your user's permissions. Runtime-capture tools actually run or attach to the target. See [[Legal and Responsible Use]].
- **Not a disassembler of its own** for deep native analysis. Native decompilation is delegated to a bring-your-own provider: [[Hopper]], [[Ghidra]] or [[IDA]] (plus [[JEB]] for Android). See [[Providers Overview]].
- **Not a cloud service.** Analysis runs locally. Your agent receives the tool results, and the agent's model provider has its own data policy.
- **Not an installer of unrelated software.** Setup never installs or upgrades Node.js, npm, Homebrew, Java, Ghidra or IDA. Hopper is the only engine setup can install, and only with explicit approval.

## The two adapters

| Adapter | How you use it | Notes |
| --- | --- | --- |
| MCP server | Your agent launches `rea-agents mcp` over stdio after [[Agent Setup]] | One long-lived connection holds sessions, retained Evidence and unknowns. See [[MCP Server]]. |
| CLI | `rea <command> ...` or `npx -y rea-agents@latest <command> ...` | Each invocation is a separate process; save results to files to reuse them. See [[CLI Reference]]. |

Both adapters share the same application workflows and evidence contracts, so a CLI result and an MCP result for the same operation carry the same Evidence shape.

## What you can analyze (summary)

| Family | Typical output | Extra requirement |
| --- | --- | --- |
| Native binaries | Pseudocode, assembly, strings, symbols, calls, references | Hopper, Ghidra or IDA |
| JavaScript / Electron | Modules, imports, source maps, routes, IPC, native add-ons | None beyond Node.js |
| Websites | Page structure, scripts, network, screenshots | A Chrome-family browser |
| .NET assemblies | Metadata, CIL, native dependencies | None (static only) |
| Android APKs / devices | Manifest, classes, decompiled methods; device state via adb | JADX + JDK, or adb, Apktool, JEB |
| Firmware, packets, signals, crashes, WASM, EVM, SQLite, Go | Format-specific facts | Usually a caller-supplied tool, often Linux-only |

The full matrix, with host requirements per target, lives in [[Targets Overview]].

## Who it is for

REA is built for lawful research: security and malware analysis, vulnerability research and coordinated disclosure, CTFs and education, interoperability, software preservation, and auditing software you own or are authorized to examine. The project explicitly refuses help with DRM/licence circumvention, cheats, and similar uses; see [[Legal and Responsible Use]].

## Where to go next

- Install and connect an agent: [[Quick Start]], then [[Installation]] and [[Agent Setup]]
- Understand the moving parts: [[How REA Works]], [[Architecture]]
- Learn the vocabulary: [[Glossary]]
- See real investigations: [[Showcases]]
- Common questions: [[FAQ]]

## Source
- https://github.com/morluto/rea/blob/main/README.md
- https://github.com/morluto/rea/blob/main/AGENTS.md
- https://github.com/morluto/rea/blob/main/docs/index.md
