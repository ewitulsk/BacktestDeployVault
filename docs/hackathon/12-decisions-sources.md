# Decision register, evidence, and source provenance

## 1. Decision status

These decisions distinguish fixed requirements from proposals and external facts. Update this register as investigations conclude; include evidence rather than replacing an unknown with an assumption.

| ID | Decision / default | Status | Owner discipline / deadline | Required resolution |
| --- | --- | --- | --- | --- |
| ADR-001 | Jupiter; HyperCore spot/perps; KyberSwap and Pacifica initial candidates | Partly fixed, partly provisional | Integrations / first 48 hours | Dated breadth/custody comparison; freeze Robinhood router and Solana perps choice |
| ADR-002 | HyperEVM hub with Robinhood/Solana spokes | Proposed architecture baseline | Contracts / interface freeze | Confirm hub custody/accounting path; no Sui runtime |
| ADR-003 | Preserve source v2 economics; reconcile ambiguities | Blocking for tranched release | Contracts/quant / before tranche implementation freeze | Exact hurdle recurrence, reset math, queue/ACK timing, fee and commitment parameters, terminal rounding |
| ADR-004 | External isolated signing capability and scoped runtime authority | Required boundary; provider undecided | Runtime/security / before live execution | Review imported signer, custody model, delegation proof, revocation/recovery |
| ADR-005 | OIDC account login plus wallet linking | Proposed default | Platform / H0 | Select identity provider, recovery/session behavior; do not require investor enrollment |
| ADR-006 | MicroVMs on DO worker pool, prefer Firecracker | Requirement plus candidate | Operations / first 48 hours | Host KVM/nested virtualization proof, isolation tests, capacity benchmark |
| ADR-007 | Transport abstraction, one active authenticated verifier per lane | Fixed design; provider lane selection gated | Contracts/integrations / first 48 hours | Exact directional mainnet support and retry/recovery evidence |
| ADR-008 | USDG on Robinhood; hub accounting asset selected/pinned at creation | User requirement plus valuation gate | Contracts/data / before R1 | Token behavior, robust valuation, fees/decimals, supported payout/rebalance path |
| ADR-009 | $20 research/$50 deployment; proposed inclusive deployment tier | Prices fixed; packaging proposed | Product / before paid activation | Final packaging and grace terms; keep usage detail internal for hackathon |
| ADR-010 | One logical vault family/one live deployment hackathon grant | Baseline interpretation | Product/platform / H0 | Freeze measured research allowance and platform capacity |
| ADR-011 | At least one bridge route targeted, otherwise independently funded spokes | Stretch integration | Integrations / before X1 | Actual tokens/lane, custody recipient, refund semantics, accounting proof |
| ADR-012 | Data licenses cover hosted computation, outputs, AI use | Gate per provider | Product/data / before source enabled | Record permitted usage; obtain written terms where needed |
| ADR-013 | Material strategy changes require published notice/exit or new vault | Required policy; duration undecided | Product/contracts / before public creation | Fix notice duration and mandate mutability in terms; no retroactive changes |
| ADR-014 | Transferable roles and upgradeable contracts; ERC-1967/UUPS baseline for EVM, native upgrade authority for Solana | User requirement; implementation baseline specified | Contracts/platform / initial contract architecture | Inventory all roles, prove multisig handoff, validate storage/migrations, and preserve upgradeability and custody identity |

No hardcoded addresses or credentials belong in decision records. Refer to logical registry IDs and evidence transaction IDs.

## 2. Source code inspected

Repository: [ewitulsk/SuiOptions](https://github.com/ewitulsk/SuiOptions).

| Reference | Inspected revision/context | Useful material |
| --- | --- | --- |
| `ewitulsk/backtest` local checkout | HEAD `6ddcf5ea4c58ade58f390f6e8097c73171d1545b`; substantial uncommitted/untracked work observed | React/Vite frontend, v2 capital spec, data room, backtester, auth, token-info, deployment-manager, DO operations |
| `claude/multichain-trading-vault-wer9q0` fetched branch | `dde8520377406dce33c3f2721da78804580024de` | Sui hub reference, Solidity spoke, wire schema, messenger, transport interfaces, tests, multichain plan/runbook |
| BacktestDeployVault `origin/staging` | `fd40b84672467d984acc3eb1e5b87c7d00919efa` before this documentation change | Initial README only |

The backtest SHA does not include local edits. Future import must pin its own selected snapshot and patch manifest. These are research references, not an assertion that source code was imported by this PR. Source docs describing production interfaces also list incomplete live integration checks; preserve that distinction.

Important source findings: global accounting already lives on the hub; Solidity currently implements the spoke, not the whole hub; the original design excludes in-flight bridging from its initial implementation; and timeout-only deposit reclamation needs the stronger handshake in this spec. Source economic hurdle wording and recurrence require reconciliation before a faithful port.

## 3. External evidence checked for this specification

Checked 2026-09-16. Provider pages and supported networks can change; revalidate implementation-critical details at adapter release. No external market dataset was downloaded during specification work.

| Source | Narrow fact used / limitation |
| --- | --- |
| [World's Fair](https://colosseum.com/worldsfair) and [official rules](https://colosseum.com/legal/Crypto%20World%27s%20Fair%20Hackathon%20Rules.pdf) | Submission deadline October 12, 2026; judging includes functionality, impact, novelty, UX, open source, business plan |
| [Colosseum eligibility FAQ](https://colosseum.com/hackathon) | Prior code may be used with disclosure; work during the competition is judged |
| [HyperCore interaction](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/interacting-with-hypercore) | Read precompiles and CoreWriter expose contract interaction paths; order actions have asynchronous processing and documented delay; prove chosen mainnet actions |
| [HyperCore/EVM timing](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/interaction-timings) | Account must exist before the EVM block; same-block funding is not sufficient to assume initialization before an action |
| [Hyperliquid architecture](https://hyperliquid.gitbook.io/hyperliquid-docs/hyperevm) | HyperEVM and HyperCore are distinct execution surfaces of Hyperliquid, not interchangeable APIs |
| [Jupiter quote documentation](https://developers.jup.ag/docs/swap/v1/get-quote) | Lists Pumpfun Bonding Curve routing; does not prove immediate availability for every token or compatibility with our vault transaction shape |
| [Pacifica market info](https://pacifica.gitbook.io/docs/api-documentation/api/rest-api/markets/get-market-info) | API can enumerate market specifications; current market counts and custody compatibility still need evidence |
| [KyberSwap Robinhood announcement](https://blog.kyberswap.com/best-dex-aggregator-api-for-swapping-on-robinhood-chain/) | Official search result advertises Robinhood aggregator support; full-page retrieval failed during review, so exact deployment/coverage claims remain unverified |
| [Robinhood Chain connection docs](https://docs.robinhood.com/chain/connecting/) | Chain-specific integration reference; not evidence of any particular router's custody compatibility |
| [Reservoir](https://docs.hydromancer.xyz/reservoir) | Current overview lists fills, candles, daily account snapshots, and one-minute 20-level L2 snapshots; S3 delivery is requester-pays |
| [Reservoir snapshots](https://docs.hydromancer.xyz/reservoir/schema-reference/snapshots) | Account history includes known gaps; inspect per-dataset coverage |
| [Tardis terms](https://docs.tardis.dev/legal/terms-of-service) | Published terms distinguish internal use, third-party services, derived outputs, and AI uses; a regular subscription is not assumed sufficient |
| [Stripe subscription webhooks](https://docs.stripe.com/billing/subscriptions/webhooks) | Subscription state changes asynchronously; validate and reconcile webhook events |
| [Firecracker getting started](https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md) | KVM-capable host/runtime prerequisites; actual DO SKU support requires a test |
| [DO live migration](https://docs.digitalocean.com/products/droplets/details/live-migration/) | Nested workloads need operational consideration during migration; page is not a guarantee for every host configuration |

Third-party pages inform feasibility; this specification's state machines, service boundaries, policy, and test requirements are proposed product engineering choices. No claims of audited safety, guaranteed profitability, universally available bridge lanes, or verified fastest execution are made.
