---
tags: [rea, overview, showcase]
source:
  - README.md
  - website/evidence/dx-ball-sound-pan.md
  - website/evidence/notion-clipboard.md
  - website/evidence/th04-bullet-ring.md
  - website/evidence/aegis-login-code.md
  - website/evidence/ctf-masked-squares.md
  - website/evidence/calculator-and-dino.md
---
# Showcases

Real investigations published on the REA website (https://rea.tools/showcase/). Each has an evidence write-up under `website/evidence/` in the repo recording the exact target digest, REA version, provider and Evidence IDs used. They are good templates for how to phrase a request and what a defensible result looks like.

> [!tip] Pattern shared by every showcase
> Pin the target by SHA-256, record the REA version and provider, cite Evidence IDs for each finding, and validate any reconstruction against the original (tests, byte comparison, or recapture). See [[Evidence Model]] and [[Recipes]].

## DX-Ball: reconstruct a sound-pan calculation
- **Target:** English DX-Ball 1.07 `DXBALL.EXE`, PE i386 (158,208 bytes).
- **Tooling:** REA 4.1.0 with [[Ghidra]] 12.1.4 on Linux x64.
- **What happened:** followed a sound call into its position-to-pan helper (`0x406400`–`0x40643e`, 63 bytes), inspected the instructions and constants, and turned incomplete pseudocode into C.
- **Result:** the reconstruction passes 3,205 original-x86 cases and reproduces all 63 compiled function bytes.
- Links: https://rea.tools/showcase/dx-ball/ · reconstruction repo https://github.com/N0zoM1z0/dx-ball
- Relevant notes: [[Native Binaries]], [[Recipes]]

## Notion: trace the Electron clipboard bridge
- **Target:** Notion Desktop 7.6.1 shipped JavaScript (tab preload `preload.js`).
- **Tooling:** REA 4.1.0 `analyze_javascript_application` on the extracted `.webpack/renderer/tab_browser_view` directory, Node.js 24.
- **What happened:** found the renderer's clipboard API, followed it through preload and IPC into the main process, and inspected the rich clipboard format.
- Link: https://rea.tools/showcase/notion/
- Relevant notes: [[JavaScript and Electron]]

## TH04: recover a DOS bullet-ring calculation
- **Target:** Japanese PC-98 TH04 (Lotus Land Story) `MAIN.EXE`, DOS MZ, x86 real mode.
- **Tooling:** REA 4.1.0 with Ghidra 12.1.4 (`MzLoader`, `x86:LE:16:Real Mode`).
- **What happened:** inspected 16-bit instructions of `bullet_velocity_and_angle_set`, recovered the fixed and aimed angle calculations, and compared reconstructed C++ with historical compiler output.
- Links: https://rea.tools/showcase/th04/ · https://github.com/N0zoM1z0/th04
- Relevant notes: [[Ghidra Variants]] (DOS MZ/COM support)

## Aegis: Android login codes
- **Target:** Aegis Authenticator v3.4.3 APK (official release; SHA-256 checked against the release asset).
- **Tooling:** REA 6.1.0 CLI with caller-supplied `jadx-headless-mcp` 0.7.1 (`REA_JADX_MCP_JAR`) and OpenJDK 21.
- **What happened:** `inspect_android_method` and `trace_android_references` recovered the six-digit time-based code calculation (counter, HMAC, 31-bit truncation); the page lets you try a reconstruction with an adjustable clock.
- Relevant notes: [[Android]]

## DownUnderCTF: masked-squares flag checker
- **Target:** stripped Linux x86-64 ELF from DownUnderCTF 2023 (`ms_flag_checker`).
- **Tooling:** REA 4.1.0 with Ghidra 12.1.4.
- **What happened:** `search_strings` → `xrefs` → `resolve_containing_procedure` → `analyze_function` located the checking functions and masks before consulting the organizers' solution; the recovered flag was compared with the original program via process capture.
- Relevant notes: [[Native Binaries]], [[Process Capture]]

## Beginner examples: Windows Calculator and the dinosaur game
Used on the website homepage. Calculator: a function dossier of the `calc.exe` launcher shows the `ShellExecuteW` call with `ms-calculator:`, and the calculation itself (a `case 0x76` path that divides by a constant 100) is found in the app's model DLL `CalcViewModel.dll`. Dinosaur: selected excerpts from the open-source t-rex-runner.

## Share your own
The README invites users to share interesting cases through an issue or pull request, including the target, the question, how REA helped and what was found.

## Source
- https://github.com/morluto/rea/blob/main/README.md#showcases
- https://github.com/morluto/rea/tree/main/website/evidence
