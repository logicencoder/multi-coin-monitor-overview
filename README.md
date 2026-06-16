# Multi-Coin Monitor

**Screen hundreds of MEXC spot pairs against Ethereum Uniswap pools on one live card board — rank CEX↔DEX spread profit before you open a deeper trading tool.**

**Multi-Coin Monitor** is a FastAPI arbitrage **screener**, not an auto-trader. It shards MEXC protobuf WebSocket subscriptions across parallel connections, refreshes Uniswap V2/V3 quotes on-chain (with Swap event triggers), walks orderbook depth at multiple USD notionals, and paints sortable cards with direction arrows, profit badges, and per-amount leg detail — so you shortlist candidates instead of researching one symbol at a time.

**Read before acting:** Uniswap-side prices use **on-chain quoter math** refreshed on a timer; MEXC fills are **simulated** by walking the live orderbook. Use this board to **shortlist** — on-chain execution truth belongs in [CEX/DEX Arb](https://github.com/logicencoder/cex-dex-arb-overview) or [eth-chain-swaps-monitor-overview](https://github.com/logicencoder/eth-chain-swaps-monitor-overview).

**Made by [Logic Encoder](https://logicencoder.com)**

Private source: [logicencoder/multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor)

---

## What you can do

| Area | In plain language |
|------|-------------------|
| **Live coin grid** | Each card shows MEXC price, Uniswap price, profit USD/%, direction arrow, and expandable amount rows |
| **Fleet scale** | Monitor ~250 enabled pairs with chunked MEXC WebSocket connections |
| **Multi-notional ladder** | Profit at nine USD sizes from $100 through $1000 on each card |
| **Filter and sort** | Search, min profit floor, profitable/unprofitable toggles, sort by profit or symbol |
| **Per-coin enable** | Disable noisy pairs without editing JSON; **Reload Coins** picks up config changes |
| **Stale detection** | Cards dim when MEXC or Uniswap price is missing |
| **Profit journal** | Profitable hits append to dated log files for later review |
| **Pool event refresh** | Uniswap Swap events trigger immediate quote refresh for that pool |

Everything pushes over **browser WebSocket** with REST fallback when the socket is slow.

---

## Feature examples (two per capability)

#### Multi-pair arbitrage screening
1. You start the app — 250 enabled coins load and cards appear as MEXC REST and WS prices arrive.
2. A pair shows a green card when any of nine USD notionals yields profit above your threshold.

#### MEXC WebSocket fleet batching
1. Two hundred fifty coins split into chunks of twelve pairs per connection — parallel `multi_mexc_ws_*` tasks run.
2. One connection drops — only that chunk reconnects with backoff while others keep running.

#### MEXC protobuf market data
1. A binary `limit.depth` message fills bid, ask, and twenty-level books with cumulative qty and USD.
2. An `aggre.deals` frame updates last trade price; the card shows trade-sourced MEXC price.

#### MEXC REST price bootstrap
1. Before WS trades arrive, batched `ticker/24hr` seeds initial `mexc_price` for the fleet.
2. A failed batch logs affected symbols — they may stay empty until WS delivers.

#### MEXC symbol validation
1. Startup `exchangeInfo` removes a coin whose `mexc_pair` was delisted from the monitoring fleet.
2. If MEXC API is down, validation is skipped fail-open — all coins are allowed until the next reload.

#### Universal pool detection
1. A V3 pool caches token0/1, quote symbol, and decimals for quoter calls.
2. An unknown pool architecture skips Uniswap pricing — the card may show MEXC-only until you fix config.

#### V2 on-chain price simulation
1. V2 reserves are read and a $500 buy quote is computed via constant-product math with fee.
2. Zero reserves yield empty prices — the coin lacks `uniswap_price` until liquidity returns.

#### V3 quoter-based DEX simulation
1. `estimate_buy_received_amount` at a pinned block produces buy/sell effective prices per USD tier.
2. When the quoter reverts, fallback slippage percentages approximate the pool — less precise than live quoter.

#### MEXC orderbook depth simulation
1. A $1000 M→U walk consumes multiple ask levels — higher effective buy price and positive impact on the card.
2. A thin book falls back to best ask; impact is still computed against the Uniswap mid.

#### Multi-notional profit ladder
1. $100 shows a small profit on the badge; $500 same direction but different absolute dollars due to depth.
2. You expand the amounts panel — all nine notionals render with per-leg token counts and impact.

#### Buy and sell tuners
1. A coin with `buy_tuner=1.0023` scales Uniswap mid up — M→U edge shrinks vs untuned.
2. Tuners display on the card and persist in `stored_coins.json` per coin id.

#### Directional arbitrage (M→U vs U→M)
1. MEXC cheap vs pool → direction `mexc_to_uni` and an up arrow on the card.
2. Pool cheap vs MEXC bids → `uni_to_mexc` and a down arrow; depth walk applies on the sell leg.

#### Pool Swap event refresh
1. A Swap on a monitored pool triggers immediate Uniswap price refresh for that coin only.
2. When the Ethereum log WebSocket disconnects, the listener retries with doubling delay up to sixty seconds.

#### Live dashboard WebSocket
1. Your browser connects — `initial_data` delivers all coin dicts and header stats in one message.
2. A profitable tick pushes `coin_update` immediately; non-profitable updates rate-limit to once per five seconds per coin.

#### Dashboard filtering and sorting
1. You search `AAVE` — only matching symbols and pairs render in the grid.
2. You sort `profit_usd` descending with min profit $5 — top opportunities float to the top.

#### Per-coin enable and disable
1. You click **Disable** on a card — `stored_coins.json` updates and the coin drops from future chunk lists after reload.
2. You disable without reload — runtime flag flips but memory state persists until **Reload Coins**.

#### Profit and stats file logging
1. A profitable opportunity writes a line with symbol, direction, MEXC/UNI prices, pool, and tuners to the daily profit log.
2. Every five minutes a summary logs TOP-1..3 profitable coins for desk review.

#### Reload Coins fleet restart
1. You edit `stored_coins.json` on disk and hit **Reload Coins** — monitoring stops, config reloads, WS tasks restart.
2. After adding a new pair in JSON, reload brings it into the chunk list without restarting Python manually.

#### Header connection health
1. The corner badge shows connected — header stats tick Messages and Connections counts.
2. After five failed WS reconnects, the client falls back to periodic REST refresh every thirty seconds.

---

## What it does not do

- **Not** auto-trading — screening and logging only; no order placement
- **Not** live on-chain execution per MEXC tick — DEX quotes refresh asynchronously
- **Not** Gate.io in the current monitor code — MEXC + Ethereum Uniswap focus
- **Not** a replacement for full CEX/DEX Arb workstation — use this to **shortlist**

Pair list, tuners, and logs stay on your machine — not published in this overview repo.

---

## Tech stack

| Layer | Technologies |
|-------|----------------|
| Backend | Python 3, FastAPI, uvicorn, asyncio, aiohttp, orjson |
| CEX feed | MEXC Spot v3 REST + protobuf WebSocket |
| DEX | web3.py, eth_defi — Uniswap V2/V3 pool and quoter |
| Ethereum events | WebSocket log subscription for pool Swap events |
| Frontend | Single-page HTML/CSS/JS — no framework |
| Configuration | `stored_coins.json` — token, pool, pair, tuners, enabled flag |
| Logging | Daily app log + `multi_coin_profits_YYYYMMDD.log` |

---

## Quick start

```bash
pip install -r requirements.txt
# Configure stored_coins.json and optional mexc_keys.json
python3 multi_coin_monitor.py   # default http://127.0.0.1:8765
```

Requires reachable Ethereum RPC for Uniswap quotes. See the private repo README and [REPOS.md](REPOS.md).

---

## Related repositories

| Repository | Role |
|------------|------|
| [multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor) | Private application code |
| [multi-coin-monitor-overview](https://github.com/logicencoder/multi-coin-monitor-overview) | This product overview |
| [cex-dex-arb-overview](https://github.com/logicencoder/cex-dex-arb-overview) | Full CEX/DEX arb workstation |

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
