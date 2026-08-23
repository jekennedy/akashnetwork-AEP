# 10. Rollout, cutover, and old-chain sunset

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-10 |
| Version | 0.9 draft, 2026-08-10 |
| Requirement family | `REQ-ROL-001–052` |

`C=S1` is cutover; `H=S2` is the last old-chain block, normally C+90 days. Calendar dates are set only through governance and the program plan.

## Phases and gates

| Phase | Work | Exit |
|---|---|---|
| P0 | Mobilize, verify volatile facts/live state, prototype, signal vote | G0 target ratification |
| P1 | Selected architecture, token/state/off-chain design, resolve G1 questions | G1 design freeze |
| P2 | Protocol, claims, pipeline, adapter, indexer, Console build | G2 code complete/testnet/audits start |
| P3 | Public testnet, audits, load, R1–R2 | feeds G3 |
| P4 | R3, binding governance, exchange commitments, ops readiness | G3 launch readiness |
| P5 | Mainnet deployment, R4, S1, root verification, claims open | G4 S1 verified |
| P6 | 90-day wind-down and weekly residuals | feeds G5 |
| P7 | S2/halt, archives, decommission, handover | G5 close-out |

Gate packages are signed, hashed, retained through H+5 years, and become contractual evidence (`REQ-ROL-001–004`).

| Gate | Must be true |
|---|---|
| G0 | Verification sprint and signal proposal complete; community/Overclock select one target; Stage B authorized. No target-specific production build before this (`REQ-ROL-005–006`). |
| G1 | Selected docs 03/04 + 05–07 freeze at v1.0; G1 decisions resolved; exchange plan and launch thresholds ratified; later changes use impact-controlled PR (`REQ-ROL-007`). |
| G2 | Full public testnet live; audit 1 started on frozen commit; R1 passed; no old-chain state-layout upgrade through H (`REQ-ROL-008–009`). |
| G3 | All critical/high findings fixed and auditor-reverified; R3 passed; claims RC ready; binding migration/S1 vote passed; war room/monitoring ready; written venue playbooks cover ≥70% trailing-90-day AKT spot volume unless leadership records a waiver (`REQ-ROL-010–013`). |
| G4 | Sunset active; matching S1 roots and seven-day verification complete; claims ≥99% success for seven days with no P1; exchange swaps active; target marketplace and first residual cycle live (`REQ-ROL-014`). |
| G5 | S2 and halt complete; archives and H+90d decommission complete; hypercare, scorecard, handover, and key ceremony accepted (`REQ-ROL-015`). |

## Old-chain governance sequence

Execute in order (`REQ-ROL-016–020`):

1. **Signal proposal (P0):** non-binding target intent and migration shape; no date/height/fund movement.
2. **Binding migration approval (P4):** pins document hash, `v3.0.0-sunset`, approved cutover instant/height formula, H=C+90d, claims windows, and point-of-no-return text.
3. **Sunset software-upgrade proposal (by S1−21d):** concrete C height, binaries/checksums, allow-list, fee policy.
4. **Halt proposal (by H−21d):** `v3.1.0-halt` at H+1.
5. **Pre-drafted emergency proposals:** cancel/replace upgrade, extend wind-down, adjust anti-spam, or supplement validator incentives; rehearse and pre-fund before G3.

Current governance planning values are 2,500 AKT deposit, three-day vote, 20% quorum, and 14-day deposit period; verify live and expedited parameters at P0.

## Cutover runbook

The executable runbook is versioned; log every deviation (`REQ-ROL-021`).

| Time | Required action |
|---|---|
| S1−35d | Target deployment live; order intake disabled |
| S1−30d | Final exchange notices; provider daemon GA and re-registration; validator/abort channel |
| S1−28d | Claims portal/CLI RC and mainnet-fork claim drill |
| S1−21d | Submit sunset proposal with C height |
| S1−14d | R4; final IBC return-home call |
| S1−7d | ≥20 genesis providers including top five GPU-capacity providers; war room/status ready |
| S1−72h | Go/no-go 1 |
| S1−48h | Publish height/time estimate every ≤12h; notify if drift >6h |
| S1−24h | Exchanges pause old-chain deposit/withdrawal; Console blocks new old-chain deployment UI; 24/7 war room starts |
| S1−6h | Final go/no-go; last abort point |
| C | Sunset activates; S1 baseline is committed state at C−1; old chain continues wind-down |
| C→+24h | Independent export/transform, exact root match, ≥4-of-5 attestation, root/Reserve/BME seed publication |
| C→+7d | Exchanges swap against reserved allocations; public root verification |
| C+24h | Target order intake enabled |
| C+7d | First residual cycle and retrospective |
| C+8–10d | Arm matched root and open self-custody claims |

Formal exchange notice and provider GA/re-registration are mandatory by S1−30d. Height, not estimated wall time, is authoritative (`REQ-ROL-022–024`).

Go/no-go roster is the Overclock chair/ops, Vendor engineering/security/claims leads, one provider, and one validator. Quorum is 5-of-7 including chair and both Vendor leads. Go requires those three unanimously plus present-member majority; the chair may abort alone (`REQ-ROL-025–026`).

S1 pipelines run from independent nodes/operators and must match bit-for-bit. Attestation is ≥4-of-5 across Overclock, Vendor, two independent community auditors, and Vendor security lead, signing root, height, and conservation totals. Publication target is ≤24 hours; >24 hours is an incident and >72 hours invokes contingency. Claims cannot open before attestation plus the full seven-day verification; exchanges are not gated by this window (`REQ-ROL-027–030`).

## War room and operating thresholds

Run 24/7 S1−24h through at least S1+72h with incident command, claims, protocol, old-chain, indexer/Console, and comms on every shift (`REQ-ROL-031–032`).

| Metric | Trigger |
|---|---|
| Claims success | <99%/15m → P1 and pause evaluation |
| Root comparison | Any mismatch → stop publication |
| Non-user protocol tx failure | >0.5%/30m → P1 |
| Escrow lag | p95 >60s/1h → P2/capacity response |
| Old-chain block interval | >15s for 30m → validator escalation |
| Online bonded power | <80% → continuity contingency |
| Exchange coverage at S1+7d | <95% exchange-held supply → BD escalation |
| Portal availability | <99.5% daily → P2 |

Solana cluster halt or EVM sequencer outage delays target openings; old-chain wind-down continues.

## Weekly wind-down

Each week: pin/export Monday, dual diff and candidate root Tuesday, ≥48-hour public dispute window, attest/publish Thursday, and update KPIs Friday. A valid objection halts only that cycle and corrected output supersedes the candidate (`REQ-ROL-033–034`).

Publish validator uptime/incentive accrual and target migration KPIs weekly: re-registered providers/GPU capacity, old/new leases, old escrow value, and residuals. Automatic interventions (`REQ-ROL-035–038`):

- <50% GPU capacity at C+30d: incentive boost and white-glove top-provider outreach.
- <60% at C+45d: add governance discussion of extension.
- new-chain leases <30% baseline at C+45d: tenant credits/comms.
- old escrow >25% S1 level at C+75d: targeted remaining-lease/forced-close outreach.

Notify active old-chain tenants weekly and directly at C+30/60/75d, with final forced-close warning by H−7d.

## Abort and contingency

Before C the old chain is unaffected. Abort through expedited cancellation until about S1−36h, then coordinated `--unsafe-skip-upgrades C` by >2/3 voting power until S1−6h; ratify on chain afterward. Rehearse both (`REQ-ROL-039`).

The point of no return is execution of S1: sunset activation at C plus start of custodial swaps. Root publication/claims-open is the final verification milestone; after claims open the S1 accounting cannot be adjusted (`REQ-ROL-040`).

| Scenario | Immediate action |
|---|---|
| Claims defect | Pause claims only, patch through expedited timelock, keep marketplace/residuals running |
| Root mismatch | Hold ≤72h, triage; third implementation adjudicates if unresolved |
| Venue delay | Preserve allocation; publish venue status; no chain change |
| Target outage | Delay target openings until stable; old chain continues |
| Provider no-show/validator attrition | Incentives/extension; worst case accelerate S2 from last good height |

Exercise every row in R3–R4. Claims guardian cannot pause marketplace, already attested residuals, or claimed AKT transfers. No silent root delay beyond 72 hours (`REQ-ROL-041–043`).

## Notifications, hypercare, and handover

Send IBC counterparty notice by S1−60d, exchange technical pack ≥16 weeks before C and final notice S1−30d, provider/validator notices S1−30d, and run public status from S1−7d through G5. The exchange pack contains snapshot rule, proof tools, allocations, target integration/finality, freeze guidance, vectors, and contacts (`REQ-ROL-044–047`).

Hypercare from S1 through H: P1 response 2h/24×7, workaround 12h, target resolution 72h; P2 response 8h, workaround 48h, resolution 7d; P3 24h; P4 72h. Exit after 30 days without P1, ≤2 open P2s, and Overclock independently executes every runbook (`REQ-ROL-048`).

Publish H/G5 scorecards: zero unrecovered fund-loss incidents, S1 root ≤24h, claims on C+8–10d schedule, and 13/13 residual cycles. Deliver executable runbooks, thresholds-as-code, dashboards, incident docs, and operator training; complete a witnessed ceremony for upgrades, timelock, guardian, keeper, attestors, and mint authorities. Archive/decommission per [06](./06-state-and-data-migration.md) (`REQ-ROL-049–052`).
