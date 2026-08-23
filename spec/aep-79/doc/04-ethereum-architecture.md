# 04. Ethereum target architecture

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-04 |
| Version | 0.9 draft, 2026-08-10 |
| Primary variant | B1: portable contracts on an existing EVM L2 selected by Q-42 |
| Non-default variant | B2: Arbitrum Orbit L3 |
| Requirement family | `REQ-EVM-001–071` |

## Deployment and platform rules

The B1 suite must be chain-agnostic Cancun/Prague-level EVM. Host-specific addresses for Pyth, stablecoin, keepers, paymaster, RPC, and explorers live in configuration and one versioned address book; retargeting before G1 must not require protocol-source changes (`REQ-EVM-001–004`).

The host selected at Gate 0 must have a native liquid stablecoin, production Pyth, ERC-4337 and automation support, acceptable fees, Stage 1+ fault proofs/forced inclusion, a decentralization roadmap, at least two WebSocket RPC providers, indexer support, and stable commercial policy (`REQ-EVM-071`).

Use one pinned Solidity compiler ≥0.8.24, OpenZeppelin v5, and Foundry. Protocol contracts use UUPS proxies authorized only by `AkashTimelock`; storage uses append-only ERC-7201 namespaces and CI layout checks. Use custom errors, NatSpec, verified source, checks-effects-interactions, reentrancy guards, `SafeERC20`, and exact/permit allowances. Disable implementation initializers and deploy+initialize atomically (`REQ-EVM-005–010`).

Events are the historical system of record. Every semantic transition emits one reconstructable event before terminal storage is deleted.

## Contract suite

| Contract | Responsibility |
|---|---|
| `AKT` | 6-decimal ERC-20, permit, timestamp-clock `ERC20Votes`; claims/emissions mint only |
| `ACT` | 6-decimal restricted ERC-20; only BME/Escrow protocol movement |
| `DeploymentRegistry` | Tenant counters, deployments/groups, manifest hash, lifecycle |
| `Marketplace` | Orders, bids, leases, matching, reclamation |
| `EscrowVault` | Lazy streaming escrow, FIFO deposits/refunds, allowances |
| `BurnMintEscrow` | AKT↔ACT queue, vault, Pyth, CR breaker |
| `ProviderRegistry` / `AuditRegistry` | Provider identity/collateral and auditor attestations |
| `AkashConfig` | Parameters, addresses, and client-version registry |
| `MigrationClaims` / `EmissionsMinter` | Migration/residual claims and hard-capped emissions interfaces |
| `AkashGovernor` / `AkashTimelock` | Voting and immutable execution delay root |

`AkashTimelock` is not upgradeable. All other privileged roles point to it or to narrowly scoped pause/cancel roles defined in [08](./08-security-and-audits.md) (`REQ-EVM-011–014`).

## Tokens and oracle

**AKT:** 6 decimals for exact uAKT parity; `permit`; timestamp-based votes. Only `MigrationClaims` and `EmissionsMinter` may mint; no freeze/blacklist/admin transfer path (`REQ-EVM-015–017`).

**ACT:** restricted 6-decimal ERC-20. User transfers revert; only `EscrowVault` and `BurnMintEscrow` may burn/mint or perform protocol-authorized movements. Escrowed ACT remains in the BME liability denominator (`REQ-EVM-018–019`).

**Stablecoin:** governed native asset selected by Q-43. Pricing stays ACT-denominated; settlement uses normalized decimals and `stable_act_parity_bps=10000` initially (`REQ-EVM-020`).

Pyth adapter validates configured feed, age ≤30 seconds, confidence ≤150 bps, exponent/positive price, and update fee. Use EMA for CR and healthy spot for swap/fallback execution. An unhealthy read always yields oracle halt, never a default price. A secondary oracle slot is reserved but disabled at launch (`REQ-EVM-051–053`).

## Deployment and marketplace

- `nextDseq[owner]` issues monotonic identifiers; max 8 groups/deployment, 4 resource units/group, 24 attributes, and 4 `signedBy` auditors. `bseq=0` remains an API field only.
- Create requires ACT-denominated deposit/prices, creates escrow and orders, then marks active. Update replaces the manifest hash; close is terminal; pause/start preserves current relist semantics.
- Bid validation checks active provider, attributes/audits/resources, price ceiling, configured collateral, and max 20 bids (hard safety ceiling 500). Reclamation remains `1h–720h`.
- Tenant explicitly accepts a bid. Lease creation starts its payment and refunds losing collateral. Tenant close relists only an open group; provider close follows reclamation. All bounded cascades fit one transaction or use permissionless idempotent reap functions (`REQ-EVM-021–034`).

Provider state separates cold owner from rotating EIP-191/ERC-1271 operator/JWT keys and TLS SPKI hashes. Deregistration is two-step and finalizes only with no open bids/leases and after its delay. Audit records retain `(provider,auditor)` provenance (`REQ-EVM-048–050`).

## Escrow

Escrow storage uses WAD (`1e18`) fixed point and Unix seconds. Rates are micro-ACT/second; the old-chain conversion at S1 is exactly `×2/13`. Each account has logical ACT/AKT/stable balances, debt/state, and at most 16 FIFO depositors; payments store rate, accrual, withdrawal, and settled time (`REQ-EVM-035–038`).

Funded delegated allowances lock assets at grant, scope use, decrement atomically, restore the originating allowance on refund, and return remainder on revoke. Deposits use `permit` or exact approval, never unlimited standing approval (`REQ-EVM-039`).

Settlement algorithm:

1. Accrue with checked integer math and truncate outflows.
2. Drain ACT, then stablecoin at governed parity, then AKT only under BME CR-halt with healthy Pyth.
3. Freeze accrual timestamp and record explicit debt when overdrawn; deposit-while-overdrawn settles first.
4. Provider withdrawal receives 100%; close settles, closes payments, and FIFO-refunds depositors/allowances.

`settle`, withdrawal, refund, close, and reaps are permissionless or role-free when no user choice is involved, idempotent, non-reentrant, and conservation-checked. Pause never blocks exits (`REQ-EVM-040–047`).

## BME

`BurnMintEscrow` stores vault, pending totals, queue records, epoch/backoff, spreads, CR thresholds, `actEscrowed`, and caps. Request escrows/burns input; execution is timestamp-gated FIFO and uses one execution-time price per batch (`REQ-EVM-054–058`).

- Nominal epoch 65 seconds; governed backoff up to 93,600 seconds.
- Max 50 records, 3 attempts, minimum 10 ACT-equivalent, governed per-account and per-epoch caps.
- CR liability includes ACT supply, escrowed ACT, and refundable credits; pending AKT is not backing.
- Warning at 9500 bps; CR halt at 9000 bps; oracle halt on bad Pyth.
- AKT→ACT stops at CR halt. ACT→AKT stays open at CR halt and closes only at oracle halt.
- Recheck CR after each mint. Vault funding is one-way; no admin withdrawal. Compensation for automation is flat, not proportional to value.

## Governance and automation

`AkashGovernor` launch defaults: one-day voting delay, three-day period, 2,500 AKT proposal threshold, and 10% supply quorum subject to Q-23. Routine parameter changes wait ≥48 hours; upgrades wait ≥7 days; Safe can pause/cancel but cannot bypass the timelock. All parameter setters validate ranges and emit old/new values (`REQ-EVM-059–061`).

Chainlink Automation is primary, Gelato secondary, and permissionless entrypoints remain available. BME execution target is due+120 seconds and overdrawn detection target ≤300 seconds; redundant operators and alerts are mandatory (`REQ-EVM-062`).

EOA clients serialize nonces per key. Account-abstraction clients use independent nonce lanes and an ERC-4337 paymaster that may accept AKT/stablecoin. EIP-7702/session enhancements are optional; plain EOA operation always works. Preconfirmation is UI-only; marketplace actions wait for unsafe inclusion, accounting/root/exchange actions use safe/finalized tiers (`REQ-EVM-064–070`).

Typical protocol flows should cost ≤$0.10 and weekly provider operations ≤$0.05 under the selected host's normal conditions. Model current, 3×, and 10× fee regimes; a sustained >3× breach invokes Q-06/B2 review (`REQ-EVM-063`).

## B2: Orbit fallback

B2 deploys the identical suite as an Arbitrum Orbit L3 settling to Arbitrum One, with AnyTrust DA (committee ≥7), a 99.9% RaaS SLA, and AKT as gas via a WAKT wrapper. Canonical AKT and migration claims remain on the parent; bridging introduces a 7-day canonical exit plus optional fast exit. Budget roughly 0.5–1 FTE and $10k–$25k/month plus then-current Orbit revenue terms. B2 is non-default because it recreates sequencer, DA, upgrade, RPC, and bridge operations.

## Intentional deltas

Accepted deltas are UUPS/ERC-7201 storage; timestamp rates/epochs; direct Pyth; explicit caps; per-tenant dseq; funded allowances; stablecoin settlement; provider owner/operator keys and staged removal; events as history; permissionless reaps; restricted ACT accounting; keeper automation; EVM nonce/reorg tiers; and no staking, IBC, CosmWasm, x509, or dead take module.
