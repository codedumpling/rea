---
tags: [rea, target, sqlite]
source:
  - docs/sqlite-databases.md
  - src/sqlite/
---
# SQLite Databases

`inspect_sqlite_database` / `rea inspect-sqlite-database` inspects one local SQLite database snapshot (plus its sibling `-wal`, if present) to understand application storage, configuration or retained evidence. No separate SQLite installation and no active native target are needed.

## What REA returns

- **Schema** in one of three scopes:
  - `inventory` (default without a table): compact table metadata and index/view/trigger names.
  - `selected_table` (default with a table): that table plus its indexes and triggers.
  - `database` (explicit): every stored declaration and supported column.
  SQL declarations keep their original text; virtual and shadow tables are labeled.
- **Optional rows** from one ordinary table: selected columns, `row_limit` (default 20), returned count and whether more rows exist.
- Lossless values: integers as decimal strings, `"Infinity"`/`"-Infinity"`/`"-0"` preserved, text with its bytes as hex plus database encoding, blobs as hex, SQL NULL as a tagged value.
- Separate SHA-256 and byte counts for the database and `-wal`; SQLite engine version.

## Requirements

> [!warning] Newer Node.js than the rest of REA
> Requires **Node.js 24.x >=24.15 or 26+** with the native `DatabaseSync.limits` API. Other REA features keep the general Node 22/24/26 requirements.

`rea capabilities --json` reports `inspect_sqlite_database` availability without opening a database; unsupported runtimes show `available: false`, `availability_code: "unsupported_version"`. See [[Readiness Checks]].

## CLI and MCP

```sh
rea capabilities --json
rea inspect-sqlite-database ./application.sqlite --json
rea inspect-sqlite-database ./application.sqlite --table settings --column key --column value --row-limit 20 --json
rea inspect-sqlite-database ./application.sqlite --schema-scope database --json
```

```json
{
  "name": "inspect_sqlite_database",
  "arguments": {
    "path": "/artifacts/application.sqlite",
    "table": "settings",
    "columns": ["key", "value"],
    "row_limit": 20
  }
}
```

MCP paths must be absolute; CLI paths resolve against the working directory.

## Worked example

> [!example] What does a desktop app store locally?
> 1. Quit the app (or use its backup workflow) so the database/WAL pair is quiescent.
> 2. `rea inspect-sqlite-database ~/.../app.db --json` → table inventory.
> 3. `rea inspect-sqlite-database ~/.../app.db --table settings --json` → expanded schema + 20 sample rows.
> 4. Narrow with repeated `--column` and raise `--row-limit` as needed.

## Limits and side effects

- **Source files are never opened by SQLite.** REA copies the database and existing WAL into an owned private temp directory first, so locks and `-shm` sidecars cannot touch the original. Extension loading and schema trust are disabled. Copies are cleaned on success, failure and cancellation.
- A file copy of a **live** pair is not an atomic snapshot — stop the writer first.
- Non-empty rollback journals (other than empty/zeroed headers left by `TRUNCATE`/`PERSIST`) are refused.
- No arbitrary SQL, view/virtual/shadow row reads, encrypted databases, repair, cross-database queries or live capture. Schema inventory never executes views or triggers.
- Row order is unspecified; a truncated sample proves nothing about omitted rows.
- Bounds: database + WAL ≤ 256 MiB; values/statements/worker reply ≤ 16 MiB each; worker 256 MiB JS heap and 30-second deadline. At most two concurrent inspections per provider (typed capacity error otherwise).
- Large Evidence can exceed MCP's response budget; export retained Evidence or use CLI JSON (see [[Evidence Export and Import]]).

> [!info] Testing
> `npm run test:sqlite` (or `npm run test:sqlite:run` when artifacts are current). CI tests Node 22.x unavailable diagnostics, 24.15.0, and latest 24.x/26.x. See [[Testing]] and [[CI Workflows]].

## Related
[[Targets Overview]] · [[Evidence Model]] · [[Process Capture]] (observe files an app writes)

## Source
- https://github.com/morluto/rea/blob/main/docs/sqlite-databases.md
