---
tags: [rea, development, design]
source:
  - docs/tool-design.md
  - AGENTS.md
  - .agents/skills/rea-tool-design/SKILL.md
---
# Tool Design Principles

These are the rules for adding or changing an MCP tool, CLI command, provider capability or Evidence semantics. The `rea-tool-design` skill (see [[Repo Agent Skills]]) points agents to the same guide.

## 1. Start from the analyst question

- Before changing the surface, inspect the current canonical contracts, the provider capabilities and the **nearest existing tool**.
- **Extend** a tool when its intent and result contract match. **Add** a tool only for a distinct outcome or a materially different authority.
- Public names and semantics stay **provider-neutral**, and exact coverage is reported per provider.
- Keep the catalog complete. Report capability or session availability instead of truncating schemas. Fewer serialized bytes do not, by themselves, mean a tool is easier for an agent to use.

## 2. Choose the shape

| Shape | Use it for | Result |
| --- | --- | --- |
| `inspect` | Facts about one target, object or resource | Relevant fields, locations, facet availability |
| `search` / `list` | Candidate discovery | Stable ordering and context; pagination when needed |
| `trace` | Relationships across evidence | Typed edges, supporting facts, unresolved paths |
| `compare` | Explicitly paired artifacts or observations | Identity, comparable coverage, supported deltas |
| `workflow` | A recurring outcome needing composition | Inline answer, contributing evidence, partial facets |
| `observe` / `capture` | Runtime behavior | Authority, launch/attach effects, lifecycle, cleanup |

These are task shapes, not mandatory name prefixes. Don't mix unrelated discovery, execution and mutation modes in one tool.

## 3. Prefer primitives, then compose workflows

- A **primitive** reports a reusable fact about an identified object or relationship. One application's business interpretation does not belong in a general contract.
- Compose a **workflow** only when repeated use shows a multi-source outcome that REA can join without hiding the caller's choices or the uncertainty.
- Batch homogeneous reads, keeping per-item outcomes.
- Keep prompts optional (see [[Agent Prompts]]). Custom orchestration languages, replay engines or prepare/execute plans need a requirement that ordinary primitives demonstrably cannot meet.

## 4. Define the contract

- Inputs are **strict**: explicit target identity, required fields, and no ignored options or repeated permission flags.
- Return requested facts **inline**, with artifact and provider identity, source locations, Evidence references, coverage and limitations. See [[Evidence Model]].
- **Missing coverage is unknown, not absence.** Keep observed, derived and inferred relationships distinct, and preserve unresolved edges.
- `relationship_coverage: complete` means only that the provider-observed direct set was returned without a known read limit. It does not claim every possible runtime edge.
- Declare process, filesystem, network, UI and mutation **effects truthfully**.
- Add limits only for real format, protocol, authority or **measured** resource constraints. An assumed agent budget is not a reason to add one. Always report pagination, truncation, cancellation and partial failure.

## 5. Implement and evaluate

- Provider protocols and parsing go in adapters. Shared CLI/MCP workflows go in the application layer (see [[Codebase Map]]).
- Measure before choosing a budget, and enforce it **before** the costly step. Account for serialization expansion, retained records and temp-file growth separately.
- Derive completeness from the records actually examined, not from diagnostic prose or an empty failure list.
- Update together: canonical contracts, output schemas, examples, generated artifacts and docs. Validate the advertised JSON Schemas after SDK conversion.
- Verify success, malformed input, unavailability, cancellation and partial outcomes. Real-engine claims need the matching lane ([[Testing]]).
- When discoverability changes, evaluate broad, direct and negative tasks on the intended host, and record the host, model, catalog, selected tools, errors and retries.

> [!tip] Performance order (AGENTS.md)
> Optimize **agent tokens and context first** (the smallest sufficient answer, keeping provenance), then **analysis time** (reuse validated results, with cache keys covering every input that affects the result), then **CPU and memory**. Measure representative completed tasks before and after the change, and keep cold and warm runs separate.

See also [[MCP Tool Catalog]], [[MCP Server]], [[Contributing]].

## Source
- https://github.com/morluto/rea/blob/main/docs/tool-design.md
- https://github.com/morluto/rea/blob/main/AGENTS.md
