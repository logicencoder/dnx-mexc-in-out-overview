# DNX MEXC In/Out Monitor

Python worker that watches a **local Dynex node** for transfers touching the known **MEXC hot-wallet prefix**, classifies each movement as exchange **inflow**, **outflow**, or **internal**, and keeps a durable SQLite ledger with hourly, daily, and weekly rollups. Operators use it when block explorers are not enough — you need period totals, catch-up after downtime, and a live tail without opening phpMyAdmin or re-scanning chain history by hand.

The worker is a **standalone terminal tool** (no browser UI). It fits beside other Logic Encoder Dynex and MEXC tooling — treasury tracking, large-transaction alerts, swap web apps — but runs as one script with one local database file.

- **Parallel historical catch-up** — walks missing block heights in batches, skips heights already scanned, then can continue into live monitoring automatically.
- **Direction-aware ledger** — **IN** when DNX arrives at the MEXC prefix, **OUT** when it leaves, **INTERNAL** for prefix-to-prefix hops; every row links to the Dynex block explorer for the transaction hash.
- **Period statistics** — hourly, daily, and weekly totals for inflow, outflow, net flow, and transaction counts; rebuild from raw history when classification rules change.
- **Display threshold** — the terminal can hide dust-sized transfers while the database still stores full amounts for accurate aggregates.
- **Headless or interactive** — CLI flags for automation under Universal Service Manager; a nine-option menu when you launch without arguments.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| Worker | Python 3, `requests`, `sqlite3`, `argparse`, `colorama`, `ThreadPoolExecutor` |
| Chain access | Dynex JSON-RPC on a co-located node (`getinfo`, `getblockbyheight`) |
| Persistence | SQLite — raw transfer ledger, scanned-block index, hourly/daily/weekly rollups |
| Logging | Append-only log file under `logs/` (USM log tail friendly) |
| Operations | CLI and interactive menu; typical production run uses `--scan-last` then live tail |
| Hosting | Self-hosted Linux with a local Dynex daemon |

## Inflow and outflow classification

For every transaction in each block, the worker inspects sender and recipient addresses against the configured MEXC prefix.

| Direction | Meaning |
|-----------|---------|
| **IN** | DNX moved from an external wallet into the MEXC prefix — exchange deposit flow |
| **OUT** | DNX left the MEXC prefix to an external wallet — exchange withdrawal flow |
| **INTERNAL** | Both sides match the prefix — internal exchange routing |

Amounts convert from raw chain units with a fixed coin divisor. Qualifying transfers deduplicate on hash, direction, and amount so replays do not inflate totals.

## Block scanning modes

**Smart scan + auto monitoring** (menu option 1) is the usual operator path: pick a starting block height, scan forward to the chain tip while skipping heights already in the scanned-block index, then continue tailing new blocks without restarting the process.

**Manual range scan** (menu option 2) walks an inclusive start/end range and stops — useful for one-off backfills.

**Real-time only** (menu option 3 or `--realtime`) watches new blocks from the current height with no historical walk.

Historical CLI equivalents:

| Flag | Behaviour |
|------|-----------|
| `--scan-last N` | Scan the last N blocks, then continue live |
| `--scan-from BLOCK` | Start historical scan from a height through the tip |
| `--scan-range START END` | Scan an inclusive block range only |
| `--realtime` | Live tail without history |

Parallel fetch workers batch block downloads; commits batch every few hundred blocks during long catch-up so SQLite stays responsive. Period rollups rebuild at the end of a large historical pass instead of updating on every row mid-scan.

## Statistics and transaction review

**Hourly, daily, and weekly views** (menu options 4–6 or `--stats hourly|daily|weekly`) print period tables: total in, total out, net flow, and transaction counts per period. Optional `--limit N` caps how many recent periods print.

**Recent transactions** (menu option 7) lists the latest ledger rows with block height, direction, amount in DNX, counterparty address snippet, timestamp, and an explorer link.

**Rebuild all statistics** (menu option 9 or `--rebuild-stats`) recomputes period tables from the raw transfer ledger after you change thresholds or fix classification logic.

**Set display threshold** (menu option 8 or `--threshold DNX`) changes only what the terminal prints live — storage always keeps the full amount.

On startup, if the database already contains MEXC flows, the worker prints lifetime inflow, outflow, and net flow before the menu or CLI action runs.

## Headless deployment

For Universal Service Manager or systemd, operators typically run with **`--scan-last`** (for example a few hundred blocks) so the process catches up and enters live monitoring without the interactive menu. The worker waits until the local Dynex node answers `getinfo` before scanning.

Writable directory beside the script holds the SQLite database, checkpoint state, and log output — not committed to git.

Private code: [dnx-mexc-in-out](https://github.com/logicencoder/dnx-mexc-in-out)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
