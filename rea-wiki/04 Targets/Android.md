---
tags: [rea, target, android, mobile]
source:
  - docs/android-analysis.md
  - docs/apktool-resource-analysis.md
  - docs/adb-device-analysis.md
  - docs/jeb-analysis.md
  - src/android/
  - src/apktool/
  - src/adb/
---
# Android

Android support is a set of cooperating families. Each one wraps a **caller-supplied** engine; REA never installs Java, the Android SDK, JADX, Apktool, adb, JEB or an emulator.

| Family | Question it answers | Engine | Static / runtime |
| --- | --- | --- | --- |
| APK code (JADX) | Manifest, classes, decompiled methods, references | jadx-headless-mcp 0.7.1 + full JDK 17+ | Static |
| Resources (Apktool) | Decoded manifest, string tables, locales, version facts | Apktool 2.7.0+ | Static |
| Devices (ADB) | Devices, packages, processes, logs, screens, APK/file transfer | Any platform-tools `adb` | Runtime (some tools mutate the device) |
| JEB projects | Engine-grade units and pseudo-code | Caller-started JEB MCP server | Static (engine-side) |
| Inventory graph | Component paths, hashes, framework hints, bridge hypotheses | Built in | Static |

Flutter APKs have their own note: [[Flutter Apps]]. Dynamic instrumentation: [[Frida]].

## APK code (JADX)

**Requirements:** a **full JDK 17 or newer** (with `jdk.compiler`) and `REA_JADX_MCP_JAR` pointing at `jadx-headless-mcp-0.7.1-all.jar`; optional `JAVA_HOME`. Hosts (README): Linux/macOS/Windows x64. The metadata bridge is really verified on macOS arm64 + OpenJDK 21; the POSIX adapter also supports Linux; Windows x64 uses an owned Job Object (verified with JDK 21). Windows arm64 is outside the boundary. Tuning: `REA_JADX_HEAP_MIB`, `REA_JADX_ACTIVE_PROCESSOR_COUNT`, or `JAVA_TOOL_OPTIONS`.

```sh
export REA_JADX_MCP_JAR=/absolute/path/jadx-headless-mcp-0.7.1-all.jar
rea inspect-android-package /path/Example.apk
rea search-android-classes /path/Example.apk MainActivity
rea inspect-android-class /path/Example.apk example.MainActivity
rea inspect-android-method /path/Example.apk example.MainActivity select --overload-index 1
rea trace-android-references /path/Example.apk example.MainActivity --method-name onCreate
```

MCP: `inspect_android_package`, `search_android_classes`, `inspect_android_class`, `inspect_android_method`, `trace_android_references` (all take `path`; plus `class_name`, `method_name`, `overload_index`). `inspect_android_package` supports `detail: "summary"`; read the manifest later with `inspect_analysis_view` view `{kind: "android", facet: "manifest", offset: 0, limit: 4096}`.

**Returns:** decoded manifest XML and summary declarations, permissions, class/resource counts; class inventories; Java or smali method bodies (`<clinit>` returns smali with `static_initializer_disassembly`); incoming static references. Signature verification is explicitly `not_performed`.

## Resources (Apktool)

```sh
export REA_APKTOOL_COMMAND=/absolute/path/apktool   # optional; else PATH
rea inspect-apktool-client
rea decode-android-resources /targets/Example.apk --locale de
rea decode-android-resources /targets/Example.apk --no-strings
```

MCP: `inspect_apktool_client`, `decode_android_resources` (`path`, optional `locale`, `include_strings`). Returns `apktool.yml` metadata as printed, decoded manifest (≤ 512 KiB), verbatim string tables (default table ≤ 2000 entries), and every `values-<locale>`. Smali is skipped by design (`--no-src`). 600 s deadline.

## Devices (ADB)

```sh
export REA_ADB_PATH=/absolute/path/to/adb            # optional; else PATH
rea inspect-adb-client
rea list-adb-devices
rea resolve-adb-packages emulator-5554 whatsapp
rea pull-adb-package emulator-5554 com.whatsapp --output-directory /tmp/acq
rea inspect-android-package /tmp/acq/com.whatsapp/base.apk
rea read-adb-logcat emulator-5554 --count 200
rea capture-adb-screen emulator-5554 --output-directory /tmp/shots
```

Read tools include `inspect_adb_device`, `inspect_adb_display`, `list_adb_features`, `list_adb_services`, `list_adb_processes`, `inspect_adb_window`, `read_adb_setting`, `read_adb_logcat`, `list_adb_packages`, `resolve_adb_packages`, `inspect_adb_package`, `list_adb_directory`, `pull_adb_package`, `pull_adb_file`, `capture_adb_screen`, `collect_adb_bugreport`.

> [!warning] Device-mutating tools
> `push_adb_file`, `install_adb_package`, `uninstall_adb_package`, `start_adb_app`, `start_adb_activity`, `stop_adb_app` change device state. They declare `mutatesTarget` and never run implicitly. Pushes refuse existing files unless `overwrite` is set and verify with the device's `sha256sum`. No tool accepts arbitrary shell, `input` or `monkey`.

## JEB projects

Start a JEB client serving MCP (default `http://127.0.0.1:8425/mcp`, override `REA_JEB_MCP_URL`). Tools: `inspect_jeb_client`, `open_jeb_project`, `list_jeb_units`, `decompile_jeb_item` (CLI: hyphenated). Details in [[JEB]].

## Inventory graph

```sh
rea project-android-application-graph '{"inventory_evidence":[<inventory_artifact Evidence>]}'
```

MCP `project_android_application_graph`. Reports component paths/hashes, runtime-family hints from exact native library names (`libreactnativejni.so`, `libhermes.so`, `libflutter.so`, `libunity.so` …) and DEX × native-library bridge hypotheses (2 MiB candidate budget). It does not decode DEX.

## Worked example

> [!example] Acquire an installed app from an emulator and read its entry activity
> ```sh
> rea list-adb-devices
> rea resolve-adb-packages emulator-5554 example
> rea pull-adb-package emulator-5554 com.example.app --output-directory /tmp/acq
> rea inspect-android-package /tmp/acq/com.example.app/base.apk
> rea search-android-classes /tmp/acq/com.example.app/base.apk MainActivity
> rea inspect-android-method /tmp/acq/com.example.app/base.apk com.example.app.MainActivity onCreate
> rea decode-android-resources /tmp/acq/com.example.app/base.apk
> ```
> If `lib/<abi>/libapp.so` is present, continue with [[Flutter Apps]]; native `.so` files go to [[Native Binaries]].

## Limits and side effects

- JADX/Apktool operations never write the original APK or run target code; they use private snapshots. No split APK/AAB handling, signature validation or Android runtime capture in the JADX family. An `.aab` is treated as a ZIP, not an APK.
- Ambiguous method overloads are rejected rather than guessed. Decompiled source is a derived representation; references are static, not observed calls.
- JADX sessions are cleaned after 60 s idle; an uncertain cleanup blocks further launches for that provider instance.
- ADB: per-invocation budgets (shell reads 20–60 s, logcat 4 MiB, screen 32 MiB, transfers/bugreports 600 s). Pull/capture destinations must not exist. Device paths are not host paths. The adb client may start its own local server on first query.

> [!info] Verification
> `npm run verify:adb`, `npm run verify:apktool -- --apk PATH`, plus the JADX real lane. See [[Testing]].

## Related
[[Flutter Apps]] · [[JEB]] · [[Frida]] · [[Native Binaries]] · [[External Tools]] · [[Apple and Harmony Apps]]

## Source
- https://github.com/morluto/rea/blob/main/docs/android-analysis.md
- https://github.com/morluto/rea/blob/main/docs/apktool-resource-analysis.md
- https://github.com/morluto/rea/blob/main/docs/adb-device-analysis.md
- https://github.com/morluto/rea/blob/main/docs/jeb-analysis.md
