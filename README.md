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

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
