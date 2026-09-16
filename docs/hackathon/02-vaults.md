# Vault contracts and economic specification

## 1. Design baseline

Port the SuiOptions v2 capital structure and custody model, using HyperEVM as the proposed canonical hub. Robinhood and Solana implement custody spokes. HyperCore spot/perps exposure is a hub-owned trading integration, not an independent share ledger. The initial single-chain path uses the same accounting without requiring remote messaging.

Contract boundaries: `VaultFactory`, `HubVault`, `CapitalAccounting`, `PositionLedger`, `WithdrawalQueue`, `IntegrationRegistry`, `ValuationRegistry`, `ProtocolTreasury`, EVM/Solana `SpokeVault`, and transport/venue/bridge adapters. Library/module decomposition may differ, but audit boundaries and invariants must remain explicit.

One curator can operate several instances of a vault family. Authority to trade is separate from authority to redeem an investor claim. Adapters can move funds only to approved custody destinations and return outputs to vault custody. Arbitrary external calls, arbitrary approvals, user-supplied recipients, and arbitrary delegatecall are prohibited. Standard proxy delegation to a governance-approved implementation is the explicit exception; callers cannot choose its delegatecall target.

## 2. Creation terms

Persist accounting currency; accepted deposit/payout assets; untranched or tranched mode; senior upside mode and parameters; junior target/maintenance buffers; hurdle rule; curator commitment; fees; lockups; withdrawal rules; mandate; supported integrations; upgrade policy; and `terms_version/spec_hash`.

Capital structure and economic terms cannot change retroactively for existing holders. Material changes require a separately specified migration or new vault. Registry upgrades alone cannot amend economic terms.

Spokes relay tranche selection without implementing a competing waterfall. Initial source-compatible claim accounting is lot-based. Transfers/splits must preserve basis, lock, tranche, and junior generation; a transferable claim interface is optional for the earliest MVP, but correct basis accounting is mandatory.

## 3. NAV and capital mutations

An appraisal covers every controlled asset, position, external account, recognized remote balance, transfer receivable, and payable exactly once. Sources carry observation height/time, sequence watermark, price provenance, confidence/freshness, and integration version. Stale, missing, divergent, or unpriceable exposure blocks issuance and valuation-dependent settlement; do not replace missing exposure with zero.

`NAV = valued controlled assets + recognized transfer receivables - recognized liabilities`

Pending unaccepted deposits are escrow outside shareholder NAV. Reserved withdrawal assets remain assets, paired with the withdrawal liability until paid. Unsolicited transfers are quarantined until explicitly recognized; donation handling must not enable first-depositor inflation. Never both count collateral/account equity and separately add positions already included in that equity.

Lock a complete appraisal and capital sequence for each deposit/fulfillment batch. Concurrent mutations invalidate it. Use checked wide intermediate arithmetic and explicitly tested rounding. Accounting prices are independent of user-composed strategy feeds.

Bound the number of active asset types, positions, and integrations per vault independently of the platform's discoverable market universe. Appraisal and maintenance cannot require an unbounded onchain loop. A staged appraisal carries its immutable snapshot/epoch and expiry, rejects intervening mutations, and commits only when all required legs are present. Benchmark worst-case permitted portfolios against gas/compute limits before publishing those limits.

## 4. Capital structure and shares

For untranched vaults, one share book receives all NAV. For tranched vaults, let `N` be NAV, `C` the accrued senior claim, `P` senior principal basis, `p` participation fraction, and `cap` the capped total return multiple:

```text
preferred = min(N, C)
residual = N - preferred
PreferredOnly: participation = 0
CappedParticipating: participation = min(floor(residual*p), max(0, floor(P*cap)-preferred))
UncappedParticipating: participation = floor(residual*p)
senior_nav = preferred + participation
junior_nav = N - senior_nav
```

All modes conserve NAV; junior absorbs losses before senior. Senior priority is not a capital guarantee. The maximum combined allocations cannot exceed realizable NAV.

Preserve the source's per-book virtual offset proposal: `O = 1_000_000`, shares minted `floor(value*(supply+O)/(tranche_nav+1))`; redemption gross value `floor(shares*(tranche_nav+1)/(supply+O))`. Accounting smallest-unit scale is pinned per vault. Differential tests must validate genesis, full redemption, dust, fees, and generation reset; no implementation may pay more than the available book due to virtual-share rounding.

Senior deposits require sufficient post-deposit junior buffer. A zero-NAV tranche with outstanding shares cannot accept ordinary deposits. Split/merge basis rules and locks follow the source lot model. Deposits cannot price from a stale appraisal or incomplete asynchronous execution state.

**Blocking economic decision:** the source hurdle text says non-compounding, while its recurrence/examples apply accrual to a growing claim. Before enabling tranches, resolve this in ADR-003 and publish an exact recurrence including elapsed-time segmentation, accrual during impairment, rounding, and rate boundaries. Do not silently invent a different financial product or let keeper call frequency change returns. Untranched implementation can proceed independently.

## 5. Fees and curator commitment

Preserve exit crystallization and per-lot basis accounting:

```text
profit = max(gross_claim_value - remaining_basis, 0)
gross_curator_fee = floor(profit * curator_fee_bps / 10_000)
platform_cut = floor(gross_curator_fee * platform_share_bps / 10_000)
curator_net = gross_curator_fee - platform_cut
investor_payout_value = gross_claim_value - gross_curator_fee
```

Platform fee is a share of the curator fee, not a second fee on all assets. Exact launch rates/caps are configuration to be frozen before public vault creation and displayed to investors; do not substitute old defaults without review. No fee on deposits or unrealized performance merely because a billing month ends.

Source behavior credits curator net as same-tranche shares at the locked ratio, with appropriate senior claim/principal adjustments, preserving remaining holders' PPS apart from specified dust. Reserve/remit the platform cash claim exactly once. At terminal settlement, curator compensation becomes an explicit cash liability instead of new shares. Test remote withdrawal crystallization and unpaid fee liabilities.

For a new senior fee share, update both senior claim and principal basis by the credited value. For senior exits, reduce claim/principal in the locked share ratio; do not leave withdrawn claim accrual assigned to remaining holders. Split lots allocate basis pro rata with specified dust ownership; transfers inherit basis and lock, rather than resetting a performance-fee liability.

Curator commitment has an enforced floor. Rotation, junior generation reset, and losses can trigger commitment breach. Fees do not provide a bypass around the floor. Port exact approved parameters and release conditions from the reconciled economic specification.

## 6. Lifecycle and action matrix

Lifecycle: `Open -> Closing -> Settled`. Risk states: `Healthy`, `CoverageBreach`, `Impaired`, `ResetPending`; commitment breach is orthogonal.

| Operation | Healthy/open | Risk-off | Closing | Settled |
| --- | --- | --- | --- | --- |
| Increase trading risk | Mandate-limited | No | No | No |
| Cancel/reduce/return funds | Yes | Yes, bounded unwind | Yes | Cleanup only |
| New senior deposits | Buffer-gated | No | No | No |
| New junior deposits | Yes | Coverage breach only; no ordinary impaired issuance | No | No |
| Request withdrawal | Yes | Yes | Yes | Redeem settlement claim |
| Fulfill junior withdrawal | Liquidity/queue-gated | Blocked for capital impairment states | Same capital gates | Frozen pool rules |
| Fulfill senior withdrawal | Liquidity/queue-gated | Yes with valid valuation | Yes | Frozen pool rules |
| Recognize inbound returns | Yes | Yes | Yes | Settled recovery policy |

Emergency pause must distinguish new risk from protective cancel/unwind/return operations. Disabling an adapter must not prevent returning its funds or settling claims. Force unwind after the documented grace period has tightly bounded permissions and no ability to redirect proceeds.

Junior reset preserves the source's objective impairment trigger, notice/seasoning, fresh execution appraisal, recapitalization minimum, and generation isolation. Old wiped shares never revive; a capital recovery before execution cancels an ineligible reset. Include the source seven-day minimum unless ADR-003 explicitly changes it. Reset and shutdown cannot erase senior liabilities.

## 7. Withdrawals and final settlement

Requests escrow shares and basis; they continue participating until the hub fulfills/prices the request. Preserve senior/junior queue ordering and risk-state gating. At fulfillment, burn shares, crystallize fees, and create any not-yet-paid fixed-asset liability. Remote messaging transports the settlement instruction, not a second opportunity to price the claim.

This deliberately separates `request` from `fulfillment/acknowledgment`; the old multichain handler's immediate pricing at request must be reconciled with the v2 queue rules before reuse. Clients show pending shares versus fixed payable amounts distinctly.

Close only after positions, unsettled orders, bridges, remote balances, fee liabilities, and pending messages are reconciled. Final tranche pools and supplies freeze once. Claims reduce pools exactly once and cannot exceed them. A transfer still in flight is not a closed vault.

## 8. Test requirements

Implement a Rust reference accounting model and shared golden vectors used by EVM tests and Solana serialization tests. Cover fee basis/splits/transfers, share inflation/donations, rounding at zero/tiny values, senior buffers, loss waterfall, partial withdrawals, queue fairness, insolvency, junior reset, rotation, settlement, and cross-chain liabilities. Fuzz stateful sequences and check conservation after every operation. Review implementation and specification together before mainnet exposure.

## 9. Transferable roles and multisig handoff

Every privileged role MUST be transferable or rotatable after deployment, including protocol/default admin, upgrade authority, curator, adapter/oracle/transport configuration, treasury administration, pauser, and operational permissions. Maintain an explicit role-to-admin hierarchy and role inventory for every contract/program. Never bind authority permanently to the deployer, an immutable EOA, or `tx.origin`; supported multisigs must be able to invoke the same privileged operations.

Root ownership/admin transfer uses nomination and acceptance by the successor, with the configured governance delay, cancellation, and events. Operational roles support controlled grant/revoke rotation; the handoff procedure verifies successor capability before removing predecessor access. Curator rotation preserves investor claims, fee basis, commitment obligations, and existing withdrawal rights. Governance role transfer does not transfer ownership of depositor assets.

Deployment-manager MUST support deployer-to-multisig handoff and subsequent multisig-to-multisig rotation, including any proxy admin, timelock, factory, and Solana program authority. After handoff it verifies the full role inventory and removes unintended deployer grants. No remaining deployer-only initialization, upgrade, emergency, or configuration path may bypass the successor. Record current authorities and pending transfers through the registry/indexer views, without introducing another address-definition source.

## 10. Upgradeable and extensible contract architecture

All stateful first-party EVM protocol components, including hubs, spokes, factories, registries, treasuries, and adapters that retain protocol state, MUST be upgradeable through a maintained standard proxy implementation. Stateless libraries and third-party venue/system contracts are outside this requirement; their binding/replacement remains governed. Per-instance initialization, stable custody addresses, preserved storage, and extension through versioned adapters are required from the initial deployment.

The baseline is OpenZeppelin `ERC1967Proxy` with a UUPS-compatible implementation and explicit upgrade authorization. An ERC-1967 proxy alone is insufficient without an upgrade mechanism. A documented Transparent/ProxyAdmin alternative is acceptable where justified, with its admin ownership included in the handoff inventory. Select one pattern per component and pin library/tool versions; do not mix UUPS and Transparent mechanisms or implement a custom proxy. Beacon upgrades require an explicit fleet-wide blast-radius decision; immutable minimal clones do not satisfy upgradeability for vault instances.

Deployment initializes the proxy atomically. Implementation contracts disable initializers; initializer/reinitializer versions, parent initialization, and authorization prevent takeover or replay. Constructors must not initialize per-proxy state. CI validates upgrade safety and storage compatibility against the actual prior release, including inherited storage, namespaced layouts, and any migration. Do not bypass validation with unexplained unsafe allowances.

Upgrade authority is separate from routine trading/pausing, transferable to a multisig or a timelock governed by that multisig. Publish the controlling authority, configured delay, implementation version/hash, and affected instances. Upgrades cannot be used as an ordinary path to rewrite existing economic terms. Nevertheless, upgrade controllers have powerful code-replacement authority: investor disclosures must state that trust assumption rather than claiming invariants are immune to malicious governance.

Upgrade plans preserve balances, shares, tranche generations, commitments, fee basis, queues/payables, message sequence/epoch state, and custody addresses. Coordinate gateway/indexer/adapter versions and capital barriers where necessary; in-flight messages must remain interpretable or drain before an incompatible wire change. Use reviewed migration calls for versioned state changes. Rollback is permitted only if storage and migration effects are demonstrably compatible; otherwise use a tested roll-forward recovery.

Solana uses the native upgradeable program mechanism rather than EVM proxies. Retain transferable upgrade authority and support a multisig-controlled authority/PDA using its supported governance execution path. Program-level admin/curator roles must rotate too. Preserve program identity and vault PDAs, version account layouts, and test account migrations with existing balances and claims. Never finalize/remove program upgrade authority as part of ordinary deployment or handoff.

Required release tests: actual multisig-authorized role handoff/rotation; predecessor denial; unauthorized upgrade rejection; initializer takeover/replay rejection; compatible upgrade with populated vault/tranche/queue/message state; incompatible storage rejection; continued upgrades after handoff; and post-upgrade deposit/trade/withdraw behavior. A new integration must be addable through the documented governed extension path without redeploying custody or discarding accounting history.

Implementation references: [OpenZeppelin proxy mechanisms](https://docs.openzeppelin.com/contracts/5.x/api/proxy), [upgrade-safe initialization and storage](https://docs.openzeppelin.com/upgrades-plugins/writing-upgradeable), [access control](https://docs.openzeppelin.com/contracts/5.x/access-control), and [Solana program authority](https://solana.com/docs/programs/deploying).
