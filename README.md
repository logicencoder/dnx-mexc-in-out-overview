# DNX MEXC In/Out Monitor

Python worker that scans a **local Dynex node** for transfers involving the known **MEXC hot-wallet address pattern**, classifies them as exchange **inflow or outflow**, and keeps a queryable SQLite ledger with hourly, daily, and weekly rollups.

Private code: [logicencoder/dnx-mexc-in-out](https://github.com/logicencoder/dnx-mexc-in-out). This public overview describes capabilities only — no RPC credentials or database files.

## The problem it solves

Block explorers show individual transactions, but they do not give you a dedicated **exchange flow ledger** with period totals. When you track Dynex treasury or MEXC-related market activity, you need a continuous record of how much DNX moved toward or away from the exchange — including catch-up after downtime and live tailing of new blocks.

## How it works

The worker connects to your own Dynex JSON-RPC (`getblockbyheight`), walks blocks in parallel, and skips heights already recorded. Each transfer touching the MEXC prefix is stored in SQLite with direction **IN** or **OUT** relative to that prefix.

The terminal can hide small amounts below a display threshold, but **every transfer is still stored** in the database for accurate aggregates.

## Capabilities

### Block scanning

Historical modes cover the last *N* blocks, a fixed height range, or a starting block through the chain tip. After catch-up, the same process can continue tailing new blocks in real time.

### SQLite history

Persistent tables hold raw MEXC-related transfers plus pre-aggregated hourly, daily, and weekly stats (total in, total out, net flow, transaction counts). A rebuild command recomputes period tables from the raw ledger if classification logic changes.

### Operator CLI and menu

Run with scan flags for automation, or launch without flags for an interactive menu — recent transactions, period stats, and aggregate rebuilds from the terminal.

Common flags:

| Flag | Purpose |
|------|---------|
| `--scan-last N` | Scan the last N blocks, then continue live |
| `--scan-from BLOCK` | Start historical scan from a height |
| `--scan-range START END` | Scan an inclusive block range |
| `--realtime` | Live tail only |
| `--stats hourly\|daily\|weekly` | Print aggregated in/out |
| `--rebuild-stats` | Rebuild period tables from stored transactions |
| `--threshold DNX` | Terminal display cutoff (storage unchanged) |

### Requirements

- Python 3 with `requests`
- A reachable **local Dynex node** (JSON-RPC)
- Writable directory for the SQLite database and checkpoint file

## Related Logic Encoder work

This monitor complements other Dynex and MEXC tooling (swap web apps, large-transaction alerts, trading stacks) but stays a **standalone worker** — one script, one database, no browser UI.

See [REPOS.md](REPOS.md) for repository links.

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
