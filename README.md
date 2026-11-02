# Multi-Coin Monitor

FastAPI browser dashboard that streams **hundreds of MEXC spot pairs** over protobuf WebSockets, estimates **CEX↔DEX spread profit** on each card, and ranks opportunities on one sortable board. Built for **screening** many listings at once — not for unattended trading.

Private code: [logicencoder/multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor). Pair list and per-coin tuners live in `stored_coins.json` in the private repo.

**What:** FastAPI app plus browser UI streaming **200+ MEXC spot pairs** over protobuf WebSockets, computing directional profit estimates vs a Uniswap-style second leg, and highlighting the best spreads in real time.  
**Why:** Manual one-pair tools do not scale when watching many listings for dislocations; operators need a single board with enable/disable per coin.  
**Who:** Logic Encoder operator researching CEX↔DEX gaps; **not** a turnkey auto-trading product for visitors.

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

© Logic Encoder.
