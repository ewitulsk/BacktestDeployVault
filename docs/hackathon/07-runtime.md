# Runtime isolation and execution gateway

## 1. Trust boundaries

Treat every strategy, dependency, uploaded archive, custom metric, and AI-produced script as untrusted. Both backtest and live workloads run in microVMs. Prefer Firecracker/KVM if the selected DigitalOcean worker supports and passes the required virtualization/security tests. Prove host capability before choosing a production SKU. If unavailable, acquire a compatible DO configuration or resolve the infrastructure constraint explicitly; do not silently replace microVMs with containers.

The existing data-room collector host is not the default home for untrusted strategy workers. Use a separate worker pool with least-privilege network access and quotas. Build jobs are isolated too, since package install scripts execute code.

## 2. Guest profile

Immutable root image, per-run writable scratch, CPU/memory/disk/process limits, bounded output, no host mounts, no cloud metadata access, no container socket, no data-lake credentials, and no general internet egress. Provide only the authenticated data/intent/checkpoint channels needed for the assigned job. Rate-limit event consumption and intent creation independently.

Artifacts are inspected for archive traversal/decompression abuse. Pin package hashes and base images; record software inventory. A dependency update creates a new artifact/image version. Guest teardown destroys transient state; retained checkpoints live encrypted in DO storage and are scoped to one tenant/deployment.

Benchmark startup, sustained events/second, resource usage, and recovery. Freeze initial resource profiles from measurements; public pricing does not need to expose detailed metering yet.

## 3. Deployment controller

States: `Draft -> Validating -> AwaitingAuthorization -> Provisioning -> Reconciling -> Running`, with `Pausing/Paused/Updating/Draining/Stopped/Failed/CapacityWait` branches. Activation requires valid artifact, compatible feeds/venues, active entitlement, capacity reservation, vault binding, mandate, custody authorization, and health checks.

Exactly one active controller lease/fencing generation can submit for a deployment/account. Gateway rejects commands from an expired generation even if a disconnected guest is still running. Persist desired state separately from observed state; retries are idempotent. Resource capacity reservation is transactional with deployment admission.

Stop policies distinguish stopping new entries, cancelling orders, controlled unwind, and terminating guest compute. A guest crash must not automatically trigger an unbounded market liquidation. The predeclared recovery policy chooses whether existing positions remain under keeper protection or unwind within limits.

## 4. Execution gateway

Rename the imported hedge-signer responsibility to execution-gateway. Keep signing as an isolated internal capability, optionally a separate process. The guest submits a typed `OrderIntent`, never unsigned arbitrary transaction bytes for blind signing.

Intent fields: deployment/lease generation, strategy version, vault/account, venue/instrument, side, quantity, limit/slippage, time-in-force, reduce-only, expiration, client ID, mandate version, registry version, and idempotency key.

Validation pipeline:

1. Authenticate workload identity and bind tenant/deployment/vault.
2. Check entitlement/capacity state without blocking protective exit operations.
3. Check active artifact, mandate, lease, and registry versions.
4. Read reconciled account state; reserve worst-case exposure for outstanding intents.
5. Apply asset/venue allowlists, trade/notional/leverage limits, price bands, stale-feed controls, and rate limits.
6. Construct and validate venue-specific transaction/action; constrain destinations and allowances.
7. Durably journal the prepared intent before signing/submission.
8. Submit; reconcile acceptance/fill/rejection using venue IDs and client IDs.
9. Release/adjust reservations only on confirmed outcomes, retaining uncertainty when necessary.

An ambiguous network timeout is not a failed order. Query by client ID or reconcile account state before retrying. Journal states include `Received/Rejected/Reserved/Prepared/Submitted/Unknown/Accepted/PartiallyFilled/Filled/Cancelled/Expired`. Record every transition and adapter version.

All manual trades use the same pipeline. Manual override may pause strategy submissions, then reconcile and notify the guest before resumption. Emergency cancellation has a narrow protective privilege; no generic signing escape hatch.

## 5. Signing and custody policy

User authorizes vault creation/deposit and delegated execution through their wallet once per defined scope. Strategies never obtain seed phrases, root wallet keys, withdrawal-capable API credentials, or arbitrary signing endpoints. Service-held credentials are per deployment/venue where feasible, revocable, rotated, audited, and never emitted in logs, reports, MCP output, artifacts, or browser storage.

Choose the concrete custody/signing provider in ADR-004 after examining the imported service. Separate deployment/admin authorities from runtime trading permissions and collector credentials. Approval to trade is not approval to transfer capital outside approved custody destinations. Onchain constraints enforce what they can; venue API permissions and gateway checks carry documented residual trust.

Any operational work involving existing AWS-managed secrets must follow the workspace's required secrets skill/runtime-resolution policy: do not retrieve secret values into agent context. This document does not select AWS hosting or authorize exporting existing credentials into the new repository. If the required skill/provider is unavailable, record the blocked credential operation and continue unrelated work.

## 6. Strategy changes and mandate enforcement

Strategy source versions are immutable and identified by content hash. Deployment updates pin a new tested artifact and preserve a visible audit history. Same-mandate updates require curator authorization, compatibility validation, position reconciliation, and checkpoint migration or explicit reset. Rollback is also a recorded version transition.

Material mandate changes (assets, venues, leverage/exposure ceilings, fee structure, custody profile, or investment thesis permissions) require a disclosed notice/exit process or a new vault. During notice, do not activate broader permissions for existing capital. Exact notice terms are fixed at creation. An arbitrary script's intent cannot be reliably inferred from its hash; enforce measurable permissions at execution boundaries.

The public page shows version history and whether manual trading is permitted/active. Source code may remain private while artifact hash, mandate, results, and performance lineage are public. Do not publish proprietary code merely because the platform implementation is open source.

Use two-step ownership transfer and separately scoped protocol governance for adapter registration, upgrades, treasury configuration, and emergency controls. The launch configuration records who can upgrade custody/accounting code and the applicable delay; mainnet administration should use a multisig with a tested recovery process. Emergency revocation can remove risk-increasing permissions promptly, but cannot seize investor claims, redirect payouts, waive tranche losses, or introduce a new unrestricted adapter. Registry metadata cannot override onchain governance checks.

Transferable roles, standard proxy/native program upgrades, multisig handoff, and populated-state upgrade tests are mandatory under [the contracts spec](02-vaults.md). The restrictions above govern ordinary operational permissions; code-upgrade authority carries the explicitly disclosed governance trust assumption described there.

## 7. Failure and security acceptance

Test guest escape prerequisites, metadata/network denial, output flooding, resource exhaustion, malicious archives, dependency build attacks, cross-tenant access, expired leases, duplicate submits, unknown orders, signer outage, venue outage, stale feed, clock drift, worker crash, and controller restart. Demonstrate revocation actually prevents new venue actions within the documented latency and that withdrawal/unwind paths do not depend on a healthy guest.

Adversarial curator tests include trading against self-controlled liquidity, manipulated small-size quotes, excessive slippage, hostile recipient substitution, and switching strategy to evade limits. Define what is prevented versus bounded/disclosed; non-withdrawable custody alone does not guarantee economically honest trading.
