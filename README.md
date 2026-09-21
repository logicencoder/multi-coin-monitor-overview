# Multi-Coin Monitor

FastAPI browser dashboard that streams **hundreds of MEXC spot pairs** over protobuf WebSockets, estimates **CEX↔DEX spread profit** on each card, and ranks opportunities on one sortable board. Built for **screening** many listings at once — not for unattended trading.

Private code: [logicencoder/multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor). Pair list and per-coin tuners live in `stored_coins.json` in the private repo.

Researching CEX↔DEX dislocations one symbol at a time does not scale. Multi-Coin Monitor shards subscriptions (about fifty pairs per MEXC socket), decodes aggregated deals and depth frames, and surfaces the best spreads with per-coin enable switches — so you shortlist candidates before opening a deeper on-chain tool.

- **Live coin grid** — each card shows MEXC price (last trade or order book bid/ask), simulated Uniswap price, best-direction profit in USD and percent, and expandable multi-notional opportunity rows.
- **Fleet controls** — search, minimum profit filter, sort by profit, symbol, or last update, and toggles for profitable, unprofitable, and **hardset** cards (major symbols that stay visible even when spread is flat).
- **Runtime enable/disable** — turn individual coins off without editing JSON; **Reload Coins** picks up config changes and restarts feeds.
- **Stale detection** — cards fade when quotes stop updating so you do not act on dead data.
- **Profit journal** — profitable hits append to dated files under `logs/` for later review.

**Read before acting:** Uniswap-side prices in this app are **simulated** from MEXC using per-coin buy/sell tuners — not a live on-chain quoter. Use the board to **shortlist** pairs. On-chain swap truth belongs in [eth-chain-swaps-monitor-overview](https://github.com/logicencoder/eth-chain-swaps-monitor-overview).

## Tech stack

| Layer | Technologies |
|-------|--------------|
| Backend | Python 3, FastAPI, uvicorn, asyncio, aiohttp, orjson, Decimal math |
| Exchange feed | MEXC Spot V3 protobuf WebSocket (`generated_proto/` decoders) |
| Frontend | Single-page dashboard — HTML, CSS, vanilla JavaScript |
| Configuration | `stored_coins.json` — token address, Uniswap V3 pool, MEXC pair, buy/sell tuners, enabled flag |
| Realtime | Browser WebSocket for `initial_data`, per-coin updates, and stats refresh |
| Logging | Daily rotating app log plus `multi_coin_profits_YYYYMMDD.log` for profitable hits |
| Hosting | Local development bind on port **8765** by default |

## Dashboard header and connection health

The sticky header shows **Total Coins**, **Profitable** count, active **Connections** (MEXC socket shards), and **Messages** received. A connection badge in the corner flips between connected and disconnected; the client reconnects with exponential backoff when the socket drops.

**Reload Coins** re-reads `stored_coins.json`, restarts WebSocket subscriptions for enabled entries, and clears the grid before fresh data arrives. If the socket is slow on first load, the UI falls back to a REST refresh after a few seconds.

## Filter and sort bar

| Control | What it does |
|---------|----------------|
| **Search** | Filters by symbol or MEXC pair substring |
| **Min Profit ($)** | Hides cards below a USD profit floor |
| **Sort By** | Profit USD, profit percent, symbol, or last update time |
| **Order** | Ascending or descending |
| **Show Profitable / Unprofitable / Hardset** | Checkbox toggles for card classes |

The grid caps visible cards at five hundred rows after filtering so the browser stays responsive during full-fleet runs.

## Coin cards

Each card uses border colour to signal state: green tint for profitable spreads, red for unprofitable, orange for **hardset** majors (ETH, BTC, stablecoins, and similar symbols always eligible to show), and dimmed grey when **stale**.

**Price row** — MEXC side shows last trade or order-book mode with bid/ask under the headline when depth is available. Uniswap side shows the tuned simulated price used for spread math.

**Profit row** — headline USD and percent for the best direction (**MEXC → Uniswap** or **Uniswap → MEXC**). Click the profit section to expand **Trading Opportunities**: a table of notionals from about **$100 through $1200** with direction shorthand, token quantity, profit, and ROI per row.

**Tuners line** — displays the coin’s configured buy and sell tuner multipliers from JSON.

**Enable / Disable** — toggles whether that coin stays in the live feed without restarting the whole process.

When a card crosses the profit threshold, the backend can log a structured line to the daily profit file (symbol, pair, direction, MEXC and Uni prices, tuners, pool reference).

## Arbitrage estimate (screening only)

For each enabled coin the backend evaluates both directions across the standard notional ladder. Calculations assume a **0.2% MEXC fee** and a **fixed USD gas estimate** on the DEX leg. Per-coin **buy_tuner** and **sell_tuner** shift the simulated Uniswap price relative to MEXC so you can model conservative or aggressive Uni legs without editing code.

Profitable flag uses a configurable USD floor (default about **$1** best profit). Cards sort by best profit when you choose **Profit USD** descending — the usual workflow for finding candidates to research in [eth-chain-swaps-monitor](https://github.com/logicencoder/eth-chain-swaps-monitor-overview).

## Coin fleet configuration

Edit `stored_coins.json` in the private repo to add or remove entries. Each row carries an numeric **id**, Ethereum **token_address**, **uniswap_v3_pool**, **mexc_pair**, optional Gate pair label, **buy_tuner** / **sell_tuner**, and **enabled**. After saving, use **Reload Coins** on the dashboard or restart the process.

## Run locally

```bash
pip install -r requirements.txt   # from private repo
python3 multi_coin_monitor.py
```

Open the dashboard at `http://127.0.0.1:8765` on the machine running the worker.

Private code: [multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor)

See [REPOS.md](REPOS.md).

---

## Operator workflows

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

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
