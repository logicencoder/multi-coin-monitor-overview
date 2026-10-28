# Multi-Coin Monitor — Market Screening

Reviewing many markets one pair at a time makes it difficult to decide where to look next. Multi-Coin Monitor brings a tracked collection into a searchable card board with live MEXC information, comparison estimates and visible feed health.

Its role is screening: narrow a larger set of markets to candidates worth investigating, then take that question to the relevant market or trading tools. The Uniswap leg is simulated and must not be mistaken for an executable DEX quote.

## Tech stack

| Layer | Technologies |
|---|---|
| Application | Python, FastAPI |
| Market feed | MEXC, Protocol Buffers |
| Interface | HTML, CSS, JavaScript |

## Start with the collection's current state

The header brings tracked-coin counts, highlighted opportunities, connection activity and received messages together. Connection indicators help establish whether the board is receiving observations before interpreting an apparently quiet market.

A reload control supports bringing the configured coin collection back into the session. The operator can distinguish the tracked selection from the cards currently visible after filtering.

## Find the markets relevant to the question

Search narrows cards by symbol or pair. Minimum-profit, sort-order and card-class controls help focus the board on a particular research task.

Sort by amount, percentage, symbol or update time depending on the question. Include or hide the supported profitable, unprofitable and always-shown classes rather than treating one default view as the only useful comparison.

## Read a card before following its headline

Each card brings the MEXC price and available bid/ask context together with the simulated comparison side. Direction, amount and percentage estimates are shown alongside the observation's current state.

Card appearance distinguishes highlighted, unprofitable, always-shown and stale entries. This helps the operator notice that an old observation and a current market reading require different interpretation.

## Compare sizes and directions

Expanded opportunities present a ladder of trade sizes in both directions, with estimated quantities, amounts and returns. This supports examining how a displayed comparison changes with the proposed size.

The DEX side uses configured simulation adjustments and assumed costs. These are screening scenarios, not evidence of available on-chain liquidity, a firm quote or achievable profit.

## Maintain the tracked selection

The operator controls which coin entries belong in the collection and which are enabled. Per-coin comparison adjustments support testing different scenarios while keeping them associated with the relevant market.

Review the selection and reload it when the research universe changes. That gives repeated screening a consistent starting point without implying that every listed market is suitable for execution.

## Keep observations available for review

Update times, connection information and retained activity support following a session and revisiting highlighted observations. When a candidate needs deeper investigation, the related swap-monitoring tools provide another part of that research workflow.

The board's practical value is prioritisation: compare a broad set, identify a question and inspect it further with an understanding of the estimate's assumptions.

---

Private code: [multi-coin-monitor](https://github.com/logicencoder/multi-coin-monitor)

[Ethereum swap monitoring](https://github.com/logicencoder/eth-chain-swaps-monitor-overview)

See [REPOS.md](REPOS.md) for related repositories.

**[Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
