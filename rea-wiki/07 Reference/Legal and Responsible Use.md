---
tags: [rea, reference, legal, security]
source:
  - DISCLAIMER.md
  - LICENSE
  - THIRD_PARTY_NOTICES.md
  - SECURITY.md
  - README.md
  - AGENTS.md
---
# Legal and Responsible Use

This note summarizes REA's four governing documents: `DISCLAIMER.md` (intended use and responsibilities), `LICENSE` (MIT), `THIRD_PARTY_NOTICES.md` (dependency and engine licensing) and `SECURITY.md` (vulnerability reporting and security boundary). It is a paraphrase, not legal advice; the original documents control.

## Intended use

REA is built for **lawful analysis**:
- security research, malware and incident analysis
- vulnerability research and coordinated disclosure
- CTFs and education
- interoperability and software preservation
- debugging or auditing software you own or are authorized to examine

### What the project will not support
- circumventing DRM, licence checks, anti-cheat or other technical protection measures
- developing game cheats or tampering with online services
- bypassing authentication, bot protection or access controls on systems you aren't authorized to test
- copying proprietary software or reproducing its protected code and assets
- evading AI model providers' usage policies or safety measures
- reselling or sharing AI accounts or API access

Requests for these in issues and discussions are closed without response and may lead to removal from the community.

> [!warning] You are responsible
> You are responsible for commands, configurations, scripts and **actions you direct an AI agent to perform** with REA. Establish legal authority and scope for every target, account, device, dataset and operation. Owning a file, public availability, or access to a debugging endpoint does not by itself grant permission for every operation.

## Runtime, network and device operations

Some workflows execute targets, attach to runtime/browser endpoints, capture traffic or act on connected devices. These can change application state, files, accounts and services. Authorization to inspect an artifact **does not necessarily** authorize execution, interception or device changes. Review agent-generated plans and scripts, use an appropriate environment for untrusted targets, and protect credentials, personal data and captured traffic. Local operation does not guarantee that executed targets and third-party tools have no external effects.

## Analysis results

Decompilation, reconstruction, heuristics and AI-assisted analysis may be incomplete or wrong; verify consequential conclusions against the evidence ([[Evidence Model]]). Obtaining source-like output **does not grant rights** to the original program, its data, or the output. Check the target's rights and the producing tool's terms before publishing or reusing results.

## Licences

### REA itself: MIT
`LICENSE`: MIT License, copyright (c) 2026 morluto. REA is provided **AS IS** without warranty; the MIT terms govern warranty and liability. The disclaimer does not amend MIT or add licence conditions.

### Third-party material (`THIRD_PARTY_NOTICES.md`)
- Third-party code keeps its own licences and notices. Inventories live in `licenses/` (e.g. `licenses/npm-dependencies.txt`, `licenses/npm-runtime-notices.txt`) and `third_party/` (provenance for evmole, ghidra-nativeaot, jadx-headless-mcp, pwndbg, pwntools, unblob, binwalk, wabt, wakaru...).
- Notable derived/bundled items: the NativeAOT parser (derived from MIT-licensed ghidra-nativeaot and dotnet/runtime), `evmole@0.9.3` (MIT), and the Windows native addon built with MinGW-w64 (GCC runtime under GPLv3 with the GCC Runtime Library Exception).
- Runtimes you supply (Node.js, Python, OpenJDK, Electron, Chromium) are not licensed or bundled by REA.
- If you redistribute REA with an external engine or modified upstream code, you must handle those licences yourself.

### Commercial tools
**You must legally obtain and be authorized to use** IDA Pro / Hex-Rays decompilers, Hopper and JEB under their licences. REA grants **no** tool licence, key or activation entitlement. Trials and demos remain subject to vendor terms (user, feature, automation and output-sharing restrictions). Cracked copies, stolen keys, unauthorized key sharing and bypassing licence restrictions are not authorized. Specific points from the notices:
- **ida-pro-mcp** is MIT (Duncan Ogilvie) but licenses only the bridge; its README states IDA Free is unsupported.
- **Hopper**: setup may download the official installer after approval, which grants no paid licence. REA's Linux CI path uses Hopper's offered demo mode and never reads or automates licence credentials.
- **JEB**: the published EULA restricts public or third-party disclosure of decompiled code and reports unless your licence permits it.

The disclaimer also notes that credits to projects (including the NSA for Ghidra and commercial vendors) do not imply endorsement, and that the lawful-use position covers all REA versions, distribution channels and platforms.

## Security policy (`SECURITY.md`)

### Reporting
- Fixes are provided for the **latest published version**; upgrade before reporting.
- Report privately via **Report a vulnerability** in the repository's **Security** tab. Do not open a public issue, even to ask for a contact.
- Include REA version, OS/distribution, provider and version, impact, reproduction conditions and suggested mitigation.
- Strip credentials, proprietary binaries, Hopper documents, Ghidra project contents, socket capability tokens and unrelated local paths from logs.

### Security boundary
- Hopper and Ghidra bridges are authenticated with **per-session random capability tokens**, passed through private session descriptors (not argv or environment).
- POSIX uses current-user Unix sockets; Windows Ghidra uses authenticated loopback TCP with a private NTFS runtime whose DACL admits only the current user and SYSTEM.
- **This is not a sandbox.** It does not protect against malicious processes already running as your user, and opening an untrusted binary delegates parsing to the local provider with your permissions.
- Ghidra sessions use an isolated temporary project and private home/cache/runtime dirs, bounded startup and messages, and delete the project on close. REA never opens or modifies user-owned Ghidra projects.
- REA never downloads, installs or modifies Ghidra or Java. On Linux, Hopper metadata is accepted only from Hopper's HTTPS endpoint, package URLs are restricted to its download origin, size is bounded, and the published checksum (SHA-1, a corruption check rather than a signature) is compared. Package managers are invoked without shell evaluation, as root or via `pkexec`, never `sudo`.

### Data handling principles (AGENTS.md)
Preserve caller-selected inputs and evidence; redact transport credentials and **explicitly marked** sensitive values; don't infer secrecy from names; don't persist the ambient environment just because a child inherits it.

## Related
- [[FAQ]] · [[Troubleshooting]] · [[Contributing]] · [[Providers Overview]]

## Source
- https://github.com/morluto/rea/blob/main/DISCLAIMER.md
- https://github.com/morluto/rea/blob/main/LICENSE
- https://github.com/morluto/rea/blob/main/THIRD_PARTY_NOTICES.md
- https://github.com/morluto/rea/blob/main/SECURITY.md
