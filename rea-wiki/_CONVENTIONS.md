---
tags: [meta]
---
# Vault conventions

This vault documents **REA (Reverse Engineer Anything)**, `morluto/rea`, package `rea-agents` v6.4.0.

## Folders
| Folder | Contents |
| --- | --- |
| `01 Overview` | What REA is, how it works, architecture, evidence model, glossary |
| `02 Getting Started` | Install, agent setup, updating, readiness checks, troubleshooting |
| `03 Usage` | CLI, MCP tools/contracts, agent prompts, sessions/snapshots/evidence, recipes |
| `04 Targets` | One note per target family (native, JS/Electron, browser, .NET, Android, …) |
| `05 Providers` | Hopper, Ghidra (+ variants), IDA, JEB, pwntools, external tools |
| `06 Development` | Codebase map, contributing, testing, tool design, releasing, CI, skills |
| `07 Reference` | Env vars, FAQ, legal/disclaimer, roadmap, doc index |

## Writing rules
- Every note starts with YAML frontmatter: `tags:` (e.g. `[rea, target]`), `source:` (list of repo paths the note is derived from).
- Link other notes with `[[Note Name]]` (file name without `.md`); link liberally, including to notes in other folders.
- Use callouts: `> [!tip]`, `> [!warning]`, `> [!info]`, `> [!example]`.
- Paraphrase repo docs; include exact commands, flags, tool names and env vars verbatim from source. Never invent a command, flag or tool name — verify it in the repo.
- Point to the canonical repo doc at the bottom of each note under `## Source`, as a GitHub link: `https://github.com/morluto/rea/blob/main/<path>`.
- Entry point: [[Home]].
