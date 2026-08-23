# 05. Token migration

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-05 |
| Version | 0.9 draft, 2026-08-10 |
| Requirement family | `REQ-TOK-001–064` |

## Conservation model

At cutover `C`, snapshot `S1` fixes the total target-chain entitlement. No old-chain event after S1 may increase it:

`S1 user claims + Wind-down Reserve = total AKT and ACT entitlement at S1`

`cumulative claims + residual allocations + remaining Reserve = S1 entitlement`

Post-S1 inflation, BME mints, module-account inflows, and other newly created old-chain coins are claim-inert. Old-chain top-ups spend value already credited at S1. Weekly residuals are incremental, FIFO-attributed, and capped by each escrow account's S1 principal. At S2 the Reserve must be fully allocated, apart from explicitly governed legal/operational holdbacks (`REQ-TOK-001–006`).

## S1 disposition

| Old-chain value/state | S1 treatment |
|---|---|
| Liquid user AKT/ACT | Merkle entitlement to the same micro-unit amount |
| Bonded/unbonding/redelegating AKT | Liquid AKT entitlement; consensus lock is not recreated |
| Rewards accrued through C and validator commission | Included at S1; no individualized C→H staking yield |
| Remaining vesting entitlement | Claim creates the same remaining vesting schedule; vested amount is liquid |
| Escrow, BME, community/distribution, IBC, gov deposits, and other module-held balances | Mint once into segregated Wind-down Reserve categories; never direct user claims unless already attributable |
| In-flight BME requests | Cancel deterministically; return input or create a Reserve-backed remint credit; do not execute across cutover |
| IBC-out AKT | Return-home campaign, then bounded foundation redemption reserve |
| Non-native IBC vouchers held on Akash | Return before H where possible; unresolved treatment is Q-35 |
| Live deployments/bids/leases/provider/audit/certs | Not state-migrated; archive and wind down under [06](./06-state-and-data-migration.md) |

Every module/account/denom class must appear exactly once in a published disposition ledger, with source balance, target bucket, transformation rule, and reconciliation total (`REQ-TOK-007–014`).

## ACT and BME seeding

User-held ACT is claimable as non-transferable target ACT. Escrowed ACT is represented by Reserve accounting and target `act_escrowed`, not duplicated as liquid supply. The target BME vault is seeded from the old BME module balance only after verifying its CR using the same price convention and target liability definition. If seed CR is below the warning/halt requirement, governance must recapitalize or change launch parameters before BME opens. In-flight queues are canceled and reconciled; no request straddles S1 (`REQ-TOK-015–018`).

## Merkle claims

### Canonical data

Tree hashing uses SHA-256 and domain separation. Leaves are sorted deterministically and encode fixed-width, big-endian fields:

`tag(1) | cosmos_address(20) | liquid_uakt(8) | liquid_uact(8) | locked_akt(8) | locked_act(8) | vest_start(8) | vest_end(8) | vest_type(1)`

The schema/version, tree padding rule, pair ordering, root totals, input manifest, proofs, and independent verifier are public. Integer micro-units only; no floating point. Roots are append-only and immutable (`REQ-TOK-019–023`).

### Ownership and recipient binding

The old Cosmos key authorizes a new Solana/EVM recipient; addresses are not algorithmically converted. The signed ADR-036-style Amino `StdSignDoc` payload binds:

- schema/version and leaf/root identifier;
- target chain ID and claims program/contract address;
- exact recipient address;
- nonce/expiry if defined by the canonical schema.

Verify secp256k1 over the exact bytes, reject high-S signatures, and derive the Cosmos address as `RIPEMD160(SHA256(compressed_pubkey))`; never trust a caller-supplied old address. Solana uses the secp256k1 recovery syscall plus deterministic RIPEMD-160; EVM uses `ecrecover`/precompiles and supplied compressed-pubkey validation. A receipt/bitmap enforces one claim per leaf/root (`REQ-TOK-024–029`).

Cosmos multisig accounts require the recorded `k-of-n` threshold of distinct participant signatures over the same payload and committed pubkey set. Do not accept one constituent key. Claims support Keplr, Leap, Cosmostation, current Ledger firmware, CLI/offline signing, and programmatic exchange flows. The portal must show the exact recipient being signed and warn on clipboard/domain mismatch; the CLI remains available if the portal is restricted or unavailable (`REQ-TOK-030–034`).

Default claim dust threshold is 10,000 micro-units (0.01 token), subject to Q-33. Dust is reported and separately accounted; no silent loss. Claims remain open for two years. Any sweep/extension after that window requires governance and legal approval (Q-02).

## Vesting

At S1, compute vested and unvested amounts from the original account type and schedule. Pay vested value liquid; recreate only the unvested remainder with the same absolute end date and applicable start/cliff behavior. Support continuous, delayed, and periodic/base vesting forms found in the live export. The claims path must not let a recipient bypass the lock by choosing an ordinary account; target vesting vaults/contracts release only elapsed entitlement. Rounding truncates consistently and reconciles to the leaf (`REQ-TOK-035–040`).

## Wind-down Reserve and residual roots

The Reserve is segregated by purpose and has no general withdrawal path. It covers old-chain module liabilities, BME seeding, weekly residuals, S2, IBC redemptions, exchange allocations, and the ratified validator wind-down budget (`REQ-TOK-041–043`).

Each weekly cycle during `C→H`:

1. Compute only newly honored old-chain flows: provider earnings/withdrawals, tenant/depositor refunds, canceled bid collateral, valid BME burn/refund effects, inbound IBC redemption, and other disposition-ledger items.
2. Apply S1-principal FIFO caps and exclude post-S1 mint/inflow value.
3. Publish CSV, inputs, candidate root, totals, and recompute tool.
4. Allow at least 48 hours for reproducible objections.
5. Append and attest the root; distribute from the Reserve. Default minimum payout is 1 token, with smaller amounts carried forward (Q-18).

A dispute stops only that cycle; corrected roots supersede rather than mutate. At H, virtually settle every remaining escrow/payment, force-close leases, include final refunds/earnings and validator incentive allocations, and publish S2 using the same verification standard (`REQ-TOK-044–050`).

## IBC and exchanges

Start the IBC return-home campaign at least 12 weeks before C. Snapshot relevant counterparty voucher supplies for outreach and prevent double redemption. After S1, the foundation redeems verified burned/escrowed vouchers from a bounded Reserve category for a default 12-month window, subject to Q-03/legal review. No persistent bridge is created (`REQ-TOK-051–055`).

Exchange work starts 3–6 months before C. Each venue receives mint/address data, snapshot rule, allocation/proof tooling, freeze window, finality guidance, test vectors, and escalation contacts. Old-chain deposits must be disabled at S1. Custodial swaps can begin at S1 against reserved venue allocations and are not delayed by the seven-day self-custody verification window. Maintain one canonical ticker/mint and coordinate DEX liquidity without duplicating supply (`REQ-TOK-056–060`).

## Emissions and treasury

Sovereign inflation ends. The target emissions component mints only according to the G1-ratified, hard-capped schedule and only to provider incentives and community treasury. Until Q-01 resolves, configure a zero schedule. Governance may adjust within the ratified cap through timelock; it cannot retroactively exceed the cap. Treasury funds move through explicit S1 disposition and governance-controlled target accounts, with published reconciliation (`REQ-TOK-061–064`).
