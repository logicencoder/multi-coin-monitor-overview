# Multi-Coin Monitor

FastAPI dashboard that streams **200+ MEXC spot pairs** over protobuf WebSockets, ranks directional **CEX↔DEX profit estimates**, and pushes live updates to a browser board. Built for screening many listings at once — not for unattended trading.

Private source: [logicencoder/multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor). Pair configuration lives in `stored_coins.json` in the private repo.

## The problem it solves

Watching one symbol at a time does not scale when you research CEX↔DEX dislocations across a long token list. Multi-Coin Monitor opens sharded MEXC connections (50 pairs each), parses aggregated deals and 20-level depth frames, and surfaces the best spreads on one sortable dashboard with per-coin enable flags.

## MEXC ingest

Multiple WebSocket connections to `wss://wbs-api.mexc.com/ws` with protobuf decoding via `generated_proto/`. This path is **production-quality** — the same binary feed traders see on the exchange.

## Arbitrage ranking (read this before acting)

For each enabled coin the UI computes multi-notional profits ($100–$1200) in both directions (MEXC→Uni and Uni→MEXC) with a 0.2% CEX fee and a fixed gas estimate, using per-coin `buy_tuner` / `sell_tuner` from config.

The **Uniswap-side price in this standalone app is simulated** — derived from MEXC with tuners, not from a live on-chain quoter. Use this board to **shortlist** pairs worth deeper analysis. On-chain swap truth belongs in [eth-chain-swaps-monitor](https://github.com/logicencoder/eth-chain-swaps-monitor-overview) — do not execute trades from this dashboard alone.

## Dashboard

- WebSocket `coin_update` stream plus REST filter/sort on `/api/coins`
- Enable or disable coins at runtime; reload config without redeploying code
- Profitable hits logged to dated files under `logs/`
- Default bind: `http://127.0.0.1:8765`

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/` | Dashboard HTML |
| GET | `/api/coins` | Filter, sort, search |
| GET | `/api/stats` | Connection and message totals |
| POST | `/api/enable_coin/{id}` | Toggle coin |
| GET | `/api/reload_coins` | Reload JSON and restart WS |
| WS | `/ws` | `initial_data`, `coin_update` |

## Quick start

```bash
python3 multi_coin_monitor.py
```

Edit `stored_coins.json` for `token_address`, `mexc_pair`, tuners, and `enabled` flags. See [REPOS.md](REPOS.md) for related repos.

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
