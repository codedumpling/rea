---
tags: [rea, usage, mcp, reference]
source:
  - src/contracts/toolContracts.ts
  - src/contracts/officialToolContracts.ts
  - src/contracts/enhancedToolContracts.ts
  - src/contracts/sessionToolContracts.ts
  - src/cliCommandNames.ts
  - docs/mcp-contracts.md
---
# MCP Tool Catalog

REA advertises **one canonical tool inventory**: `TOOL_CONTRACTS` in `src/contracts/toolContracts.ts`, built from about 30 per-family contract files. This note groups every public tool (roughly 187 in this source revision) by area, with one line on what each does, paraphrased from its contract description.

> [!info] The authoritative list is live
> The machine-readable catalog (`docs/public/product-catalog.json`) is **generated** by `npm run docs:generate` and isn't checked in. For a running server, `tools/list` gives the schemas and `binary_session` → `result.tool_availability` tells you what's callable right now. See [[MCP Server]].

Columns: **Tool** is the MCP name. **CLI** is the matching `rea` command (`-` means MCP-only; see [[CLI Reference]]).

## Session, Evidence and investigation
| Tool | CLI | Purpose |
| --- | --- | --- |
| `open_binary` | - | Validate and open a local executable, bundle, archive, JS, source map, plist or analysis DB. Takes `provider_id`, `snapshot_path`, `format` |
| `close_binary` | - | Drain requests, optionally save a snapshot (`snapshot_path`, `overwrite`), and release every provider resource |
| `binary_session` | - | Report target, provider, tool availability, client features and server identity **without starting analysis** |
| `export_evidence_bundle` | - | Atomically write the session's deterministic Evidence bundle to a path |
| `import_evidence_bundle` | - | Validate and merge a bundle from a path |
| `get_evidence_bundle` | - | Return retained Evidence. Use `detail: "summary"` plus `filters` to discover IDs cheaply |
| `inspect_analysis_view` | `inspect-analysis-view` | Summary, page or item view of retained or inline Evidence, without re-running the producer |
| `compare_bundles` | `compare` | Compare two canonical bundles by record membership and unknown history |
| `compare_artifacts` | - | Compare artifact inventories by occurrence path, content, metadata and relations |
| `compare_functions` | - | Compare explicit `analyze_function` Evidence sets |
| `find_changed_behavior` | - | Aggregate validated process and artifact comparison Evidence |
| `build_call_path` | - | Shortest direct-callee paths from function dossiers |
| `correlate_static_and_runtime` | - | Evaluate caller-declared static↔runtime hypotheses |
| `verify_reconstruction` | - | Check a finite typed specification against a bundle |
| `list_unknowns` / `record_unknown` / `update_unknown` / `verify_unknown_resolution` | - | Ledger of residual unknowns (see [[Evidence Model]]) |
| `get_navigation_context` | - | Selected document, cursor and containing procedure in one call |
| `inspect_address_context` | - | Name, procedure, comments and bookmarks at one address |
| `capture_process_scenario` | `capture-process` | Run one command under a PTY and capture process Evidence |
| `compare_process_captures` | `compare-process-captures` | Compare two process captures across terminal, lifecycle, FS, HTTP and WS |

## Native analysis: composed workflows
| Tool | CLI | Purpose |
| --- | --- | --- |
| `binary_overview` | `analyze`, `inspect` | Metadata, segments, procedure/string counts |
| `analyze_function` | `function` | Complete function dossier |
| `inspect_native_api` | `inspect-native-api` | Native API boundary of one function |
| `trace_feature` | `trace` | Literal query → strings/procedures → xrefs → containing procedures |
| `get_call_graph` | - | Exhaustive caller/callee traversal from one procedure |
| `trace_call_path` | - | Call path from a start to an optional goal procedure |
| `batch_decompile` | - | Decompile several procedures concurrently |
| `trace_native_values` | `trace-native-values` | Bounded static def-use/call dependency graph |
| `trace_native_ui_action` | `trace-native-ui-action` | Compiled UI action → handlers → static calls |
| `inspect_native_dispatch_metadata` | `inspect-native-dispatch-metadata` | ObjC class/method and Swift mangled symbols |
| `get_objc_classes` / `get_objc_protocols` | - | Deduplicated ObjC class and protocol labels |
| `analyze_swift_types` | - | Categorise Swift procedure names by mangling |
| `find_xrefs_to_name` | - | Resolve an exact name, then return its xrefs |

## Native analysis: provider primitives
Mostly one-to-one bridge operations (Hopper/Ghidra/IDA). See [[Providers Overview]].

| Tool | Purpose |
| --- | --- |
| `procedure_pseudo_code` (CLI `decompile`) | Decompile one procedure |
| `read_function_instructions` (CLI `instructions`) | Every raw instruction of a procedure |
| `procedure_assembly`, `procedure_info`, `procedure_address` | Assembly, metadata, canonical entry address |
| `procedure_callers`, `procedure_callees`, `procedure_references` | Direct call/reference edges |
| `resolve_containing_procedure` | Map any address to its containing procedure |
| `search_strings`, `search_procedures` (CLI `search`) | Literal or regex search |
| `xrefs` (CLI `xrefs`) | References to an address |
| `list_documents`, `list_segments`, `list_procedures`, `list_strings`, `list_names`, `list_bookmarks` | Inventories |
| `address_name`, `comment`, `inline_comment`, `next_address`, `prev_address` | Point reads |
| `current_document`, `current_address`, `current_procedure`, `goto_address` | Hopper GUI cursor state |
| `set_address_name`, `set_addresses_names`, `set_comment`, `set_inline_comment`, `set_bookmark`, `unset_bookmark` | Analyst metadata writes with readback |
| `annotate_native_function` | Atomic rename/comment and a refreshed dossier (Ghidra) |
| `read_bytes`, `address_to_file_offset` | Analysed memory and file-offset mapping |
| `inspect_native_instruction`, `resolve_native_call_targets` | One instruction and one call site |
| `inspect_native_data_type`, `inspect_native_load_image` | Recovered types and DOS MZ/COM load-image checks |

## Apple / Mach-O
| Tool | Purpose |
| --- | --- |
| `inspect_macho`, `list_architectures` | Slices, load commands, imports/exports, sections |
| `inspect_signature` | Code-signing identity, entitlements, hardened runtime |
| `inspect_plist` | Parse Info.plist or any plist |
| `demangle_swift` | Demangle Swift symbols without Hopper |
| `trace_dylib_resolution` | How dyld resolves `@rpath`/`@loader_path` loads |
| `decode_interface_builder`, `inspect_keyed_archive`, `inspect_asset_catalog` | Storyboards/nibs, NSKeyedArchiver, Assets.car |
| `observe_native_ui`, `capture_native_ui_scenario` | Accessibility observation and scripted UI steps |
| `observe_native_calls` | Launch under LLDB and record breakpoint hits |
| `project_apple_application_graph` | IPA/.app/DMG inventory → app graph |

See [[Apple and Harmony Apps]].

## Artifacts and offline formats
| Tool | Purpose |
| --- | --- |
| `inspect_artifact`, `extract_artifact` | Archive/package inventory and extraction into a fresh temp dir |
| `inspect_binary_layout` | Offline ELF layout and mitigations (pwntools) ([[ELF Layout and Recorded Crashes]]) |
| `inspect_recorded_crash` | Linux ELF64 core inspection |
| `inspect_pe_resources` | PE resource tree |
| `inspect_go_binary` | Embedded Go build metadata ([[Go Binaries]]) |
| `inspect_wasm_artifact` | WABT validation and WAT ([[WASM Artifacts]]) |
| `inspect_evm_interface` | EVM bytecode selectors ([[EVM Bytecode]]) |
| `inspect_sqlite_database` | SQLite schema and row samples ([[SQLite Databases]]) |
| `inspect_packet_capture` | TShark field extraction ([[Network and Packet Captures]]) |
| `decode_signal_recording` | sigrok decoding ([[Signal Recordings]]) |
| `inspect_firmware_regions`, `extract_firmware` | Binwalk / Unblob ([[Firmware]]) |

## .NET managed code
`inspect_managed_artifact`, `inspect_managed_members`, `inspect_managed_native_boundaries`, `compare_managed_members`, `verify_managed_native_boundaries`, `import_managed_reconstruction`, `project_managed_application_graph`. All are execution-free PE/CLI metadata and CIL workflows. See [[DotNET Assemblies]].

## Android, Flutter, JEB and adb
| Group | Tools |
| --- | --- |
| Static APK (JADX) | `inspect_android_package`, `search_android_classes`, `inspect_android_class`, `inspect_android_method`, `trace_android_references`, `project_android_application_graph` |
| apktool | `inspect_apktool_client`, `decode_android_resources` |
| Flutter | `identify_flutter_build`, `inspect_dart_aot` |
| JEB | `inspect_jeb_client`, `open_jeb_project`, `list_jeb_units`, `decompile_jeb_item` |
| adb read | `inspect_adb_client`, `list_adb_devices`, `inspect_adb_device`, `list_adb_packages`, `resolve_adb_packages`, `inspect_adb_package`, `read_adb_logcat`, `list_adb_processes`, `list_adb_directory`, `list_adb_features`, `list_adb_services`, `inspect_adb_display`, `inspect_adb_window`, `read_adb_setting` |
| adb acquire | `pull_adb_package`, `pull_adb_file`, `capture_adb_screen`, `collect_adb_bugreport` |
| adb mutate (`mutatesDevice`) | `push_adb_file`, `install_adb_package`, `uninstall_adb_package`, `start_adb_app`, `start_adb_activity`, `stop_adb_app` |

See [[Android]], [[Flutter Apps]], [[JEB]].

## JavaScript, Electron and application graphs
| Tool | Purpose |
| --- | --- |
| `analyze_javascript_application` | Static application graph from an ASAR or extracted tree |
| `recover_javascript_sources` | Wakaru-based readable module recovery |
| `trace_application_feature` | Trace a literal seed through the graph |
| `trace_javascript_semantics` | Static data-flow, call/return and closure relations |
| `compare_application_versions`, `compare_source_to_bundle`, `compare_javascript_export_shapes` | Version, source↔bundle and export-shape comparisons |
| `reconcile_javascript_runtime` | Join static graphs with passive CDP/Inspector Evidence |
| `list_electron_targets`, `inspect_electron_page`, `capture_electron_scenario` | Electron observation |
| `list_javascript_runtime_targets`, `observe_javascript_runtime` | Node/Electron V8 Inspector |
| `project_harmony_application_graph` | HarmonyOS HAP/HSP inventory → app graph |
| `build_reconstruction_obligation_ledger`, `evaluate_reconstruction_coverage` | Reconstruction obligations and coverage ([[Evidence Export and Import]]) |

See [[JavaScript and Electron]] and [[JavaScript Runtime Observation]].

## Browser and web
| Tool | Purpose |
| --- | --- |
| `list_browser_targets`, `inspect_web_page` | CDP discovery and passive page inspection |
| `analyze_web_bundle` | Chunk graph, routes and endpoints from page JS |
| `observe_web_session` | Timed observation window while the user drives the page |
| `discover_webmcp_tools` | Inventory page-registered WebMCP tools (never invoked) |
| `capture_web_screenshot`, `compare_web_screenshots` | PNG capture and pixel diff |
| `compare_web_captures` | Passive or scenario capture diff |
| `capture_browser_scenario` | Controlled scripted browser run |
| `export_web_scripts`, `trace_web_source_location`, `trace_web_module_imports` | Script export, source-map positions, ES import tracing |
| `observe_web_execution`, `inspect_web_event_listeners` | Precise coverage window and DOM listeners |
| `inspect_web_network_capture` | Offline HAR/mitmproxy/SAZ/qlog inspection |

See [[Websites and Browser Observation]].

## Frida
`list_frida_devices`, `list_frida_processes`, `start_frida_session`, `load_frida_script`, `resume_frida_session`, `unload_frida_script`, `frida_session_status`, `close_frida_session`, and the one-shot `instrument_with_frida`. See [[Frida]].

> [!tip] Effects
> Each contract carries effect metadata such as `readOnlyHint`, `destructiveHint`, `executesForeignCode` and `mutatesDevice`. An analysis that only adds Evidence is non-read-only (it changes session state) but **not** destructive. See [[Tool Design Principles]].

## Source
- https://github.com/morluto/rea/blob/main/src/contracts/toolContracts.ts
- https://github.com/morluto/rea/blob/main/src/cliCommandNames.ts
- https://github.com/morluto/rea/blob/main/docs/mcp-contracts.md#generated-catalog
