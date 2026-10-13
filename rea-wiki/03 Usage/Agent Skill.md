---
tags: [rea, usage, skill, agents]
source:
  - .agents/skills/reverse-engineer-anything/SKILL.md
  - .agents/skills/reverse-engineer-anything/references/connection-and-recovery.md
  - .agents/skills/reverse-engineer-anything/references/evidence-workflows.md
  - .agents/skills/reverse-engineer-anything/references/native-and-artifacts.md
  - .agents/skills/reverse-engineer-anything/references/javascript-applications.md
  - .agents/skills/reverse-engineer-anything/references/android-applications.md
  - .agents/skills/reverse-engineer-anything/references/runtime-observation.md
---
# Agent Skill

REA ships a workflow skill named **`reverse-engineer-anything`** (`.agents/skills/reverse-engineer-anything/`, metadata `version: "41"`). `rea setup` installs it next to the MCP registration, by default under `~/.agents/skills` (`--skill`/`--skill=false` and `--skill-client` control this; see [[Agent Setup]]). It tells the agent **when** to use REA and **how** to work with it. It doesn't add tools itself.

> [!info] Skill ≠ server
> Installing the skill doesn't register the MCP server, install engines, or add tools to a running agent. A skill from repository `main` can describe capabilities that an older npm server lacks. The agent is told to trust the **connected server's actual tool list and schemas**.

The repo's *development* skills (`rea-tool-design`, `rea-changelog-update`) are covered in [[Repo Agent Skills]].

## When to use REA (trigger)

Use REA when a claim depends on a **shipped binary or package, decompilation, passive runtime evidence, or comparison** that available source can't establish. **Skip REA** for ordinary analysis of a complete source repository, and don't run readiness or provider commands in that case.

## Connect only when needed
- If REA tools are present and the registration isn't known to be stale, go straight to the target. **Don't run doctor or setup before every investigation.**
- If tools are missing, blocked, or the provider is unavailable, read `connection-and-recovery.md`, fix the specific blocker, and resume the original task.
- Prefer an installed `rea`. Fall back to `npx -y rea-agents@latest` only when acquiring the package is authorized.
- Setup starts with a read-only plan (`rea setup --client codex --dry-run --json`), shows the paths, backups and skill replacement, then applies with `--yes`. Setup **never installs unrelated software** (Node, Java, Ghidra, IDA, JADX, Binwalk, …).

## Route the target first

| Target | First tool |
| --- | --- |
| ASAR / extracted JS or Electron tree | `analyze_javascript_application` |
| ZIP/APK/IPA/MSIX/AppX/DMG member inventory | `open_binary`, then `inspect_artifact` |
| APK code/classes/methods | `inspect_android_package`, then focused Android tools |
| Managed PE/CLI | `inspect_managed_artifact` |
| Firmware image | `inspect_firmware_regions` / `extract_firmware` |
| .NET NativeAOT | Native Ghidra analysis |
| Open browser page / Electron runtime | `list_browser_targets` / `list_electron_targets` |
| HAR/SAZ/mitmproxy/qlog | `inspect_web_network_capture` |
| Packet capture / logic recording | `inspect_packet_capture` / `decode_signal_recording` |
| WASM / EVM / Go / SQLite | `inspect_wasm_artifact` / `inspect_evm_interface` / `inspect_go_binary` / `inspect_sqlite_database` |
| ELF layout / ELF core / PE resources | `inspect_binary_layout` / `inspect_recorded_crash` / `inspect_pe_resources` |
| Native executable or analysis DB | `open_binary`, then focused tools (`binary_overview` if useful) |
| Existing IDA MCP setup | `open_binary` with the **original binary** and `provider_id: "ida"` (never pass `.idb`/`.i64`). IDA has no `binary_overview` |

> [!warning] Never guess the app
> If the user says "my app" without naming one, **ask**. Paths in unrelated reports or test fixtures don't count as a selection, and the agent must never pick an example app on the user's behalf.

## Work summary-first
- Start with the default result and its inline Evidence. For large or unknown-size JS apps and ELF layouts, use `"detail": "summary"`, then `inspect_analysis_view` on `parent_evidence_id`.
- Page module identities and select by `node_id`, since one path can have several graph roles.
- **Don't repeat** an identical analysis or a rejected call unless an input changed or a blocker was resolved.
- On `resource_constraint` / `transport`, use the reported reference with `inspect_analysis_view`, or `export_evidence_bundle`.
- If the client truncates a successful result, treat the preview as incomplete. Don't parse it as canonical JSON or re-run the producer.
- Lost the parent ID? Use `get_evidence_bundle` with `detail: "summary"` and exact filters.

## Epistemic rules
- Every conclusion distinguishes **observations, inferences and unknowns**, cites Evidence IDs, and keeps limitations and incomplete coverage. Static analysis is never described as observed execution. See [[Evidence Model]].
- Runtime requests execute only the declared target and lifecycle. Never broaden them.
- REA needs no permission grants, but the **client's** approval policy still applies.

## Plan broader investigations (7 steps)
1. Turn the request into a checklist of questions, each with the evidence it needs.
2. Reuse existing session state, snapshots, bookmarks and prior Evidence. Don't open duplicate sessions.
3. Start with the smallest overview and follow each question from entry point to result.
4. Inspect packaged resources and config with format-aware parsers, not filenames.
5. Corroborate with the right evidence type. Use runtime observation only when static evidence can't answer.
6. Split independent questions across parallel workers when they're available.
7. Keep a finding ledger: conclusion, Evidence IDs, confidence, search boundary, unknowns.

Before finishing, mark each checklist item answered, partial or unresolved. A broad investigation is not "complete" while required questions are open. Close native sessions with `close_binary`.

## Reference guides (read only the relevant one)

| File | Covers |
| --- | --- |
| `connection-and-recovery.md` | Observation-to-remedy table (no tools, stale registration, Codex `approval policy is never`, `provider_unavailable`, PATH issues). Lists the supported client IDs (`claude_code`, `codex`, `cursor`, `gemini_cli`, …). Explains why Codex tool approval is separate from REA readiness, with an optional per-tool `approval_mode = "approve"` TOML example. Includes a CLI fallback recipe |
| `native-and-artifacts.md` | Go, SQLite, recorded crashes, ELF layout, WASM views, native facets via `inspect_analysis_view`, `observe_native_calls`, DOS MZ/COM, raw-binary, ObjC metadata, managed PE/CLI and NativeAOT, packages and extraction, `trace_dylib_resolution`, `annotate_native_function`, PE resources |
| `javascript-applications.md` | `recover_javascript_sources`, `source_paths` selection, `max_heap_mb`/`analysis_timeout_ms`, module page/item JSON, scoped traces, oversized-display recovery, version and export-shape comparison, retained references |
| `android-applications.md` | JADX tool order, `REA_JADX_MCP_JAR`, Java 17+, `REA_JADX_HEAP_MIB`, static-only caveats |
| `runtime-observation.md` | Passive CDP rules (no clicks, navigation or evaluation), redactions, `reconcile_javascript_runtime`, the Fiddler Everywhere workflow |
| `evidence-workflows.md` | Inventory views, residual unknowns, comparison tools and their limits, process-capture finalization, the `api-surface` view |

## Related
[[Agent Prompts]] · [[MCP Tool Catalog]] · [[Recipes]] · [[Troubleshooting]] · [[Agent Setup]]

## Source
- https://github.com/morluto/rea/blob/main/.agents/skills/reverse-engineer-anything/SKILL.md
- https://github.com/morluto/rea/tree/main/.agents/skills/reverse-engineer-anything/references
