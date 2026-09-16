# Crypto World's Fair implementation specification

Specification version: 0.1. Date: 2026-09-16. Target branch: `staging`.

Status: proposed implementation baseline for review. The user requirements recorded here are fixed; selected engineering defaults are explicit proposals, and external integration gates remain verifiable tasks. This is a complete hackathon scope, not a statement that its components are already built.

## Read order and authority

| Document | Responsibility |
| --- | --- |
| [00 Product](00-product.md) | Requirements, scope, experience, delivery order |
| [01 Architecture](01-architecture.md) | Services, registry, models, source imports |
| [02 Vaults](02-vaults.md) | Custody, capital structure, fees, valuation, lifecycle |
| [03 Multichain](03-multichain.md) | Hub/spokes, messages, remote deposits and withdrawals, bridges |
| [04 Venues](04-venues.md) | Hyperliquid, Robinhood, Solana integration contracts |
| [05 Data](05-data.md) | Historical ingestion, collectors, catalog, licensing, live feeds |
| [06 SDK and backtests](06-sdk-backtests.md) | Identical strategy artifact, simulation, reproducibility, reports |
| [07 Runtime](07-runtime.md) | MicroVMs, execution gateway, signing boundary, recovery |
| [08 Product surfaces](08-product-surfaces.md) | Frontend, API, GraphQL, MCP, skills, vault pages |
| [09 Accounts and billing](09-accounts-billing.md) | Auth, wallets, Stripe, entitlements, capacity |
| [10 Infrastructure](10-infrastructure.md) | DigitalOcean, CI/CD, operations, release requirements |
| [11 Delivery](11-delivery.md) | Work packages, dependencies, milestones, acceptance matrix |
| [12 Decisions and sources](12-decisions-sources.md) | Open gates, research evidence, source provenance |

MUST marks a release requirement. SHOULD marks a recommended implementation choice requiring a documented reason to depart. A gate is a concrete proof required before enabling that capability; it is not a request to stop unrelated implementation.

Precedence: explicit user requirements, resolved decision records, this specification, then imported source documentation. Imported Sui documents are references, not authority to reintroduce Sui, AWS hosting, or testnet-first release requirements.

## Product promise

One immutable strategy artifact can be backtested, paper-traded, and deployed against curator-controlled vaults. The platform supplies historical research data, configurable live feeds, execution, isolation, reporting, and public investment pages. AI can operate the same workflow through MCP and skills.

## Completion levels

- **H0:** production foundation, live landing page, accounts, free hackathon access, billing integration.
- **H1:** remotely executed Hyperliquid backtests with the Python SDK and useful reports.
- **H2:** complete Hyperliquid mainnet MVP, initially untranched, with deposit/trade/withdraw demonstrated and identical strategy artifact.
- **H3:** tranched vaults, manual trading, version updates, mature public vault experience, and AI funnel.
- **R1:** Robinhood mainnet spoke with shared hub accounting, USDG deposits, spot router, and withdrawal settlement.
- **S1:** Solana mainnet spoke, Jupiter including Pump.fun, and one selected perpetuals venue.
- **X1:** at least one verified vault-to-vault bridge route and a coordinated multichain strategy demonstration.

H2 is the first usable MVP, not the whole hackathon scope. R1, S1, and H3 are planned hackathon deliverables. X1 is a targeted stretch integration with a fully specified interface; unsupported token routes must not prevent independently funded spokes from functioning.
