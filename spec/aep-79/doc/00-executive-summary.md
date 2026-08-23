# 00. Executive summary

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-00 |
| Version | 0.9 draft, 2026-08-10 |
| Owner | Overclock Labs |
| Audience | Governance, leadership, vendors, providers, exchanges, and community |

## Program in one page

AEP-79 proposes rebuilding the Akash decentralized-compute marketplace on a shared high-throughput chain and winding down `akashnet-2`. Gate 0 chooses one of two fully specified paths:

| Path | Target | Strengths | Principal uncertainty |
|---|---|---|---|
| A | Solana mainnet | DePIN distribution, low fees, high throughput, native Pyth, Token-2022 | Exchange/custody support and the Alpenglow transition |
| B | Existing general-purpose EVM L2, selected at Gate 0 | Mature Solidity/audit tooling, exchange and enterprise reach, standard governance/claims stack | Host-chain fees, policy, sequencer, and decentralization |

An Arbitrum Orbit L3 is specified as a non-default EVM fallback. Ethereum L1, a new sovereign rollup, SVM L2s, Cosmos shared security, and the status quo do not meet the combined operating-cost, distribution, and scalability goals. See [02](./02-target-selection.md).

## What is preserved

- The `deployment → group → order → bid → lease` lifecycle and tenant-selected winning bid.
- SDL and canonical manifest hashing; manifests remain off chain.
- ACT-denominated pricing, AKT/ACT burn-mint escrow (BME), and the `9500/9000` bps collateral-ratio warning/halt thresholds.
- Lazy streaming escrow, FIFO multi-depositor refunds, overdrawn behavior, delegated deposits with allowance restoration, AKT fallback under BME CR-halt, and 100% provider payout.
- Provider records, auditor-signed attributes, reclamation windows (`1h–720h`), and client-facing sequence semantics.

The implementation is a behavioral port, not a byte-for-byte state migration. Live leases remain on the old chain and end or redeploy during wind-down.

## What changes

- Consensus, validators, staking, slashing, sovereign inflation, and Akash-maintained Cosmos forks end.
- IBC and general-purpose CosmWasm are not carried forward; Pyth is read directly.
- The x509 certificate registry is replaced by wallet-signed JWTs and provider owner/operator/TLS keys anchored in the provider registry.
- Block-driven handlers become permissionless cranks on Solana or redundant keeper automation on EVM.
- Protocol time uses Unix seconds. Streaming rates use fixed point `×10^18`; the one-time per-block conversion is exactly `×2/13` for the old 6.5-second block target.
- Governance becomes Realms + Squads + timelock on Solana or OpenZeppelin Governor + Safe + timelock on EVM. Launch code is upgradeable with a defined path to DAO control and core immutability.
- Reduced, hard-capped emissions replace validator inflation and fund provider incentives plus the community treasury; the curve is decided at G1.

## Migration shape

`C` is cutover and the first snapshot (`S1`); `H` is the final old-chain block, normally `C+90d`; `S2` is the residual snapshot at `H`.

1. Before `C`, deploy the target protocol, open provider re-registration, rehearse the migration, coordinate exchanges, and campaign for IBC vouchers to return home.
2. At `C`, activate `v3.0.0-sunset`. New old-chain marketplace intake stops, but existing leases may settle, withdraw, close, refund, top up, and burn ACT.
3. `S1` credits liquid balances, bonded/unbonding stake, rewards accrued through `C`, and remaining vesting entitlements. Module-held funds become a target-chain Wind-down Reserve. Total target entitlement is fixed at `S1`.
4. Publish the S1 root within 24 hours after two independent pipelines agree. Exchanges may execute custodial swaps immediately; self-custody claims open after a seven-day public verification window, about `C+8–10d`.
5. During `C→H`, weekly residual roots pay old-chain provider earnings and refunds from the Reserve. Credits are incremental, FIFO-attributed, and capped at S1 principal. Post-S1 old-chain minting or module inflows create no new claim value.
6. At `H`, virtually settle remaining accounts, publish `S2`, distribute all residual entitlement, halt with `v3.1.0-halt`, publish archives, then decommission infrastructure by `H+90d`.

There is no persistent bridge. Claims use a two-year window; stranded IBC vouchers use a bounded foundation redemption process.

## Delivery and control

- Eight phases (`P0–P7`) and six decision gates (`G0–G5`) separate target selection, design freeze, code complete, launch readiness, S1 verification, and close-out.
- Verification is parity-first: golden vectors mechanically extracted from current keepers and at least 250 real mainnet lifecycles, exact integer comparison, property tests, full-stack scenarios, public testnet, load tests, and four rehearsals (`R1–R4`).
- Launch requires two independent audits of each on-chain tranche, fixes re-verified, a public audit competition for marketplace/economics, an infrastructure review, a live bounty, verifiable builds, monitoring, and rehearsed pause/recovery paths.
- Planning envelope: `142–200` person-months, peaking near `13–14 FTE`, plus Overclock commitment of at least two counterpart engineers. External audits, bounty, infrastructure, testnet rewards, legal, and archive hosting are separate client costs.

## Highest risks

| Risk | Primary control |
|---|---|
| Claims honeypot exploit | Immutable roots, one-claim receipts, constrained minting, two audits, formal/property checks, velocity breaker |
| Snapshot/supply mismatch | Two independent exporters, exact root match, conservation proof, public verification window |
| Escrow or BME parity defect | Golden-vector replay, integer-exact comparison, invariants, economic red-team |
| Governance rejection | Signal vote before G0, binding vote before irreversible action |
| Provider or validator attrition | Early re-registration, incentives, uptime commitments, backstop capacity |
| Exchange delay and holder confusion | 3–6 month outreach, venue playbooks, volume-coverage gate, status reporting |
| Target instability or fee shock | Gate 0 evidence, performance/chaos tests, delay authority, EVM portability |
| Scope/audit schedule growth | Frozen requirements, change control, staged audits, contingency reserve |

The point of no return is execution of S1: sunset activation at `C` plus the start of exchange custodial swaps. Abort paths exist only before that point.

Start with [13](./13-open-questions-and-assumptions.md) for what is fixed versus open, [10](./10-rollout-and-cutover.md) for execution, and [12](./12-risk-register.md) for operational risk.
