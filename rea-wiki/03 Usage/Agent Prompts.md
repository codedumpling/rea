---
tags: [rea, usage, mcp, prompts]
source:
  - docs/mcp-prompts.md
  - src/contracts/promptContracts.ts
  - src/server/registerPrompts.ts
  - src/server/promptCompletion.ts
  - README.md
---
# Agent Prompts

There are two layers here:
1. **Natural-language requests** you type to your agent. With the [[Agent Skill]] installed, the agent decides which REA tools to call.
2. **Guided MCP prompts** that REA itself advertises through the MCP `prompts` capability. They are provider-neutral investigation workflows that an MCP client can list and render.

## Talking to your agent

The README's canonical example:

```text
Understand how search works in the Notes app, show me the evidence, and build a
similar feature for my project.
```

Replace the app and feature with your own. Requests that work well:

```text
Trace how the clipboard bridge works in /Applications/Notion.app — renderer API,
preload, IPC, main process. Cite Evidence IDs and list unknowns.
```
```text
Open /absolute/path/to/game.exe with Ghidra, find the function that computes the
sound pan from the ball position, decompile it and reconstruct it in C.
```
```text
Compare /apps/v1/app.asar and /apps/v2/app.asar and tell me what changed in the
settings sync feature. Don't call something removed unless coverage is complete.
```
```text
I have Chrome open with --remote-debugging-port=9222 on my staging site. List the
targets, inspect the checkout page and summarise the API endpoints it calls.
```

> [!tip] Be specific about the target
> The skill tells agents to **ask which app** when you only say "my app". They won't pick an example app on your behalf. Give an absolute path, or a name that matches one installed artifact.

> [!warning] Ask for evidence, not just answers
> REA separates **observations, inferences and unknowns** (see [[Evidence Model]]). Asking for Evidence IDs and residual unknowns keeps the agent from presenting static pseudocode as observed runtime behaviour.

## Guided MCP prompts

`docs/mcp-prompts.md` lists six prompts (defined in `src/contracts/promptContracts.ts`):

| Prompt | Purpose | Arguments (required in **bold**) |
| --- | --- | --- |
| `investigate_feature` | Trace a feature from discovery into function evidence | **`feature`**, `target_path`, `document`, `procedure`, `provider_id` |
| `compare_application_versions` | Compare two shipped artifacts, with optional static/runtime follow-up | **`left_target_path`**, **`right_target_path`**, `left_manifest_id`, `right_manifest_id`, `focus_occurrence_id` |
| `verify_reconstruction` | Evaluate a finite reconstruction spec against retained comparison Evidence | **`reconstruction_goal`**, `comparison_evidence_id`, `provider_id` |
| `trace_crash` | Correlate a crash symptom with static paths and optional process capture | **`crash_signal`**, `target_path`, `document`, `procedure`, `capture_evidence_id` |
| `audit_residual_unknowns` | Audit current unknown heads and evidence-qualified resolution | **`audit_scope`**, `unknown_id`, `evidence_id` |
| `prepare_bounded_process_capture` | Design a bounded process experiment *before* running anything | **`behavior_question`**, **`executable`**, **`working_directory`**, `prior_capture_evidence_id` |

Each rendered prompt lists **optional** starting points that use current tool names. For example, `investigate_feature` suggests:
1. `analyze_javascript_application` directly for a JS/Electron directory (no `open_binary` needed), or `open_binary` for a file or `.app`.
2. `current_document` to confirm which document is bound.
3. `search_strings` / `search_procedures` / `procedure_address`.
4. `inspect_analysis_view` on retained Evidence, then `trace_application_feature` once a seed is known.
5. `analyze_function`, `xrefs`, `procedure_callers`, `procedure_callees`, `trace_feature`.
6. `record_unknown` for open questions.

Prompts don't call tools themselves, and they don't add or remove any. Argument values and completion choices are rendered as **untrusted selection data**, never as instructions.

### Using them over raw MCP

```json
{ "method": "prompts/list", "params": {} }
```
```json
{
  "method": "prompts/get",
  "params": {
    "name": "investigate_feature",
    "arguments": { "feature": "offline search", "document": "Notes" }
  }
}
```

The server advertises `listChanged: true`, so it sends `notifications/prompts/list_changed` when a registered prompt is updated.

## Session-aware completion

`completion/complete` completes **prompt arguments only**, not arbitrary tool arguments. Suggestions are live projections of the current session:

| Argument family | Source |
| --- | --- |
| Document | Provider `list_documents` |
| Procedure | Full `list_procedures` inventory. A name is offered only if it maps to one address |
| Provider | `auto` plus sorted deep candidates |
| Evidence | IDs in the session ledger |
| Process capture | Retained process-capture Evidence IDs |
| Artifact manifest / occurrence | IDs from schema-valid retained inventories |
| Residual unknown | Current non-resolved heads |

Matching is a Unicode-normalised, case-insensitive prefix match. Results are deduplicated and sorted by code point, with no paging and **no cache**.

> [!info] Empty completions don't mean "nothing there"
> Procedure completion only reads a provider that's *already ready*. It never starts analysis. Run a query first if you need the inventory. Closing the binary clears the ledger and unknowns, so their IDs stop being suggested.

```json
{
  "method": "completion/complete",
  "params": {
    "ref": { "type": "ref/prompt", "name": "investigate_feature" },
    "argument": { "name": "procedure", "value": "0x10" },
    "context": { "arguments": { "document": "Notes" } }
  }
}
```

## Related
[[MCP Server]] · [[MCP Tool Catalog]] · [[Recipes]] · [[Agent Skill]] · [[Showcases]]

## Source
- https://github.com/morluto/rea/blob/main/docs/mcp-prompts.md
- https://github.com/morluto/rea/blob/main/src/contracts/promptContracts.ts
- https://github.com/morluto/rea/blob/main/README.md
