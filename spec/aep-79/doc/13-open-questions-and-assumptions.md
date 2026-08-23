# 13. Decisions, assumptions, and open questions

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-13 |
| Version | 0.9 draft, 2026-08-10 |
| Purpose | Canonical index of what is fixed, assumed, and unresolved |

Reopening a fixed decision requires client approval, impact analysis, and affected-document updates. A failed assumption triggers immediate review of its linked design/risk. Resolve each question by its deadline and record the answer as a decision amendment.

## Decisions

| ID | Status | Decision |
|---|---|---|
| D-01 | Open at G0 | Choose one path: Solana mainnet or an existing EVM L2. Stage A provides evidence; governance/leadership decides, not the Vendor. |
| D-02 | Fixed | No new sovereign chain. EVM B1 is an existing L2 selected by Q-42. B2 is a non-default Orbit L3 on Arbitrum One with AnyTrust and AKT gas; reconsider only under Q-06. |
| D-03 | Provisional G1 | Solana AKT is one 6-decimal Token-2022 mint with metadata only, no hooks/freeze. Use legacy SPL only if Q-05 venue/custody evidence fails. |
| D-04 | Fixed | EVM AKT is 6-decimal OpenZeppelin ERC-20 with permit and timestamp-clock `ERC20Votes`. Six decimals eliminate migration rescaling. |
| D-05 | Fixed | Dual snapshot: S1 at C, weekly residuals, S2 at H=C+90d; Merkle claims and exchange swaps; no persistent bridge. S1 fixes supply. Point of no return is sunset activation plus start of custodial swaps. Publish root ≤24h, verify publicly seven days, open self-custody claims ≈C+8–10d. |
| D-06 | Fixed | Bonded/unbonding AKT and rewards through C become liquid S1 entitlement; vesting is recreated with original absolute schedule. C→H staking yield is not individualized; Q-13 is the continuity compensation. |
| D-07 | Provisional G1 | Campaign to return IBC-out AKT before C, then bounded foundation redemption from reserved supply; no per-counterparty proof bridge. |
| D-08 | Fixed | Do not migrate live marketplace state. Existing leases wind down for 90 days; providers/tenants redeploy, and old-chain earnings/refunds are paid through residual roots. |
| D-09 | Fixed | Preserve order→bid→tenant-selected lease, SDL/manifest hashing, dseq/gseq/oseq UX; no SDL redesign. |
| D-10 | Fixed | Drop x509 `x/cert`. Provider registry anchors cold owner, rotating operator/JWT keys, and TLS SPKI; tenant auth is wallet-signed JWT (EdDSA or EIP-191/ERC-1271). |
| D-11 | Provisional G1 | Solana governance: Realms + Squads + timelock. EVM: OZ Governor + Safe + timelock. EVM quorum launch default 10% supply, calibrated by Q-23. |
| D-12 | Fixed; curve Q-01 | Sovereign inflation ends. Hard-capped target emissions go only to provider incentives and community treasury under DAO+timelock. |
| D-13 | Fixed | Direct Pyth pull feeds. During C→H retain pinned old-chain Pyth execute/oracle messages and operate Hermes through H. |
| D-14 | Fixed; asset Q-43 | Add one governed native liquid stablecoin as settlement beside AKT. Prices remain ACT-denominated; default parity is `stable_act_parity_bps=10000`; PSM decision is Q-22. |
| D-15 | Fixed | Launch upgradeable behind multisig+timelock, then DAO-only control and a published path to immutable claims/escrow core. |
| D-16 | Fixed | Dedicated indexer: Solana Geyser/webhooks or EVM logs → Postgres, preserving Console shapes. `/v1` freezes archive/Cosmos shapes; `/v2` is target-native with `chain_id`. |
| D-17 | Provisional G1 | Add `pkg/chain` to provider-services. Solana may use a localhost signer sidecar if Q-15 says Go-only is insufficient; EVM uses standard generated bindings. |
| D-18 | Fixed | `v3.0.0-sunset` at C blocks new marketplace intake and raises fees; `v3.1.0-halt` has no binary at H+1. H is last committed/S2 height; decommission by H+90d. |
| D-19 | Fixed | ACT remains non-transferable and pricing remains ACT. Target escrow burns ACT on deposit and tracks `act_escrowed`; BME CR denominator includes minted plus escrowed liability. |
| D-20 | Fixed | Port queued epoch-batched AKT↔ACT BME, 9500/9000-bps breaker, spread, vault, and CR-halt exit. Use timestamp cranks/keepers and governed per-epoch caps (Q-25); tip source/rate Q-39. |
| D-21 | Fixed | Port lazy per-second escrow, permissionless settle, FIFO multi-depositor/refund, overdrawn, and AKT fallback. Rates are `u128 ×10^18`; old conversion is exact `×2/13`. Deposit allowances are funded-at-grant, restore-on-refund, revoke-returns-remainder. |
| D-22 | Fixed | Do not port general CosmWasm/awasm; it only served Pyth plumbing, now direct. |
| D-23 | Fixed | Shard live state per entity; close terminal state/rent; append-only events are history. Replace hook cascades with guarded permissionless reaps ≤60s p95; Solana losing-bid refunds are asynchronous. |
| D-24 | Fixed | Preserve provider reclamation bounds exactly: minimum 1 hour, maximum 720 hours. |

Additional fixed bounds pending corpus validation: 8 groups/deployment, 4 resource units/group, 24 attributes, 4 `signed_by`, and initial 16 depositors.

### Ratified amendments

These stable amendment IDs refine the parent decisions; their substance is integrated above and in the owning technical document.

| ID | Refinement |
|---|---|
| D-02.a | B2 is an Orbit L3 on Arbitrum One with AnyTrust, parent-chain claims/AKT, and a WAKT gas wrapper; B1 host stays configurable until G1. |
| D-04.a | EVM AKT includes timestamp-clock `ERC20Votes`. |
| D-05.a | S1 execution is the point of no return; abort paths end before C. |
| D-05.b | Publish root ≤24h, verify for seven days, then open self-custody claims ≈C+8–10d; exchange swaps start at S1. |
| D-05.c | Supply is fixed at S1; later mints/inflows are claim-inert and residuals are FIFO-capped to S1 principal. |
| D-06.a | S1 includes rewards only through C; Q-13 compensates C→H continuity. |
| D-10.a | Provider identity is cold owner + rotating hot keys + TLS SPKI; JWT uses EdDSA or EIP-191/ERC-1271 and offline/challenge modes. |
| D-11.a | EVM Governor launch quorum defaults to 10% of supply, subject to Q-23. |
| D-13.a | Keep the pinned old-chain Pyth/Hermes path alive until H. |
| D-14.a | Stablecoin settles ACT obligations at governed parity, default 10,000 bps; asset Q-43 and optional PSM Q-22. |
| D-16.a | `/v1` is frozen Cosmos/archive API; `/v2` is target-native with `chain_id`. |
| D-18.a | Sunset is `v3.0.0-sunset`; halt is no-binary `v3.1.0-halt` at H+1; S2 virtually settles remaining escrow. |
| D-19.a | Burn ACT on escrow deposit and include `act_escrowed` in CR liability. |
| D-20.a | Use permissionless Solana cranks or Chainlink+Gelato EVM automation; add governed per-epoch caps. |
| D-21.a | Rates are `u128 ×10^18` per second; conversion from current rates is exactly `×2/13`. |
| D-21.b | Deposit allowances are funded at grant, restored on refund, and return remainder on revoke. |
| D-23.a | Guarded reaps replace keeper hooks with ≤60s p95; Solana losing-bid refunds are asynchronous. |

## Assumptions

| ID | Assumption and validation | If false |
|---|---|---|
| A-01 | `akashnet-2` stays healthy; no state-layout upgrade G2→H. Coordinate/freeze. | Rework pipeline/sunset/schedule. |
| A-02 | Governance approves migration and halt sequence. Signal then binding votes. | Stop/replan entire program. |
| A-03 | Overclock supplies ≥2 counterpart engineers plus product/comms. Contract it. | Schedule/cost rebaseline. |
| A-04 | 85–90% circulating AKT is reachable by venues/active claims within 12 months. Track commitments/claims. | Extend support/liquidity/comms. |
| A-05 | Providers accept one daemon upgrade and re-registration. Validate council/testnet. | Launch capacity risk. |
| A-06 | No economically material adversarial old-chain fork. Validate venue identity alignment. | Ticker/brand/venue dispute. |
| A-07 | Ecosystem facts are current as of 2026-08. Reverify all at kickoff. | Reopen target/design/cost. |
| A-08 | Selected target has an unrestricted native liquid stablecoin. Verify Q-43. | D-14/target viability. |
| A-09 | Venues retain AKT ticker. Confirm Q-04. | Exchange/comms rework. |
| A-10 | SDL/manifest semantics need no change. Conformance-test existing library. | Protocol/provider/client redesign. |
| A-11 | Console API shapes can remain except address/hash. Contract-test. | Indexer/Console change request. |
| A-12 | Baseline is node `096bff57`; fold any pre-G1 changes into 01/14. | Rebaseline architecture/parity. |
| A-13 | Alpenglow changes timing/finality, not program semantics. Track/soak. | Solana schedule/design review. |
| A-14 | Client decides G0 ≤10 business days after M1. | Stage B slips day-for-day. |
| A-15 | One prime Vendor; approved subcontractors remain under prime liability. | SOW/vendor governance change. |
| A-16 | Full archive state/tx/event history is available. Provision at kickoff. | Golden vectors/residual pipeline blocked. |
| A-17 | 4-of-7, ≤2/org, ≥3 jurisdictions is launch baseline; ceremony fixes exact roster. | Security design reapproval. |
| A-18 | New bounds fit full mainnet SDL corpus. Validate Q-38. | Resize/account/gas redesign. |
| A-19 | Venues can claim programmatically and permanently disable old-chain deposits at S1. Test Q-04. | Custodial swap redesign/delay. |
| A-20 | Dual-chain provider Kubernetes names do not collide. Test with chain namespace. | Provider adapter/name migration. |
| A-21 | `bseq=0` is API-only and can leave target identity. Confirm parity. | Seed/storage/client change. |
| A-22 | API baseline v0.2.14 matches cited v0.3.0 shapes. Rebaseline at kickoff. | 01/14 corrections. |
| A-23 | >2/3 bonded power remains online C→H with Q-13. Obtain signed commitments. | Restart or early S2. |

## Open questions

| ID | Decision needed | Owner | Due |
|---|---|---|---|
| Q-01 | Emissions rate, decay, provider/treasury split, hard cap | Finance/community; Vendor models | G1 |
| Q-02 | Policy/legal treatment after two-year unclaimed window | Legal | G2 |
| Q-03 | IBC counterparty set, redemption duration, KYC/policy | Overclock/legal | G1 |
| Q-04 | Venue list, lead times, volume coverage, integration needs | Overclock BD | G1 |
| Q-05 | Token-2022 venue/custody matrix and legacy-SPL fallback trigger | Vendor | G1 |
| Q-06 | Sustained B1 fee/throughput threshold that reopens Orbit B2 | Vendor/Overclock | G1 |
| Q-07 | Post-launch owner/budget for sponsor, indexer, RPC, automation | Overclock | G2 |
| Q-08 | Provider/bid collateral, non-refundable fee, and slashing/forfeiture policy | Protocol WG | G1 |
| Q-09 | Archive vendor, ≥5-year retention, and funding | Overclock | G3 |
| Q-10 | Claims legal/MiCA/sanctions/portal screening posture | Legal | G2 |
| Q-11 | Stock Realms versus minimal fork; council/veto gaps | Vendor | G1 |
| Q-12 | Program counter dseq versus any block-height dependency | Vendor/Console | G1 |
| Q-13 | Validator/delegator wind-down incentives, power caps, uptime bands | Overclock/community | G2 |
| Q-14 | Console managed-wallet migration/custody/comms | Console | G2 |
| Q-15 | Whether Go-only Solana integration removes sidecar | Vendor prototype | G1 |
| Q-16 | ACT metadata and wallet/Console presentation | Console/Vendor | G2 |
| Q-17 | Sunset allow-list and old-chain fee/gas anti-spam values | Vendor/core | G3 |
| Q-18 | Residual cadence/minimum payout; weekly/1-token defaults | Vendor | G2 |
| Q-19 | Live inflation/vesting/supply/params/state/op-mix export | Vendor | G1 |
| Q-20 | Audit firms and budget; ≥2 protocol + migration review | Overclock/Vendor | G1 |
| Q-21 | Provider-testnet rewards budget/source | Overclock | G1 |
| Q-22 | Stablecoin parity ratification and optional PSM | Protocol WG | G1 |
| Q-23 | EVM Governor quorum/threshold calibration | Governance WG | G1 |
| Q-24 | Secondary oracle and switch criteria | Vendor | G1 |
| Q-25 | BME per-epoch volume-cap values | Vendor/protocol WG | G1 |
| Q-26 | Validator uptime commitment mechanism/threshold | Overclock | G2 |
| Q-27 | At least two venues for pre-G3 sandbox test | Overclock BD | G2 |
| Q-28 | Immutability burn-in N; default proposal 12 months | Steering | G2 |
| Q-29 | Claims 24-hour velocity-breaker threshold | Vendor/steering | G2 |
| Q-30 | Whether monitors may auto-trigger preauthorized intake pause | Steering | G2 |
| Q-31 | Bounty platform and final tiers | Overclock | G2 |
| Q-32 | Economic-security reviewer for BME/emissions | Overclock/Vendor | G1 |
| Q-33 | Claim dust threshold; default 10,000 micro-units, and legal sweep | Legal/steering | G2 |
| Q-34 | DEX liquidity amount, venues, LP policy, treasury carve-out | Overclock finance | G2 |
| Q-35 | Non-native IBC voucher balances remaining at H | Overclock/legal | G3 |
| Q-36 | Signer sidecar language/package if Q-15 retains it | Vendor | G1 |
| Q-37 | CLI support for Cosmos multisig ADR-036 signing | Vendor | G2 |
| Q-38 | Validate all explicit bounds against mainnet SDL; confirm depositors=16 | Vendor | G1 |
| Q-39 | Crank tip funding source and rates | Vendor/protocol WG | G1 |
| Q-40 | Whether provider may re-bid after closing its own bid | Vendor | G1 |
| Q-41 | Actual BME pricing window: apparent 1h BME versus 5s oracle TWAP | Vendor code trace | G1 |
| Q-42 | EVM B1 host chain against 04 criteria | Overclock/Vendor | G0 |
| Q-43 | Native settlement stablecoin by liquidity, issuer/regulation, freeze policy, decimals, availability | Overclock/Vendor | G1 |

## Reverify at kickoff

- Solana: fee/CU/account limits, rent, Token-2022 venue/custody support, Pyth AKT feed, Alpenglow/Firedancer status, Realms maintenance, RPC/ALT tooling.
- EVM: candidate host fees/throughput/fault proofs/forced inclusion/sequencer roadmap/policy, blob fees, ERC-4337 and keeper availability, Pyth, native stablecoin, indexer support, Orbit costs/terms.
- Both: wallet/Ledger/WalletConnect support, audit-firm availability, RPC SLAs, exchange requirements, and current legal/regulatory posture.
