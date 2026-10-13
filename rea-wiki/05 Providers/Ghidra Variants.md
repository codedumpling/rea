---
tags: [rea, provider, ghidra]
source:
  - docs/windows-ghidra-p0.md
  - docs/ghidra-dos.md
  - docs/ghidra-mips.md
  - docs/ghidra-psp.md
  - docs/ghidra-riscv.md
  - docs/ghidra-nativeaot.md
  - docs/raw-binary.md
  - bridge/ghidra/
  - bridge/nativeaot/
---
# Ghidra Variants

The [[Ghidra]] provider has several specialized **lanes**. Each lane has a narrow admission boundary and a committed analysis profile. All of them use the ordinary CLI/MCP tools (`open_binary` with `provider_id: "ghidra"`, or `--provider ghidra`); there are no variant-specific tool catalogs. A target outside every lane gets a specific unsupported constraint back. REA does not guess a "close enough" profile.

| Variant | What it admits | Host | Verify |
| --- | --- | --- | --- |
| Windows P0 | Native x86/x86-64 PE apps and DLLs | Windows 10+ x64, local NTFS | `verify:ghidra:windows`, `verify:ghidra:windows:package`, `verify:windows-native` |
| DOS MZ / COM | 16-bit real-mode MZ; COM only with explicit `dos-com` | Linux x64, macOS arm64 verified | `verify:ghidra:dos`, `verify:ghidra:com` |
| Raw binary | Headerless x86 / x86_64 / arm / arm64 with caller-supplied entry | Linux/macOS | `verify:ghidra:raw` |
| MIPS ELF | ELF32 o32 MIPS32r2 exec/rel; MIPS16 shared profile | Linux/macOS | `verify:ghidra:mips`, `:mips-shared`, `:mips-relocatable` |
| PSP (Allegrex) | PSPSDK static ELF32 with the `ghidra-allegrex` extension | Linux x64 | `node scripts/verify-real-ghidra-mips.mjs --psp` |
| RISC-V ELF | LE ELF32/ELF64 `ET_EXEC`, double-float ABI | Linux/macOS | `verify:ghidra:riscv` |
| NativeAOT | x64 PE with .NET 8 RTR 9.1 metadata | any Ghidra host (parser) | `verify:ghidra:nativeaot` |

## Windows Ghidra P0 (experimental)

- **Boundary:** Windows 10+ x64; Node 22.x (≥22.19), 24.x (≥24.11) or 26+; operator-installed Ghidra 12.1.x and a 64-bit JDK 21+. It accepts explicit **native, non-managed x86/x86-64 PE applications or DLLs** on fixed local NTFS volumes, and supports the **25 read-only operations only**. Annotations, raw-binary, DOS and other formats are unavailable. Managed PE goes to the separate .NET provider ([[DotNET Assemblies]]).
- **Native authority comes bundled** in the npm package: Job Object ownership, protected private-runtime DACLs (current user plus SYSTEM) and handle-based path admission. You never build the addon. Reparse points, alternate streams, UNC paths, SUBST-style aliases and non-NTFS filesystems are rejected.
- **Identity:** the PE header and SHA-256 come from the same open handle. The snapshot is streamed from that handle, and the bridge refuses to serve until the imported program's SHA-256 matches.
- **Transport:** an authenticated listener on `127.0.0.1` with an ephemeral port and a random 256-bit bearer token (redacted from diagnostics). The child process is assigned to the Job Object while still suspended, with kill-on-close.
- DLLs are imported as immutable copies. REA never loads the DLL or runs its entry routine.

```powershell
$env:GHIDRA_INSTALL_DIR = "C:\tools\ghidra_12.1.4_PUBLIC"
$env:JAVA_HOME = "C:\tools\jdk-21"
$env:REA_ANALYSIS_PROVIDER = "ghidra"
rea doctor --json
rea providers --json
rea inspect "C:\fixtures\sample.exe" --provider ghidra --format json
```

> [!warning] Not a sandbox
> The private runtime protects REA's files. Ghidra still runs with your user authority, and if the owner process dies abruptly it can leave private runtime files behind. A cold import can exceed a client's default request timeout.

## DOS MZ and COM

- MZ imports use `MzLoader`, language `x86:LE:16:Real Mode`, compiler `default` and load segment `0x1000`. The classifier validates page counts, relocations and the entry point. NE/LE/LX are explicitly unsupported.
- **COM has no header**, so you must select it explicitly with `--target-format dos-com` or `"format": "dos-com"`. The file must be 1..65280 bytes and is mapped at `1000:0100` (linear `0x10100`).
- Addresses are **linear** (`segment*16+offset`). `procedure_info` reports `body` ranges with **inclusive** ends.
- `inspect_native_load_image` independently verifies the snapshot against the loaded Program and returns `verified`, `mismatch` or `unsupported`.
- No BIOS/DOS-interrupt emulation is performed. For packed programs, analyze a separately unpacked copy and keep both digests.

```bash
rea inspect /abs/legacy.exe --provider ghidra --json
rea function /abs/legacy.com 0x10100 --target-format dos-com --provider ghidra --json
```

## Headerless raw binaries

You supply both the architecture (`x86`, `x86_64`, `arm`, `arm64`) and the address where file offset 0 is mapped. REA never infers either from a `.bin` suffix.

```bash
rea inspect /abs/ecu.bin --target-format raw-binary --architecture arm --entry-point 0x8000000 --provider ghidra --json
```

Imports use `BinaryLoader` with `x86:LE:32:default`/`gcc`, `x86:LE:64:default`/`gcc`, `ARM:LE:32:v7`/`default` or `AARCH64:LE:64:v8A`/`default`. The profile records `language_selection: caller-supplied-v1`. ARM entries must be 4-byte aligned. Big-endian, Thumb-only and Cortex-M are unsupported, and Hopper, IDA and Windows P0 all refuse raw binaries.

## MIPS ELF

- **Standard lane:** ELF32 `ET_EXEC` or `ET_REL`, either endianness, explicit o32 plus `EF_MIPS_ARCH_32R2`, and no ASE or machine flags. The `PT_MIPS_ABIFLAGS`/`SHT_MIPS_ABIFLAGS` record must declare FP32 double/FPXX or soft-float. FP64, NAN2008, ELF64 and n32/n64 are refused.
- **MIPS16 shared profile:** `ET_DYN` with exactly the MIPS16 ASE, using the generic `MIPS:<LE|BE>:32:default` processor so mixed `ISA_MODE` is preserved.
- **Soft-float:** REA installs a derived `__rea_soft_float` prototype via `ReaGhidraMipsSoftFloat.java`, so float arguments are not read from FPU registers the code never wrote.
- Relocatable `.ko` objects get analysis-only addresses.
- Verification toolchain variables: `REA_MIPS_CLANG`, `REA_MIPS_READELF`, `REA_MIPS_AS`, `REA_MIPS_LD`.

## PSP / Allegrex

This lane needs the bring-your-own [ghidra-allegrex](https://github.com/kotcrab/ghidra-allegrex) extension installed at `$GHIDRA_INSTALL_DIR/Ghidra/Extensions/ghidra-allegrex`. User-home extensions are not searched. Admission requires Linux x64 and an LE ELF32 `ET_EXEC` with `e_flags=0x10a23001` and a matching ABI-flags record. REA uses `PspElfLoader` with `Allegrex:LE:32:default`. It fingerprints the extension files (bounded) and binds them to the profile. If the extension is missing, you get `PSP_EXTENSION_UNAVAILABLE` and no fallback. PRX, PBP, ISO and CSO are out of scope.

## RISC-V ELF

The lane admits LE ELF32/ELF64 `ET_EXEC` with `EF_RISCV_FLOAT_ABI_DOUBLE`. It refuses big-endian, RVE, TSO, soft/single/quad float and `ET_REL`/`ET_DYN`. The language is **pinned** to `RISCV:LE:32:default` or `RISCV:LE:64:default` with `gcc`, because Ghidra 12.1.4 otherwise auto-selects an Andes model for RV32. A conflicting `REA_GHIDRA_LANGUAGE_ID`/`REA_GHIDRA_COMPILER_SPEC_ID` is refused. Verification uses `REA_RISCV_CLANG`, `REA_RISCV_LLD` and `REA_RISCV_READELF`.

## NativeAOT metadata

- **Built-in, read-only parser** for x64 PE32+ with .NET 8 RTR 9.1. It works on the exact session snapshot in an in-memory overlay and never defines Ghidra types. Results come through `inspect_native_load_image` (`observations.metadata_recovery`) and `inspect_native_data_type` with a MethodTable address. The parser is adapted from Washi1337/ghidra-nativeaot (MIT).
- **Optional annotation adapter:** a JAR you build yourself, which can create `/NativeAOT/...` DataTypes inside the ephemeral Program. It is unavailable on Windows P0.

```sh
git submodule update --init third_party/ghidra-nativeaot
npm run build:ghidra:nativeaot
export REA_GHIDRA_NATIVEAOT_JAR="$PWD/_reference/nativeaot-integration/extension/rea-ghidra-nativeaot.jar"
```

The JAR's SHA-256 is committed to the profile and checked again before loading. JARs execute inside Ghidra, so supply only one you trust. A PE/CLI or ReadyToRun assembly is not NativeAOT; use `inspect_managed_artifact` for those.

See also [[Firmware]], [[DotNET Assemblies]], [[Native Binaries]].

## Source
- https://github.com/morluto/rea/blob/main/docs/windows-ghidra-p0.md
- https://github.com/morluto/rea/blob/main/docs/ghidra-dos.md
- https://github.com/morluto/rea/blob/main/docs/raw-binary.md
- https://github.com/morluto/rea/blob/main/docs/ghidra-mips.md
- https://github.com/morluto/rea/blob/main/docs/ghidra-psp.md
- https://github.com/morluto/rea/blob/main/docs/ghidra-riscv.md
- https://github.com/morluto/rea/blob/main/docs/ghidra-nativeaot.md
