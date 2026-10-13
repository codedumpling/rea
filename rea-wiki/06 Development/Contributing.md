---
tags: [rea, development, contributing]
source:
  - CONTRIBUTING.md
  - AGENTS.md
  - SECURITY.md
---
# Contributing

REA welcomes focused bug fixes, documentation improvements, tests and reverse-engineering workflows. For large contract or architecture changes, explain the intended scope and link the relevant issue or design discussion first.

## Choosing a contribution

- Explain the concrete problem, keep related corrections together, and report the verification you ran **and the coverage you could not run**.
- Contributions must justify their review and maintenance cost. Bundle cosmetic polish with a useful fix. Confusing or incorrect user instructions are worth a standalone fix. Other cosmetic-only suggestions belong in an issue, not separate PRs.
- Follow the [[Codebase Map|architecture and layering]], the [[Tool Design Principles]] for MCP changes, and the performance priorities in AGENTS.md: agent tokens first, then analysis time, then CPU/memory.

## Contribution scope

**In scope:** format inspection, ordinary archive and compression extraction, disassembly, decompilation, xrefs, general static deobfuscation, vulnerability analysis, evidence reporting, security fixes and regression tests. Static *detection* of protections is in scope. *Defeating* protections is not.

**Not integrated or distributed** (this applies to first-party providers, plugins, bundled scripts, defaults, examples and docs alike):
- Purpose-built workflows that defeat commercial packing or code virtualization.
- Anti-debugging or anti-instrumentation evasion, and DRM, license, anti-cheat or access-control bypass.
- Exploit or payload automation, credential theft, covert persistence, unauthorized access, or bulk attack workflows.

Runtime work needs explicit boundaries: launching or attaching to processes, injecting instrumentation, running scripts, and changing memory or registers. Each needs a review of target selection, execution and mutation authority, provenance, limits, cancellation and cleanup. See [[Legal and Responsible Use]].

> [!warning] Fail-closed integration
> If scope fit, authorization boundaries, side effects or the required verification remain materially unresolved, **do not merge, enable or ship the capability**. Passing CI, finished code, upstream popularity or maintainer silence is not scope approval. A new provider, or any capability involving target execution, injection, mutation or protection recovery, needs an **explicit maintainer scope decision** before substantial implementation. Exploratory prototypes and draft PRs are welcome, but they do not authorize integration.

**Starting a proposal:** describe the problem and the outcome you want. You do not need code, a full design, legal analysis or proof of authorization to open a discussion. Before integration, maintainers and implementers settle: the analyst question and why existing operations fall short; scope fit and non-goals; every execution, injection, mutation, network or persistence effect (including upstream behavior); and how verification will use self-owned or properly licensed samples. Private vulnerability reports follow `SECURITY.md`.

## Development setup

```bash
nvm use            # toolchain pinned by .nvmrc and package.json#packageManager
npm ci             # exact deps + Husky hooks (no build)
npm run check:fast # cached typecheck + lint (also the pre-push hook)
npm run build:cached   # build CLI/MCP runtime + packaged skill
```

`npm run build` forces those tasks to rerun. After a package, lockfile or managed-skill version change, run `npm run metadata:generate` before building.

| Command | Use |
| --- | --- |
| `npm run test:local -- PATH...` | Source tests without building |
| `npm run test:focused -- PATH...` | Exact tests, building required artifacts first |
| `npm run check:changed` | Cached static checks plus tests affected since the merge base with `origin/main` |
| `npm run check` | Run before requesting review |
| `npm run docs:check` | When contracts or generated metadata change |
| `npm run check:pr` | Optional full local gate (broad changes or CI diagnosis) |
| `npm run verify:package` + `npm pack --dry-run` | Packaging, setup or distribution changes |
| `npm run verify:test-discovery` | After adding or moving tests |

Provider changes require the matching **real-provider lane** (for example `npm run verify:hopper`, `verify:hopper:linux`, `verify:ghidra`, `verify:ida`). State which host and provider workflows you ran. See [[Testing]].

## Generated files

- `docs:generate` prepares documentation and portable managed evidence. `evidence:generate` prepares only the managed commitment.
- Ignored outputs: `docs/public/product-catalog.json`, `docs/verification/managed-conformance-*.json`, `skills/`. CI validates them and keeps them as artifacts. **Do not commit them.**
- Tracked generated sources that are checked for freshness: `src/generatedPackageMetadata.ts`, `docs/error-contract.schema.json`.
- Edit authored skill instructions in `.agents/skills/reverse-engineer-anything/`. See [[Repo Agent Skills]].
- Build and generation commands hold a SQLite lock under `.cache/rea-command-locks/`. Never delete it while commands are running.

## Temporary files and logs

- Tests use `createTestTempDirectory` (`tests/fixtures/temporaryDirectory.ts`). `npm run verify:test-temp-hygiene` checks that no REA-owned paths are left behind. Never glob-delete shared `/tmp/rea-*`.
- `REA_LOG_LEVEL` = `trace|debug|info|warn|error|fatal|silent`. Logs go to stderr. MCP defaults to `info`; CLI logging is opt-in. Arguments, tokens and environment data are redacted.

## Commits and PRs

Use **Conventional Commit** subjects and PR titles (CI checks the PR title). Mark breaking changes with `!` or a `BREAKING CHANGE:` footer, and include a migration (see [[Releasing]]). Describe the behavior change, the validation you ran, and the scope of any real-provider verification. Pre-commit runs `lint-staged`; pre-push runs `check:fast`.

## Repository invariants (AGENTS.md highlights)

- Local-only tool. Observations, derivations, inferences and unknowns stay distinct, and Evidence provenance is preserved through both adapters.
- Configuration changes are additive, idempotent and backed up. Installers never install unrelated software (Homebrew, Node, Java, Ghidra). `rea setup` shows its plan and requires approval.
- Parse unknown values at the boundary into `Result` and the tagged errors. Keep identity values separate from display values. Retain ownership of any resource whose cleanup failed. Reuse `src/process/` supervision.
- When fixing a bug, fix the producing representation and remove superseded code. A passing regression test alone is not enough.

See also [[Codebase Map]], [[CI Workflows]], [[Website]].

## Source
- https://github.com/morluto/rea/blob/main/CONTRIBUTING.md
- https://github.com/morluto/rea/blob/main/AGENTS.md
