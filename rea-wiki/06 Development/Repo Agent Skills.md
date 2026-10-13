---
tags: [rea, development, skills, agents]
source:
  - .agents/skills/rea-changelog-update/SKILL.md
  - .agents/skills/rea-tool-design/SKILL.md
  - .agents/skills/reverse-engineer-anything/SKILL.md
  - AGENTS.md
  - CONTRIBUTING.md
  - website/AGENTS.md
  - experiments/analysis-efficiency/README.md
---
# Repo Agent Skills

The repository is set up for AI coding agents working **on REA itself**. Root `AGENTS.md` (plus `website/AGENTS.md` for the site) holds the invariants. `.agents/skills/` holds three authored skills.

| Skill | Audience | Purpose |
| --- | --- | --- |
| `reverse-engineer-anything` | **End users' agents** (shipped) | The product workflow skill that teaches an agent how to use REA |
| `rea-tool-design` | REA contributors | Designing or changing tools, CLI/MCP contracts, provider capabilities and Evidence semantics |
| `rea-changelog-update` | Maintainers | Preparing release changelogs and GitHub release notes with verified contributor thanks |

## `reverse-engineer-anything` (the shipped skill)

- Authored in `.agents/skills/reverse-engineer-anything/SKILL.md` (metadata `version: "41"`), with on-demand references in `references/`: `android-applications.md`, `connection-and-recovery.md`, `evidence-workflows.md`, `javascript-applications.md`, `native-and-artifacts.md`, `runtime-observation.md`.
- It tells agents to use REA when a claim depends on a shipped binary or package, decompilation, runtime evidence or version comparison, and **not** for ordinary analysis of a complete source repository.
- The build (`npm run build:cached`) bundles it into the ignored `skills/` output. `rea setup` installs it into each selected client's personal skill directory. `rea doctor --skill --json` checks the installed copies.
- `npm run verify:package` installs the bundle with Skills 1.7.2 using default repository-root discovery, so a clean checkout is directly discoverable.

User-facing details are in [[Agent Skill]] and [[Agent Setup]].

> [!warning] Edit the source, not the output
> Never edit `skills/` (generated and ignored). Edit `.agents/skills/reverse-engineer-anything/` and regenerate. After a managed-skill version change, run `npm run metadata:generate` before building.

## `rea-tool-design`

This is a thin router to [[Tool Design Principles]] (`docs/tool-design.md`). Before changing anything, inspect the current contract and the nearest existing tool. For a design request, return the proposed boundary, input/result shape, Evidence semantics, nearest alternative and verification approach. For implementation, consult [[Testing]] for the affected lanes and [[Contributing]] for generated-file ownership.

## `rea-changelog-update`

This skill prepares notes. It does **not** authorize merging, tagging or publishing.

1. **Inventory before prose.** Fetch complete history and tags, take the baseline tag (`rea-agents-PREVIOUS`) from `.release-please-manifest.json` on the release PR's base branch, and pin that branch's SHA:
   ```bash
   node scripts/release-notes.mjs inventory \
     --repository morluto/rea --base rea-agents-PREVIOUS \
     --target FULL_BASE_BRANCH_SHA --version VERSION \
     --manifest /tmp/rea-VERSION.contributions.json
   node scripts/release-notes.mjs record \
     --manifest /tmp/rea-VERSION.contributions.json --summary \
     --output /tmp/rea-VERSION.record.md
   ```
2. **Write for users.** Rewrite only the newest `## [VERSION]` section under `### Highlights`, `### Changes`, `### Fixes` (and `### ⚠ BREAKING CHANGES` when needed), grouped by analyst workflow.
3. **Retain credit.** Use inline `Thanks @...`. Never guess a handle for an unresolved identity, and never thank the merger as a substitute.
4. Finish with `check` and `render`, following [[Releasing]].

## Agent-facing invariants (AGENTS.md)

- REA is local-only. Observations, derivations, inferences and unknowns must stay distinct.
- Dependencies flow inward (see [[Codebase Map]]). Ghidra and IDA are bring-your-own. IDA analysis never saves or closes an attached GUI database.
- Toolchain: `.nvmrc` plus `package.json#packageManager`. Core commands: `npm ci`, `npm run build:cached`, `npm run test:local -- PATH...`, `npm run test:focused -- PATH...`, `npm run check:fast`, `npm run docs:check`.
- Turborepo note: resolve the installed `turbo` package and read its bundled docs before changing Turbo config. Its behavior can differ from what you may remember.

## Experiments

`experiments/analysis-efficiency/` tracks measured token, time and memory studies (issue #2016) that support the performance priorities. Examples include native entry views, EVM descriptors, managed artifact summaries, preset lifecycles and IDA native Windows measurements. It contains its scripts (`measure.mjs`, `compare.mjs`, `agent-trial.py`, `resource-run.py`) and `corpus.json`. Generated `results/` and downloaded artifacts stay out of the checkout.

## Source
- https://github.com/morluto/rea/blob/main/.agents/skills/rea-changelog-update/SKILL.md
- https://github.com/morluto/rea/blob/main/.agents/skills/rea-tool-design/SKILL.md
- https://github.com/morluto/rea/blob/main/.agents/skills/reverse-engineer-anything/SKILL.md
- https://github.com/morluto/rea/blob/main/AGENTS.md
