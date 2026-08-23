# 09. Testing and verification

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-09 |
| Version | 0.9 draft, 2026-08-10 |
| Requirement family | `REQ-TST-001–064` |

Behavioral equivalence with `akashnet-2` is the primary acceptance criterion. Prove it with recorded execution, exact integer comparison, properties, full-stack scenarios, and rehearsals—not code-review assertion.

## Verification layers

| Layer | Scope | Cadence |
|---|---|---|
| L1 unit | Instruction/function correctness and errors; LiteSVM/Mollusk or Foundry | Every PR |
| L2 property/fuzz/invariant | Stateful adversarial operations and [08](./08-security-and-audits.md) I-1–I-7 | Bounded PR, 4h nightly, 24h weekly |
| L3 integration localnet | Target protocol + tokens/oracle/cranks/indexer/provider/Console/claims | PR subset; full nightly |
| L4 differential parity | Current keeper/mainnet vectors replayed on target | PR core; full nightly/release |
| L5 fork | Real Pyth/stablecoin/governance dependencies | Nightly and release |
| L6 public testnet | Real wallets, providers, hardware, and operations | Continuous |
| L7 rehearsals | S1, wind-down, residuals, S2, rollback | R1–R4 |
| L8 chaos/load | Throughput, congestion, dependency faults | Weekly and pre-gate |
| L9 audits/bounty | Findings become permanent regression tests | Each finding |

Keep all harnesses, vectors, CI, and reports in public repositories or pinned submodules, runnable without Vendor infrastructure. Name a test owner and map every requirement to evidence. Every external defect gets a failing regression before its fix merges (`REQ-TST-001–005`).

## Golden-vector parity harness

A canonical JSON vector contains version, ID/family/source, pre-state, ordered timestamped operations, expected post-state, transfers, events, and normalization ledger. Generate vectors mechanically from instrumented current Go keeper tests and archive-node mainnet histories; self-replay them through the current keepers before accepting them (`REQ-TST-006–010`).

Required families: escrow settle, FIFO, overdrawn, fallback, refund/allowance; BME queue/CR/status; market match/reclamation; deployment/group lifecycle. By G2 include at least 250 complete mainnet lifecycles and 500 total vectors. Sample long leases, multiple denoms/depositors, authz funding, overdrawn closure, BME transitions, and high-frequency withdrawal.

Normalization permits only:

1. the single shared rate conversion `per_block ×2/13 = per_second`; and
2. signed-off `ROUND-nn` entries describing an unavoidable rounding difference and worst-case bound.

Everything uses integer/fixed-point math. After normalization, all micro-unit balances, transfers, debt, spread, states, and relevant events must match exactly. Each release publishes corpus counts, results, rounding ledger, divergences, and reproduction instructions. Escrow/BME/market/claims/token changes cannot merge with a parity failure (`REQ-TST-011–016`).

## Properties, coverage, and budgets

Encode I-1–I-7 once in a declarative form used by both fuzz suites and runtime monitors. Add settle idempotency, close-always-refunds, allowance restoration, concurrent one-claim, and Solana rent-refund properties (`REQ-TST-017`).

- Overall line coverage ≥85%.
- Branch coverage 100% for claims verification, escrow settlement/FIFO/overdrawn/fallback, and BME CR/status.
- PR fuzz ≤10 minutes; nightly ≥4 hours; weekly ≥24 hours with persistent corpus.
- Solana tests execute compiled programs, validate account model, enforce declared CU budgets, and block >5% unapproved regression.
- EVM Foundry invariants run at least 256 runs × depth 15 nightly with adversarial/reentrant handlers; gas snapshots block >5% unapproved regression (`REQ-TST-018–024`).

## One-command integration environment

Within five minutes, one deterministic command must start local chain, all programs/contracts, AKT/ACT/test stablecoin, scriptable Pyth (fixed/ramp/stale/confidence spike), crank/keeper, Postgres indexer, provider adapter with mock Kubernetes, Console dev build, fixture claims, and invariant monitor. Every scenario asserts both chain and indexer state (`REQ-TST-025–029`).

| IDs | Scenario group |
|---|---|
| S-01–S-04 | Happy lifecycle; multi-depositor FIFO/allowance; overdrawn; cascade/relist/losing-bid refund |
| S-05–S-07 | Bid cap/matching/audited attrs; pause/start/update manifest; close/refund |
| S-08–S-12 | AKT fallback; BME batch/breaker/retry; reclamation |
| S-13–S-14 | Provider key rotation and end-to-end JWT authorization |
| S-15–S-18 | Single/multisig/vesting claims and weekly residual distribution |
| S-19–S-23 | Overdrawn top-up, stablecoin, oracle degradation, governance param, emissions |
| S-24–S-26 | Indexer replay/reorg, manifest hash, one-hour automation outage and recovery |

The PR subset must include the main lifecycle, escrow edge cases, BME halt/fallback, JWT, single claim, and indexer recovery.

## Public testnet

1. Internal devnet: 14 consecutive days, full scenarios green, clean invariants.
2. Incentivized provider testnet for at least six weeks: ≥25 independent providers across ≥3 regions, ≥5 GPU providers, each sustaining leases for ≥2 weeks, ≥500 deployments, load targets met, no open sev-1/2.
3. Community dress rehearsal: real pipeline claims, provider re-registration, residual cycle, and exchange sandbox; no blocker for 14 days.

Continuously deploy `main` to internal devnet; tagged signed releases promote to public testnet. Include the bounty by phase 2 and generate a claims rehearsal from a testnetified real export (`REQ-TST-030–034`).

## Rehearsals

| Run | Purpose | Minimum pass criteria |
|---|---|---|
| R1 | Pipeline shakeout on forked mainnet | Independent roots match; ≤2× time budget; ≥100 sampled claims incl. vesting |
| R2 | Full cutover/wind-down | Timing met; real Ledger/multisig/vesting claims; two residual cycles; pre-S1 rollback drill |
| R3 | Frozen community-observed dress | All R2 criteria, no off-runbook intervention, exchange sandbox |
| R4 | Fresh final dress at S1−14d | Reconfirm R3 on frozen release/runbook; failure invokes abort review |

Run R1–R3 before G3. Any material post-R3 change repeats the rehearsal. Each root must match across independent implementations. Exercise claims-open after the complete seven-day verification sequence on a compressed rehearsal clock. At least two venues test swap allocations, pause/resume, and address validation. Publish timings, roots, samples, defects, and runbook changes (`REQ-TST-035–042`).

## Performance and chaos

Derive the operation mix from Q-19 mainnet data and deliver the load generator. Acceptance (`REQ-TST-043–053`):

- ≥50 mixed marketplace tx/s for 60 minutes with no protocol failures.
- ≥500 tx/s burst for 60 seconds with bounded, lossless retry.
- due→settled lag ≤60 seconds p95.
- chain event→API ≤2 seconds p95 and ≤10 seconds p99.
- order seen→provider bid landed ≤5 seconds p95.
- Weekly one-hour automation outage, stale oracle, RPC brownout, indexer restart, and EVM reorg chaos with zero invariant breach.
- Solana: congestion fee policy lands ≥99% within ceilings and shared-PDA contention respects current CU limits.
- EVM: replay historical L1/blob fee spikes; correctness operations may delay only within documented degraded bounds.
- A >10% performance regression blocks release review.

## CI, release, and traceability

PR-required suites must finish within 30 minutes. Nightly adds full vectors/scenarios/fork and four-hour fuzz; weekly adds deep fuzz/chaos; release runs everything. Regenerate/diff IDL or ABI on every build, require semver and changelog for drift, verify reproducible bytecode/source, sign tags, and deploy main→devnet within one hour through the same promotion pipeline used for mainnet (`REQ-TST-054–059`).

Maintain a machine-readable matrix mapping every in-scope `REQ-*` to deliverable, test/property/scenario/rehearsal, result, and waiver. At G3 there are no orphans; waivers are client-approved, time/gate-bounded, and never cover security-critical requirements. Gate evidence includes matrix, parity/coverage/CU/gas reports, rehearsal/testnet reports, and defects (`REQ-TST-060–063`).

Before public cutover announcement, mainnet smoke with real funds: one complete deployment→lease→settle→close, one BME round trip, and three reserved claims (hot wallet, Ledger, multisig), all reconciled to the micro-unit (`REQ-TST-064`).
