# 11. Vendor scope of work

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-11 |
| Version | 0.9 draft, 2026-08-10 |
| Client | Akash Network via Overclock Labs |
| Requirement family | `REQ-SOW-001–054` |

The executed contract prevails, followed by this SOW, numbered requirements in docs 03–10, then informative text. Pin the incorporated AEP-79 version and trace every obligation to a `REQ-*` or `D-WS*` identifier. Conflicts and changes use written change control (`REQ-SOW-001–002`).

## Commercial structure

| Stage | Price | Scope |
|---|---|---|
| A: target-neutral, T0→M1 | Firm fixed | Volatile-fact/live-state verification, economics start, migration design, both-path feasibility/benchmarks, Q-15 prototype, Gate 0 package |
| B: selected target, G0→M8 | Fixed by milestone, bid separately per path | One target only: build, audit, testnet, launch, S1, wind-down, S2, hypercare, handover |

Do not start target-exclusive production implementation before accepted M1 and G0. Retain the unselected design for bounded fallback documentation only. Rebaseline Stage B from the actual G0 date (`REQ-SOW-003–006`).

## Workstreams and deliverables

| WS | Scope | Required outputs / acceptance |
|---|---|---|
| WS0 Program/architecture | Plan, weekly/monthly reporting, traceability, risk/decision/change control, gate packages | D-WS0.1–0.5; every requirement has status/deliverable/test; packages ≥5 business days before gate (`REQ-SOW-007–009`) |
| WS1 Economics | Live supply workbook, ≥3 emissions scenarios, BME parameter study, validator continuity budget | D-WS1.1–1.5; reproducible source+pinned inputs; Vendor models options, governance decides (`REQ-SOW-010–011`) |
| WS2 On-chain protocol | Selected tokens, marketplace, escrow, BME, provider/audit, config/governance, oracle, stablecoin, emissions | D-WS2.1–2.6; 100% selected `REQ-SOL-*` or `REQ-EVM-*`, verified mainnet builds, code/test traceability (`REQ-SOW-012–013`) |
| WS3 Migration engine | Two exporters, Merkle/claims/vesting, weekly residuals, IBC redemption, sunset/halt, reconciliation | D-WS3.1–3.7; independent non-overlapping implementations produce identical roots every run; residual lag ≤7 days (`REQ-SOW-014–016`) |
| WS4 Off-chain | Provider adapter/optional sidecar, indexer, TS/Go SDK, CLI, portal, provider wizard, Console integration | D-WS4.1–4.7; preserve Console API shapes except address/hash; ship guides/matrices with clients (`REQ-SOW-017–018`) |
| WS5 Security | Threat reviews, two-firm on-chain audits, migration review, bounty, monitors, ceremonies, incident drills | D-WS5.1–5.6; no open critical/high at mainnet/G3; bounty live before deployment through hypercare (`REQ-SOW-019–021`) |
| WS6 Quality/testnets | Parity vectors, invariants/fuzz, public testnet, rehearsals, load/chaos | D-WS6.1–6.5; mechanically derived vectors; at least R1–R3 before G3 plus R4 per rollout (`REQ-SOW-022–023`) |
| WS7 Launch/wind-down | Mainnet, S1, ≈13 residual cycles, S2/halt/archives, hypercare | D-WS7.1–7.5; act only on recorded go; P1 response ≤2h and mitigation plan ≤24h, P2 ≤1 business day (`REQ-SOW-024–025`) |
| WS8 Docs/handover | Protocol/API, provider/tenant/holder/exchange guides, runbooks, training | D-WS8.1–8.5; exchange pack ≥16 weeks before C; receiving operator completes residual cycle and incident drill unaided (`REQ-SOW-026–027`) |

Console feature implementation, venue execution, validator operation, legal advice, the unselected target, and post-hypercare operations are excluded unless separately contracted.

## Milestones and acceptance

| M | Gate | Acceptance headline |
|---|---|---|
| M0 | — | Mobilized; verification sprint, environments, CI, plan |
| M1 | feeds G0 | Stage A/G0 package, feasibility baseline, migration design, economics v1, Q-15 result |
| M2 | G1 | Selected design/parameters frozen, tokens on devnet, parity harness v1 |
| M3 | — | Feature-complete devnet lifecycle, two exporters, adapter/indexer, internal security current |
| M4 | G2 | Public testnet/claims/SDK/portal, audits underway, exchange pack, R1 |
| M5 | G3 | Audits closed, R3/load pass, verified mainnet deploy, keys/monitoring/bounty, provider onboarding |
| M6 | G4 | S1/claims/exchange/residual cycle 1; reconciliation clean |
| M7 | feeds G5 | All residuals, S2, halt, archives, final conservation |
| M8 | G5 | Post-S2 hypercare exit, handover, warranty transition |

A milestone is achieved only when its deliverables/evidence are accepted and attached gates pass. Partial acceptance requires a punch list and date. Client review target is 10 business days. Fixed payments attach only to accepted milestones; percentages live in the commercial plan (`REQ-SOW-028–032`). Each submission includes a live demo plus test/audit/rehearsal/reconciliation evidence appropriate to the table.

## Effort and team

Planning envelope, not a Vendor commitment:

| WS | Person-months |
|---|---:|
| WS0 program/architecture | 12–16 |
| WS1 economics | 6–9 |
| WS2 on-chain | 40–55 |
| WS3 migration | 18–26 |
| WS4 off-chain | 30–42 |
| WS5 security | 10–14 |
| WS6 quality/testnet | 16–22 |
| WS7 launch/operations | 6–10 |
| WS8 docs/handover | 4–6 |
| **Total** | **142–200** |

Indicative FTE by phase: Stage A 7; build 13; harden 13.5; launch/wind-down 7.5; hypercare 3.5. Core roles: engagement lead, protocol architect, target-chain engineers, Go/backend, TypeScript/indexer/portal, SRE, QA, security/audit liaison, and technical writer. Stage tails are fractional operations.

Name the engagement lead, architect, security lead, and two senior target engineers. Allocate them ≥80% G0→G4; substitutions need consent, equivalent skill, 10 business days' notice, and Vendor-paid overlap. More than two substitutions before G4 triggers remediation/possible termination. Overclock supplies ≥2 counterpart engineers plus product/comms, governance, finance, Console, archive access, and timely decisions (`REQ-SOW-033–035`).

## Decision rights

| Activity | Accountable | Vendor role |
|---|---|---|
| Target and tokenomics decisions | Governance | Evidence/model/consult |
| Implementation | Vendor | Accountable and responsible |
| Deliverable acceptance, audits/contracts, exchange/comms, legal | Overclock | Deliver/fix/support; do not self-authorize |
| Governance proposals | Governance | Technical drafting support |
| S1/S2 and roots | Overclock | Compute/execute/verify |
| Keys/incident command | Overclock | Ceremony and technical/on-call lead |
| Old-chain validator coordination | Overclock | Build artifacts/support |

The Vendor acts on another accountable party's recorded instruction for target, economics, proposals, exchange, or legal matters (`REQ-SOW-036`).

## Commercial controls

- Fixed milestone price is preferred; a capped T&M pool may cover exceptional operations. Rate card is fixed for the engagement (`REQ-SOW-037`).
- Any change to a requirement, acceptance criterion, milestone date, or price needs a CR listing affected IDs, cost, schedule, and risk. Both parties target 10 business days; security emergency work may start on joint written authorization and regularize within 10 days (`REQ-SOW-038–040`).
- Deliverables are Apache-2.0 or MIT compatible, developed publicly under `akash-network` from day one except embargoed security/key material; no incompatible/proprietary critical dependency without CR (`REQ-SOW-041–043`).
- After M8, provide a 90-day warranty at hypercare P1/P2 response. Rebaseline Stage B at M1 through CR (`REQ-SOW-044–045`).
- On termination, deliver current repos, rotated/handed credentials, open items, and continuation report within 15 business days; pay accepted plus demonstrably completed work (`REQ-SOW-046`).

Client-paid exclusions include audits ($400k–$1m planning range), bounty ($250k–$1m ceiling), RPC/indexer/testnet ($10k–$40k/month), testnet rewards, venue fees, ≥5-year archives, $10k–$30k ceremony hardware, and legal. Any Vendor-administered third-party spend is pre-approved pass-through at zero markup (`REQ-SOW-047`). All figures require current quotes.

## Vendor qualification and reporting

Proposal evidence (`REQ-SOW-048–049`): two relevant production protocol launches with ≥$100m secured peak, comparable migration/claims experience, three published audits and exploit disclosure, open-source history, two ≥$1m/12-month references, and named ≥80%-available personnel. Submit per-workstream approach, team/phase staffing, fixed M0–M8 prices for both Stage B paths, rates, numbered assumptions, evidence, and SOW exceptions.

Weekly reports show milestone/workstream RAG, completed/next work, decisions/blockers, risks, requirement/test/audit metrics, budget/forecast, and changes. Monthly steering reviews schedule, spend, risk, staffing, quality, and decisions. Gate reviews use [10](./10-rollout-and-cutover.md) evidence. Escalation runs engineer→workstream lead→program leads→steering with explicit decision SLAs; all delivery and evidence tooling remains client-accessible (`REQ-SOW-050–054`).
