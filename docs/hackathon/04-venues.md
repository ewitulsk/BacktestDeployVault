# Venue integrations and chain expansion

## 1. Selection and adapter contract

| Environment | Selected direction | Finalization requirement |
| --- | --- | --- |
| Hyperliquid | Native HyperCore spot and perpetuals, hub on HyperEVM | Mainnet contract-controlled execution, valuation, unwind, and withdrawal proof |
| Robinhood Chain | KyberSwap initial spot-router candidate | Compare actual chain-specific liquidity coverage and popularity evidence; prove vault-call compatibility |
| Solana spot | Jupiter | Prove Pump.fun before/after graduation and vault-owned execution |
| Solana perps | Pacifica initial candidate; compare all viable current contenders | Pick widest active market coverage among candidates satisfying custody, API, valuation, and unwind requirements |

Jupiter and Hyperliquid scope are implementation choices. KyberSwap/Pacifica are provisional engineering defaults, not claims of independently proven market leadership. Resolve ADR-001 within the first integration research milestone, before committing to those adapters. Ranking metrics: active tradable markets/liquidity sources, representative route availability, sustainable depth, market-data coverage, authentication and custody support, withdrawal control, and maintenance burden. Preserve dated evidence and exact API/program versions.

Every adapter implements capability discovery, instrument resolution, quote/prepare, validate, submit, reconcile, cancel where supported, raw position/balance observation, valuation inputs, unwind, and return-to-vault. Spot swaps are not falsely modeled as cancellable resting orders. Unsupported semantics return typed capability errors before deployment.

An adapter release includes an audit-boundary document, authorized program/contract references, transaction decoder, allowed destination rules, fee model, failure states, and simulation model. Contract/venue upgrades trigger compatibility review rather than silently accepting changed behavior.

## 2. Hyperliquid first implementation

### Ownership and execution

Hub/adapter owns its HyperCore account and all paths back to vault custody. Bootstrap and fund the account, confirm initialization, then enable trading. Separate EVM funding confirmation from Core account readiness.

Two execution profiles:

1. **Contract-mediated baseline:** constrained hub/adapter calls submit supported actions through CoreWriter. This maximizes enforceable onchain intent checks. Orders are asynchronous; an EVM receipt does not prove order acceptance/fill. Document its latency as measured, including protocol-side delay.
2. **Delegated API execution:** enable only after proving the contract-owned account can authorize/revoke the intended trading agent and that its exact permissions cannot transfer custody assets. Gateway policy protects service use, but a venue agent may have broader trading power than per-order onchain restrictions. Explicitly disclose that trust boundary. Never switch to a service-owned EOA holding pooled principal to make the demo easier.

Profile selection is recorded in vault terms and execution metadata. The first production path can use the verified contract-mediated profile; a faster profile is a separate integration release. The strategy SDK stays unchanged.

Support marketable limit/IOC, resting limit, cancel, cancel-all, reduce-only, client order IDs, tick/size rounding, minimum notional, margin mode where supported, and account-specific fee tiers. Use gateway-issued unique client IDs and persisted nonce domains. Spot inventory and perp margin conversions remain controlled operations, not arbitrary transfers.

### Accounting and observation

Normalize spot available/locked balances, open orders, perp collateral/equity, unrealized P&L, settled funding, realized P&L, fees, liquidation/ADL, and pending class/Core-EVM transfers. Specify whether venue equity already includes collateral and funding to prevent duplication. Verify decimal conversions per instrument.

Read precompiles/approved verified observations provide accounting inputs; the API/indexer provides user views and execution reconciliation. Class transfers, bridge transfers, and CoreWriter submissions have distinct pending states. Prevent share mutations against incoherent in-flight snapshots; a short quiescence/checkpoint barrier is acceptable for the baseline if necessary to price safely.

Initial market support includes native crypto spot/perps. HIP-3 markets are collected where licensed and feasible, but execution is capability-gated by collateral, deployer identifiers, account abstraction, and valuation support. Never interpret a prefixed market as a native asset index.

### Release proof

Contract-owned funds make one spot buy/sell and one perp open/reduce/close; order rejection/cancel is reconciled; margin and funding affect NAV correctly; funds return to deposit-asset custody; an investor withdraws. Restart after an uncertain submit must not duplicate the order. Measure both baseline and optional API profile separately.

## 3. Robinhood expansion

Robinhood means Robinhood Chain, not brokerage accounts or Robinhood's retail trading API. Use the EVM spoke from the imported design after review and removal of Sui bindings. USDG is the initial requested deposit/payout asset, subject to registry/token behavior and oracle verification. Hub accounting remains in its selected accounting asset; conversion is explicit.

Router supports multiple DEX sources; direct Uniswap-only implementation does not satisfy the intended breadth. Compare KyberSwap with current supported alternatives using identical quote baskets, sizes, slippage bounds, and actual executability. A router that supports Robinhood Wallet on another chain is not evidence of Robinhood Chain support.

Backend obtains quotes; the adapter/gateway decodes and validates the executable plan. Bind amount, input/output tokens, minimum output, receiver, spender, deadline, chain, allowed calls, and quote version. Reject calldata that grants unrestricted approvals, changes custody recipient, invokes unknown targets, or exceeds the mandate. Measure actual post-swap balances; never trust reported output alone.

Support token decimals and fee behavior explicitly. Tokens with unsupported transfer taxes, rebasing, hooks, or restrictions are unavailable until an adapter handles them. Router discovery does not automatically authorize an asset. Tokenized real-world assets may have transfer/holder restrictions; expose those as capabilities rather than assuming all tokens are permissionless.

Acceptance: USDG deposit notice and hub issuance, spot purchase and sale, hub valuation of raw spoke holdings, stale-state handling, and USDG withdrawal complete on mainnet. No perps integration is planned on this chain for the hackathon.

## 4. Solana expansion

### Spoke and Jupiter

Use vault PDA-controlled custody with explicit signer/account validation. Every adapter checks account owners, program IDs, expected mint and token program, writable/signer flags, destination accounts, and instruction discriminators. Reject arbitrary instructions or address lookup table substitutions. Resolve lookup tables through trusted backend reads and validate the expanded transaction before signing/submission.

Jupiter integration must use an instruction path compatible with program-owned vault custody; a wallet-only convenience API is insufficient. Verify account/compute limits, transaction size, CPI compatibility, supported token programs, quote expiry, prioritization fees, and post-swap minimum received. Expose costs to simulation and execution journals.

Pump.fun acceptance is explicit: identify and trade an ungraduated curve, identify and trade a graduated pool, cope with graduation between quote and submission, and sell through a valid path. Retry by refreshing state under the same economic intent, never blindly resubmit a stale route. If Jupiter cannot satisfy a necessary path, isolate a direct Pump adapter behind the same spot-router interface; do not silently remove the hard requirement.

### Perpetuals

Enumerate active markets and inspect the custody model of shortlisted venues early. For an API-based venue, prove a PDA/vault-controlled collateral path or a constrained integration with equivalent control. An offchain agent API by itself is not proof that pooled assets can be held safely. Reject candidates requiring unrestricted withdrawal authority in the strategy runtime.

Select the broadest compatible venue, record snapshot counts and exclusions, and implement one adapter. If no candidate meets all requirements, record the specific blocker and keep the capability disabled; the spec does not authorize weakening custody or claiming completed perps support.

### Memecoin valuation and data

Dynamic discovery records launch time, mint, curve/pool transitions, token extensions, and tradability events. Historical universes include failed/delisted tokens. New token selection cannot use information that became known only after the simulated decision.

NAV for illiquid assets needs conservative, manipulation-resistant liquidation value and exposure limits, not last-trade price alone. Quotes at tiny size cannot mark a large holding. If reliable appraisal is unavailable, reject new exposure or segregate it into an explicitly supported mandate/valuation mode; do not disable accounting checks for memecoins. A curator may otherwise extract value by trading against their own pool even without a withdrawal function.

## 5. Bridge and cross-venue demonstrations

Implement the bridge interface in [03 Multichain](03-multichain.md). Prove at least one supported stable-asset route if achievable, with custody-restricted recipients and a real refund/recovery path. UI shows unavailable routes explicitly.

Cross-venue funding strategy uses pre-funded balances, venue-specific funding timestamps, margin reserves, and independently tracked legs. Maximum unhedged exposure, worst-case funding, fees, liquidation distance, and failure of the second leg are simulated before live activation. Bridge latency is a rebalance concern and never presented as atomic execution.
