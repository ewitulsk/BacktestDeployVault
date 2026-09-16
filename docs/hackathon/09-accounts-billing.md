# Accounts, subscriptions, and capacity

## 1. Account model

Adapt the existing auth-service after reviewing its Sui-specific assumptions. Use a Rust service with a standard OIDC login integration as the preferred first account path; provider choice belongs in ADR-005. Store stable internal user/workspace IDs rather than using email or wallet address as the only identity.

Users can link multiple EVM/Solana wallets using chain-specific signed challenges bound to domain, chain, nonce, expiry, and session. Verify signature and nonce consumption server-side. Changing login identity or wallet association requires proof of control; a public wallet address is not authentication.

Roles: owner, curator/developer, viewer, billing manager, and operator as applicable. Begin with a personal workspace while preserving ownership fields for future teams. Tenant authorization is enforced per resource, including job/artifact/report paths and MCP tools. Secure HTTP-only sessions, CSRF/origin checks, session revocation, and login throttling are required.

Investors do not need platform accounts. Public views and wallet-scoped transaction preparation remain available without subscription onboarding. A wallet challenge may establish a short-lived investor session when needed; this is not mandatory enrollment in the strategy platform.

## 2. Pricing and grants

Display the planned research subscription at $20/month and deployment/execution subscription at $50/month. Proposed packaging: the $50 tier includes research; this is an explicit configurable product default to settle before paid launch, not a claim that the user selected additive pricing. Do not publish exact future compute/storage allowances until measured.

Hackathon grant: free access without card, configurable sensible research budget, one logical vault family, and one active strategy deployment. Multiple approved spokes belong to that logical vault; they are not independently charged slots. Backtest worker use is separate from the single live-strategy allowance. A stopped/replaced strategy can reuse the slot after its active lease is revoked and positions are reconciled.

One-vault grant counts created nonterminal families; failed creation attempts do not consume it. A completed closure can release eligibility under an explicit policy so users are not permanently trapped by a failed experiment. Keep grant history to prevent replay abuse.

Allowance values live in versioned operator configuration and are shown before a job starts. No unlimited compute promise. Freeze initial budgets after benchmark results; adjust future grants without retroactively altering completed reports.

## 3. Stripe implementation

Implement Checkout, customer portal, subscription state synchronization, webhook handling, and test-mode end-to-end checks during H0. Server resolves products/prices from billing configuration; clients cannot supply an arbitrary authoritative price. Stripe object IDs are not chain addresses, but should still be centrally configured.

Webhook ingress validates signatures against raw request bytes, durably stores unique event IDs, acknowledges quickly, and processes asynchronously. Handle duplicate/out-of-order events, delayed invoices, payment failure, cancellation, and subscription changes. Reconcile with Stripe's current object state rather than assuming event arrival order. A browser return from Checkout does not itself grant paid access.

Internal entitlement grants combine paid subscription, hackathon campaign, and limited operator grants. Persist source, valid interval, capabilities, quota policy version, and revocation reason. Hackathon users do not require a fake paid subscription. Checkout can remain feature-disabled publicly while integration tests and production configuration readiness are complete.

Enabling actual charges requires deliberate product activation and user checkout consent; ending a free grant must not silently enroll a user in billing. Public pricing says subscriptions are free during the hackathon and vault performance fees still apply.

## 4. Usage and global admission

Track backtest reserved/consumed CPU time, elapsed time, input volume, concurrent jobs, output volume, and failure classification. Initial user-visible allowance can be a simple credit budget derived from estimates. Refund or release unused reservations according to documented failure rules; retries do not reserve twice.

Global capacity is distinct from user entitlement. Scheduler admission considers available memory/vCPU, sandbox limits, live-worker reserve, and per-venue/account limits. Paid/free eligibility does not guarantee an immediate slot. Return `CapacityWait` with queue position/status and allow cancellation.

Live trading has priority over research bursts. Reserve room for keeper/recovery services and replacement workers. Benchmark maximum safe active guests before advertising deployment availability. Collect cost/usage internally without turning the hackathon pricing page into a metering specification.

## 5. Expiry and operational safety

Loss of paid/hackathon entitlement blocks new research/deployment admission and starts the published live-deployment grace policy. Never disable investor withdrawals, report access needed for their claims, cancellation, or protective unwind because of billing.

After grace, stop new risk, cancel resting orders, and follow the deployment's predeclared supervised exit/hold policy. Do not kill the only process capable of managing open positions. Keepers and gateway protective actions remain available until obligations are resolved. Data retention/export policy is disclosed separately from billing status.

## 6. Acceptance

Prove OIDC/session and wallet-link isolation, cross-tenant denial, free signup without Stripe, one-vault/one-live-strategy enforcement under concurrent requests, capacity wait, signed webhook verification, duplicate/out-of-order webhook reconciliation, upgrade/downgrade/cancel behavior, and safe live-deployment expiry without stranded investor funds.
