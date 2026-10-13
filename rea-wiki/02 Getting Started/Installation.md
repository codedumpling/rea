---
tags: [rea, getting-started, install]
source:
  - docs/installation.md
  - README.md
  - install.sh
  - package.json
  - SECURITY.md
---
# Installation

REA ships as the npm package **`rea-agents`**. You can run it through a package runner (`npx` / `bunx`), install it globally, or use the optional curl wrapper. Agent registration is a separate step covered in [[Agent Setup]].

## Requirements

| Item | Requirement |
| --- | --- |
| Node.js | 22.x (>=22.19), 24.x (>=24.11) or 26+. Node 23, 25 and prereleases are unsupported. |
| npm | The npm paired with that Node. REA never upgrades Node.js, npm or Homebrew. |
| Native host baseline (Hopper) | macOS 12+, Ubuntu 24.04+, Fedora 41+, Nobara 44+, 64-bit Arch Linux, CachyOS, or Omarchy |
| Ghidra / IDA | Provider-specific host requirements; Windows Ghidra uses the experimental P0 boundary |
| Target-specific tools | See [[Targets Overview]] and [[External Tools]] |

> [!info] Bun
> Bun works as a runner (`bunx rea-agents setup`). `rea doctor`'s `node` check reports Bun's declared Node.js compatibility version. Bun does not enforce the V8 heap flags REA gives helper processes, so Evidence records those limits as unavailable under Bun. CI tests Node.js only.

## Ways to install

### 1. Package runner (no install)
```bash
npx rea-agents setup            # guided agent setup
npx rea-agents@latest setup     # explicitly the latest release
npx -y rea-agents@latest analyze-javascript-application /absolute/path/to/app --json
```
- A package-runner invocation may pick up a REA version installed in the current project; use `@latest` to request the latest release explicitly.
- `--yes` placed **before** the package name approves npm's download only; it does not approve REA's setup changes.
- With Bun: `bunx rea-agents setup` or `bunx rea-agents@latest setup`.

### 2. Global install
```bash
npm install --global rea-agents
rea --help
```
> [!warning] Local install does not put `rea` on PATH
> `npm install rea-agents` without `--global` installs the executable only in the project's `node_modules/.bin`.

### 3. Curl wrapper (installs the global npm package only)
```bash
curl -fsSL https://raw.githubusercontent.com/morluto/rea/main/install.sh | bash
```
It prints the version, runtime, npm command and destination before installing. With a controlling terminal it starts `rea setup`; otherwise it prints the command to run later. Pass options with `bash -s --`:
```bash
curl -fsSL https://raw.githubusercontent.com/morluto/rea/main/install.sh |
  bash -s -- --dry-run
```
Supported options: `--version <semver>`, `--dry-run`, `--no-setup`, `--no-prompt`, `--verbose`. Neither `--no-prompt` nor a non-interactive shell grants permission to install external dependencies. Add the reported executable directory (commonly `~/.local/bin`) to your `PATH` if needed.

### 4. MCP Registry
REA is listed as `io.github.morluto/rea`. Registry-aware clients can discover it and launch the existing npm package via `npx ... mcp`. For manual configuration see [[Agent Setup#Manual MCP registration]].

### 5. Pinning or rolling back a version
```bash
npm exec --yes --package=rea-agents@2.4.0 -- rea setup
```
Setup always pins persistent MCP registrations to the exact version that performed setup.

## Released package vs repository main

- `@latest` selects the npm release; repository main documents current code and may include unreleased features.
- At the docs' last check (2026-10-11) the npm release was **6.4.0**. DeepSeek Harness (`dsh`) support is on main only, not in 6.4.0.
- Check the published version: `npm view rea-agents dist-tags.latest`.
- Use the connected server's actual tool list and input schemas for feature selection; a skill or doc from main may describe tools your installed release lacks.

## Installing analysis engines

REA does **not** install Ghidra, Java, IDA, JEB, JADX, Binwalk, Unblob or other engines. The only exception is Hopper:

- **macOS:** approved setup downloads the official DMG, checks its published size and digest, validates the bundle, and atomically installs to `~/Applications/Hopper Disassembler.app`, then opens Hopper so you can choose demo mode or activate a licence. No Homebrew, no admin access.
- **Linux:** approved setup verifies Hopper's official `.deb`, `.rpm` or Arch package and invokes the native package manager (`apt-get`, `dnf` or `pacman`, as root or via `pkexec`; never `sudo`). It also installs demo-session dependencies (Xvfb, Python 3, X11, XTEST) on supported distributions. Unattended installation requires `--yes --install-hopper`.
- **Windows:** Hopper installation is unavailable.

Ghidra setup is "configure, don't install":
```bash
export GHIDRA_INSTALL_DIR=/absolute/path/to/ghidra_12.1.4_PUBLIC
export JAVA_HOME=/absolute/path/to/jdk-21 # optional if java/javac are on PATH
rea doctor --json
rea setup
```
Setup can save verified Ghidra/JDK paths into agent registrations so GUI-launched clients do not depend on your shell's PATH. Full details: [[Ghidra]], [[Hopper]], [[IDA]].

## Next steps
- Register your agent: [[Agent Setup]]
- Verify readiness: [[Readiness Checks]]
- Keep it updated: [[Updating and Uninstalling]]

## Source
- https://github.com/morluto/rea/blob/main/docs/installation.md
- https://github.com/morluto/rea/blob/main/install.sh
- https://github.com/morluto/rea/blob/main/SECURITY.md
