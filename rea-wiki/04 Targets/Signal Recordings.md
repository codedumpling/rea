---
tags: [rea, target, signals, hardware]
source:
  - docs/signal-recordings.md
  - src/signals/SigrokRecordingProvider.ts
---
# Signal Recordings

`decode_signal_recording` / `rea decode-signal-recording` answers **what a selected protocol decoder observes in an existing logic-analyzer recording** (for example UART, SPI, I²C captured as VCD). It uses caller-supplied **sigrok-cli 0.7.2** with libsigrokdecode. It never acquires samples or talks to hardware.

## What REA returns

- Input path and digest, exact selections (decoders, input format, annotations — omitted ones recorded as `null`).
- Producer annotation text with sample-number ranges, verbatim (e.g. `1104-1938 uart-1: 52`), paged by display line.
- `output_window` with `total_lines`, `returned_lines`, `has_more` (these count **display lines**, not annotations or samples).
- Tool configured/resolved paths, digests, full version banner (incl. library versions), stderr and `tool.version_diagnostics`.

## Requirements and host support

| Item | Value |
| --- | --- |
| Host | **Linux only** — macOS and Windows are refused (address-space limit can't be enforced) |
| Engine | sigrok-cli 0.7.2 + libsigrokdecode, selected with `REA_SIGROK_COMMAND` |
| Limiter | util-linux `prlimit` (default `/usr/bin/prlimit`; override `REA_SIGROK_PRLIMIT_COMMAND`) |

See [[External Tools]] and [[Environment Variables]].

## CLI and MCP

```sh
export REA_SIGROK_COMMAND=/absolute/path/to/sigrok-cli
rea decode-signal-recording /artifacts/uart.vcd \
  --decoders uart:rx=RX:baudrate=9600 --input_format vcd \
  --annotations uart=rx-data --format json
```

```json
{
  "name": "decode_signal_recording",
  "arguments": {
    "path": "/artifacts/uart.vcd",
    "decoders": "uart:rx=RX:baudrate=9600",
    "input_format": "vcd",
    "annotations": "uart=rx-data"
  }
}
```

- `decoders` uses sigrok's stack/option/channel syntax; channel names must match the recording. REA does not guess wiring, baud rate or protocol.
- Omit `input_format` for sigrok autodetection; omit `annotations` to keep every class.
- Paging: `line_offset`, `line_limit` (default 50), or MCP `line_limit: null` / CLI `--all_lines`.
- No arbitrary command flags are accepted. Each selector ≤ 8,192 UTF-16 code units; total selector arguments ≤ 32 KiB.

## Worked example

> [!example] Read a UART boot log from a board capture
> 1. Record the TX line with a logic analyzer and save as VCD (outside REA).
> 2. `rea decode-signal-recording ./boot.vcd --decoders uart:rx=RX:baudrate=115200 --annotations uart=rx-data --all_lines --format json`.
> 3. Read the annotation lines; correlate strings with firmware analyzed via [[Firmware]] and [[Native Binaries]].

## Limits and side effects

- Decodes a private read-only copy; the source is never passed to sigrok. Copy, executable and limiter are re-verified after decoding.
- Bounds: 32 MiB input, 30-second deadline, 4 MiB stdout+stderr; `prlimit` 1 GiB address space, 64 MiB per file, 30 CPU s, no core dumps. Oversized output fails rather than truncating.
- One decode at a time per provider (retryable capacity error otherwise).
- Display text is **not** parsed into protocol facts or lossless payload bytes; annotation ranges are sample numbers, not timestamps. Empty output doesn't prove absence of traffic. Pages from different source digests must not be combined.
- Caller-supplied tools and decoder modules are trusted; hashes identify bytes, not publisher authenticity. This is not an OS sandbox.

> [!info] Verification
> `npm run build:cached` then `REA_SIGROK_COMMAND=... node scripts/verify-signal-recording.mjs` (synthesizes a 9600-baud UART VCD carrying `REA`). See [[Testing]].

## Related
[[Network and Packet Captures]] · [[Firmware]] · [[External Tools]] · [[Targets Overview]]

## Source
- https://github.com/morluto/rea/blob/main/docs/signal-recordings.md
