---
tags: [rea, development, release]
source:
  - docs/releasing.md
  - CONTRIBUTING.md
  - .github/workflows/release.yml
  - release-please-config.json
  - .agents/skills/rea-changelog-update/SKILL.md
---
# Releasing

Releases are automated with **Release Please** and published from the exact reviewed merge commit, to npm (`rea-agents`) and the official MCP Registry (`io.github.morluto/rea`).

## Automatic path (default)

1. Every push to `main` opens or refreshes a **Release Please PR** with the next version and changelog.
2. Review its migration notes and wait for the current CI checks to pass.
3. **Merge the PR into main.** This starts publication.
4. The `Release` workflow validates the release metadata and ancestry, creates the tag (`rea-agents-VERSION`), and both publishers build **that exact SHA**. Later commits on main cannot change it.
5. Check both publication jobs and the published-package canary. A merged PR or tag alone does not prove publication finished.

Ordinary main pushes and closed-but-unmerged PRs can never publish.

## Versioning policy

REA uses the `always-bump-minor` strategy: every release bumps the **minor** version and resets patch, **including breaking releases**. So 6.0.0 is followed by 6.1.0. The SemVer format does not promise compatibility on a minor bump. Always read the changelog's breaking-change section.

> [!warning] Breaking-change markers
> Before adding `!` or a `BREAKING CHANGE` footer, a PR must name a previously valid call or configuration that will now fail or change meaning, explain why compatibility cannot be preserved, and give the migration. Prefer optional additions, adapters and deprecation periods. The supported public surface is: documented CLI commands and options, MCP tool names and input/result contracts, saved Evidence formats, and runtime requirements.

## Contributor-aware release notes

Use the `rea-changelog-update` skill ([[Repo Agent Skills]]) together with `scripts/release-notes.mjs`:

```bash
npm run release:notes -- inventory   # Git range + paginated GitHub PR/author/issue metadata
npm run release:notes -- record --summary
npm run release:notes -- check
npm run release:notes -- render
```

- Rewrite the newest `CHANGELOG.md` section into **Highlights / Changes / Fixes** (plus `⚠ BREAKING CHANGES`) with inline `Thanks @...` credit. Credit each contribution once, and leave out maintainer `@morluto` self-thanks.
- Save the finalized inventory as `docs/releases/VERSION.contributions.json` on the release branch.
- Add the `rea:release-notes-finalized` label before editing the live PR, so main-push automation won't overwrite your edits.
- Before tagging, the workflow checks that the inventory target equals the merge's first parent, compares the PR body with the changelog, and rejects changes outside release-metadata paths.

## Manual checkpoint path (optional)

Use this when a release needs an independently frozen application commit:

```bash
git branch release/5.0.0 SOURCE_SHA && git push origin release/5.0.0
gh workflow run release.yml --ref main -f release_branch=release/5.0.0 -f phase=prepare
# review, approve CI on the final bot head, merge the bot PR into release/5.0.0
gh workflow run release.yml --ref release/5.0.0 -f release_branch=release/5.0.0 -f phase=publish
```

- Branch names use exact SemVer (`release/6.1.0`, `release/6.1.0-rc.1`). Package, lockfile, manifest, registry metadata and changelog must all agree.
- The optional `RELEASE_PLEASE_TOKEN` secret lets bot PRs trigger CI automatically. With the default `GITHUB_TOKEN`, a maintainer must approve the runs.
- Stable versions publish to npm `latest`; prereleases publish to `next`.
- Publication builds the bundled Windows controls, verifies the packaged artifact, publishes to npm, runs the published CLI/MCP canary and update-path checks, then publishes MCP Registry metadata.
- **Sync back to main** with a **merge commit** (not squash or rebase), so the tag stays in main's ancestry. Then check with `git merge-base --is-ancestor rea-agents-5.0.0 origin/main`. The sync must include `.release-please-manifest.json`, `package.json`, `package-lock.json`, `CHANGELOG.md`, `server.json` and versioned doc examples, followed by `npm run docs:generate` and `npm run docs:check`.

## Partial publication

Re-run the failed jobs **in the original run**, so the release SHA stays fixed. If npm was already published, the npm job verifies that version instead of publishing again. Never move a tag, delete a release or unpublish from npm as a retry. A defective package needs a reviewed fix and a new version.

Record these separately: the GitHub release and tag SHA, the npm version and integrity (canary passed), and the MCP Registry version matching npm.

See also [[CI Workflows]], [[Updating and Uninstalling]], [[Contributing]].

## Source
- https://github.com/morluto/rea/blob/main/docs/releasing.md
- https://github.com/morluto/rea/blob/main/.github/workflows/release.yml
