---
tags: [rea, target, firmware, embedded]
source:
  - docs/firmware-analysis.md
  - docs/raw-binary.md
  - src/firmware/
  - third_party/README.md
---
# Firmware

REA inspects firmware images with caller-supplied **Binwalk 3.1.x** (region inspection) and **Unblob 26.6.x** (explicit extraction), then hands verified native children to [[Ghidra]]. No device, root privilege, mount, emulation or execution of extracted programs is involved. Arbitrary firmware bytes do **not** pass native executable admission — you select a returned ELF for native analysis.

## What REA returns

- **`inspect_firmware_regions`**: Binwalk signature regions with offsets into the original firmware, `reported_size` and `size_basis=provider_reported_validation_unknown`.
- **`extract_firmware`**: regular file paths with independently verified SHA-256, producer hashes when available, chunks, diagnostics and Unblob task derivations (logical references `$input`, `$output`, `$output/<relative-path>`).
- Both: original path and SHA-256, raw producer reports, executable path/digest/version, limitations, `complete` vs `partial` status.

## Requirements and host support

| Item | Value |
| --- | --- |
| Verified host | Linux x64 |
| Region inspection | Binwalk 3.1.x (parser verified with 3.1.0) via `REA_BINWALK_COMMAND` |
| Extraction | Unblob 26.6.x (verified with 26.6.4) via `REA_UNBLOB_COMMAND`, plus format extractors on `PATH` (e.g. `debugfs` for ext2/3/4) |
| Limiter | util-linux `prlimit` (override `REA_FIRMWARE_PRLIMIT_COMMAND`) |

Inspection and extraction have independent prerequisites. Unblob's Landlock sandbox stays enabled; REA never retries with `--no-sandbox`. Optional pinned upstream sources: `git submodule update --init third_party/binwalk third_party/unblob` (not built into npm packages). See [[External Tools]].

## CLI and MCP

```sh
export REA_BINWALK_COMMAND=/absolute/path/to/binwalk
export REA_UNBLOB_COMMAND=/absolute/path/to/unblob
rea inspect-firmware-regions /path/to/firmware.bin
rea extract-firmware /path/to/firmware.bin /path/to/new-output
rea extract-firmware /path/to/firmware.bin /path/to/selected-output \
  --offset 64 --length 4096 --max-depth 3
```

```json
{
  "name": "extract_firmware",
  "arguments": {
    "path": "/path/to/firmware.bin",
    "output_directory": "/path/to/new-output",
    "range": { "offset": 64, "length": 4096 },
    "max_depth": 3
  }
}
```

Staging budget options: `max_output_bytes` / `max_output_files` (CLI `--max-output-bytes` / `--max-output-files`); depth defaults to 3 (range 1–10).

## Worked example

> [!example] Router image → BusyBox function
> ```sh
> rea inspect-firmware-regions ./router.bin                   # find squashfs / kernel regions
> rea extract-firmware ./router.bin /tmp/router-out           # new, absent directory
> ```
> Then open a verified child:
> ```json
> { "name": "open_binary", "arguments": { "path": "/tmp/router-out/.../usr/bin/program", "provider_id": "ghidra" } }
> ```
> and continue with `analyze_function` or `inspect_native_api` ([[Native Binaries]]). MIPS and RISC-V ELF lanes have their own admission rules — see [[Ghidra Variants]].

> [!tip] Headerless flash dumps
> If the image is a bare code blob rather than a container, use Ghidra's raw-binary mode (`--target-format raw-binary --architecture arm --entry-point 0x8000000`) described in [[Native Binaries]].

## Limits and side effects

- Inputs: non-empty regular files ≤ 128 MiB, copied and hashed into a private workspace; no in-place modification.
- 120-second deadline; `prlimit` 1 GiB address space per process, 120 CPU s, no core dumps; JSON reports ≤ 8 MiB; ≤ 8 MiB combined stdout/stderr per command. One heavy operation per provider instance.
- Destination must be absent, absolute, with an existing parent; existing destinations are never overwritten. Publication is not an atomic rename; failed output is rolled back. Symlinks/special files are recorded, not published; permissions and ownership are not reproduced.
- Signatures and confidence don't prove valid partitions, encryption or runtime behavior. Decompressed children have unknown original byte ranges. `complete` means the engine finished its configured extraction — not complete firmware reconstruction.
- Not covered: PPC decompilation, raw flash load profiles, bootloader/device-tree semantics, vulnerability scanning, emulation, hardware runtime capture.

> [!info] Verification
> `npm run fixtures:firmware` then `npm run verify:firmware`; optional `REA_FIRMWARE_VERIFY_EXT4=1` and `REA_FIRMWARE_VERIFY_GHIDRA=1` lanes. See [[Testing]].

## Related
[[Native Binaries]] · [[Ghidra]] · [[Ghidra Variants]] · [[Signal Recordings]] · [[External Tools]]

## Source
- https://github.com/morluto/rea/blob/main/docs/firmware-analysis.md
- https://github.com/morluto/rea/blob/main/docs/raw-binary.md
