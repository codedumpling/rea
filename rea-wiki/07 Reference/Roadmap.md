---
tags: [rea, reference, roadmap]
source:
  - docs/roadmap.md
  - CONTRIBUTING.md
  - AGENTS.md
  - CHANGELOG.md
---
# Roadmap

REA's roadmap (`docs/roadmap.md`) is deliberately short: it names priorities and the bar new capabilities must clear, and delegates detailed tracking to GitHub issues.

## Trackers
| Tracker | Scope |
| --- | --- |
| [Platform tracker #32](https://github.com/morluto/rea/issues/32) | Capability and verification work across platforms |
| [Process/Hopper tracker #48](https://github.com/morluto/rea/issues/48) | Process capture and Hopper work |
| [Browser tracker #39](https://github.com/morluto/rea/issues/39) | Browser observation work |

What is available *today* is documented in the website guides (https://rea.tools/guides/), the installation guide ([[Installation]]) and the generated tool catalog ([[MCP Tool Catalog]]).

## Priorities
1. **Native depth:** expand architecture, type and indirect-call verification.
2. **Cross-layer correlation:** connect static findings and runtime observations across application layers (see [[JavaScript Runtime Observation]]).
3. **Managed code:** improve obfuscated managed-code comparisons and verified native links ([[DotNET Assemblies]]).
4. **Process and protocol coverage:** extend process, protocol, filesystem, reconnect and build-comparison coverage ([[Process Capture]]).
5. **New providers/targets:** evaluated only when an analyst workflow demonstrates a capability gap.

## Admission criteria for new capabilities
- Must return useful evidence through the **shared CLI/MCP workflows**.
- Must preserve observations and unknowns ([[Evidence Model]]).
- Must declare actual authority and lifecycle constraints.
- Agents compose experiments from ordinary commands, scripts and fixture servers; custom orchestration needs a demonstrated requirement.
- Provider and platform claims require their corresponding **real verification lanes** ([[Testing]]). Simulated providers do not establish real-provider behaviour.
- Tool boundaries follow [[Tool Design Principles]]; the existing deep-provider contract is described by [[Ghidra]] semantics.

> [!info] Scope decisions first
> Per AGENTS.md/CONTRIBUTING.md, new providers or capabilities involving target execution, injection, mutation or protection recovery need a **maintainer scope decision** before substantial implementation. Early proposals, small prototypes and draft PRs are welcome to resolve questions, but do not authorize integration. See [[Contributing]].

## Setup policy stays fixed
Setup remains additive, reuses existing tools, and requires approval for disclosed configuration and Hopper installation changes. New installer choices need an implemented toolchain and explicit scope; setup will not install or upgrade unrelated runtimes or dependencies.

## Recent direction (from CHANGELOG 6.4.0)
For context on where work has been going: Android device/ADB and Apktool support, a JEB provider, summary/selected views for large analyses, isolated JavaScript analysis with heap/timeout limits, Ghidra language/compiler overrides and analysis seeds, RPC discovery (tRPC, GraphQL), Ghidra PE DLL and MIPS support, and Pi/Hermes/Qwen Code setup. See [[Updating and Uninstalling]] for breaking changes.

## Source
- https://github.com/morluto/rea/blob/main/docs/roadmap.md
- https://github.com/morluto/rea/blob/main/CHANGELOG.md
