# Repository implementation rules

Read [the implementation specification](docs/hackathon/README.md) before implementing a component. User instructions take precedence. An unresolved decision blocks only the dependent capability, not unrelated work.

- Backend services and execution infrastructure are Rust. Strategy authoring SDK is Python. Frontend is React, TypeScript, and Vite. EVM contracts are Solidity; Solana programs are Rust. Do not introduce Sui runtime dependencies.
- Deploy infrastructure to DigitalOcean. Reuse the existing SuiOptions DigitalOcean framework through a documented import; do not migrate this product to AWS by copying legacy infrastructure.
- Historical market data, live collector output, and all strategy backtest execution stay in DigitalOcean. Never download, cache, or commit actual market datasets on developer machines or GitHub runners. Synthetic fixtures and code-only checks are permitted locally; real-data tests run through the remote backend.
- There is one authoritative address registry and one write path. Import deployment-manager, deployments, and token-info patterns as described in the spec. Never duplicate literal deployment, token, router, program, oracle, bridge, or system-contract addresses in application code, frontend bundles, SDKs, skills, docs, or environment defaults. Consumers resolve logical IDs. Contract configuration and generated artifacts derive from the registry.
- Frontends read contract state only through backend/indexed views. Wallet signing/submission is the transaction exception. Strategies use separate market-data streams and execution APIs.
- Preserve microservice boundaries, typed contracts, explicit ownership, and clear audit boundaries between vault accounting and venue/bridge integrations.
- Strategies are untrusted. Both backtests and live strategies execute in microVMs; container isolation alone does not meet the requirement. Never place custody authority inside a strategy guest.
- Every user capability must have a typed API and MCP surface. Wallet/financial actions require the user's scoped authorization; skills do not grant privileges or receive private keys.
- All imported work must be in conspicuously labeled provenance commits, with upstream commit, paths, license inventory, and separately recorded uncommitted changes. Do not mix imports and new behavior in one commit.
- Public-chain vault deployments target mainnet after the specified tests and release gates. Local execution/fork simulation and synthetic contract tests are required; a testnet-first product is not required.
- Adapt GitHub Actions, deployment-manager, token-info, monitoring, and operational tooling to this repository. Never leave references that mutate the source project's infrastructure or deployments accidentally.
- Do not claim that documentation, an imported implementation, a passing mock test, or a mainnet smoke transaction constitutes an audit.
