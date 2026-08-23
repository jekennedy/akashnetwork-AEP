# 03. Solana target architecture

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-03 |
| Version | 0.9 draft, 2026-08-10 |
| Scope | Path A on Solana mainnet |
| Requirement family | `REQ-SOL-001–073` |

This document is the compact normative design for the Solana path. [01](./01-current-architecture.md) defines behavior, [05](./05-token-migration.md) defines claims/supply, and [14](./14-appendix-protocol-mapping.md) maps entrypoints.

## Foundation

- Use Anchor v1.x. Pin the toolchain; Pinocchio is allowed only for CU-critical code that remains byte-compatible with the Anchor accounts, discriminators, instruction encoding, and IDL (`REQ-SOL-001–002`).
- Deploy with the current supported loader, use deterministic/verifiable builds, publish on-chain IDLs and typed clients, and gate compute regressions in CI (`REQ-SOL-003–006`).
- Use `Clock.unix_timestamp`, never slot/epoch cadence, for economics (`REQ-SOL-007`).
- Store working state per entity, close terminal accounts and refund rent, and treat append-only events as historical truth.
- All amounts are `u64` micro-units with 6 decimals. Rates and fractional math are `u128 ×10^18`; timestamps are Unix seconds; Borsh uses fixed-width little-endian integers (`REQ-SOL-011–012`).

## Program suite and call boundaries

| Program | Responsibility |
|---|---|
| `akash-deployment` | Tenant counters, deployments, groups, manifest hashes, lifecycle |
| `akash-market` | Orders, bids, leases, matching, reclamation |
| `akash-escrow` | Accounts, payments, settlement, deposit allowances |
| `akash-bme` | AKT↔ACT queue, vault, oracle gating, CR breaker, ACT gateway |
| `akash-provider-registry` | Provider owner/operator/TLS keys, attributes, collateral |
| `akash-audit` | Auditor attestations |
| `akash-config` | Governed parameters, program/address/version registry |
| `akash-claims` | S1, residual, S2, vesting, and Reserve interfaces |
| `akash-emissions` | Hard-capped post-migration emissions |

Permitted protocol CPIs are deployment→market/escrow, market→escrow/deployment hook, escrow→BME ACT gateway/token programs, and BME/claims/emissions→token programs. Program IDs come from config and are validated. No upward escrow→market hook is allowed: escrow emits terminal state; permissionless guarded reaps complete market/deployment cascades within 60 seconds p95. Deepest call chain must fit Solana invoke depth (`REQ-SOL-008–010`, `016–017`).

Every state change emits one event via event CPI. Events contain the full entity identity and all economic amounts; schemas are append-only (`REQ-SOL-014–015`). Every account records its rent payer, closes child-first, and cannot close while live children remain (`REQ-SOL-018`).

## Canonical addresses and bounds

PDA seed schemas are frozen after launch (`REQ-SOL-013`). Core seeds:

| Account | Seeds |
|---|---|
| Tenant/deployment/group | `tenant,owner`; `deployment,owner,dseq`; `group,owner,dseq,gseq` |
| Order/bid/lease | `order,owner,dseq,gseq,oseq`; `bid,...,provider,bseq`; `lease,...,provider` |
| Escrow/payment | `escrow,entity`; `vault,escrow,mint`; `payment,lease` |
| Allowance | `allowance,granter,grantee`; `allowance_vault,allowance,mint` |
| BME | `bme_state`; `bme_vault`; `act_auth`; `swap,direction,seq`; `tip_treasury` |
| Provider/audit | `provider,owner`; `audit,provider,auditor` |
| Config | `params,module_tag`; `registry` |

All numeric seed components use fixed-width little-endian encoding. Enforce launch bounds: 8 groups/deployment, 4 resource units/group, 24 attributes, 4 `signed_by` auditors, 16 escrow depositors, 3 provider signing keys, and 3 allowance mints. Validate these against the mainnet SDL corpus before G1 (`REQ-SOL-019`, Q-38).

## Tokens and oracle

**AKT.** Prefer Token-2022, 6 decimals, metadata only; no transfer hook, delegate, fee, confidential extension, or freeze authority. Claims initially control minting, then transfer it to the emissions PDA. If Q-05 finds insufficient exchange/custody support, use one canonical legacy SPL mint with Metaplex metadata; no other design changes (`REQ-SOL-022–024`).

**ACT.** Token-2022, 6 decimals, `NonTransferable`, no freeze authority. Because it cannot move into vaults, escrow deposit burns ACT and increments `act_escrowed`; payout/refund remints and decrements it. Only the BME gateway or BME swap execution may mint/burn. CR liability is `ACT mint supply + act_escrowed` (`REQ-SOL-025–027`).

**Settlement stablecoin.** A governed native mint selected by Q-43, normalized for decimals. Lease prices remain ACT-denominated; stablecoin settles at `stable_act_parity_bps`, initially 10,000. Config controls activation and limits (`REQ-SOL-028–029`).

**Pyth.** Validate receiver owner, full verification, configured AKT/USD feed ID, `publish_time ≥ now−30s`, and confidence ≤150 bps. Fail closed. Use EMA for CR and healthy spot for swap execution and AKT fallback (`REQ-SOL-020–021`).

## Config, provider, and audit

`akash-config` owns typed, versioned parameter PDAs and the authoritative program/address/min-client-version registry. Only the governance executor may mutate them; values are range-checked and changes emit complete old/new values (`REQ-SOL-030–032`).

A provider record contains cold owner authority, up to three rotating Ed25519 operator/JWT keys, TLS SPKI hashes, host URI, contact, attributes, collateral, active/open counters, status, and timestamps. Create/update/rotate/deactivate are owner-signed. Final close requires zero live bids/leases and expiry of the configured withdrawal delay. Rotation history is emitted; no deletion of a live provider (`REQ-SOL-033–036`).

Audit state is one `(provider,auditor)` PDA containing signed attributes. Only the auditor can upsert/delete; matching verifies provenance and configured auditors. Close empty attestations and refund their rent (`REQ-SOL-037–038`).

## Deployment and market

- A per-tenant counter assigns monotonic `dseq`; `gseq` and `oseq` retain current semantics; `bseq=0` is wire compatibility only.
- Create deployment is pending until its escrow and group orders exist, then activates atomically or through an idempotent finalize step. Deposit and group pricing are ACT-denominated. Update only changes a different canonical manifest hash.
- Close, pause, and start preserve current state gates. Closing is terminal; start creates the next `oseq` order (`REQ-SOL-039–045`).
- Orders reference bounded specs rather than a global book. Bids require provider/audit/resource match, price ≤ order ceiling, configured collateral, and max 20 active bids. Provider-initiated close respects `1h–720h` reclamation.
- Tenant creates the lease from a chosen bid. Order becomes matched immediately; payment starts at the bid's per-second rate. Losing bids are asynchronously, permissionlessly closed/refunded. Tenant lease close relists only if the group stays open (`REQ-SOL-046–052`).

Bid-slot anti-spam economics and provider collateral/slashing remain Q-08; they must be resolved without changing tenant-selects-bid ordering.

## Escrow

Escrow accounts hold logical balances in ACT, AKT, and the selected stablecoin, plus at most 16 FIFO depositor entries. Payments store rate, accrued amount, settled time, state, and debt. All math is checked fixed point and outflows truncate (`REQ-SOL-053–056`).

Delegated allowances are funded at grant into allowance vaults, scoped to deployment/bid deposits, decremented atomically, restored only to the original allowance on refund, and return unspent funds on revoke. No generic authz successor is required (`REQ-SOL-057`).

Settlement is lazy and permissionless:

1. Accrue `rate_per_second × elapsed`, using `×10^18` fixed point.
2. Drain ACT, then stablecoin at governed parity, then AKT only when BME is `halt_cr` and Pyth is healthy.
3. If insufficient, record debt/overdrawn and freeze the settlement timestamp consistently with current behavior.
4. Withdraw pays 100% to the provider. Close settles, closes payments, FIFO-refunds depositors/allowances, then exposes guarded reaps.

Settle, withdraw, close, refund, and reaps are idempotent and permissionless where no owner choice is involved. Conservation and overdrawn postconditions must abort atomically (`REQ-SOL-058–061`).

## BME

BME holds the AKT vault, ACT authority, queue sequences, `act_escrowed`, status, epoch time, caps, and pending totals. Swap requests escrow/burn input immediately. Execution is permissionless, FIFO, time-gated, and uses one healthy execution-time price per batch (`REQ-SOL-062–066`).

- Nominal epoch: 65 seconds; governed exponential backoff up to 93,600 seconds.
- Max 50 records/epoch, max 3 attempts, minimum 10 ACT-equivalent, plus governed per-account and per-epoch volume caps.
- CR uses vault AKT net of pending AKT and liability `ACT supply + act_escrowed + refundable ACT credits`.
- Status: healthy, warning at 9500 bps, `halt_cr` at 9000 bps, `halt_oracle` on unhealthy Pyth.
- AKT→ACT is blocked at CR halt. ACT→AKT remains allowed at CR halt but is blocked at oracle halt. Recheck CR after each mint and abort before crossing the halt threshold.
- Vault funding is one-way governance action; no administrative withdrawal. Crank tips are flat, not value-based.

State/status updates and due epochs must complete within 60 seconds p95 under load (`REQ-SOL-067–068`).

## Governance, clients, and runtime

Realms governs parameters and proposals; Squads holds launch upgrade authority behind a public timelock. Routine changes wait at least 48 hours; upgrades follow the stricter [08](./08-security-and-audits.md) policy. Claims/emissions expose only the interfaces in [05](./05-token-migration.md). Authority moves from launch multisig to DAO control, then claims/escrow core may become immutable (`REQ-SOL-069–070`).

Clients use versioned transactions, canonical frozen ALTs, simulation-derived CU limits, bounded priority-fee escalation, configurable commitment, and idempotency keys. A relayer may sponsor fees but is never required; retries must distinguish expired blockhash from landed transactions (`REQ-SOL-071–073`). Avoid shared writable hot accounts; shard per entity and process queues in bounded records.

## Intentional deltas

The accepted deltas are: Unix time instead of blocks; direct Pyth pull/EMA; guarded asynchronous hook cascades and losing-bid refunds; explicit bounds; per-tenant dseq; funded deposit allowances; stablecoin settlement; Token-2022 ACT burn/mint accounting; per-entity accounts and rent refunds; crank execution; owner/operator provider keys; staged provider deregistration; `act_escrowed` in CR; program registry replacing discovery; no staking/IBC/CosmWasm/x509/take; and no dependency on Alpenglow slot/finality constants.
