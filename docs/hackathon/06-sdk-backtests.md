# Python SDK and backtest engine

## 1. Artifact contract

A strategy artifact contains Python source, entrypoint, SDK version, manifest, locked dependencies, optional parameter schema, and a content hash. Build an immutable guest image with pinned base/runtime and dependency hashes. Network dependency installation happens in an isolated build worker with restricted access, never inside a live trading guest. Backtest and deployment reference the identical artifact hash.

Manifest fields: strategy ID/version; instrument selectors or dynamic universe rules; required event types; feed aliases; execution venue aliases; order capabilities; allowed parameters; warmup requirements; resource profile; checkpoint schema; and deterministic seed policy. Configuration supplies provider/venue bindings and vault permissions without editing source.

Do not design a generic arbitrary Python hosting service. Only supported dependencies and SDK-mediated data/execution interfaces are guaranteed. Reject unsupported packages/capabilities before reserving a live slot.

## 2. Stable SDK surface

Proposed minimal user contract:

```python
from bdv import Strategy

class Example(Strategy):
    def on_start(self, ctx):
        ctx.subscribe("reference_price")
        ctx.schedule_every("rebalance", seconds=60)

    def on_event(self, ctx, event):
        # Read normalized events, positions, and SDK state.
        pass

    def on_timer(self, ctx, timer):
        # Submit typed intents; never construct chain transactions here.
        pass

    def on_order_update(self, ctx, update):
        pass

    def on_stop(self, ctx, reason):
        pass
```

This is an interface proposal, not code claimed to run before the SDK exists. Freeze version 1 methods and event schemas before strategy templates are implemented.

`ctx` provides event clock, seeded RNG, read-only positions/balances/open orders, feed access, bounded persistent state, timers, logger/metrics, and execution methods: submit limit/marketable order, cancel, replace when supported, reduce/close, and target-position helper. Order methods return intent IDs, not an assumed immediate fill. Cross-venue strategies submit independently tracked legs.

The SDK cannot authorize transfers, wallet creation, arbitrary signatures, registry changes, or increases to the vault mandate. Deposit/bridge/rebalance capabilities, when permitted, are typed, custody-restricted operations with separate policy.

## 3. Event and ordering model

Events contain unique ID, schema version, venue/instrument, source timestamp, availability/receipt timestamp, source sequence, and provenance. Types include trade, quote, depth snapshot/delta, bar close, funding estimate/settlement, instrument listing/delisting, order update, fill, balance/position update, timer, health change, and checkpoint/recovery.

Backtest dispatch uses when information would have been available, not merely when it was economically effective. Bar close is delivered after the bar ends. Funding observations distinguish predictions from settled payments. If archive receipt time is unavailable, apply and disclose a configurable latency model. Stable tie-breaking makes repeated runs reproducible.

The strategy has one serialized logical event loop per deployment. Python batch/vectorized helpers may optimize calculation without exposing future data. Checkpoints preserve strategy state and processed sequence; external execution journals are authoritative for whether an order occurred.

Dynamic universes are point-in-time queries over listing/chain discovery events. No access to future token survival, later market metadata, future parameter search results, or subsequent data corrections within an old run.

## 4. Backtest job API and state

Request references artifact hash, parameter values, feed bindings, execution bindings, immutable dataset manifest, interval, warmup, starting balances/collateral, capital structure where applicable, fee/slippage/latency profiles, seed, and reporting options.

States: `Validating -> Reserved -> Queued -> Preparing -> Running -> Reporting -> Completed`, with `Rejected/Cancelled/Failed`. A retry receives a new attempt under the same logical job; it cannot charge/reserve twice. Save structured failures and resource usage. Timeouts terminate guest workloads and release reservations.

Rust owns replay, economic simulation, order lifecycle, and report computation. Python computes strategy decisions inside the microVM. Use bounded batches/vsock or an equivalent authenticated guest protocol to limit per-tick overhead; benchmark realistic event volumes early. Users never run platform strategies against downloaded local data.

## 5. Simulator fidelity

| Model | Required data | Claims allowed |
| --- | --- | --- |
| Bar/trade replay | Bars/trades and explicit spread/slippage model | Directional behavior under assumptions |
| Sampled-depth replay | Timestamped snapshots | Liquidity estimates near observed times, not exact intervening queue/fills |
| Sequenced depth replay | Initial snapshot plus complete deltas | Book evolution with explicit queue/latency assumptions |
| AMM/curve simulation | Historical reserves, fees, token/program state | Counterfactual route execution within observed state assumptions |

One-minute L2 snapshots cannot reconstruct continuous market making or sniping fills. Binance/Coinbase observations can drive a Hyperliquid signal but cannot substitute for Hyperliquid execution depth without an explicit proxy label. Historical Jupiter route availability is not automatically recoverable from token prices.

Model maker/taker fees, spread, price impact, quantity/tick rounding, gas/priority fees, funding intervals, margin requirements, leverage, liquidation/ADL where supported, order rejection, partial fills, cancellation races, delayed acknowledgments, and venue downtime. Capacity tests sweep notional size against liquidity assumptions. A counterfactual trade changes modeled inventory and potentially available liquidity; never give all strategies unlimited fills at the historical best price.

Backtest confidence is a description of data/model coverage, not a fabricated probability of profit. Flag unsupported behavior. If required semantics cannot be simulated, refuse a deployment-parity claim while permitting a clearly labeled research-only run.

## 6. Results

Every run outputs machine-readable summary, metric definitions, equity/returns series, drawdowns, trade/order ledger, positions/exposure, cost attribution, quality coverage, warnings, and provenance. Charts use these same versioned tables, allowing AI to inspect the values behind the graph.

Required metrics: total/net return, annualized return only with period caveats, volatility, Sharpe/Sortino with explicit frequency and risk-free assumptions, max drawdown, drawdown duration, downside tail statistics where sample size permits, win rate, profit factor, expectancy, turnover, gross/net exposure, leverage, liquidation distance, funding P&L, execution costs, fill/reject rates, and latency distributions. Undefined metrics are null with reasons, not zero or infinity disguised as success.

Required charts: equity versus an appropriate disclosed benchmark, underwater curve, return distribution/calendar heatmap, attribution by asset/venue/cost, exposure over time, trade timeline, and data-quality overlay. Add parameter sensitivity, capacity, latency/slippage sweeps, and out-of-sample/walk-forward views as H3 research features. Keep training/selection/test windows separate and record all compared runs to expose selection bias.

Support optional portfolio/tranche simulation with deposits/withdrawals and capital shocks, using the same reference economics as contracts. Report gross strategy and depositor-net performance separately, including platform/curator fees.

## 7. Paper and live parity

Paper mode uses current feeds and simulated orders with the same guest and intent schemas. Live mode replaces the execution backend; data and policy remain explicit configuration. Persist the bundle hash `(artifact, SDK, dependency image, parameters, feeds, mandate, adapter versions)` on every deployment and report.

Recovery begins with venue/account reconciliation and a checkpoint compatibility check; then invoke the strategy with a recovery event. Do not blindly replay submitted orders from Python state. Manual trades emit account/order events so the strategy sees their effects. Prevent simultaneous manual and automated contradictory writers through the gateway's account ordering and override policy.

## 8. Acceptance

An example runs in backtest, paper, and live modes with no source changes. Deterministic replay repeats results for the same manifest/seed. Tests demonstrate no future bar access, point-in-time asset universes, correct funding and fee accounting, unsupported-capability rejection, partial fills, cancellation races, checkpoint restart, and honest report labels. All actual replay acceptance runs execute remotely in DO; synthetic SDK unit tests can run in CI.
