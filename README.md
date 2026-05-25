# BusinessGame

A multiplayer supply-chain economics simulation where players run companies inside a three-tier economy. Each calendar day resolves as a game turn: players set prices and standing strategies, and demand flows through the chain via a softmax clearing engine that weights price against KPI scores.

---

## Economy Model

The economy is structured as a directed supply chain with four layers:

```
Production → Distribution → Retail → Public
```

| Tier         | Buys From    | Sells To     | Role                                         |
|--------------|--------------|--------------|----------------------------------------------|
| Production   | —            | Distribution | Manufactures raw goods into finished product |
| Distribution | Production   | Retail       | Warehouses and bulk-moves inventory          |
| Retail       | Distribution | Public       | Last-mile sales to end consumers             |
| Public       | Retail       | —            | Simulated demand pool (NPC)                  |

Players occupy one or more tiers. Multiple players can compete within the same tier — the clearing engine allocates demand across suppliers based on price and KPI score.

---

## Turn Resolution

Turns resolve once per simulated day. Each day:

1. **Strategy phase** — Players set unit prices and standing orders for the coming day (buy limits, sell limits, fulfillment priority rules).
2. **Demand generation** — The public demand pool is computed for the day based on macro conditions.
3. **Clearing** — Demand cascades upward through the chain. At each tier boundary, a **softmax clearing engine** distributes demand across competing suppliers:

   ```
   P(supplier_i) = exp(score_i / τ) / Σ exp(score_j / τ)
   ```

   where `score_i` is a weighted combination of price competitiveness and KPI rating, and `τ` is a temperature parameter controlling how winner-take-all the market is.

4. **Fulfillment** — Allocated orders are filled against available inventory. Shortfalls are tracked and affect reliability KPIs.
5. **Accounting close** — Revenue, COGS, and cash movements are posted; financial statements are updated.

---

## KPI System

Each player's market attractiveness is driven by three KPIs that feed directly into the clearing engine's score:

| KPI           | What drives it                                                  | Decays? |
|---------------|-----------------------------------------------------------------|---------|
| **Reputation**   | Sustained pricing fairness, long-term customer retention     | Slowly  |
| **Reliability**  | On-time and in-full fulfillment rate over a rolling window   | Quickly |
| **Quality**      | Product grade; set at production, propagates down the chain  | No      |

KPI scores are normalised to [0, 1]. A new player starts at neutral (0.5) across all three.

---

## Financial Model

Every player maintains a full set of financial statements, updated at the close of each day:

### Income Statement
- Revenue (units sold × price)
- Cost of Goods Sold (purchase cost + inbound logistics)
- Gross Profit
- Operating Expenses (storage, overhead)
- EBIT / Net Income

### Balance Sheet
- Assets: Cash, Inventory (at cost), Receivables
- Liabilities: Payables, Short-term debt
- Equity: Retained earnings

### Cash Flow Statement
- Operating cash flow (net income ± working capital changes)
- Investing cash flow (capital expenditure)
- Financing cash flow (debt drawdown / repayment)

Financial data is queryable per-player and per-day, enabling historical analysis and in-game strategic decisions.

---

## Multiplayer & Game Loop

- **Asynchronous turns** — players submit strategies before a daily deadline; the engine resolves all turns simultaneously.
- **Persistent world** — the simulation clock advances regardless of player activity; inactive players lose ground to active competitors.
- **Configurable scenarios** — market size, number of tiers occupied by NPCs, demand volatility, and clearing temperature are all scenario parameters.

---

## Project Status

> **Early development.** Tech stack TBD. This repository tracks design, architecture decisions, and implementation as the project is built out.

---

## Repository Structure (planned)

```
BusinessGame/
├── engine/        # Core simulation: clearing, KPI, accounting
├── server/        # Game server, API, session management
├── client/        # Player-facing interface
├── data/          # Scenario configs, seed data
├── docs/          # Design documents and specs
└── tests/         # Test suites
```

---

## Contributing

This project is in its design phase. Open an issue to discuss mechanics, architecture, or tech stack choices before submitting code.

---

## License

TBD
