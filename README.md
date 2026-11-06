# Multi-Coin Monitor — Market Screening

Comparing a coin between a centralised exchange and a Uniswap pool one pair at a time makes it hard to decide where to look next. Multi-Coin Monitor tracks a stored collection of coins at once: the MEXC price arrives live, the price in each coin's Uniswap V2 or V3 pool is read from an Ethereum node, and a card board shows which coins currently differ enough to be worth a closer look. It is a screening board for one operator, and every figure is an estimate calculated from observed prices, not an executable quote.

## Tech stack

| Layer | Technologies |
|---|---|
| Application | Python with FastAPI, a WebSocket feed to the browser |
| MEXC data | MEXC WebSocket with Protocol Buffers messages, bid and ask prices and order-book depth |
| Ethereum data | Web3.py and eth-defi against an Ethereum node, reading Uniswap V2 reserves and V3 pool state |
| Interface | HTML, CSS and JavaScript card board |

## Header and connection state

The header shows the number of tracked coins, how many are currently profitable, the connection status, active connections and the count of messages received. A Reload coins button stops the monitoring, clears the state and reloads the stored collection, which is the way to apply a changed list. The status helps tell a quiet market from a board that is not receiving data.

## Search, filter and sort

A search box narrows the cards by symbol or pair, and a minimum-profit field hides coins below a dollar amount. Switches show or hide profitable and unprofitable coins, so the board can be reduced to the coins with a gap or widened to see everything. Cards sort by profit in USD, profit in percent, symbol or last update, ascending or descending.

## Reading a card

Each card shows the MEXC price with its bid and ask, the Uniswap price of the coin's pool, the direction with the better outcome (buy on MEXC and sell on Uniswap, or the reverse), and the estimated profit in dollars and percent. The MEXC side can follow the last trade or the order book, and each coin has a buy and a sell adjustment factor that tunes the prices it compares. A card is marked stale when one of the two prices is missing or zero, so an old observation is not read as a current one.

## Trade sizes

An expanded card lists the result for nine trade sizes in dollars: 100, 150, 200, 250, 300, 400, 500, 750 and 1,000, in both directions. For each size the MEXC side walks the order book to get the average price and the price impact, and the DEX side is taken from the pool: the constant-product formula with the 0.3 percent fee for V2, and the pool's own price estimate for V3, with its impact. The profit shown is the gap between those two prices at that size. MEXC trading fees and Ethereum gas are not deducted, so the figure is the price difference, not take-home profit, and real execution can differ.

## Maintaining the collection

The stored collection holds, for each coin, its MEXC pair, token address, pool address, pool fee tier and the two adjustment factors. A coin can be enabled or disabled through the API, and the collection is reloaded from storage on request. When a coin needs a deeper look, the Ethereum swap monitoring tools provide the next step.

---

Private code: [multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor)

[Ethereum swap monitoring](https://github.com/logicencoder/eth-chain-swaps-monitor-overview)

See [REPOS.md](REPOS.md) for related repositories.

**[Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
