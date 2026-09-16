# Data room, collection, and live feeds

## 1. Location and source reuse

Import the entire SuiOptions data-room source, preserving bronze/raw, normalized silver, derived gold, adapters, schemas, gap tracking, deterministic transforms, catalog, jobs, and isolation checks. All actual historical data, live captures, scratch space, replay caches, and strategy testing remain in DigitalOcean. The existing data-room droplet is reused only after ownership, capacity, current workloads, backups, and storage endpoints are inventoried.

Never read Terraform state or credential files merely to infer deployment shape; use deployment metadata and scoped operator tooling. The documentation PR does not authorize deleting existing workloads. Infrastructure adoption must preserve the old service until the migration/reuse plan has verified equivalent collection and recovery.

DigitalOcean object storage is the lake authority; local droplet disks provide bounded durable spooling and remote worker caches. If existing storage is outside DO, schedule a server-to-server migration into DO before marking the location requirement complete. Third-party upstream storage, such as Reservoir's S3 bucket, is a source, not the product's storage destination.

## 2. Historical ingestion

Hydromancer Reservoir is the first Hyperliquid source. Current documentation lists fills, one-second candles, daily account snapshots, and 20-level order-book snapshots at one-minute cadence; inspect actual schemas/partitions at ingestion. Daily availability and a marketing history cutoff do not establish complete coverage for every market or dataset.

Ingestion jobs execute in DO, use bounded concurrency, estimate download/request cost, validate checksums/schema/partition counts, and persist checkpoints. Requester-pays transfer cost belongs in job accounting. No workstation download step. Do not duplicate subset fills (liquidations, ADL, TWAP, builder) in an all-fills union.

Support Binance/Coinbase sources already present where terms permit. Integrate additional sources through the same adapters. Tardis and paid Pyth/Helius tiers are future capabilities; no assumption that a normal subscription licenses a hosted third-party backtesting product.

## 3. Dataset catalog and immutability

Catalog key: provider, venue, instrument ID, data kind, schema version, date/time partition, event-time interval, capture-time interval, normalization version, and source checksum. Track listing/delisting times, symbol aliases, contract specification changes, feed license, and embargo/export policy.

Every partition has status `Discovered/Fetching/Validated/Published/Quarantined/Superseded` plus row counts, gaps, duplicates, timestamp anomalies, and reason codes. An immutable manifest selects exact partition versions for a backtest. Corrections create new versions instead of silently changing old results.

Gold datasets include bars, returns/volatility, funding, spreads, selected liquidity features, and quality summaries. Their provenance refers to silver inputs and transform version. Generated reports refer to the same manifest. Keep raw event timestamps and received timestamps distinct; archival data lacking receipt time cannot claim measured historical arrival latency.

## 4. Collection breadth and tiers

Prefer all discoverable supported markets for instrument metadata and trades/funding; automatically discover new listings and retain delisted history. Full depth collection is configurable by actual provider limits and storage budget. Measure bytes/day, parse cost, connection count, rate-limit headroom, and replay demand before expanding.

| Tier | Contents | Intended use |
| --- | --- | --- |
| Catalog | Markets, token identities, listings, specification changes | Point-in-time universes |
| Broad | Trades, funding, bars, top-of-book where available | Directional and lower-frequency research |
| Deep | Sequenced depth snapshots/deltas for selected markets | Execution-sensitive tests |
| Chain launch | Mint/curve/pool events and state needed for routing | Pump.fun lifecycle research |

Collector requirements: heartbeat, reconnect with backoff, monotonic source sequence tracking where supplied, snapshot-plus-delta recovery, gap markers, spool durability, backpressure, explicit dropped-data alarms, and deterministic deduplication. A reconnect does not erase a capture gap.

Live trade feeds never pause because a nightly backfill saturates the host. Separate priority/resource pools; object ingestion lag is visible. Backfill and normalization jobs are restartable and idempotent.

## 5. Live strategy feeds

Start with Binance and Coinbase provider adapters, plus each execution venue's required market/account feeds. Hyperliquid venue state is needed even when the strategy's reference price comes from Coinbase. Add paid providers through the same capability contract later.

A `FeedConfig` maps logical subscriptions to providers, instruments, normalization rules, aggregation, freshness limits, and failure behavior. Aggregators support a documented median/weighted policy, maximum divergence, minimum healthy sources, stale-source exclusion, and circuit breaker. A mix of USDT/USD/USDC quotes must explicitly handle quote-currency conversion, not average unlike prices.

Version feed configuration and make the same transformation available to replay. If a historical input is absent, fail validation or require an explicitly labeled substitute run; do not silently test a different feed than deployment uses.

Strategy feeds are centralized reference data, not decentralized oracles. Vault issuance/withdrawal valuation uses independently governed accounting feeds. Strategies cannot change NAV by changing their feed weights.

## 6. Data licensing and output control

Keep a provider policy record covering commercial computation, third-party hosted use, derived analytics, visualization, retention, AI access, and redistribution. Tardis published terms require particular attention; obtain written coverage for this product rather than treating hidden raw downloads as sufficient. Free sources also need terms review.

User code may observe replay events inside its guest but receives no bucket credentials, raw partition download endpoint, arbitrary network egress, or unrestricted filesystem export. Host-generated report metrics and approved chart schemas are the normal export surface. Bound and inspect logs/custom metrics for market-data disclosure; recognize that arbitrary code can encode data into outputs, so technical filtering alone is not a complete licensing solution. The chosen license must tolerate the actual supported output model.

Users and AI can access their trade ledger, model configuration, result tables, and approved chart data. These are strategy results, not a raw provider-data API. Private strategy source and run results are tenant-private unless explicitly shared.

## 7. Acceptance

Prove resumable import; schema drift quarantines rather than corrupts; duplicate source files do not double-count; delisted assets remain queryable; replay manifests are stable; remote output excludes provider partitions; source gaps appear in reports; collection continues under backtest load; and deleting a transient worker does not lose lake data.
