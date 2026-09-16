# Hub, spokes, messaging, and bridges

## 1. Authority and accounting

One HyperEVM hub owns the economic ledger. Robinhood and Solana spokes report authenticated quantities and execute hub decisions. They never independently mint economic supply, compute NAV, or run a tranche waterfall. A spoke's displayed share balance is a mirror of hub authority.

At first, capital may remain on the chain where deposited. A common global NAV is compatible with geographically separate liquidity, but a global claim does not guarantee immediate liquidity on any chosen chain. Display payout location, asset, queue, and estimated liquidity separately.

The logical vault has one mandate and capital structure, with chain-specific execution permissions. Funds must remain attributable by vault family throughout external execution and bridging.

## 2. Wire protocol

Adapt the imported Rust `vault-messages` schema; create versioned Solidity/Rust/Solana codecs and shared golden vectors. Remove Sui-specific BCS assumptions from the new protocol. Envelope fields:

`protocol_version, vault_family_id, source_chain, destination_chain, source_instance, destination_instance, lane_epoch, sequence, message_id, payload_type, payload_hash, observed_height, observed_time, registry_version`.

Chain IDs are registry-resolved typed values, not unchecked values supplied by the caller. Authentication binds chain, app, family, endpoint, and payload. Keep one active verifier per lane epoch. Local sequence checks supplement the chosen transport's authenticity/finality guarantees.

Persist messages and receipts; reject replay/foreign family/source. Out-of-order messages may be buffered but cannot advance economic state past a missing prerequisite. Ordered state-sync snapshots must include the causal watermark of deposits, payouts, trades, and transfers they cover so replay does not double-apply balance changes.

Core payloads: `DepositNotice`, `DepositDecision`, `DepositActivated`, `CancelDepositRequest`, `CancelDepositDecision`, `WithdrawRequest`, `WithdrawDecision`, `PayoutReceipt`, `StateSync`, `ConfigSync`, `TransferPrepared`, `TransferDispatched`, `TransferSettled`, `TransferFailed`.

## 3. Deposit state machine and cancellation

Spoke states: `Pending -> Accepted -> Active`, or `Pending -> CancelRequested -> Refunded`. The exact activation boundary must match the hub's recognized pending obligation and share issuance atomically.

1. Deposit locks measured received tokens in pending escrow and emits an authenticated notice. Pending funds cannot trade.
2. Hub validates eligibility, tranche, version, and complete appraisal; exactly once either rejects or issues the corresponding claim and commits acceptance.
3. Spoke applies the decision; accepted funds become active. Duplicate decisions have no additional effect. Hub accounts for accepted-but-unactivated assets consistently as a recognized escrow receivable, not both active balance and deposit amount.
4. A depositor can request cancellation after the configured timeout. The spoke does **not** unilaterally refund based solely on elapsed time.
5. Hub serializes acceptance versus cancellation. If not accepted, commit cancellation and forbid future issuance for that deposit. If already accepted, cancellation is rejected and acceptance is replayed; normal redemption is available.
6. Spoke refunds only after authenticated cancellation/rejection or a separately specified proof that acceptance is impossible.

**Required correction to the imported design:** a hub ACK generated before a deadline can still arrive after a spoke-local refund timeout. A timestamp margin alone cannot guarantee safety under unbounded network delay. The handshake above trades refund liveness during a partition for conservation. Do not market an unconditional timed refund. A more advanced proof-based timeout is out of initial scope unless formally specified and tested.

Test acceptance lost in transit, cancellation racing acceptance, late notice, replay, partition, and recovery. At no point may an investor retain both refunded principal and a valid issued claim.

## 4. Withdrawal state machine

Request escrows hub-recognized shares and preserves source lot/tranche/generation identity. Request processing respects the hub queue and current risk state. On fulfillment the hub uses a complete appraisal, burns the fulfilled shares, crystallizes fees, and fixes a payout-asset quantity. The hub books that quantity as a payable before sending the instruction.

Spoke pays only the authenticated beneficiary and fixed amount. Insufficient liquidity produces a FIFO payable with any available amount reserved. Track cumulative authorized, paid, and receipted quantities. Partial payments require unique receipt identities; the sum cannot exceed authorized quantity.

Hub subtracts the liability from NAV until matching payments reduce both custody assets and payable. State snapshots and receipts reconcile by watermark, not by independently subtracting the same payout twice. Fixed USDG liabilities are valued at the same approved USDG price as their backing assets; never assume every stablecoin equals one dollar.

New risk is blocked while a due payout needs local liquidity. Unwind, asset returns, and approved rebalancing to fund payouts remain allowed. Fees have separate payable records and are never silently taken from another investor's reserved withdrawal.

## 5. Remote truth and valuations

Spoke observations originate in onchain state or a separately specified verified venue-attestation path. The messenger cannot fabricate balances. RPC reads alone are not cryptographic evidence to a hub contract.

Each integration must supply an authenticated raw-state schema and hub valuation adapter. For external venues that cannot expose provable account state through the chain, explicitly document the attestation trust assumption, signer set, freshness, limits, and dispute/stop behavior before enabling pooled funds. Do not describe an operator signature as trustless.

Appraisal requires all bound spokes at acceptable freshness and coherent sequence watermarks. An unhealthy spoke can block global issuance and price-dependent settlement; dashboards show which dependency is stale. Previously fixed payables may continue settling safely without repricing. Do not loosen freshness during incidents to make the UI appear healthy.

### Capital mutation barrier

Freshness alone is insufficient: a recent remote snapshot can precede a trade, payout, or accepted deposit that changes the quantities used to price new shares. For the first multichain release, use explicit capital epochs. Fence new exposure-changing commands across the family, obtain the matching onchain/spoke barrier acknowledgments, reconcile pending orders/transfers and all relevant message watermarks, then build the hub appraisal and process a bounded batch of capital mutations. Release the barrier after settlement decisions are committed. Existing positions are still marked using current approved prices; a barrier does not freeze market prices.

Resting orders that can change exposure must be cancelled/reconciled or represented by a separately proved conservative settlement model before the epoch is eligible. An unresolved API submission cannot be assumed absent. Chain adapters enforce the epoch where possible; the delegated API profile explicitly relies on its documented gateway/agent authority boundary. A failed barrier expires without processing new capital, releases operational locks through a recovery transition, and leaves protective unwind available. Do not require investors to wait for all market risk to disappear; require the accounting state to be coherent.

This baseline may batch deposits and price withdrawals at intervals. UI reports that timing. Optimize toward continuous issuance only after an ADR proves equivalent accounting safety under concurrent cross-chain execution, rather than removing the barrier to improve a demo.

## 6. Transport selection and recovery

Retain transport-agnostic interfaces and assess the imported LayerZero/CCIP adapters. Live support on each **exact directional mainnet lane**, arbitrary message support, finality, code/interface versions, fees, and retry behavior must be evidenced before selecting a production transport. Do not assume old Sui/Robinhood assertions prove HyperEVM/Robinhood/Solana coverage.

Select one proven primary per lane. A secondary is useful only after its own tests; it is not a requirement to operate two verifiers concurrently. A relayer-only development endpoint is prohibited for production custody messaging.

Lane states: `Unbound -> Configured -> Verified -> Active -> Paused -> Draining -> Unbound`. Switch endpoint using a new epoch, a reconciled cutoff sequence, peer bindings, and a tested recovery procedure. The imported pattern that communicates a switch through the old endpoint may be unusable if that endpoint has failed. Freeze affected capital operations until the documented recovery procedure can reconcile outstanding messages; do not create a silent admin-forged message bypass.

Per-chain fee pots fund message fees; user transactions still pay applicable chain gas. Alert before fee-pot depletion. Failed sends must not strand a half-recorded local deposit. Backfill from finalized event history after service restart.

## 7. Bridge integration

Messaging and asset transfer are distinct capabilities. Prioritize one verified stable-asset vault-to-vault route; evaluate Relay, NEAR Intents, and native token rails according to actual source/destination/token support. USDG transformation may require swap plus bridge plus destination swap. Candidate names do not imply supported routes.

Transfer record: family, transfer ID, source/destination instances, assets and amounts, minimum net receipt, maximum fees/slippage, expiry, transport/route version, source transaction, destination beneficiary, refund destination, settlement observations, and accounting state.

State machine: `Proposed -> Reserved -> Dispatched -> InTransit -> Settled`, with `Failed/RefundPending/Refunded/ReconciliationRequired` branches. A source debit becomes one conservatively valued receivable. A destination credit replaces that receivable; it is not extra NAV. Never keep both the full source balance and receivable. A timeout is not proof of loss or refund. Partial fills/refunds must conserve the total claim.

Destination and refund recipients must resolve to approved custody accounts of the same vault family. No arbitrary recipient supplied by strategy or curator. Validate router return data, calldata, allowance, amounts, and destination chain. Bridge adapter disablement blocks new transfers but permits settlement/refunds.

Funding-arbitrage demonstrations pre-fund both venues, then rebalance independently. A bridge cannot make two trades atomic. Enforce temporary unhedged exposure budgets and a failed-second-leg unwind policy.

## 8. Multichain acceptance

Prove simultaneous deposits on two chains use one supply; no dilution from delayed snapshots; stale spoke behavior; replay/out-of-order recovery; acceptance/cancellation race safety; partial payout conservation; no double counting during bridge settlement; stablecoin depeg accounting; tranche impairment affecting all spokes; and safe drain/unbind. A mainnet spoke without a working recovery/unwind path cannot accept public deposits.
