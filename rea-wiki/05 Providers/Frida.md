---
tags: [rea, provider, frida, runtime]
source:
  - docs/frida-instrumentation.md
  - docs/testing.md
  - src/frida/FridaInstrumentationManager.ts
  - src/cliCommandNames.ts
  - package.json
---
# Frida

REA exposes [Frida](https://frida.re/) device and process discovery, attach or spawn sessions, and **caller-authored JavaScript instrumentation** through the CLI and MCP. The Node bindings (`frida` 17.23.2) are an **optional** package dependency and load only when a Frida operation actually runs. REA never installs Frida server on a target.

Frida is an auxiliary runtime capability, not a deep `open_binary` provider. See [[Providers Overview]].

> [!warning] Execution authority
> Frida scripts run **inside the selected target**, with that process's memory, filesystem and network authority. This is runtime instrumentation, not read-only analysis. Choose targets you are authorized to instrument. See [[Legal and Responsible Use]].

## Tools

| Surface | Names |
| --- | --- |
| MCP | `list_frida_devices`, `list_frida_processes`, `start_frida_session`, `load_frida_script`, `resume_frida_session`, `unload_frida_script`, `frida_session_status`, `close_frida_session`, `instrument_with_frida` |
| CLI | `list-frida-devices`, `list-frida-processes`, `instrument-with-frida` |

`instrument-with-frida` is a one-shot command. It accepts one inline script or script file, observes messages for a bounded duration, and returns Evidence containing the source hash and the captured messages.

## Session lifecycle

- A **spawned** process starts paused, so scripts can load before it runs. Use `resume_frida_session` to let it continue.
- Closing a session unloads its scripts and detaches. If a spawn is still paused, REA resumes it only after confirming that the instrumentation has been released. If both cleanup paths fail, the target stays paused and the error is kept.
- **REA never kills the target.** MCP sessions end when the server process ends.
- If startup cleanup fails, REA keeps the acquired handles so it can retry cleanup later. CLI commands wait for provider cleanup before returning.
- If cancellation arrives during a native Frida call, REA waits for that call to return, closes any session it created, and returns a partial observation. That observation reports whether a resume was seen and whether cleanup completed; the final liveness of the target is unknown.

## Remote devices

Pass `host[:port]` (bracket IPv6 addresses), plus any token, certificate, origin or keepalive options, **on the operation itself**. The keepalive interval is in seconds. Authentication values exist only for the active call or session and are **excluded from Evidence**. Never put credentials in the endpoint address.

## Evidence

- Message capture is bounded to **16 MiB** and reports truncation.
- Script source is **not** stored. Its path and SHA-256 are.
- Device or endpoint, mode, program or PID and spawn arguments are retained. Authentication values are not.
- Platform metadata the device does not report stays unknown.

## Verification

`npm run verify:frida` needs a Linux REA host with the optional binding installed. It spawns a short-lived local Node.js fixture, loads a harmless marker script, captures the message and detaches. Add `-- --address HOST:PORT --pid PID` for the separate remote-attach lane, with optional `--token`, `--certificate`, `--origin` and `--keepalive-interval`. `npm run verify:frida:bindings` typechecks the bindings. Mock tests do not establish compatibility with a real device or server.

[Luma](https://luma.frida.re/) is Frida's standalone GUI. REA does not automate it.

See also [[JavaScript Runtime Observation]], [[Process Capture]], [[Android]].

## Source
- https://github.com/morluto/rea/blob/main/docs/frida-instrumentation.md
- https://github.com/morluto/rea/blob/main/docs/testing.md#frida-instrumentation
