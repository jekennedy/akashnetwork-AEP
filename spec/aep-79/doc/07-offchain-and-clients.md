# 07. Off-chain services and clients

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-07 |
| Version | 0.9 draft, 2026-08-10 |
| Requirement family | `REQ-OFF-001–062` |

## Blast radius

The target changes chain access, signing, addresses, events, indexing, fees, wallets, explorers, and deployment configuration. It does not change Kubernetes workload orchestration, SDL, canonical manifest serialization/hash, or the provider HTTP/gRPC business APIs. A component inventory with owner, repo, target changes, test coverage, release, and rollback status is a G1 deliverable (`REQ-OFF-001–004`).

## Provider daemon adapter

Introduce a narrow `pkg/chain` interface covering authoritative entity reads, subscriptions, transaction simulation/signing/submission, fee estimates, version/config lookup, and health. Kubernetes/manifest/bid logic must not import chain-specific packages (`REQ-OFF-005–009`).

- **Solana:** Go performs reads and simple flows. Q-15 decides whether transaction assembly needs a localhost-only Rust/TypeScript signer sidecar. The sidecar never receives the cold owner key, exposes a minimal authenticated gRPC API, builds only allow-listed instructions, and returns simulation/landing results.
- **EVM:** use generated ABI bindings/viem-compatible transaction data, per-key nonce queues, replacement rules, and safe/finalized read tiers.

Events are at-least-once. Persist `(chain,tx,event-index)` before side effects, deduplicate, reconnect from a cursor, reconcile periodically against direct chain state, and handle Solana fork/commitment or EVM reorg replay. All write workflows use a durable intent journal and idempotency key; retries must determine whether the prior transaction landed. Order-seen→bid-landed target is ≤5 seconds p95 under load (`REQ-OFF-010–017`).

Use a cold provider owner for registration, collateral, and operator rotation; hot operator keys for bidding/lease operations. Bound hot-wallet fee/collateral float, rotate without downtime, alert on unexpected registry changes, and support hardware/offline owner signing (`REQ-OFF-018–021`).

## Authentication and identity

The x509 registry is retired. The on-chain provider record is the trust root for (`REQ-OFF-022–030`):

- cold owner authority;
- up to three rotating JWT/operator signing keys;
- TLS SPKI hashes and endpoint;
- key status/history.

Provider gateways use Web PKI plus optional/required registry-pinned SPKI. Tenants prove lease authority with wallet-signed JWTs using the existing access levels/scopes and short expiry. Solana uses Ed25519/Sign-In-With-Solana style challenges; EVM uses EIP-191/SIWE and ERC-1271 for smart accounts. Support both offline-signed tokens and interactive nonce challenges. Verify issuer, audience, chain, provider, lease tuple, scopes, issued/expiry times, signature, current registered key, and authoritative live lease state. Cache registry keys no longer than five minutes and reject a rotated/revoked key.

Manifest submission remains unchanged: canonicalize and SHA-256 hash exactly as today; the on-chain deployment stores the hash only. Cross-language mainnet manifest vectors must pass in CI (`REQ-OFF-031–033`).

## Indexing and API continuity

Run a dedicated canonical indexer: Geyser/webhooks→Postgres on Solana or standard logs→a Ponder/Envio/Subsquid-class pipeline on EVM. It must ingest 100% of protocol events from genesis, deduplicate, backfill, survive restarts/reorgs/upgrades, expose health/head lag, and reconcile sampled state against direct RPC (`REQ-OFF-034–041`).

Targets under sustained load:

- event→API visibility p95 ≤2 seconds on Solana and ≤4 seconds on EVM (program-wide acceptance target may tighten to 2 seconds in [09](./09-testing-and-verification.md));
- no silent gaps; raw events retained;
- at least two independent RPC/ingestion providers and a self-host recovery path.

Freeze `/v1` for old-chain/archive shapes (bech32 addresses and Cosmos transaction hashes). `/v2` is target-native and includes `chain_id`; preserve existing Console resource shapes except unavoidable address/hash/finality fields. During dual-chain operation, namespace all entities by chain to prevent numeric dseq or Kubernetes-name collisions (`REQ-OFF-042–045`).

## SDKs, CLI, and configuration

Ship (`REQ-OFF-046–052`):

- TypeScript SDK generated from on-chain IDL/ABI, using `@solana/kit`/Codama or viem primitives;
- Go SDK exposing the adapter types needed by providers and migration tooling;
- `akash` CLI v3 with familiar create/update/close, bid/lease, provider/audit, escrow, BME, governance, claim, and migration commands;
- offline/hardware/multisig claim signing, transaction simulation, fee display, commitment/finality selection, and machine-readable output.

IDL/ABI changes are versioned and CI-diffed. `akash-config`/`AkashConfig` is authoritative for program/contract addresses, token/oracle endpoints, semantic versions, and minimum client versions. Clients refuse unsafe incompatible versions but provide actionable upgrade messages.

## Console, wallets, fees, and public infrastructure

Launch wallet support is verified, not assumed. Cover major browser/mobile wallets, WalletConnect for EVM, Ledger, Squads/Safe, and programmatic custodial flows. Console managed-wallet migration is Q-14; it must not silently map old keys to a new address (`REQ-OFF-053–055`).

Fee sponsorship is optional. Solana uses a co-signing fee payer; EVM uses ERC-4337 paymaster/bundler. Enforce allow-listed methods, per-user/IP quotas, maximum fee, daily/global spend caps, replay protection, circuit breaker, monitoring, and user-paid fallback. Display both native gas and any AKT/stablecoin charge. Operations ownership is Q-07 (`REQ-OFF-056–057`).

Provide canonical explorer links, verified source/IDL/ABI, public RPC/API documentation, rate limits, and a status page. Contract at least two commercial RPC providers with WebSocket support; monitor them independently and publish degraded-mode behavior (`REQ-OFF-058–059`).

## Claim and migration UX

The portal/CLI must support S1, weekly residual, S2, vesting, exchange, IBC-redemption, Ledger, and Cosmos multisig cases; show leaf/root/recipient before signing; serve proofs from reproducible public artifacts; link only to the canonical domain; and expose CLI/direct-program operation if the web UI is unavailable or geo-restricted. Provider tooling performs re-registration, keys/TLS, collateral, audit re-attestation, and a test bid (`REQ-OFF-060–061`).

## Cutover

Release target adapters/indexer/SDKs in testnet first, then provider daemon GA and re-registration by S1−30d. Run old and new adapters in separate processes/configurations during the 90-day window; never submit the same intent to both chains. Console blocks new old-chain deployment UI at C but keeps old leases visible/read-only with close/top-up guidance. At H switch old-chain data to archive-only, retain `/v1`, and retire write credentials. All steps require rollback/config-switch instructions and rehearsals (`REQ-OFF-062`).
