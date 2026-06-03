# DNX MEXC In/Out Monitor (overview)

Tracks **Dynex (DNX) transfers to and from the MEXC exchange wallet pattern** by scanning blocks from your own Dynex node. Built for operators who need reliable **exchange inflow/outflow** history without manually reading the block explorer.

**Code (private):** [logicencoder/dnx-mexc-in-out](https://github.com/logicencoder/dnx-mexc-in-out)  
**Delivery:** SOL only — Python worker + SQLite, supervised via Universal Service Manager

---

## What

| Capability | Detail |
|------------|--------|
| **Block scan** | Parallel fetch from local Dynex RPC; skips already scanned heights |
| **IN / OUT classification** | Labels flows relative to the MEXC prefix address |
| **SQLite history** | Every transfer stored; terminal can hide small amounts under a DNX threshold |
| **Period stats** | Hourly, daily, weekly aggregates (in, out, net) |
| **Catch-up + live** | `--scan-last N` then continues tailing new blocks (USM default: 200) |
| **Operator menu** | Interactive stats, recent txs, rebuild aggregates |

---

## Why

MEXC-related DNX movement is a useful signal for treasury and market ops. Block explorers do not give you a dedicated, queryable **exchange flow ledger** with rollups. This worker keeps that ledger on your server next to `dynexd`.

---

## Who

- **Operator** running Dynex + MEXC workflows on SOL  
- **Not** a public website product — no WordPress plugin in this repo  
- Pairs logically with **DNX swap** and **large-tx** monitors, but separate codebase

---

## Stack

Python 3, `requests`, SQLite, local Dynex node (`getblockbyheight` JSON-RPC). See private `ARCHITECTURE.md` for tables and CLI flags.
