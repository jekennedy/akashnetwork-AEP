# 06. State and data migration

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-06 |
| Version | 0.9 draft, 2026-08-10 |
| Requirement family | `REQ-STA-001–064` |

## State treatments

Every exported object receives exactly one treatment (`REQ-STA-001–006`):

| Treatment | Examples |
|---|---|
| Claim | Liquid balances, stake/unbonding/rewards through C, vesting entitlement |
| Reserve | Escrow/BME/community/IBC/gov and other module liabilities |
| Re-register | Providers, operator keys, audit attestations |
| Wind down then S2 | Deployments, groups, orders, bids, leases, payments, residual refunds/earnings |
| Archive/retire | Validators, staking metadata, IBC clients/channels, x509 certs, CosmWasm, historical events |

The supply perimeter is fixed at S1. Only value traceable to an S1 entitlement or Reserve principal can create target credit. Post-S1 mints and inflows are ignored except as operational state needed to finish old-chain accounting.

## Deterministic export pipeline

Use the exact release-native `akash export`/state APIs at a pinned binary and container digest, with `for_zero_height=false`. Two independently synced archive nodes must agree on height, block ID, app hash, and export bytes before transformation (`REQ-STA-007–010`).

Pipeline stages:

1. Pin chain ID, height, block/app hashes, binary/source commit, genesis hash, container/tool versions, RPC endpoints, and all inputs in a signed manifest.
2. Export raw state, transactions/events needed for replay, and live parameters.
3. Normalize protobuf/JSON deterministically; sort by binary primary key/address; use integer micro-units only.
4. Produce the disposition ledger, claim rows, vesting rows, Reserve categories, BME seed, IBC inventory, provider/audit inventory, and old-marketplace inventory.
5. Build roots/proofs and supply-conservation reports with two independent implementations.

Per run, publish raw export, normalized tables, full leaf CSV/JSON, roots/proofs, Reserve ledger, module totals, exception/dust ledger, tool source, lockfiles/container digests, and a one-command verifier. Hash every artifact in the manifest (`REQ-STA-011–018`).

Rounding is explicit and deterministic: no binary floating point; fixed decimal exponents; truncate at specified transfer boundaries; record aggregate dust by asset and source. Target-specific address/encoding occurs only after canonical Cosmos-side normalization. The pipeline must complete compute in ≤6 hours and publish an attested root ≤24 hours after C at twice the measured mainnet data size (`REQ-STA-019–023`).

## Residual replay

Weekly residuals are computed by event-sourced replay between successive pinned heights, then reconciled to state. The classifier must cover deployment/market close, escrow settle/withdraw/refund/top-up, bid collateral, BME burn/cancel/refund, inbound IBC returns, bank movements affecting reserved categories, governance deposits, and terminal virtual settlement. Unknown or ambiguous flows halt the cycle; they are not guessed (`REQ-STA-024–028`).

For each escrow account, attribute target credit in original depositor FIFO order using `Depositor.Height` and cap total credit at S1 principal. Post-S1 top-ups spend already-credited value and cannot mint a second entitlement. Provider earnings are credited only from Reserve-backed S1 escrow principal. At H, simulate settlement at the final timestamp, close remaining payments/accounts, refund depositors, and assign provider earnings even if the old-chain message was never submitted (`REQ-STA-029–033`).

Publish each cycle's start/end heights and app hashes, event set, state reconciliation, per-address incremental amounts, exclusions, carry-forward dust, candidate root, and reproducible command. Target lag is ≤7 days; candidate publication by 96 hours leaves a 48-hour dispute window (`REQ-STA-034–037`).

## Verification

- Build two transformation implementations with non-overlapping engineers and no shared code beyond canonical schemas. They must produce byte-identical roots for S1, every residual, and S2.
- Check supply conservation, uniqueness, non-negative values, module-balance coverage, vesting totals/end dates, queue cancellation, IBC coverage, FIFO caps, no post-S1 value creation, and root/proof recomputation.
- S1/S2 receive a seven-day public verification period. Weekly roots receive at least 48 hours.
- Every report binds the source block/app hash. S1 registration requires the matched roots and signed attestations from Vendor, Overclock, at least one independent verifier, and participating community/validator reviewers as defined in [10](./10-rollout-and-cutover.md).
- Any mismatch is blocking. Correct by publishing a new append-only candidate; never mutate a registered root (`REQ-STA-038–045`).

## Sunset upgrade

`v3.0.0-sunset` activates at C without changing state layout or balances. It installs a recursive default-deny message filter and anti-spam parameters. The allow-list is limited to wind-down operations (`REQ-STA-046–052`):

| Allowed C→H | Restrictions |
|---|---|
| Escrow deposit, settle, withdraw, close/refund | Existing accounts only; no new deployment/bid accounts |
| Market/deployment close, pause, reclaim | No new deployment/order/bid/lease intake |
| BME ACT→AKT burn/refund and vault-safe operations | No AKT→ACT minting that creates claim value |
| Bank sends | Needed for refunds/operations; no claim effect for new value |
| Pyth CosmWasm execute + oracle add | Pinned contract/source only; Hermes pusher remains operated until H |
| Staking/distribution/slashing/evidence | Chain liveness and rewards already accounted only through C; later mint is claim-inert |
| Limited governance, authz, feegrant, cert revoke | Emergency operations only; no state-expanding replacements |
| IBC client/packet processing | Inbound return packets only; new outbound transfers blocked |
| Software upgrade | Emergency changes and final halt |

Provisional anti-spam values are 0.5 uAKT/gas and 30 million block gas, calibrated by Q-17. Keep oracle updates alive: otherwise `halt_oracle` prevents ACT refunds and AKT fallback. Apply no state-layout-affecting old-chain upgrade from G2 through H except an audited emergency.

At H, record the last committed block and S2 export. Schedule `v3.1.0-halt` at H+1 with no binary so validators stop consistently. H normally equals C+90 days; only governance may change it (`REQ-STA-053–055`).

## Validator continuity

Old-chain consensus needs >2/3 bonded voting power through H. Before C, obtain signed commitments and fund Q-13. Weekly, measure signed-block uptime and voting power; publish eligibility/accrual. The final formula must cap concentration, use agreed uptime bands (proposals 90%/80%), forfeit on equivocation, and never create entitlement above its S1 Reserve category. Alert below 80% online; below 2/3 invokes coordinated restart or accelerated S2 from the last good height. Slashing remains active, including double-sign protection (`REQ-STA-056–059`).

## Archives, decommission, and rehearsals

At H publish at least: raw final export, genesis and upgrade binaries/checksums, block/tx/event archive, indexer database, provider/audit/cert history, weekly/S1/S2 manifests and proofs, static explorer/API, source/toolchain, and validator/IBC metadata. Sign and content-hash artifacts, mirror across independent providers, document retrieval, and retain for at least five years subject to Q-09 (`REQ-STA-060–062`).

Keep read-only RPC/explorer/snapshot services during the 90-day post-H transition; then remove validators/seeds/relayers/Hermes/write APIs, rotate credentials, preserve archives, and publish the decommission record.

Run R1 pipeline shakeout, R2 full wind-down/residual rehearsal, R3 mainnet-scale frozen dress rehearsal before G3, and R4 fresh final dress at S1−14d. Every run uses both exporters and records timings, roots, discrepancies, and corrective changes (`REQ-STA-063–064`).
