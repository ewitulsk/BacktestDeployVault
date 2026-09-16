# Product requirements and scope

## 1. Users and outcomes

**Strategy developer/curator:** develops Python with their own editor or AI; uploads an artifact; configures data and execution venues; runs backtests; studies results; creates a vault; deploys the exact artifact; monitors, updates, or manually trades within the mandate.

**Investor:** browses public vaults without a platform account; examines thesis, fees, risk mandate, tranches, and verified live performance; connects a wallet; deposits; tracks positions and withdrawals. A subscription is never required to redeem capital.

**AI client:** explains the product, discovers available data and capabilities, creates strategy code outside the frontend, submits jobs, reads reports, prepares wallet and deployment actions, and creates a vault presentation using the same authorized APIs.

**Operator:** manages ingestion, capacity, releases, incidents, reconciliation, and service health. Operators must not acquire arbitrary withdrawal powers merely by operating infrastructure.

## 2. Fixed requirements

| ID | Requirement |
| --- | --- |
| P01 | Full stack on Hyperliquid first, then Robinhood Chain, then Solana |
| P02 | No Sui contracts, services, or runtime dependencies in the new product |
| P03 | Preserve SuiOptions trading-vault economics and modular audit boundaries, explicitly resolving source inconsistencies before porting |
| P04 | Curator selects untranched or senior/junior capital structure at creation |
| P05 | Hyperliquid native spot/perps; Robinhood spot router only; Solana spot router and one perps venue |
| P06 | Solana spot MUST support Pump.fun, including bonding-curve and graduated trading |
| P07 | One logical multichain vault can have one hub and multiple custody/execution spokes |
| P08 | Rust backend; React/TypeScript/Vite frontend; Python strategy SDK |
| P09 | Same immutable Python artifact for backtest, paper, and live modes |
| P10 | Untrusted backtests and live strategies isolated in microVMs |
| P11 | No per-trade user signature prompts after scoped authorization |
| P12 | All historic/live collected market data and strategy testing reside in DigitalOcean |
| P13 | Frontend contract reads come from indexed backend state |
| P14 | One address authority, via adapted deployment-manager/deployments/token-info |
| P15 | Frontend supports the entire lifecycle except editing strategy source code |
| P16 | Complete API/MCP parity and skills for the user funnel |
| P17 | Production-ready deployment/operations from the first public release |
| P18 | Public-chain vault releases directly target mainnet, backed by extensive testing |
| P19 | Stripe early; hackathon free; sensible backtest budget and one vault/one live strategy per user |
| P20 | Public pricing communicates $20/month research and $50/month deployment; detailed paid allowances remain undecided |
| P21 | Vault platform performance fee remains active during free subscription access |
| P22 | Early landing page, signups, source provenance, and public demo readiness |

## 3. Journey and states

1. Discover the lifecycle and pricing on the landing page; sign up without a card during the hackathon.
2. Choose an example or obtain Python from an AI; upload source/package and locked dependencies.
3. Validate required feeds, instruments/universe selectors, venues, resource budget, and SDK version.
4. Select dataset interval, starting capital, simulated venue costs, feed mapping, and risk constraints.
5. Reserve allowance and run remotely. Show queued/running/failed/completed states and actionable failures.
6. Explore performance, risk, attribution, trades, coverage, and execution assumptions. Compare runs without silently changing their definitions.
7. Create a vault with selected capital structure, public mandate, fees, chain deployments, and withdrawal terms.
8. Seed with the curator's required commitment, authorize scoped execution, and deploy the selected artifact.
9. Publish a public vault page. A separate wallet can invest without becoming a subscriber.
10. Monitor health and performance; pause, manually trade, or upgrade strategy subject to policy; withdraw and close.

The product must distinguish backtested, paper, and live results. A new strategy version cannot inherit a previous version's backtest label as if it had been tested. No positive-return guarantee or implication that a senior tranche cannot lose capital.

## 4. Hackathon and future boundaries

Hackathon includes all three chain integrations, multichain accounting, both capital structures, manual execution, immutable versions, AI tools, billing readiness, basic capacity governance, and production operations. Build the earliest usable path before expanding coverage.

Post-hackathon objectives: purchased long-history datasets after licensing, paid Pyth/Helius provider tiers, specialized low-latency execution, professional sniping performance, extensive venue coverage, richer allocation tooling, and additional bridge routes. The design supports these without advertising them as shipped.

Do not claim four years of history across every venue merely because a provider sells a four-year plan. Coverage is always instrument/venue/dataset-specific.

## 5. Success evidence

Measure signup-to-first-run, first-run-to-second-run, run-to-live-deployment, independent investor deposits, completed withdrawals, report usage, and user-reported execution/data problems. TVL alone does not prove a useful strategy platform.

The submission should show an actual strategy artifact hash used for both backtest and live execution, indexed evidence of transactions, a separate investor depositing, a withdrawal finishing, and an AI operating the same workflow. Each chain expansion needs its own proof; a mocked route is not a completed integration.
