# Frontend, APIs, indexers, MCP, and skills

## 1. Frontend surfaces

Use React, TypeScript, Vite, React Router, and typed API/query clients adapted from SuiOptions. The product contains no source-code editor. Users may inspect a read-only artifact summary, upload files, choose parameters, and use their external editor/AI.

| Surface | Required behavior |
| --- | --- |
| Landing page | Clear idea-to-backtest-to-funded-vault story, example report, chain roadmap/status, pricing path, free hackathon access, signup |
| Onboarding | Account creation, product explanation, templates/SDK guidance, wallet connection when needed |
| Strategy library | Upload, validation errors, immutable versions, manifest/capability summary, private by default |
| Backtest configuration | Data interval/coverage, feeds, execution venues, capital, parameters, assumptions, allowance estimate |
| Job view | Queue state, progress, cancellation, safe logs, retry, actionable failures |
| Report | Interactive charts/tables, metric definitions, quality warnings, run comparison, approved exports |
| Vault creation | Chain selection, capital mode, tranche preview, mandate, fees, lockups, curator commitment, transaction preparation |
| Deployment | Artifact/run selection, feed bindings, risk policy, capacity status, authorization, live state |
| Curator console | Orders, positions, funding, NAV, fees, feeds, health, manual controls, version changes, spokes, bridges |
| Public vault directory | Strategy/curator search, chains, capital mode, live history, capacity/status, fees; no login required |
| Public vault detail | Thesis, mandate, tranches, live performance, backtest separately labeled, holdings/exposure policy, versions, deposit/withdraw |
| Investor positions | Wallet-scoped claims, pending deposits, shares, fixed payables, fees/basis, queue, transaction status |
| Account/billing | Entitlement status, hackathon grant, Stripe portal/checkout when enabled |

Wallets sign user transactions and display network/amount/recipient. The frontend obtains contract addresses, balances, allowances, quotes, NAV, and transaction preparation from backend services. Audit wallet SDK defaults so incidental application reads do not bypass the indexer rule. Backend receipt polling/indexing drives transaction status after submission.

Indexed views carry `as_of_height/time`, finality, lag, and health. Stale views are visibly stale. Critical transaction preparation checks current constraints on the server and expires promptly; the contract remains final authority. A price preview is not a guaranteed mint/payout amount.

Public pages must not expose private strategy source, internal debug logs, personal account details, or raw licensed provider data. Shareable reports use explicit publication settings. Fees and tranche outcomes are understandable before signing.

## 2. API conventions

Rust HTTP API publishes OpenAPI and generates TypeScript/Python/MCP contracts. Version major breaking changes. Mutations use authenticated principals, idempotency keys, request IDs, and structured errors. Long jobs return `202` with resource ID; clients poll or use SSE for updates. Pagination uses opaque cursors and stable order. Monetary fields are unit-tagged decimal strings.

Error envelope: `code, message, request_id, retryable, field_errors, capability, remediation`. Codes include `data_coverage_insufficient`, `unsupported_capability`, `artifact_invalid`, `allowance_exhausted`, `capacity_wait`, `registry_unverified`, `indexer_stale`, `mandate_violation`, `authorization_required`, and `reconciliation_required`.

| API family | Representative operations | Authorization |
| --- | --- | --- |
| `/v1/catalog` | Datasets, instruments/universes, providers, venue capabilities | Public metadata or account policy |
| `/v1/registry` | Networks, tokens, components, vault instances, snapshot versions | Public reads; restricted mutations |
| `/v1/strategies` | Upload/create, validate, list/get versions, archive | Owning workspace |
| `/v1/backtests` | Estimate, create, list/get, cancel, retry | Owning workspace and allowance |
| `/v1/backtests/{id}/report` | Summary, metric tables, charts, trades, comparison | Owner or explicit share policy |
| `/v1/feed-configs` | Create/version/test configurations | Owning workspace |
| `/v1/vaults` | Public browse/detail, creation intent, publish thesis | Public read; curator mutation |
| `/v1/vaults/{id}/transactions` | Prepare deposit, withdrawal, claim, tranche/commitment actions | Wallet intent; no subscription required for investor actions |
| `/v1/deployments` | Validate/prepare/start, pause/resume/stop, version update | Curator + entitlement + scoped authorization |
| `/v1/orders` | Manual order/cancel, status, fills | Curator, gateway policy |
| `/v1/transfers` | Quote/prepare/authorize/reconcile vault rebalance | Curator and explicit mandate |
| `/v1/accounts` | Session, wallet challenge/link, preferences | Current principal |
| `/v1/billing` | Plans/status, checkout, portal | Account billing role |
| `/v1/investments` | Wallet positions, pending claims, history | Public-chain information with request controls |
| `/v1/operations` | Health, queues, reconciliation, capacity | Operator roles |

Uploaded artifacts use short-lived, tenant-scoped upload authorization; validate content hash before commit. Artifact/result downloads enforce the original resource ACL, not only possession of a guessed object path.

## 3. Indexer and GraphQL contracts

Each chain's Rust indexer exposes normalized raw-ish GraphQL consistent with the source project's pattern. Product-api consumes these schemas and maintains product-oriented views. Expose vaults, capital terms, share lots/generations, deposits, withdrawal requests/payables, appraisals, fees, adapter positions, messages, transfers, curator changes, and deployment events.

Indexing rules: durable finalized checkpoints; transaction/event identity deduplication; replayable transformations; rollback/reorg handling; transactional event+projection writes; bounded backfills; chain-specific finality; and reindex tools. Hyperliquid merges HyperCore execution with HyperEVM submission identities without claiming same-time settlement. Solana uses account snapshots plus events where event-only reconstruction is incomplete.

Market data is not crammed into GraphQL. Strategies receive high-throughput streams; reports use bounded paginated/columnar result access. Cross-service aggregation cannot present mismatched snapshots as a single authoritative onchain appraisal.

## 4. MCP capability parity

Expose domain tool groups through one discoverable product MCP endpoint initially; services may split later without changing business authorization. Tool schemas derive from the API contract. Suggested groups:

- `catalog.*`: discover data, markets, providers, venue capabilities, SDK/schema docs.
- `strategies.*`: create/upload/validate/version artifacts and templates.
- `backtests.*`: estimate/run/status/cancel, inspect reports, charts, and approved result tables.
- `vaults.*`: list/explain/create-intent, inspect terms, prepare investor transactions, publish thesis.
- `deployments.*`: validate/prepare/activate, monitor, pause/resume, update/rollback.
- `execution.*`: authorized manual trades/cancels, order/position history.
- `wallets.*`: explain/connect or initiate supported wallet-creation UX, prepare signing requests.
- `billing.*`: explain pricing/entitlements, open user-controlled checkout/portal.
- `frontends.*`: export a public vault presentation manifest, API schema, assets, and generated frontend starter instructions.

Long-running tools return job IDs and structured progress. Reports return chart specifications/rendered artifacts plus underlying approved result tables and definitions. AI must be able to inspect a chart's values, not only its image.

Read, research, live-trade, vault-admin, and billing scopes are distinct. A tool call cannot override entitlement, capacity, wallet authorization, or mandate. Signed user consent for a deployment enables subsequent autonomous orders within scope; it does not authorize unrestricted future deposits or mandate changes.

Treat uploaded strategies, token metadata, vault theses, and tool results as data, not instructions. AI wallet onboarding must use an approved wallet's user-controlled creation flow; seed phrases/private keys never enter chat or MCP responses. Wallet software/provider selection remains an integration decision, not an excuse to generate credentials in an LLM context.

## 5. Skills to ship

Product overview/onboarding; wallet setup for EVM and Solana; SDK strategy authoring; data/venue selection; backtest submission; report interpretation; deployment and recovery; vault investment/tranches; manual trading; strategy updates; and creating a custom vault frontend.

Example strategy skills: trend following, mean reversion, inventory-aware market making, basis/funding arbitrage, and dynamic memecoin universes. Each explains required data, execution assumptions, costs, risk, and failure cases. Examples are educational templates, not advertised profitable defaults. Strategies requiring unsupported data are labeled research-only until coverage exists.

Custom frontend generation uses a public presentation manifest and API client; it resolves addresses through the registry and reads contract state through backend views. It may not embed secrets or bypass mandate/deposit rules. Generated copy must distinguish curator claims from verified live results.

## 6. UX and parity acceptance

Run one user journey entirely through the frontend without coding in it, and the same journey through MCP plus wallet confirmations. Compare permissions, results, and errors. An unauthenticated investor can browse and complete a wallet-based deposit/withdrawal flow. Validate small screens, keyboard navigation, chart alternatives, empty/loading/error states, and clear pending-versus-final transaction labels.
