# Production infrastructure and GitHub Actions

## 1. Production from the first public release

DigitalOcean remains the hosting platform. Import and adapt `infra-do`, deployment scripts, container builds, monitoring, and Actions from SuiOptions. Older AWS resource definitions and historical variable names do not imply an AWS deployment requirement. Infrastructure changes are managed as code and reviewed through this repository.

Before adopting the existing data-room droplet: record resource identity, ownership, region, disk/spool use, current services, collection continuity, backup status, and rollback procedure. Do not let a new Terraform workspace attempt to recreate or destroy an existing managed droplet. Choose one infrastructure-state owner and migrate/import state deliberately without storing it in Git.

## 2. Deployment topology

Separate logical pools: edge/API/control plane, chain indexers/keepers/messenger, data ingestion/normalization, isolated build workers, backtest microVM workers, and live strategy microVM workers. Initial host sharing is allowed for trusted services with resource bounds; untrusted guests and build jobs stay off custody/control-plane and collector hosts.

Use a DO VPC, restricted ingress, TLS, private databases/object storage access, service identities, and audited operator access. Public ingress exposes frontend/API/MCP and webhook routes; databases, guest control APIs, signers, and internal GraphQL are not publicly reachable.

Postgres provides durable control state, jobs/outbox/inbox, billing, registry instance catalog, and product/indexer projections with service-owned schemas. Durable execution and message journals must survive worker loss. Queue choice may begin with Postgres-backed leasing/outbox; do not introduce a new broker before its operational need is demonstrated.

Object storage holds raw/normalized/derived datasets, artifacts, reports, checkpoints, and backups in separate prefixes/buckets with distinct policies. Store uploaded strategy artifacts separately from public reports. Cloud worker caches and collector spools have high-water limits and alerts.

## 3. Configuration and deployment manager

Adapt deployment-manager to orchestrate compile, deploy, initialize, verify, registry write, and post-deployment smoke checks for EVM and Solana. Chain backend interfaces return typed deployment results, not parsed human log fragments. Deployment steps are resumable and record transaction IDs before proceeding.

Environment (`staging`, `prod`) is distinct from chain network (`mainnet`, local simulation). Both staging and prod may use separately scoped mainnet deployments; never share custody roles or addresses accidentally. Mainnet is the product target, with synthetic/fork tests before release.

Manager records registry changes atomically after confirmed results, validates code/interface identity, and handles partial deployment without overwriting successful unrelated records. A failed initialization leaves a disabled registry record and a recovery action, not an apparently active integration. Contract upgrade/ownership changes are separate audited operations; container rollback cannot roll back onchain state.

Manager supports proxy deployment with atomic initialization, implementation verification, role handoff, and governed upgrade/migration bundles as specified in [the contracts spec](02-vaults.md). Store proxy, implementation, admin/timelock, storage-layout/version metadata, and authority transitions in the canonical registry; product clients continue resolving the proxy/custody address. Verify both implementation code and proxy configuration after every upgrade. For Solana, record program version, account-schema version, and native upgrade authority. Rehearse a multisig-executed upgrade and confirm old deployer access is removed before declaring handoff complete.

No production address in per-service environment defaults. Distribute a versioned registry snapshot. Services report the registry/config/image versions they are running. Incompatible schema or registry revisions block activation.

## 4. CI workflows

| Workflow | Required checks/results |
| --- | --- |
| PR validation | Rust format/lint/tests; TS checks/build; Python SDK tests/types; documentation links; schema compatibility |
| EVM contracts | Pinned compiler/Foundry, unit/fuzz/invariant tests, gas/size tracking, approved fork simulations, proxy upgrade/storage validation, multisig handoff and populated-state migration tests |
| Solana programs | Pinned toolchain, program/instruction tests, adversarial account validation, build verification, multisig authority transfer and account-layout migration tests |
| Data room | Scoped crates, deterministic synthetic transforms, schema fixtures, dependency isolation |
| Contract/schema parity | Shared economic/wire vectors, generated client drift, registry consistency |
| Policy | No actual datasets, secrets scanning, address-authority checks, no Sui dependencies |
| Build | Affected-service matrix, immutable image/artifact hashes, dependency inventory |
| Remote integration | Authenticated dispatch to DO backend; synthetic/local-chain tests or DO-hosted real-data runs; only bounded summaries returned |
| Deploy staging/prod | Per-environment serialization, migrations, rollout, readiness/smoke, version recording |
| Operations | Explicit service restart/stop/redeploy/monitoring synchronization and recovery workflows |

Import selective-deploy dependency mapping and test the mapping itself. A shared crate change rebuilds all affected services. Compare against last successfully deployed revision, not just the last Git commit. Never advance the deployment marker after a partial failed rollout.

Use minimum GitHub permissions, pinned/reviewed actions, protected release environments, and no production credentials for untrusted fork PRs. Build artifacts once and promote by digest. Concurrent deployment-manager runs cannot race registry writes.

No historical market data or live capture is fetched onto GitHub runners. Contract fork tests must not become a hidden market-data download pipeline. Realistic dataset tests dispatch to DO and return pass/fail, counts, and approved analytics only.

## 5. Rollout, migration, and rollback

Deploy backward-compatible database expansion before consumers; destructive contraction happens only after compatibility checks and backup. Drain in-flight jobs or preserve leases/journals across deploys. Live strategy updates use the runtime's fenced transition, not an uncontrolled process restart.

Readiness checks verify required dependencies and reconciliation state. A gateway does not accept new trading merely because its HTTP port opened. Roll back application images/config only when compatible with schema and registry state; otherwise roll forward with a documented recovery path. Preserve existing positions, nonce domains, and unknown submission records.

## 6. Observability and incident controls

Carry source-project structured logging and alert conventions into all services. Trace user command -> job -> deployment -> order/message/transaction -> indexed confirmation. Logs contain IDs and bounded metadata, never secrets or unbounded raw market feeds.

Metrics: API latency/errors, indexer lag/reorgs, collector gaps/spool, normalization backlog, dataset coverage, backtest queue/resource usage, guest starts/crashes, live event lag, order intent-to-ack/fill, unknown submissions, reconciliation divergence, stale appraisals, payout age, bridge/message age, fee-pot balance, billing webhook lag, and capacity headroom.

Page on conditions that can affect funds or data continuity: unresolved execution, custody balance mismatch, stale valuation with active exposure, exhausted fee pot, prolonged payout blockage, transport mismatch, collector loss, and worker-control compromise. Each alert has an owner, runbook, and tested stop/recovery action.

Operational objectives for the hackathon: no acknowledged job or order intent lost after one worker restart; no duplicate economic action after retry; recorded recovery-time and restore-point measurements; timely stale-state detection before configured trading/valuation limits; and visible degraded service status. Freeze numeric SLOs from measured load before public live deployment rather than inventing an availability guarantee.

## 7. Backup, restore, and failure drills

Back up database/control journals and immutable configuration; protect object data against accidental deletion and validate lifecycle policies. Restore to a separate DO environment, replay indexers, and reconcile account state before resuming any trading. Restore success means usable application state, not merely a readable archive.

Required drills: database restart, worker death after submit, collector disk pressure, feed disconnect, indexer lag, transport stall, expired runtime authorization, registry outage/cache expiry, and deployment rollback. Document safe manual actions without relying on the original developer's laptop.

Day-one readiness is these verified behaviors plus release evidence, not a claim of external audit or an obligation to run a large multi-region cluster immediately. Region selection for latency is based on measured data and execution endpoints; Tokyo is a candidate, not a hardcoded promise.
