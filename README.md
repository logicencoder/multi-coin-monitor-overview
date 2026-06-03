# Multi-Coin Monitor — overview

Public description of the **Multi-Coin Monitor** — MEXC spot price dashboard with CEX↔DEX arbitrage ranking. Private code: [multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor).

## What it is

**What:** FastAPI app plus browser UI streaming **200+ MEXC spot pairs** over protobuf WebSockets, computing directional profit estimates vs a Uniswap-style second leg, and highlighting the best spreads in real time.  
**Why:** Manual one-pair tools do not scale when watching many listings for dislocations; operators need a single board with enable/disable per coin.  
**Who:** LogicEncoder operator researching CEX↔DEX gaps; **not** a turnkey auto-trading product for visitors.

## MEXC ingest (production-quality)

**What:** Multiple connections (50 pairs each) to `wss://wbs-api.mexc.com/ws`, aggregated deals and 20-level depth channels, custom protobuf parsing.  
**Why:** MEXC rate limits and frame size require sharded connections; binary protos are mandatory at this scale.  
**Who:** Operator sees live CEX prices that match what traders see on the exchange.

## Arbitrage math (important limitation)

**What:** For each coin, the UI ranks profits for notionals ($100–$1200) in both directions with CEX fee and a fixed gas estimate, using per-coin `buy_tuner` / `sell_tuner` from `stored_coins.json`.  
**Why:** Quick screening of which pairs deserve deeper analysis in real on-chain monitors.  
**Who:** Operator must understand: in this standalone app the **Uniswap leg is simulated** (derived from MEXC × tuners), not read from a live quoter. **Real DEX truth** lives in [eth-chain-swaps-monitor](https://github.com/logicencoder/eth-chain-swaps-monitor) on SOL — do not execute trades from this dashboard alone.

## Dashboard features

**What:** WebSocket `coin_update` stream to the browser, REST filter/sort, per-coin enable flags, profit logging to dated log files.  
**Why:** Reduce noise when only a subset of pairs matters; keep an audit trail of flagged opportunities.  
**Who:** Operator tuning `stored_coins.json` without redeploying code.

## Stack (private repo)

| Piece | Role |
|-------|------|
| `multi_coin_monitor.py` | FastAPI + MEXC WS hub |
| `multi_coin_monitor.html` + `static/multi_coin_monitor.js` | UI |
| `stored_coins.json` | Pair config |
| `generated_proto/` | MEXC protobuf modules |

Default dev URL: `http://127.0.0.1:8765` on SOL/WSL.

## Related repositories

| Repo | Role |
|------|------|
| [multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor) | Private code |
| [eth-chain-swaps-monitor](https://github.com/logicencoder/eth-chain-swaps-monitor) | Real on-chain swap monitoring |
| [mexc_trading_app](https://github.com/logicencoder/mexc_trading_app) | Single-symbol MEXC trading UI |

See [REPOS.md](REPOS.md).

## Licensing

© LogicEncoder.
