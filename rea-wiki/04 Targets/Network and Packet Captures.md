---
tags: [rea, target, network, capture]
source:
  - docs/web-network-captures.md
  - docs/packet-captures.md
  - src/captures/TsharkCaptureProvider.ts
---
# Network and Packet Captures

REA inspects **already-recorded** traffic. It never opens an interface, starts a proxy listener, launches a browser or fetches recorded URLs. Two independent families:

| Family | Tool (MCP / CLI) | Formats | Engine |
| --- | --- | --- | --- |
| Saved web traffic | `inspect_web_network_capture` / `rea inspect-web-network-capture` | HAR 1.2, Fiddler Everywhere SAZ, native mitmproxy, aioquic JSON qlog 0.3 | Built in; mitmdump for native mitmproxy |
| Recorded packets | `inspect_packet_capture` / `rea inspect-packet-capture` | Anything the installed TShark dissects (network, USB, Bluetooth) | Caller-supplied TShark 4.x |

For live browser traffic use [[Websites and Browser Observation]].

## Saved web traffic

### What REA returns

- `artifact` (selected file + stable snapshot digest) and `records` in producer order with original zero-based ordinals and JSON pointers / byte ranges / SAZ member names.
- `reported`: the producer's fields (HAR container metadata separated, `log.entries` replaced by `null`).
- `numeric_literals` (exact lexemes, unsafe integers), `binary_fields` (bytes, base64, SHA-256 — independent of declared sizes), `redactions`.
- An offline **derived API surface** view: request lines grouped by method, origin and path template (`{numeric}`, `{uuid}`, `{hex}`, `{iso-date}`) — HAR only.

### Requirements

| Format | Requirement |
| --- | --- |
| `har` | None (HAR 1.2 profile only) |
| `saz` | None; needs an enforced decoder memory bound (Node heap or Windows Job Object) |
| `qlog` | None; aioquic 1.5.0 / JSON qlog 0.3 profile |
| `mitmproxy` | Caller-supplied **mitmdump 12.2.3 on Linux** via `REA_MITMDUMP_COMMAND` |

### CLI and MCP

```sh
rea inspect-web-network-capture /absolute/session.har har --json
rea inspect-web-network-capture /absolute/session.har har --record 2 --record 0 --json
rea inspect-web-network-capture /absolute/session.saz saz --record 0 --json
rea inspect-web-network-capture /absolute/origin.qlog qlog --json
REA_MITMDUMP_COMMAND=/absolute/mitmdump rea inspect-web-network-capture /absolute/session.mitm mitmproxy --json
```

```json
{
  "name": "inspect_web_network_capture",
  "arguments": { "capture_path": "/absolute/session.har", "format": "har", "record_ordinals": [2, 0] }
}
```

Mark secrets with MCP `sensitive_values` or repeated CLI `--sensitive-value`: matching fields are replaced by `null` (declarations are never persisted, only their count).

> [!tip] Big captures over MCP
> Request `detail: "summary"`, then page with `inspect_analysis_view` using `{"kind":"retained-evidence","evidence_id":"<parent_evidence_id>"}` and `view: {"kind":"capture","detail":"preview","offset":0,"limit":20}` (or `"detail":"complete"`), or `view: {"kind":"api-surface","offset":0,"limit":50}` for the endpoint inventory.

> [!info] Fiddler Everywhere MCP is separate
> Live Fiddler traffic is available through Telerik's own MCP server (default `http://localhost:8868/mcp`). REA does not register or wrap it; use REA's SAZ workflow for saved files.

## Recorded packets (TShark)

### Requirements

**Linux only**: TShark 4.x (`REA_TSHARK_COMMAND`) and util-linux `prlimit` at `/usr/bin/prlimit` (override with `REA_TSHARK_PRLIMIT_COMMAND`). macOS and Windows are refused because the integration requires an enforced address-space limit. See [[External Tools]].

### CLI and MCP

```sh
export REA_TSHARK_COMMAND=/absolute/path/to/tshark
rea inspect-packet-capture /artifacts/device.pcapng \
  --fields ip.addr --fields udp.port --fields data.data \
  --display_filter 'udp' --format json
```

```json
{
  "name": "inspect_packet_capture",
  "arguments": { "path": "/artifacts/device.pcapng", "fields": ["ip.addr", "udp.port", "data.data"], "display_filter": "udp" }
}
```

Options: `display_filter`, `decode_as` (e.g. `["udp.port==5555,dns"]`, repeat `--decode_as`), `packet_limit` (input packets read **before** filtering), `two_pass` (default true; CLI `--no-two_pass`), paging with `match_limit` (default 50) / `match_offset`, or `match_limit: null` / CLI `--all_matches`.

**Returns:** `frame.number` plus selected fields as ordered string arrays (missing fields stay absent), `output_window`, source path/digest/size, tool path/digest/version banner, arguments and stderr.

## Worked example

> [!example] From a HAR to an API inventory to a source location
> 1. `rea inspect-web-network-capture /abs/session.har har --json > har.json`.
> 2. `rea inspect-analysis-view` with that Evidence inline and `view: {"kind":"api-surface"}` → grouped endpoints.
> 3. Inspect the initiating script in a live capture ([[Websites and Browser Observation]]) and map it with `trace_web_source_location`.

> [!example] Decode a custom UDP protocol as DNS
> `rea inspect-packet-capture ./dev.pcap --fields dns.qry.name --decode_as 'udp.port==5555,dns' --format json`

## Limits and side effects

- Offline; original files are unchanged. Every record is validated before selection, so one malformed record fails the whole call.
- Web captures: 32 MiB input, 96 MiB decoder reply, 64 nesting levels; HAR/SAZ/qlog decode in a 192 MiB Node heap; mitmproxy under 768 MiB / 30 s CPU / 96 MiB output. SAZ expanded contents ≤ 32 MiB; encrypted SAZ unsupported. Bodies are not dechunked/decompressed.
- API-surface groups make no handler-identity, schema or causality claim.
- TShark: 32 MiB input, 30 s per command, 4 MiB stdout+stderr; `prlimit` 1 GiB address space, 64 MiB per file, 30 CPU s, no core dumps. One inspection at a time per provider. Runs on a private read-only copy with an empty `WIRESHARK_CONFIG_DIR` and name resolution disabled; installed TShark plugins/config remain trusted.
- An empty result doesn't prove absence of traffic outside the selected recording, filter or read range. Dissector output is an observation, not verified device behavior.

> [!info] Verification
> `REA_TSHARK_COMMAND=... node scripts/verify-packet-capture.mjs` (after `npm run build:cached`). See [[Testing]].

## Related
[[Websites and Browser Observation]] · [[Signal Recordings]] · [[External Tools]] · [[Evidence Model]]

## Source
- https://github.com/morluto/rea/blob/main/docs/web-network-captures.md
- https://github.com/morluto/rea/blob/main/docs/packet-captures.md
