# Delivery plan and acceptance criteria

## 1. Sequencing and ownership

Target submission: October 12, 2026. Dates below are planning targets, not promises of completed work. Workstreams run concurrently once shared interfaces are frozen; each package has one accountable team owner assigned in the issue tracker.

| Window | Priority outcome |
| --- | --- |
| Sep 16-18 | Provenance import, interface freeze, chain/transport/runtime feasibility, production skeleton, landing/signup, Stripe skeleton |
| Sep 19-23 | Hyperliquid historical/live data, SDK and remote reports, vault hub/gateway implementation, end-to-end synthetic path |
| Sep 24-28 | Hyperliquid mainnet MVP; deposit/trade/withdraw; same artifact live; production recovery drills |
| Sep 29-Oct 3 | Tranched experience, manual trading/versioning, Robinhood spoke/router/shared accounting |
| Oct 4-8 | Solana spoke, Jupiter/Pump.fun and selected perps; bridge route if verified; full AI parity |
| Oct 9-12 | User fixes, load/recovery evidence, public demo/submission, provenance and new-work summary |

Begin Robinhood/Solana feasibility in the first window even though release order is sequential. Do not discover an incompatible custody model in the final week. Keep the working Hyperliquid product releasable while expansion work is behind capability flags.

## 2. Work packages

| ID / owner discipline | Deliverable | Dependencies | Done when |
| --- | --- | --- | --- |
| W01 / platform | Provenance imports and repo hygiene | Source snapshots/licenses | Imports isolated, no data/credentials/state, Sui-dependent boundaries inventoried |
| W02 / platform | Registry, token-info, deployment manager | W01, schema agreement | All consumers resolve one authority; EVM deploy/verify path; Solana backend scheduled |
| W03 / operations | DO adoption and CI/CD | W01 | Verified deployment, monitoring, restore, rollback; collector continuity |
| W04 / product | Landing, signup, accounts, Stripe | W03 baseline | Public signup free without card; test billing lifecycle works |
| W05 / data | Reservoir ingest, catalog, live collectors | W01/W03 | Remote partitions/versioned coverage and feeds; gaps visible |
| W06 / quant-runtime | SDK, event protocol, simulator | W05 schemas | Immutable artifact remote replay, reproducibility, correct order lifecycle |
| W07 / runtime | MicroVM builds/scheduler/gateway | W02/W03/W06 interfaces | Isolation, bounded capacity, fenced recovery, autonomous authorized orders |
| W08 / contracts | HyperEVM hub and economic library | W01/W02, ADR-003 for tranches | Invariants and reference parity; deposit/withdraw/fees; capital modes |
| W09 / integrations | HyperCore adapter and indexer | W02/W07/W08 | Mainnet spot/perp lifecycle and indexed reconciled NAV |
| W10 / product-quant | Reports and research UX | W05/W06 | Useful charts/metrics, lineage, approved export, comparisons |
| W11 / product | Curator/investor surfaces | W04/W08/W09/W10 | Public browse, create, deploy, invest, withdraw without source editor |
| W12 / AI | MCP and skill suite | Shared API schemas | Same journey and permissions as UI, chart/result analysis |
| W13 / contracts-integrations | Messaging and Robinhood spoke/router | W02/W08, transport/custody gate | Shared NAV, USDG spot execution, cancellation/payout races tested |
| W14 / contracts-integrations | Solana spoke/Jupiter/perps | W02/W08/W13 wire protocol, venue gate | Pump lifecycle, perps and shared accounting proven |
| W15 / integrations | Vault bridge/rebalance | W13/W14 where route supported | One route conserves value through success/refund/partial settlement |
| W16 / product-runtime | Manual trading and safe strategy updates | W07/W11 | Shared gateway policy, version lineage, notice and recovery behavior |
| W17 / verification | Adversarial/economic/recovery review | Continuous | Evidence for every enabled feature; no unacknowledged severe findings |
| W18 / product | Submission and user evidence | H2 baseline, expansion milestones | Honest demo, source/new-work disclosure, architecture, pricing, traction |

## 3. First 48-hour decisions

Freeze the canonical registry schema, SDK event/intent protocol, economic state-machine interface, and job/artifact IDs. Prove DO microVM host support; HyperCore contract-owned trade/unwind path; exact messaging lanes; Robinhood router coverage; Solana perps custody; and USDG valuation source. These are bounded engineering investigations with dated evidence, not open-ended research.

Resolve perps/router choices early. If a candidate fails, compare the next compatible candidate without changing the product's custody requirement. Record decisions and rationale. Keep a visible dependency board so independent frontend/data work continues.

## 4. Release acceptance matrix

| Test ID | Scenario | Milestone |
| --- | --- | --- |
| A01 | Fresh signup, grant, backtest allowance, billing test lifecycle | H0 |
| A02 | Single address authority, generated/client consistency, missing registry fails closed | H0 |
| A03 | Existing DO collector survives adoption; deployment and restore documented | H0 |
| A04 | Upload -> validate -> remote microVM backtest -> charts/tables | H1 |
| A05 | Repeated manifest/artifact/seed yields same result; future data inaccessible | H1 |
| A06 | Data gaps, sampled-depth limitations, fees/slippage/funding reported | H1 |
| A07 | Same artifact hash executes in paper and live with no source edits | H2 |
| A08 | Hyperliquid mainnet spot and perps, partial/cancel/reject, withdraw | H2 |
| A09 | Worker dies after submit; restart reconciles without duplicate exposure | H2 |
| A10 | Different wallet invests and redeems without platform subscription | H2 |
| A11 | Platform/curator fees and basis conserve assets and shares | H2/H3 |
| A12 | Senior/junior waterfall, impairment, buffer, reset, settlement vectors | H3 |
| A13 | Manual/automated conflict control and immutable strategy updates | H3 |
| A14 | UI and MCP complete equivalent authorized journeys | H2 basic, H3 complete |
| A15 | Robinhood deposit/router trade/withdraw participates in global NAV | R1 |
| A16 | Delayed deposit ACK versus refund race cannot create unbacked shares | R1 |
| A17 | Duplicate/out-of-order messages, snapshot watermarks, partial payouts | R1 |
| A18 | Solana vault-controlled Jupiter trade before/after Pump graduation | S1 |
| A19 | Selected Solana perp adapter opens/reduces/closes and returns collateral | S1 |
| A20 | Simultaneous deposits across chains preserve one supply and tranche books | R1/S1 |
| A21 | Bridge partial receipt/refund never doubles NAV or changes recipient | X1 |
| A22 | Subscription expiry/capacity exhaustion does not strand withdrawals | H2 |
| A23 | Guest cannot access host/other tenant/credentials/raw lake export | H1/H2 |
| A24 | Direct frontend contract reads absent; indexed lag shown | All |
| A25 | No real market data on laptops or CI runners | All |
| A26 | Curator attacks via hostile recipient, manipulated pool/price, strategy limits | Every relevant venue |
| A27 | Capital epoch fences concurrent remote trading and reconciles all snapshot/message watermarks before issuance | R1/S1 |
| A28 | Maximum supported portfolio can complete appraisal/settlement within gas/compute limits | Every contract release |

Evidence records include commit/artifact/registry versions, environment, test seed or remote run ID, observed results, and links to transactions where appropriate. Do not commit sensitive payloads or raw market datasets as evidence.

## 5. Economic reconciliation examples

Use exact integer-scaled variants of these examples as shared tests:

- **Pending deposit:** vault NAV 1,000; remote 100 remains pending, so shareholder NAV remains 1,000. Acceptance creates the corresponding shares and recognizes 100 exactly once; delayed activation cannot add another 100.
- **Withdrawal:** assets 1,000; fulfill a 100 claim and burn its shares; assets stay 1,000 while liabilities become 100, so remaining NAV is 900. Pay 40: assets 960, liability 60, NAV 900. Final 60: assets 900, liability zero, NAV 900. Receipt/snapshot ordering cannot change that result.
- **Bridge:** source assets 500 plus other assets 500; send 100 into a receivable with expected fee 2. After final recognition, source 400 + other 500 + receivable 98 = 998. On destination settlement, replace receivable with 98 destination assets. Partial refund follows its exact proved amounts.
- **Depeg:** asset and payable quantities revalue consistently; a fixed USDG liability does not become fixed USD value merely because the hub accounts in USD terms.
- **Tranche loss:** NAV 1,000, senior claim 600; a 300 loss leaves senior 600/junior 100 in preferred-only mode. A further 200 loss leaves senior 500/junior zero. Test participatory modes separately.

## 6. Mainnet readiness

Before each adapter/vault capability opens to users: required invariant/adversarial tests pass, addresses/code/interfaces are verified in the registry, permissions are checked, recovery/unwind and withdrawals work, indexers are current, operations have alerts/runbooks, and a small end-to-end mainnet lifecycle is evidenced. Exposure caps expand only after the relevant checks. Do not equate extensive tests with an external audit.

New features cannot bypass a failed prerequisite to meet the demo deadline. If a dependency is unresolved, leave the feature visibly unavailable and document the missing proof; preserve the working release and honestly report incomplete hackathon scope. Multi-chain requirements remain tracked rather than being silently reclassified as post-hackathon.

## 7. Submission package

Provide product URL, public vault examples, concise demo video, architecture diagram, supported-capability matrix, pricing path, user/usage evidence, source import manifest, exact new-work summary, and known limitations. Demonstrate useful research even when a strategy loses money. Show that backtested returns and live returns are different datasets and that the team can operate the product after submission.
