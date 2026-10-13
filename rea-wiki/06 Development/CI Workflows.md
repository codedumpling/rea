---
tags: [rea, development, ci]
source:
  - .github/workflows/
  - docs/testing.md
  - docs/releasing.md
  - CONTRIBUTING.md
  - scripts/ci/
---
# CI Workflows

All workflows live in `.github/workflows/`. The main `CI` workflow classifies changes first (`scripts/ci/plan.mjs`, `scopes.mjs`) and runs only the relevant lanes. One aggregate **`CI required`** job is the merge gate. Different kinds of evidence, such as static checks, tests, docs, package and real systems, run as separate jobs, so one can never stand in for another.

## `ci.yml` — CI

Triggers: push to `main`, pull requests to `main` and `release/*`, and manual dispatch.

| Job | What it does |
| --- | --- |
| Conventional PR title | Checks the PR title (PRs only) |
| Classify changes | Computes scopes and the platform matrix |
| Static checks | typecheck, lint, format, knip, metadata, jscpd, TODO scan, architecture guards; reports uploaded as `static-check-reports` |
| Test shard (1/6 … 6/6) | Six native Vitest shards, no retries |
| Test (coverage + timing) | Merges shard blobs into coverage, JUnit and timing; publishes `test-timings` on complete success |
| Build package | Builds and saves the portable Turbo cache |
| SQLite runtime | Calls `sqlite-runtime.yml` |
| Source-built conformance fixtures | Builds native fixtures when relevant |
| Build bundled Windows x64 native controls | Builds the Node-API artifact (on Linux) |
| Native package baseline (platform/arch matrix) | Installs the package and checks the baseline on hosted runners |
| Real Apple artifact CLI/MCP E2E | Apple metadata lanes |
| Reconstruction readiness | `npm run verify:readiness`; full report kept as an artifact |
| Windows capability and package lane | `windows-latest`: curated tests, native inspection, Node Inspector, path boundaries on Node 22/24 |
| Docs Generation / Validate workflows | Generated docs and workflow linting |
| Reusable real lanes | `real-browser`, `real-binary-layout`, `real-evm-interface`, `real-javascript-recovery`, `real-recorded-crash`, `real-web-network-captures`, `real-web-runtime`, `real-web-source-map`, `nativeaot-fixtures`, `website-check`, each gated on its change scope |

## Real-system workflows

| Workflow | Trigger | Runner |
| --- | --- | --- |
| `real-hopper.yml` (Real Hopper E2E) | manual | self-hosted macOS + hopper |
| `real-hopper-linux.yml` (Real Hopper Linux E2E) | manual | self-hosted Linux + hopper |
| `real-ghidra-windows.yml` (Real Ghidra Windows E2E) | `repository_dispatch` (`real-ghidra-windows`) | Linux artifact build, then a protected self-hosted Windows x64 runner (default-branch code only) |
| `real-go-binary.yml` | manual + PR path filters | hosted |
| `real-packet-capture.yml` | manual + PR path filters | hosted (TShark) |
| `real-signal-recording.yml` | manual + PR path filters | hosted (sigrok) |
| `real-termux.yml` | manual | Termux Android browser smoke test |
| `real-browser.yml`, `real-binary-layout.yml`, `real-evm-interface.yml`, `real-javascript-recovery.yml`, `real-recorded-crash.yml`, `real-web-network-captures.yml`, `real-web-runtime.yml`, `real-web-source-map.yml` | `workflow_call` from CI and/or manual | hosted |
| `nativeaot-fixtures.yml` | call/manual | source-built NativeAOT fixtures |
| `sqlite-runtime.yml` | call | SQLite runtime compatibility |

> [!info] Licensed engines are not in hosted CI
> Real Hopper and Windows Ghidra lanes need self-hosted runners that already have the engine provisioned. Hosted Windows CI verifies native controls and packaged startup separately from real Ghidra acceptance. Any provider claim in a PR still needs the matching `verify:*` lane, run and reported (see [[Testing]]).

## Release and docs

| Workflow | Purpose |
| --- | --- |
| `release.yml` (Release) | On main push: refreshes the Release Please PR. On merged PR close: validates, tags and publishes to npm and the MCP Registry. Manual dispatch with `release_branch` + `phase` (`prepare`/`publish`) for frozen checkpoints. See [[Releasing]] |
| `pages.yml` (Documentation build) | Manual VitePress build of `docs/`. Does **not** publish |
| `website-check.yml` (Website checks) | Static site checks, called from CI |
| `website-pages.yml` (Publish REA website) | Manual. `target` = `both` or `pages`. Publishes `website/` to GitHub Pages and/or Cloudflare. See [[Website]] |

## Caching

The build job saves portable Turbo outputs keyed on `test:prepare --dry=json` task hashes, runner OS and architecture, `.npmrc`, `.nvmrc` and the lockfile. Eight Linux real-system lanes restore this cache without waiting for the build job. Native outputs, dependency installs and package archives stay host-local. Every command still runs, and Turbo rebuilds on a cache miss.

See also [[Testing]], [[Releasing]], [[Codebase Map]].

## Source
- https://github.com/morluto/rea/blob/main/.github/workflows/ci.yml
- https://github.com/morluto/rea/blob/main/.github/workflows/release.yml
- https://github.com/morluto/rea/blob/main/docs/testing.md#coverage-and-timing
